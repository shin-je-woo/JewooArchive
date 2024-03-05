# 💡 Synchronized Collection
- 동기화된 컬렉션은 대표적으로 다음과 같은 클래스가 있다.
  - Vector
  - Hashtable
  - Collections.synchronizedXXX
- 이 클래스 모두 public으로 선언된 메소드에 `synchronized` 키워드를 사용하여 내부의 값을 한 스레드만 사용할 수 있도록 제어하면서 동시성을 보장하고 있다.

### Vector
```java
public class Vector<E> extends AbstractList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
    ...
    public synchronized boolean add(E e) {
                modCount++;
                add(e, elementData, elementCount);
                return true;
    }
    ...
}
```
- Vector 클래스에서 요소를 추가하는 add() 메소드를 살펴 보면 `synchronized` 키워드가 보인다.
- 즉, 내부적으로 Vector에서 요소 삽입 연산이 진행될 때 동기화가 보장된다.

### Hashtable
```java
ublic class Hashtable<K,V> extends Dictionary<K,V>
    implements Map<K,V>, Cloneable, java.io.Serializable {
        ...
        public synchronized boolean contains(Object value) {
        if (value == null) {
            throw new NullPointerException();
        }

        Entry<?,?> tab[] = table;
        for (int i = tab.length ; i-- > 0 ;) {
            for (Entry<?,?> e = tab[i] ; e != null ; e = e.next) {
                if (e.value.equals(value)) {
                    return true;
                }
            }
        }
        return false;
    }
        ...
}
```
- Hashtable 클래스에서 동일한 value가 존재하는지 확인하는 contains() 메소드를 살펴보면 Vector 클래스와 동일하게 `synchronized` 키워드가 사용된 것을 확인할 수 있다.

### Collections.synchronizedXXX
- Collections.synchronizedList() 메소드를 사용해 생성되는 SynchronizedList 클래스를 살펴 보자.

```java
static class SynchronizedList<E> extends SynchronizedCollection<E>
        implements List<E> {

        final Object mutex;

        ...
        public E get(int index) {
                synchronized (mutex) {return list.get(index);}
        }

        public E set(int index, E element) {
                synchronized (mutex) {return list.set(index, element);}
        }

        public void add(int index, E element) {
                synchronized (mutex) {list.add(index, element);}
        }

        public E remove(int index) {
                synchronized (mutex) {return list.remove(index);}
        }
        ...
}
```
- SynchronizedList 클래스의 메소드를 살펴 보면 모두 `synchronized` 키워드가 사용된 것을 알 수 있다.
- 다만, `synchronized` 블록을 사용하여 뮤텍스를 통해 동기화를 구현하였다.
- 모든 메소드는 mutex 객체를 공유하므로 하나의 스레드가 `synchronized` 블록에 들어간 순간 다른 메소드의 `synchronized` 블록이 다 lock이 걸린다.

## 📌 Synchronized Collection의 문제점
- 멀티 스레드 환경에서 Synchronized Collection 을 사용해야 하는 경우도 있지만, 웬만해서는 다른 동기화 방식을 사용하는 것이 좋다.
- 이유는 크게 두 가지가 있다.

### 1. 여러 연산을 묶어 단일 연산처럼 사용하는 경우
- Synchronized Collection 클래스는 멀티 스레드 환경에서 동시성을 보장한다.
- 하지만 여러 연산을 묶어 하나의 단일 연산처럼 활용해야 하는 경우 문제가 발생한다.
- 이러한 동기화된 컬렉션을 사용해도 올바르게 동작하지 않을 수도 있다.

```java
final List<String> list = Collections.synchronizedList(new ArrayList<String>());
final int nThreads = 2;
ExecutorService es = Executors.newFixedThreadPool(nThreads);

for (int i = 0; i < nThreads; i++) {
    es.execute(new Runnable() {

        public void run() {
            while(true) {
                try {
                    list.clear();
                    list.add("888");
                    list.remove(0);
                } catch(IndexOutOfBoundsException e) {
                    e.printStackTrace();
                }
            }
        }
    });
}
```
- 위의 코드를 실행하면 Thread A가 remove(0) 을 하는 순간에 Thread B가 clear() 를 한다면 에러가 발생한다.
- 따라서 아래와 같이 동기화 블록으로 묶어야 한다.

```java
synchronized (list) {
    list.clear();
    list.add("888");
    list.remove(0);
}
```

### 2. 성능 저하
- 공유 객체를 사용하려는 모든 메소드를 `synchronized` 메소드로 만들어 버리거나, 메소드 내부에 동일한 `synchronized` 블록을 정의하게 된다면?
- 하나의 스레드가 lock을 획득하는 순간 다른 스레드는 모든 동기화 메소드를 이용하지 못하고 blocking 상태가 된다.
- 이러한 반복된 현상은 성능 저하로 이어질 수 있다.

# 💡 Concurrent Collection
- java.util.concurrent 패키지에서 제공하는 병렬 컬렉션의 종류는 많지만, 주요한 클래스만 다룬다.

