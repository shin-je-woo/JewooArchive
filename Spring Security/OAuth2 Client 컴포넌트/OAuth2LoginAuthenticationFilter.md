# 💡 OAuth2LoginAuthenticationFilter
- `OAuth2LoginAuthenticationFilter` 는 `/login/oauth2/code/{registrationId}` 로 들어온 요청을 처리하는 필터이다.
- Authorization Server의 로그인 과정 이후 받은 code를 이용해서 액세스 토큰 발행 및 사용자 정보를 요청하여 받아온다.
- `AbstractAuthenticationProcessingFilter` 를 상속받는다. 즉, 로그인 처리 관련 필터이다.

# 💡 동작 과정 분석
▶️ OAuth2LoginAuthenticationFilter - OAuth2 로그인 처리 필터
```java
@Override
public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response)
        throws AuthenticationException {
    // redirect된 uri의 쿼리파라미터를 추출한다. (보통 code, state)
    MultiValueMap<String, String> params = OAuth2AuthorizationResponseUtils.toMultiMap(request.getParameterMap());

    // 정상적으로 api요청이 왔는지 확인
    if (!OAuth2AuthorizationResponseUtils.isAuthorizationResponse(params)) {
        OAuth2Error oauth2Error = new OAuth2Error(OAuth2ErrorCodes.INVALID_REQUEST);
        throw new OAuth2AuthenticationException(oauth2Error, oauth2Error.toString());
    }

    // OAuth2AuthorizationRequestRedirectFilter에서 저장했던 OAuth2AuthorizationRequest 객체 가져오기
    // 스프링 시큐리티는 기본적으로 session을 스토어로 사용
    OAuth2AuthorizationRequest authorizationRequest = this.authorizationRequestRepository
            .removeAuthorizationRequest(request, response);
    if (authorizationRequest == null) {
        OAuth2Error oauth2Error = new OAuth2Error(AUTHORIZATION_REQUEST_NOT_FOUND_ERROR_CODE);
        throw new OAuth2AuthenticationException(oauth2Error, oauth2Error.toString());
    }

    // registrationId를 통해 ClientRegistration 가져오기
    String registrationId = authorizationRequest.getAttribute(OAuth2ParameterNames.REGISTRATION_ID);
    ClientRegistration clientRegistration = this.clientRegistrationRepository.findByRegistrationId(registrationId);
    if (clientRegistration == null) {
        OAuth2Error oauth2Error = new OAuth2Error(CLIENT_REGISTRATION_NOT_FOUND_ERROR_CODE,
                "Client Registration not found with Id: " + registrationId, null);
        throw new OAuth2AuthenticationException(oauth2Error, oauth2Error.toString());
    }

    // redirect된 uri로부터 받은 Parameter 추출 후 OAuth2AuthorizationResponse객체로 변경 (보통 code, state)
    String redirectUri = UriComponentsBuilder.fromHttpUrl(UrlUtils.buildFullRequestUrl(request))
            .replaceQuery(null)
            .build()
            .toUriString();
    OAuth2AuthorizationResponse authorizationResponse = OAuth2AuthorizationResponseUtils.convert(params,
            redirectUri);

    // 토큰 요청을 위한 OAuth2LoginAuthenticationToken객체 생성
    Object authenticationDetails = this.authenticationDetailsSource.buildDetails(request);
    OAuth2LoginAuthenticationToken authenticationRequest = new OAuth2LoginAuthenticationToken(clientRegistration,
            new OAuth2AuthorizationExchange(authorizationRequest, authorizationResponse));
    authenticationRequest.setDetails(authenticationDetails);

    // 중요!! providerManager에서 OAuth2LoginAuthenticationProvider.authenticate() 실행
    OAuth2LoginAuthenticationToken authenticationResult = (OAuth2LoginAuthenticationToken) this
            .getAuthenticationManager().authenticate(authenticationRequest);

    // 받아온 token converting
    OAuth2AuthenticationToken oauth2Authentication = this.authenticationResultConverter
            .convert(authenticationResult);
    Assert.notNull(oauth2Authentication, "authentication result cannot be null");
    oauth2Authentication.setDetails(authenticationDetails);
    OAuth2AuthorizedClient authorizedClient = new OAuth2AuthorizedClient(
            authenticationResult.getClientRegistration(), oauth2Authentication.getName(),
            authenticationResult.getAccessToken(), authenticationResult.getRefreshToken());

    this.authorizedClientRepository.saveAuthorizedClient(authorizedClient, oauth2Authentication, request, response);
    return oauth2Authentication;
}
```

