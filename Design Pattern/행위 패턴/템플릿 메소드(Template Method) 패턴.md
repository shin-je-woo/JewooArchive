# 💡 템플릿 메서드(Template Method) 패턴

템플릿 메서드(Template Method) 패턴은 여러 클래스에서 공통으로 사용하는 메서드를 템플릿화 하여 상위 클래스에서 정의하고, 하위 클래스마다 세부 동작 사항을 다르게 구현하는 패턴이다.

즉, 변하지 않는 기능(템플릿)은 상위 클래스에 만들어두고 자주 변경되며 확장할 기능은 하위 클래스에서 만들도록 하여, 상위의 메소드 실행 동작 순서는 고정하면서 세부 실행 내용은 다양화 될 수 있는 경우에 사용된다.

템플릿 메소드 패턴은 상속이라는 기술을 극대화하여, 알고리즘의 뼈대를 맞추는 것에 초점을 둔다. 이미 수많은 프레임워크에서 많은 부분에 템플릿 메소드 패턴 코드가 우리도 모르게 적용되어 있다.

# 💡 템플릿 메서드 패턴 구조

![image](https://github.com/user-attachments/assets/bc974092-b9f6-4b9c-9dee-8c0958712b1f)

### AbstractClass(추상 클래스)
- 템플릿 메소드를 구현하고, 템플릿 메소드에서 돌아가는 추상 메소드를 선언한다.
- 이 추상 메소드는 하위 클래스인 ConcreteClass 역할에 의해 구현된다.

### ConcreteClass(구현 클래스)
- AbstractClass를 상속하고 추상 메소드를 구체적으로 구현한다.
- ConcreteClass에서 구현한 메소드는 AbstractClass의 템플릿 메소드에서 호출된다.

# 💡 hook 메서드

![image](https://github.com/user-attachments/assets/5c3ce8ac-6396-49d5-a598-ee7b50b13b16)

훅(hook) 메소드는 부모의 템플릿 메서드의 영향이나 순서를 제어하고 싶을때 사용되는 메서드 형태를 말한다.

위의 그림에서 보듯이 템플릿 메서드 내에 실행되는 동작을 step2() 라는 메서드의 참, 거짓 유무에 따라 다음 스텝을 어떻게 이어나갈지 지정한다. 이를 통해 자식 클래스에서 좀더 유연하게 템플릿 메서드의 알고리즘 로직을 다양화 할 수 있다는 특징이 있다.

훅 메소드는 추상 메소드가 아닌 일반 메소드로 구현하는데, 선택적으로 오버라이드 하여 자식 클래스에서 제어하거나 아니면 놔두거나 하기 위해서 이다.

# 💡 템플릿 메서드 패턴 흐름

![image](https://github.com/user-attachments/assets/f805564f-6ac1-40c6-87e7-09c21ee41513)

```java
public abstract class AbstractTemplate {

    // 템플릿 메소드 : 메서드 앞에 final 키워드를 붙이면 자식 클래스에서 오버라이딩이 불가능함.
    // 자식 클래스에서 상위 템플릿을 오버라이딩해서 바꾸는 행위를 금지
    public final void templateMethod() {
        // 상속하여 구현되면 실행될 메소드들
        step1();
        step2();
        
        if(hook()) { // 안의 로직을 실행하거나 실행하지 않음
            // ...
        }
        
        step3();
    }
    
    boolean hook() {
        return true;
    }
    
    // 상속하여 사용할 것이기 때문에 protected 접근제어자 설정
    protected abstract void step1();
    protected abstract void step2();
    protected abstract void step3();
}
```

```java
public class ImplementationA extends AbstractTemplate {

    @Override
    protected void step1() {}

    @Override
    protected void step2() {}

    @Override
    protected void step3() {}
}

public class ImplementationB extends AbstractTemplate {

    @Override
    protected void step1() {}

    @Override
    protected void step2() {}

    @Override
    protected void step3() {}

    // hook 메소드를 오버라이드 해서 false로 하여 템플릿에서 마지막 로직이 실행되지 않도록 설정
    @Override
    protected boolean hook() {
        return false;
    }
}
```

```java
class Client {
   public static void main(String[] args) {
       // 1. 템플릿 메서드가 실행할 구현화한 하위 알고리즘 클래스 생성
       AbstractTemplate templateA = new ImplementationA();

       // 2. 템플릿 실행
       templateA.templateMethod();
   }
}
```

# 💡 장단점

### 장점

- 클라이언트가 대규모 알고리즘의 특정 부분만 재정의하도록 하여, 알고리즘의 다른 부분에 발생하는 변경 사항의 영향을 덜 받도록 한다.
- 상위 추상클래스로 로직을 공통화 하여 코드의 중복을 줄일 수 있다.
- 서브 클래스의 역할을 줄이고, 핵심 로직을 상위 클래스에서 관리하므로써 관리가 용이해진다.

### 단점

- 상속에서 오는 단점들을 그대로 안고간다. 특히 자식 클래스가 부모 클래스와 컴파일 시점에 강하게 결합되는 문제가 있다.
- 알고리즘의 제공된 골격에 의해 유연성이 제한될 수 있다.
- 알고리즘 구조가 복잡할수록 템플릿 로직 형태를 유지하기 어려워진다.
- 추상 메소드가 많아지면서 클래스의 생성, 관리가 어려워질 수 있다.
- 상위 클래스에서 선언된 추상 메소드를 하위 클래스에서 구현할 때, 그 메소드가 어느 타이밍에서 호출되는지 클래스 로직을 이해해야 할 필요가 있다.
- 로직에 변화가 생겨 상위 클래스를 수정할 때, 모든 서브 클래스의 수정이 필요 할수도 있다.

> 이런 상속의 단점을 보완하기 위해 전략(Strategy) 패턴을 사용하기도 한다. 전략 패턴은 "상속보다는 컴포지션을" 이라는 컨셉이다.
