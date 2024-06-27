# 💡 CyclicBarrier 란?

- CyclicBarrier 는 공통된 장벽 지점에 도달할 때까지 일련의 스레드가 서로 기다리도록 하는 동기화 보조 도구이다.
- CyclicBarrier 는 대기 중인 스레드가 해제된 후에 재 사용할 수 있기 때문에 순환 장벽이라고 부른다.
- CyclicBarrier는 옵션으로 Runnable 명령을 지원하는데 이 명령은 마지막 스레드가 도착한 후에 각 장벽 지점마다 한 번씩 실행되는 장벽액션(barrierAction) 역할을 수행한다.
- 이 Runnable 은 스레드가 장벽 이후 실행을 계속하기 전에 공유 상태를 업데이트하는 데 유용하다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/13f96ba1-b1c6-41a4-bbb7-d38704dde224)

### 사용용도

- 여러 스레드가 병렬로 작업을 수행하다가 특정 단계에 도달하거나 모든 스레드가 특정 작업을 완료하고 모이는 지점에서 사용된다.
  - 예를 들어 병렬 계산 작업 중 중간 결과를 모두 계산한 후에 다음 단계로 진행하기 위해 스레드들이 모이는 경우에 유용하다.
- 고정된 수의 스레드가 동시에 특정 작업을 수행하고 모든 스레드가 작업을 완료하고 모이는 시점에서 다음 단계를 진행할 때 사용된다.
- 즉, 여러 스레드가 협력하여 작업을 나누고 동기화하는 데에 적합하다.

# 💡 CyclicBarrier API

### CyclicBarrier(int parties, Runnable barrierAction)

- 주어진 수의 파티(스레드)가 CyclicBarrier 에서 await()을 호출하는 경우에 작동하며 CyclicBarrier 가 작동될 때 마지막으로 CyclicBarrier 에 진입한 스레드에 의해 수행되는 장벽 액션을 실행할 새로운 CyclicBarrier를 생성한다.

### int await() throws InterruptedException, BrokenBarrierException

- 이 장벽에서 모든 파티가 await 를 호출할 때까지 기다리며 현재 스레드가 마지막으로 도착한 것이 아닌 경우 다음 중 하나가 발생할 때까지 대기 상태에 있다.
  - 마지막 스레드가 도착함
  - 다른 스레드가 현재 스레드를 인터럽트 함
  - 다른 스레드가 다른 대기 중인 스레드 중 하나를 인터럽트 함
  - 다른 스레드가 장벽 대기 중에 타임아웃 됨
  - 다른 스레드가 이 장벽에 대해 reset 을 호출함
- 메서드 진입 시 인터럽트 상태가 설정되거나 또는 대기 중인 동안 인터럽트 되면 InterruptedException이 발생하며 현재 스레드의 인터럽트 상태는 초기화된다.
- 어떤 스레드가 대기 중일 때 장벽이 리셋되거나 장벽이 끊어진 경우 또는 await가 호출될 때 장벽이 깨진 경우에는 BrokenBarrierException이 발생한다.
- 어떤 스레드가 대기 중일 때 인터럽트가 발생하면 다른 대기 중인 스레드도 모두 BrokenBarrierException 을 던지고 장벽이 깨진 상태로 전환된다.
- 만약 현재 스레드가 마지막으로 도착한 스레드이고 장벽 액션이 제공된 경우 현재 스레드가 해당 액션을 실행한다.
- 장벽 액션 중에 예외가 발생하면 해당 예외가 현재 스레드에서 전파되고 장벽은 깨진 상태로 전환된다.

### public boolean await(long timeout, TimeUnit unit) throws InterruptedException

- await() 메서드와 대부분 동일하며 timeout 설정과 관련된 부분만 차이가 난다.
- 이 장벽에서 모든 파티가 await 를 호출할 때까지 기다리며 현재 스레드가 마지막으로 도착한 것이 아닌 경우 다음 중 하나가 발생할 때까지 대기 상태에 있다.
  - 지정된 타임아웃이 경과함
- 지정된 대기 시간이 경과하면 TimeoutException이 발생하며 시간이 0 이하이면 메서드는 전혀 대기하지 않는다.

### int getParties()

- 이 장벽을 작동시키기 위해 필요한 파티의 수(스레드 수)를 반환한다.

### boolean isBroken()

- 현재 장벽이 깨진 상태인지 확인한다.
- 만약 생성 이후나 마지막 재 설정 이후로 인터럽트나 타임아웃으로 인해 하나 이상의 스레드가 이 장벽에서 벗어났거나 예외로 인해 장벽 액션이 실패했다면 true 를 반환하고, 그렇지 않으면 false를 반환한다.

### void reset()

- 장벽을 초기 상태로 재 설정한다.
- 현재 장벽에서 대기 중인 스레드가 있는 경우 이들은 BrokenBarrierException을 반환한다.

# 💡 CyclicBarrier 기본 구현

ex) 스레드가 병렬 계산 작업 중 중간 결과를 모두 계산한 후에 다음 단계인 장벽 액션에서 중간 결과를 최종 합산하기 전에 스레드들을 모이게 한다.

```java
public class CyclicBarrierExample {
    static int[] parallelSum = new int[2];

    public static void main(String[] args) {
        int[] numbers = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
        int numThreads = 2; // 장벽을 구성하는 스레드 수
        CyclicBarrier barrier = new CyclicBarrier(numThreads, new BarrierAction(parallelSum));

        for (int i = 0; i < numThreads; i++) {
            new Thread(new Worker(i, numbers, barrier, parallelSum)).start();
        }
    }
}

class BarrierAction implements Runnable {
    private final int[] parallelSum;

    public BarrierAction(int[] parallelSum) {
        this.parallelSum = parallelSum;
    }

    @Override
    public void run() { // 모든 스레드가 장벽에 모인 시점에 실행된다.
        int finalSum = 0;
        for (int sum : parallelSum) {
            finalSum += sum;
        }
        System.out.println("Final Sum: " + finalSum);
    }
}

class Worker implements Runnable {
    private final int id;
    private final int[] numbers;
    private final CyclicBarrier barrier;
    private final int[] parallelSum;

    public Worker(int id, int[] numbers, CyclicBarrier barrier, int[] parallelSum) {
        this.id = id;
        this.numbers = numbers;
        this.barrier = barrier;
        this.parallelSum = parallelSum;
    }

    public void run() {
        int start = id * (numbers.length / 2);
        int end = (id + 1) * (numbers.length / 2);
        int sum = 0;
        for (int i = start; i < end; i++) {
            sum += numbers[i];
        }
        parallelSum[id] = sum; //각 스레드는 병렬로 데이터를 나누어 연산한다.

        try {
            barrier.await(); // 모든 스레드가 합산을 완료할 때까지 대기한다.
        } catch (InterruptedException | BrokenBarrierException e) {
            e.printStackTrace();
        }
    }
}
```
