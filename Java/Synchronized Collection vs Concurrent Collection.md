# 💡 Synchronized Collection
- 동기화된 컬렉션은 대표적으로 다음과 같은 클래스가 있다.
  - Vector
  - Hashtable
  - Collections.synchronizedXXX
- 이 클래스 모두 public으로 선언된 메소드에 `synchronized` 키워드를 사용하여 내부의 값을 한 스레드만 사용할 수 있도록 제어하면서 동시성을 보장하고 있다.

### 🟩 Vector
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

### 🟩 Hashtable
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

### 🟩 Collections.synchronizedXXX
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

### 🟩 CopyOnWriteArrayList
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

### 🟩 CopyOnWriteArraySet
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
- add() 메소드도 CopyOnWriteArrayList의 메소드를 차용한다

```java
public boolean addIfAbsent(E e) {
    Object[] snapshot = getArray();
    return indexOfRange(e, snapshot, 0, snapshot.length) < 0
        && addIfAbsent(e, snapshot);
}

private boolean addIfAbsent(E e, Object[] snapshot) {
        synchronized (lock) {
            Object[] current = getArray();
            int len = current.length;
            if (snapshot != current) {
                // Optimize for lost race to another addXXX operation
                int common = Math.min(snapshot.length, len);
                for (int i = 0; i < common; i++)
                    if (current[i] != snapshot[i]
                        && Objects.equals(e, current[i]))
                        return false;
                if (indexOfRange(e, current, common, len) >= 0)
                        return false;
            }
            Object[] newElements = Arrays.copyOf(current, len + 1);
            newElements[len] = e;
            setArray(newElements);
            return true;
        }
  }
```
- addIfAbsent() 메소드를 살펴 보면, 쓰기 동작 시 잠금을 걸고 배열 복사를 수행한다.
- 따라서 CopyOnWriteArraySet도 CopyOnWriteArrayList과 마찬가지로 쓰기 작업은 많이 하지 않는 것이 좋다.

### 🟩 ConcurrentHashMap
- 동기화된 HashMap를 생성하는 방법은 아래 두 가지가 있다.
  - Collections.synchronizedMap(new HashMap<>())
  - ConcurrentHashMap
- ConcurrentHashMap은 HashMap과 동일하게 Hash를 기반으로 하는 Map이다.
- synchronizedMap에 비해 더욱 효율적인 방법으로 동시성을 보장한다.
- 자바 8 이전에는 ReentrantLock을 상속 받는 Segment를 이용해, 영역을 구분함으로써 영역 별로 잠금을 하는 방식이었다.
- 자바 8 이후부터는 각 테이블 버킷을 독립적으로 잠그는 방식을 사용한다.
- 빈 버킷에 노드를 삽입할 경우에는 잠금 대신 CAS 알고리즘을 사용하고, 그 외의 변경은 각 버킷의 첫 번째 노드를 기준으로 부분적인 잠금(synchronized block)을 획득하여 스레드 경합을 최소화하며 동시성을 보장한다.
- ConcurrentHashMap에 새로운 노드를 삽입하는 putVal() 메소드의 코드를 보며, 동시성을 어떻게 보장하는지 확인해 보자. 참고로 다음 예제 코드는 자바 11 기준이다.
- putVal() 메소드는 크게 아래와 같이 두 가지 경우(분기는 총 네 부분)로 나눌 수 있다.
  - 빈 해시 버킷에 노드를 삽입하는 경우
  - 해시 버킷에 이미 노드가 존재하는 경우
 
