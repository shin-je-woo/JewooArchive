# 💡 싱글톤(SingleTon) 패턴

싱글톤 패턴이란 단 하나의 유일한 객체를 만들기 위한 코드 패턴이다.

쉽게 말하자면 메모리 절약을 위해, 인스턴스가 필요할 때 똑같은 인스턴스를 새로 만들지 않고 기존의 인스턴스를 가져와 활용하는 기법을 말한다.

따라서 보통 싱글톤 패턴이 적용된 객체가 필요한 경우는 그 객체가 리소스를 많이 차지하는 역할을 하는 무거운 클래스일때 적합하다.

대표적으로 데이터베이스 연결 모듈을 예로 들 수 있는데, 데이터베이스에 접속하는 작업(I/O 바운드)은 그 자체로 무거운 작업에 속하며, 한번만 객체를 생성하고 돌려쓰면 되지 굳이 여러번 생성할 필요가 없기 때문이다.

이밖에도 디스크 연결, 네트워크 통신, DBCP 커넥션풀, 스레드풀, 캐시, 로그 기록 객체 등에 이용된다.

# 💡 싱글톤 패턴 구현 기법

싱글톤 패턴을 구현하는 다양한 방법이 있다.

아래는 대표적인 방법 7가지이다.

1) Eager Initialization
2) Static block initialization
3) Lazy initialization
4) Thread safe initialization
5) Double-Checked Locking
6) **Bill Pugh Solution**
7) **Enum 이용**

## Eager Initialization

- 한번만 미리 만들어두는, 가장 직관적이면서도 심플한 기법
- static final 이라 멀티 쓰레드 환경에서도 안전함
- 그러나 static 멤버는 당장 객체를 사용하지 않더라도 메모리에 적재하기 때문에 만일 리소스가 큰 객체일 경우, 공간 자원 낭비가 발생함
- 예외 처리를 할 수 없음

```java
public class Singleton {
    // 싱글톤 클래스 객체를 담을 인스턴스 변수
    private static final Singleton INSTANCE = new Singleton();

    // 생성자를 private로 선언 (외부에서 new 사용 X)
    private Singleton() {}

    public static Singleton getInstance() {
        return INSTANCE;
    }
}
```

## Static block initialization

- static block 을 이용해 예외를 잡을 수 있음
- 그러나 여전히 static 의 특성으로 사용하지도 않는데도 공간을 차지함

```java
public class Singleton {
    // 싱글톤 클래스 객체를 담을 인스턴스 변수
    private static Singleton instance;

    // 생성자를 private로 선언 (외부에서 new 사용 X)
    private Singleton() {}
    
    // static 블록을 이용해 예외 처리
    static {
        try {
            instance = new Singleton();
        } catch (Exception e) {
            throw new RuntimeException("싱글톤 객체 생성 오류");
        }
    }

    public static Singleton getInstance() {
        return instance;
    }
}
```

- 객체 생성에 대한 관리를 내부적으로 처리
- 메서드를 호출했을 때 인스턴스 변수의 null 유무에 따라 초기화 하거나 있는 걸 반환하는 기법
- 위의 미사용 고정 메모리 차지의 한계점을 극복
- 그러나 쓰레드 세이프(Thread Safe) 하지 않은 치명적인 단점을 가지고 있음

```java
public class Singleton {
    // 싱글톤 클래스 객체를 담을 인스턴스 변수
    private static Singleton instance;

    // 생성자를 private로 선언 (외부에서 new 사용 X)
    private Singleton() {}
	
    // 외부에서 정적 메서드를 호출하면 그제서야 초기화 진행 (lazy)
    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton(); // 오직 1개의 객체만 생성
        }
        return instance;
    }
}
```

# 💡 Thread safe initialization

- `synchronized` 키워드와 같은 동기화 기법을 통해 메서드에 쓰레드들을 하나하나씩 접근하게 하도록 설정한다.
- 하지만 여러개의 모듈들이 매번 객체를 가져올 때 `synchronized` 메서드를 매번 호출하여 동기화 처리 작업에 overhead가 발생해 성능 하락이 발생한다.

