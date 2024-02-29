# 💡 MockMvc 란
- 스프링 프레임워크에서 제공하는 웹 애플리케이션 테스트용 라이브러리를 의미.
- MockMvc를 사용하면 HTTP 요청을 작성하고 컨트롤러의 응답을 검증할 수 있다.
- 즉, 웹 애플리케이션을 서버에 배포하지 않고도 스프링 mvc의 동작을 재현할 수 있다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/5d1f7851-80ed-45ac-8e1e-c2f633a90643)

1. TestCase → MockMvc  
    - TestCase 내에서 MockMvc 객체를 생성한다.
    - 이 객체는 테스트할 컨트롤러와 상호작용을 하는 데 사용된다.

2. MockMvc → TestDispatcher Servlet
    - MockMvc를 사용하여 원하는 엔드포인트에 요청을 보낸다. (해당 요청에 필요한 파라미터, 헤더 또는 쿠키 등을 설정할 수 있다.)
    - 예를 들어, GET 요청을 보내고 싶다면 perform(MockMvcRequestBuilders.get("/endpoint"))와 같이 요청을 설정한다.
    - 파라미터 설정은 param("paramName", "paramValue")와 같이 설정할 수 있다.

3. TestDispatcher Servlet → Controller
    - 요청을 수행할 컨트롤러로 요청을 보낸다.

4. MockMvc → TestCase
    - andExpect 메서드를 사용하여 응답의 상태코드, 헤더, 본문 등을 검증할 수 있다.
    - 예를 들어, 응답 본문의 내용을 검증하고 싶다면 andExpect(content(). string("expectedValue"))와 같이 검증한다.
  
# 💡 MockMvc 관련 주요 API

### MockMvc - perform() 메소드
- [JavaDoc - MockMvc](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/web/servlet/MockMvc.html#method.summary)
- MockMvc 관련 자바독을 보면 MockMvc는 perform() 이라는 메소드를 지원하고 있는 것을 볼 수 있다.
- perform() 메소드의 주요 역할은 다음과 같다.
  - DispatcherServlet에 요청을 의뢰하는 역할을 한다.
  - `RequestBuilder` 인터페이스를 인수로 전달한다.
  - `ResultActions` 이라는 인터페이스를 반환한다.

### ResultActions - andExpect() 메소드
- [JavaDoc - ResultActions](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/web/servlet/ResultActions.html)