```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
        if (key == null || value == null) throw new NullPointerException();
        int hash = spread(key.hashCode());
        int binCount = 0;
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh; K fk; V fv;
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();
                        // (1)
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                                // (2)
                if (casTabAt(tab, i, null, new Node<K,V>(hash, key, value)))
                    break;
            }
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
            else if (onlyIfAbsent // check first node without acquiring lock
                     && fh == hash
                     && ((fk = f.key) == key || (fk != null && key.equals(fk)))
                     && (fv = f.val) != null)
                return fv;
                        // (3)
            else {
                V oldVal = null;
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        if (fh >= 0) {
                            binCount = 1;
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                                                // (4)
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                Node<K,V> pred = e;
                                                                // (5)
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key, value);
                                    break;
                                }
                            }
                        }
                                                // (6)
                        else if (f instanceof TreeBin) {
                            Node<K,V> p;
                            binCount = 2;
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                           value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                        else if (f instanceof ReservationNode)
                            throw new IllegalStateException("Recursive update");
                    }
                }
                                ...
            }
        }
                ...
    }
```

#### 빈 해시 버킷에 노드를 삽입하는 경우
(1) 새로운 노드를 삽입하기 위해, 해당 버킷 값(tabAt())을 가져와 비어 있는지 확인한다.
```java
static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
        return (Node<K,V>)U.getObjectAcquire(tab, ((long)i << ASHIFT) + ABASE);
}
```
(2) 노드에 담겨 있는 volatile 변수에 접근하여 기존 값(null)과 비교한 후, 같으면 새로운 노드를 저장한다. 같지 않으면, for문으로 다시 돌아간다. 이 방식은 CAS 알고리즘이다.
```java
static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i, Node<K,V> c, Node<K,V> v) {
        return U.compareAndSetObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
}
```
- CAS 알고리즘을 사용하여 원자성과 가시성 문제를 해결하여 동시성을 보장한다.

#### 해시 버킷에 이미 노드가 존재하는 경우
(3) 이미 노드가 존재하는 경우에는 synchronized block을 사용하여 하나의 스레드만 접근하도록 제어한다. 이때, 비어있지 않은 Node<K, V> 타입의 해시 버킷에 잠금을 걸기 때문에 동일한 버킷에 접근하는 스레드는 Blocking 상태가 된다.  
(4) 새로운 노드로 교체한다.  
(5) 해시 충돌이 일어난 경우 Separate Chain에 추가한다.  
(6) 해시 충돌이 일어난 경우 트리에 추가한다.  

# 💡 정리
### Vector, HashTable, Collections.SynchronziedXXX의 문제점은?
- Vector, HashTable, SynchronziedXxx 클래스는 `synchronized` 메소드 또는 블록을 사용하며, 하나의 잠금 객체를 공유한다.
- 따라서, 컬렉션에 하나의 스레드가 잠금을 획득하는 경우 다른 스레드들은 모든 메소드를 사용하지 못하고 Blocking 상태가 된다.
- 이는 애플리케이션 성능 저하의 원인이 될 수 있다.

### SynchronizedList와 CopyOnArrayList의 차이점은?
- SynchronizedList는 읽기와 쓰기 동작시 인스턴스 자체에 잠금이 걸린다.
- 그러나 CopyOnArrayList는 쓰기 동작시 해당 블록에 잠금을 걸고 원본 배열에 있는 요소를 복사하여 새로운 임시 배열을 만들고, 이 임시 배열에 쓰기 동작을 수행한 후 원본 배열을 갱신한다.
- 덕분에 읽기 동작은 잠금이 걸리지 않아 SynchronizedList보다 읽기 성능이 좋다.
- 그러나 쓰기 동작은 비용이 높은 배열 복사 작업을 하기 때문에 SynchronizedList보다 쓰기 성능이 떨어진다.
- 따라서, 변경 작업보다 읽는 작업이 많을 경우 CopyOnArrayList를 사용하는 것이 더 효과적이다.

### SynchronizedMap과 ConcurrentHashMap의 차이점은?
- SynchronziedMap은 읽기와 쓰기 동작 시 인스턴스 자체에 잠금이 걸린다.
- 그러나 ConcurrentHashMap은 각 테이블 버킷을 독립적으로 잠그는 방식을 사용한다.
- 만약, 빈 버킷에 노드를 삽입할 경우 잠금(Lock) 대신 CAS 알고리즘을 사용하고, 그 외의 변경은 접근한 버킷에만 잠금이 걸려 스레드 경합을 최소화하며 동시성을 보장해준다.
