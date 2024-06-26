# Previous

- CAS(Compare and Swap, 비교 후 치환) 는 멀티 스레드 환경에서 스레드 간의 경쟁 조건을 방지하고 락을 사용하지 않고도 공유 변수의 값을 원자적으로 변경하는 방법을 제공한다.
- **CAS 는 CPU 캐시와 메인메모리의 두 값을 비교하고 그 값이 동일할 경우 새로운 값으로 교체하는 동기화 연산**으로 여러 스레드가 공유하는 메모리 영역을 보호하는 데 사용된다.
- CAS 는 락 기반의 동기화보다 경량화되어 있으며 락을 사용하지 않기 때문에 대기하지 않는 넌블록킹 실행이 가능하고 경쟁 조건과 데드락을 피할 수 있다.
- CAS는 조건에 따라 실패하고 다시 시도해야 할 수 있기 때문에 동시적으로 접근하는 요청의 수가 많은 경쟁 조건일 경우 효율성이 저하될 수 있다.
- CAS는 주로 하드웨어 수준(CPU) 에서 지원되는 연산이며 Java에서는 java.util.concurrent.atomic 패키지에 있는 원자적 연산을 통해 CAS를 지원하고 있다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/ca154d02-ec39-4352-ac4c-7f1f1c0de9e3)

# 💡 CAS 동작 방식

![image](https://github.com/shin-je-woo/TIL/assets/39439576/3d1d99a8-55b7-4855-823c-60382fa570ec)

# 💡 CAS 기본 구현

```java
import java.util.concurrent.atomic.AtomicInteger;

public class MultiThreadCASExample {
    private static AtomicInteger value = new AtomicInteger(0);
    private static final int NUM_THREADS = 3;
    public static void main(String[] args) {
        Thread[] threads = new Thread[NUM_THREADS];

        for (int i = 0; i < NUM_THREADS; i++) {
            threads[i] = new Thread(() -> {
                for (int j = 0; j < 5; j++) {
                    int expectedValue, newValue;
                    do {
                        expectedValue = value.get();
                        newValue = expectedValue + 1;
                    } while (!value.compareAndSet(expectedValue, newValue)); // 반환 값이 false 이면 true 가 반환 될 때까지 재시도
                    System.out.println(Thread.currentThread().getName() + " : " + expectedValue + " , " + newValue);
                }
            });
            threads[i].start();
        }
        for (Thread thread : threads) {
            try {
                thread.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println("Final value: " + value.get());
    }
}
```

1) 3개의 스레드가 각각 5번씩 CAS 연산을 시도한다.
2) compareAndSet 메서드를 사용해서 기대값(CPU캐시에 저장된 값)과 메인메모리에 저장된 값을 비교하여 일치할 때까지 시도한다.
3) CAS 연산이 성공할 때마다 각 스레드는 결과를 출력하고, 모든 스레드가 작업을 마친 후 최종 값을 출력한다.
4) 실행 결과에서는 CAS 연산이 동시에 여러 스레드에서 수행되더라도 적절히 값을 증가시키는 것을 확인할 수 있다.

### 실행 결과

![image](https://github.com/shin-je-woo/TIL/assets/39439576/e5a95658-7ff5-4c11-ada8-c36ad0794d3e)

