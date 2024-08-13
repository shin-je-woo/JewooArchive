# 💡 전략(Strategy) 패턴

전략 패턴은 실행(런타임) 중에 알고리즘 전략을 선택하여 객체 동작을 실시간으로 바뀌도록 할 수 있게 하는 행위 디자인 패턴이다.

여기서 '전략'이란 일종의 알고리즘이 될 수 도 있으며, 기능이나 동작이 될 수도 있는 특정한 목표를 수행하기 위한 행동 계획을 말한다.

즉, 어떤 일을 수행하는 알고리즘이 여러가지 일때, 동작들을 미리 전략으로 정의함으로써 손쉽게 전략을 교체할 수 있는, 알고리즘 변형이 빈번하게 필요한 경우에 적합한 패턴이다.

# 💡 전략 패턴 구조

![image](https://github.com/user-attachments/assets/ea5ab0af-64c4-43fb-8e2f-051d39f1b0df)

### 전략 알고리즘 객체들
- 알고리즘, 행위, 동작을 객체로 정의한 구현체

### 전략 인터페이스
- 모든 전략 구현체에 대한 공용 인터페이스
- 변하는 알고리즘

### 컨텍스트(Context) 
- 알고리즘을 실행해야 할 때마다 해당 알고리즘과 연결된 전략 객체의 메소드를 호출
- 변하지 않는 템플릿 역할

### 클라이언트
- 특정 전략 객체를 컨텍스트에 전달 함으로써 전략을 등록하거나 변경하여 전략 알고리즘을 실행한 결과를 누린다.

# 💡 전략 패턴은 OOP를 가장 잘 표현한 패턴

GoF 디자인 패턴에서는 전략 패턴을 다음과 같이 정의한다.

1) 동일 계열의 알고리즘군을 정의하고 
2) 각각의 알고리즘을 캡슐화하여 
3) 이들을 상호 교환이 가능하도록 만든다.
4) 알고리즘을 사용하는 클라이언트와 상관없이 독립적으로
5) 알고리즘을 다양하게 변경할 수 있게 한다. 

사실 전략 패턴은 자바의 여러 객체 지향 문법 기법들인 SOLID 원칙의 OCP 원칙, DIP 원칙과 합성(compositoin), 다형성(polymorphism), 캡슐화(encapsulation) 등 OOP 기술들의 총 집합 버전이라고 보면 된다.

![image](https://github.com/user-attachments/assets/8f918a55-a91a-4c2a-ad67-9d58305e2c3f)

따라서 위의 전략 패턴의 정의를 다음과 같이 빗대어 설명하면 이해하기 쉬울 것이다.

1) 동일 계열의 알고리즘군을 정의하고 → 전략 구현체로 정의
2) 각각의 알고리즘을 캡슐화하여 → 인터페이스로 추상화
3) 이들을 상호 교환이 가능하도록 만든다. → 합성(composition)으로 구성
4) 알고리즘을 사용하는 클라이언트와 상관없이 독립적으로 → 컨텍스트 객체 수정 없이
5) 알고리즘을 다양하게 변경할 수 있게 한다. → 메소드를 통해 전략 객체를 실시간으로 변경함으로써 전략을 변경

# 💡 전략 패턴 흐름

![image](https://github.com/user-attachments/assets/46756595-4ded-4a2c-a51d-5770a45e0b2e)

```java
// 전략(추상화된 알고리즘)
public interface IStrategy {
    void doSomething();
}

// 전략 알고리즘 A
public class ConcreteStrategyA implements IStrategy {

    @Override
    public void doSomething() {
        System.out.println("ConcreteStrategyA doSomething");
    }
}

// 전략 알고리즘 B
public class ConcreteStrategyB implements IStrategy {

    @Override
    public void doSomething() {
        System.out.println("ConcreteStrategyB doSomething");
    }
}
```

```java
// 컨텍스트(전략 등록/실행)
public class Context {

    private final IStrategy strategy; // // 전략 인터페이스를 합성(composition)

    public Context(IStrategy strategy) {
        this.strategy = strategy;
    }

    public void doSomething() {
        strategy.doSomething();
    }
}
```

```java
// 클라이언트(전략 교체/전략 실행한 결과를 얻음)
public class Client {

    public static void main(String[] args) {
        IStrategy concreteStrategyA = new ConcreteStrategyA();
        Context contextA = new Context(concreteStrategyA);
        contextA.doSomething();

        IStrategy concreteStrategyB = new ConcreteStrategyB();
        Context contextB = new Context(concreteStrategyB);
        contextB.doSomething();
    }
}
```

`Context` 는 변하지 않는 로직을 가지고 있는 템플릿 역할을 하는 코드이다. 전략 패턴에서는 이것을 컨텍스트(문맥)라 한다.

쉽게 이야기해서 컨텍스트(문맥)는 크게 변하지 않지만, 그 문맥 속에서 `strategy` 를 통해 일부 전략이 변경된다 생각하면 된다.

`Context` 는 내부에 `Strategy strategy` 필드를 가지고 있다. 이 필드에 변하는 부분인 `Strategy` 의 구현체를 주입하면 된다.

전략 패턴의 핵심은 `Context` 는 `Strategy` 인터페이스에만 의존한다는 점이다. 덕분에 `Strategy` 의 구현체를 변경하거나 새로 만들어도 `Context` 코드에는 영향을 주지 않는다.

스프링이 의존관계 주입에서 사용하는 방식이 바로 전략 패턴이다.

# 💡 Strategy vs Temaplate Method

### 공통점

- 전략 패턴과 템플릿 메서드 패턴은 알고리즘을 때에 따라 적용한다는 컨셉으로써, 둘이 공통점을 가지고 있다.
- 전략 및 템플릿 메서드 패턴은 개방 폐쇄 원칙(OCP)을 충족하고 코드를 변경하지 않고 소프트웨어 모듈을 쉽게 확장할 수 있도록 하는 데 사용할 수 있다. 

### 차이점

- 전략 패턴은 합성(composition)을 통해 해결책을 강구하며, 템플릿 메서드 패턴은 상속(inheritance)을 통해 해결책을 제시한다.
- 그래서 전략 패턴은 클라이언트와 객체 간의 결합이 느슨한 반면, 템플릿 메서드 패턴에서는 두 모듈이 더 밀접하게 결합된다. (결합도가 높으면 안좋음)
- 전략 패턴에서는 대부분 인터페이스를 사용하지만, 템플릿 메서드 패턴에서는 주로 추상 클래스나 구체적인 클래스를 사용한다.
- 따라서 단일 상속만이 가능한 자바에서 상속 제한이 있는 템플릿 메서드 패턴보다는, 다양하게 많은 전략을 implements 할 수 있는 전략 패턴이 많이 사용되는 편이다.
