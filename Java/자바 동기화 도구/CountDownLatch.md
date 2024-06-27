# 💡 CountDownLatch 란?

- 하나 이상의 스레드가 다른 스레드에서 수행되는 일련의 작업이 완료될 때까지 기다릴 수 있게 해주는 동기화 보조 도구이다.
- CountDownLatch는 주어진 카운트로 초기화되고 await 메서드는 현재 카운트가 countDown 메서드의 호출로 인해 0이 될 때까지 블록되며 그 이후에 모든 대기 중인 스레드가 해제되고 await() 이후 처리가 이루어진다.
- CountDownLatch 은 일회성으로 처리된다. 즉 카운트를 재 설정할 수 없다. (카운트를 재설정하는 버전이 필요한 경우 CyclicBarrier 를 사용한다.)
- 한 스레드가 여러 번 countDown() 을 호출해도 된다.
- 다음과 같은 상황에 유용하게 사용된다.
  - 여러 개의 스레드가 병렬로 실행될 때 특정 작업이 시작되거나 완료될 때까지 다른 스레드들이 기다리도록 하는 경우
  - 여러 스레드가 초기화 작업을 마칠 때까지 기다렸다가 모든 스레드가 완료되면 마무리 작업을 수행하는 경우

![image](https://github.com/shin-je-woo/TIL/assets/39439576/1aa04f09-79a3-4cac-b0f7-32e4f473033a)

# 💡 CountDownLatch API

### CountDownLatch(int count)

- 주어진 카운트로 초기화된 CountDownLatch를 생성한다.

### void await() throws InterruptedException

- 현재 카운트가 0 이라면 이 메서드는 즉시 반환되며 카운트가 0 보다 크면 두 가지 사항 중 하나가 발생할 때까지 대기 상태에 있게 된다.
  - countDown 메서드의 호출로 인해 카운트가 0 이 되는 경우
  - 다른 스레드가 현재 스레드를 인터럽트 시키는 경우
- 메서드 진입 시 인터럽트 상태가 설정되거나 또는 대기 중인 동안 인터럽트 되면 InterruptedException이 발생하며 현재 스레드의 인터럽트 상태는 초기화된다.

### boolean await(long timeout, TimeUnit unit) throws InterruptedException

- 현재 카운트가 0 이라면 이 메서드는 즉시 반환되며 카운트가 0 보다 크면 세 가지 사항 중 하나가 발생할 때까지 대기 상태에 있게 된다.
  - countDown 메서드의 호출로 인해 카운트가 0 이 되는 경우
  - 다른 스레드가 현재 스레드를 인터럽트 시키는 경우
  - 지정된 대기 시간이 경과하는 경우
- 메서드 진입 시 인터럽트 상태가 설정되거나 또는 대기 중인 동안 인터럽트 되면 InterruptedException이 발생하며 현재 스레드의 인터럽트 상태는 초기화된다.
- 지정된 대기 시간이 경과하면 false 값을 반환하며 시간이 0 이하인 경우 메서드는 전혀 대기하지 않는다.

### void countDown()

- 카운트를 감소시켜 카운트가 0 이 되면 모든 대기 중인 스레드를 해제한다.

### long getCount()

- 현재 카운트를 반환하며 이 메서드는 주로 디버깅 및 테스트 목적으로 사용된다.
