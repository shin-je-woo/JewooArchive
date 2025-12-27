# Spring Batch에서 JpaPagingItemReader로 읽으면서 같은 조건을 업데이트하면 데이터가 누락되는 이유와 해결책

배치에서 흔히 쓰는 흐름이 있다.

- 특정 조건을 만족하는 레코드를 읽는다.
- 처리 완료 표시로 상태 컬럼을 바꿔서 저장한다.

이때 `JpaPagingItemReader`를 쓰면, 처리 중 상태 변경 때문에 다음 페이지에서 데이터가 건너뛰어져 누락될 수 있다. `OFFSET` 기반 페이징은 조회 결과 집합이 실행 중에 변하면 안정적으로 동작하지 않는다.

## 문제 요약

`JpaPagingItemReader`는 페이지 단위로 쿼리를 반복 실행한다. 내부적으로는 pageSize와 page를 기준으로 `OFFSET` 성격의 페이징을 수행한다.

그런데 Processor나 Writer에서 조회 조건에 영향을 주는 컬럼을 업데이트하면, 다음 페이지를 조회할 때 결과 집합이 줄어든 상태에서 `OFFSET`이 적용된다. 그 순간 남아있는 레코드가 통째로 스킵될 수 있다.

## 왜 누락이 생기는가

예시로 조건이 `status = READY`인 데이터를 pageSize 10으로 읽는다고 가정한다.

1. 첫 조회(Page 0)에서 10개를 읽는다.
2. 처리 과정에서 그 10개의 `status`를 `DONE`으로 바꾸고 저장한다.
3. 다음 조회(Page 1)는 `OFFSET 10`으로 실행된다.
4. 그런데 DB에는 이제 `READY`가 5개만 남아 있다.
5. `READY` 5개에 `OFFSET 10`을 적용하면 결과는 0건이다.
6. 남은 5개는 영원히 읽히지 않고 누락된다.

핵심은 다음과 같다.

- `OFFSET`은 **지금 시점의 결과 집합**에서 앞부분을 건너뛴다.
- 배치 실행 중에 결과 집합이 줄어들면, 건너뛰는 위치가 앞으로 당겨져서 일부가 통째로 빠진다.

`ORDER BY`를 붙여도 정렬의 일관성에는 도움이 되지만, 결과 집합 자체가 줄어드는 문제는 해결하지 못 한다.

## 해결 방법 1: JpaCursorItemReader로 전환

`JpaCursorItemReader`는 커서 스트리밍처럼 동작한다. 페이지를 넘기며 쿼리를 다시 날리는 구조가 아니라, 한 번 만든 결과 흐름을 순차적으로 소비하는 방식에 가깝다. 그래서 처리 중 상태 변경으로 조회 조건을 벗어나더라도, 페이징 누락 문제는 크게 줄어든다.

### 예시 코드

```java
@StepScope
@Bean
public JpaCursorItemReader<MyEntity> reader(
    @Value("#{jobParameters['targetDate']}") LocalDate targetDate
) {
    return new JpaCursorItemReaderBuilder<MyEntity>()
        .name("myCursorReader")
        .entityManagerFactory(entityManagerFactory)
        .queryString("""
            SELECT e
            FROM MyEntity e
            WHERE e.targetDate = :targetDate
              AND e.status = :status
            ORDER BY e.id
        """)
        .parameterValues(Map.of(
            "targetDate", targetDate,
            "status", Status.READY
        ))
        .build();
}
```

### 장점

- 처리 중 업데이트로 조회 조건이 바뀌어도, `OFFSET` 스킵 때문에 빠지는 형태의 누락이 거의 사라진다.
- 페이지 재조회가 없어서 흐름이 단순해진다.

### 단점과 주의점

- 멀티스레드 Step에 그대로 쓰면 위험할 수 있다. 커서 기반 리더는 보통 스레드 세이프하지 않다.
- DB 커넥션을 더 오래 잡고 있을 수 있다. 실행 시간이 길면 운영 관점에서 부담이 될 수 있다.
- 대량 처리에서 메모리와 persistence context 관리가 중요하다. 필요하면 chunk 크기, fetch size, clear 전략을 같이 점검해야 한다.

## 해결 방법 2: 읽기와 쓰기를 분리해서 조회 대상 스냅샷을 고정

`JpaPagingItemReader`를 유지하고 싶으면, 핵심은 **읽는 동안 조회 조건이 변하지 않게** 만드는 거다.

대표적인 패턴은 다음 중 하나다.

- Step 1에서 처리 대상의 PK 목록만 수집한다. (조건 변경 없음)
- Step 2에서 PK 목록 기반으로 조회하고 업데이트한다.

또는

- 처리 대상에 배치 실행 id 같은 마킹 컬럼을 먼저 업데이트해서 스냅샷을 고정한다.
- 그 마킹 값을 기준으로 읽고 처리한다.

이 방식은 안정성이 높지만, 구현이 번거롭고 추가 컬럼이나 임시 저장소가 필요할 수 있다.

## 해결 방법 3: OFFSET 대신 정렬 키 기반 페이징을 쓰는 방식으로 전환

근본적으로 OFFSET 페이징은 변하는 결과 집합에 취약하다. 마지막으로 읽은 키 이후를 기준으로 읽는 키셋 페이징 스타일이 더 안전하다.

JPA 리더만 고집하지 않는다면, JDBC 기반 리더로 정렬 키 중심 페이징을 구성하는 것도 대안이 된다. 다만 이 글의 초점은 JPA라서 선택지로만 언급한다.

## 선택 가이드

- 읽으면서 같은 조건을 업데이트해야 한다.
    - 1순위: `JpaCursorItemReader`
    - 2순위: 스냅샷 패턴 (PK 목록, 마킹 컬럼)
- 병렬 처리나 멀티스레드가 중요하다.
    - 커서 리더는 조심해야 한다.
    - 스냅샷 패턴이나 키 기반 페이징이 더 안전한 경우가 많다.
- 실행 시간이 길고 DB 리소스 점유가 민감하다.
    - 커서 방식은 커넥션 점유를 포함한 운영 리스크를 점검해야 한다.

## 결론

이 이슈의 본질은 **조회 결과 집합이 변하는데 OFFSET 페이징을 계속 적용하는 것**이다. `JpaPagingItemReader`는 구조적으로 이 상황에 취약하고, `JpaCursorItemReader`는 흐름 자체가 달라서 누락 문제를 크게 줄일 수 있다.
