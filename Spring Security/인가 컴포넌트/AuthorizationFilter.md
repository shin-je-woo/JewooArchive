# 💡 AuthorizationFilter
- `AuthorizationFilter` 는 `FilterSecurityInterceptor` 를 대체한다. (스프링 시큐리티 5.5부터)
- `AuthorizationFilter` 는 `HttpServletRequests` 에 대한 인가처리를 제공하는 보안필터이다.
- 기본 아키텍처는 다음 이미지와 같다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/236e36b6-d45e-43a5-b66c-21315da10be9)

1. `AuthorizationFilter`는 `SecurityContextHolder`로 부터 `Authentication`을 얻어온다.
2. `Authentication`과 `HttpServletRequest`를 `AuthorizationManager`에게 전달한다.
    - 참고로 `RequestMatcherDelegatingAuthorizationManager`는 직접 권한 처리를 하는 것이 아니라 `RequestMatcher`를 통해 매치되는 `AuthorizationManager`의 구현체에게 권한 처리를 위임한다. (AuthenticationManager가 AuthenticationProvider에게 인증처리를 위임하는 것과 비슷한 듯)
3. 매치되는 구현체가 있다면, 해당 구현체는 사용자의 권한을 체크한다.
4. 적절한 권한이라면 다음 요청 프로세스를 계속 이어 나간다.
5. 적절한 권한이 아니라면 `AccessDeniedException`을 던지고, `ExceptionTranslationFilter`가 예외를 처리하게 된다.

# 코드 분석
```java
public class AuthorizationFilter extends GenericFilterBean {

    private SecurityContextHolderStrategy securityContextHolderStrategy = SecurityContextHolder
            .getContextHolderStrategy();
    
    private final AuthorizationManager<HttpServletRequest> authorizationManager;
    
    //...

    // (1)
    public AuthorizationFilter(AuthorizationManager<HttpServletRequest> authorizationManager) {
        Assert.notNull(authorizationManager, "authorizationManager cannot be null");
        this.authorizationManager = authorizationManager;
    }

    //...
    
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain chain)
            throws ServletException, IOException {

        HttpServletRequest request = (HttpServletRequest) servletRequest;
        HttpServletResponse response = (HttpServletResponse) servletResponse;
        //...
        try {
            AuthorizationDecision decision = this.authorizationManager.check(this::getAuthentication, request); // (2)
            this.eventPublisher.publishAuthorizationEvent(this::getAuthentication, request, decision);
            if (decision != null && !decision.isGranted()) {
                throw new AccessDeniedException("Access Denied"); // (3)
            }
            chain.doFilter(request, response); // (4)
        }
        finally {
            request.removeAttribute(alreadyFilteredAttributeName);
        }
    }
    
    private Authentication getAuthentication() {
        Authentication authentication = this.securityContextHolderStrategy.getContext().getAuthentication();
        if (authentication == null) {
            throw new AuthenticationCredentialsNotFoundException(
                    "An Authentication object was not found in the SecurityContext");
        }
        return authentication;
    }
    
    //...
    //...
}
```
1. DI를 통해 `AuthorizationManager`를 할당한다. (스프링 시큐리티 기본설정으로는 `RequestMatcherDelegatingAuthorizationManager`가 inject되는 것 같다.)
2. `authorizationManager.check()`메서드의 인자로 `Authentication`과 `HttpServletRequest`를 넘겨주며 인가처리를 요청한다.
    - 특이하게 `Authentication`객체가 아니라 `Supplier<Authentication>`를 인자로 넘겨주어 `Authentication`조회를 지연시킨다.
    - 이러면 실제 인증조회 횟수는 많아질 수 있지만, 필요할 때만 인증조회를 하게되는 장점이 있다.
3. 만약, 인가에 실패하면 `AccessDeniedException`을 발생시킨다. (이 Exception은 `ExceptionTranslationFilter`가 받아서 처리한다.)
4. 인가에 문제가 없다면 다음 필터를 실행한다.
