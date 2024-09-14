# 💡 JVM 이란?

- JVM이란 Java Virtual Machine 의 약자로, 자바 가상 머신이라고 부른다.
- 자바와 운영체제 사이에서 중개자 역할을 수행하며 운영체제에 독립적으로 동작할 수 있게 도와준다.
- 또한, 가비지 컬렉터를 수행하면서 자동으로 메모리를 관리해준다.

# 💡 간단하게 JVM 동작방식 알아보기

![image](https://github.com/user-attachments/assets/7f47d683-e22b-4fb6-bf28-bcd0bc236f6f)

1) 자바 프로그램이 실행되면 JVM이 메모리를 할당받는다.
2) 자바 컴파일러(javac)에 의해 컴파일된 자바 바이트 코드(.class)들은 `Class Loader`에 의해 `Runtime Data Area`에 올라간다.
3) `Runtime Data Area`에 로딩된 바이트 코드들은 `Execution Engine`에 의해 해석되고 실행된다.
4) 이 과정에서 `Execution Engine`에 의해 `Garbage Collector`가 동작되기도 한다.

- JVM은 크게 4가지 영역으로 구성된다.
- `Class Loader`, `Execution Engine`, `Garbage Collector`, `Runtime Data Area`
- 각 영역을 하나씩 알아보자.

# 💡 JVM 구조

![image](https://github.com/user-attachments/assets/f27a47e1-6a6b-4028-87cc-120d123ecf05)

## Class Loader

- 런타임 시 동적으로 바이트 코드(.class)를 JVM의 메모리 영역인 `Runtime Data Area`에 적재한다.
- Loading, Linking, Initializing 3단계를 통해 클래스 파일을 로드하고, 검증하고, 초기화하는 과정을 거친다.

## Execution Engine

- 클래스 로더에 의해 Runtime Data Area에 적재된 바이트 코드를 해석하고 실행한다.
- 바이트 코드(.class)는 운영체제에서 바로 수행할 수 있는 코드가 아니라 JVM이 이해할 수 있도록 컴파일된 코드이다.
- 실행 엔진은 바이트 코드를 운영체제가 이해할 수 있는 언어로 해석한다.
- 인터프리터, JIT 컴파일러 2가지 방식으로 바이트 코드를 해석한다.

### 인터프리터

- 명령어를 하나씩 읽어서 해석하고 실행한다.
- JVM에서 바이트코드는 기본적으로 인터프리터에 의해 해석된다.
- 그러나, 같은 코드라도 매번 해석하고 실행해야 하기 떄문에 속도가 느리다.

### JIT 컴파일러

- 인터프리터의 단점을 보완한다.
- 매번 하나씩 읽어서 수행하는 것이 아니라, 바이트 코드 전체를 컴파일 하여 Native Code로 변경한다.
- 이후, 네이티브 코드로 직접 실행한다.
- 이 과정은 비용이 비싸기 때문에 인터프리터로 동작하다가 일정 기준이 넘어가면 JIT컴파일러 방식으로 동작한다.

## Garbage Collector

- 가비지 콜렉터는 힙(Heap) 영역에 생성된 객체 중 더이상 참조되지 않는 객체를 찾아 제거하여 메모리를 확보한다.
- 개발자가 메모리관리에 특별히 신경쓸 필요가 없다.
- GC는 성능 튜닝의 대상이 되기도 한다.

## Runtime Data Area

- JVM의 메모리 영역으로 자바 런타임에 사용하는 데이터를 적재하는 공간이다.
- 크게 5가지 영역으로 구성된다.
- Method Area, Heap Area, Stack Area, PC Register, Native Method Stack

### Method Area

- 모든 쓰레드가 공유하는 메모리 영역
- 클래스, 인터페이스, 메서드, 필드, static과 같이 클래스와 관련된 데이터가 저장되는 공간이다.

### Heap Area

- 모든 쓰레드가 공유하는 메모리 영역
- 객체의 인스턴스, 배열 등 Reference Type이 저장되는 공간이다.
- Garbage Collector가 참조되지 않는 메모리를 확인하고 제거하는 영역이다.

### Stack Area

- int, long, boolean 등 기본 자료형을 저장하는 공간이다.
- 메서드의 매개변수, 지역변수, 리턴 값들을 임시 저장하는 공간이다.

### PC Register

- 현재 수행중인 JVM 명령어 주소를 저장하는 공간이다.
- 쓰레드가 어떤 부분을 어떤 명령어로 수행할지 기록한다.

### Native Method Stack

- 자바 외의 언어로 작성된 네이티브 코드를 위한 메모리 영역이다.
