# 💡 OAuth2AuthorizationRequestRedirectFilter
- `OAuth2AuthorizationRequestRedirectFilter` 는 `/oauth2/authorization/{registrationId}` 로 들어온 요청을 처리하는 필터이다.
- Authorization Server의 로그인 페이지로 redirect 시켜주는 역할을 수행한다.
- 이 때, 리다이렉션될 uri를 만드는 역할은 `OAuth2AuthorizationRequestResolver` 가 수행하게 된다.

# 💡 동작 과정 분석
![image](https://github.com/shin-je-woo/TIL/assets/39439576/e73401d7-94f4-41ae-8ff3-bdb0e0691fc9)

- 필터를 처음 실행하면, `OAuth2AuthorizationRequestResolver` 의 `resolve()` 메서드를 실행해서 `OAuth2AuthorizationRequest` 객체를 얻어온다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/02ee1ec1-31f9-4b8a-80c8-40dddcd15e5e)

- `OAuth2AuthorizationRequestResolver` 는 `/oauth2/authorization/{registrationId}` 
