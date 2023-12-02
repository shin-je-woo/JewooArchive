# 💡 Authentication
- `Authentication` 인터페이스는 Spring Security 내에서 두 가지 사용목적이 있다.
  - 첫 번째. 로그인 시 자격증명을 위해 `AuthenticationManager`에 전달되는 용도 -> 이 경우에 `isAuthenticated()`는 false이다.
  - 두 번째. 사용자의 인증 정보를 저장하는 토큰개념.
- 인증 후에는 최종 인증 결과(user 객체, 권한 정보)를 담고 `SecurityContext` 에 저장되어 전역적으로 참조가 가능하다.
- **Authentication의 대표적인 구현체**
  - `UsernamePasswordAuthenticationToken`
  - `AnonymousAuthenticationToken`
  - `RememberMeAuthenticationToken`
- **Authentication의 구조**
  - **principal** : 사용자를 식별하는 정보. 보통 `UserDetails`의 인스턴스인 경우가 많다.
  - **credentials** : 사용자 비밀번호. 대부분의 경우, 사용자가 인증된 후 유출되지 않도록 지워진다.
  - **authorities** : 인증된 사용자의 권한 목록
  - **details** : 인증 부가 정보
  - **authenticated** : 인증 여부

![image](https://github.com/shin-je-woo/TIL/assets/39439576/0fd5bae4-0b25-4687-aa7d-b7cca05f11cb)

# 💡 SecurityContextHolder, SecurityContext
![image](https://github.com/shin-je-woo/TIL/assets/39439576/25354684-eb2b-4329-ba19-4a5d391ab13c)

- Spring Security 인증 모델의 핵심은 `SecurityContextHolder`이다.
- `SecurityContext`를 가지고 있는 객체이다.
- `SecurityContext` 저장 방식(`SecurityContextHolderStrategy`)
  - **MODE_THREADLOCAL**: 스레드당 SecurityContext객체를 할당. 스프링 시큐리티의 기본 설정(default)
  - **MODE_INHERITABLETHREADLOCAL**: 메인 스레드와 자식 스레드에 관하여 동일한 SecurityContext를 유지
  - **MODE_GLOBAL**: 응용 프로그램에서 단 하나의 SecurityContext를 저장한다.
- `SecurityContext`는 `Authentication` 객체가 저장되는 보관소이다.
- 다음과 같은 코드를 이용해 어디서나 인증객체를 얻어올 수 있다.
```java
Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
```
- 참고) Spring Security의 `FilterChainProxy`는 `SecurityContext`가 항상 지워지는 것을 보장한다.
