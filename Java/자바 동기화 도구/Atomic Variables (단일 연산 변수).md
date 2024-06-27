# Previous

- 단일연산변수는 락을 사용하지 않고도 여러 스레드 간에 안전하게 값을 공유하고 동기화하는 데 사용되며 기본적으로 volatile 의 속성을 가지고 있다.
- 단일연산변수는 원자적인(read-modify-write) 연산을 지원하여 내부적으로 Compare and Swap (CAS) 연산을 사용하여 데이터의 일관성과 안정성을 유지한다.
- 단일연산변수는 간단한 연산의 경우 락을 사용하는 것보다 월등히 빠른 성능을 보여 주지만 연산이 복잡하거나 시간이 오래 걸리는 작업은 락을 사용하는 것보다 오버헤드가 커질 수 있다.
- 단일연산변수는 단일 연산에 대해 원자성을 보장하지만 여러 연산을 조합한 복잡한 동작에 대해서는 원자성이 보장되지 않을 수 있으며 강력한 동기화 메커니즘을 고려해야 한다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/4a94edd4-d47e-4729-9222-c25866e62418)

# 💡 단일 연산 클래스(Atomic Class)

단일연산 변수를 사용하기위한 여러종류의 단일연산 클래스가 제공된다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/4bd17e65-6501-4bfb-a5af-1bc3834f94eb)

# 💡 Atomic의 CAS 구현

AtomicInteger의 incrementAndGet 메서드를 보면 내부적으로 CAS 알고리즘을 구현하고 있는 것을 알 수 있다.

```java
public class AtomicInteger extends Number implements java.io.Serializable {

    private static final Unsafe U = Unsafe.getUnsafe();
    private static final long VALUE = U.objectFieldOffset(AtomicInteger.class, "value");
    private volatile int value;
    
    public final int incrementAndGet() {
            return U.getAndAddInt(this, VALUE, 1) + 1;
    }
}

public final class Unsafe {

    @HotSpotIntrinsicCandidate
    public final int getAndAddInt(Object o, long offset, int delta) {
            int v;
            do {
                    v = getIntVolatile(o, offset);
            } while (!weakCompareAndSetInt(o, offset, v, v + delta));
                    return v;
    }
}
```

AtomicInteger는 weakCompareAndSetInt 의 return 값이 true 로 반환할 될 때까지 무한 루프를 돌면서 기다린다. 

그러면 여기서 의문이 하나 생긴다. 의미없이 무한 루프를 돌면서 CPU를 붙잡고 있는데 synchronized 보다 나을까 하는 의문이다.

synchronized 는 스레드가 suspending과 resuming 하는데에 자원 소모가 발생하고 (context switch 비용), 이 과정은 lock이 걸린 모든 스레드가 공통적으로 겪는 과정이다. 

그렇기 때문에 비록 무한 루프를 돌지만 true를 return 받는 즉시 다음 작업을 수행할 수 있는 Atomic이 성능적으로 우수하다. (연산의 수행시간이 context switch 하는 시간보다 짧을 경우에)

여기서 눈에 띄는 점은 int value에 붙어있는 `volatile` 이라는 키워드다.

> 📌 volatile
> 
> volatile 은 배타적 수행과는 상관없지만 항상 최근에 기록된 값을 읽는 것을 보장해준다.   
> 메인 메모리에 있는 값을 직접 읽어서, 캐시 메모리를 사용하는 구조상 발생할 수 있는 가시성의 문제를 해결해줄 수 있다.

### Atomic의 volatile 한정자

```java
class AtomicInteger {

    private volatile int value;
    
    public AtomicInteger(int initialValue) {
        value = initialValue;
    }
    
    public AtomicInteger() {
    }
    
    public final int get() {
        return value;
    }
    
    public final void set(int newValue) {
        value = newValue;
    }
    
    ...
}
```

get()과 set()은 그 자체로 원자 연산이다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/780d4203-5ff6-42b6-942a-1c51c4cde6ed)

get()과 set()은 그 자체로 atomic 연산이기 때문에 race condition이 발생하지 않는다. 따라서 volatile 한정자를 사용해서 가시성에 대한 문제만을 해결해준것이다.

다음 두개의 코드를 확인해보자.

