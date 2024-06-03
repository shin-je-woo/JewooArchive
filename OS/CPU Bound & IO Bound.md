# Previous

- 프로세스는 CPU 작업과 I/O 작업의 연속된 흐름으로 진행된다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/446351c4-52ff-4ed6-b756-5aea1ab62bff)

![image](https://github.com/shin-je-woo/TIL/assets/39439576/a97b2755-6463-4684-aa2d-1356513ce930)

- 프로세스는 CPU 명령어를 수행하다가 I/O 를 만나면 대기하고 I/O 작업이 완료되면 다시 CPU 작업을 수행한다.
- 특정한 태스크가 완료될 때까지 이를 계속 반복한다.

# 💡 CPU Burst & I/O Burst

- Burst? 한 작업을 짧은 시간동안 집중적으로 연속해서 처리하거나 실행하는 것.

## CPU Burst

- CPU를 연속적으로 사용하면서 명령어를 실행하는 구간을 의미한다.
- 프로세스가 CPU 명령어를 실행하는 데 소비하는 시간
- 프로세스의 RUNNING 상태를 처리한다.

## I/O Burst

- 연속적으로 I/O를 실행하는 구간으로 I/O작업이 수행되는 동안 대기하는 구간
- 프로세스가 I/O 요청 완료를 기다리는 데 걸리는 시간
- 프로세스의 WAITING 상태를 처리한다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/bdcf1b11-aae2-4c18-8dab-956fec03cdf7)

# 💡 CPU Bounded Process & I/O Bounded Process

- 프로세스마다 CPU 버스트와 I/O 버스트가 차지하는 비율이 균일하지 않다.
- 이 비율을 기준으로 해서 CPU 바운드 프로세스와 I/O 바운드 프로세스로 나눌 수 있다.

## CPU Bounded Process

- CPU Burst 작업이 많은 프로세스로서 I/O Burst 가 거의 없는 경우에 해당한다.
- 머신러닝, 블록체인, 동영상 편집 프로그램 등 CPU 연산 위주의 작업을 하는 경우를 의미한다
- 멀티 코어의 **병렬성**을 최대한 활용해서 처리 성능을 극대화 하도록 스레드를 운용한다.
- 일반적으로 CPU 코어수와 스레드 수의 비율을 비슷하게 설정한다. (병렬성↑)

![image](https://github.com/shin-je-woo/TIL/assets/39439576/a18f3be9-ffd8-45c0-a684-045b4c18a8f4)


## I/O Bounded Process

- I/O Burst 가 빈번히 발생하는 프로세스로서 CPU Burst 가 매우 짧다.
- 파일, 키보드, DB, 네트워크 등 외부 연결이나 입출력 장치와의 통신 작업이 많은 경우에 해당한다.
- CPU 코어가 많을 경우 멀티 스레드의 **동시성**을 최대한 활용하여 CPU 가 Idle 상태가 되지 않도록 하고 최적화 된 스레드 수를 운용해서 CPU 의 효율적인 사용을 극대화 한다.
- 일반적으로 동시성을 높이려면 CPU 코어수보다 스레드 수가 많은게 좋다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/9f86b6de-8e5b-4e87-8a8d-5d25882e718c)
