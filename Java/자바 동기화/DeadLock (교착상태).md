# Previous

- DeadLock (교착상태) 이란 프로세스나 스레드들이 서로가 소유하고 있는 자원을 기다리며 무한히 대기하고 있는 상태를 말한다.
- 교착상태에서는 아무런 진전도 이루어지지 않아 작업이 진행되지 않는 문제가 발생한다.
- DeadLock 은 동일한 환경과 코드에서 발생할 수도 있고 발생하지 않을 수도 있다.

# 💡 데드락 발생 조건

데드락은 다음의 네 가지 조건을 동시에 만족할 때 발생한다. 한 가지라도 만족하지 않으면 데드락이 발생하지 않는다.

1. **상호 배제(Mutual Exclusion)**
    - 자원은 한 번에 하나의 스레드만 사용할 수 있다.
2. **점유 대기(Hold and Wait)**
    - 스레드가 최소한 하나의 자원을 보유한 상태에서 다른 자원을 기다리고 있다.
3. **비선점(No Preemption)**
    - 자원을 할당 받은 스레드가 자원을 스스로 반납하기 전까지 자원을 강제로 빼앗을 수 없다.
4. **순환 대기(Circular Wait)**
    - 각 스레드는 순환적으로 다음 스레드가 요구하는 자원을 가지고 있어 사이클이 형성된다.
  
# 💡 데드락 사례

### 락 순서에 의한 데드락

```java
private static final Object lock1 = new Object();
private static final Object lock2 = new Object();

public static void main(String[] args) {
    Thread thread1 = new Thread(() -> {
        process1();
    });
    Thread thread2 = new Thread(() -> {
        process2();
    });
    thread1.start();
    thread2.start();
}

private static void process1() throws InterruptedException {
    synchronized (lock1) { // (1) thread1 이 lock1 을 획득함(Hold)
        System.out.println("thread 1 acquired lock1");
        synchronized (lock2) { // (3) thread1 이 lock2 가 해제되기를 기다림 (Wait)
            System.out.println("thread 1 acquired lock2");
        }
    }
}

private static void process2() throws InterruptedException {
    synchronized (lock2) { // (2) thread2 가 lock2 를 획득함 (Hold)
        System.out.println("thread 2 acquired lock2");
        synchronized (lock1) { // (4) thread2 가 lock1 이 해제되기를 기다림 (Wait)
            System.out.println("thread 2 acquired lock1");
        }
    }
}
```

### 동적인 락 순서에 의한 데드락

```java
public class MethodParameterDeadlock {
    private static final Object lock1 = new Object();
    private static final Object lock2 = new Object();

    public static void main(String[] args) {
        Thread thread1 = new Thread(() -> {
            methodWithLocks(lock1, lock2);
        });
        Thread thread2 = new Thread(() -> {
            methodWithLocks(lock2, lock1);
        });
        thread1.start();
        thread2.start();
    }

    private static void methodWithLocks(Object lockA, Object lockB) {
        synchronized (lockA) { // thread1 은 lock1, thread2 는 lock2 를 획득함
            System.out.println("메소드: lockA 획득");
            synchronized (lockB) { // thread1 은 lock2, thread2 는 lock1 의 해제를 기다림
                System.out.println("메소드: lockB 획득");
            }
        }
    }
}
```

### 객체 간의 데드락

```java
public class DeadlockExample {
    public static void main(String[] args) {
        ResourceA resourceA = new ResourceA();
        ResourceB resourceB = new ResourceB();
        Thread thread1 = new Thread(() -> {
            resourceA.methodA(resourceB);
        });
        Thread thread2 = new Thread(() -> {
            resourceB.methodB(resourceA);
        });
        thread1.start();
        thread2.start();
        // thread1 은 resourceA, thread2 는 resourceB 의 모니터를 소유함
        // thread1 은 resourceB, thread2 는 resourceA 의 모니터 해제를 기다림
    }
}

public class ResourceA {
    public synchronized void methodA(ResourceB resourceB) {
        System.out.println(Thread.currentThread().getName() + ": methodA 실행");
        resourceB.methodB2();
    }

    public synchronized void methodA2() {
        System.out.println(Thread.currentThread().getName() + ": methodA2 실행");
    }
}

public class ResourceB {
    public synchronized void methodB(ResourceA resourceA) {
        System.out.println(Thread.currentThread().getName() + ": methodB 실행");
        resourceA.methodA2();
    }

    public synchronized void methodB2() {
        System.out.println(Thread.currentThread().getName() + ": methodB2 실행");
    }
}
```

