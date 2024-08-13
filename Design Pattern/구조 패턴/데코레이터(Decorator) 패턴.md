# 💡 데코레이터(Decorator) 패턴

데코레이터 패턴(Decorator Pattern)은 대상 객체에 대한 기능 확장이나 변경이 필요할때 객체의 결합을 통해 서브클래싱 대신 쓸수 있는 유연한 대안 구조 패턴이다.

객체 지향 프로그래밍에서 원본 객체에 대해서 무언가를 장식하여 부가기능을 가지게 만드는 것이 데코레이터 패턴이다.

데코레이터 패턴을 이용하면 필요한 추가 기능의 조합을 런타임에서 동적으로 생성할 수 있다. 
데코레이터할 대상 객체를 새로운 행동들을 포함한 특수 장식자 객체에 넣어서 행동들을 해당 장식자 객체마다 연결시켜, 서브클래스로 구성할때 보다 훨씬 유연하게 기능을 확장 할 수 있다. 
그리고 기능을 구현하는 클래스들을 분리함으로써 수정이 용이해진다.

# 💡 데코레이터 패턴 구조

![image](https://github.com/user-attachments/assets/5b5f48ae-3d54-4ffe-80ad-21b8aef973ca)

### Component (Interface) 
- 원본 객체와 장식된 객체 모두를 묶는 역할

### ConcreteComponent
- 원본 객체 (데코레이팅 할 객체)

### Decorator
- 추상화된 장식자 클래스
- 원본 객체를 합성(composition)한 wrappee 필드와 인터페이스의 구현 메소드를 가지고 있다.

### ConcreteDecorator
- 구체적인 장식자 클래스
- 부모 클래스가 감싸고 있는 하나의 Component를 호출하면서 호출 전/후로 부가적인 로직을 추가할 수 있다.

```java
// 원본 객체와 장식된 객체 모두를 묶는 인터페이스
public interface IComponent {
    String operation();
}

// 장식될 대상 객체
public class ConcreteComponent implements IComponent {

    @Override
    public String operation() {
        return "ConcreteComponent";
    }
}
```

```java
// 장식자 추상 클래스
public abstract class Decorator implements IComponent {

    private final IComponent wrappedComponent; // 원본 객체를 composition

    public Decorator(IComponent wrappedComponent) {
        this.wrappedComponent = wrappedComponent;
    }

    @Override
    public String operation() {
        return wrappedComponent.operation(); // 위임
    }
}

// 장식자 클래스
public class MessageDecorator extends Decorator {

    public MessageDecorator(IComponent wrappedComponent) {
        super(wrappedComponent);
    }

    @Override
    public String operation() {
        String value = super.operation();
        return extraOperation(value);
    }

    private String extraOperation(String value) {
        return "*****" + value + "*****";
    }
}

// 장식자 클래스
public class TimeDecorator extends Decorator {

    public TimeDecorator(IComponent wrappedComponent) {
        super(wrappedComponent);
    }

    @Override
    public String operation() {
        LocalDateTime now = LocalDateTime.now();
        System.out.println(now);
        return super.operation();
    }
}
```

```java
public class Client {

    public static void main(String[] args) {
        // 1. 원본 객체 생성
        ConcreteComponent concreteComponent = new ConcreteComponent();
        // 2. 장식 더하기
        MessageDecorator messageDecorator = new MessageDecorator(concreteComponent);
        // 3. 장식 더하기
        TimeDecorator timeDecorator = new TimeDecorator(messageDecorator);
        // 장식된 객체의 장식된 기능 실행
        String result = timeDecorator.operation();
        System.out.println(result);
    }
}
```

# 💡 프록시 패턴 & 데코레이터 패턴

둘다 프록시를 사용하는 방법이지만 GOF 디자인 패턴에서는 이 둘을 의도(intent)에 따라서 프록시패턴과 데코레이터 패턴으로 구분한다.

- 프록시 패턴: 접근 제어가 목적
- 데코레이터 패턴: 새로운 기능 추가가 목적

둘다 프록시를 사용하지만, 의도가 다르다는 점이 핵심이다. 용어가 프록시 패턴이라고 해서 이 패턴만 프록시를 사용하는 것은 아니다. 데코레이터 패턴도 프록시를 사용한다.
