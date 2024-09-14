# 💡 Garbage Collection(GC) 이란?

- GC는 더이상 사용되지 않는 객체를 식별하여 자동으로 제거하는 프로세스를 말한다.
- GC는 언제 실행될지 알 수 없으며, GC가 실행되면 GC를 실행하는 쓰레드 외에는 모두 멈추기 떄문에 주의가 필요하다. (Stop The World 라고 한다.)

# 💡 가비지 컬렉션 대상

- 객체에 레퍼런스가 존재하면 Reachable로 구분하고, 존재하지 않으면 Unreachable로 구분한다.
- JVM에서 객체들은 Heap 영역에 생성되고, Stack과 Method영역에서 Heap에 있는 객체의 주소를 참조하게 된다.
- 만약 Heap에 있는 객체를 참조하던 참조 변수가 삭제되게 되면 이 객체는 더이상 참조되지 않는 Unreachable 상태가 된다.
- GC는 이렇게 Unreachable 상태의 객체를 식별하고 삭제하게 된다.

![image](https://github.com/user-attachments/assets/8b09b817-42dc-4128-bc1d-ad643a57a495)

# 💡 Heap 영역

![image](https://github.com/user-attachments/assets/8e3b7bdb-887c-4bb2-a4cd-655026a65b28)

- JVM의 힙(Heap)영역은 동적으로 생성되는 객체들이 저장되는 공간이며, GC의 주요 대상이 된다.
- JVM은 Heap영역을 Young과 Old영역으로 구분하여 메모리를 관리한다.

### Young 영역(Young Generation)

- 새롭게 생성된 객체가 할당되는 영역
- 대부분의 객체가 금방 Unreachable 되기 떄문에 Young영역에 생성되었다가 사라진다.
- Young 영역에 대한 GC를 Minor GC라고 한다.

### Old 영역(Old Generation)

- Young영역에서 Reachable 상태를 오래 유지한 객체들이 복사되는 공간
- 보통 Young영역보다 크게 할당되며, GC가 자주 발생하지는 않는다.
- Old 영역에 대한 GC를 Major GC 또는 Full GC라고 부른다.

![image](https://github.com/user-attachments/assets/f10eb829-179b-4df7-9723-27a9e761d17d)

### Eden

- 객체가 생성되면 맨 처음 할당되는 영역
- Minor GC가 발생하면 Reachable한 객체는 Survivor 영역으로 보내지고, 나머지는 sweep(삭제)된다.

### Survivor 0 / Survivor 1

- 최소 한번 이상의 GC가 발생해서 살아남은 객체가 존재하는 영역
- Survivor0 또는 Survivor1 둘 중 하나는 꼭 비어 있어야 한다.

# 💡 Minor GC 과정

- Young Generation은 짧게 살아남은 객체들이 존재하는 공간이다.
- Young Generation의 공간은 Old Generation의 공간보다 상대적으로 작기 때문에 메모리 상의 객체를 찾아 제거하는데 상대적으로 적은 시간이 걸린다.
- 때문에 Young Generation에서 발생하는 GC를 Minor GC라고 부른다.

1) 처음 생성된 객체는 Young Generation 영역의 일부인 Eden 영역에 위치
2) 객체가 계속 생성되어 Eden영역이 꽉차게 되면 Minor GC 발생
3) Mark 동작을 통해 Reachable 객체 식별
4) Eden 영역에서 살아남은 객체는 Survivor 영역으로 이동
5) 참조되지 않는 객체 (Unreachable) 의 메모리 해제(sweep)
6) 살아남은 객체들의 age 1씩 증가
7) 이 과정을 반복

# 💡 Major GC 과정

- Old Generation은 길게 오래 남은 객체들이 존재하는 공간이다.
- Minor GC 과정에서 age값이 임계치를 넘은 객체들이 Old Generation으로 이동된다.
- Major GC는 Young Generation의 객체들이 계속 Promotion 되어 Old Generation의 메모리가 부족해지면 발생한다.

1) 객체의 age가 임계값에 도달하게 되면 Old Generation으로 이동한다. 이 과정을 Promotion이라고 한다.
2) Promotion이 계속 발생하여 Old Generation의 공간이 부족해지면 Major GC가 발생한다.

- Major GC는 Old Generation이 가득 차면 발생하는 단순한 방식이다.
- Old 영역에 할당된 메모리가 허용치를 넘게 되면, Old 영역에 있는 모든 객체들을 검사하여 참조되지 않는 객체들을 한꺼번에 삭제하는 Major GC가 실행되게 된다.
- 하지만 Old Generation은 Young Generation에 비해 상대적으로 큰 공간을 가지고 있어, 이 공간에서 메모리 상의 객체 제거에 많은 시간이 걸리게 된다.
- Major GC에서는 Stop The World를 조심해야 한다.
- Major GC가 일어나면 Thread가 멈추고 Mark and Sweep 작업을 해야 해서 CPU에 부하를 주기 때문에 멈추거나 버벅이는 현상이 일어나기 때문이다.
