# RDS 복제 아키텍처

## 1. 분류 기준

- RDS의 복제 구조는 **고가용성(HA)**, **읽기 확장(Read Scaling)**, **장애 복구(DR)** 라는 세 목적을 기준으로 나눠서 이해해야 한다.
- 같은 복제라도 무엇을 최우선으로 두느냐에 따라 구조가 달라진다.
- Multi-AZ DB 인스턴스는 고가용성 중심, Read Replica는 읽기 확장 중심, Multi-AZ DB 클러스터는 고가용성과 읽기 처리를 함께 고려한 구조다.

## 2. Single-AZ, Multi-AZ DB 인스턴스, Read Replica, Multi-AZ DB 클러스터

### Single-AZ

- 단일 인스턴스 구조다.
- 장애 전환 대상이 없고, 읽기 분산용 복제본도 없다.
- 구조가 가장 단순하지만, 고가용성과 읽기 확장 측면에서는 기준점 역할만 한다.

### Multi-AZ DB 인스턴스

- Primary와 별도의 AZ에 있는 **동기식 standby**로 구성된다.
- 핵심 목적은 장애 대비다.
- standby는 failover 지원을 위해 유지되며 **읽기 트래픽은 받지 않는다**.
- 따라서 이 구조는 성능 확장보다 서비스 지속성을 위한 구조다.
- 동기 복제 특성상 Single-AZ보다 **쓰기와 commit 지연이 증가할 수 있다**.

### Read Replica

- 읽기 전용 복제본이다.
- RDS는 먼저 소스 인스턴스의 스냅샷으로 복제본을 만든 뒤, 이후 변경분은 **DB 엔진의 비동기 복제 방식**으로 반영한다.
- 애플리케이션은 일반 DB 인스턴스처럼 read replica에 접속할 수 있고, 주 용도는 읽기 부하 분산, 리포팅, 데이터 웨어하우징성 조회이다.
- 필요하면 replica를 승격해 독립 인스턴스로 전환할 수도 있다.

### Multi-AZ DB 클러스터

- writer 1대와 reader 2대가 **3개의 서로 다른 AZ**에 배치되는 구조다.
- 이 구조는 DB 엔진의 **네이티브 복제 기능**을 사용하고, **반동기식(semisynchronous) 복제**를 사용한다.
- 즉 writer의 변경은 각 reader로 전달되고, **최소 1개의 reader가 수신 확인을 해야 commit**이 성립한다.
- reader는 failover 대상이면서 동시에 읽기 트래픽도 처리한다.
- AWS는 이 구조가 Multi-AZ DB 인스턴스보다 **더 높은 읽기 처리량과 더 낮은 쓰기 지연**을 제공할 수 있다고 설명한다.

## 3. 복제 계층의 차이

- 이론적으로 보면 Multi-AZ DB 인스턴스와 Read Replica는 복제라는 이름은 같아도 성격이 다르다.
- Multi-AZ DB 인스턴스는 서비스 관점에서 **primary-standby 동기 복제 기반의 failover topology**다. 평상시 standby는 사용자 트래픽을 처리하지 않고, 장애 시 대체 노드가 되는 데 집중한다. 즉, **가용성 확보를 위해 유휴 자원을 허용하는 구조**다.
- 반면 Read Replica는 **엔진 복제 기반의 scale-out topology**다. 평상시 트래픽을 받아 일을 하고, 원본과의 최신성 차이를 일부 감수하는 대신 읽기 처리량을 늘린다. 즉, **강한 최신성보다 처리량과 확장성을 우선하는 구조**다.
- Multi-AZ DB 클러스터는 이 둘의 중간에 있다. standby를 놀리지 않고 reader로 활용하지만, 여전히 엔진 기반 복제이므로 **lag이 완전히 사라지지는 않는다**. 즉 HA와 활용성의 절충형이라고 보는 게 적절하다.

## 4. 동기, 비동기, 반동기 복제의 의미

### 동기 복제

- 동기 복제는 primary의 commit 경로 안에 복제 확인이 들어간다.
- 따라서 장애 시 데이터 손실 가능성을 줄이고 failover 시 일관성을 높인다.
- 대신 쓰기 경로가 느려질 수 있다.
- Multi-AZ DB 인스턴스가 이 철학에 가깝다.
- AWS도 이 구조에서 동기 standby와 쓰기/commit 지연 증가 가능성을 명시한다.

### 비동기 복제

- 비동기 복제는 primary의 commit이 먼저 완료되고, replica는 뒤따라간다.
- 읽기 확장에는 유리하지만 **stale read**와 **replica lag**을 구조적으로 피할 수 없다.
- AWS는 MySQL read replica에서 `ReplicaLag`와 `BinLogDiskUsage` 증가가 발생할 수 있다고 설명하며, 소스의 높은 write volume과 replica 쪽 적용 직렬화가 lag의 원인이 될 수 있다고 밝힌다.

