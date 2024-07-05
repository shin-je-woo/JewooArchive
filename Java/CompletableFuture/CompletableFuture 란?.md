# Previous

- CompletableFuture 는 비동기 프로그래밍을 쉽게 다루고 복잡한 비동기 작업을 효과적으로 처리할 수 있도록 해주는 도구로 Java 8 에 도입 되었다.
- CompletableFuture 는 값과 상태를 설정하여 명시적으로 완료 시킬 수 있는 Future 로서 코드의 가독성을 높이고 비동기 작업의 조합을 간단하게 처리할 수 있다.
- Java 에서 비동기 및 병렬 프로그래밍의 능력을 향상시키는 데 중요한 역할을 하고 있는 클래스이다.

# 💡 Future의 단점 및 한계

- Java5에 Future가 추가되면서 비동기 작업에 대한 결과값을 반환 받을 수 있게 되었다. 하지만 Future는 다음과 같은 한계점이 있었다.
  - 외부에서 완료시킬 수 없고, get의 타임아웃 설정으로만 완료 가능
  - 블로킹 코드(get)를 통해서만 이후의 결과를 처리할 수 있음
  - 여러 Future를 조합할 수 없음 ex) 회원 정보를 가져오고, 알림을 발송하는 등
  - 여러 작업을 조합하거나 예외 처리할 수 없음
- 그래서 Java8에서는 이러한 문제를 모두 해결한 CompletableFuture가 등장하게 되었다.
- `CompletableFuture` 는 기존의 Future를 기반으로 외부에서 완료시킬 수 있어서 CompletableFuture라는 이름을 갖게 되었다.
- `Future` 외에도 `CompletionStage` 인터페이스도 구현하고 있는데, `CompletionStage` 는 작업들을 중첩시키거나 완료 후 콜백을 위해 추가되었다.
- 예를 들어 Future에서는 불가능했던 "몇 초 이내에 응답이 안 오면 기본값을 반환한다." 와 같은 작업이 가능해진 것이다.
- 즉, Future의 진화된 형태로써 외부에서 작업을 완료시킬 수 있을 뿐만 아니라 콜백 등록 및 Future 조합 등이 가능하다는 것이다.

# 💡 Future & CompletableFuture

![image](https://github.com/shin-je-woo/TIL/assets/39439576/6eaf0035-a176-4935-b2b3-d53c97bf871e)

![image](https://github.com/shin-je-woo/TIL/assets/39439576/c53bf889-d563-4218-8ff4-81f0c50078b8)

![image](https://github.com/shin-je-woo/TIL/assets/39439576/d539a41b-efee-4cfb-bb70-25e2d987fe5a)
