# Previous

- 자바 스레드는 OS 스케줄러에 의해 실행 순서가 결정되며, 스레드 실행 시점을 JVM 에서 제어할 수 없다.
- 새로운 스레드는 현재 스레드와 독립적으로 실행되고, 최대 한번 시작할 수 있으며, 스레드가 종료된 후에는 다시 시작 할 수 없다.

# 💡 스레드 실행

## start()

- 스레드를 실행시키는 메서드로 시스템 콜을 통해서 커널에 커널 스레드 생성을 요청한다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/6fdf7ddd-f982-4060-aa43-00b20e057a07)

## run()

- 스레드가 실행이 되면 해당 스레드에 의해 자동으로 호출되는 메서드이다.
- Thread 의 run() 이 자동 호출되고 여기서 Runnable 구현체가 존재할 경우 Runnable 의 run() 을 실행하게 된다.
- public static void main(String[] args) 메서드가 메인 스레드에 의해 자동으로 호출되는 것과 비슷한 원리이다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/5a1d135f-0e4a-43fe-a8b8-670c1d89e0ff)

- ❗ 주의 할 것은 만약 start() 가 아닌 run() 메서드를 직접 호출하면 새로운 스레드가 생성되지 않고 직접 호출한 스레드의 실행 스택에서 run() 이 실행될 뿐이다 .

![image](https://github.com/shin-je-woo/TIL/assets/39439576/1d2683f9-b085-4e4f-bc61-b241a1b377da)
