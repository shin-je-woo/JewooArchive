# 💡 추상 팩토리(Abstract Factory) 패턴

추상 팩토리 패턴은 연관성이 있는 객체 군이 여러개 있을 경우 이들을 묶어 추상화하고, 어떤 구체적인 상황이 주어지면 팩토리 객체에서 집합으로 묶은 객체 군을 구현화 하는 생성 패턴이다. 
클라이언트에서 특정 객체를 사용할때 팩토리 클래스만을 참조하여 특정 객체에 대한 구현부를 감추어 역할과 구현을 분리시킬 수 있다.

즉, 추상 팩토리의 핵심은 **제품 '군' 집합**을 타입 별로 찍어낼수 있다는 점이 포인트 이다.
예를들어 모니터, 마우스, 키보드를 묶은 전자 제품군이 있는데 이들을 또 삼성 제품군이냐 애플 제품군이냐 로지텍 제품군이냐에 따라 집합이 브랜드 명으로 여러갈래로 나뉘게 될때, 
복잡하게 묶이는 이러한 제품군들을 관리와 확장하기 용이하게 패턴화 한것이 추상 팩토리 패턴이다.

# 💡 추상 팩토리 패턴 구조

![image](https://github.com/user-attachments/assets/bdfb3a55-a728-4739-88f8-51e6465951fa)

### AbstractFactory

- 최상위 공장 클래스. 여러개의 제품들을 생성하는 여러 메소드들을 추상화 한다.

### ConcreteFactory

- 서브 공장 클래스들은 타입에 맞는 제품 객체를 반환하도록 메소드들을 재정의한다.

### AbstractProduct

- 각 타입의 제품들을 추상화한 인터페이스

### ConcreteProduct (ProductA ~ ProductB) 

- 각 타입의 제품 구현체들. 이들은 팩토리 객체로부터 생성된다. 

### Client

- Client는 추상화된 인터페이스만을 이용하여 제품을 받기 때문에, 구체적인 제품, 공장에 대해서는 모른다.

# 💡 Abstract Factory vs Factory Method

둘다 팩토리 객체를 통해 구체적인 타입을 감추고 객체 생성에 관여하는 패턴 임에는 동일하다. 
또한 공장 클래스가 제품 클래스를 각각 나뉘어 느슨한 결합 구조를 구성하는 모습 역시 둘이 유사하다.


그러나 주의할 것은 추상 팩토리 패턴이 팩토리 메서드 패턴의 상위 호환이 아니라는 점이다. 
두 패턴의 차이는 명확하기 때문에 상황에 따라 적절한 선택을 해야 한다.

팩토리 메서드 패턴은 **객체 생성 전후로 해야 할 일**의 공통점을 정의하는데 초점을 맞추는 반면, 추상 팩토리 패턴은 생성해야 할 객체 **집합 군**의 공통점에 초점을 맞춘다.

# 💡 추상 팩토리 패턴 흐름

![image](https://github.com/user-attachments/assets/3e895c31-7ca2-46dc-a0d3-1b6cfff2946d)

▶️ 제품(Product) 클래스

```java
// Product A 제품군
public interface AbstractProductA {
}

// Product A - 1
public class ConcreteProductA1 implements AbstractProductA {
}

// Product A - 2
public class ConcreteProductA2 implements AbstractProductA {
}
```

```java
// Product B 제품군
public interface AbstractProductB {
}

// Product B - 1
public class ConcreteProductB1 implements AbstractProductB {
}

// Product B - 2
public class ConcreteProductB2 implements AbstractProductB {
}
```

▶️ 공장(Factory) 클래스

```java
public interface AbstractFactory {
    AbstractProductA createProductA();
    AbstractProductB createProductB();
}

// Product A1와 B1 제품군을 생산하는 공장군 1 
public class ConcreteFactory1 implements AbstractFactory {
    public AbstractProductA createProductA() {
        return new ConcreteProductA1();
    }
    public AbstractProductB createProductB() {
        return new ConcreteProductB1();
    }
}

// Product A2와 B2 제품군을 생산하는 공장군 2
public class ConcreteFactory2 implements AbstractFactory {
    public AbstractProductA createProductA() {
        return new ConcreteProductA2();
    }
    public AbstractProductB createProductB() {
        return new ConcreteProductB2();
    }
}
```

▶️ 클라이언트

```java
class Client {
    public static void main(String[] args) {
    	AbstractFactory factory = null;
        
        // 1. 공장군 1을 가동시킨다.
        factory = new ConcreteFactory1();

        // 2. 공장군 1을 통해 제품군 A1를 생성하도록 한다 (클라이언트는 구체적인 구현은 모르고 인터페이스에 의존한다)
        AbstractProductA product_A1 = factory.createProductA();
        System.out.println(product_A1.getClass().getName()); // ConcreteProductA1

        // 3. 공장군 2를 가동시킨다.
        factory = new ConcreteFactory2();

        // 4. 공장군 2를 통해 제품군 A2를 생성하도록 한다 (클라이언트는 구체적인 구현은 모르고 인터페이스에 의존한다)
        AbstractProductA product_A2 = factory.createProductA();
        System.out.println(product_A2.getClass().getName()); // ConcreteProductA2
    }
}
```

코드를 보면 똑같은 createProductA() 메서드를 호출하지만 어떤 팩토리 객체이냐에 따라 반환되는 제품군이 다르게 된다.

# 💡 패턴 사용 시기

- 관련 제품의 다양한 제품 군과 함께 작동해야 할때, 해당 제품의 구체적인 클래스에 의존하고 싶지 않은 경우
- 여러 제품군 중 하나를 선택해서 시스템을 설정해야하고 한 번 구성한 제품을 다른 것으로 대체할 수도 있을 때
- 제품에 대한 클래스 라이브러리를 제공하고, 그들의 구현이 아닌 인터페이스를 노출시키고 싶을 때

# 💡 장단점

### 장점

- 객체를 생성하는 코드를 분리하여 클라이언트 코드와 결합도를 낮출 수 있다.
- 제품 군을 쉽게 대체 할 수 있다.
- 단일 책임 원칙 준수
- 개방 / 폐쇄 원칙 준수

### 단점

- 각 구현체마다 팩토리 객체들을 모두 구현해주어야 하기 때문에 객체가 늘어날때 마다 클래스가 증가하여 코드의 복잡성이 증가한다. (팩토리 패턴의 공통적인 문제점)
- 기존 추상 팩토리의 세부사항이 변경되면 모든 팩토리에 대한 수정이 필요해진다. 이는 추상 팩토리와 모든 서브클래스의 수정을 가져온다. 
- 새로운 종류의 제품을 지원하는 것이 어렵다. 새로운 제품이 추가되면 팩토리 구현 로직 자체를 변경해야 한다.