# 💡 데드락 방지와 원인 추적

1. **한번에 하나 이상의 락을 사용하지 않는다.**
    - 데드락은 스레드가 락을 중첩으로 제어하면서 발생하는 경우가 많기 때문에 가능하면 스레드가 두 개 이상의 락을 제어하는 상황을 만들지 않도록 하는 것이 좋다.
2. **락의 순서를 잘 조정한다.**
    - 불가피하게 여러 개의 락을 사용해야 한다면 락의 점유 순서를 일정한 순서로 정해주도록 함으로써 데드락이 발생할 수 있는 조건 중 하나인 순환 대기를 방지하도록 한다.
3. **락 타임아웃을 건다.**
    - 락을 요청할 때 일정 시간 이내에 락을 얻지 못하면 다른 작업을 수행하도록 타임아웃을 설정한다.
    - 락 획득에 타임아웃 오류가 나면 오래 기다리지 않고 제어권이 다시 돌아오기 때문에 현재 소유한 락을 해제하고 잠시 기다리다가 데드락 상황이 지나가면 다시 정상으로 동작할 수 있다.
4. **메서드는 오픈 호출 형태로 구현한다.**
    - 락을 전혀 확보하지 않은 상태에서 메서드를 호출하는 것을 오픈 호출이라고 하며 락을 전체 메서드에 적용하지 않고 락이 필요한 임계영역만 보호하도록 한다.
    - 여러 개의 락을 호출하더라도 동시에 락을 점유하는 것이 아닌 순차적으로 락을 획득하고 해제하는 방식으로 메서드를 호출하도록 한다.
5. **스레드 덤프를 활용한다.**
    - 스레드 덤프에는 실행중인 스레드의 모든 스택 트레이스가 담겨져 있고 락과 관련된 정보도 포함되어 있기 때문에 이를 활용해서 어디에서, 어떤 스레드가, 어느 시점에, 왜 데드락이 발생했는지 원인을 추척 및 분석하고 해결방안을 모색할 수 있다.
  
# 💡 데드락 방지

### 락 순서를 조절한다.

```java
private static final Object lock1 = new Object();
private static final Object lock2 = new Object();

public static void main(String[] args) {
    Thread thread1 = new Thread(() -> {
        process1();
    });
    Thread thread2 = new Thread(() -> {
        process2();
    });
    thread1.start();
    thread2.start();
}

private static void process1() throws InterruptedException {
    synchronized (lock1) { // (1) thread1 이 lock1 을 획득함
        System.out.println("thread 1 acquired lock1");
        synchronized (lock2) { (2) thread1 가 lock2 를 획득함
            System.out.println("thread 1 acquired lock2");
        }
    }
}

private static void process2() throws InterruptedException {
    synchronized (lock1) { // (3) thread2 가 lock1 해제를 기다린 후 획득함
        System.out.println("thread 2 acquired lock1");
        synchronized (lock2) { // (4) thread2 가 lock2 를 획득함
            System.out.println("thread 2 acquired lock2");
        }
    }
}
```

### 락에 타임아웃을 건다.

스레드가 동시에 lock1 과 lock2 을 점유한 상태에서 순환대기가 일어나더라도 타임아웃이 발생하면 락을 기다리지 않고 false 를 리턴하기 때문에 데드락이 발생하지 않는다.

