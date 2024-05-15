# Previous

테스트용 데이터소스를 생성하다가 발생한 문제가 있어 포스팅한다.

내가 하고싶었던 설정은 테스트환경에서 h2 in-memory DB를 설정하고, dialect는 mysql(운영환경)로 설정하길 원했다.

# 💡 @DataJpaTest

레포지토리(데이터 억세스 레이어) 만 테스트 하고 싶을 때는 어떻게 할까?

스프링에서는 `@DataJpaTest` 애노테이션을 제공하는데, 데이터 억세스 레이어와 관련된 컴포넌트만 스프링 빈으로 등록해서 테스트할 수 있는 애노테이션이다.

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@BootstrapWith(DataJpaTestContextBootstrapper.class)
@ExtendWith(SpringExtension.class)
@OverrideAutoConfiguration(enabled = false)
@TypeExcludeFilters(DataJpaTypeExcludeFilter.class)
@Transactional
@AutoConfigureCache
@AutoConfigureDataJpa
@AutoConfigureTestDatabase
@AutoConfigureTestEntityManager
@ImportAutoConfiguration
public @interface DataJpaTest {
//...
}
```

# 💡 Test용 데이터소스 dialect 설정

`src/test/resources/application-test.yml` 를 만들고 아래와 같이 작성하자.

```yml
spring:
  datasource:
    url: jdbc:h2:mem:test_db;MODE=MYSQL;DATABASE_TO_LOWER=TRUE
    username: test_user
    password:
    driver-class-name: org.h2.Driver

  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        dialect: org.hibernate.dialect.MySQL8Dialect
```

데이터 소스를 설정할 때 다음 설정을 살펴보자.
- `url: jdbc:h2:mem:test_db;MODE=MYSQL;DATABASE_TO_LOWER=TRUE`

jdbc url을 위와같이 설정하면 H2 데이터베이스 호환모드를 MYSQL로 변경할 수 있다. [h2 document](http://www.h2database.com/html/features.html)

![image](https://github.com/shin-je-woo/TIL/assets/39439576/469090e0-de2c-42f8-8370-b9427a734f57)

데이터 소스를 설정했다면, 테스트 환경에서 `@ActiveProfiles` 애노테이션을 통해 spring profile을 test 로 작동하도록 설정해야 한다.

```java
@DataJpaTest
@Import(JpaConfig.class)
@ActiveProfiles(profiles = "test")
public abstract class BaseRepositoryTest {

    @Autowired
    protected EntityManager em;
}
```

# 💡 @AutoConfigureTestDatabase

위와 같이 설정하고 테스트를 실행하면 오류가 난다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/d683401b-52f6-4325-a4fc-8b1feb274e6b)

오류를 해석해보면 CREATE DDL 을 실행할 때 syntax가 맞지 않는다는 내용인데, 하나씩 살펴보자.

```
create table apply (id bigint not null auto_increment, created_at datetime(6), updated_at datetime(6), job enum ('BACKEND_DEVELOPER','DATA_SCIENTIST','DESIGNER','DEVOPS_DEVELOPER','MOBILE_DEVELOPER','PLANNER','WEB_DEVELOPER'), status enum ('ACCEPTED','EXCLUDED','REJECTED','WAITING'), type enum ('APPLY','OFFER'), applicant_id bigint, project_id bigint, primary key (id)) engine=InnoDB;
```

Hibernate가 생성해주는 create구문에 `engine=InnoDB;` 가 붙어있는 것을 확인할 수 있는데, 위에서 설정이 정상적으로 완료되었다면 H2 DB가 mysql 호환모드로 동작하여 문제가 발생하지 않았을 것이다.

뭔가 문제가 있다는 뜻인데, 하나씩 로그를 더 살펴보자.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/dcc5a539-a1c5-49c4-8739-a58020fa38f4)

로그를 보면, spring profile은 test로 정상적으로 동작하고 있지만, 연결된 DB가 설정한 것과 다른 것을 알 수 있다.

왜 이런 것일까?

테스트용 클래스에 작성했던 애노테이션 중 `@DataJpaTest` 에는 `@AutoConfigureTestDatabase` 애노테이션이 붙어있다.

`@AutoConfigureTestDatabase` 애노테이션은 테스트용 DB를 자동으로 구성해주는 것인데, 이 설정이 동작하여 내가 설정했던 데이터소스로 동작하지 않았던 것이다.

테스트용 DB를 내가 설정한 DB로 변경하려면 `@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)` 애노테이션을 붙여줘야 한다.

```java
@DataJpaTest
@Import(JpaConfig.class)
@ActiveProfiles(profiles = "test")
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE) // @DataJpaTest에서 제공하는 테스트용 DB 사용 X -> application-test.yml에서 정의한 데이터소스 사용
public abstract class BaseRepositoryTest {

    @Autowired
    protected EntityManager em;
}
```

![image](https://github.com/shin-je-woo/TIL/assets/39439576/0319746a-eb9c-457f-846f-7f2904ada8e5)

이렇게 설정을 마무리하면 내가 설정한 db로 동작하게 된다.
