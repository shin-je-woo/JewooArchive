# Previous

[Whatpl 프로젝트](https://github.com/whatpl/whatpl-backend) 중에 프론트엔드 개발자분이 백엔드를 로컬에서 띄우고 싶다고 하셔서 원격연결로 로컬세팅을 지원해주었다.

당시에 운영환경에서는 nginx가 애플리케이션 앞단에서 응답을 보낼 때 CORS 관련 헤더를 붙여서 나가게 되어 있어 프론트 개발자분 로컬에서 별도의 CORS 설정이 필요했다.

스프링 시큐리티 문서를 찾아보니 역시나 CORS관련 필터를 제공하고 있었다. [CorsFilter](https://docs.spring.io/spring-security/reference/servlet/integrations/cors.html)

그런데 문제는, `CorsFilter` 를 적용했음에도 프론트 개발자분 로컬환경에서 여전히 CORS관련 설정이 적용되지 않았다.

이번 포스팅에서 CorsFilter가 어떻게 동작하는지 알아보고, 해당 문제를 어떻게 해결했는지 작성해보려고 한다.

# 💡 CorsFilter

먼저, 스프링 시큐리티가 제공하는 `CorsFilter`가 무엇이고, 어떻게 동작하는지 알아보자.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/73a95bf3-36f8-49ab-b572-aef5898ee241)

![image](https://github.com/shin-je-woo/TIL/assets/39439576/d82f2706-7f54-46d4-8623-fa39688acedf)

위 설정대로 `SecurityFilterChain` 을 Bean으로 등록하면 아래 그림처럼 `FilterChainProxy` 에 `CorsFilter`가 적용된다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/3521a03e-1f04-4292-b39c-44102843ca1e)

`CorsFilter` 는 `OncePerRequestFilter` 를 상속받고, `CorsProcessor` 에 CORS 관련 설정을 위임한다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/b237ab66-b156-4317-962c-74ad00fb88e6)

`CorsProcessor` 의 기본적인 구현체는 `DefaultCorsProcessor` 이고, 해당 구현체에서 CORS 관련 응답 헤더가 설정된다.

▶️ DefaultCorsProcessor.processRequest() 메서드
```java
@Override
@SuppressWarnings("resource")
public boolean processRequest(@Nullable CorsConfiguration config, HttpServletRequest request,
                              HttpServletResponse response) throws IOException {

    Collection<String> varyHeaders = response.getHeaders(HttpHeaders.VARY);
    if (!varyHeaders.contains(HttpHeaders.ORIGIN)) {
        response.addHeader(HttpHeaders.VARY, HttpHeaders.ORIGIN);
    }
    if (!varyHeaders.contains(HttpHeaders.ACCESS_CONTROL_REQUEST_METHOD)) {
        response.addHeader(HttpHeaders.VARY, HttpHeaders.ACCESS_CONTROL_REQUEST_METHOD);
    }
    if (!varyHeaders.contains(HttpHeaders.ACCESS_CONTROL_REQUEST_HEADERS)) {
        response.addHeader(HttpHeaders.VARY, HttpHeaders.ACCESS_CONTROL_REQUEST_HEADERS);
    }

    if (!CorsUtils.isCorsRequest(request)) {
        return true;
    }

    if (response.getHeader(HttpHeaders.ACCESS_CONTROL_ALLOW_ORIGIN) != null) {
        logger.trace("Skip: response already contains \"Access-Control-Allow-Origin\"");
        return true;
    }

    boolean preFlightRequest = CorsUtils.isPreFlightRequest(request);
    if (config == null) {
        if (preFlightRequest) {
            rejectRequest(new ServletServerHttpResponse(response));
            return false;
        }
        else {
            return true;
        }
    }

    return handleInternal(new ServletServerHttpRequest(request), new ServletServerHttpResponse(response), config, preFlightRequest);
}
```

▶️ DefaultCorsProcessor.handleInternal() 메서드
```java
protected boolean handleInternal(ServerHttpRequest request, ServerHttpResponse response,
                                 CorsConfiguration config, boolean preFlightRequest) throws IOException {

    String requestOrigin = request.getHeaders().getOrigin();
    String allowOrigin = checkOrigin(config, requestOrigin);
    HttpHeaders responseHeaders = response.getHeaders();

    if (allowOrigin == null) {
        logger.debug("Reject: '" + requestOrigin + "' origin is not allowed");
        rejectRequest(response);
        return false;
    }

    HttpMethod requestMethod = getMethodToUse(request, preFlightRequest);
    List<HttpMethod> allowMethods = checkMethods(config, requestMethod);
    if (allowMethods == null) {
        logger.debug("Reject: HTTP '" + requestMethod + "' is not allowed");
        rejectRequest(response);
        return false;
    }

    List<String> requestHeaders = getHeadersToUse(request, preFlightRequest);
    List<String> allowHeaders = checkHeaders(config, requestHeaders);
    if (preFlightRequest && allowHeaders == null) {
        logger.debug("Reject: headers '" + requestHeaders + "' are not allowed");
        rejectRequest(response);
        return false;
    }

    responseHeaders.setAccessControlAllowOrigin(allowOrigin);

    if (preFlightRequest) {
        responseHeaders.setAccessControlAllowMethods(allowMethods);
    }

    if (preFlightRequest && !allowHeaders.isEmpty()) {
        responseHeaders.setAccessControlAllowHeaders(allowHeaders);
    }

    if (!CollectionUtils.isEmpty(config.getExposedHeaders())) {
        responseHeaders.setAccessControlExposeHeaders(config.getExposedHeaders());
    }

    if (Boolean.TRUE.equals(config.getAllowCredentials())) {
        responseHeaders.setAccessControlAllowCredentials(true);
    }

    if (Boolean.TRUE.equals(config.getAllowPrivateNetwork()) &&
            Boolean.parseBoolean(request.getHeaders().getFirst(ACCESS_CONTROL_REQUEST_PRIVATE_NETWORK))) {
        responseHeaders.set(ACCESS_CONTROL_ALLOW_PRIVATE_NETWORK, Boolean.toString(true));
    }

    if (preFlightRequest && config.getMaxAge() != null) {
        responseHeaders.setAccessControlMaxAge(config.getMaxAge());
    }

    response.flush();
    return true;
}
```

