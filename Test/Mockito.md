# 💡 Mock과 Mockito 개념
- Mock
  - 진짜 객체와 비슷하게 동작하지만 프로그래머가 직접 그 객체의 행동을 관리하는 객체.
  - 실제 객체를 만들어 사용하기에 시간, 비용 등의 Cost가 높거나 혹은 객체 서로간의 의존성이 강해 구현하기 힘들 경우 가짜 객체를 만들어 사용한다.
  - Java Mock Framework: Mockito, EasyMock, JMock
- Mock객체
  - **모의 객체**(Mock Object)란 주로 객체 지향 프로그래밍으로 개발한 프로그램을 테스트 할 경우 **테스트를 수행할 모듈과 연결되는 외부의 다른 서비스나 모듈들을 실제 사용하는 모듈을 사용하지 않고 실제의 모듈을 "흉내"내는 "가짜" 모듈**을 작성하여 테스트의 효용성을 높이는데 사용하는 객체이다.
  - **사용자 인터페이스(UI)나 데이터베이스 테스트 등과 같이 자동화된 테스트를 수행하기 어려운 때 널리 사용된다.**
  - 실제 객체를 만들기에는 비용과 시간이 많이 들거나 의존성이 크게 걸쳐져 있어서 테스트 시 제대로 구현하기 어려울 경우 가짜 객체를 만들어서 사용한다.
- Mockito
  - Mock 객체를 쉽게 만들고 관리하고 검증할 수 있는 방법을 제공한다.

# 💡 Mock 객체가 필요한 이유
```java
@Test
void createStudyService() {
    // StudyService에 의존성을 주입해주기 위해 MemberService의 구현체를 만들어준다.
    MemberService memberService = new MemberService() {
        @Override
        public Member findById(Long memberId) {
            return null;
        }
    };

    // StudyService에 의존성을 주입해주기 위해 StudyRepository의 구현체를 만들어준다.
    StudyRepository studyRepository = new StudyRepository() {
        @Override
        public Study save(Study study) {
            return null;
        }
    };

    // StudyService를 만들기 위해선 MemberService와 StudyRepository가 필요하다.
    StudyService studyService = new StudyService(memberService, studyRepository);

    assertNotNull(studyService);
}
```
- **위와 같이 구현체는 없지만, 어떤 객체를 만들때 구현체가 필요한 경우가 Mock 객체를 만들기 가장 좋은 순간이다.**
  - 위 코드에서 StudyService를 만들기 위해선 MemberService와 StudyRepository가 필요하다.
  - 그래서 두 구현체를 직접 만들어 주입해주었다.
- **하지만 Mock을 사용할 경우 직접 구현해주지 않아도 된다.**
- Mokcito로 Mock을 사용하는 방법을 알아보자.

# 💡 Mock 객체 만들기
### 1. Mockito.mock() 메서드로 만드는 방법
```java
@Test
void createStudyService() {
    // Mock 객체를 만들어준다.
    MemberService memberService = Mockito.mock(MemberService.class);
    StudyRepository studyRepository = Mockito.mock(StudyRepository.class);

    StudyService studyService = new StudyService(memberService, studyRepository);

    assertNotNull(studyService);
}
```

### 2. @Mock애노테이션으로 만드는 방법
- JUnit 5 extension으로 `MockitoExtension` 을 사용해야 한다.
- 필드와 메서드 매개변수에 사용할 수 있다.

```java
// 필드
@ExtendWith(MockitoExtension.class)
class StudyServiceTest {

    @Mock
    private MemberService memberService;

    @Mock
    private StudyRepository studyRepository;

    @Test
    void createStudyService() {
        StudyService studyService = new StudyService(memberService, studyRepository);

        assertNotNull(studyService);
    }
}
```

```java
// 매개변수
@ExtendWith(MockitoExtension.class)
class StudyServiceTest {

    @Test
    void createStudyService(@Mock MemberService memberService, @Mock StudyRepository studyRepository) {
        StudyService studyService = new StudyService(memberService, studyRepository);

        assertNotNull(studyService);
    }
}
```

# 💡 Stubbing (Mock 객체 행위 정의하기)
- Stubbing이란 Mock 객체를 조작하는 것을 의미한다.
  - 모든 Mock 객체의 기본 행동 (조작하지 않는 경우)
  - `return`값이 있는 메서드는 `Null`을 리턴한다.
  - `Optional`의 경우는 `Optional.empty`을 리턴한다.
  - 상태로 `Primitive` 타입을 가지고 있다면 기본값을 가진다. (int는 0, boolean이면 false...)
  - 컬렉션은 비어있는 컬렉션
  - `void` 메서드는 예외를 던지지 않고 아무런 일도 발생하지 않는다.
 
### Mock 객체 조작
- 특정한 매개변수를 받은 경우 특정한 값을 리턴하거나 예외를 던지도록 만들 수 있다.
  - [Mockito Stubbing 공식 문서](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#stubbing)
  - [Argument Matchers (여러가지 매개변수 조작 방법)](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#argument_matchers)
 
```java
Member member = new Member(1L, "shinjw0926@naver.com");

// Stubbing -> 이 메서드 안에서 memberService.findById(1L)을 하면 위의 member가 리턴된다.
when(memberService.findById(1L)).thenReturn(member);
assertAll(
    () -> assertEquals(memberService.findById(1L).getId(), 1L),
    () -> assertEquals(memberService.findById(1L).getEmail(), "shinjw0926@naver.com");
);
```

- void메서드 특정 매개변수를 받거나 호출된 경우 예외를 발생시킬 수 있다.
  - [Stubbing void methods with exceptions](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#5)

```java
// memberService.validate()는 void 메서드
doThrow(new IllegalArgumentException()).when(memberService).validate(1L);
assertThrows(IllegalArgumentException.class, () -> memberService.validate(1L));
assertDoesNotThrow(() -> memberService.validate(2L));
```

- 메서드가 동일한 매개변수로 여러변 호출될 때 각기 다르게 행동하도록 조작할 수 있다.
  - [Stubbing consecutive calls](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#10)

```java
Member member = new Member(1L, "shinjw0926@naver.com");

when(memberService.findById(any()))
    .thenReturn(member)
    .thenThrow(new RuntimeException())
    .thenReturn(null);

assertEquals(memberService.findById(1L).getEmail(), "shinjw0926@naver.com");
assertThrows(RuntimeException.class, () -> memberService.findById(1L));
assertNull(memberService.findById(1L));
```
