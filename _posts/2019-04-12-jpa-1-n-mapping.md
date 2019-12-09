---
layout: post
title: "[JPA] 1:N 매핑"
date: 2019-04-12 09:05:25
category: 
 
- JPA
tags: 
- JPA
---

# 1.들어가며

JPA 연관관계 매핑중, 실무에서 가장 많이 쓰이는 1:N (일대다) 매핑에 대해서 알아보자.  
일대다 매핑을 하기 위해서 하나의 팀에 여러명의 멤버가 속해있는, 1:N 관계를 생각해보자.  
관계 매핑에는 `방향성`과 `외래키의 주인` 두 가지 상황이 발생 한다.(객체와 테이블간의 패러다임 불일치 때문에)  
일반적으로 객체에서는 참조를 통해서 방향성을 갖는다.

### 객체에서 방향성이란?

```java
@Data
public class Member {
    private Team team;
}
```

memeber.getTeam() 을 통해서 team을 조회할 수 있다.  
즉, member -> team간의 단방향(방향성)을 갖지만, 반대의 방향성은 존재하지 않는다.  
만약에 반대 상황도 존재할려면

```java
@Data
public class Team {
    private List<Member> memberList;
}
```

이렇게 설정해야지만, team.getMemberList()를 통해서 접근 할 수 있다. 이렇게 객체간의 참조를 통해서 탐색하는 과정을 `객체 그래프 탐색`이라고 한다.

### 외래키의 주인?

역시나 객체와 테이블간의 패러다임 불일치 중에 하나로, 외래키를 통해서 Join을 하게 되면, 각 테이블에서 외래키를 어디에 두고 사용하는 지는 중요하지 않다. 공유하는 느낌.

하지만, 객체에서는 각 엔티티마다 연관관계 매핑을 해야 하고, 외래키의 주인을 따로 설정해야 한다. 보통은 `@OneToMany(mappedBy = "team")` 를 통해서 mappedBy 프로퍼티의 반대편에서 외래키를 가지고 있다.

# 2.예제 코드

먼저 member -> team 의 단방향 관계를 살펴 보자.

```java
@Data
@Table(name = "MEMBER")
@Entity
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    private String name;
    private Long age;

    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;

}
```

```java
@Data
@Table(name = "TEAM")
@Entity
public class Team {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    private String name;

    // 양방향 관계일 때만 설정함
    //@OneToMany(mappedBy = "team")
    //private List<Member> memberList;
}
```

Member과 Team간의 연관관계 매핑에서 핵심은

`@ManyToOne` 과 `@JoinColumn` 의 쌍으로 이루어진다. 특히나 JoinColumn은 표시하지 않아도 JPA가 자동으로 해당 필드(team)에 \_ID를 붙여서 자동으로 생성해 주지만, 실무에서는 명시적으로 사용하는 것을 권한다.

실제 테스트 케이스를 통해서 검ㄴ증해보자.

```java
@RunWith(SpringRunner.class)
@SpringBootTest
@Rollback(false)
public class DemoApplicationTests {

    @Autowired
    private MemberRepository memberRepository;
    @Autowired
    private TeamRepository teamRepository;

    @Test
    public void save() {
        Team team = new Team();
        team.setName("A팀");
        teamRepository.save(team); // team 저장

        Member member = new Member();
        member.setName("andrew");
        member.setAge(32L);
        member.setTeam(team); //member의 team setting
        memberRepository.save(member); //member 저장

        Optional<Member> optionalMember = memberRepository.findById(member.getId());

        //테스트 검증 코드
        Assert.assertEquals(optionalMember.get().getTeam().getName(), "A팀");
    }
}
```

각각 Team과 Member가 저장되고, member.getTeam()를 통해서 객체 그래프 탐색까지 잘 되는 것을 확인할 수 있습니다.

