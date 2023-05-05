HikariDataSource를 사용하던 중 신기한 기능을 발견해서 기록하기 위해 작성한다.  

먼저 작성했던 코드를 보자.
```java
@Slf4j
@RequiredArgsConstructor
public class MemberServiceV2 {

    private final DataSource dataSource;
    
    public void accountTransfer(String fromId, String toId, int money) throws SQLException {

        try (Connection con = dataSource.getConnection()) {
            try {
                con.setAutoCommit(false); //트랜잭션 시작
                //비즈니스 로직 수행
                bizLogic(con, fromId, toId, money);
                con.commit(); //성공 시 커밋
            } catch (Exception e) {
                con.rollback(); //실패 시 롤백
                throw new IllegalStateException(e);
            } finally {
                log.info("==================트랜잭션 작업 수행 후 Connection반납할 때 autoCommit을 돌려놓습니다.");
                //con.setAutoCommit(true); //커넥션 풀을 고려해서 Connection반납할 때는 autoCommit을 true로 돌려놓고 반납
            }
        }
    }
}
```
* DB Connection을 얻어올 때 try-with-resources를 사용해서 자원 사용 후 Connection을 자동으로 반납하는 코드를 작성했다.
* DataSource는 HikariDataSource를 DI 해주었다.
* 위 코드를 실행하면 트랜잭션을 시작하기 위해 autoCommit을 데이터베이스 설정과 다르게 false로 설정해 주었는데, 신기한 log가 발견되었다.
![image](https://user-images.githubusercontent.com/39439576/236372826-bc7caa35-b2bf-4db6-b2c9-53e60542b01d.png)
* autoCommit을 true로 돌려놓지 않고 Connection을 HikariCP에 반납했는데, `Hikari에서 autoCommit을 Reset`했다는 내용의 log였다.
* 위 코드에서 주석 친 `con.setAutoCommit(true)`코드로 의도적으로 autoCommit을 돌려놓고 반납하려 했는데, 히카리가 알아서 해주고 있었다.
* StackOverFlow에 비슷한 질문 글이 있어 질문글을 보고 마무리 하겠다. [히카리CP autoCommit Reset관련 질문글](https://stackoverflow.com/questions/41202242/reset-autocommit-on-connection-in-hikaricp)

![image](https://user-images.githubusercontent.com/39439576/236373203-fd193dba-cd93-4b41-8e40-0e15d9287e00.png)
* 질문글은 내 상황과 완전히 동일했고, 답변은 히카리CP에서 반납된 Connection의 autoCommit설정이 DB설정과 다를 경우 커넥션 반납 시 원상복구 시킨다는 내용이다.
* 실제로, 위 코드에서 주석 친 `con.setAutoCommit(true)`을 주석해제하고 실행하면 Hikari의 autoCommit Reset 로그는 남지 않았다.
* 정리하자면, HikariCP에서는 Connection을 반납할 때 autoCommit을 DB설정과 다르게 반납해도 알아서 원상복구 시킨다.