```java
public class Singleton {
    private static Singleton instance;

    private Singleton() {}

    // synchronized 메서드
    public static synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

# 💡 Double-Checked Locking

- 매번 synchronized 동기화를 실행하는 것이 문제라면, 최초 초기화할때만 적용 하고 이미 만들어진 인스턴스를 반환할때는 사용하지 않도록 하는 기법
- 이때 인스턴스 필드에 volatile 키워드를 붙여주어야 I/O 불일치 문제를 해결 할 수 있다.
- 그러나 volatile 키워드를 이용하기위해선 JVM 1.5이상이어야 되고, JVM에 대한 심층적인 이해가 필요하여, JVM에 따라서 여전히 쓰레드 세이프 하지 않는 경우가 발생하기 때문에 사용하기를 지양하는 편이다.

```java
public class Singleton {
    private static volatile Singleton instance; // volatile 키워드 적용해서 가시성 확보

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
        	  // 동기화 이후 더블 체킹
            synchronized (Singleton.class) { 
                if(instance == null) { 
                    instance = new Singleton(); // 최초 초기화만 동기화 작업이 일어나서 리소스 낭비를 최소화
                }
            }
        }
        return instance; // 최초 초기화가 되면 앞으로 생성된 인스턴스만 반환
    }
}
```

# 💡 Bill Pugh Solution (LazyHolder)

- 권장되는 두가지 방법중 하나
- 멀티쓰레드 환경에서 안전하고 Lazy Loading(나중에 객체 생성) 도 가능한 완벽한 싱글톤 기법
- 클래스 안에 내부 클래스(holder)를 두어 JVM의 클래스 로더 매커니즘과 클래스가 로드되는 시점을 이용한 방법 (스레드 세이프함)
- static 메소드에서는 static 멤버만을 호출할 수 있기 때문에 내부 클래스를 static으로 설정
- 이 밖에도 내부 클래스의 치명적인 문제점인 메모리 누수 문제를 해결하기 위하여 내부 클래스를 static으로 설정

```java
public class Singleton {

    private Singleton() {}

    // static 내부 클래스를 이용
    // Holder로 만들어, 클래스가 메모리에 바로 로드되지 않고 getInstance 메서드가 호출되어야 로드됨
    private static class SingleInstanceHolder {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return SingleInstanceHolder.INSTANCE;
    }
}
```

1) 우선 내부클래스를 static으로 선언하였기 때문에, 싱글톤 클래스가 초기화되어도 SingleInstanceHolder 내부 클래스는 메모리에 로드되지 않음
2) 어떠한 모듈에서 getInstance() 메서드를 호출할 때, SingleInstanceHolder 내부 클래스의 static 멤버를 가져와 리턴하게 되는데, 이때 내부 클래스가 한번만 초기화되면서 싱글톤 객체를 최초로 생성 및 리턴하게 된다.
3) 마지막으로 final 로 지정함으로서 다시 값이 할당되지 않도록 방지한다.

# 💡 Enum 사용

- 권장되는 두가지 방법중 하나 
- enum은 애초에 멤버를 만들때 private로 만들고 한번만 초기화 하기 때문에 thread safe함.
- enum 내에서 상수 뿐만 아니라, 변수나 메서드를 선언해 사용이 가능하기 때문에, 이를 이용해 싱글톤 클래스 처럼 응용이 가능

```java
public enum SingletonEnum {
    INSTANCE;

    private final Client dbClient;
	
    SingletonEnum() {
        dbClient = Database.getClient();
    }

    public static SingletonEnum getInstance() {
        return INSTANCE;
    }

    public Client getClient() {
        return dbClient;
    }
}

public class Main {
    public static void main(String[] args) {
        SingletonEnum singleton = SingletonEnum.getInstance();
        singleton.getClient();
    }
}
```
