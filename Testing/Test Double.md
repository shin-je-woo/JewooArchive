## 💡 테스트 대역이란

- 테스트 대역(Test Double)은 테스트 중 실제 객체를 대신해 사용하는 객체다.
- 테스트 환경을 분리하고 예측 가능하게 만들며, 빠르고 신뢰할 수 있는 테스트 작성에 중요한 도구다.
- 테스트 대역은 단순히 기술이 아니라 **유연한 설계를 가능하게 하는 사고방식**이며, 테스트 피라미드에서 소형 테스트를 더 많이 만들기 위해 반드시 필요하다.
- "대역"은 '대역폭(bandwidth)'이 아닌 영화에서 **스턴트맨처럼 대신 행동하는 사람**을 의미.
- 즉, **실제 객체를 대체해 행동하는 테스트용 객체**를 의미한다.
- 영어로는 **test double**, 흔히 더블(double)이라고도 부른다.

### 테스트 대역이 필요한 이유

- 테스트 중 외부 API, 데이터베이스, 메시지 브로커 등과의 연결은 느리거나 불안정할 수 있음
- 외부 의존성을 제거함으로써 테스트 실패율을 줄이고 테스트 환경을 단순화할 수 있음
- 빠른 실행과 일관된 결과를 갖는 테스트 작성 가능
- 테스트 가능한 설계(인터페이스 기반)를 유도함

## 💡 Dummy

- 정의: 아무 동작도 하지 않으며, 단순히 **자리를 채우기 위한 객체**
- 용도: 테스트에 사용되지 않지만, 메서드 시그니처나 생성자에 인자가 필요한 경우

```java
public class DummyVerificationEmailSender implements VerificationEmailSender {
    @Override
    public void send(User user) {
        // 아무 동작 없음
    }
}

-> 테스트 대상 클래스에 주입하여 실제 동작 없이 테스트 진행 가능
```

## 💡 Stub

-	정의: 미리 정해진 값을 반환하거나 예외를 던지는 객체
-	용도: 특정 흐름이나 조건을 유도할 때 사용
-	특징: 상태 기반 검증에 적합

```java
public class StubClockHolder implements ClockHolder {
    @Override
    public long millis() {
        return 1672531200000L; // 고정된 시간
    }
}

-> Stub을 통해 회원가입 시각, 토큰 만료 시간 등을 테스트에 맞게 고정할 수 있음
```

## 💡 Fake

- 정의: 실제 구현과 유사하지만 간단한 대체 구현체 (ex. In-memory DB)
-	용도: 외부 리소스 의존성을 제거하고 빠른 테스트 환경 구축
-	특징: Fake는 테스트를 위한 자체적인 논리를 갖고 있다.

```java
public class FakeUserRepository implements UserRepository {
    private final Map<String, User> store = new HashMap<>();

    @Override
    public User save(User user) {
        store.put(user.getEmail(), user);
        return user;
    }

    @Override
    public Optional<User> findByEmail(String email) {
        return Optional.ofNullable(store.get(email));
    }
}
```

## 💡 Mock

- 정의: 사전에 정의한 기대 동작이 있었는지 검증하는 객체.
-	용도: 메서드 호출 및 상호 작용을 기록하고, 실제로 상호 작용이 일어났는지. 어떻게 상호 작용이 일어났는지를 확인하는 데 사용
-	특징: 행위 기반 테스트에 적합

```java
EmailSender mockSender = mock(EmailSender.class);

UserService service = new UserService(mockSender);
service.register("test@example.com");

verify(mockSender).send("test@example.com", "Welcome!");
```

## 💡 Spy

- 정의: 실제 동작을 수행하면서 호출 여부나 인자 등을 기록하는 객체 (앞선 대역들과 다르게 진짜 객체임)
-	용도: 실제 구현과 호출 기록을 동시에 확인할 필요가 있을 때

```java
class SpyEmailSender implements EmailSender {
    private boolean called = false;

    @Override
    public void sendVerificationRequired(User user) {
        called = true;
    }

    public boolean wasCalled() {
        return called;
    }
}
```

### 📌 Mock vs Spy

