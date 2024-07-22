# Previous
* 진행하던 프로젝트에서 CustomException을 정의했는데, 이상하게 @ExceptionHandler가 작동하지 않는 문제가 발생했다.
* `HandlerExceptionResovler`를 살펴보다가 발견한 내용이 있어 기록한다.

# 문제상황
* 비즈니스 로직 오류처리를 위한 커스텀Exception을 하나 만들었다.
* 해당 오류가 발생하면 오류내용 발생원인과 예외메시지를 response body에 넣고, HttpStatus를 200으로 처리해줄 생각이었다.
* 오류처리용 클래스에 `@RestControllerAdvice` 어노테이션을 달고, 오류처리용 메서드에 `@ExceptionHandler` 어노테이션을 달아주었다.
* 다음은 작성했던 코드를 재구성한 코드이다. (사내코드여서 단순화한 코드를 예시로 들겠다.)
```java
@RestControllerAdvice
public class BioCustomExceptionHandler {

    @ResponseStatus(HttpStatus.OK)
    @ExceptionHandler(BioCustomException.class)
    public BioResponse<?> handleBioCustomException(BioCustomException e) {
        ...
        // 오류내용 발생원인 및 예외메시지 포맷팅 and JSON return
    }
}
```
* 컨트롤러에서 `BioCustomException` 예외를 던지면 `HandlerExceptionResovler`를 구현한 `ExceptionHandlerExceptionResolver` 에서 handleBioCustomException 메서드를 호출해주기를 기대했다.
* 그런데, 호출되지 않는 문제가 있어 왜 호출되지 않는지 궁금해졌다.
* 어디선가 Exception을 잡아서 처리하고 있으리라고 짐작은 되었으나, 직접 코드로 확인해보고 싶었다.

# 분석내용
* 먼저 다이어그램을 보자.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/c65c72bf-7fb8-487f-9f2b-9469e940e71e)
* `ExceptionHandlerExceptionResolver` 는 `HandlerExceptionResovler` 를 구현하고 있다.
* `HandlerExceptionResovler` 인터페이스는 `HandlerExceptionResolverComposite` 에 다양한 ExceptionResolver를 가지고 있다.
  * `ExceptionHandlerExceptionResolver` : 가장 높은 우선순위. `@ExceptionHandler` 를 처리한다. (이 글의 분석대상)
  * `ResponseStatusExceptionResolver`
  * `DefaultHandlerExceptionResolver`

#### ExceptionHandlerExceptionResolver는 @ExceptionHandler를 어떻게 호출할까?
▶️ ExceptionHandlerExceptionResolver 구현체의 getExceptionHandlerMethod 메서드
```java
protected ServletInvocableHandlerMethod getExceptionHandlerMethod(HandlerMethod handlerMethod, Exception exception) {
    Class<?> handlerType = (handlerMethod != null ? handlerMethod.getBeanType() : null);

    if (handlerMethod != null) {
      ExceptionHandlerMethodResolver resolver = this.exceptionHandlerCache.get(handlerType);
      if (resolver == null) {
        resolver = new ExceptionHandlerMethodResolver(handlerType);
        this.exceptionHandlerCache.put(handlerType, resolver);
      }
      Method method = resolver.resolveMethod(exception);
      if (method != null) {
        return new ServletInvocableHandlerMethod(handlerMethod.getBean(), method);
      }
    }

    for (Entry<ControllerAdviceBean, ExceptionHandlerMethodResolver> entry : this.exceptionHandlerAdviceCache.entrySet()) {
      if (entry.getKey().isApplicableToBeanType(handlerType)) {
        ExceptionHandlerMethodResolver resolver = entry.getValue();
        Method method = resolver.resolveMethod(exception);
        if (method != null) {
          return new ServletInvocableHandlerMethod(entry.getKey().resolveBean(), method);
        }
      }
    }

    return null;
  }

}
```
```java
private final Map<Class<?>, ExceptionHandlerMethodResolver> exceptionHandlerCache =
        new ConcurrentHashMap<Class<?>, ExceptionHandlerMethodResolver>(64);

private final Map<ControllerAdviceBean, ExceptionHandlerMethodResolver> exceptionHandlerAdviceCache =
        new LinkedHashMap<ControllerAdviceBean, ExceptionHandlerMethodResolver>();
```
* `exceptionHandlerCache` 에서 처리할 메서드를 확인한다. 클래스(보통은 컨트롤러)에 `@ExceptionHandler` 어노테이션이 붙은 메서드를 가지고 있다.
* 처리할 메서드가 컨트롤러에 정의되어 있으면 해당 컨트롤러의 `@ExceptionHandler` 어노테이션이 붙은 메서드를 호출한다.
* 처리할 메서드가 없다면 `exceptionHandlerAdviceCache` 에서 처리할 메서드를 확인한다. `@ControllerAdvice` 가 붙은 클래스의 `@ExceptionHandler` 메서드를 가지고 있다.
* 정리하면, 우선순위는 1. 클래스레벨의 @ExceptionHandler -> 2. @ControllerAdvice레벨의 @ExceptionHandler 인 것이다.
* 이렇게 구한 HandlerMethod를 invoke한다.

#### 사내 코드 분석
* 위 분석 코드를 디버깅해보니, 에러가 발생했을 때 내가 정의한 BioCustomException을 처리할 수 있는 `exceptionHandlerCache` 값이 존재했다.
* 사내코드를 보니 상속받은 공통 컨트롤러에서 `UncheckedException` 을 처리하는 `@ExceptionHandler` 가 존재했다.
* 따라서, `UncheckedException` 을 상속받는 나의 커스텀익셉션이 발생했을 때 이 메서드에서 처리되고 있었다.
* ❗ 분석 내용대로 클래스레벨의 @ExceptionHandler가 @ControllerAdvice의 @ExceptionHandler보다 우선순위가 높기 때문에 내가 만든 @ControllerAdvice가 동작하지 않았던 것이다.

# 💡 해결
* `@ExceptionHandler` 를 통해 `UncheckedExcepion` 을 처리하는 공통 컨트롤러는 사내 Maven에서 관리되고 있었다.
* 해당 컨트롤러에는 UncheckedExcepion 외에도 사내에서 정의한 Exception을 처리할 수 있는 메서드들이 있었는데, 내가 구현하고자 하는 예외처리코드와는 거리가 있었다.
* 사내 Maven에서 관리되는 사내용 라이브러리를 내가 고칠 수는 없는 입장이어서 분석 내용대로 내가 만든 Api컨트롤러에서 `@ExceptionHandler` 를 정의해주었다.
* `@ControllerAdvice` 를 이용해 글로벌하게 나의 커스텀익셉션을 처리할 수는 없지만, 커스텀익셉션이 발생하는 컨트롤러가 2개여서 어느정도 타협을 하기로 했다.
* **이 내용으로 보고를 올리고, 다음 버전에서 에러 처리 로직이 @ControllerAdvice가 적용된 별도의 에러 처리 클래스로 이동되었음을 확인했다.**
