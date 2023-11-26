# 💡 AuthorizationManager
- `AuthorizationManager` 는 실제 인증처리를 진행하는 컴포넌트이다.
- `AccessDecisionManager`와 `AccessDecisionVoter`를 모두 대체한다.
- `AuthorizationFilter`는 하나의 `AuthorizationManager`를 갖는데, 대표적으로 Request URL을 처리하는 `RequestMatcherDelegatingAuthorizationManager`가 있다.

```java
@FunctionalInterface
public interface AuthorizationManager<T> {

    default void verify(Supplier<Authentication> authentication, T object) {
        AuthorizationDecision decision = check(authentication, object);
        if (decision != null && !decision.isGranted()) {
            throw new AccessDeniedException("Access Denied");
        }
    }

    @Nullable
    AuthorizationDecision check(Supplier<Authentication> authentication, T object);
}
```
- `AuthorizationManager`는 default메서드 verify()와 abstract메서드 check()로 정의되어 있다.
- check()메서드는 Authentication타입의 Supplier와 제네릭 T타입 객체를 갖는데, `AuthorizationFilter`에 inject되는 `AuthrorizationManager`는 `HttpServletRequest` 타입을 갖는다.
- 따라서 check()메서드는 인증객체와 사용자요청을 통해 인증을 진행한다.
- `Authentication`객체를 Supplier로 제공받는 이유는 실제 인증과정에서 인증객체를 조회하도록 지연시키기 위함이다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/f8634dd5-69d1-420d-b9b5-1c2c6b361d17)

# 💡 RequestMatcherDelegatingAuthorizationManager
- `RequestMatcherDelegatingAuthorizationManager`는 `AuthorizationManager`의 대표적인 구현 클래스이다.
- 직접 인증 처리를 수행 하지 않고 `RequestMatcher`를 통해 매치되는 `AuthorizationManager`구현 클래스에게 인증처리를 위임한다.
- 필드로 인증을 위임할 `AuthorizationManager`리스트를 가지고 있다.
  
![image](https://github.com/shin-je-woo/TIL/assets/39439576/2db367c7-a1ff-44ad-ba87-af47eb8dabc0)

- `AuthorizationFilter`의 필드에 `AuthorizationManager`가 하나 있는데, 아래 이미지는 `AuthorizationFilter`에 inject된 `RequestMatcherDelegatingAuthorizationManager`를 디버깅한 모습이다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/d7e5bdad-2ffb-4633-a10d-90e73a10be41)

▶️ RequestMatcherDelegatingAuthorizationManager 코드
```java
public final class RequestMatcherDelegatingAuthorizationManager implements AuthorizationManager<HttpServletRequest> {

    //...
    private final List<RequestMatcherEntry<AuthorizationManager<RequestAuthorizationContext>>> mappings;
    //...

    @Override
    public AuthorizationDecision check(Supplier<Authentication> authentication, HttpServletRequest request) {
        for (RequestMatcherEntry<AuthorizationManager<RequestAuthorizationContext>> mapping : this.mappings) {
            RequestMatcher matcher = mapping.getRequestMatcher();
            MatchResult matchResult = matcher.matcher(request); // (1)
            if (matchResult.isMatch()) {
                AuthorizationManager<RequestAuthorizationContext> manager = mapping.getEntry();
                return manager.check(authentication,
                        new RequestAuthorizationContext(request, matchResult.getVariables())); // (2)
            }
        }
        return DENY;
    }

    //...
}
```
- check()메서드를 보면 (1)번에서 현재 요청(request)과 필드로 등록된 `AuthorizationManager`의 `RequestMatcher`가 일치하는지 확인한다.
- (2)번에서 일치하는 `AuthorizationManager`의 check()메서드를 호출하여 인증처리를 위임한다.

📌 참고) RequestMatcher
- `RequestMatcher`는 다음 이미지처럼 시큐리티 설정클래스에서 등록하게 된다.
![image](https://github.com/shin-je-woo/TIL/assets/39439576/123449ec-76ad-4290-b113-4603b9afd8ec)
- `RequestMatcher`를 등록하는 다양한 방법은 [스프링 시큐리티-Authorize HttpServletRequests](https://docs.spring.io/spring-security/reference/servlet/authorization/authorize-http-requests.html)를 참고하자.
