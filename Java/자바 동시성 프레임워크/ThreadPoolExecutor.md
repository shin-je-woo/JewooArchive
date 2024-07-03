# Previous

- ThreadPoolExecutor 는 ExecutorService 를 구현한 클래스로서 매개변수를 통해 다양한 설정과 조정이 가능하며 사용자가 직접 컨트롤 할 수 있는 스레드 풀이다.
- ThreadPoolExecutor를 직접 생성하면 Executors 의 팩토리 메서드로 스레드 풀을 생성했을 때보다 스레드 풀의 세세한 조정이 가능하다는 장점이 있다.
- 생성자
  - `public ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue,ThreadFactory threadFactory, RejectedExecutionHandler handler)`
- 생성자의 매개변수를 알아보고, 동작방식에 대해 알아보자.

# 💡 corePoolSize & maximumPoolSize

- ThreadPoolExecutor 는 corePoolSize 및 maximumPoolSize 로 설정된 개수에 따라 풀 크기를 자동으로 조정한다.
- ThreadPoolExecutor 는 새 작업이 제출 될 때 corePoolSize 미만의 스레드가 실행 중이면 corePoolSize 가 될 때까지 새 스레드를 생성한다.
- corePoolSize 를 초과할 경우 큐 사이즈가 남아 있으면 큐에 작업을 추가하고 큐가 가득 차 있는 경우는 maximumPoolSize 가 될 때까지 새 스레드가 생성된다.
- setCorePoolSize 및 setMaximumPoolSize 메서드를 사용하여 동적으로 값을 변경할 수 있다.
- 기본적으로 스레드 풀은 스레드를 미리 생성하지 않고 새 작업이 도착할 때만 생성하지만 prestartCoreThread 또는 prestartAllCoreThreads 메서드를 사용하여 동적으로 재 정의할 수 있다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/368f1199-f778-46b5-9ca4-91aeec4b952f)

# 💡 keepAliveTime

- corePoolSize 보다 더 많은 스레드가 존재하는 경우 각 스레드가 keepAliveTime 보다 오랜 시간 동안 유휴 상태였다면 해당 스레드는 종료된다.
- keep-alive 정책은 corePoolSize 스레드보다 많은 스레드가 있을 때만 적용 되지만 allowCoreThreadTimeOut(boolean) 메서드를 사용하여 core 스레드에도 적용할 수 있다.
- Executors.newCachedThreadPool() 로 풀이 생성된 경우 대기 제한 시간이 60초이며 Executors.newFixedThreadPool() 로 생성된 경우 제한 시간이 없다.

# 💡 BlockingQueue

- 기본적으로 스레드 풀은 작업이 제출되면 corePoolSize 의 새 스레드를 추가해서 작업을 할당하고 큐에 작업을 바로 추가하지 않는다.
- corePoolSize 를 초과해서 스레드가 실행 중이면 새 스레드를 추가해서 작업을 할당하는 대신 큐에 작업을 추가한다. (큐가 가득찰 때까지)
- 큐에 공간이 가득차게 되고 스레드가 maxPoolSize 이상 실행 중이면 더 이상 작업은 추가되지 않고 거부된다.

## SynchronousQueue

- Executors.newCachedThreadPool() 에서 사용한다.
- 내부적으로 크기가 0 인 큐로서 스레드 간에 작업을 직접 전달하는 역할을 하며 작업을 대기열에 넣으려고 할 때 실행할 스레드가 없으면 새로운 스레드가 생성된다.
- 요소를 추가하려고 하면 다른 스레드가 해당 요소를 꺼낼 때까지 현재 스레드는 블로킹되고 요소를 꺼내려고 하면 다른 스레드가 요소를 추가할 때까지 현재 스레드는 블로킹된다.
- SynchronousQueue 는 평균적인 처리보다 더 빨리 작업이 요청되면 스레드가 무한정 증가할 수 있다.

## LinkedBlockingQueue

