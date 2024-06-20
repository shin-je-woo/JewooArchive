# Previous

- 자바는 단일 연산 특성(원자성)을 보장하기 위해 `synchronized` 키워드를 제공하고 있으며 synchronized 구문을 통해 모니터 영역(임계영역)을 동기화 할수 있다.
- synchronized 는 명시적으로 락을 구현하는 것이 아닌 자바에 내장된 락으로서 이를 암묵적인 락(Intrinsic Lock) 혹은 모니터락 (Monitor Lock) 이라고 한다. (고유락)
- synchronized 은 동일한 모니터를 가진 객체에 대해 오직 하나의 스레드만 임계영역에 접근할 수 있도록 보장하며 모니터의 조건 변수를 통해 스레드간 협력으로 동기화를 보장해 준다.
- synchronized 가 적용된 한 개의 메서드만 호출해도 같은 모니터의 모든 synchronized 메서드까지 락에 잠기게 되어 락이 해제될 때 까지는 접근이 안되는 특징을 가지고 있다.
- 락은 스레드가 synchronized 블록에 들어가기 전에 자동 확보되며 정상적이든 비정상적이든 예외가 발생해서든 해당 블록을 벗어날 때 자동으로 해제된다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/d322540d-1c78-400f-9539-ce5fa25486e2)

# 💡 synchronized 동기화 방식

❗ **스레드 간 객체의 메서드를 동기화하기 위해서는 스레드는 같은 객체의 모니터를 참조하고 있어야 한다!**

## 메서드 동기화 방식 - synchronized method

```java
public synchronized void syncMethod1() {
    // 동기화가 필요한 영역
}

public static synchronized void syncMethod2() {
    // 동기화가 필요한 영역
}
```

- 메소드 전체가 임계 영역(critical section)이 된다. 즉, 메소드 내의 모든 코드가 동기화 된다.
- 동시성 문제를 한번에 편리하게 제어할 수 있는 장점은 있으나 메서드 내 코드의 세부적인 동기화 구조를 가지기 어렵다.
- 메서드 전체를 동기화하기 때문에 동기화 영역이 클 경우 성능저하를 가져온다.
- 인스턴스 메서드 동기화와 정적 메서드 동기화 방식이 있다.

## 블록 동기화 방식 - synchronized block

```java
public void syncMethod1() {
    synchronized(object) {
        //동기화가 필요한 영역
    }
}

public static void syncMethod2() {
    synchronized(Class) {
        //동기화가 필요한 영역
    }
}
```

- 특정 블록을 정해서 임계 영역(critical section)을 구성한다. 즉 블록 내의 코드만 동기화 된다.
- 메서드 동기화 방식에 비해 좀 더 세부적으로 임계영역을 정해서 필요한 블록만 동기화 구조를 가질 수 있다.
- 메서드 전체를 동기화 하는 것보다 동기화 영역이 작고 효율적인 구성이 가능하기 때문에 성능 저하가 덜하다.
- 인스턴스 블록 동기화와 정적 블록 동기화 방식이 있다.

# 💡 synchronized 메서드 동기화

## 인스턴스 메소드 동기화 (synchronized method)

- 인스턴스 단위로 모니터가 동작하며 동일한 인스턴스 안에서 synchronized 가 적용된 곳은 하나의 락을 공유한다.
- 인스턴스가 여러개일 경우 인스턴스별로 모니터 객체를 가지므로 스레드는 모니터 별로 락을 획득해서 동기화 영역에 진입하고 빠져 나올 때 락을 해제 할 수 있다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/3a16a350-b097-409a-b2e4-fe8abaf9db4f)

## 정적 메소드 동기화 (static synchronized method)

- 클래스 단위로 모니터가 동작하며 synchronized 가 적용된 곳은 하나의 락을 공유한다.
- 인스턴스와는 별개의 모니터를 가지고 임계 영역을 동기화 하기 때문에 인스턴스 단위로 메서드를 호출할지라도 락은 클래스 단위로 스레드간 공유된다.
- 클래스는 메모리에 오직 하나만 존재하므로 하나의 모니터를 공유해서 동기화 하고자 할 때 사용 할 수 있다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/9c7ac023-a3cf-4068-afbb-2eaf7a5c0225)

## 인스턴스 메소드 동기화 (synchronized method) + 정적 메소드 동기화 (static synchronized method)

- `synchronized` method 와 `static synchronized` method 가 혼용되었을 경우는 각 모니터별로 동기화를 진행한다.
- 모니터가 섞여 있기 때문에 동기화가 의도한대로 정확하게 동작하는지 주의가 필요하다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/d7167c82-1328-44a9-b96a-f0410a48a4ed)