```java
public class NestedLockTimeoutExample {
    private static final Lock lock1 = new ReentrantLock();
    private static final Lock lock2 = new ReentrantLock();

    public static void main(String[] args) {
        Thread thread1 = new Thread(() -> {
            try {
                if (lock1.tryLock(2, TimeUnit.Second)) {
                    try {
                        System.out.println("스레드1: lock1 획득");
                        if (lock2.tryLock(2, TimeUnit.Second)) {
                            try {
                                System.out.println("스레드1: lock2 획득");
                            } finally {
                                lock2.unlock();
                                System.out.println("스레드1: lock2 해제");
                            }
                        } else {
                            System.out.println("스레드1: lock2 획득 실패");
                        }
                    } finally {
                        lock1.unlock();
                        System.out.println("스레드1: lock1 해제");
                    }
                } else {
                    System.out.println("스레드1: lock1 획득 실패");
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        Thread thread2 = new Thread(() -> {
            try {
                if (lock2.tryLock(2, TimeUnit.Second)) {
                    try {
                        System.out.println("스레드2: lock2 획득");
                        if (lock1.tryLock(2, TimeUnit.Second)) {
                            try {
                                System.out.println("스레드2: lock1 획득");
                            } finally {
                                lock1.unlock();
                                System.out.println("스레드2: lock1 해제");
                            }
                        } else {
                            System.out.println("스레드2: lock1 획득 실패");
                        }
                    } finally {
                        lock2.unlock();
                        System.out.println("스레드2: lock2 해제");
                    }
                } else {
                    System.out.println("스레드2: lock2 획득 실패");
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        thread1.start();
        thread2.start();
    }
}
```

### 메서드 오픈 호출

```java
public class DeadlockExample {
    public static void main(String[] args) {
        ResourceA resourceA = new ResourceA();
        ResourceB resourceB = new ResourceB();
        Thread thread1 = new Thread(() -> {
            resourceA.methodA(resourceB);
        });
        Thread thread2 = new Thread(() -> {
            resourceB.methodB(resourceA);
        });
        thread1.start();
        thread2.start();
        // thread1 은 resourceA, thread2 는 resourceB 의 모니터를 소유함
        // thread1 은 resourceA 모니터를 해제하고 resourceB 모니터를 소유
        // thread2 는 resourceB 모니터를 해제하고 resourceA 모니터를 소유
    }
}

public class ResourceA {
    public void methodA(ResourceB resourceB) {
        synchronized (this) {
            System.out.println(Thread.currentThread().getName() + ": methodA 실행");
        }
        resourceB.methodB2();
    }

    public synchronized void methodA2() {
        System.out.println(Thread.currentThread().getName() + ": methodA2 실행");
    }
}

public class ResourceB {
    public void methodB(ResourceA resourceA) {
        synchronized (this) {
            System.out.println(Thread.currentThread().getName() + ": methodB 실행");
        }
        resourceA.methodA2();
    }

    public synchronized void methodB2() {
        System.out.println(Thread.currentThread().getName() + ": methodB2 실행");
    }
}
```

# 💡 그 외 활동성 문제

## 기아상태(Starvation)

- 멀티스레드 환경에서 한 스레드가 자원을 계속해서 얻지 못하여 영원히 대기하는 상태를 말한다.
- 다른 스레드들이 우선적으로 자원을 점유하거나 실행되어 해당 스레드가 자원을 얻지 못하게 되면 기아상태가 발생한다.
- 기아 상태를 해결하기 위해서는 우선순위를 기반으로 한 스케줄링이 아니라 공정성을 고려하여 모든 스레드가 공평하게 실행될 수 있도록 우선순위를 조정하는 등의 설계가 뒷받침 되어야 한다.

## 라이브락(Livelock)

- 라이브락은 데드락과 비슷한 상태로, 여러 스레드가 서로의 자원을 기다리면서 진행이 지연되는 상태를 말한다.
- 라이브락은 데드락과는 다르게 작업이 멈추지 않고 계속해서 진행되지만, 실질적인 작업이 진행되지 않거나 진행이 제대로 이루어지지 않는 상태를 의미한다.
- 라이브락 상태에서는 스레드가 서로의 자원을 기다리기 때문에 작업이 충분히 느려질 수 있으며, 서로가 양보하는 동작이 반복되면서 진행이 지연되는 상태가 발생한다.
