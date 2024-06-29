# 💡 Previous

- 자바에서 Future 는 비동기 작업의 결과를 나중에 가져올 수 있도록 도와주는 인터페이스이다.
- Future 는 비동기 작업이 완료되었는지 여부를 확인할 수 있고 조건에 따라 작업을 취소할 수 있으며 작업의 결과를 얻는 방법을 제공한다.
- Future 는 작업의 결과를 가져올 때까지 블로킹되며 여러 작업을 조합하는 문제, 예외 처리의 어려움 등이 존재하는데 이런 단점을 보완하기 위해 자바 8부터는 CompletableFuture 와 같은 개선된 비동기 도구들이 제공되고 있다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/fdcf5475-836c-4c2b-855c-03e59bcfa503)

# 💡 Future & Callable

- Future 와 Callable 은 자바에서 비동기 작업을 처리하기 위해 서로 관련된 관계를 가진 클래스들이다.
  - Callable 은 비동기 작업의 실행 내용을 정의하는 인터페이스로서 call() 메서드를 구현하여 작업을 정의하고 작업의 결과를 반환한다.
  - Future 은 Callable 에서 반환한 비동기 작업의 결과를 가져오기 위한 인터페이스로서 작업이 완료될 때까지 결과를 기다리거나 작업이 완료되면 결과를 가져올 수 있다.
- ExecutorService 의 submit() 메서드는 Callable 을 받아 작업을 실행하고 Future를 반환 한다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/70a19697-d0eb-4270-8c25-e17e2c0cfbf9)

# 💡 Future API

### boolean cancel(boolean mayInterruptIfRunning);

- 작업이 이미 완료되었거나 취소되었거나 다른 이유로 취소할 수 없는 경우 => 아무런 일도 발생하지 않으며 false 를 반환한다.
- 작업이 시작되지 않은 경우 => 해당 작업은 실행되지 않으며 true 를 반환한다.
- 작업이 이미 시작된 경우 => mayInterruptIfRunning 매개변수에 따라 결정된다.
  - mayInterruptIfRunning 파라미터가 true 인 경우 => 해당 작업을 중지시키기 위해 현재 작업을 실행 중인 스레드를 인터럽트 한다. 작업결과를 가져 올 때 취소 예외가 발생한다.
  - mayInterruptIfRunning 파라미터가 false 인 경우 => 진행 중인 작업을 완료 할 수 있다. 작업결과를 가져 올 때 취소 예외가 발생한다.
- 이 메서드의 반환 값은 작업이 현재 취소되었는지 여부를 반드시 나타내지 않기 때문에 정확한 결과는 isCancelled 메서드를 사용해야 한다.
- 작업과 연관된 모든 대기 중인 스레드를 깨우고 시그널을 보내며 done()을 호출하고 callable을 null로 설정한다.

### boolean isCancelled()

- 이 작업이 정상적으로 완료되기 전에 취소되었으면 true 를 반환한다.
- 취소되었는지 여부를 명확하게 확인할 때 사용한다.

### boolean isDone()

- 작업이 완료된 경우 true를 반환하며 완료는 정상 종료, 예외 또는 취소를 포함한 모든 경우에 true를 반환한다.

### V get() throws InterruptedException, ExecutionException

- 작업 결과를 반환하며 작업이 완료될 때 까지 스레드는 대기한다. (블로킹 된다.)
- 작업이 예외를 던져 실패하면 해당 예외가 Future.get()을 호출할 때 발생한다. 실패했을 경우 실패한 정보나 결과값을 Future 에서 얻어 올 수 없고 예외 구문에서 확인해야 한다.
- 예외 발생
  - `CancellationException` : 작업이 취소된 경우
  - `ExecutionException` :­ 작업이 예외를 발생시킨 경우
  - `InterruptedException` :­ 현재 스레드가 대기 중에 인트럽트 된 경우
### V get(long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException
- 지정한 시간 동안 대기하고 해당 시간 내에 완료되면 작업결과를 반환하고 시간 내에 완료되지 않으면 `TimeoutException` 예외를 발생시킨다.

# 💡 FutureTask - Future의 구현체

![image](https://github.com/shin-je-woo/TIL/assets/39439576/cae2939c-c1a0-4434-995d-949b12814693)

# 💡 작업 취소 흐름도

![image](https://github.com/shin-je-woo/TIL/assets/39439576/2dfa782b-da1a-4eec-a939-7cc7e077bf53)