### CopyOnWriteArrayList
- 동기화된 ArrayList를 생성하는 방법은 아래 두 가지가 있다.
  - Collections.synchronizedList()
  - CopyOnWriteArrayList
- Collections.synchronizedList()는 JDK 1.2 버전에 추가되었다.
- 이 컬렉션은 모든 읽기와 쓰기 동작에 대해 동기화가 되어 있으므로 비효율적인 설계라고 볼 수 있다.
- 따라서 이를 보완한 CopyOnWriteArrayList가 등장하게 되었다.

#### 읽기 동작
- SynchronizedList는 읽기와 쓰기 동작 시 인스턴스 자체에 잠금이 걸린다.
- 그러나 CopyOnWriteArrayList는 모든 쓰기 동작 시 원본 배열에 있는 요소를 복사하여 새로운 임시 배열을 만들고, 이 임시 배열에 쓰기 동작을 수행한 후 원본 배열을 갱신한다.
- 덕분에 읽기 동작은 잠금이 걸리지 않으므로 SynchronizedList보다 성능이 좋다.

```java
public class CopyOnWriteArrayList<E> implements List<E>, RandomAccess, Cloneable, java.io.Serializable {

    final transient Object lock = new Object();
    private transient volatile Object[] array;

        ...
        public E get(int index) {
        return elementAt(getArray(), index);
    }
        ...
}
```
- 위는 get() 메소드인데, 읽기 동작 시 synchronized가 없으므로 잠금이 걸리지 않는다.

#### 쓰기 동작
- CopyOnWriteArrayList는 쓰기 동작 수행 시 명시적 락을 사용한다.
- 결국 두 종류의 컬렉션 모두 이 동작에서 잠금이 걸린다.
- 이때 CopyOnWriteArrayList는 비용이 상대적으로 높은 배열 복사 작업을 하기 때문에 **상당한 쓰기 작업이 수행되면 성능 이슈가 발생할 수 있다**.

```java
public class CopyOnWriteArrayList<E> implements List<E>, RandomAccess, Cloneable, java.io.Serializable {

    final transient Object lock = new Object();
    private transient volatile Object[] array;

        public void add(int index, E element) {
        synchronized (lock) {
            ...
            int numMoved = len - index;
            if (numMoved == 0)
                newElements = Arrays.copyOf(es, len + 1);
            else {
                newElements = new Object[len + 1];
                System.arraycopy(es, 0, newElements, 0, index);
                System.arraycopy(es, index, newElements, index + 1,
                                 numMoved);
            }
                        ...
        }
    }
}
```
- 위는 add() 메소드인데, synchronized 블럭을 통해 잠금을 걸고 배열을 복사 작업을 수행한다.

#### 순회 동작 - CopyOnWriteArrayList.iterator()
- CopyOnWriteArrayList에서 iterator를 뽑아내는 시점의 컬렉션 데이터를 기준으로 반복한다.
- 반복하는 동안 컬렉션에 데이터가 추가되거나 삭제되는 내용은 반복문과 상관 없는 복사본을 대상으로 반영하기 때문에 동시 사용성에 문제가 없다.

### CopyOnWriteArraySet
- 동기화된 Set을 생성하는 방법은 아래 두 가지가 있다.
  - Collections.synchronizedSet()
  - CopyOnWriteArraySet
- 메소드 네이밍을 보면 알 수 있듯이, CopyOnWriteArrayList와 동작 방식이 자료 구조 특징을 제외하면 거의 동일하다.

#### 읽기 동작
```java
public class CopyOnWriteArraySet<E> extends AbstractSet<E> implements java.io.Serializable {

    private final CopyOnWriteArrayList<E> al;

        public boolean contains(Object o) {
        return al.contains(o);
    }
}
```
- 위는 contains() 메소드인데, CopyOnWriteArraySet은 내부적으로 CopyOnWriteArrayList을 정의하고, CopyOnWriteArrayList의 메소드를 사용하는 것을 알 수 있다.

```java
public boolean contains(Object o) {
        return indexOf(o) >= 0;
}

public int indexOf(Object o) {
        Object[] es = getArray();
        return indexOfRange(o, es, 0, es.length);
}

    private static int indexOfRange(Object o, Object[] es, int from, int to) {
        if (o == null) {
            for (int i = from; i < to; i++)
                if (es[i] == null)
                    return i;
        } else {
            for (int i = from; i < to; i++)
                if (o.equals(es[i]))
                    return i;
        }
        return -1;
}
```
- CopyOnWriteArrayList의 contains() 메소드를 살펴보면, 잠금이 걸리지 않는 것을 알 수 있다.

#### 쓰기 동작
```java
public class CopyOnWriteArraySet<E> extends AbstractSet<E> implements java.io.Serializable {

        private final CopyOnWriteArrayList<E> al;

        public boolean add(E e) {
        return al.addIfAbsent(e);
    }
}
```