Mock과 Spy는 모두 **행위 기반 테스트**에서 사용되는 테스트 대역이지만, 그 목적과 동작 방식에는 분명한 차이가 있다.   
Mock은 **행위를 감시만 한다**고 이해하면 되고, Spy는 **진짜처럼 행동하면서 감시도 한다**고 이해하면 된다.

| 항목           | Mock                                                  | Spy                                                   |
|----------------|-------------------------------------------------------|--------------------------------------------------------|
| 목적           | **기대하는 행위가 발생했는지 검증**                   | **실제 동작 여부 + 호출 여부 확인**                   |
| 동작 여부      | 실제 동작하지 않음 (모두 시뮬레이션)                  | 실제 객체처럼 동작함                                   |
| 기본 동작      | 반환값은 설정하지 않으면 `null`, 0, false 등 기본값   | 원래 객체의 로직을 그대로 따름                         |
| 행위 검증      | `verify()`를 통해 **정확한 호출 여부 및 횟수** 확인    | `verify()`도 가능하며, 직접 호출 여부를 기록하는 방식도 사용 |
| 예시 상황      | 메서드가 호출되었는지, 인자가 정확했는지를 검증       | 메서드가 실제로 수행되었는지 + 로직까지 함께 확인       |
| 실사용 목적    | 외부 시스템 호출 여부만 검증할 때 유용                | **기능도 수행**하고 **검증도 필요한 경우** 적합        |
| 대표 활용 예   | 메일 전송, 알림 발송 등 외부 시스템 호출 검증         | 리포지토리 저장, 내부 카운트 증가 등 실제 동작 + 검증 |

```java
@Test
public void mock_versus_spy() {
    // given
    List<Integer> mockedList = Mockito.mock(ArrayList.class);
    List<Integer> spyList = Mockito.spy(new ArrayList<Integer>());

    // when
    mockedList.add(1);
    spyList.add(1);


    // then 1
    // 둘 다 add 메서드가 호출된 것을 확인할 수 있습니다.
    verify(mockedList).add(1);
    verify(spyList).add(1);

    // then 2
    // 하지만 List 객체의 내부 상태 변화가 다릅니다.
    assertThat(mockedList.size()).isEqualTo(0); // Mock에는 값이 들어있지 않습니다.
    assertThat(spyList.size()).isEqualTo(1); // Spy에는 실제로 값이 들어갑니다.
)
```

### 📌 상태 기반 검증 & 행위 기반 검증

단위 테스트에서 결과를 검증하는 방법은 크게 두 가지가 있다.

### 상태 기반 검증 (State-Based Verification)

- 테스트 후 객체의 **상태나 반환값을 검사**하여 기대한 결과가 나왔는지 확인하는 방식
- **입력 → 상태 변화 → 출력**이 테스트 포인트
- 주로 **Stub, Fake**와 함께 사용됨

```java
@Test
void 유저는_북마크를_toggle_해서_제거_할_수_있다() {
    // given
    User user = User.builder()
            .bookmark(new ArrayList<>())
            .build();
    user.appendBookmark("foobar"); // 북마크를 미리 추가해둡니다.

    // when
    user.toggleBookmark("foobar");

    // then
    // user는 foobar를 북마크로 갖고 있어선 안 됩니다.
    assertThat(user.hasBookmark("foobar")).isFalse();
}
```

### 행위 기반 검증 (Behavior-Based Verification)

- 테스트 대상이 협력 객체에게 어떤 메시지를 보냈는지 검증하는 방식
-	메서드가 호출되었는지, 어떤 인자로 호출되었는지 확인
-	주로 Mock, Spy와 함께 사용됨


```java
@Test
void 유저는_북마크를_toggle_해서_제거_할_수_있다() {
    // given
    User user = User.builder()
            .bookmark(new ArrayList<>())
            .build();
    user.appendBookmark("foobar"); // 북마크를 미리 추가해둡니다.

    // when
    user.toggleBookmark("foobar");

    // then
    // user.removeBookmark("foobar")가 호출됐는지 확인합니다.
    verify(user).removeBookmark("foobar");
}
```
