# Previous
* 최근 진행하고 있는 프로젝트에서 API를 만들던 중 겪었던 문제를 해결하는 과정을 작성하려고 한다.
* 사내 패키지는 자바8의 날짜와 시간API가 아닌 java.util.Date를 사용하고 있었다.

# 문제 상황
* 아래와 같이 리스트를 출력하는 평범한 GET API를 작성하였다.
```java
@ResponseBody
@ResponseStatus(HttpStatus.OK)
@GetMapping(value = "/excludes", produces = MediaType.APPLICATION_JSON_VALUE)
public BioResponse<List<BioExcludeResponseDTO>> excludeList() {

   List<BioExcludeResponseDTO> bioExcludeList = bioExcludeDAO.findAll();

   return new BioResponse<>(true, bioExcludeList);
}
```
* 고민하게 만들었던 BioExcludeResponseDTO를 살펴보자.
```java
@Getter
@NoArgsConstructor(access = AccessLevel.PRIVATE)
@Builder @AllArgsConstructor(access = AccessLevel.PRIVATE)
public class BioExcludeResponseDTO {

    private String bioExcludeId;
    private String empName;
    private String empPosName;

    @JsonFormat(shape=JsonFormat.Shape.STRING, pattern="yyyy-MM-dd")
    private Date bgnDt;

    @JsonFormat(shape=JsonFormat.Shape.STRING, pattern="yyyy-MM-dd")
    private Date endDt;
}
```
* 위 코드와 같이 `com.fasterxml.jackson.annotation.JsonFormat` 을 사용하면 당연히 Date포맷이 원하는대로 지정될 줄 알았다.
* 그러나, 결과를 보니 적용되지 않았다.

