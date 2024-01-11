# 💡 JWT(JSON Web Token)란?
- HTTP는 기본적으로 state-less를 지향하고 있고, JWT를 이용하면 state-less하게 서버-클라이언트 통신이 가능하다.
  - state-less(무상태)란? 서버-클라이언트 구조에서 서버가 클라이언트의 상태를 가지고 있지 않은 상태
- 장점: 서버의 확장성이 높다. (scale-out)
- 단점: state-ful 방식보다 비교적 많은 양의 데이터가 반복적으로 전송되므로 네트워크 성능저하가 될 수 있다.
- 단점: 데이터 노출로 인한 보안문제가 발생할 수 있다.

# 💡 JWT 구조
- JWT는 점(.)으로 구분된 3개의 파트로 구성된다.
  - 헤더(Header)
  - 페이로드(Payload)
  - 서명(Signature)
- xxxxx.yyyyy.zzzzz

## 헤더(Header)
- 헤더는 일반적으로 2개로 이루어져 있다.
  - 토큰타입
  - 암호화 알고리즘(예를 들어,  HMAC SHA256 인지 RSA인지)

```JSON
{
  "alg": "HS256",
  "typ": "JWT"
}
```

## 페이로드 (Payload)
- 페이로드는 클레임으로 구성되어 있다.
- 클레임은 엔티티(일반적으로 사용자정보) 및 추가데이터에 대한 설명이다.
- 클레임은 데이터 축소를 위해 일반적으로 3글자를 권장한다.
- 클레임에는 등록된 클레임(registered), 공개 클레임(public), 비공개 클레임(private) 세 가지 유형이 있다 .

### 등록된 클레임 (Registered claims)
- 필수는 아니지만 권장되는 미리 정의된 클레임 집합
- iss(issuer) : 토큰 발급자
- sub(subject) : 토큰 제목 - 일반적으로 사용자에 대한 식별 값
- aud(audience) : 토큰 대상자
- exp(expiration time) : 토큰 만료 시간, 시간은 NumericDate 형식으로 되어있어야 하며 (예: 1480849147370) 언제나 현재 시간보다 이후로 설정되어 있어야 한다.
- iat(issued at) : 토큰 발급 시간

### 공개 클레임 (Public claims)
- 공개 클레임들은 충돌이 방지된 (collision-resistant) 이름을 가지고 있어야 한다.
- 충돌을 방지하기 위해서 클레임 이름을 URI 형식으로 지어야 한다.

```JSON
{
    "https://github.com/shin-je-woo": true
}
```

### 비공개 클레임 (Private claims)
- 등록된 클레임도아니고, 공개된 클레임들도 아니다.
- 클라이언트와 서버 간의 협의하에 사용되는 맞춤 클레임이다.

```JSON
{
    "username": "shinjewoo"
}
```

## 서명 (Signature)
- 서명은 인코딩된 헤더, 인코딩된 페이로드를 합친 후 주어진 비밀키로 해시하여 생성한다.
- 예를 들어 HMAC SHA256 알고리즘을 사용하는 경우 서명은 다음과 같은 방식으로 생성된다.

```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```
- 서명은 메시지가 도중에 변경되지 않았는지 확인하는 데 사용된다.
- 서버가 개인키를 가지고 Signature를 복호화한 다음, 클라이언트가 보낸 토큰의 header가 복호화된 header와 일치하는지, 토큰의 payload가 복호화된 payload와 일치하는지 확인한다.
- 만약 클라이언트가 보낸 header, payload가 복호화된 header, payload와 다르다면 변조된 토큰으로 인지할 수 있다.
