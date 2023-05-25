## 즉시로딩과 지연로딩이란?

아래와 같은 연관관계가 있다고 가정하자
![image](https://github.com/deepsj1012/TIL/assets/67497759/3e5697b8-51d0-44a1-a8e5-270d4f2d4364)

위의 연관관계를 보고 엔티티를 작성해보면

```java
@Entity
public class Member {
    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @Column(name = "USERNAME")
    private String username;

    @ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn(name = "TEAM_ID")
    private Team team;
}
```
```java
@Entity
public class Team {

    @Id
    @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;

    private String name;

    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();
}
```
Team과 Member는 ***양방향 연관관계*** 이고, 연관관계의 주인은 Member이다. (Team 객체의 @OneToMany(mappedBy = "team") 부분 확인)

Member의 fetch = FetchType.EAGER 부분을 알아보자. 참고로 @ManyToOne 의 기본 fetch 타입은 EAGER이기 때문에 때문에 생략하게되면 EAGER로 동작한다. (~ToMany에서는 Lazy, ~ToOne에서는 Eager 가 dafault 이다.)

이제 예제 코드를 통해 로그를 조회해보자(EAGER).
```
Team team = new Team();
team.setName("teamA");
em.persist(team);

Member member = new Member();
member.setUsername("member1");
member.setTeam(team);
em.persist(member);

em.flush();
em.clear();

Member findMember = em.find(Member.class, member.getId());
```
Team 객체와 Member 객체를 만들고 Member 객체의 Setter 메소드를 통해 Team 객체를 세팅해주고   
em.find() 메소드로 ***Member***를 조회해 보았다.   


```
Hibernate: 
    select
        member0_.MEMBER_ID as MEMBER_I1_0_0_,
        member0_.TEAM_ID as TEAM_ID3_0_0_,
        member0_.USERNAME as USERNAME2_0_0_,
        team1_.TEAM_ID as TEAM_ID1_1_1_,
        team1_.name as name2_1_1_ 
    from
        Member member0_ 
    left outer join
        Team team1_ 
            on member0_.TEAM_ID=team1_.TEAM_ID 
    where
        member0_.MEMBER_ID=?
```
결과를 확인해보면 Member를 조회했는데 Team까지 join이 되는 것을 확인할 수 있다.
그럼 이제 반대로 LAZY로 바꾸어 실행해보자
 
로그 확인(LAZY)
```
Hibernate: 
    select
        member0_.MEMBER_ID as MEMBER_I1_0_0_,
        member0_.TEAM_ID as TEAM_ID3_0_0_,
        member0_.USERNAME as USERNAME2_0_0_ 
    from
        Member member0_ 
    where
        member0_.MEMBER_ID=?
```
확인 결과, EAGER와는 다르게 Member만 조회해오는 것을 확인할 수 있다.
   
- 그래서 EAGER와 LAZY의 차이점은?   
> EAGER는 연관관계에 있는 테이블까지 전부 조회를 하고, LAZY는 요구된 것만 조회해오고 연관관계에 있는 나머지 테이블의 데이터는 조회를 미룬다.

- 그럼 EAGER와 LAZY는 언제? 어떨 때? 사용해야 할까?
> 비지니스 로직 상 Member 데이터가 필요한 곳에 Team 데이터를 항상 같이 사용할 필요가 있다면 ***EAGER***   
> Member를 사용하는 곳에서 Team 데이터가 필요치 않다면 ***LAZY***로 설정하는 것이 좋다.