![image](https://user-images.githubusercontent.com/39439576/236819770-811df5d8-8c0f-492d-9c26-1be9030e762a.png)

# 원인
* 디버깅으로 따라가 보니 설정용 스프링빈에서 사내 라이브러리를 이용해 JSON직렬화될 때 Date타입의 필드들은 정해진 대로만 직렬화되고 있었다.
![image](https://user-images.githubusercontent.com/39439576/236823847-f788be4f-2340-42c3-b040-c42de94e85a0.png)
* 문제는, 사내 라이브러리에서 @JsonFormat과 같은 애노테이션 고려 없이 설정한 값으로만 직렬화를 수행하고 있었다.
* 사내에서는 이 경우를 대비해 특정 클래스를 Controller에서 상속받거나 컴포지션으로 구성해서 특정 메소드를 호출하면 DateFormat을 지정할 수 있긴 했다.
* 하지만, 이 경우에도 모든 필드에 동일한 Format이 적용되었고, 나는 이것이 유연성이 너무 떨어진다고 생각했다.
* 예를 들어, BioExcludeResponseDTO에서 bgnDt와 endDt의 DateFormat이 달라야 한다면? 여기서 Date필드가 더 추가되어 다른 Format으로 출력해야 한다면?
* 또한, 사내 라이브러리는 정해진 규격이 있어서 해당 방식을 이용하려면 HttpServletRequest와 HttpServletResponse를 API의 파라미터로 무조건 받아야 했다. 내 API에서는 관심대상이 전혀 아님에도 말이다.
* 현재 방식으로는 위 2가지 문제를 해결하지 못했다.

# 해결과정
* 사실, 사내의 라이브러리를 애노테이션을 고려하도록 수정하면 기존의 소스코드들의 수정 없이 기능을 확장할 수 있었다.
* 하지만, 사내 라이브러리는 연구소에서부터 배포되어 이 내용을 건의하고 검토하고 다시 배포되기까지 너무 오래 걸린다고 판단했다.
* 그래서 내가 작성하고 있는 API에서는 사내 배포된 Serializer가 아닌 내가 만든 Serializer를 구현체로 사용하도록 해야 했다.

### JsonSerializer와 ContextualSerializer를 이용해 새로운 Date타입 Serializer만들기
* 먼저 사내 Serializer를 대체할 새로운 Date타입 Serializer를 만들어보자.
```java
public class CustomDateSerializer extends JsonSerializer<Date> implements ContextualSerializer {

    private final String dateFormat;

    private static final String[] allowedDateFormat = new String[]{"yyyy-MM-dd HH:mm:ss", "yyyy-MM-dd'T'HH:mm:ss", "yyyy-MM-dd'T'HH:mm:ss.SSSZ", "yyyy-MM-dd HH:mm", "yyyy-MM-dd HH", "yyyy-MM-dd"};

    private CustomDateSerializer() {
        this.dateFormat = "yyyy/MM/dd";
    }

    private CustomDateSerializer(final String dateFormat) {
        if (Arrays.stream(allowedDateFormat).noneMatch(dateFormat::equals)) {
            throw new BizException("not allowed date-format!!");
        }
        this.dateFormat = dateFormat;
    }

    @Override
    public void serialize(Date value, JsonGenerator gen, SerializerProvider serializers) throws IOException {
        SimpleDateFormat sdf = new SimpleDateFormat(this.dateFormat, Locale.ENGLISH);
        gen.writeString(sdf.format(GlobalTime.getUtcCalDate(value, true)));
    }


    @Override
    public JsonSerializer<?> createContextual(SerializerProvider prov, BeanProperty property) throws JsonMappingException {
        if (property.getAnnotation(CustomDateJsonFormat.class) == null) {
            return this;
        }
        return new CustomDateSerializer(property.getAnnotation(CustomDateJsonFormat.class).format());
    }
}
```
* 이 클래스의 핵심은 Override한 2개의 메소드 `serialize()`와 `createContextual()`이다.
* Jackson에서는 Date타입 필드를 직렬화 하기 전에 `ContextualSerializer`인터페이스의 `createContextual()`메소드가 먼저 호출되도록 설계되어 있다.
* 이 메소드는 런타임에 `BeanProperty`를 사용해 직렬화대상 객체의 여러가지 정보를 들여다볼 수 있다.
* (리플렉션처럼 private 기본생성자가 호출되는데, 내부적으로 리플렉션이 구현되어 있는지까지는 확인하지 못했다.)
* 그래서 이 메소드에서 특정 애노테이션(내가 작성할 Custom애노테이션)의 작성여부를 검사하도록 했다.
* 필드에 `CustomDateJsonFormat` 애노테이션이 있을 경우엔 애노테이션에서 작성한 값으로 dateFormat을 지정하도록 했고, 없을 경우 기본 값으로 "yyyy/MM/dd"를 사용하도록 했다.
* 이제 `createContextual`에서 사용할 Custom애노테이션을 살펴보자.
```java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface CustomDateJsonFormat {
    String format() default "yyyy-MM-dd";
}
```
* 내가 필요한 정보만 간단하게 작성했다.
* 현재는 필드레벨에서만 사용할 수 있게 작성했는데, 추후 시간이 되면 클래스레벨까지 올려볼 생각이다.
* 이제 내가 작성한 Custom애노테이션과 CustomSerializer를 적용해보자.
* 먼저 응답DTO를 아래와 같이 수정해주었다.
```java
@Getter
@NoArgsConstructor(access = AccessLevel.PRIVATE)
@Builder @AllArgsConstructor(access = AccessLevel.PRIVATE)
public class BioExcludeResponseDTO {

    private String bioExcludeId;
    private String empName;
    private String empPosName;

    @CustomDateJsonFormat // default로 "yyyy-MM-dd" 적용됨
    @JsonSerialize(using = CustomDateSerializer.class)
    private Date bgnDt;

    @JsonSerialize(using = CustomDateSerializer.class) // default로 "yyyy/MM/dd" 적용됨
    private Date endDt;
}
```

![image](https://user-images.githubusercontent.com/39439576/236824268-7c8e10ec-8245-41f3-98d2-33b56e4d82df.png)

# 정리
* 사내 라이브러리를 그대로 사용하면 원하는 필드만 클라이언트에서 dateFormat을 손봐야 하는 엄청나게 번거로운 작업이 발생할 수 있었다.
* 그리고, API의 관심사가 아닌 파라미터를 무조건 받아야 하는 문제점도 해결했다.
* 사내 라이브러리에서 제공하는 DateSerializer를 내가 작성한 DateSerializer를 참고해서 손보면 글로벌설정으로 애노테이션 없이 사용할 수 있다. (기존 코드를 수정하지 않고 확장성을 얻을 수 있다.)
* 추가로 dateFormat을 필드마다 애노테이션으로 간단하게 설정해줄 수 있는 유연성을 얻게 되었다.

### 참고사이트
* [[Java] 7. 어노테이션을 이용한 JSON 객체 Date 형식 변환(@JsonFormat, @DateTimeFormat)](https://linked2ev.github.io/java/2020/11/06/JAVA-7.-%EC%96%B4%EB%85%B8%ED%85%8C%EC%9D%B4%EC%85%98%EC%9D%84-%EC%9D%B4%EC%9A%A9%ED%95%9C-JSON-%EA%B0%9D%EC%B2%B4-Date-%ED%98%95%EC%8B%9D-%EB%B3%80%ED%99%98(@JsonFormat,-@DateTimeFormat)/)

* [Jackson Serialize 시 클래스 정보가 필요할때](https://multifrontgarden.tistory.com/206)

* [Jackson custom date serializer](https://stackoverflow.com/questions/27247767/jackson-custom-date-serializer)
