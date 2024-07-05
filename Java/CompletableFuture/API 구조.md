# Previous

- CompletableFuture 는 비동기 작업과 함수형 프로그래밍의 콜백 패턴을 조합한 Future 라 할 수 있으며 2 가지 유형의 API 로 구분할 수 있다.
- `CompletableFuture` 는 `Future` 와 `CompletionStage` 를 구현한 클래스로서 Future + CompletionStage 라고 정의할 수 있다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/4078345a-6205-49eb-a145-d57c8e6ef8ac)

### 📌 CompletionStage

- CompletionStage 는 비동기 작업을 위한 콜백 함수 API 를 제공하며 어떤 작업이 완료된 후에 실행되어야 하는 후속 작업을 정의하는 데 사용된다.
- CompletionStage 는 "이 작업이 끝나면, 그 다음에 이 작업을 해라"와 같은 연쇄적인 비동기 작업을 표현하기 위한 도구로 사용된다.
- 한 작업의 완료는 자동으로 다음 작업의 시작을 트리거할 수 있어, 여러 비동기 작업들을 연속적으로 연결하여 실행할 수 있게 해 준다.

# 💡 CompletableFuture API - 비동기 작업을 시작, 실행, 완료 등의 API 제공

![image](https://github.com/shin-je-woo/TIL/assets/39439576/1bd80026-aa68-42a6-9286-9d0520a0f37d)

# 💡 CompletionStage API - 비동기 작업을 조작, 예외처리 등의 API 제공

![image](https://github.com/shin-je-woo/TIL/assets/39439576/9262881e-4374-491c-adfb-71bc5109583f)

# 💡 비동기 작업 유형

![image](https://github.com/shin-je-woo/TIL/assets/39439576/44e7bad2-1f50-4d18-84a2-3f2c4ceded71)

# 💡 비동기 작업 실행 객체

- CompletableFuture API 를 실행하면 내부적으로 API 와 연결되는 객체가 생성되어 동기 또는 비동기 작업을 수행하게 된다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/c478d46b-b4d6-44ee-b15b-2b384f0e6578)
