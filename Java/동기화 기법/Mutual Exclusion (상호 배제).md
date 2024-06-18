# Previous

- 뮤텍스(Mutual Exclusion) 또는 상호 배제는 공유 자원에 대한 경쟁 상태를 방지하고 동시성 제어를 위한 락 메커니즘이다.
- 스레드가 임계영역에서 Mutex 객체의 플래그를 소유하고 있으면(락 획득) 다른 스레드가 액세스할 수 없으며 해당 임계영역에 액세스하려고 시도하는 모든 스레드는 차단되고 Mutex 객체 플래그가 해제된 경우(락 해제)에만 액세스할 수 있다.
- 이 메커니즘은 Mutex 락을 가진 오직 한개의 스레드만이 임계영역에 진입할 수 있으며 락을 획득한 스레드만이 락을 해제 할 수 있다.

# 💡 Mutex

![image](https://github.com/shin-je-woo/TIL/assets/39439576/558997f4-cf3f-457c-9edd-98006ad1d417)

```java
public class Mutex {

    private boolean lock = false;

    public synchronized void acquired() {
        while (lock) {
            try {
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        this.lock = true;
    }

    public synchronized void release() {
        this.lock = false;
        notify();
    }
}
```

```java
public class SharedData {

    private int sharedValue = 0;
    private final Mutex mutex;

    public SharedData(Mutex mutex) {
        this.mutex = mutex;
    }

    public void sum() {
        try {
            mutex.acquired(); // Lock 을 획득
            for (int i = 0; i < 10_000_0000; i++) {
                sharedValue++; // critical section. 공유데이터 읽기/쓰기 작업
            }
        } finally {
            mutex.release(); // Lock 해제
        }
    }
    public int getSum() {
        return sharedValue;
    }
}
```

# 💡 뮤텍스 문제점

- **데드락(Deadlock)**
  - 데드락은 두 개 이상의 스레드가 서로가 가진 락을 기다리면서 상호적으로 블로킹되어 아무 작업도 수행할 수 없는 상태를 의미하며 잘못된 뮤텍스 사용으로 인해 데드락이 발생할 수 있다.
- **우선 순위 역전(Priority Inversion)**
  - 우선 순위 역전은 높은 우선 순위를 가진 스레드가 낮은 우선 순위를 가진 스레드가 보유한 락을 기다리는 동안 블록되는 현상으로 높은 우선 순위를 가진 스레드의 작업이 지연될 수 있다. 우선 순위 상속으로 해결할 수 있다
- **오버헤드**
  - 뮤텍스를 사용하면 여러 스레드가 경합하면서 락을 얻기 위해 스레드 스케줄링이 발생한다. 이로 인해 오버헤드가 발생하고 성능이 저하될 수 있다.
- **성능 저하**
  - 뮤텍스를 사용하면 락을 얻기 위해 스레드가 대기하게 되고, 스레드의 실행 시간이 블록되면서 성능 저하가 발생할 수 있다.
- **잘못된 사용**
  - 뮤텍스를 적절하게 사용하지 않거나 잘못된 순서로 락을 해제하는 경우 예기치 않은 동작이 발생할 수 있다.
