# 💡 스프링 Cache 추상화
- 애플리케이션을 개발하며 쓰기 동작보다 읽기 동작이 많은 데이터가 있다면 캐시 도입을 고민할 수 있다.
- 캐시 기술에는 여러가지가 있는데, 대표적으로 EhCache, Caffeine Cache, Redis 등이 있다.
- 스프링은 이런 캐시 기술들을 추상화하여 사용하기 쉽게 제공한다. (like. DataSource, 스프링의 PSA)
- 당연한 얘기지만, 실제로 캐시를 관리하는 것은 특정 캐시 기술이다. (스프링은 추상화만 제공)
- 스프링의 Cache 관련 애노테이션은 AOP기반으로 동작하기 때문에 내부 호출과 같은 이슈를 주의하여 사용해야 한다.
- 참고) [docs.spring.io - Cache Abstraction](https://docs.spring.io/spring-framework/reference/integration/cache.html)

# 💡 CacheManager
캐시 추상화에서는 캐시 기술을 지원하는 캐시 매니저를 Bean으로 등록해야 한다.

- **ConcurrentMapCacheManager**: JRE에서 제공하는 ConcurrentHashMap을 캐시 저장소로 사용할 수 있는 구현체다. 캐시 정보를 Map 타입으로 메모리에 저장해두기 때문에 빠르고 별다른 설정이 필요 없다는 장점이 있지만, 실제 서비스에서 사용하기엔 기능이 빈약하다.
- **SimpleCacheManager**: 기본적으로 제공하는 캐시가 없다. 사용할 캐시를 직접 등록하여 사용하기 위한 캐시 매니저 구현체다.
- **EhCacheCacheManager**: Java에서 유명한 캐시 프레임워크 중 하나인 EhCache를 지원하는 캐시 매니저 구현체다.
- **CaffeineCacheManager**: Java 8로 Guava 캐시를 재작성한 Caffeine 캐시 저장소를 사용할 수 있는 구현체다. EhCache와 함께 인기 있는 매니저인데, 이보다 좋은 성능을 갖는다고 한다.
- **JCacheCacheManager**: JSR-107 표준을 따르는 JCache 캐시 저장소를 사용할 수 있는 구현체다.
- **RedisCacheManager**: Redis를 캐시 저장소로 사용할 수 있는 구현체다.
- **CompositeCacheManager**: 한 개 이상의 캐시 매니저를 사용할 수 있는 혼합 캐시 매니저다.

# 💡 @EnableCaching
- `@Cacheable` 과 같은 애노테이션 기반의 캐시 기능을 사용하기 위해서는 먼저 별도의 선언이 필요하다.
- Spring Cache를 사용하려면 `@EnableCaching` 설정을 해주어야 한다.

```java
@EnableCaching
@Configuration
@RequiredArgsConstructor
public class CacheConfig {

    private final RedisConnectionFactory redisConnectionFactory;

    @Bean
    public CacheManager cacheManager() {
        RedisCacheConfiguration defaultConfig = RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofHours(1))
                .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new Jackson2JsonRedisSerializer<>(Object.class)));

        Map<String, RedisCacheConfiguration> configurations = new HashMap<>();
        configurations.put("noticeCache", defaultConfig.entryTtl(Duration.ofMinutes(30)));
        configurations.put("accountCache", defaultConfig.entryTtl(Duration.ofDays(1)));

        return RedisCacheManager.RedisCacheManagerBuilder
                .fromConnectionFactory(redisConnectionFactory)
                .cacheDefaults(defaultConfig)
                .withInitialCacheConfigurations(configurations)
                .build();
    }
}
```

# 💡 @Cacheable
- 캐시 저장소에 캐시 데이터를 저장하거나 조회하는 기능을 사용할 수 있다.
- `@Cacheable` 애노테이션이 정의된 메서드를 실행하면 데이터 저장소에 캐시 데이터 유무를 확인한다.
- 캐시에 데이터가 있다면 메서드를 실행하지 않고 바로 데이터를 리턴한다.
- 만약 예외가 발생하면 캐시 데이터는 저장하지 않는다.

|속성|설명|타입|
|:---:|:---|:---:|
|cacheNames|캐시 이름(설정 메서드 리턴값이 저장되는)|String[]|
|value|cacheNames의 alias|String[]|
|key|동적인 키 값을 사용하는 SpEL 표현식. 동일한 cache name을 사용하지만 구분될 필요가 있을 때 사용되는 값|String|
|condition|	SpEL 표현식이 참일 경우 캐싱 적용. or, and 등 조건식 및 논리연산 가능|String|
|unless|condition과 반대로 SpEL 표현식이 참일 경우 캐싱 적용 X|String|
|cacheManager|사용 할 CacheManager 지정|String|
|sync|여러 스레드가 동일한 키에 대한 값을 로드하려고 할 경우, 기본 메서드의 호출을 동기화함. 캐시 구현체가 Thread safe 하지 않는 경우, 캐시에 동기화를 걸 수 있는 속성|boolean|

```java
@Cacheable(value = "noticeCache", key="#noticeId")
public NoticeResponse getNoticeById(Long noticeId) {
    return noticeRepository.findById(noticeId);
}
```

- `@Cachable` 은 메서드의 인자로 캐시 키를 설정한다.
- 만약, 인자가 여러 개일 경우 모두 조합하여 캐시 키를 설정한다. 여기서는 Long noticeId 이다.
- 캐시 데이터는 메서드의 리턴값으로 설정한다. 여기서는 NoticeResponse 이다.
- `CacheManager` 스프링 빈에 설정된 `StringRedisSerializer` 와 `Jackson2JsonRedisSerializer` 로 캐시 키는 문자열로 변경되어 저장되고, 캐시 데이터는 JSON 형식으로 변경되어 저장된다.
- ❗ `@Cacheable` 애노테이션을 사용할 때는 메서드의 인자와 리턴 타입 변경에 유의해야 한다.
  - 운영 중인 시스템의 인자를 추가한다면 캐시 키 값이 변경될 수 있다.
  - 그러면 데이터 저장소에 저장된 데이터를 활용할 수 없게 된다.
  - 메서드의 리턴 타입을 다른 클래스로 변경한다면, 데이터 저장소에 저장된 데이터가 역직렬화되는 과정 중 에러가 발생할 수 있다.
 
# 💡 @CacheEvict
- 캐시 데이터를 캐시에서 제거하는 목적으로 사용하는 애노테이션이다.
- 원본 데이터를 변경하거나 삭제하는 메서드에 해당 애노테이션을 적용하면 된다.
- 원본 데이터가 변경되면 캐시에서 삭제하고 `@Cacheable` 애노테이션이 적용된 메서드가 실행되면 다시 변경된 데이터가 저장되기 때문이다.

|속성|설명|타입|
|:---:|:---|:---:|
|cacheNames|캐시 이름(설정 메서드 리턴값이 저장되는)|String[]|
|value|cacheNames의 alias|String[]|
|key|동적인 키 값을 사용하는 SpEL 표현식. 동일한 cache name을 사용하지만 구분될 필요가 있을 때 사용되는 값|String|
|condition|	SpEL 표현식이 참일 경우 캐싱 적용. or, and 등 조건식 및 논리연산 가능|String|
|cacheManager|사용 할 CacheManager 지정|String|
|allEntries|캐시 내의 모든 리소스를 삭제할 지의 여부|boolean|
|beforeInvocation|true 일 경우 메서드 수행 이전 캐시 리소스 삭제, false 일 경우 메서드 수행 후 캐시 리소스 삭제|boolean|

```java
// notice.id와 같은 키값을 가진 캐시를 제거한다.
@CacheEvict(value="noticeCache", key="#notice.id")
public void updateNotice(Notice notice) {
    // ...
}
 
// 모두 제거한다.
@CacheEvict(value = "noticeCache", allEntires = true)
public void clearNotice() {
    // ...
}
```

# 💡 @CachePut
- 캐시를 생성하는 기능만 제공하는 애노테이션이다.
- `@Cachable` 과 유사하게 실행 결과를 캐시에 저장하지만, 조회 시에 저장된 캐시의 내용을 사용하지는 않고 항상 메소드의 로직을 실행한다.
- 메서드 결과가 캐시가 되었음에도 메서드를 항상 실행시키고 싶다면, `@CachePut` 을 사용하자

```java
@CachePut(key = "testKey", condition="#caching")
public Object modifySome(boolean caching) {
	  // ...
}
```

# 💡 @Caching
- 두 개 이상의 캐시 애노테이션을 조합하여 사용한다.
- 여러 개의 캐싱 동작(@Cacheable, @CacheEvict, @CachePut)을 하나로 묶을 때 사용한다.

|속성|설명|타입|
|:---:|:---|:---:|
|cacheable|적용 될 @Cacheable array를 등록|Cacheable[]|
|evict|적용 될 @CacheEvict array를 등록|CacheEvict[]|
|put|적용 될 @Cacheput array를 등록|CachePut[]|

# References
- [Cache Abstraction - Spring Framework Docs](https://docs.spring.io/spring-framework/reference/integration/cache.html)
- [Cache#1 스프링의 캐시 추상화(@Cacheable)](https://jiwondev.tistory.com/282)
- [Spring Boot 에서 Cache 사용하기](https://bcp0109.tistory.com/385)
