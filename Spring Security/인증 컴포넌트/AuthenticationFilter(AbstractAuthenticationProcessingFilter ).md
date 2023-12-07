# 💡 AbstractAuthenticationProcessingFilter
```java
public abstract class AbstractAuthenticationProcessingFilter extends GenericFilterBean
        implements ApplicationEventPublisherAware, MessageSourceAware {

    private SecurityContextHolderStrategy securityContextHolderStrategy = SecurityContextHolder.getContextHolderStrategy();

    private AuthenticationManager authenticationManager;

    private RequestMatcher requiresAuthenticationRequestMatcher;

    private AuthenticationSuccessHandler successHandler = new SavedRequestAwareAuthenticationSuccessHandler();

    private AuthenticationFailureHandler failureHandler = new SimpleUrlAuthenticationFailureHandler();

    private SecurityContextRepository securityContextRepository = new RequestAttributeSecurityContextRepository();

    //...
}
```
- `AbstractAuthenticationProcessingFilter`는 인증 요청을 처리하는 인증 필터이다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/6a94fbf7-cfb8-4f7a-800f-fb593e6a3592)

## 🔴 동작 과정
1. 인증요청request가 `private RequestMatcher requiresAuthenticationRequestMatcher`와 일치하는지 확인하고, 일치하지 않으면 다음 필터체인을 실행한다.
```java
protected boolean requiresAuthentication(HttpServletRequest request, HttpServletResponse response) {
    if (this.requiresAuthenticationRequestMatcher.matches(request)) {
        return true;
    }
    return false;
}
```
2. 사용자가 제출한 자격 증명(ID, PW)으로 `Authentication` 객체를 생성한다. 이 때 생성된 인증 유형은 `AbstractAuthenticationProcessingFilter`의 하위 클래스에 따라 다르다.
예를 들어 `UsernamePasswordAuthenticationFilter`는 `HttpServletRequest`에 제출된 사용자 이름과 비밀번호에서 `UsernamePasswordAuthenticationToken`을 생성한다.
```java
@Override
public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {
    if (this.postOnly && !request.getMethod().equals("POST")) {
        throw new AuthenticationServiceException("Authentication method not supported: " + request.getMethod());
    }
    String username = obtainUsername(request);
    username = (username != null) ? username.trim() : "";
    String password = obtainPassword(request);
    password = (password != null) ? password : "";
    UsernamePasswordAuthenticationToken authRequest = UsernamePasswordAuthenticationToken.unauthenticated(username, password);
    // Allow subclasses to set the "details" property
    setDetails(request, authRequest);
    return this.getAuthenticationManager().authenticate(authRequest);
}
```
3. 생성된 `Authentication`객체로 `AuthenticationManager`에게 인증을 요청한다.
