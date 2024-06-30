# 💡 스레드 풀 실행 및 관리

ExecutorService 는 비동기 작업을 실행하고 관리하기 위한 두 가지 메서드를 제공한다.

### `void execute(Runnable r)`

- 작업을 제출하면 작업을 실행하고 종료한다.

### `Future submit(Callable c)`

- 작업을 제출하면 작업을 실행함과 동시에 Future 를 반환한다.
- Future 에는 결과 값을 포함하고 있다.
 
![image](https://github.com/shin-je-woo/TIL/assets/39439576/058d820d-be00-43a5-b491-014a4988dafb)

## execute() vs submit()

![image](https://github.com/shin-je-woo/TIL/assets/39439576/628791f2-af1d-4eef-82ac-7ac8041bbe61)

## API

### `Future<T> submit(Callable<T> task)`

- Callable 작업을 실행하는 메서드로 작업의 결과를 나타내는 Future 를 반환하며 Future 에는 작업의 결과가 저장된다.

### `Future<T> submit(Runnable task, T result)`

- Runnable 작업을 실행하는 메서드로 작업의 결과를 나타내는 Future 를 반환하며 Future 에는 지정된 결과가 저장된다.

### `Future<?> submit(Runnable task)`

- Runnable 작업을 실행하는 메서드로 작업의 결과를 나타내는 Future 를 반환하며 Future 에는 아무런 결과가 존재하지 않는다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/49b14feb-e392-45ee-81bb-7ec9aab8c869)

### 비동기 작업의 결과가 필요없는 경우
  - execute(Runnable)
### 비동기 작업의 결과가 필요한 경우
  - submit(Callable)

# 💡 스레드 풀 중단 및 종료

## 작업 중단 및 종료

ExecutorService 는 스레드 풀을 종료하기 위한 두 가지 메서드를 제공한다.

### `void shutdown()` - 정상적인 스레드 풀 종료

- 이전에 제출된 작업은 실행하고 더 이상 새로운 작업은 수락하지 않는다. 작업이 모두 완료되면 ExecutorService가 종료된다.
- 실행중인 스레드를 강제로 인트럽트 하지 않기 때문에 인트럽트에 응답하는 작업이나 InterruptedException 예외 구문을 작성할 필요가 없다.

### `List<Runnable> shutdownNow()` - 강제적인 스레드 풀 종료

- 이전에 제출된 작업도 취소하고 현재 실행중인 작업도 중단하려고 시도한다. 그리고 작업 대기 중이었던 작업 목록을 반환한다.
- 실행 중인 스레드를 강제로 인터럽트 하지만 해당 작업이 인터럽트에 응답하는 작업이 아닌 경우 작업 종료를 보장하지 않는다.
  - 작업스레드가 작업을 강제로 종료 하기 위해서는 Thread.isInterrupted() 나 sleep() 과 같은 인터럽트 관련 API 를 사용해야 한다.

> shutdown 후 작업을 제출할려고 시도하면 RejectedExecutionException 예외가 발생한다.   
> shutdown 호출한 스레드는 실행 중인 작업이 종료될 때까지 기다리지 않고 바로 다음 라인을 실행한다. 만약 스레드가 메서드 호출 후 블록킹 되기 위해서는 awaitTermination 을 사용해야 한다.

## 작업 종료 대기 및 확인

ExecutorService 는 작업 종료 대기 및 확인을 위한 메서드를 제공한다.

### `boolean awaitTermination(long timeout, TimeUnit unit) throws InterruptedException`

- 종료 요청 후 모든 작업이 실행 완료될 때까지 또는 타임아웃이 발생하거나 현재 스레드가 인터럽트될 때까지 블록된다.
- 종료가 완료되면 true를 반환하고 종료가 타임아웃 발생 전에 완료되지 않으면 false 를 반환한다.

### `boolean isShutdown()`

- ExecutorService 의 shutdown() 또는 shutdownNow() 메서드가 호출 후 종료 절차가 시작되었는지를 나타내며, 종료 절차 중이거나 완전히 종료된 상태일 때 true를 반환한다.

### `boolean isTerminated()`

- ExecutorService 가 완전히 종료되어 더 이상 어떠한 작업도 수행하지 않는 상태인지를 나타낸다.
- 모든 작업과 스레드가 완전히 종료된 후에 true를 반환한다.
- shutdown 또는 shutdownNow가 먼저 호출 되지 않은 경우에는 isTerminated 가 절대 true가 되지 않는다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/5d340a74-fe54-43fb-a3af-be7ec727f3a5)

# 💡 다중 작업 처리

ExecutorService 는 여러 작업을 실행하고 결과를 수집하는 데 사용되는 메서드를 제공한다.

## invokeAll

invokeAll() 은 작업들 중 가장 오래 걸리는 작업 만큼 시간이 소요된다.

### `List<Future> invokeAll(Collection<? extends Callable<T>> tasks) throws InterruptedException`

- 여러 개의 Callable 작업을 동시에 실행하고 모든 작업이 완료될 때까지 블록되며 각 작업의 결과를 나타내는 Future 객체의 리스트를 반환한다.
- 작업이 완료되면 Future.isDone()이 true 가 되며 작업은 정상적으로 종료되거나 예외를 던져 종료될 수 있다.
- 대기 중에 인터럽트가 발생한 경우 아직 완료되지 않은 작업들은 취소된다.

### `List<Future> invokeAll(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit) throws InterruptedException`

- 기본 메서드와 기능은 동일하고 시간 경과와 관련된 부분만 차이가 난다.
- 타임아웃이 발생하거나 모든 작업이 완료될 때 까지 블록되며 각 작업의 결과를 나타내는 Future 객체의 리스트를 반환한다.
- 타임아웃이 발생한 경우 이 작업 중 일부는 완료되지 않을 수 있으며 완료되지 않은 작업은 취소된다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/92b9b117-1871-4c42-b32c-02582ba6eb17)

## invokeAny

invokeAny() 는 작업들 중 가장 짧게 걸리는 작업 만큼 시간이 소요된다.

### `T invokeAny(Collection<? extends Callable<T>> tasks) throws InterruptedException, ExecutionException`

- 여러 개의 Callable 작업을 동시에 실행하고 그 중 가장 빨리 성공적으로 완료된(예외를 던지지 않은) 작업의 결과를 반환한다.
- 어떤 작업이라도 성공적으로 완료하면 블록을 해제하고 해당 작업의 결과를 반환한다.
- 정상적인 반환 또는 예외 발생 시 완료되지 않은 작업들은 모두 취소된다.

### `T invokeAny(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException`

- 기본 메서드와 기능은 동일하고 시간 경과와 관련된 부분만 차이가 난다.
- 주어진 시간이 경과하기 전에 예외 없이 성공적으로 완료된 작업의 결과를 반환한다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/e7314f32-b966-4c81-b4b1-7db6e0a0d68e)