### 반동기 복제

- 반동기 복제는 동기와 비동기 사이의 절충안이다.
- Multi-AZ DB 클러스터에서는 적어도 한 reader의 acknowledgment가 필요하지만, **모든 replica에서 완전히 실행 완료될 때까지 기다리지는 않는다**.
- 이 때문에 순수 비동기보다는 안전하고, 순수 동기보다는 쓰기 경로가 가볍다.
- 다만, lag 자체는 여전히 존재할 수 있다.

## 5. 장애조치 관점의 차이

- Multi-AZ DB 인스턴스의 장애조치는 비교적 단순하다. standby는 이미 동기 상태로 유지되기 때문에, 핵심은 새 primary로의 전환이다. 따라서 이 구조의 설계 목적은 장애 시 바로 넘길 수 있는 대기 노드 확보에 있다.
- Read Replica는 기본적으로 failover용 자동 대체 노드가 아니다. 필요하면 **수동 승격**을 통해 독립 인스턴스로 바꿀 수 있지만, 이는 본래 읽기 확장 구조를 재구성하는 행위다. 즉 read replica는 본질적으로 HA보다는 scale-out 구조다.
- Multi-AZ DB 클러스터의 failover는 더 복합적이다. reader 중 하나가 새 writer가 되는데, AWS는 **가장 최근 변경 기록을 가진 reader**를 기준으로 failover를 관리한다고 설명한다. 또한 MySQL Multi-AZ DB 클러스터에서는 남아 있는 reader들이 **미적용 트랜잭션을 먼저 반영한 뒤** 새 writer 승격이 진행되므로, failover 시간은 lag의 영향을 받는다.

## 6. 하이브리드 구조의 의미

- 실무에서 자주 쓰는 구조는 **Multi-AZ DB 인스턴스 + Read Replica** 조합이다. 이 구조는 역할 분리가 명확하다.
    - **Multi-AZ DB 인스턴스**: 서비스 연속성을 보장하는 HA
    - **Read Replica**: 조회 트래픽을 분산하는 scale-out
- 즉, 장애를 버티는 구조와 부하를 나누는 구조를 따로 가져가는 방식이다. AWS도 Multi-AZ 배포에 read replica를 함께 둘 수 있으며, 이 경우 primary는 standby에 동기 복제하고 read replica에는 비동기 복제한다고 설명한다.

## 7. 이론적으로 따라오는 운영 함의

- 첫째, **읽기 일관성 모델이 달라진다**. primary는 최신 쓰기를 즉시 반영하지만, read replica는 뒤따라가기 때문에 read-after-write 보장이 약해진다. 따라서 최신성이 중요한 조회는 primary로, 지연 허용이 가능한 조회는 replica로 보내는 라우팅 전략이 필요하다. 이건 옵션이 아니라 비동기 복제를 쓰는 순간 따라오는 설계 원칙이다.
- 둘째, **lag은 단순한 성능 문제가 아니라 failover 시간과 데이터 최신성 문제**다. Multi-AZ DB 클러스터에서는 lag이 failover 시간에 직접 연결되고, read replica에서는 stale read와 모니터링 포인트가 된다. 그래서 `ReplicaLag`는 단순 관측 지표가 아니라 아키텍처의 한계와 상태를 보여주는 핵심 지표다.
- 셋째, **엔진 기반 복제는 테이블 설계의 영향을 더 크게 받는다**. AWS는 RDS for MySQL Multi-AZ DB 클러스터에서 **모든 테이블에 primary key를 둘 것을 강하게 권장**한다. 이는 단순 권고가 아니라 복제 오류와 apply 효율을 줄이기 위한 구조적 요구사항으로 이해하는 게 맞다.

## 8. 핵심 정리

- RDS MySQL 복제 아키텍처의 본질은 다음 세 문장으로 요약할 수 있다.
- **Multi-AZ DB 인스턴스는 고가용성을 위한 동기식 standby 구조다.** standby는 읽기 확장용이 아니다.
- **Read Replica는 읽기 확장을 위한 비동기 엔진 복제 구조다.** 읽기 처리량은 늘어나지만 최신성 차이와 lag을 감수해야 한다.
- **Multi-AZ DB 클러스터는 고가용성과 읽기 처리를 통합한 반동기식 3노드 구조다.** reader를 활용할 수 있지만, 엔진 기반 복제인 만큼 lag과 failover 적용 지연을 계속 관리해야 한다.
