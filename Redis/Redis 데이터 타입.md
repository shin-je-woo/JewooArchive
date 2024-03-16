# 💡 Strings
- Redis String 유형은 Redis 키와 연결할 수 있는 가장 간단한 유형이다.
- Memcached에서의 유일한 데이터 타입이자, Redis에서도 자연스러운 값이다.
- 기본적으로 GET, SET을 이용해 값을 입력하고 조회할 수 있다.
- 문자열, 숫자, serialized object(JSON string) 등을 저장할 수 있다.

```bash
> set mykey somevalue
OK
> get mykey
"somevalue"
```

- SET은 키가 이미 존재하는 경우 이미 저장된 기존 값을 대체한다.
- SET 명령에는 추가 인수로 제공되는 여러 옵션이 있다.
- 예를 들어, 키가 이미 존재하는 경우 실패하도록 요청하거나, 키가 이미 존재하는 경우에만 성공하도록 요청할 수 있다. (NX/XX)

```bash
> set mykey newval nx # 키가 존재하지 않을 경우에만 set
(nil)
> set mykey newval xx # 키가 이미 존재하는 경우에만 set
OK
```

- 비록 문자열들이 레디스의 기본값일지라도, 그것들로 수행할 수 있는 흥미로운 연산들이 있다.
- 예를 들어, atomic increment이다.

```bash
> set counter 100
OK
> incr counter
(integer) 101
> incr counter
(integer) 102
> incrby counter 50
(integer) 152
```

- INCR 명령은 1)문자열 값을 정수로 구문 분석하고, 2)1씩 증분한 다음, 3)마지막으로 얻은 값을 새 값으로 설정한다.
- 비슷한 명령어로는 INCRBY, DECR and DECRBY가 있다.
- INCR 명령은 atomic하다. 즉, 여러 클라이언트가 동시에 같은 키에 INCR 명령어를 입력한다고 해서 경쟁 상태에 들어가지 않는다.
- 한번에 여러 값을 설정하고 가져오는 기능도 있다. MSET과 MGET을 이용하면 된다.

```bash
> mset a 10 b 20 c 30
OK
> mget a b c
1) "10"
2) "20"
3) "30"
```

### Altering and querying the key space
- 특정 타입에 속하지 않는 명령어도 있다.
- EXISTS 명령은 지정된 키가 데이터베이스에 있는지 여부에 대한 결과값으로 1 또는 0을 반환한다.
- DEL 명령은 값과 상관없이 키 및 관련 값을 삭제한다.

```bash
> set mykey hello
OK
> exists mykey
(integer) 1
> del mykey
(integer) 1
> exists mykey
(integer) 0
```

- DEL 명령어를 통해 key가 제거되었는지의 여부도 확인할 수 있다.(1: 제거됨, 0:제거되지않음;값이 원래 없었음)
- TYPE 명령어는 키의 타입을 알려준다.

```bash
> set mykey x
OK
> type mykey
string
> del mykey
(integer) 1
> type mykey
none
```

### Key expiration
- Key expiration을 통해 키의 유효시간을 설정할 수 있다. 유효 시간(TTL, Time To Live)이 지나면 키는 자동으로 파기된다.
- 이 기능의 특징은 다음과 같다.
  - 초 또는 밀리초 단위로 설정할 수 있다.
  - 만료시간 단위는 항상 1 밀리초이다.
  - expiration에 대한 정보는 항상 디스크에 복제되고 저장된다. 만료시간을 특정 날짜로 저장하기때문에 레디스 서버가 중지되더라도 시간은 계속 흐른다.

```bash
> set key some-value
OK
> expire key 5 # 5초로 TTL을 설정한다.
(integer) 1
> get key (immediately)
"some-value"
> get key (after some time)
(nil)
```

- EXPIRE 이외에도 키 생성과 함께 만료시간을 설정할 수도 있다.

```bash
> set key 100 ex 10 # 10초로 TTL을 설정한다.
OK
> ttl key
(integer) 9
```

