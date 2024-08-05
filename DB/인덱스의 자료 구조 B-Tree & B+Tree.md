# 💡 B-Tree 란?

![image](https://github.com/user-attachments/assets/3c437be6-e606-46db-952d-cd7543d0fd09)

B-Tree는 이진 탐색 트리(binary search tree, BST)의 일반화된 형태이다. 
이진 탐색 트리가 자식 노드가 최대 2개인 트리라면, B-Tree는 자식 노드가 N개인 트리이다. 즉, 자식 노드의 개수를 늘리고 트리의 전체 높이를 줄여서 빠른 탐색 속도를 얻을 수 있다. 최대 M개의 자식을 가질 수 있는 B-Tree를 M차 B-Tree라고 한다.

B-Tree의 중요한 특징으로는 **리프 노드(leaf node)가 모두 같은 레벨을 갖는다는 점**이다. 
일반적인 이진 트리는 **한쪽 방향으로만 편향될 가능성이 존재**한다. 
이런 경우 트리 자료구조의 빠른 탐색 속도라는 이점을 누리지 못하고, 링크드리스트와 같이 동작할 수 있는 위험을 갖는다. 
하지만, B-Tree는 의도적으로 리프 노드의 레벨을 동일하게 유지함으로써 위와 같은 문제를 해결한다. 
이런 특성을 갖는 트리를 **Self-Balanced Tree**라고 한다.

B-Tree는 일반적인 이진 트리와 다르게, 한 노드에 여러 데이터를 가질 수 있다. 
데이터베이스에서는 B-Tree의 노드를 **페이지 또는 블럭**이라고 부르며 **노드 내의 데이터를 키(key)라고 부른다.**
M차 B-Tree의 노드는 최대 M-1 개의 키를 가질 수 있다. 
또한, **노드 내부의 키들은 항상 오름차순으로 정렬된 상태를 유지한다.**

노드의 각 키는 실제 데이터의 물리적 위치를 가리키고 있는 **데이터 포인터(data pointer)를 가지고 있다.**
키를 기준으로 데이터를 탐색한 뒤, 일치하는 키를 발견한 경우 데이터 포인터가 가리키는 곳으로 이동해 실제 데이터를 찾을 수 있다.
데이터베이스에서는 특정 컬럼으로 인덱스를 생성할 수 있는데, 이때 컬럼의 값이 키가 되고, 테이블의 행(데이터의 물리적 위치)이 데이터 포인터가 된다.

# 💡 B+Tree 란?

![image](https://github.com/user-attachments/assets/5c47f8f6-b59e-4508-8c5f-0b7621448c86)

B+Tree는 B-Tree를 개선시킨 자료구조이다. 실제로 MySQL(InnoDB) 등 많은 DBMS에서는 B-Tree 대신 B+Tree를 사용한다.

B+Tree는 B-Tree와 다르게 **오직 리프 노드의 키만 데이터 포인터를 가지고 있다.**
B-Tree와 다르게, 리프 노드에만 데이터 포인터가 존재하므로 노드에 더 많은 키를 보관할 수 있게 된다. 
이는 곧 트리의 높이가 낮아지는 것을 의미하고, 탐색 속도가 B-Tree에 비해 더 향상됨을 의미한다. 
다만, 이런 특징으로 트리에 중복된 키가 존재할 수 있다.

또한 B+Tree는 B-Tree과 다르게 **순차 검색에 유리하다.** 데이터베이스 특성상 부등호 연산(범위 검색)이 많이 발생하게 된다. 
이때, **B-Tree는 트리 순회를 해야하기 때문에 부등호 연산이 조금 오래 걸린다.** 
이런 단점을 극복하기 위해 B+Tree는 **리프 노드를 연결 리스트(linked list)로 연결했다.** 
즉, 정렬된 상태의 리프 노드를 순차 검색할 수 있게 된 것이다. 이로 인해 인덱싱에 더욱 적합한 자료구조가 될 수 있었다.

# 💡 DB 인덱스가 B tree 계열을 사용하는 이유

- DB는 기본적으로 Secondary Storage (SSD, HDD)에 저장된다.
- B Tree 인덱스는 이진 탐색 트리에 비해 Secondary storage 접근을 적게 한다. (트리의 높이가 낮기 때문)
- B Tree 노드는 block 단위의 저장 공간을 효율적으로 사용할 수 있다.
- Hash기반 인덱스는 eqaual연산에는 시간 복잡도가 O(1)로 효율적이지만, 범위 검색이나 정렬에는 사용할 수 없는 단점이 있다.

# References

- [Youtube-쉬운코드](https://www.youtube.com/watch?v=bqkcoSm_rCs&list=PLcXyemr8ZeoREWGhhZi5FZs6cvymjIBVe&index=27)
- [인덱스에서 B+Tree를 사용하는 이유](https://munak.tistory.com/182)
- [인덱스와 인덱싱 알고리즘](https://hudi.blog/db-index-and-indexing-algorithms/)
- [[자료구조/균형트리] B-트리, B+트리란?](https://suyeonme.tistory.com/102)
