# Previous

- 기본적으로 스레드의 run() 은 예외를 던질 수 없기 때문에 예외가 발생할 경우 run() 안에서만 예외를 처리해야 한다.
- RuntimeException 타입의 예외가 발생할 지라도 스레드 밖에서 예외를 캐치할 수 없고 사라진다.
- 자바에서는 스레드가 비정상적으로 종료되었거나 특정한 예외를 스레드 외부에서 캐치하기 위해서 `UncaughtExceptionHandler` 인터페이스를 제공한다.

# 💡 UncaughtExceptionHandler

- 캐치 되지 않는 예외에 의해 Thread가 갑자기 종료했을 때에 호출되는 핸들러 인터페이스
- 어떤 원인으로 인해 스레드가 종료되었는지 대상 스레드와 예외를 파악할 수 있다.

```java
// 예외가 발생하면 uncaughtException 이 호출되고 대상 스레드 t 와 예외 e 가 인자로 전달된다.
@FunctionalInterface
public interface UncaughtExceptionHandler {
    void uncaughtException(Thread t, Throwable e);
}
```

# 💡 예외 관련 Thread API

- static void setDefaultUncaughtExceptionHandler
  - 모든 스레드에서 발생하는 uncaughtException 을 처리하는 정적 메서드
- void setUncaughtExceptionHandler(UncaughtExceptionHandler ueh)
  - 대상 스레드에서 발생하는 uncaughtException 을 처리하는 인스턴스 메서드
  - setDefaultUncaughtExceptionHandler 보다 우선순위가 높다.
 
```java
public class UncaughtExceptionHandlerExample {

    private static final Logger LOGGER = Logger.getLogger(UncaughtExceptionHandlerExample.class.getName());

    public static void main(String[] args) {
        Thread thread = new Thread(() -> {
            System.out.println("스레드 시작!");
            // 예기치 않은 예외 발생
            throw new RuntimeException("예기치 않은 예외가 발생했습니다.");
        });

        // 스레드의 UncaughtExceptionHandler 설정
        thread.setUncaughtExceptionHandler((t, e) -> {
            LOGGER.log(Level.SEVERE, t.getName() + " 에서 예외가 발생했습니다.", e);
            // 오류가 발생한 경우 알림 서비스 호출 (예: 이메일 또는 Slack 알림)
            sendNotificationToAdmin(e);
        });
        thread.start();
    }

    // 알림 서비스를 호출하는 메서드
    private static void sendNotificationToAdmin(Throwable e) {
        System.out.println("관리자에게 알림: " + e.getMessage());
    }
}
```

# 💡 예외처리 흐름

![image](https://github.com/shin-je-woo/TIL/assets/39439576/50a44fc0-7b55-47ae-9e1f-0d2a91f3ead7)