▶️ OAuth2LoginAuthenticationProvider - 토큰 발급요청 위임 및 loadUser() 호출
```java
@Override
public Authentication authenticate(Authentication authentication) throws AuthenticationException {
    OAuth2LoginAuthenticationToken loginAuthenticationToken = (OAuth2LoginAuthenticationToken) authentication;
    if (loginAuthenticationToken.getAuthorizationExchange().getAuthorizationRequest().getScopes()
            .contains("openid")) {
        return null;
    }
    OAuth2AuthorizationCodeAuthenticationToken authorizationCodeAuthenticationToken;
    try {
        // 토큰 받아오는 과정 - OAuth2AuthorizationCodeAuthenticationProvider 에게 위임
        authorizationCodeAuthenticationToken = (OAuth2AuthorizationCodeAuthenticationToken) this.authorizationCodeAuthenticationProvider
                .authenticate(new OAuth2AuthorizationCodeAuthenticationToken(
                        loginAuthenticationToken.getClientRegistration(),
                        loginAuthenticationToken.getAuthorizationExchange()));
    }
    catch (OAuth2AuthorizationException ex) {
        OAuth2Error oauth2Error = ex.getError();
        throw new OAuth2AuthenticationException(oauth2Error, oauth2Error.toString(), ex);
    }
    OAuth2AccessToken accessToken = authorizationCodeAuthenticationToken.getAccessToken();
    Map<String, Object> additionalParameters = authorizationCodeAuthenticationToken.getAdditionalParameters();

    // 1) OAuth2UserService<OAuth2UserRequest, OAuth2User> 의 loadUser() 메서드 호출
    OAuth2User oauth2User = this.userService.loadUser(new OAuth2UserRequest(
            loginAuthenticationToken.getClientRegistration(), accessToken, additionalParameters));
    Collection<? extends GrantedAuthority> mappedAuthorities = this.authoritiesMapper
            .mapAuthorities(oauth2User.getAuthorities());
    OAuth2LoginAuthenticationToken authenticationResult = new OAuth2LoginAuthenticationToken(
            loginAuthenticationToken.getClientRegistration(), loginAuthenticationToken.getAuthorizationExchange(),
            oauth2User, mappedAuthorities, accessToken, authorizationCodeAuthenticationToken.getRefreshToken());
    authenticationResult.setDetails(loginAuthenticationToken.getDetails());
    return authenticationResult;
}
```

- 1)번 과정에서 OAuth2UserService<OAuth2UserRequest, OAuth2User> 의 loadUser() 메서드가 호출되는 걸 확인할 수 있는데, 보통 이걸 커스텀해서 서비스에 맞는 로그인 or 회원가입 절차를 구현한다.

▶️ OAuth2AuthorizationCodeAuthenticationProvider - Authorization Server에 토큰 발급 요청
```java
@Override
public Authentication authenticate(Authentication authentication) throws AuthenticationException {
    OAuth2AuthorizationCodeAuthenticationToken authorizationCodeAuthentication = (OAuth2AuthorizationCodeAuthenticationToken) authentication;
    OAuth2AuthorizationResponse authorizationResponse = authorizationCodeAuthentication.getAuthorizationExchange()
            .getAuthorizationResponse();
    if (authorizationResponse.statusError()) {
        throw new OAuth2AuthorizationException(authorizationResponse.getError());
    }
    OAuth2AuthorizationRequest authorizationRequest = authorizationCodeAuthentication.getAuthorizationExchange()
            .getAuthorizationRequest();
    if (!authorizationResponse.getState().equals(authorizationRequest.getState())) {
        OAuth2Error oauth2Error = new OAuth2Error(INVALID_STATE_PARAMETER_ERROR_CODE);
        throw new OAuth2AuthorizationException(oauth2Error);
    }
    // 토큰 받아오기
    OAuth2AccessTokenResponse accessTokenResponse = this.accessTokenResponseClient.getTokenResponse(
            new OAuth2AuthorizationCodeGrantRequest(authorizationCodeAuthentication.getClientRegistration(),
                    authorizationCodeAuthentication.getAuthorizationExchange()));

    // Authentication 타입의 객체인 OAuth2AuthorizationCodeAuthenticationToken를 만들어 리턴
    OAuth2AuthorizationCodeAuthenticationToken authenticationResult = new OAuth2AuthorizationCodeAuthenticationToken(
            authorizationCodeAuthentication.getClientRegistration(),
            authorizationCodeAuthentication.getAuthorizationExchange(), accessTokenResponse.getAccessToken(),
            accessTokenResponse.getRefreshToken(), accessTokenResponse.getAdditionalParameters());
    authenticationResult.setDetails(authorizationCodeAuthentication.getDetails());
    return authenticationResult;
}
```

▶️ 
```java
@Override
public OAuth2AccessTokenResponse getTokenResponse(
        OAuth2AuthorizationCodeGrantRequest authorizationCodeGrantRequest) {
    Assert.notNull(authorizationCodeGrantRequest, "authorizationCodeGrantRequest cannot be null");
    // 토큰 요청 url 가진 request 생성
    RequestEntity<?> request = this.requestEntityConverter.convert(authorizationCodeGrantRequest);
    // 내부적으로 restTemplate사용해서 토큰을 발급받는다.
    ResponseEntity<OAuth2AccessTokenResponse> response = getResponse(request);
    OAuth2AccessTokenResponse tokenResponse = response.getBody();
    Assert.notNull(tokenResponse,
            "The authorization server responded to this Authorization Code grant request with an empty body; as such, it cannot be materialized into an OAuth2AccessTokenResponse instance. Please check the HTTP response code in your server logs for more details.");
    return tokenResponse;
}
```
