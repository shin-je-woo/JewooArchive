# Previous

- 하나의 CPU는 동일한 시간에 하나의 작업(Task)만 수행할 수 있기 때문에, 여러 프로세스를 동시에 실행할 수 없다.
- 하나의 CPU 에서 여러 프로세스를 동시성으로 처리하기 위해서는 한 프로세스에서 다른 프로세스로 전환해야 하는데, 이것을 컨텍스트 스위치(context switch) 라고 한다.

# 💡 Context

- 프로세스 간 전환을 위해서는 이전에 어디까지 명령을 수행했고, CPU Register 에는 어떤 값이 저장되어 있는지에 대한 정보가 필요하다.
- Context 는 CPU가 해당 프로세스를 실행하기 위한 프로세스의 정보를 의미하며 이 정보들은 운영체제가 관리하는 PCB 라고 하는 자료구조의 공간에 저장된다.

# 💡 PCB (Process Control Block)

- 운영체제가 시스템 내의 프로세스들을 관리하기 위해 프로세스마다 유지하는 정보를 담는 커널 내의 자료구조이다.
- 컨텍스트 스위칭은 CPU가 프로세스 간 PCB 정보를 교체하고 캐시를 비우는 일련의 과정이라 볼 수 있다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/f86c6d29-7075-43d5-9ed9-da6c993a0594)

# 💡 프로세스 상태

- 프로세스는 New(생성), 준비(Ready), 실행(Running), Blocked(대기), Exit(종료) 상태를 가진다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/8a7e74b1-0b7c-4679-afee-9d94bbbce09b)

- **New**: 프로세스를 생성하고 있는 단계로 커널 영역에 PCB가 만들어진 상태
- **Ready**: 프로세스가 CPU를 할당받기 위해 기다리고 있는 상태
- **Running**: 프로세스가 CPU를 할당받아 명령어를 실행 중인 상태
- **Waiting**: 프로세스가 I/O 작업 완료 혹은 사건 발생을 기다리고 있는 상태
- **Terminated**: 프로세스가 종료된 상태

# 💡 컨텍스트 스위칭이 일어나는 조건

- 실행 중인 프로세스에서 I/O 호출이 일어나 해당 I/O 작업이 끝날때 까지 프로세스 상태가 running 에서 waiting 상태로 전이된 경우
- Round Robin 스케줄링 등 운영체제의 CPU 스케줄러 알고리즘에 의해 현재 실행중인 프로세스가 사용할 수 있는 시간 자원을 모두 사용했을 때 해당 프로세스를
중지하고(ready 상태로 전이) 다른 프로세스를 실행시켜주는 경우

![image](https://github.com/shin-je-woo/TIL/assets/39439576/ae18c1c9-4c45-4f40-9ac5-458edbd4ee3c)

💡 컨텍스트 스위칭 동작 과정

![image](https://github.com/shin-je-woo/TIL/assets/39439576/4903c6cb-d38a-4e78-9c3a-b1cfdaac188a)

# 💡 스레드 컨텍스트 스위칭

## TCB (Thread Control Block)

- Thread 상태정보를 저장하는 자료구조이며, PC와 Register Set(CPU 정보), 그리고 PCB를 가리키는 포인터를 가진다.
- 스레드가 하나 생성될 때마다 PCB 내에서 TCB가 생성되며 컨텍스트 스위칭이 일어나면 기존의 스레드 TCB 를 저장하고 새로운 스레드의 TCB 를 가져와 실행한다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/c3b72a5a-42a7-44be-acc2-1845853505ab)

## 프로세스 vs 스레드

- 프로세스는 컨텍스트 스위칭 할 때 메모리 주소 관련 여러가지 처리(CPU 캐시 초기화, TLB 초기화, MMU 주소 체계 수정 등..) 를 하기 때문에 오버헤드가 크다.
- 스레드는 프로세스 내 메모리를 공유하기 때문에 메모리 주소 관련 추가적인 작업이 없어 프로세스에 비해 오버헤드가 작아서 컨텍스트 스위칭이 빠르다.
- 스레드는 생성하는 비용이 커서 많은 수의 스레드 생성은 메모리 부족 현상이 발생하거나 빈번한 컨텍스트 스위칭으로 인해 어플리케이션의 성능이 저하될 수 있다.