```java
public class AtomicIntegerExample {

    private static AtomicInteger counter = new AtomicInteger(0);
    private static int NUM_THREADS = 5;
    private static int NUM_INCREMENTS = 1_000_000;

    public static void main(String[] args) {

        Thread[] threads = new Thread[NUM_THREADS];

        for (int i = 0; i < NUM_THREADS; i++) {
            threads[i] = new Thread(()->{
                for (int j = 0; j < NUM_INCREMENTS; j++) {
                    counter.set(counter.get() + 1); // 여기서 문제 발생
                    System.out.println("Counter: " + counter.get());
                }
            });
            threads[i].start();
        }
        for(Thread thread:threads){
            try {
                thread.join();
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }
        System.out.println("Final value: " + counter.get());
    }
}
```

위 코드를 보면 `counter.set(counter.get() + 1);` 에서 문제가 발생하는데, Atomic의 get과 set은 원자적 연산을 보장하지만, 읽고 쓰는 모든 연산에 대해 원자성을 보장하지는 않는다.

쓰기작업 중간에 다른 쓰레드가 동시에 쓰는 race condition이 발생할 수 있다.

위 코드는 아래와 같이 `counter.incrementAndGet();` 으로 일련의 연산에 대한 원자성을 보장해야 race condition이 발생하지 않는다.

```java
public class AtomicIntegerExample {

    private static AtomicInteger counter = new AtomicInteger(0);
    private static int NUM_THREADS = 5;
    private static int NUM_INCREMENTS = 1000000;

    public static void main(String[] args) {

        Thread[] threads = new Thread[NUM_THREADS];

        for (int i = 0; i <NUM_THREADS; i++) {
            threads[i] = new Thread(()->{
                for (int j = 0; j < NUM_INCREMENTS; j++) {
                    counter.incrementAndGet(); // 원자성 보장
                    System.out.println("Counter: " + counter.get());
                }
            });
            threads[i].start();
        }
        for(Thread thread:threads){
            try {
                thread.join();
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }
        System.out.println("Final value: " + counter.get());
    }
}
```

# 💡 AtomicBoolean

### AtomicBoolean()

- 초기 값이 false 로 설정된다.

### AtomicBoolean(boolean initialValue)

- 초기 값이 지정된 값으로 설정된다.

### boolean get()

- 현재 값을 가져온다.

### void set(boolean newValue)

- 새로운 값으로 설정한다.

### boolean getAndSet(boolean newValue)

- 현재 값을 가져오고 새로운 값을 설정한다.

### boolean compareAndSet(boolean expect, boolean update)

- 현재 값이 기대한 값과 같으면 새로운 값을 설정하고 true 를 반환하고 일치하지 않으면 아무런 동작 없이 false 를 반환한다.

# 💡 AtomicInteger

### AtomicInteger()

- 초기 값이 0 으로 설정된다.

### AtomicInteger(int initialValue)

- 초기 값이 지정된 값으로 설정된다.

### int getAndIncrement()

- 현재 값을 반환하고 +1을 증가시킨다.

### int incrementAndGet()

- 현재 값에 +1을 증가하고 결과값을 반환한다.

### int getAndDecrement()

- 현재 값을 반환하고 -1을 감소시킨다.

### int getAndAdd(int delta)

- 현재 값을 반환하고 지정된 값 만큼 더한다.

### int addAndGet(int delta)

- 현재 값에 지정된 값 만큼 더하고 결과 값을 반환한다.

### int decrementAndGet()

- 현재 값에 지정된 값 만큼 감소시키고 결과 값을 반환한다.

### int getAndUpdate(IntUnaryOperator updateFunction)

- 현재 값을 반환하고 람다 함수로 실행된 값으로 업데이트 한다.

### int updateAndGet(IntUnaryOperator updateFunction)

- 람다 함수로 실행된 값으로 업데이트하고 결과값을 반환한다.

# 💡 AtomicReference<V>

### AtomicReference()

- 초기 값이 null 로 설정된다.

### AtomicReference(V initialValue)

- 초기 값이 지정된 값으로 설정된다.

### V get()

- 현재 값을 가져온다.

### void set(V newValue)

- 새로운 값으로 설정한다.

### V getAndSet(V newValue)

- 현재 값을 가져오고 새로운 값을 설정한다.

### boolean compareAndSet(V expect, V update)

- 현재 값이 기대한 값과 같으면 새로운 값을 설정하고 true 를 반환하고 일치하지 않으면 아무런 동작없이 false 를 반환한다.

### V getAndUpdate(UnaryOperator<V> updateFunction)

- 현재 값을 반환하고 람다 함수로 실행된 값으로 업데이트 한다.

### V updateAndGet(UnaryOperator <V> updateFunction)

- 람다 함수로 실행된 값으로 업데이트하고 결과값을 반환한다.
