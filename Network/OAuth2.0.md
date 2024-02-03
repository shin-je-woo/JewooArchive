# 💡 OAuth2.0이란?
- Open Authorization 2.0 혹은 OAuth2.0은 웹 및 애플리케이션 인증 및 권한 부여를 위한 개방형 표준 프로토콜이다.
- Third-Party Application이 사용자의 리소스에 접근하기 위한 절차를 정의하고 서비스 제공자의 API를 사용할 수 있는 권한을 부여한다.
- OAuth 2.0은 다양한 클라이언트 환경에 적합한 인증(Authentication) 및 인가(Authorization)의 부여(위임) 방법(Grant Type)을 제공하고, 그 결과로 클라이언트에 접근 토큰(Access Token)을 발급하는 것에 대한 구조(framework)이다.
- 대표적으로 네이버 로그인, 구글 로그인과 같은 소셜 미디어 간편 로그인이 있다.

# 💡 OAuth2.0 역할
### 리소스 소유자(Resource Owner)
- OAuth 2.0 프로토콜을 사용하여 보호되는 리소스에 대한 액세스 권한을 부여하는 사용자이다.
- 클라이언트를 인증(Authorize)하는 역할을 수행한다.
- 예를 들어 네이버 로그인에서 네이버 아이디를 소유하고 Third-Party Application(클라이언트)에 네이버 아이디로 소셜 로그인 인증을 하는 사용자를 의미한다.
- 개념적으로는 Resource Owner가 자격을 부여하는 것이지만, 일반적으로 권한 서버(Authorization Server)가 Resource Owner와 Client 사이에서 중개 역할을 수행하게 된다.

### 클라이언트(Client)
- OAuth 2.0을 사용하여 리소스에 접근하려는 Third-Party Application이나 서비스이다.
- 보통 우리가 개발하려는 서비스가 된다.

### 권한 서버(Authorization Server)
- 권한 서버는 클라이언트가 리소스 소유자의 권한을 얻을 수 있도록 도와주는 서버이다.
- Client의 접근 자격을 확인하고 권한을 부여하거나, Access Token을 발급해 주는 역할을 수행한다.

### 리소스 서버(Resource Server)
- 리소스 서버는 보호되는 리소스를 호스팅하는 서버로, 액세스를 허용하거나 거부한다.
- 이 서버는 OAuth 2.0 토큰을 사용하여 클라이언트에게 리소스에 액세스할 권한을 부여하고 실제 데이터를 제공합니다.

# 💡 인증 방식 및 동작 과정
- OAuth 2.0에서는 클라이언트 애플리케이션이 리소스 서버에 접근할 권한을 부여하고 관리하기 위해 다양한 권한 부여 방식이 제공된다.
- 이 중 가장 보편적으로 사용되는 방식인 인증 코드 부여 방식에 대해서만 자세히 다룬다.

###  Authorization Code Grant / 권한 부여 코드 승인 방식
- 권한 부여 코드 요청 시, 자체 생성한 Authorization Code를 전달하는 방식으로 response_type=code, grant_type=authorization_code의 형식으로 요청한다.
- OAuth2에서 가장 기본이 되는 방식이며, 간편 로그인 기능에서 사용되는 방식이다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/81fe8037-ab5c-4bac-a9e1-f040512da26f)
- 권한 부여 코드 요청 시, response_type=code로 요청하게 되면 Client는 권한 서버(Authorization Server)에서 제공하는 로그인 페이지로 리다이렉트 시킨다.
- Resource Owner가 로그인을 하면 Authorization Server는 권한 부여 코드 요청 시 전달받은 redirect_uri로 Authorization Code를 담아 리다이렉트 시킨다.
- Client는 전달받은 Authorization Code를 Authorization Server의 API를 통해 Access Token으로 교환한다.
- 받아온 Access Token으로 Resource Server에 보호된 자원을 요청한다.

### 📌 Authorization Code는 왜 필요한가?
- 만약 Authorization Code를 발급받는 과정이 생략되고 Access Token을 바로 발급 받는다면, Authorization Server는 Access Token을 전달하기 위한 방법으로 권한 부여 코드 요청 시 전달받은 redirect_uri을 사용할 수밖에 없다.
- Redirect URI을 통해 데이터를 전달하는 방법은 URI 자체에 데이터를 실어 전달하는 방법 밖에 없으며, 이 방법을 사용하면 중요한 데이터인 Access Token이 브라우저에 바로 노출되게 된다.
- 이러한 문제를 보완하기 위해 Authorization Code 발급 과정이 추가되었다.
- 보통 개발을 하게 되면 프론트엔드에서 Authorization Code를 백엔드의 토큰발급 엔드포인트로 전달하고, 백엔드에서 Authorization Server의 API로 토큰발급을 요청한다.
- 이 과정을 통해 백엔드 사이에서 Access Token이 전달되기 때문에 액세스 토큰의 탈취 위험이 줄어드는 등, 다른 방식에 비해 보안적으로 안전하다는 특징이 있다.
