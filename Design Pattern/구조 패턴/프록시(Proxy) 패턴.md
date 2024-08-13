# 💡 프록시(Proxy) 패턴

프록시 패턴(Proxy Pattern)은 대상 원본 객체를 대리하여 대신 처리하게 함으로써 로직의 흐름을 제어하는 행동 패턴이다.

프록시(Proxy)의 사전적인 의미는 '대리인'이라는 뜻이다. 
즉, 누군가에게 어떤 일을 대신 시키는 것을 의미하는데, 이를 객체 지향 프로그래밍에 접목해보면 클라이언트가 대상 객체를 직접 쓰는게 아니라 중간에 프록시(대리인)을 거쳐서 쓰는 코드 패턴이라고 보면 된다. 

따라서 대상 객체(Subject)의 메소드를 직접 실행하는 것이 아닌, 대상 객체에 접근하기 전에 프록시(Proxy) 객체의 메서드를 접근한 후 추가적인 로직을 처리한뒤 접근하게 된다.

![image](https://github.com/user-attachments/assets/5bf45f8d-b45b-4a85-a63a-3c4ad919f78f)

이렇게 대리자를 통해 이용하는 방식을 취하는 이유는, 대상 클래스가 민감한 정보를 가지고 있거나 인스턴스화 하기에 무겁거나 추가 기능을 가미하고 싶은데, 원본 객체를 수정할수 없는 상황일 때를 극복하기 위해서이다.

대리자가 중간에서 원본 객체 대신 여러가지 일을 수행할 수 있다.

### 보안(Security)
- 프록시는 클라이언트가 작업을 수행할 수 있는 권한이 있는지 확인하고 검사 결과가 긍정적인 경우에만 요청을 대상으로 전달한다.

### 캐싱(Caching)
- 프록시가 내부 캐시를 유지하여 데이터가 캐시에 아직 존재하지 않는 경우에만 대상에서 작업이 실행되도록 한다.

### 데이터 유효성 검사(Data validation)
- 프록시가 입력을 대상으로 전달하기 전에 유효성을 검사한다.

### 지연 초기화(Lazy initialization)
- 대상의 생성 비용이 비싸다면 프록시는 그것을 필요로 할때까지 연기할 수 있다.

### 로깅(Logging)
- 프록시는 메소드 호출과 상대 매개 변수를 인터셉트하고 이를 기록한다.

### 원격 객체(Remote objects)
- 프록시는 원격 위치에 있는 객체를 가져와서 로컬처럼 보이게 할 수 있다.

# 💡 프록시 패턴 구조

프록시는 다른 객체에 대한 접근을 제어하는 개체이다. 
여기서 다른 객체를 대상(Subject)라고 부른다. 프록시와 대상은 동일한 인터페이스를 가지고 있으며 이를 통해 다른 인터페이스와 완전히 호환되도록 바꿀수 있다.

![image](https://github.com/user-attachments/assets/d2dd000d-08ef-4e41-913d-0a71dac6e86e)

### Subject
- Proxy와 RealSubject를 하나로 묶는 인터페이스 (다형성)
- 대상 객체와 프록시 역할을 동일하게 하는 추상 메소드 operation() 를 정의한다.
- 인터페이스가 있기 때문에 클라이언트는 Proxy 역할과 RealSubject 역할의 차이를 의식할 필요가 없다.

### RealSubject
- 원본 대상 객체

### Proxy
- 대상 객체(RealSubject)를 중계할 대리자 역할
- 프록시는 대상 객체를 합성(composition)한다.
- 프록시는 대상 객체와 같은 이름의 메서드를 호출하며, 별도의 로직을 수행 할수 있다. (인터페이스 구현 메소드)
- 프록시는 흐름제어만 할 뿐 결과값을 조작하거나 변경시키면 안 된다.

### Client 
- Subject 인터페이스를 이용하여 프록시 객체를 생성해 이용.
- 클라이언트는 프록시를 중간에 두고 프록시를 통해서 RealSubject와 데이터를 주고 받는다.

# 💡 예제

대상 객체가 값을 반환하려면 1초 동안의 시간이 소요되는 작업을 진행한다고 가정하자. 그리고, 대상 객체는 매번 같은 값을 반환한다.

이 때, 프록시를 도입하여 대상 객체의 값을 캐싱하여 저장해두고, 클라이언트에게 이미 조회된 데이터를 제공하면 성능 상 이점이 있다.

```java
public interface Subject {
    String operation();
}
```

```java
// 대상 객체
public class RealSubject implements Subject {

    @Override
    public String operation() {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        return "fixed value";
    }
}
```

```java
// 프록시 객체
public class CacheProxy implements Subject {

    private final Subject target;
    private String cacheValue;

    public CacheProxy(Subject target) {
        this.target = target;
    }

    @Override
    public String operation() {
        // 대상 객체의 반환값을 캐싱해둔다. (아래의 코드는 동기화가 필요하지만 생략)
        if (cacheValue == null) {
            cacheValue = target.operation();
        }
        // 클라이언트는 캐싱된 값을 반환받는다.
        return cacheValue;
    }
}
```

```java
public class Client {

    public static void main(String[] args) {
        Subject subject = new CacheProxy(new RealSubject());
        String operation1 = subject.operation(); // 처음만 1초 소요
        String operation2 = subject.operation(); // 이후부터 캐시된 데이터 조회
        String operation3 = subject.operation(); // 이후부터 캐시된 데이터 조회
        String operation4 = subject.operation(); // 이후부터 캐시된 데이터 조회
        System.out.println("operation1 = " + operation1);
        System.out.println("operation2 = " + operation2);
        System.out.println("operation3 = " + operation3);
        System.out.println("operation4 = " + operation4);
    }
}
```

# 💡 장단점

### 장점

- 개방 폐쇄 원칙(OCP) 준수 -> 기존 대상 객체의 코드를 변경하지 않고 새로운 기능을 추가할 수 있다.
- 단일 책임 원칙(SRP) 준수 -> 대상 객체는 자신의 기능에만 집중 하고, 그 이외 부가 기능을 제공하는 역할을 프록시 객체에 위임하여 다중 책임을 회피 할 수 있다.
- 원래 하려던 기능을 수행하며 그외의 부가적인 작업(로깅, 인증, 네트워크 통신 등)을 수행하는데 유용하다.
- 클라이언트는 객체를 신경쓰지 않고, 서비스 객체를 제어하거나 생명 주기를 관리할 수 있다.
- 사용자 입장에서는 프록시 객체나 실제 객체나 사용법은 유사하므로 사용성에 문제 되지 않는다.

### 단점

- 많은 프록시 클래스를 도입해야 하므로 코드의 복잡도가 증가한다.
  - 예를들어 여러 클래스에 로깅 기능을 가미 시키고 싶다면, 동일한 코드를 적용함에도 각각의 클래스에 해당되는 프록시 클래스를 만들어서 적용해야 되기 때문에 코드량이 많아지고 중복이 발생 된다.
  - 자바에서는 동적 프록시(Dynamic Proxy) 기법을 이용해서 해결할 수 있다.
- 프록시 클래스 자체에 들어가는 자원이 많다면 서비스로부터의 응답이 늦어질 수 있다.
