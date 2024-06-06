# Previous

- 자바 스레드는 JVM 에서 User Thread 를 생성할 때 시스템 콜을 통해서 커널에서 생성된 Kernel Thread 와 1:1 로 매핑이 되어 최종적으로 커널에서 관리된다.
- JVM 에서 스레드를 생성할 때 마다 커널에서 자바 스레드와 대응하는 커널 스레드를 생성한다.
- 자바에서는 Platform Thread 로 정의되어 있다. 즉 OS 플랫폼에 따라 JVM 이 사용자 스레드를 매핑하게 된다.

#### 📌 Platform threads

> Thread supports the creation of platform threads that are typically mapped 1:1 to kernel threads scheduled by the operating system

![image](https://github.com/shin-je-woo/TIL/assets/39439576/b7d081c9-6af9-431a-9893-8adb723df5a2)

# 💡 자바 스레드 구조

![image](https://github.com/shin-je-woo/TIL/assets/39439576/dd42b8ff-81ee-45ba-a22c-d6c4a645b3bf)

# 💡 스레드 생성

## Thread 클래스 상속하는 방법

```java
public class ExtendThreadExample {
    public static void main(String[] args) {
        MyThread myThread = new MyThread();
        myThread.start();
    }
}

class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + " :스레드 실행 중.. ");
    }
}
```

- 작업내용을 스레드 내부에 직접 재정의해서 실행한다.

1) Thread 클래스 상속
2) run() 메서드 오버라이드
3) start() 메서드 실행

## Runnable 인터페이스를 구현하는 방법

```java
public class ImplementRunnableExample {
    public static void main(String[] args) {
        MyRunnable task = new MyRunnable();
        Thread thread = new Thread(task);
        thread.start();
    }
}

class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + ": 스레드 실행 중");
    }
}
```

- 작업내용을 Runnable 에 정의해서 스레드에 전달하면 스레드는 Runnable 을 실행한다.

1) Runnable 인터페이스 구현
2) run() 메서드 오버라이드
3) Thread 생성 시 생성자에 전달
4) start() 메서드 실행

# 💡 다양한 스레드 생성 패턴

## Thread 를 상속한 클래스

```java
Thread thread = new WokerThread();
thread.start();
```

- 가장 기본적인 방식이며 Thread 클래스를 반드시 상속받아야 한다.
- 상속의 특성상 컴파일 타임 시점에 실행코드가 결정되어 동적인 기능 변경이 불가능하다는 단점이 있다.

## Thread 익명 클래스

```java
Thread thread = new Thread() {
    @Override
    public void run() {
        // 작업 내용
    }
};

thread.start();
```

- 스레드 객체를 참조하거나 재활용하지 않고 일회용으로만 사용할 경우 사용

## Runnable 구현

```java
Runnable task = new ExecuteTask();
Thread thread = new Thread(task);
thread.start();
```

- Runnable 을 태스크로 활용하는 방식으로서 선호하는 방식이다.
- 스레드와 실행하고자 하는 태스크를 분리함으로써 좀 더 유연하고 확장 가능한 구조로 구현이 가능하다.

## Runnable 익명 클래스

```java
Thread thread = new Thread(new Runnable() {
    @Override
    public void run() {
        // 작업 내용
    }
});

thread.start();
```

- Runnable 타입을 참조하거나 재활용하지 않고 일회용으로만 사용할 경우 사용

## Runnable 람다 방식

```java
Thread thread = new Thread(() -> // 작업 내용));
thread.start();
```

- Runnable 을 람다 형식으로 구현함으로써 코드가 간결해진다.
