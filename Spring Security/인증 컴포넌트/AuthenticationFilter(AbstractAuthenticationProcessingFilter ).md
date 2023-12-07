# 💡 AbstractAuthenticationProcessingFilter
```java
public abstract class AbstractAuthenticationProcessingFilter extends GenericFilterBean
        implements ApplicationEventPublisherAware, MessageSourceAware {

    private SecurityContextHolderStrategy securityContextHolderStrategy = SecurityContextHolder.getContextHolderStrategy();

    private AuthenticationManager authenticationManager;

    private RequestMatcher requiresAuthenticationRequestMatcher;

    private SessionAuthenticationStrategy sessionStrategy = new NullAuthenticatedSessionStrategy();

    private AuthenticationSuccessHandler successHandler = new SavedRequestAwareAuthenticationSuccessHandler();

    private AuthenticationFailureHandler failureHandler = new SimpleUrlAuthenticationFailureHandler();

    private SecurityContextRepository securityContextRepository = new RequestAttributeSecurityContextRepository();

    //...
}
```
- `AbstractAuthenticationProcessingFilter`는 인증 요청을 처리하는 인증 필터이다.
- 간단하게 로그인을 처리하는 필터로 이해하면 된다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/6a94fbf7-cfb8-4f7a-800f-fb593e6a3592)

## 🔴 인증 과정
1. 인증요청request가 `private RequestMatcher requiresAuthenticationRequestMatcher`와 일치하는지 확인하고, 일치하지 않으면 다음 필터체인을 실행한다.
(참고. `requiresAuthenticationRequestMatcher`는 생성자로 값을 세팅할 수 있다. 커스텀AuthenticationFilter를 만들 때 참고)
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

## 🔴 인증 성공시
```java
protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain,
                                        Authentication authResult) throws IOException, ServletException {
    SecurityContext context = this.securityContextHolderStrategy.createEmptyContext();
    context.setAuthentication(authResult);
    this.securityContextHolderStrategy.setContext(context);
    this.securityContextRepository.saveContext(context, request, response);
    if (this.logger.isDebugEnabled()) {
        this.logger.debug(LogMessage.format("Set SecurityContextHolder to %s", authResult));
    }
    this.rememberMeServices.loginSuccess(request, response, authResult);
    if (this.eventPublisher != null) {
        this.eventPublisher.publishEvent(new InteractiveAuthenticationSuccessEvent(authResult, this.getClass()));
    }
    this.successHandler.onAuthenticationSuccess(request, response, authResult);
}
```
- `SessionAuthenticationStrategy`에 새 로그인이 통보된다. 전략에 따라 기존 세션을 종결시킬지, 다중 로그인을 허용할지 등을 결정한다. `setSessionAuthenticationStrategy()`메서드로 설정할 수 있다. 기본값은 `NullAuthenticatedSessionStrategy`이다.
- 인증결과인 `Authentication`객체가 `SecurityContextHolder`에 담긴다. 또한, `SecurityContextRepository`에 `SecurityContext`를 담아둔다.
- `RememberMeServices.loginSuccess()`메서드가 호출된다. (remember-me가 설정되어 있지 않으면 아무런 작업도 수행하지 않는다.)
- `AuthenticationSuccessHandler.onAuthenticationSuccess()`메서드가 호출된다. (보통 커스텀하여 사용한다.)

## 🔴 인증 실패시
```java
protected void unsuccessfulAuthentication(HttpServletRequest request, HttpServletResponse response,
                                          AuthenticationException failed) throws IOException, ServletException {
    this.securityContextHolderStrategy.clearContext();
    this.logger.trace("Failed to process authentication request", failed);
    this.logger.trace("Cleared SecurityContextHolder");
    this.logger.trace("Handling authentication failure");
    this.rememberMeServices.loginFail(request, response);
    this.failureHandler.onAuthenticationFailure(request, response, failed);
}
```
- `The SecurityContextHolder`를 비운다.
- `RememberMeServices.loginFail()`메서드가 호출된다. (remember-me가 설정되어 있지 않으면 아무런 작업도 수행하지 않는다.)
- `AuthenticationFailureHandler.onAuthenticationFailure()`메서드가 호출된다. (보통 커스텀하여 사용한다.)
