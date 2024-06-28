# Previous

- 자바 Executor Framework 는 자바의 java.util.concurrent 패키지에 포함된 스레드 관리와 병렬 처리를 위한 고급 기능들을 제공하는 포괄적인 라이브러리이다.
- Executor Framework 는 복잡한 스레드 생성, 관리, 동기화 등의 작업을 단순화하고 성능을 향상시키기 위한 다양한 클래스와 인터페이스를 제공하고 있다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/1dd60109-615c-4d77-958d-c1482a7cefa9)

# 💡 Executor

- 제출(Submit)된 Runnable 작업을 실행(Execute) 하는 객체
- Executor 인터페이스는 각 작업의 실행(실행방법, 스레드 사용, 스케줄링 등의 세부 사항) 과 작업의 제출을 분리하는 방법을 제공한다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/cc60f17f-0b5f-42ab-9af0-357bd799c1e0)

# 💡 작업제출 & 작업실행

![image](https://github.com/shin-je-woo/TIL/assets/39439576/c56d72fd-1db9-4c33-928e-c41fa02e18cb)

- 직접 스레드를 생성하고 작업을 실행하는 것이 아니라 작업을 제출하면 스레드 생성과 작업 실행은 Executor 에서 처리하도록 하는 것이 더 유연하고 좋은 설계이다.

## 더 필요한 기능

- **작업 관리의 부족**
  - 작업의 기본적인 제출과 실행만 다루며, 작업의 상태 추적, 작업 결과 반환, 작업 취소와 같은 작업 관리 기능이 제공되지 않는다.
- **작업 결과 반환의 어려움**
  - 작업이 완료되었을 때 결과를 반환받는 기능이 없어서 작업의 완료 여부 확인이나 결과 처리에 추가적인 구현이 필요하다.
- **스레드 종료의 어려움**
  - 스레드를 명시적으로 종료하거나 정리하는 기능이 없어서 스레드가 더 이상 필요하지 않을 때 스레드를 올바르게 종료하기 어렵다.
- **작업 간 상호작용의 어려움**
  - 작업을 비동기적으로 실행할 경우 작업 결과 공유나 스레드 간 여러 작업의 상호작용이 어렵다.
 
# 💡 ExecutorService

- ExecutorService는 작업(Runnable, Callable) 관리를 위한 인터페이스이다. (작업 등록, 작업 상태추적, 작업 결과 반환, 작업 취소 등등)
- ExecutorService 는 종료를 관리하는 메서드와 하나 이상의 비동기 작업 진행 상황을 추적하는 데 사용할 수 있는 Future 를 생성할 수 있는 메서드를 제공하는 Executor 이다.
- **ExecutorService 는 작업 제출부터 작업 실행, 작업 완료, 스레드 풀 종료, 자원 회수까지의 과정을 포함하는 라이프 사이클을 가지고 있다**
- 그래서 쓰레드 풀은 기본적으로 ExecutorService 인터페이스를 구현한다.
- 대표적으로 ThreadPoolExecutor가 ExecutorService의 구현체인데, ThreadPoolExecutor 내부에 있는 블로킹 큐에 작업들을 등록해둔다.
- ExecutorService가 제공하는 퍼블릭 메소드들은 다음과 같이 분류 가능하다.
  - 라이프사이클 관리를 위한 기능들
  - 비동기 작업을 위한 기능들

![img](https://github.com/shin-je-woo/TIL/assets/39439576/b3260f46-c574-4bb4-8d2f-26fcfa552412)

![image](https://github.com/shin-je-woo/TIL/assets/39439576/09613505-778a-43e7-a65c-ca736315df43)