- Executors.newFixedThreadPool() 에서 사용한다.
- 무제한 크기의 큐로서 corePoolSize 의 스레드가 모두 사용 중인 경우 새로운 작업이 제출 되면 대기열에 등록하고 대기하게 된다.
- 무제한 크기의 큐이기 때문에 corePoolSize 의 스레드만 생성하고 더 이상 추가 스레드를 생성하지 않기 때문에 maximumPoolSize 를 설정해도 아무런 효과가 없다.
- LinkedBlockingQueue 방식은 일시적인 요청의 폭증을 완화하는 데 유용할 수 있지만 평균적인 처리보다 더 빨리 작업이 도착할 경우 대기열이 무한정 증가할 수 있다.

## ArrayBlockingQueue

- 내부적으로 고정된 크기의 배열을 사용하여 작업을 추가하고 큐를 생성할 때 최대 크기를 지정해야 하며 한 번 지정된 큐의 크기는 변경할 수 없다.
- 큰 대기열과 작은 풀을 사용하면 CPU 사용량 OS 리소스 및 컨텍스트 전환 오버헤드가 최소화 되지만 낮은 처리량을 유발할 수 있다.
- 작은 대기열과 큰 풀을 사용하면 CPU 사용률이 높아지지만 대기열이 가득 찰 경우 추가적인 작업을 거부하기 때문에 처리량이 감소할 수 있다.

# 💡 RejectedExecutionHandler

- execute(Runnable) 로 제출된 작업이 풀의 포화로 인해 거부 될 경우 execute 메서드는 RejectedExecutionHandler.rejectedExecution() 메서드를 호출한다.
- 미리 정의된 네 가지 핸들러 정책 클래스가 제공되며 직접 사용자 정의 클래스를 만들어 사용할 수도 있다.
  - `ThreadPoolExecutor.AbortPolicy` ­- 기본 값이며 작업 거부 시 RejectedExecutionException 예외를 발생시킨다.
  - `ThreadPoolExecutor.CallerRunsPolicy` -­ Executor 가 종료 되지 않은 경우 execute 를 호출한 스레드 자체가 작업을 실행한다.
  - `ThreadPoolExecutor.DiscardPolicy` - 거부된 작업은 그냥 삭제된다.
  - `ThreadPoolExecutor.DiscardOldestPolicy` -  Executor 가 종료되지 않은 경우 대기열의 맨 앞에 있는 작업을 삭제하고 실행이 재 시도 된다. 재 시도가 실패하면 반복 될 수 있다.
 
![image](https://github.com/shin-je-woo/TIL/assets/39439576/01577dd3-845d-424e-b5ea-3afa11198a5e)

# 💡 ThreadPoolExecutor Hook

- ThreadPoolExecutor 클래스는 스레드 풀을 관리하고 작업 실행 시점에 특정 이벤트를 처리하기 위한 Hook 메서드를 제공한다.
- ThreadPoolExecutor 클래스를 상속하고 Hook 메서드를 재 정의하여 작업 스레드 풀의 동작을 사용자 정의할 수 있으며 3 개의 Hook 메서드가 있다.

### beforeExecute(Thread t, Runnable r)

- 작업 스레드가 작업을 실행하기 전에 호출되는 메서드로서 제출된 각 작업마다 한 번씩 호출되고 작업 실행 전에 원하는 동작을 추가할 수 있다.

### afterExecute(Runnable r, Throwable t)

- 작업 스레드가 작업 실행을 완료한 후에 호출되는 메서드로서 제출된 각 작업마다 한 번씩 호출되고 작업 실행 후에 원하는 동작을 추가하거나 예외 처리를 수행할 수 있다.

### terminated()

- 스레드 풀이 완전히 종료된 후에 호출되는 메서드로서 스레드 풀이 종료되면 이 메서드를 재정의하여 클린업 작업을 수행할 수 있다.
- 스레드 풀이 종료될 때 한번만 호출된다.

```java
public class CustomThreadPoolExecutor extends ThreadPoolExecutor {

    public CustomThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue) {
        super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue);
    }

    @Override
    protected void beforeExecute(Thread t, Runnable r) {
        System.out.println("Thread " + t.getName() + " is about to execute task " + r.toString());
    }

    @Override
    protected void afterExecute(Runnable r, Throwable t) {
        if (t != null) {
            System.out.println("Task " + r.toString() + " encountered an exception: " + t);
        } else {
            System.out.println("Task " + r.toString() + " has completed successfully.");
        }
    }

    @Override
    protected void terminated() {
        System.out.println("Thread pool has been terminated.");
    }
}
```
