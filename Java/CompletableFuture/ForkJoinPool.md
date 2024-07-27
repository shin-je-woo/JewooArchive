# Previous

- ForkJoinPool 은 Java 에서 병렬 처리를 지원하는 스레드 풀로서 Java 7부터 도입 되었다.
- ForkJoinPool 은 작업을 작은 조각으로 나누고 병렬로 처리하여 다중 코어 프로세서에서 효율적으로 작업을 수행할 수 있도록 도와준다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/23cf4d9b-edb5-4de3-95d7-44087999d7eb)

# 💡 분할-정복 알고리즘(Parallelism)

![image](https://github.com/shin-je-woo/TIL/assets/39439576/cafb721d-eefe-430d-b9f8-d0767e01dec9)

# 💡 Work-Stealing

![image](https://github.com/shin-je-woo/TIL/assets/39439576/36dcd0d0-d587-44d7-9382-f4ec2bc05751)

- 스레드는 글로벌 큐에 Task 가 존재하면 자신의 WorkQueue -> head 에 push 해 놓고 Task 처리 시 head 에서 Task 를 pop 해서 가져온다.
- 스레드가 자신만의 WorkQueue -> head 에 Task 를 push 하고 pop 하기 때문에 별도의 동기화 처리가 필요하지 않게 된다.
- 다른 스레드가 자신의 WorkQueue 에 Work Stealing 을 하는 경우 WorkQueue 의 tail 에서 pop 을 하기 때문에 동기화 없이 Lock-Free 하게 구현할 수 있다.
- 결과론적으로 스레드는 자신의 WorkQueue -> head 에서 대부분 작업을 수행하고 Work-Stealing 하는 다른 스레드는 tail 에서 작업이 수행되므로 스레드 간 경합이 현저히 줄어든다.
- 다만 스레드의 개수가 상당히 많아지면 Work-Strealing 을 시도하는 스레드 간 경합이 심해질 수 있다.

# 💡 Fork-Join 구현

- ForkJoinPool 프레임워크에서 사용되는 작업 유형으로 `RecursiveTask` 및 `RecursiveAction` 이 있으며 `ForkJoinTask` 를 상속하고 있고, 병렬 작업을 수행한다.
- 추상 클래스인 RecursiveTask 또는 RecursiveAction 을 상속해서 Fork-Join 기능을 구현할 수 있다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/9a3cccd8-5a60-4052-af87-03d8a23047ea)

### RecursiveTask

- 병렬 작업을 수행하고 작업 결과를 반환한다.
- compute() 메서드를 오버라이드하여 작업을 정의하고 결과를 반환한다.
- 분할-정복 알고리즘에서 큰 작업을 작은 하위 작업으로 분할하고 각 하위 작업의 결과를 합산하여 최종 결과를 반환할 때 사용된다.

### RecursiveAction

- 병렬 작업을 수행하지만 작업 결과를 반환하지 않는다.
- compute() 메서드를 오버라이드하여 작업을 정의하며 반환 값이 없는 void 형식이다.
- 배열의 요소를 병렬로 업데이트하거나 로그를 기록하는 작업 등에서 사용된다.

# 💡 ForkJoinPool 생성

- 기본적으로 애플리케이션에서 공용으로 사용하는 스레드는 CPU 코어 개수 -1 개 만큼 생성된다. `Runtime.getRuntime().availableProcessors()-1`
- ForkJoinPool.commonPool() 은 전체 애플리케이션에서 스레드를 공용으로 사용하기 때문에 다음과 같은 주의가 필요하다.
- **스레드 블로킹**
  - I/O 바운드 작업은 스레드를 블록시키는 작업으로 commonPool()에서 실행 시 스레드 부족으로 다른 작업이 지연 될 수 있다.
  - 별도의 스레드 풀을 생성하여 I/O 작업과 CPU 작업을 분리하고 I/O 작업을 별도의 스레드에서 처리하는 것을 고려해야 한다.
- **Starvation (기아 상태)**
  - I/O 작업이 지속적으로 블록되면 CPU 작업이 실행 기회를 얻지 못하고 기아 상태에 빠질 수 있다.
  - 스레드 풀을 분리하여 CPU 작업이 충분한 실행 기회를 얻도록 관리해야 하고 대신 스레드가 필요 이상으로 생성되어 리소스 비용이 커지지 않도록 해야 한다.
 
![image](https://github.com/shin-je-woo/TIL/assets/39439576/141dd702-e0b5-441a-a710-bb94ff46f1e1)

# 💡 ForkJoinPool 실행 구조

![image](https://github.com/shin-je-woo/TIL/assets/39439576/a64f2f59-575d-4eea-bc0f-f9fc127b7330)

# 💡 스레드 풀과 비동기 작업

![image](https://github.com/shin-je-woo/TIL/assets/39439576/982266eb-8ff7-4846-b650-90c6ef44f99b)

- CompletableFuture 의 순차적 비동기 작업에서 ForkJoinPool 은 Work-Stealing 기반으로 수행되기 때문에 비동기 함수로 실행되더라도 반드시 새로운 스레드가 생성되지 않을 수 있다.
- 이것은 ForkJoinPool 의 자체 최적화에 따른 처리방식 일 수 있다.
- ThreadPoolExecutor 에서 실행되는 비동기 작업은 설정한 코어 사이즈 개수 만큼 새로운 스레드가 생성됨을 알 수 있다.
