# 💡 DelegatingFilterProxy
![image](https://github.com/shin-je-woo/TIL/assets/39439576/a5486f43-0bba-421c-b95b-5b438e3ce187)

- 서블릿 컨테이너는 자체 표준을 사용하여 필터 인스턴스를 등록할 수 있지만 Spring에서 정의한 Bean을 인식하지 못하는 문제가 있다. (서블릿 컨테이너에서 Spring을 알지 못한다.)
- `DelegatingFilterProxy`는 스프링빈을 인식할 수 있는 필터로 Spring이 제공하는 클래스이다. (스프링 시큐리티 스팩 아님)
- `DelegatingFilterProxy`는 서블릿 컨테이너의 라이프사이클과 Spring의 ApplicationContext 사이를 연결하는 필터 구현을 제공한다. (서블릿 컨테이너와 스프링 컨테이너의 다리 역할)
- 참고로 위 그림의 `FilterChain`은 Servlet Container가 관리하는 `ApplicationFilterChain`이다. (`SecurityFilterChain`과 다르다.)
- 정리하면, `DelegatingFilterProxy`는 서블릿 필터이며, Spring 컨테이너가 관리는 Filter Bean을 가지고 있고, 이 Filter Bean은 `FilterChainProxy`이며 이 객체안에서 스프링 시큐리티 역할을 수행한다.

### DelegatingFilterProxy는 어떻게 등록될까?
```java
@AutoConfiguration(after = SecurityAutoConfiguration.class)
@ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET)
@EnableConfigurationProperties(SecurityProperties.class)
@ConditionalOnClass({ AbstractSecurityWebApplicationInitializer.class, SessionCreationPolicy.class })
public class SecurityFilterAutoConfiguration {

    private static final String DEFAULT_FILTER_NAME = AbstractSecurityWebApplicationInitializer.DEFAULT_FILTER_NAME;
    // DEFAULT_FILTER_NAME = "springSecurityFilterChain"

    @Bean
    @ConditionalOnBean(name = DEFAULT_FILTER_NAME)
    public DelegatingFilterProxyRegistrationBean securityFilterChainRegistration(
            SecurityProperties securityProperties) {
        DelegatingFilterProxyRegistrationBean registration = new DelegatingFilterProxyRegistrationBean(
                DEFAULT_FILTER_NAME);
        registration.setOrder(securityProperties.getFilter().getOrder());
        registration.setDispatcherTypes(getDispatcherTypes(securityProperties));
        return registration;
    }

    //...
}
```
- `DelegatingFilterProxyRegistrationBean`을 생성하는 조건으로 `@ConditionalOnBean(name = DEFAULT_FILTER_NAME)`이 설정되어 있다.
- 즉, "springSecurityFilterChain" 라는 이름을 가진 스프링빈이 등록되어 있어야 하는데, 이게 바로 `FilterChainProxy`이다.

# 💡 FilterChainProxy
- `FilterChainProxy`는 `SecurityFilterChain`을 통해 필터들에게 보안을 위임하는 필터이다.
- `FilterChainProxy`는 스프링 Bean이므로 일반적으로 `DelegatingFilterProxy`에 래핑된다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/1df44130-0358-4b08-b6fe-ebe417436941)

# 💡 요청처리 과정
![image](https://github.com/shin-je-woo/TIL/assets/39439576/ade30b85-602c-4a8c-b435-b835b0ca8e1b)
- HTTP요청이 들어오면 Servlet 컨테이너에서 요청을 받는다.
- `DelegatingFilterProxy` Filter가 요청을 받으면 `FilterChainProxy` 빈을 스프링 컨테이너에서 찾고, 요청을 전달한다.
- `FilterChainProxy`는 `SecurityFilterChain`에 등록된 각 필터들에게 보안처리를 위임한다.
- 모든 필터가 동작한 후에 `DeispatcherServlet`에 요청을 전달한다.



