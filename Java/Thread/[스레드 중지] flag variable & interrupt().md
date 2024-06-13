# Previous

- 자바에서는 무한 반복이나 지속적인 실행 중에 있는 스레드를 중지하거나 종료할 수 있는 API 를 더 이상 사용할 수 없다. (suspend(), stop())
- 스레드를 종료하는 방법은 플래그 변수를 사용하거나 interrupt() 를 활용해서 구현할 수 있다.

# 💡 Flag Variable

- 플래그 변수의 값이 어떤 조건에 만족할 경우 스레드의 실행을 중지하는 방식
- 플래그 변수는 동시성 문제로 가능한 `atomic` 변수나 `volatile` 키워드를 사용하도록 한다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/5c9767a6-f4a2-4752-9050-429ec6f3368f)

# 💡 interrupted() & isInterrupted()

- 실행 중인 스레드에 interrupt() 하게 되면 인터럽트 상태를 사용해서 종료기능을 구현할 수 있다..
- interrupt() 한다고 해서 스레드가 처리하던 작업이 중지되는 것이 아니며, 인터럽트 상태를 활용하여 어떤 형태로든 스레드를 제어할 수 있다.

```java
public static class MyRunnable implements Runnable {
    @Override
    public void run() {
        while (!Thread.interrupted()) {
            log.debug("Thread is running");
        }
        log.debug("Task is completed");
    }
}
```

- 스레드가 실행되면 `Thread.interrupted()` 가 false 이므로 반복문을 계속 실행한다.
- 인터럽트가 발생하면 `Thread.interrupted()` 는 true 이고, 반복문을 빠져 나오면서 스레드는 종료된다.
- 인터럽트 상태는 해제 된다. (interrupted = false)

```java
public static class MyRunnable implements Runnable {
    @Override
    public void run() {
        while (!Thread.currentThread().isInterrupted()) {
            log.debug("Thread is running");
        }
        log.debug("Task is completed");
    }
}
```

- 스레드가 실행되면 Thread.currentThread().isInterrupted() 가 false 이므로 반복문을 계속 실행한다
- 인터럽트가 발생하면 Thread.currentThread().isInterrupted() 은 true 이고, 반복문을 빠져 나오면서 스레드는 종료된다.
- 인터럽트 상태는 계속 유지 된다. (interrupted = true)

# 💡 InterruptedException

- 대기 중인 스레드에 interrupt() 하게 되면 `InterruptedException` 예외가 발생한다.
- 이 예외 구문에서 종료기능을 구현할 수 있다.

```java
public static class MyRunnable implements Runnable {
    @Override
    @SuppressWarnings("BusyWait")
    public void run() {
        while (true) {
            try {
                Thread.sleep(1000);
                log.debug("Thread is running");
            } catch (InterruptedException e) {
                log.debug(e.getMessage());
                break;
            }
        }
        log.debug("Task is completed");
    }
}
```

- 대기 중인 스레드에 인터럽트가 발생하면 `InterruptedException` 예외가 발생하고 예외 구문에서 반복문을 빠져 나오면서 스레드는 종료된다.
- 인터럽트 상태는 해제 된다. (interrupted = false)

```java
public static class MyRunnable implements Runnable {
    @Override
    public void run() {
        while (true) {
            try {
                if (Thread.interrupted()) {
                    throw new InterruptedException("interrupt occurred");
                }
                log.debug("Thread is running");
            } catch (InterruptedException e) {
                log.debug(e.getMessage());
                Thread.currentThread().interrupt(); // 필요한 경우 인터럽트 상태를 원복한다.
                break;
            }
        }
        log.debug("Task is completed");
    }
}
```

- 대기중인 스레드가 아니라면 인터럽트가 발생해도 `InterruptedException` 예외가 발생하지 않으므로 강제로 발생시키는 방법도 있다.
- 인터럽트가 발생하면 강제로 `InterruptedException` 예외를 던지고 예외 구문에서 반복문을 빠져 나오면서 스레드는 종료된다.
- 인터럽트 상태는 해제 된다. (interrupted = false)
