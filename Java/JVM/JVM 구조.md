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
