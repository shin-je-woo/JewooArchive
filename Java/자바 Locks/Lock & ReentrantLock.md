# Previous

- Lock 구현은 synchronized 구문과 마찬가지로 `상호배제` 와 `가시성` 기능을 가진 동기화 기법이며 synchronized 보다 더 확장된 락 작업을 제공한다.
- Lock 구현은 락 획득 시 블록되지 않는 비 차단 시도(tryLock()), 인터럽트가 가능한 방식으로 락을 획득하는 시도(lockInterruptibly) 및 시간 제한을 둔 방식으로 락을 획득하는 시도(tryLock(long, TimeUnit))와 같은 추가 기능을 제공한다.
- synchronized 사용은 락 획득과 락 해제가 블록 구조화된 방식으로 발생하도록 강제한다.
  - 여러 락을 획득하면 반드시 반대 순서로 해제해야 하며 모든 락은 동일한 문장 블록 범위에서 획득하고 해제되어야 한다.
- Lock 구현은 락을 더 유연한 방식으로 작업할 수 있도록 지원한다.
  - 노드 A의 락을 획득한 다음 노드 B의 락을 획득하고 A를 해제하고 C를 획득하고 B를 해제하고 D를 획득하고 이런 식으로 진행할 수 있다.
  - 즉, 락을 다른 범위에서 획득하고 해제하도록 허용하며 락을 어떤 순서로든 획득하고 해제할 수 있도록 허용한다.
- synchronized 는 블록을 벗어나면 락 해제가 자동적으로 이루어지지만 Lock 구현은 명시적으로 락을 해제해 주어야 한다.

# 💡 Lock 구조