# 💡 Lists
- 레디스의 list는 Linked list로 구현되어 있다.
  - List 안에 수백만 개의 요소가 있더라도 List의 head 부분이나 tail 부분에 새 요소를 추가하는 작업이 일정한 시간에 수행된다는 것을 의미한다.
  - 10개의 요소가 있는 List의 head에 요소를 추가하는 속도는 천만 개의 요소가 있는 List의 head에 요소를 추가하는 속도와 같다.
  - push/pop에 최적화 O(1) 되어 있다.
- 즉, Stack과 Queue를 구현하는 데 주로 사용된다.

### First steps with Redis Lists
- LPUSH 명령어는 리스트의 왼쪽(맨앞)에, RPUSH 명령어는 리스트의 오른쪽(맨뒤)에 요소를 삽입한다.
- LRANGE 명령어는 리스트의 요소를 추출한다.

```bash
> rpush mylist A
(integer) 1
> rpush mylist B
(integer) 2
> lpush mylist first
(integer) 3
> lrange mylist 0 -1 # 첫번째(0)부터 마지막번째 요소(-1)까지 출력하라.
1) "first"
2) "A"
3) "B"
```

- LRANGE 명령어는 2개의 인덱스를 갖고 있는데 시작 인덱스와 끝 인덱스를 의미한다. -1은 마지막, -2는 끝에서 2번째를 의미한다.
- 한번에 여러 요소를 삽입할 수도 있다.

```bash
> rpush mylist 1 2 3 4 5 "foo bar"
(integer) 9
> lrange mylist 0 -1
1) "first"
2) "A"
3) "B"
4) "1"
5) "2"
6) "3"
7) "4"
8) "5"
9) "foo bar"
```

- RPOP, LPOP은 요소를 pop하는 기능이다. 즉, 요소를 출력함과 동시 제거하는 기능이다.

```bash
> rpush mylist a b c
(integer) 3
> rpop mylist
"c"
> rpop mylist
"b"
> rpop mylist
"a"
```

### Common use cases for lists
- List 자료형은 아래 2가지 케이스에서 유용하게 사용할 수 있다.
  - 사용자가 SNS에 가장 최근에 올린 포스트를 기억해야 할 때
  - 생산자 - 소비자 패턴(생산자는 작업을 넣고, 소비자는 작업을 처리하는 구조)에서 유용하게 사용할 수 있다. 레디스는 이를 위한 List 명령어를 제공한다.