# 💡 문제 상황

앞에서 봤던 `FilterChainProxy` 에 등록된 필터목록을 다시 확인해보자.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/3521a03e-1f04-4292-b39c-44102843ca1e)

`CorsFilter` 앞에 3개의 필터가 먼저 적용되게 되는데, 중요한건 커스텀해서 등록한 `JwtAuthenticationFilter` 이다.

`JwtAuthenticationFilter`는 요청이 들어오면 `Authorization`헤더에 있는 JWT를 읽어서 `Authentication`객체로 변환 후, `SecurityContextRepository`에 저장하는 필터이다.

쉽게 설명하면 이 필터는 JWT를 읽고 `SecurityContextPersistenceFilter`의 역할을 대신한다.

문제는 이 필터가 `SecurityContextHolderFilter`에서 load하는데 필요한 `SecurityContextRepository`에 저장하는 코드 때문에 `CorsFilter`보다 앞에 위치하게 된다는 점이다.

▶️ JwtAuthenticationFilter.doFilterInternal() 메서드
```java
@Override
protected void doFilterInternal(@Nonnull HttpServletRequest request, @Nonnull HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
    String jwt = extractToken(request);
    try {
        Authentication authentication = jwtService.resolveToken(jwt);
        // 인증된 사용자가 프로필을 작성했는지 확인한다.
        checkHasProfile(request, authentication);
        // SecurityContext 에 Authentication 저장
        SecurityContext securityContext = SecurityContextHolder.createEmptyContext();
        securityContext.setAuthentication(authentication);
        SecurityContextHolder.setContext(securityContext);
        // SecurityContextHolderFilter 에서 load 될 SecurityContext 저장
        securityContextRepository.saveContext(securityContext, request, response);
        filterChain.doFilter(request, response);
    } catch (ExpiredJwtException e) {
        responseError(response, ErrorCode.EXPIRED_TOKEN);
    } catch (MalformedJwtException e) {
        responseError(response, ErrorCode.MALFORMED_TOKEN);
    } catch (SignatureException e) {
        responseError(response, ErrorCode.INVALID_SIGNATURE);
    } catch (BizException e) {
        responseError(response, e.getErrorCode());
    }
}
```

위 코드에서 JWT를 읽다가 토큰이 만료되거나, 변조된 토큰이거나 하는 등의 문제가 발생하면 `responseError` 처리를 하게 되는데,

`responseError`는 다음 filterChain을 호출하지 않고, 바로 에러를 응답한다.

▶️ JwtAuthenticationFilter.responseError() 메서드
```java
private void responseError(HttpServletResponse response, ErrorCode errorCode) throws IOException {
    response.setStatus(HttpStatus.UNAUTHORIZED.value());
    response.setContentType(MediaType.APPLICATION_JSON_VALUE);
    response.setCharacterEncoding(StandardCharsets.UTF_8.displayName());
    ErrorResponse errorResponse = ErrorResponse.of(errorCode);
    objectMapper.writeValue(response.getWriter(), errorResponse);
}
```

바로 이 부분에서 CORS 관련 응답헤더가 설정되지 않았던 것이다.😥

# 💡 해결 & CORS 관련 헤더는 어디에서 설정하는 것이 좋을까?

프론트엔드 개발자분 로컬환경에서는 저 부분에서도 CORS관련 설정을 가져와서 응답헤더에 set하도록 `JwtAuthenticationFilter`를 수정하여 해결했다.

운영환경에서는 nginx를 사용하기 때문에 익숙한대로 nginx에 CORS 설정을 했었는데, 그 당시에는 익숙한 방식대로 설정했던 것 같다.

프로젝트 초기에 다른 개발자 분이 "CORS설정을 스프링에서 하지 않고 nginx에서 하신 이유가 있나요?" 라고 질문하셨을 때 명쾌하게 답변을 하지 못했었다.

이번 경우처럼 SpringSecurity에서 제공하는 `CorsFilter`를 사용하면 간단하게 CORS관련 설정을 할 수 있겠지만, `CorsFilter`까지 요청이 가지 않으면 무용지물이 된다.

그래서 확실하게 CORS관련 설정을 하려면 nginx같이 애플리케이션에서 앞단에서 하는 것이 좋겠다는 생각을 하게 되었다. (애플리케이션에서는 어떤 일이 발생할지 모르기 때문에..)
