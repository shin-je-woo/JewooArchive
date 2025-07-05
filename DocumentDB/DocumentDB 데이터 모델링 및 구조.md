## 💡 기본 구조

Amazon DocumentDB는 MongoDB와 호환되는 NoSQL 문서형 데이터베이스다.   
관계형 DB의 테이블-행-열 구조 대신 다음과 같은 단위를 사용한다.  

| 개념 | 역할 |
|------|------|
| **Database** | 논리적 데이터베이스 단위 |
| **Collection** | 하나의 테이블에 해당하는 문서 컨테이너 |
| **Document** | JSON-like 데이터 단위 |
| **Field** | Document 내의 속성 (Key-Value) |

## 💡 Document 예시

### users 컬렉션 예시

```json
{
  "_id": ObjectId("662b123..."),
  "name": "Alice",
  "email": "alice@example.com",
  "age": 29,
  "address": {
    "city": "Seoul",
    "zip": "12345"
  },
  "roles": ["admin", "editor"]
}
```

### orders 컬렉션 예시

```json
{
  "_id": ObjectId("662b456..."),
  "userId": ObjectId("662b123..."),
  "items": [
    { "productId": "p001", "quantity": 2, "price": 10000 },
    { "productId": "p002", "quantity": 1, "price": 5000 }
  ],
  "createdAt": ISODate("2024-06-01T12:00:00Z"),
  "status": "PAID"
}
```

## 💡 모델링 전략

DocumentDB에서는 데이터를 어떻게 설계할지에 따라 성능과 유지보수 난이도가 크게 달라진다.   
대표적인 모델링 전략은 두 가지다.

### Embedded vs Referenced

| 모델 | 설명 | 예시 |
|------|------|------|
| Embedded | 문서 안에 관련 데이터를 함께 포함시킴 | 주소 정보가 사용자 문서 안에 포함 |
| Referenced | 다른 컬렉션의 `_id`를 참조하여 연결 | 주문 문서가 `userId`를 참조 |

#### Embedded 모델 특징
- Join 없이 한 번의 조회로 필요한 정보 모두 확보
- 일관된 데이터 구조로 빠른 읽기 성능 확보
- 중첩 데이터가 크면 문서 크기(16MB) 제한에 도달할 수 있음

#### Referenced 모델 특징
- 관계형 데이터처럼 재사용 가능
- 데이터 변경 시 중앙 관리 가능
- Join(lookup)을 수동으로 수행해야 하고, 쿼리 복잡도 증가

> 일반적으로 **읽기 최적화 → Embedded**,  
> **쓰기/재사용 최적화 → Referenced** 모델이 적합하다.

## 💡 지원 필드 타입

DocumentDB는 MongoDB와 동일하게 다양한 JSON 타입을 지원한다.   

| 타입 | 예시 | 설명 |
|------|------|------|
| `String` | `"hello"` | 문자열 |
| `Number` | `42`, `3.14` | 정수 또는 실수 |
| `Boolean` | `true`, `false` | 참/거짓 |
| `Date` | `ISODate("2025-01-01T00:00:00Z")` | 날짜/시간 객체 |
| `Array` | `[1, 2, 3]` | 값의 리스트 |
| `Object` | `{ a: 1, b: 2 }` | 중첩된 JSON 객체 |
| `ObjectId` | `ObjectId("...")` | MongoDB의 기본 ID 필드, PK 역할 |
| `Null` | `null` | 빈 값 |

> 모든 문서는 `_id` 필드를 기본으로 가지며, 별도 지정하지 않으면 자동으로 `ObjectId`가 생성된다.

## 💡 인덱스 설계

DocumentDB는 성능을 위해 다양한 인덱스를 지원한다.   

| 인덱스 유형 | 설명 |
|-------------|------|
| **Single Field Index** | 단일 필드에 대한 기본 인덱스 |
| **Compound Index** | 다중 필드 조합 인덱스 |
| **Multikey Index** | 배열 필드에 대한 인덱스 |
| **Hashed Index** | 해시 기반 인덱스 (DocumentDB에선 샤딩용 아님) |
| **TTL Index** | 설정한 시간 이후 문서 자동 삭제 (예: 로그 만료용) |

인덱스를 잘못 설계하면 쿼리 속도 저하, 메모리 사용량 증가가 발생할 수 있으므로 반드시 쿼리 패턴에 맞춰 인덱스를 설계해야 한다.

## 💡 문서 크기 제한

- DocumentDB에서 하나의 문서는 **최대 16MB**까지 저장 가능
- 이미지, 영상, PDF 등 바이너리 데이터는 DocumentDB 대신 **S3, GridFS** 같은 외부 스토리지 사용 권장

## 💡 설계 팁

- 스키마는 자유롭지만, **팀 내 컨벤션으로 스키마 일관성 유지** 권장
- 자주 접근되는 필드는 반드시 인덱스 고려
- 정규화보다는 **도메인 중심 데이터 설계와 응답 최적화**에 집중
- 중첩 구조를 활용하되, 과도한 Depth는 피할 것
- Write-heavy 시스템에서는 Embedded 구조가 오히려 부작용이 될 수 있음

## 💡 RDB와의 개념 비교

| RDB 용어 | DocumentDB 대응 |
|----------|-----------------|
| 데이터베이스 | Database |
| 테이블 | Collection |
| 행(Row) | Document |
| 열(Column) | Field |
| Primary Key | `_id` 필드 |
| Foreign Key | 다른 문서의 `_id`를 참조 (Join은 애플리케이션에서 직접 처리) |