![image](https://github.com/shin-je-woo/TIL/assets/39439576/7e45d516-48e9-4565-9374-e9cb5b080744)

# 💡 ReentrantLock API

### void lock()

- 락이 다른 스레드에 의해 보유되고 있지 않다면 락을 즉시 획득하고 락의 보유 횟수를 1로 설정한다.
- 현재 스레드가 이미 이 락을 보유하고 있다면 보유 횟수가 1 증가하고 메서드는 즉시 반환된다. (락의 재진입 가능)
- 락이 다른 스레드에 의해 보유되어 있다면 현재 스레는 락이 획득될 때까지 대기하며 이후에 락을 성공적으로 획득하면 보유 횟수가 1로 설정된다.

### void lockInterruptibly() throws InterruptedException

- 현재 스레드가 인터럽트 되지 않는 한 락을 획득하며 다른 스레드에 의해 보유되지 않는다면 락을 즉시 획득하고 락의 보유 횟수를 1로 설정한다.
- 현재 스레드가 이미 이 락을 보유하고 있다면 보유 횟수가 1 증가하고 메서드는 즉시 반환된다.
- 락이 다른 스레드에 의해 보유되어 있다면 락이 현재 스레드에 의해 획득될 때까지 대기한다.
- 현재 스레드가 이 메서드에 진입할 때 인터럽트 상태가 설정되어 있는 경우나 또는 락을 획득하는 도중에 인터럽트가 발생한 경우 InterruptedException이 발생하며 인터럽트 상태는 초기화 된다.
- 락을 정상적으로 또는 재진입으로 획득하는 것보다 인터럽트에 응답하는 것이 우선적으로 처리된다.

```java
try {
    // 락 획득을 시도하며 인터럽트에 의해 중단 가능
    lock.lockInterruptibly();
    System.out.println("Lock acquired");
} catch (InterruptedException e) {
    // 락 시도 중에 인트럽트에 걸리면 예외구분에서 별도 처리
    System.out.println("Interrupted while acquiring lock");
} finally {
    lock.unlock();
    System.out.println("Lock released");
}
```

### boolean tryLock()

- 이 락이 호출 시점에 다른 스레드에 의해 보유되지 않을 때만 락을 획득하고 락의 보유 횟수를 1로 설정하고 true 를 반환한다.
- 이 락이 공정성을 가지도록 설정되었더라도 현재 다른 스레드가 락을 기다리는지 여부와 관계없이 락이 사용 가능한 경우 즉시 락을 획득한다. (비공정성)
- 현재 스레드가 이미 이 락을 보유하고 있다면 보유 횟수가 1 증가하고 true 를 반환한다.
- 락이 다른 스레드에 의해 소유되어 있다면 이 메서드는 즉시 false 값을 반환한다. 그리고 이 메서드는 락을 획득하지 못하더라도 스레드가 대기하거나 차단되지 않는다.

```java
try {
    // 락획득 여부를 즉시 반환함
    // 락 획득을 실패하더라도 스레드가 대기하거나 차단되지 않음
    if (lock.tryLock()) {
        System.out.println(" Lock acquired")
    } else {
        // 락을 획득하지 못했을 경우 별도 처리
        System.out.println("Lock not acquired");
    }
} finally {
    lock.unlock(); // finally 에서 락 해제
    System.out.println("Lock released");
}
```

### boolean tryLock(long time, TimeUnit unit) throws InterruptedException

- 주어진 대기 시간 내에 다른 스레드에 의해 보유되지 않으면 락을 획득하고 락의 보유 횟수를 1로 설정하고 true 를 반환한다.
- 이 락이 공정성을 가지도록 설정 되었다면 락이 사용 가능한 경우에 즉시 락을 획득하지 않는다. 이는 tryLock() 메서드와는 대조적이라 할 수 있다. (공정성)
- 현재 스레드가 이미 이 락을 보유하고 있다면 보유 횟수가 1 증가하고 메서드는 true 를 반환하고 락이 다른 스레드에 의해 보유되어 있다면 락이 획득될 때까지 대기한다.
- 현재 스레드가 이 메서드를 호출할 때 인터럽트 상태가 설정되어 있거나 락을 획득하는 동안 인터럽트가 발생한 경우 InterruptedException 이 발생 되고 인터럽트 상태가 초기화된다.
- 지정된 대기 시간이 경과하면 값 false 가 반환되며 만약 시간이 0보다 작거나 같으면 메서드는 전혀 대기하지 않는다.
- 락의 정상적인 또는 재진입 획득 및 대기 시간 경과 보다 인터럽트에 응답하는 것이 우선적으로 처리된다.

```java
try {
    // 최대 2초 동안 락 획득을 시도
    // 시간 경과하면 락 획득 실패하고 false 반환
    if (lock.tryLock(2, TimeUnit.SECONDS)) {
        try {
            System.out.println("Lock acquired");
        } finally {
            lock.unlock();
            System.out.println("Lock released");
        }
    } else {
        // 락을 획득하지 못했을 경우 별도 처리
        System.out.println("Could not acquire lock within 2 seconds");
    }
} catch (InterruptedException e) {
    // 락 시도 중에 인트럽트에 걸리면 예외구문에서 별도 처리
    e.printStackTrace();
}
```

### void unlock()

- 이 락을 해제하려고 시도한다.
- 락을 해제하려면 동일한 스레드에서 lock() 메서드가 호출된 횟수와 동일한 횟수로 호출해야 한다. 즉, unlock() 메서드가 호출될 때 마다 락 카운트가 감소되며 락 카운트가 0 이면 락이 해제된다.
- 현재 스레드가 이 락의 소유자가 아닌 경우 IllegalMonitorStateException 이 발생한다.

### Condition newCondition()

- Lock 인스턴스와 함께 사용하기 위한 Condition 인스턴스를 반환한다.
- Condition 인스턴스는 Object 모니터 메서드(wait, notify 및 notifyAll)와 동일한 용도를 지원하며 대기 또는 신호 메서드가 호출될 때 락이 없을 경우 IllegalMonitorStateException 이 발생한다.
- 대기 메서드가 호출되면 락이 해제되고 신호 메서드가 호출되어 대기 메서드에서 반환하기 전에 스레드는 락을 다시 획득한다.
- 스레드가 대기하는 동안 인터럽트가 발생하면 대기가 종료되고 InterruptedException 이 발생하며 스레드의 인터럽트 상태가 초기화된다.
- 대기 중인 스레드는 FIFO 순서로 신호를 받는다. 즉, 먼저 대기한 스레드 순서로 신호를 받게 된다.
- 신호에 의해 대기 메서드에서 깨어난 스레드의 락 재획득 순서는 기본적으로 락을 획득하는 방식과 동일하나, 공정 락의 경우 가장 오래 기다리는 스레드에게 우선권이 주어진다.

# 💡 synchronized & Lock 구현 비교

- synchronized 구문은 락의 획득과 해제가 내장되어 있어 암묵적인 락이라고 하고 Lock 은 락의 회득과 해제를 직접 명시해서 사용하기 때문에 명시적인 락이라고 한다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/5ae69cb2-4bee-46e8-afa4-f1929eddb266)

# 💡 synchronized 와 ReentrantLock 선택 기준

### ReentrantLock

- **lockInterruptibly()**
  - 락 획득을 시도하거나 대기하는 중에 중단이 필요한 경우
- **tryLock()**
  - 비차단 락 획득이 필요한 경우
- **tryLock(long time, TimeUnit unit)**
  - 지정된 시간 안에 락 획득이 필요한 경우
- **new ReentrantLock(true)**
  - 공정하게 락을 획득하는 정책을 사용하는 경우
- **{ lock.lock() } { … } { lock.unlock() }**
  - 락의 획득과 해제가 단일 블록을 벗어는 경우
 
### synchronized

- ReentrantLock 의 기능이 필요하지 않은 경우
- 사용하기 더 편리하고 익숙하다.
- 성능상 크게 차이가 나지 않으며 락 해제를 명시적으로 할 필요가 없다.
- 복잡하지 않고 문법적으로 더 간단하며 단순한 동기화에서는 가독성이 좋을 수 있다.
