# Previous

- join() 메서드는 한 스레드가 다른 스레드가 종료될 때까지 실행을 중지하고 대기상태(WAITING)에 들어갔다가 스레드가 종료되면 실행대기 상태로 전환된다.
- 스레드의 순서를 제어하거나, 다른 스레드의 작업을 기다려야 하거나, 순차적인 흐름을 구성하고자 할 때 사용할 수 있다.
- Object 클래스의 `wait()` 네이티브 메서드로 연결되며 시스템 콜을 통해 커널모드로 수행한다. 내부적으로 `wait()` & `notify()` 흐름을 가지고 제어한다.

# 💡 API 및 예외 처리

### void join() throws InterruptedException

- 스레드의 작업이 종료 될 때까지 대기 상태를 유지한다.

### void join(long millis) throws InterruptedException

- 지정한 밀리초 시간 동안 스레드의 대기 상태를 유지한다.
- 밀리초에 대한 인수 값은 음수가 될 수 없으며 음수 일 경우 IllegalArgumentException 이 발생한다.

### void join( long millis, int nanos) InterruptedException

- 지정한 밀리초에 나노초를 더한 시간 동안 스레드의 대기 상태를 유지한다.
- 나노초의 범위는 0 에서 999999 이다.

### InterruptedException

- 스레드가 인터럽트 될 경우 InterruptedException 예외가 발생시킨다
- 다른 스레드는 join() 을 수행 중인 스레드에게 인터럽트, 즉 중단(멈춤) 신호를 보낼 수 있다.
- InterruptedException 예외가 발생하면 스레드는 대기상태에서 실행대기 상태로 전환되어 실행상태를 기다린다.

### 기본 코드

```java
public static void main(String[] args) {
    Runnable r = new MyRunnable();
    Thread thread = new Thread(r);
    thread.start();
    try { // try-catch 문으로 예외 처리 해 주어야 한다
        thread.join(); // 메인 스레드는 thread 의 작업이 종료될 때 까지 일시정지한다
    } catch (InterruptedException e) {
        // 인터럽트 발생 시 예외 처리 필요
    }
}
```

# 💡 join() 작동방식

### wait() & notify()

![image](https://github.com/shin-je-woo/TIL/assets/39439576/94b0fa82-c384-4664-835a-80a43b1466b9)

### interrupt() 발생

![image](https://github.com/shin-je-woo/TIL/assets/39439576/b8058f8e-92e6-4e75-af68-23d0b076b6bc)

# 💡 join() 작동 방식 정리

- join() 을 실행하면 OS 스케줄러는 join() 을 호출한 스레드를 대기 상태로 전환하고 호출 대상 스레드에게 CPU 를 할당한다.
- 호출 대상 스레드의 작업이 종료되면 join() 을 호출한 스레드는 실행 대기 상태로 전환 되고 CPU 가 실행을 재개할 때 까지 기다린다.
- join() 을 호출한 스레드가 실행 대기에서 실행 상태가 되면 그 스레드는 남은 지점부터 실행을 다시 시작한다.
- 호출 대상 스레드가 여러 개일 경우 각 스레드의 작업이 종료될 때 까지 join() 을 호출한 스레드는 대기와 실행을 재개하는 흐름을 반복한다.
- join() 을 호출한 스레드가 인터럽트 되면 해당 스레드는 대기에서 해제되고 실행상태로 전환되어 예외를 처리하게 된다.
