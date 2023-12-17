# 💡 PermissionEvaluator
- 메소드 시큐리티 기능 중 `hasPermission`은 객체레벨에서 인가처리를 제공하는 `PermissionEvaluator`의 후킹포인트이다.
- permission을 evaluation하는 컴포넌트로 `PermissionEvaluator`인터페이스가 제공된다.
- 기본 구현체는 `DenyAllPermissionEvaluator`로써, 모두 false를 반환한다.
- 따라서, 애플리케이션에 맞게 `PermissionEvaluator`을 커스텀해야 한다.

## PermissionEvaluator 만들기
```java
@Slf4j
@RequiredArgsConstructor
public class CustomPermissionEvaluator implements PermissionEvaluator {

    private final PostRepository postRepository;

    @Override
    public boolean hasPermission(Authentication authentication, Object targetDomainObject, Object permission) {
        // 필요시 구현
        return false;
    }

    @Override
    public boolean hasPermission(Authentication authentication, Serializable targetId, String targetType, Object permission) {
        UserPrincipal userPrincipal = (UserPrincipal) authentication.getPrincipal();
        Post post = postRepository.findById((Long) targetId)
                .orElseThrow(PostNotFound::new);

        if (!post.getUserId().equals(userPrincipal.getUserId())) {
            log.error("[인가실패] 해당 사용자가 작성한 글이 아닙니다. targetId={}", targetId);
            return false;
        }

        return true;
    }
}
```
- `PermissionEvaluator`를 구현해서 애플리케이션에 적절한 인가처리를 작성한다.
- 위 예제코드는 간단하게 게시글의 작성자id와 현재 로그인한(authentication)사용자의 id만 비교한다.
- 아래 그림처럼 authentication의 각 targetType별로 인가처리를 지정해줄 수도 있다. (이 경우에는 authentication의 authorities를 추가해주는 기능도 구현해야 한다.)

![image](https://github.com/shin-je-woo/TIL/assets/39439576/b51c6c33-7616-4b56-b108-90d49ae2719a)


## MethodSecurity 설정
```java
@Configuration
@EnableMethodSecurity
@RequiredArgsConstructor
public class MethodSecurityConfig {

    private final PostRepository postRepository;

    @Bean
    public MethodSecurityExpressionHandler methodSecurityExpressionHandler() {
        DefaultMethodSecurityExpressionHandler expressionHandler = new DefaultMethodSecurityExpressionHandler();
        expressionHandler.setPermissionEvaluator(new CustomPermissionEvaluator(postRepository));
        return expressionHandler;
    }
}
```
- 설정클래스레벨에서 `@EnableMethodSecurity`을 적용해야 메소드 시큐리티가 적용된다.
- `MethodSecurityExpressionHandler`에 `setPermissionEvaluator()`메서드로 `PermissionEvaluator`을 등록해주고 스프링 빈으로 설정해주면 된다.

## 사용예제
- 위 설정으로 `@PreAuthorize`에 `hasPermission`을 사용할 수 있다.
- 사용할 수 있는 두 가지 형태(method)가 존재한다.

1. 게시글 Object에 권한이 있는지 확인하는 소스
```java
@PreAuthorize("hasRole('ADMIN') and hasPermission('POST', 'DELETE')")
@DeleteMapping("/posts/{postId}")
public void delete(@PathVariable Long postId) {
    postService.delete(postId);
}
```

2. 게시글 고유번호 id를 넘겨 확인하는 소스
```java
@PreAuthorize("hasRole('ADMIN') and hasPermission(#postId, 'POST', 'DELETE')")
@DeleteMapping("/posts/{postId}")
public void delete(@PathVariable Long postId) {
    postService.delete(postId);
}
```
- 두 메소드가 `CustomPermissionEvaluator`에서 만든 두 개의 메소드로 각각 호출 되기 때문에 필요에 따라 구현하면 된다.
- 예제에서는 targetId가 존재하는 경우만 구현했으므로 2번째만 사용가능하고, 1번쨰는 필요시 구현하면 된다.
