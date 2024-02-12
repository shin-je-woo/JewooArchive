# 💡 OAuth2AuthorizationRequestRedirectFilter
- `OAuth2AuthorizationRequestRedirectFilter` 는 `/oauth2/authorization/{registrationId}` 로 들어온 요청을 처리하는 필터이다.
- Authorization Server의 로그인 페이지로 redirect 시켜주는 역할을 수행한다.
- 이 때, 리다이렉션될 uri를 만드는 역할은 `OAuth2AuthorizationRequestResolver` 가 수행하게 된다.

# 💡 동작 과정 분석
![image](https://github.com/shin-je-woo/TIL/assets/39439576/e73401d7-94f4-41ae-8ff3-bdb0e0691fc9)

- 필터를 처음 실행하면, `OAuth2AuthorizationRequestResolver` 의 `resolve()` 메서드를 실행해서 `OAuth2AuthorizationRequest` 객체를 얻어온다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/02ee1ec1-31f9-4b8a-80c8-40dddcd15e5e)

- `OAuth2AuthorizationRequestResolver` 는 `/oauth2/authorization/{registrationId}` 에서 registrationId를 추출한다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/678dc734-d0aa-43c4-bd2b-e52f8396e12a)

- 추출한 `registrationId` 로 `OAuth2AuthorizationRequest` 객체를 만든다.
- 이 떄, `ClientRegistration` 객체는 application.yml에 등록한 Authorization Server 정보이다.
- `OAuth2AuthorizationRequest` 객체를 통해 `OAuth2AuthorizationRequestRedirectFilter` 에서 `sendRedirectForAuthorization(request, response, authorizationRequest);` 를 호출한다.

#### 📌 ClientRegistration 주요 정보

![image](https://github.com/shin-je-woo/TIL/assets/39439576/4781e6a4-f0b6-4ed5-bdab-8b008de67d23)

- authorizationUri: Authorization Server의 uri (네이버 경우 : https://nid.naver.com/oauth2.0/authorize)
- authorizationRequestUri: authorizationUri + 쿼리 파라미터
- 쿼리 파라미터로 넘겨지는 값
  - responseType (보통은 code)
  - clientId
  - state
  - scope
  - redirectUri (Authorization Server의 로그인 과정 이후 redirec될 uri (client는 해당 url로 code값을 포함해 redirect받는다.)
