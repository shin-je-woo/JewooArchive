# 💡 AuthenticationManager
- `AuthenticationManager`는 Spring Security의 필터가 인증을 수행하는 방법을 정의하는 API이다.
- 즉, `AuthenticationManager`는 인증을 수행한 후 `Authentication`객체를 리턴하는데, 시큐리티 필터는 반환된 `Authentication`을 `SecurityContext`에 설정한다.
- `AuthenticationManager`의 대표적인 구현체는 `ProviderManager`이다.

```java
public interface AuthenticationManager {
    Authentication authenticate(Authentication authentication) throws AuthenticationException;
}
```

# 💡 ProviderManager
- `AuthenticationManager`의 대표적인 구현체이다.
- `ProviderManager`는 `AuthenticationProvider`의 목록을 가지고 있고, 이들을 순회하면서 인증을 위임하는 역할을 수행한다.
- 참고로, 인증작업 이후 `ProviderManager`는 `Authentication`객체의 `Credential`정보를 지운다. (설정으로 지우지 않을 수도 있다.)

# 💡 AuthenticationProvider
```java
public interface AuthenticationProvider {

    Authentication authenticate(Authentication authentication) throws AuthenticationException;

    boolean supports(Class<?> authentication);
}

```

- `AuthenticationProvider`는 실제로 인증을 수행하는 컴포넌트이다.
- 각 `AuthenticationProvider`는 `Authentication`객체 유형에 따라 인증 방법이 다르다.
  - `supports`메서드에서 `Authentication`유형을 파악하고, `authenticate`메서드에서 인증을 수행한다.
  - `DaoAuthenticationProvider`는 유저이름/패스워드 기반 인증 제공
  - `JwtAuthenticationProvider`는 JWT token 인증 제공
 

- 아래는 내가 작성한 간단한 `AuthenticationProvider`이다.
- 커스텀한 `UserDetailsService`에서 사용자정보(`UserDetails`)를 가져오고, `Authentication`객체(`CustomAuthenticationToken`)에 담아 반환한다.

```java
@RequiredArgsConstructor
public class CustomAuthenticationProvider implements AuthenticationProvider {

    private final UserDetailsService userDetailsService;
    private final PasswordEncoder passwordEncoder;

    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {

        String username = authentication.getName();
        String password = (String) authentication.getCredentials();

        UserDetails userDetails = userDetailsService.loadUserByUsername(username);
        if (!(userDetails instanceof AccountContext accountContext))
            throw new IllegalStateException("CustomUserDetailsService를 등록해주세요.");

        // 사용자가 인증요청한 패스워드와 사용자의 PW(DB에 저장)와 다르면 exception
        if (!passwordEncoder.matches(password, accountContext.getAccount().getPassword())) {
            throw new BadCredentialsException("BadCredentialsException");
        }

        return new CustomAuthenticationToken(accountContext, null, accountContext.getAuthorities());
    }

    @Override
    public boolean supports(Class<?> authentication) {
        return authentication.isAssignableFrom(AjaxAuthenticationToken.class);
    }
}
```
