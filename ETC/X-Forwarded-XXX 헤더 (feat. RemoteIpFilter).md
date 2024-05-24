# Previous

애플리케이션 개발을 하다보면 nginx 를 WAS의 프록시 용도로 사용할 때가 있다.

nginx.conf 와 같은 설정 파일에서 proxy 설정 시 아래와 같은 설정을 할 경우가 있는데, proxy에서는 요청헤더에 아래의 값을 담아 서버로 전달하게 된다.

```
proxy_set_header        Host $host;
proxy_set_header        X-Real-IP $remote_addr;
proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header        X-Forwarded-Proto $scheme;
```

# 💡 X-Forwarded-XXX 헤더

## [X-Forwarded-For](https://developer.mozilla.org/ko/docs/Web/HTTP/Headers/X-Forwarded-For)

HTTP 프록시나 로드 밸런서를 통해 웹 서버에 접속하는 클라이언트의 원 IP 주소를 식별하는 HTTP 요청 헤더 중 하나이다.

클라이언트와 서버 중간에서 트래픽이 프록시나 로드 밸런서를 거치면, 서버 접근 로그에는 프록시나 로드 밸런서의 IP 주소만을 담고 있어서  클라이언트의 원 IP 주소를 보기 위해서  `X-Forwarded-For` 요청 헤더가 사용된다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/2b377332-0740-45e8-8f05-896847141252)

```
X-Forwarded-For: <client>, <proxy1>, <proxy2>
```

하나의 요청이 여러 프록시들을 거치면, 각 프록시의 IP 주소들이 차례로 열거된다.

즉, 가장 오른쪽 IP 주소는 가장 마지막에 거친 프록시의 IP 주소이고, 가장 왼쪽의 IP 주소는 최초 클라이언트의 IP 주소다.

위 그림처럼 프록시와 로드 밸런서를 거친 경우, MY-SERVER의 `X-Forwarded-For` 헤더 값은 아래와 같다.

```
X-Forwarded-For: 211.12.14.10, 10.20.11.19, 10.20.11.17
```

## [X-Forwarded-Proto](https://developer.mozilla.org/ko/docs/Web/HTTP/Headers/X-Forwarded-Proto)

클라이언트가 프록시 또는 로드 밸런서에 접속하는데에 사용했던 프로토콜(HTTP 또는 HTTPS)이 무엇인지 확인하는 HTTP 요청 헤더 중 하나이다.

서버 접근 로그들은 서버와 로드 밸런서 사이에서 사용된 프로토콜을 포함하고 있지만, 클라이언트와 로드밸런서에 사용한 프로토콜은 포함되어 있지 않다.

클라이언트와 로드밸런서 간의 사용된 프로토콜을 확인하기 위해서 `X-Forwarded-Proto` 요청 헤더가 사용된다.

```
X-Forwarded-Proto: <protocol>
```

클라이언트가 https 프로토콜을 사용하여 요청한 경우, `X-Forwarded-Proto` 헤더값은 아래와 같다.

```
X-Forwarded-Proto: https
```

## [X-Forwarded-Host](https://developer.mozilla.org/ko/docs/Web/HTTP/Headers/X-Forwarded-Host)

클라이언트가 요청한 원래 Host 헤더를 식별하는 HTTP 요청 헤더 중 하나이다.

리버스 프록시(로드밸런서, CDN) 에서 Host 이름과 포트는 요청을 처리 하는 Origin 서버와 다를 수 있는데 이러한 경우 원래 사용된 Host 를 확인 하려면 `X-Forwarded-Host` 요청 헤더가 사용된다.

```
X-Forwarded-Host: <host>
```

https://github.com/shin-je-woo 로 요청한 경우, `X-Forwarded-Host` 값은 아래와 같다.

```
X-Forwarded-Host: https://github.com/shin-je-woo
```

# 💡 RemoteIpFilter

Tomcat에서는 `X-Forwarded-XXX` 헤더를 어떻게 해석하고 사용할까?

톰캣에서는 `RemoteIpFilter` 라는 서블릿 필터에서 `X-Forwarded-XXX` 라는 헤더를 읽어와서 request 정보를 바꿔치기 한다.

해당 설정은 톰캣의 server.xml 에 다음과 같이 `RemoteIpValve` 설정을 통해 값을 매칭시켜줘야 한다.

▶️ RemoteIpValve 설정

```
<Valve className="org.apache.catalina.valves.RemoteIpValve"
			  remoteIpHeader="x-forwarded-for"
			  proxiesHeader="x-forwarded-by"
			  protocolHeader="x-forwarded-proto" />
```

