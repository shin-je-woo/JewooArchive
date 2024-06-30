# Previous

- 주어진 지연 시간 후에 명령을 실행하거나 주기적으로 실행할 수 있는 ExecutorService 를 상속한 인터페이스이다.
- 작업의 예약 및 실행을 위한 강력한 기능을 제공하여 시간 기반 작업을 조절하거나 주기적인 작업을 수행하는 데 유용하다.

# 💡 ScheduledExecutorService API

### `ScheduledFuture<?> schedule(Runnable command, long delay, TimeUnit unit)`

- 주어진 지연 시간 이후에 Runnable 작업을 예약하고 ScheduledFuture를 반환한다. 예약된 작업은 한 번만 실행된다.
- command: 실행할 작업, delay: 실행을 지연할 시간, unit: 지연 매개변수의 시간 단위

### `ScheduledFuture<V> schedule(Callable<V> callable, long delay, TimeUnit unit)`

- 주어진 지연 시간 이후에 Callable 작업을 예약하고 ScheduledFuture를 반환한다. 예약된 작업은 한 번만 실행된다.
- callable : 실행할 함수, delay: 실행을 지연할 시간, unit: 지연 매개변수의 시간 단위

### `ScheduledFuture<?> scheduleAtFixedRate(Runnable command, long initialDelay, long period, TimeUnit unit)`

- 초기 지연 시간 이후에 Runnable 작업을 주기적으로 실행하도록 예약하고 ScheduledFuture 를 반환한다. 이후에 주어진 주기로 실행된다.
- command: 실행할 작업, initialDelay: 첫 번째 실행을 지연할 시간, period : 연속적인 실행 사이의 주기, unit: initialDelay와 period 파라미터의 시간 단위

### `ScheduledFuture<?> scheduleWithFixedDelay(Runnable command, long initialDelay, long delay, TimeUnit unit)`

- 초기 지연 이후에 Runnable 작업을 주기적으로 실행하도록 예약하고 ScheduledFuture를 반환한다. 작업이 완료되고 나서 지연 시간 후 실행된다.
- command: 실행할 작업, initialDelay: 첫 번째 실행을 지연할 시간, delay: 연속적인 실행 사이의 지연 시간, unit: initialDelay 및 delay 파라미터의 시간 단위

# 💡 ScheduledFuture

- ScheduledFuture 는 ScheduledExecutorService 를 사용하여 작업을 예약한 결과이다.
- 주요 목적은 지연이나 주기적인 작업 실행을 위한 것이며 결과를 처리하는 것은 아니다.
- 주로 예약작업을 취소하기 위해 사용된다.
- `long getDelay(TimeUnit unit)` : 작업이 실행 되기까지 남은 지연 시간을 반환한다.
