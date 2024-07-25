# Previous

Java 에서 Lambda 를 사용하다 보면 아래와 같은 메시지를 만나게 된다.

> Variable is accessed from within inner class needs to be final or effectively final

"내부클래스(Inner Class)안에 있는 변수에 접근할 때는 그 값이 final 이나 effectively final 형태여야 한다."

보통 이런 에러는 람다 내부에서 외부의 지역변수를 변경하는 코드를 작성했을 때 발생한다.

```java
int cnt = 0;
userList.forEach(user -> {
    if (user.getAge() > 30) {
        cnt++;
    }
});
```

# 💡 람다와 지역변수

Lambda 에서는 다음과 같은 특징이 있다.

- 람다식은 외부 block 에 있는 변수에 접근할 수 있다.
- 외부에 있는 변수가 지역변수 일 경우 final 혹은 effectively final 인 경우에만 접근이 가능하다.
- 복사된 지역 변수 값은 람다식 내부에서도 변경할 수 없다.

#### 📌 effectively final?

- A non-final local variable or method parameter whose value is never changed after initialization is known as effectively final.
- Java 8 에 추가된 syntactic sugar 일종으로, 초기화된 이후 값이 한 번도 변경되지 않았다면 effectively final 이라고 할 수 있다.

람다는 왜 이러한 제약 조건을 만들었을까?

### 람다식에서 참조하는 외부 지역 변수는 복사 값이다.

지역 변수를 관리하는 스레드와 람다식이 실행되는 스레드가 다를 수 있다.

만약, 지역 변수를 할당한 스레드가 사라져서 변수가 Stack에서 제거되었는데도 람다를 실행하는 스레드에서 해당 변수에 접근하려 할 수 있다. (문제 발생 여지가 있다.)

또한, 지역 변수는 Stack영역에 저장된다. Stack은 스레드 고유의 공간이며, 스레드끼리 공유하지 않는다.

이런 이유로 자바에서는 원래 변수에 접근을 허용하는 것이 아니라 지역 변수의 복사본을 람다에 제공한다.

따라서, 복사본의 값이 바뀌지 않아야 하므로 지역 변수에는 한 번만 값을 할당해야 한다는 제약이 생긴 것이다.

### 인스턴스 변수는 람다에서 자유롭게 캡처할 수 있다.

지역 변수는 Stack에 저장되는 반면, 인스턴스 변수는 Heap에 저장된다.

Stack은 스레드마다 별도의 공간을 할당받지만, Heap영역은 모든 스레드가 공유한다.

또한, 값이 메모리에서 바로 회수되지 않기 때문에 람다 실행 시 값이 존재하는 것을 보장할 수 있다.

그래서 인스턴스 변수는 람다에서 자유롭게 캡처할 수 있는 것이다.

# 💡 정리

- 람다식에서 외부 지역 변수를 참조하여 사용할 경우 final 혹은 effectively final 이어야 하는 이유는 지역 변수가 Stack(스택)에 저장되기 때문이다.
- 람다식에서는 값을 바로 참조하는 것에 제약이 있어 복사된 값을 이용하게 되는데, 이때 멀티쓰레드 환경에서 복사될/된 값이 변경 가능할 경우 이로 인한 이슈를 대응할 수 없다.
