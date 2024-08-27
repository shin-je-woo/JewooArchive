# Previous

JPA를 사용하다 보면 컬렉션이 포함된 엔티티의 페이징 처리가 필요한 경우가 있다.

보통 조회 성능개선을 위해 연관된 엔티티를 fetch join하게 되는데, 컬렉션 엔티티가 포함된 fetch join은 페이징이 불가능하다. (정확히는 메모리에서 페이징을 하기 때문에 매우 위험하다.)

그래서 컬렉션이 포함된 엔티티를 페이징 처리해야 할 경우에는 ~ToOne 관계에만 fetch join을 하고, ~ToMany 관계는 hibernate.default_batch_fetch_size 옵션으로 해결한다.

DTO로 조회할 때도 컨셉은 동일하다.

먼저, ~ToOne 관계는 join 하여 페이징 결과를 받고, 나머지 컬렉션은 별도로 in 절을 통해 조회하면 된다.

만약, 컬렉션을 많이 포함한 엔티티를 페이징 처리할 경우 ~ToMany에 있는 엔티티를 조회할 때 성능개선은 어떻게 할까?

하나의 스레드(동기방식)로 모든 컬렉션을 조회하지 말고, 여러 개의 스레드(비동기)가 컬렉션을 나눠서 조회하고, 결과를 합치면 조회성능이 향상될 수 있을 것이다.

# 💡 동기방식의 조회

먼저, 아래와 같은 엔티티가 있다고 해보자.

#### ▶️ Project 엔티티
```java
@Getter
@Entity
@Table(name = "project")
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Project extends BaseTimeEntity {

    // ...

    @OneToMany(mappedBy = "project", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<ProjectSkill> projectSkills = new ArrayList<>();

    @OneToMany(mappedBy = "project", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<RecruitJob> recruitJobs = new ArrayList<>();

    @OneToMany(mappedBy = "project", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<ProjectComment> projectComments = new ArrayList<>();

    @OneToMany(mappedBy = "project", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<ProjectLike> projectLikes = new ArrayList<>();

    @OrderBy("createdAt asc")
    @OneToMany(mappedBy = "project", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<ProjectParticipant> projectParticipants = new ArrayList<>();

    // ...
}
```

Project 엔티티에는 5개의 ~ToMany를 가지고 있다.

페이징 쿼리에는 5개의 컬렉션을 모두 포함해야 하는 상황이라고 가정해보자.

아래는 페이징 조회 쿼리 일부이다.

```java
Override
public Slice<ProjectInfo> search(Pageable pageable, ProjectSearchCondition searchCondition) {
    // Project (루트) 조회
    List<ProjectInfo> projectInfos = findProjectInfos(pageable, searchCondition);
    List<Long> projectIds = toProjectIds(projectInfos);
    // ProjectSkill (1:N) 조회
    Map<Long, List<ProjectSkill>> projectSkillMap = findProjectSkillMap(projectIds);
    // RecruitJob (1:N) 조회
    Map<Long, List<RecruitJob>> recruitJobMap = findRecruitJobMap(projectIds);
    // ProjectParticipant (1:N) 조회
    Map<Long, List<ProjectParticipant>> participantMap = findParticipantMap(projectIds);
    // ProjectLike (1:N) 조회
    Map<Long, List<ProjectLike>> projectLikeMap = findProjectLikeMap(projectIds);
    // ProjectComment (1:N) 조회
    Map<Long, List<ProjectComment>> projectCommentMap = findProjectCommentMap(projectIds);

    ...
    // 루트 DTO인 ProjectInfo 에 ~ToMany 엔티티 조합하는 로직
```

먼저, `List<ProjectInfo> projectInfos` 루트를 조회한다. 이 쿼리는 ~ToOne관계만 fetch join 하여 메모리에서 페이징 될 일이 없게 한다.

그 후, ~ToMany 관계의 엔티티를 따로따로 조회한 후, projectInfos에 조회결과를 조합한다.

성능개선 포인트는 6개의 조회쿼리(1개의 루트 조회쿼리, 5개의 ~ToMany 조회쿼리)를 각각 나눠서 실행하면 총 조회 시간은 순차적으로 6개의 쿼리를 호출하는 것보다 빨라질 것이다.

이론적으로 총 조회시간은 6개의 쿼리 중 가장 큰 조회시간일 것이다.

먼저, 순차적으로 6개의 쿼리를 조회하는 로직으로 성능을 알아보자.

## 성능 확인

![image](https://github.com/shin-je-woo/TIL/assets/39439576/38110413-3754-41ac-b348-cdfe8a9771cc)

nGrinder 로 agent1개, Vuser 10으로 세팅하고 테스트 시 TPS는 197이 나오는 것을 확인할 수 있다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/4acbd593-e606-46fa-b4c3-cab0a9070ada)

scouter 로 모니터링 시 Heap과 CPU 가 비교적 안정적으로 수행되는 것을 알 수 있다.

# 💡 비동기방식의 조회

자바에서는 비동기 방식의 API를 제공하고 있는데, CompletableFuture를 이용해서 비동기를 구현해보자.

페이징 조회쿼리를 다음과 같이 바꾸자.