- 트위터에서 가장 최근 트윗을 가져올 때 레디스를 이용한다. [참고](https://www.infoq.com/presentations/Real-Time-Delivery-Twitter/)
- 사용자가 사진을 업로드하면 photo_id를 LPUSH를 통해 리스트에 넣고, 여러 사용자들이 메인페이지에 접속할 때마다 LRANGE 0 9 명령어를 통해 최근 10개의 트윗을 가져온다.

### Capped lists
- SNS 업데이트, 로그를 포함해 어떤 데이터든지 최신 데이터를 저장하기 위해 List를 사용한다.
- 레디스의 LTRIM 명령어를 사용하면 최신 N개의 항목만 기억하고 가장 오래된 항목은 모두 폐기하면서 목록을 제한된 컬렉션으로 사용할 수 있다.
- LTRIM 명령어는 LRANGE 명령어와 비슷하나, 특정 범위의 요소를 보여주는 대신 특정 범위의 요소만을 제외한 모든 요소를 제거한다는 특징이 있다.

```bash
> rpush mylist 1 2 3 4 5
(integer) 5
> ltrim mylist 0 2 # 0번쨰 인덱스부터 2번째 인덱스까지를 제외한 모든 요소를 제거한다.
OK
> lrange mylist 0 -1
1) "1"
2) "2"
3) "3"
```

- 이를 통해 매우 간단하지만 유용한 패턴이 가능해진다.
- 즉, List push 작업 + List trim 작업을 함께 수행하여 새 요소를 추가하고 특정 범위를 넘어가는 요소를 제거하는 방식이다.

```bash
LPUSH mylist <some element>
LTRIM mylist 0 999
```

- 위의 조합은 새 요소를 추가하고 1000개의 최신 요소만 목록으로 가져온다.
- 여기에 LRANGE를 사용하면 오래된 데이터를 기억할 필요 없이 상위 항목의 데이터만을 가져올 수 있다.
- LRANGE 명령어의 시간복잡도가 O(N)이기에 제한된 데이터만을 가져오는 것은 성능상 매우 중요하다. (LinkedList의 시간복잡도 참고.)

# 💡 Sets
- 레디스의 Set 자료형은 정렬되지 않은 문자열 집합을 의미한다.
- SADD 명령어는 set에 새로운 요소를 삽입한다.
- 또한, 여러 집합 간의 교집합, 합집합 또는 차집합 등과 같은 집합에 대한 다양한 연산을 수행할 수도 있다. (intersection, union, difference)

```bash
> sadd myset 1 2 3
(integer) 3
> smembers myset
1) "1"
2) "2"
3) "3"
```

- SISMEMBER 명령어를 통해 이미 요소가 존재하는지 확인할 수 있다.

```bash
> sismember myset 3 # Set에 값이 존재하는 경우
(integer) 1
> sismember myset 30 # Set에 값이 존재하지 않는 경우
(integer) 0
```

- 집합은 객체 간의 관계를 표현하는 데 유용하다. 예를 들어, 태그를 구현하기 위해 Set을 쉽게 사용할 수 있다.
- 이 문제를 모델링하는 간단한 방법은 태그를 지정할 모든 개체에 대해 집합을 갖는 것이다. 집합에는 개체와 연결된 태그의 ID가 포함된다.
- 하나의 예로 뉴스 기사에 태그를 다는 상황을 들 수 있다.
- 기사 ID 1000에 태그 1, 2, 5 및 77이 지정된 경우 집합은 다음 태그 ID를 뉴스 항목과 연결할 수 있다.

```bash
> sadd news:1000:tags 1 2 5 77
(integer) 4
```

- SMEMBERS 명령어로 뉴스 기사에 달린 모든 태그를 확인할 수 있다.

```bash
> smembers news:1000:tags
1) "1"
2) "2"
3) "5"
4) "77"
```

- SCARD 명령어로 cardinality를 확인할 수 있다.

```bash
> scard news:1000:tags
(integer) 4
```

- 또는, 반대로 주어진 태그에 뉴스기사를 연결할 수도 있다.

```bash
> sadd tag:1:news 1000
(integer) 1
> sadd tag:2:news 1000
(integer) 1
> sadd tag:5:news 1000
(integer) 1
> sadd tag:77:news 1000
(integer) 1
```

- SINTER 명령어로 각 태그에 달린 뉴스기사의 교집합을 찾을 수도 있다.

```bash
> sinter tag:1:news tag:2:news tag:5:news tag:7:news
1) "1000"
```

- SINTER 외에도 SDIFF, SUNION 과 같은 집합 관련 명령어가 있다.

# 💡 Hashes
- Redis 해시는 key-value 쌍의 컬렉션으로 구성된 레코드 유형이다.
- 다양한 속성을 갖는 객체의 데이터를 저장할 때 유용하다.

```bash
> hset user:1000 username shinjewoo birthyear 1994 visit 33
(integer) 3
> hget user:1000 username
"shinjewoo"
> hmget user:1000 username birthyear visit
1) "shinjewoo"
2) "1994"
3) "33"
> hgetall user:1000
1) "username"
2) "shinjewoo"
3) "birthyear"
4) "1994"
5) "visit"
6) "33"
```

- 위 예제와 같이 HMSET, HMGET과 같은 Multi operation 기능을 제공한다.
- 또한, HINCRBY와 같이 단일 필드에서도 작업을 수행할 수 있는 명령도 있다.

```bash
> hincrby user:1000 visit 10
(integer) 43
> hincrby user:1000 visit 10
(integer) 53
```

# 💡 Sorted sets
- Unique string을 연관된 score를 통해 정렬된 집합이다. (Set의 기능 + 추가로 score 속성 저장)
- 내부적으로 Skip List + Hash Table로 이루어져 있고, score 값에 따라 정렬을 유지한다.
- Sorted set은 스코어를 기준으로 정렬된다. 만약 스코어가 같다면 사전순서대로 정렬된다.

```bash
> zadd hackers 1940 "Alan Kay"
(integer) 1
> zadd hackers 1957 "Sophie Wilson"
(integer) 1
> zadd hackers 1953 "Richard Stallman"
(integer) 1
> zadd hackers 1949 "Anita Borg"
(integer) 1
> zadd hackers 1965 "Yukihiro Matsumoto"
(integer) 1
> zadd hackers 1914 "Hedy Lamarr"
(integer) 1
> zadd hackers 1916 "Claude Shannon"
(integer) 1
> zadd hackers 1969 "Linus Torvalds"
(integer) 1
> zadd hackers 1912 "Alan Turing"
(integer) 1
```

- ZADD 명령어는 SADD와 유사하지만 추가할 요소 앞에 스코어를 사용한다. 또한, ZADD는 여러 개의 Score-Key 쌍을 자유롭게 지정할 수 있다.
- ZRANGE 명령어로 정렬된 값을 확인할 수 있다.

```bash
> zrange hackers 0 -1
1) "Alan Turing"
2) "Hedy Lamarr"
3) "Claude Shannon"
4) "Alan Kay"
5) "Anita Borg"
6) "Richard Stallman"
7) "Sophie Wilson"
8) "Yukihiro Matsumoto"
9) "Linus Torvalds"
```

- 만약 역순으로 정렬하고 싶다면 ZREVRANGE 명령어를 사용하면 된다.

```bash
> zrevrange hackers 0 -1
1) "Linus Torvalds"
2) "Yukihiro Matsumoto"
3) "Sophie Wilson"
4) "Richard Stallman"
5) "Anita Borg"
6) "Alan Kay"
7) "Claude Shannon"
8) "Hedy Lamarr"
9) "Alan Turing"
```

- WITSCORES 명령어를 사용해 스코어를 반환할 수도 있다.

```bash
> zrange hackers 0 -1 withscores
1) "Alan Turing"
2) "1912"
3) "Hedy Lamarr"
4) "1914"
5) "Claude Shannon"
6) "1916"
7) "Alan Kay"
8) "1940"
9) "Anita Borg"
10) "1949"
11) "Richard Stallman"
12) "1953"
13) "Sophie Wilson"
14) "1957"
15) "Yukihiro Matsumoto"
16) "1965"
17) "Linus Torvalds"
18) "1969"
```

### Operating on ranges
- Sorted set은 범위 내에서 동작할 수 있다.
- 만약 1950년대 이전에 태어난 모든 사람을 조회한다면 ZRANGEBYSCORE 명령어를 사용하면 된다.

```bash
> zrangebyscore hackers -inf 1950 # score가 1950 이하인 모든 Key 조회
1) "Alan Turing"
2) "Hedy Lamarr"
3) "Claude Shannon"
4) "Alan Kay"
5) "Anita Borg"
```

- 요소의 범위를 제거할 수도 있다.
- ZREMRANGEBYSCORE 명령어로 1940년에서 1960년 사이에 태어난 모든 사람을 분류된 집합에서 제거해보자.

```bash
> zremrangebyscore hackers 1940 1960
(integer) 4

> zrange hackers 0 -1
1) "Alan Turing"
2) "Hedy Lamarr"
3) "Claude Shannon"
4) "Yukihiro Matsumoto"
5) "Linus Torvalds"
```

- 또 다른 매우 유용한 명령어는 get-rank이다.
- 정렬된 집합에서 요소의 위치가 무엇인지 물어볼 수 있다.

```bash
> zrank hackers "Hedy Lamarr"
(integer) 1
```

- ZREVRANK 명령어를 통해 뒤에서 몇 등인지 알 수도 있다.

```bash
> ZREVRANK hackers "Alan Turing"
(integer) 4
```
