# Previous

애플리케이션을 개발하면서 동시성 문제 (공유 데이터에 여러 스레드가 액세스하여 제어할 때 발생하는 문제) 를 마주칠 때가 있다.

사이드 프로젝트로 진행하고 있는 [Whatpl](https://github.com/whatpl/whatpl-backend) 에서도 발생했다.

Whatpl에서는 다음과 같은 상황이 발생할 수 있다.

- 사용자가 모집글에 지원 요청을 하는 순간에 모집이 마감되는 경우
- 프로젝트 등록자가 최대 모집인원을 수정하는 순간 지원 요청이 오는 경우
- 등등 다양한 상황에서 동시성 이슈가 발생할 수 있다.

동시성 문제는 이전에 TIL [동시성 문제 다루기](https://github.com/shin-je-woo/TIL/blob/main/ETC/%EB%8F%99%EC%8B%9C%EC%84%B1%20%EB%AC%B8%EC%A0%9C%20%EB%8B%A4%EB%A3%A8%EA%B8%B0.md) 에서도 다룬 바 있다.

이번에는 Whatpl에 분산락을 왜 적용했는지, Redisson을 선택한 이유는 무엇인지, 분산락을 어떻게 적용하고 문제를 해결하는지 작성해본다.

# 💡 분산락을 선택한 이유

Whatpl의 경우 ec2에 도커로 애플리케이션 인스턴스가 1개만 떠있고(GREEN-BLUE 배포라 하나는 down상태), DB도 RDS에 1개만 떠있다.

그러면 최적의 선택은 애플리케이션 또는 DB에 Lock을 거는 방식이 가장 이상적일 것이다. 분산락과 같이 외부와 통신할 필요가 없어 성능상 우위이며 코드 직관성이 더 좋기 때문이다.

그럼에도 분산락을 선택한 이유는 추후 애플리케이션 인스턴스 및 DB 확장성을 고려해서이다.

인스턴스를 여러 개 띄울 경우 애플리케이션에서 동시성을 제어하기 쉽지 않게되고, DB도 여러 개 구성할 경우 DB Lock만으로는 동시성 제어가 어렵다.

이런 이유로 추후 확장성을 고려해 분산락을 선택하게 되었다.

# 💡 Redisson 라이브러리 선택한 이유

분산락은 Redis, Mysql, Zookeeper 여러 플랫폼에서 구현이 가능하다.

이 중 Redis를 사용한 이유는 Whatpl 인프라에서 이미 사용중이어서 추가 인프라 비용이 들지 않는다는 점 때문이었다.

참고로, Mysql 로 분산락을 구현하는 예제는 우아한 형제들 기술 블로그에 잘 정리되어 있다. [MySQL을 이용한 분산락으로 여러 서버에 걸친 동시성 관리](https://techblog.woowahan.com/2631/)

Redis를 선택하고 나서 `Lettuce`와 `Redisson` 선택지가 생기는데, 그 중에 `Redisson` 을 선택한 이유는 다음과 같다.

### 편리한 Lock Interface 제공

`Lettue` 의 경우 개발자가 직접 lock, unlock 코드를 구현해야 한다. (`setnx`, `setex` 등등의 코드)

반면, `Redisson` 의 경우 자바의 `Lock` 인터페이스를 구현한 다양한 API 를 제공하고 있어 사용이 조금 더 편리하다는 장점이 있다.

그래서 lock, unlock, timeout 등을 안전하게 사용할 수 있다.

### Redis에 어울리는 방식

Redis는 사용자 요청에 대해 싱글 스레드로 동작한다. 이 말은 여러 요청을 동시에 실행할 수 없다는 뜻으로, 원자적 연산을 구현하기에 적합한 구조이다.

다만, 싱글 스레드로 하나의 요청이 매우 긴 러닝타임을 가지고 있다면 다음 요청이 대기하게 됨으로써 전체적인 성능이 하락하게 된다.

`Lettuce` 를 사용하지 않은 이유가 바로 이것이다.

`Lettuce` 는 Redis에 `setnx`, `setex` 등의 명령어를 Redis에게 지속적으로 요청해 락을 획득할 수 있는지 확인한다.  (스핀락 방식)

요청이 많아질 수록 Redis 에 부하가 더 많이 가게 되는 방식이다.

반면, `Redisson` 은 Redis의 Pub/Sub 구조를 이용해 락을 획득하기 때문에 Redis에 가해지는 부하가 상대적으로 적다.

# 💡 분산락 적용

분산락은 한번만 사용하지 않고 여러번 사용할 수 있어야 하고, 비즈니스 로직을 오염시키지 않아야 한다.

따라서, 분산락을 `스프링 AOP` 를 이용해서 구현하였다.

### ▶️ @DistributedLock
```java
/**
 * Redisson 을 이용한 분산락 AOP 적용 애노테이션
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface DistributedLock {

    /**
     * 락 이름
     */
    String name();

    /**
     * 락 대기 시간
     * 락 획득을 위해 waitTime 만큼 대기한다.
     */
    long waitTime() default 5L;

    /**
     * 락 임대 시간
     * 락을 획득한 이후 leaseTime 이 지나면 락을 해제한다.
     */
    long leaseTime() default 3L;

    /**
     * 락 시간 단위
     */
    TimeUnit unit() default TimeUnit.SECONDS;
}
```

포인트컷 으로 사용할 `@DistributedLock` 애노테이션이다.

Redisson에서 제공하는 API의 매개변수들을 설정할 수 있도록 여러 매개변수를 정의했다.

### ▶️ 분산락 Aspect
```java
@Slf4j
@Aspect
@Component
@RequiredArgsConstructor
public class DistributedLockAspect {

    private static final String REDISSON_LOCK_PREFIX = "lock:";
    private final RedissonClient redissonClient;
    private final AopTransaction aopTransaction;

    /**
     * aopTransaction.proceed(joinPoint) - 트랜잭션이 수행된 이후 락 해제가 보장된다.
     */
    @Around("@annotation(com.whatpl.global.aop.annotation.DistributedLock)")
    public Object execute(final ProceedingJoinPoint joinPoint) throws Throwable {
        MethodSignature signature = CastUtils.cast(joinPoint.getSignature());
        DistributedLock distributedLock = signature.getMethod().getAnnotation(DistributedLock.class);

        String name = REDISSON_LOCK_PREFIX + getSpELValue(signature.getParameterNames(), joinPoint.getArgs(), distributedLock.name());
        RLock lock = redissonClient.getLock(name);

        try {
            boolean available = lock.tryLock(distributedLock.waitTime(), distributedLock.leaseTime(), distributedLock.unit());
            if (!available) {
                log.info("DistributedLock 획득 실패. name = {}", name);
                return false;
            }

            return aopTransaction.proceed(joinPoint);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            throw new RuntimeException("DistributedLock 작업 쓰레드가 인터럽트 되었습니다. name = " + name, e);
        } finally {
            if (lock.isHeldByCurrentThread()) {
                try {
                    lock.unlock();
                } catch (IllegalMonitorStateException e) {
                    log.error("DistributedLock 이미 해제 되었습니다. name = {}", name);
                }
            }
        }
    }

    private Object getSpELValue(String[] parameterNames, Object[] args, String key) {
        ExpressionParser parser = new SpelExpressionParser();
        StandardEvaluationContext context = new StandardEvaluationContext();

        for (int i = 0; i < parameterNames.length; i++) {
            context.setVariable(parameterNames[i], args[i]);
        }

        return parser.parseExpression(key).getValue(context, Object.class);
    }
}
```

`@DistributedLock` 포인트컷에서 수행되는 어드바이스에서는 다음과 같은 작업을 수행한다.

1. 락 획득을 지정된 시간만큼 시도한다. (락 타임아웃을 지정함으로써 스레드가 무한대기에 빠지지 않도록 한다.)
2. `@DistributedLock` 애테이션이 선언된 메서드를 수행한다.
3. 락을 해제한다. 이 때, 트랜잭션이 수행된 이후 락 해제를 보장한다.

### ❗ 트랜잭션 수행 이후 Lock 해제 보장

여기서 3번이 중요한 과정인데, Lock은 트랜잭션이 수행된 이후 해제가 보장되어야 한다는 점이다.

만약, 트랜잭션이 커밋되기 전에 Lock이 해제되면 어떻게 될까?

재고 시스템을 예제로 생각해보자.

1. Client1, Client2가 동시에 재고차감 로직을 요청한다.
2. Client1이 Lock을 획득하고 Client2는 Lock획득을 대기한다.
3. Client1이 재고를 차감한다. (10 - 1 = 9)
4. **이 때, Client1이 Lock을 해제한다.**
5. **Client2가 Lock을 획득하고 재고를 차감한다. (10 - 1 = 9)**
6. Client1이 Commit한다. (재고 = 9)
7. Client2가 Lock을 해제하고 트랜잭션을 Commit한다. (재고 = 9)

10이었던 재고가 2번의 차감 요청에도 1개만 수량이 차감된다.

즉, 트랜잭션 수행 전에 Lock이 해제되면 데이터 정합성에 문제가 발생한다.

따라서, 분산락 Aspect에서 분산락을 수행하는 로직은 별도의 트랜잭션을 생성하여 트랜잭션이 수행된 이후 Lock이 해제됨을 보장해야 한다.

### ▶️ AopTransaction
```java
/**
 * AOP 적용 시 트랜잭션 분리를 위한 클래스
 */
@Component
public class AopTransaction {

    /**
     * 물리 트랜잭션을 분리
     * 트랜잭션 어드바이스와 커스텀 어드바이스 순서에 상관 없이 커스텀 어드바이스에서 트랜잭션이 수행됨을 보장
     * 커스텀 어드바이스에서 finally 구문 수행 시 필요 (ex. 락 해제)
     */
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public Object proceed(final ProceedingJoinPoint joinPoint) throws Throwable {
        return joinPoint.proceed();
    }
}
```

`@Distributed` 분산락 어드바이스가 `@Transactional` 트랜잭션 어드바이스보다 먼저 수행되면 트랜잭션 수행 이후 Lock이 해제됨이 보장되지만, 반대의 경우는 트랜잭션 수행 이후 Lock 해제를 보장할 수 없다.

따라서, `@Distributed` 가 선언된 메서드에서는 `Propagation.REQUIRES_NEW` 전파옵션을 통해 별도의 트랜잭션을 수행하여 트랜잭션이 수행됨을 보장해야 한다.

`Propagation.REQUIRES_NEW` 옵션은 기존 트랜잭션 여부와 상관 없이 항상 새로운 물리 트랜잭션을 만든다는 특징이 있다. (만약, 기존 트랜잭션이 존재한다면 기존 트랜잭션은 Pending 되어 커넥션 점유시간이 길어질 수 있다는 점을 주의해야 한다.)

### ▶️ 분산락 AOP 적용 코드

```java
@Transactional
@DistributedLock(name = "'project:'.concat(#projectId)")
public void modifyProject(final Long projectId, final ProjectUpdateRequest request) {
    Attachment representImage = getRepresentImage(request.getRepresentImageId());
    Project project = projectRepository.findWithRecruitJobsById(projectId)
            .orElseThrow(() -> new BizException(ErrorCode.NOT_FOUND_PROJECT));
    project.modify(request, representImage);
}
```

# 정리

`@Transactional` 처럼 메서드에 `@Distributed` 애노테이션만 붙이면 분산락을 수행할 수 있게 되었다.

비즈니스 로직을 오염시키지도 않고, 분산락을 재사용할 수 있는 장점을 얻었다.

# Reference

- [재고시스템으로 알아보는 동시성이슈 해결방법](https://www.inflearn.com/course/%EB%8F%99%EC%8B%9C%EC%84%B1%EC%9D%B4%EC%8A%88-%EC%9E%AC%EA%B3%A0%EC%8B%9C%EC%8A%A4%ED%85%9C/dashboard)
- [A Guide to Redis with Redisson](https://www.baeldung.com/redis-redisson)
- [풀필먼트 입고 서비스팀에서 분산락을 사용하는 방법 - Spring Redisson](https://helloworld.kurly.com/blog/distributed-redisson-lock/)
- [SpEL(Spring Expression Langauge) 사용법 + 어노테이션에 SpEL로 값 전달하기](https://devoong2.tistory.com/entry/SpELSpring-Expression-Langauge-%EC%82%AC%EC%9A%A9%EB%B2%95-%EC%96%B4%EB%85%B8%ED%85%8C%EC%9D%B4%EC%85%98%EC%97%90-SpEL%EB%A1%9C-%EB%B3%80%EC%88%98-%EC%A0%84%EB%8B%AC%ED%95%98%EA%B8%B0)
