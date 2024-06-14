# Previous

- 자바에서 스레드는 오직 자신만이 접근해서 읽고 쓸수 있는 로컬 변수 저장소를 제공하는데, 이를 `ThreadLocal` 이라고 한다.
- 각 스레드는 고유한 `ThreadLocal` 객체를 속성으로 가지고 있으며 `ThreadLocal` 은 스레드 간 격리되어 있다.
- 스레드는 `ThreadLocal` 에 저장된 값을 특정한 위치나 시점에 상관없이 어디에서나 전역변수처럼 접근해서 사용할 수 있다.
- 모든 스레드가 공통적으로 처리해야 하는 기능이나 객체를 제어해야 하는 상황에서 스레드마다 다른 값을 적용해야 하는 경우 사용한다. (인증 주체 보관, 트랜잭션 전파, 로그 추적기 등)

![image](https://github.com/shin-je-woo/TIL/assets/39439576/841c8c37-68a7-489b-a602-9df7cd6f1d00)

# 💡 ThreadLocal API

- **void set(T value)**
  - 스레드 로컬에 값을 저장한다.
- **T get()**
  - 스레드 로컬에 저장된 값을 가져온다.
- **void remove()**
  - 스레드 로컬에 저장된 값을 삭제한다.
- **withInitial(Supplier<? extends S> supplier)**
  - 스레드 로컬을 생성하면서 특정 값으로 초기화한다.
 
```java
private final static ThreadLocal<String> threadLocal = ThreadLocal.withInitial(() -> "defaultName"); // 생성 시 디폴트 값으로 초기화
threadLocal.set(“Hello World”) // Hello World 저장
threadLocal.get(); // Hello World 조회
threadLocal.remove(); // Hello World 제거
```

# 💡 Thread & ThreadLocal

- 스레드는 `ThreadLocal` 에 있는 `ThreadLocalMap` 객체를 자신의 `threadLocals` 속성에 저장한다.
- 스레드 생성 시 `threadLocals` 기본값은 null 이며 `ThreadLocal` 에 값을 저장할 때 `ThreadLocalMap` 이 생성되고 `threadLocals` 와 연결된다.
- 스레드가 전역적으로 값을 참조할 수 있는 원리는 스레드가 `ThreadLocal` 의 `ThreadLocalMap` 에 접근해서 여기에 저장된 값을 바로 꺼내어 쓸수 있기 때문이다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/b599c741-2c99-43de-97f2-1f65128ab3d4)

- `ThreadLocalMap` 은 항상 새롭게 생성되어 스레드 스택에 저장되기 때문에 근본적으로 스레드간 데이터가 공유 될 수 없고, 따라서 동시성 문제가 발생하지 않는다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/0e184adf-4ed6-4f16-b795-b1c047a00d84)

# 💡 ThreadLocal 작동원리

- `ThreadLocal` 은 `Thread` 와 `ThreadLocalMap` 을 연결하여 스레드 전용 저장소를 구현하고 있는데, 이것이 가능한 이유는 바로 `Thread.currentThread()` 를 참조할 수 있기 때문이다.
- `Thread.currentThread()` 는 현재 실행중인 스레드의 객체를 참조하는 것으로서 CPU 는 오직 하나의 스레드만 할당받아 처리하기 때문에 `ThreadLocal` 에서 `Thread.currentThread()` 를 참조하면 지금 실행 중인 스레드의 로컬 변수를 저장하거나 참조할 수 있게 된다.
- `ThreadLocal` 에서 현재 스레드를 참조할 수 있는 방법이 없다면 값을 저장하거나 요청하는 스레드를 식별할 수 없기 때문에 `Thread.currentThread()` 는 `ThreadLocal` 의 중요한 데이터 식별 기준이 된다.

## 데이터 저장

![image](https://github.com/shin-je-woo/TIL/assets/39439576/a8d5e3a5-aa90-4e3a-b69b-ff4e386092c4)

▶️ ThreadLocal.set() 메서드
```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        map.set(this, value);
    } else {
        createMap(t, value);
    }
}
```

## 데이터 조회

![image](https://github.com/shin-je-woo/TIL/assets/39439576/471c26c0-5ca4-473a-b933-d0d6b4960c72)

▶️ ThreadLocal.get() 메서드
```java
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
```

# 💡 ThreadLocal 사용 시 주의사항

- ThreadLocal 에 저장된 값은 스레드마다 독립적으로 저장되기 때문에 저장된 데이터를 삭제하지 않아도 메모리를 점유하는 것 외에 문제가 되지는 않는다.
- 그러나 **스레드 풀**을 사용해서 스레드를 운용한다면 반드시 ThreadLocal 에 저장된 값을 삭제해 주어야 한다.
- 스레드풀은 스레드를 재사용하기 때문에 현재 스레드가 이전의 스레드를 재사용한 것이라면 이전의 스레드에서 삭제하지 않았던 데이터를 참조할 수 있기 때문에 문제가 될 수 있다.