```java
public Slice<ProjectInfo> searchProjectList(Pageable pageable, ProjectSearchCondition searchCondition) {
    List<ProjectInfo> projectInfos = projectRepository.findProjectInfos(pageable, searchCondition);

    CompletableFuture.allOf(
            projectReadAsyncService.mergeSkills(projectInfos),
            projectReadAsyncService.mergeRemainedJobs(projectInfos),
            projectReadAsyncService.mergeLikes(projectInfos),
            projectReadAsyncService.mergeComments(projectInfos)
    ).join();

    return new SliceImpl<>(projectInfos, pageable, PaginationUtils.hasNext(projectInfos, pageable.getPageSize()));
}
```

루트를 조회하는 쿼리는 동일하고, 4개의 merge~ 로 시작되는 메서드들이 비동기로 수행되는 메서드들이다.

merge~ 로 시작되는 메서드들은 아래와 같이 CompletableFuture에 ~ToMany 컬렉션 조회쿼리를 제출하고, ExecutorService가 제공하는 스레드 풀 (여기서는 ForkJoinPool) 에서 조회쿼리가 동시에 실행된다.

```java
@Service
@RequiredArgsConstructor
public class ProjectReadAsyncService {

    private final ProjectRepository projectRepository;
    private static final ExecutorService executor = Executors.newWorkStealingPool();

    /**
     * projectInfos 를 순회하면서 skills 필드를 set 합니다.
     */
    public CompletableFuture<Void> mergeSkills(List<ProjectInfo> projectInfos) {
        return CompletableFuture.supplyAsync(() -> projectRepository.findProjectSkillMap(toProjectIds(projectInfos)), executor)
                .thenAccept(skillMap -> projectInfos.forEach(projectInfo -> {
                    long projectId = projectInfo.getProjectId();
                    List<Skill> skills = skillMap.get(projectId)
                            .stream()
                            .map(ProjectSkill::getSkill).toList();
                    ProjectInfo.Editor.fromSkills(skills).merge(projectInfo);
                }));
    }
    // ...
}
```

### 조회 쿼리 쓰레드 확인

![image](https://github.com/shin-je-woo/TIL/assets/39439576/45c76117-6177-47eb-b17f-dd8f70e07a66)

로그에 찍히는 조회 쿼리를 수행하는 쓰레드가 모두 다른 것을 확인할 수 있다.

## 성능 확인

![image](https://github.com/shin-je-woo/TIL/assets/39439576/5e33d0ae-878e-4868-b586-7d30c8baab7a)

동기방식과 동일조건에서 부하테스트 수행 시 TPS 258가 나왔다.

❗**동기방식과 비교해보았을 때 약 31% 의 성능개선이 있다.**

지금 애플리케이션에서는 ~ToMany쿼리의 수행시간이 길지 않아 많은 개선은 기대할 수 없지만, 수행시간이 긴 조회쿼리가 많을 수록 더 많은 성능개선이 될 것이라 생각한다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/9ce5bf35-1960-4986-968b-08348c45de4c)

# 💡 비동기 조회 주의점

비동기 방식으로 DB를 조회할 때는 주의해야할 점이 몇가지가 있다.

## 스프링 트랜잭션

스프링의 트랜잭션은 내부적으로 `ThreadLocal` 을 사용한다.

즉, 트랜잭션 전파옵션의 기준은 같은 쓰레드인지 여부이다.

비동기 방식으로 프로그래밍을 하게 되면 멀티쓰레드방식을 사용하게 되는데, 비동기작업을 제출한 쓰레드와 비동기작업을 수행하는 쓰레드가 서로 다르기 때문에 같은 트랜잭션에서 동작하지 않는다.

지금 포스팅하는 경우와 같이 조회만 하는 경우에 크게 문제가 되지 않겠지만, 데이터를 추가/업데이트/삭제하는 작업을 멀티쓰레드로 처리하게 될 경우 조심해야 한다.

이 경우에는 작업쓰레드에서 예외가 발생할 경우 CompletableFuture에서 제공하는 예외처리 API를 통해 호출쓰레드를 롤백하는 작업을 추가로 해줘야할 수도 있다.

## DB 커넥션 풀

비동기 멀티쓰레드로 조회쿼리를 구성할 경우 동기방식보다 DB커넥션을 더 많이 이용하게 된다.

만약, 조회요청이 몰리는 경우 DB성능저하, 애플리케이션이 DB Connection을 대기하다가 timeout되는 경우가 생길 수 있다.

한번의 조회 Task에서 최대 연결될 DB Connection과 WAS로 들어오는 최대 요청수를 잘 고려하여 DB 커넥션 풀 사이즈를 조정할 필요가 있다.

# 💡 결론

~ToMany 관계에 있는 여러 조회 쿼리를 비동기로 조회하고 결과를 조합하면 성능 개선을 기대할 수 있다.

~ToMany 관계가 많으면 많을수록, 하나의 조회쿼리가 오래 걸리면 오래 걸릴수록 더 많은 성능 개선을 기대할 수 있다.

스프링 트랜잭션을 이용할 경우 서로 다른 쓰레드 (비동기 작업을 수행하는 쓰레드와 호출하는 쓰레드) 간 트랜잭션은 공유되지 않으니 별도의 오류 처리가 필요하다.

peak 조회를 고려하여 적절한 DB 커넥션 풀을 설정해야 한다.