▶️ RemoteIpFilter

```java
package org.apache.catalina.filters;

public class RemoteIpFilter extends GenericFilter {

    //...
    private String protocolHeader = "X-Forwarded-Proto";
  
    //...
    public void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
            throws IOException, ServletException {

        //... 
        XForwardedRequest xRequest = new XForwardedRequest(request);
        
        //...
        
        if (protocolHeader != null) {
            String protocolHeaderValue = request.getHeader(protocolHeader);
            if (protocolHeaderValue == null) {
                // Don't modify the secure, scheme and serverPort attributes
                // of the request
            } else if (isForwardedProtoHeaderValueSecure(protocolHeaderValue)) {
                xRequest.setSecure(true);
                xRequest.setScheme("https");
                setPorts(xRequest, httpsServerPort);
            } else {
                xRequest.setSecure(false);
                xRequest.setScheme("http");
                setPorts(xRequest, httpServerPort);
            }
        }
        
        chain.doFilter(xRequest, response);
        //...
    }

    private boolean isForwardedProtoHeaderValueSecure(String protocolHeaderValue) {
        if (!protocolHeaderValue.contains(",")) {
            return protocolHeaderHttpsValue.equalsIgnoreCase(protocolHeaderValue);
        }
        String[] forwardedProtocols = commaDelimitedListToStringArray(protocolHeaderValue);
        if (forwardedProtocols.length == 0) {
            return false;
        }
        for (String forwardedProtocol : forwardedProtocols) {
            if (!protocolHeaderHttpsValue.equalsIgnoreCase(forwardedProtocol)) {
                return false;
            }
        }
        return true;
    }

    //...
}
```

`RemoteIpFilter` 에서는 `X-Forwarded-XXX` 헤더가 있을 경우(프록시나 로드밸런서에서 넘겨준 헤더겠죠?) XForwardedRequest라는 새로운 객체에 값을 담아둔다.

이후, 다음 필터체인을 호출할 때 원본 `HttpServletRequest` 가 아닌 새로 세팅한 `XForwardedRequest`로 바꿔치기해서 넘겨주게 된다.

그럼 이후에 호출되는 필터, 서블릿들에서는 (대표적으로 스프링에서는 DelegatingFilterProxy, DispatcherServlet) `X-Forwarded-XXX` 헤더가 해석된 `XForwardedRequest` 를 사용하게 된다.

참고로, Spring Boot는 내장 톰캣을 사용하는데, 스프링 부트의 Auto Configuration 기능에 의해 위의 설정들이 기본적으로 세팅된다.

▶️ 스프링 부트의 TomcatWebServerFactoryCustomizer

```java
package org.springframework.boot.autoconfigure.web.embedded;

public class TomcatWebServerFactoryCustomizer
		implements WebServerFactoryCustomizer<ConfigurableTomcatWebServerFactory>, Ordered {

    //...

    private void customizeRemoteIpValve(ConfigurableTomcatWebServerFactory factory) {
        ServerProperties.Tomcat.Remoteip remoteIpProperties = this.serverProperties.getTomcat().getRemoteip();
        String protocolHeader = remoteIpProperties.getProtocolHeader();
        String remoteIpHeader = remoteIpProperties.getRemoteIpHeader();
        // For back compatibility the valve is also enabled if protocol-header is set
        if (StringUtils.hasText(protocolHeader) || StringUtils.hasText(remoteIpHeader)
                || getOrDeduceUseForwardHeaders()) {
            RemoteIpValve valve = new RemoteIpValve();
            valve.setProtocolHeader(StringUtils.hasLength(protocolHeader) ? protocolHeader : "X-Forwarded-Proto");
            if (StringUtils.hasLength(remoteIpHeader)) {
                valve.setRemoteIpHeader(remoteIpHeader);
            }
            valve.setTrustedProxies(remoteIpProperties.getTrustedProxies());
            // The internal proxies default to a list of "safe" internal IP addresses
            valve.setInternalProxies(remoteIpProperties.getInternalProxies());
            try {
                valve.setHostHeader(remoteIpProperties.getHostHeader());
            } catch (NoSuchMethodError ex) {
                // Avoid failure with war deployments to Tomcat 8.5 before 8.5.44 and
                // Tomcat 9 before 9.0.23
            }
            valve.setPortHeader(remoteIpProperties.getPortHeader());
            valve.setProtocolHeaderHttpsValue(remoteIpProperties.getProtocolHeaderHttpsValue());
            // ... so it's safe to add this valve by default.
            factory.addEngineValves(valve);
        }
    }
}
```
