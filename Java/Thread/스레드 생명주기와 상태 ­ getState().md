# Previous

- 자바 스레드는 생성과 실행 그리고 종료에 따른 상태를 가지고 있으며, JVM 에서는 6가지의 스레드 상태가 존재한다. OS 스레드 상태를 의미하지 않는다.
- 자바 스레드는 어떤 시점이든 6 가지 상태 중 오직 하나의 상태를 가질 수 있다.
- 자바 스레드의 현재 상태를 가져오려면 Thread 의 getState() 메서드를 사용하여 가져 올 수 있다.
- Thread 클래스에는 스레드 상태에 대한 ENUM 상수를 정의하는 Thread.State 클래스를 제공한다.

# 💡 스레드 상태

|상태|ENUM|설명|
|:------|:---|:---|
|객체 생성|NEW|스레드 객체가 생성됨, 아직 시작되지 않은 스레드 상태|
|실행 대기|RUNNABLE|실행 중이거나 실행 가능한 스레드 상태|
|일시 정지|WAITING|대기 중인 스레드 상태로서 다른 스레드가 특정 작업을 수행하기를 기다림|
|일시 정지|TIMED_WAITING|대기 시간이 지정된 스레드 상태로서 다른 스레드가 특정 작업을 수행하기를 기다림|
|일시 정지|BLOCKED|모니터 락(Lock)이 해제될 때 까지 기다리며 차단된 스레드 상태|
|종료|TERMINATED|실행이 완료된 스레드 상태|

# 💡 스레드 생명주기

![image](https://github.com/shin-je-woo/TIL/assets/39439576/4b56e46f-b36c-4564-beec-3a3062b6302e)

#### 참고) ­ Running 상태
- 스레드에는 없는 상태정보이나, 전체 생명주기 흐름을 이해하기 위해 실행 중인 상태로 표시한 것이며, Runnable 상태에 포함된 개념이라 볼 수 있다.

## NEW (객체 생성 상태)

- 스레드 객체는 생성 되었지만, 아직 start() 하지 않은 상태로서 JVM 에는 객체가 존재하지만 아직 커널로의 실행은 안된 상태

![image](https://github.com/shin-je-woo/TIL/assets/39439576/892559b9-38e2-4e26-b9ed-ce5838c859e0)

## Runnable (실행 대기 상태)

- start() 를 실행하면 내부적으로 커널로의 실행이 일어나고 커널 스레드로 1:1 매핑된다.
- 스레드는 바로 실행 상태가 아닌 언제든지 실행할 준비가 되어 있는 실행 가능한 상태가 된다.
- 스레드가 실행상태로 전환하기 위해서는 현재 스레드가 어떤 상태로 존재하든지 반드시 실행 대기 상태를 거쳐야 한다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/91e7d66c-34c6-4dba-8260-82e904ffe0d4)

## OS Scheduling

- 실행 가능한 상태의 스레드에게 실행할 시간을 제공하는 것은 OS 스케줄러의 책임이다.
- 스케줄러는 멀티 스레드 환경에서 각 스레드에게 고정된 시간을 할당해서 실행 상태와 실행 가능한 상태를 오가도록 스케줄링 한다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/82029c3a-498a-43f6-889b-880f4e7a7c33)

## Runnable (실행 상태) :이해를 돕기 위해 Running으로 표현

- 스레드는 스케줄러에 의해 스케줄링 되면 실행 상태로 전환되고, CPU 를 할당 받아 run() 메서드를 실행한다.
- 스레드는 아주 짧은 시간동안 실행된 다음 다른 스레드가 실행될 수 있도록 CPU를 일시 중지하고 다른 스레드에 양도하게 된다.
- 실행 상태에서 생성과 종료 상태를 제외한 다른 상태로 전환될 때 스레드 혹은 프로세스 간 컨텍스트 스위칭이 일어난다고 할 수 있다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/eda06b8b-bcb1-4847-aba0-4ec63715e34f)

## Running → Runnable

- 실행 상태에서 스레드의 yield() 메서드를 호출하거나, 운영체제 스케줄러에 의해 CPU 실행을 일시 중지하는 경우 실행 가능한 상태로 전환된다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/52b3e007-4cc6-4c4f-a0e5-0b4ddbab6fa2)

## Timed Waiting (일시 정지 상태, 지정된 시간이 있는 경우)

- 스레드는 sleep 및 time-out 매개 변수가 있는 메서드를 호출할 때 Timed Waiting 상태가 된다.
- 스레드의 대기 시간이 길어지고, CPU 할당을 받지 못하는 상황이 지속적으로 발생하면 기아 상태가 발생하게 되는데, 이 상황을 피할 수 있다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/9c0aa0f5-8873-4704-bfed-c2aaef50ed7b)

## Timed Waiting → Runnable

- 스레드가 대기상태의 지정 시간이 완료되거나, 다른 스레드에 의해 인터럽트가 발생하거나, 대기가 해제되도록 통지를 받게 되면 실행 대기 상태가 된다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/e66007b5-ee7b-4321-837d-8f3a4f2a9704)

## Running → Blocked

- 임계 영역 동시적 접근. 즉, 멀티 스레드 환경에서 각 스레드가 동기화 된 임계 영역에 접근을 시도할 때 스레드는 Blocked 상태가 된다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/0c82bc4d-e22d-45d4-8fe0-034c53671545)

## Blocked

- 스레드가 동기화 된 임계 영역에 접근을 시도하다가 Lock 을 획득하지 못해서 차단된 상태
- 스레드는 Lock 을 획득할 때까지 대기한다.
- 만약, Lock을 획득하면 다시 Runnable 상태가 된다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/e642b8f1-6133-4955-8fd7-fde13bc07daa)

## Waiting

- 스레드가 실행 상태에서 다른 스레드가 특정 작업을 수행하기를 기다리는 상태
- wait() 은 다른 스레드에 의해 notify()받을 때까지, join() 은 스레드의 실행이 종료되거나 인터럽트가 발생할 때까지 대기한다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/9297bdd7-c7a5-4b79-b57a-7daa3026bb2a)

## Waiting → Runnable

- Waiting 상태의 스레드가 다른 스레드에 의해 notify() 혹은 notifyAll() 이 일어나면 Runnable 상태가 된다.
- 다른 스레드에 의해 인터럽트가 발생할 경우 Runnable 상태로 전환된다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/189f166b-b75f-46ba-a67d-85a5dc7e5455)

## Terminated (실행 종료 상태)

- 실행이 완료되었거나 오류 또는 처리되지 않은 예외와 같이 비정상적으로 종료된 상태
- 종료된 스레드는 더 이상 사용할 수 없다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/145364ed-86f7-4c0a-8d00-624036434f73)

## 스레드 생명주기 정리

- 스레드의 실행 관점에서 보면 출발지가 스레드의 `start()` 메소드 실행이라면, 목적지는 스레드의 `run()` 메소드 실행이다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/fd25e764-91f5-480f-81f5-1fc17509f7e0)
