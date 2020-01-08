---
layout: post
title: "SpringBoot, JPA, H2 database 셋팅"
categories: JPA
redirect_from: 
- 2019/04/12/jpa-h2-setting/
date: 2019-04-12 09:05:02
---
## 목표
이번 시간에는 SpringBoot, JPA, H2(DB)를 통한 간단한 Member 엔티티를 만들고, Junit 테스트로 검증하는 샘플 프로젝트에 대해서 알아보도록 하겠습니다.

## 의존성 추가

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>

```
`spring-data-jpa` 와 `h2(인메모리)` 디펜던시를 추가합니다.

## application.properties 추가

```properties
# H2 설정
spring.h2.console.enabled=true
spring.h2.console.path=/h2

# Datasource 설정
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.url=jdbc:h2:mem:test
spring.datasource.username=sa
spring.datasource.password=
```

애플리케이션을 실행하고, `localhost:8080/h2` 주소로 접속하면, 인메모리 DB인 H2 데이터베이스를 사용할 수 있습니다. 물론 인메모리 이기 때문에 애플리케이션을 재 실행하면 데이터가 날라갑니다.(휘발성 ~ )


## Member 엔티티
```java
@Data
@Table(name = "MEMBER")
@Entity
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    @Column(name = "NAME")
    private String name;
    
    @Column(name = "AGE")
    private Integer age;

}
```

- `@Entity`는 Member 클래스가 엔티티임을 명시함
- `@Table`은 Member 엔티티와 매핑할 테이블명을 지정한다.(클래스명과 정확히 일치한다면 굳이 명시 하지 않아도 된다.)
- `@Id`는 Persiscontext에서 식별할 수 있는 값을 나타낸다.(반드시 유니크 한 값)
- `@GeneratedValue`는 Id가 생성되는 전략을 나타내는데, 기본 전략은 AUTO로 설정되어 있기 때문에 각각 다른 Database의 Id 생성 전략을 유연하게 대응 할 수 있다. 예를 들어 Oracle은 Sequence라는 개념이 들어가지만, Mysql, Mariadb에서는 그렇지 않다.

## JpaRepository 를 상속 받는다.
```java
@Repository
public interface MemberRepository extends JpaRepository<Member, Long> {
}
```
JpaRepository(인터페이스)를 상속받음으로써 Jpa가 구현해 놓은 구현체 중에서 [`SimpleJpaRepository`](https://docs.spring.io/spring-data/jpa/docs/current/api/org/springframework/data/jpa/repository/support/SimpleJpaRepository.html)를 사용한다. 해당 클래스는 기본적인 CRUD 메서드를 이미 구현했기 때문에 그대로 사용하면 매우 편리하다.


```properties
## Hibernate
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```
추가적으로 `application.properties`에 쿼리가 날라갈 때, console에 formatting된 sql문을 보기 위해서 설정을 추가한다.

## 테스트 케이스 작성

```java
@RunWith(SpringRunner.class)
@SpringBootTest
@Rollback(false)
public class MemberServiceTests {

    @Autowired
    private MemberRepository memberRepository;

    @Test
    public void saveMemberTest() {

        //given
        Member member = new Member();
        member.setName("andrew");
        member.setAge(32);
        memberRepository.save(member);

        // when
        Member retrivedMember = memberRepository.findById(member.getId()).get();

        // then
        Assert.assertEquals(retrivedMember.getName(), "andrew");
        Assert.assertEquals(retrivedMember.getAge(), Integer.valueOf(32));
    }
```

이름과 나이를 갖는 Member를 저장하고, DB로 부터 불러오고 조회 한 데이터를 기존의 데이터와 비교하는 테스트 케이스

- @SpringBootTest는 전체 빈을 다 등록하고, 모든 application.properties를 반영해서 테스트를 하기 때문에 통합테스트용으로 많이 사용합니다. (그 외에 컨트롤러, 서비스, 레포지 토리 테스트는 Slicing 테스트로 단위 테스트적인 성격을 갖음)
- @Rollback(false) - 우리는 H2 DB이기 때문에 크게 상관이 없지만, 실제 물리적인 테스트용 데이터베이스를 구축하고 테스트 하는 경우에, 매 테스트마다 DataBase가 오염 되지 않기를 바랄 수도 있고, 실제 테스트 결과를 DB에 쌓음으로써 확인하고 싶은 경우도 있기 때문에 해당 옵션을 통해서 제어 할 수 있음
- 테스트 케이스 작성 요령이 많겠지만, 기본 적으로 given, when, then을 통해서 어떤 상황이 주어지고(given), 언제 그 상황을 가지고오고(when) 그리고 그 결과를 비교한다(then)의 주석을 달아주면 가독성이 좋음


```
Hibernate:
    drop table member if exists
Hibernate:
    drop sequence if exists hibernate_sequence
Hibernate: create sequence hibernate_sequence start with 1 increment by 1
Hibernate:

    create table member (
       id bigint not null,
        age integer,
        name varchar(255),
        primary key (id)
    )
```

처음에 member테이블이 존재하면 테이블을 삭제하고 다시 만드는 쿼리가 발생한다. 신기한게 application.properties에 `spring.jpa.hibernate.ddl-auto=create-drop` ~~이런 옵션을 주지 않았음에도 H2 db특성상(인메모리) 기존의 테이블을 전부 날리고 다시 생성하는 가 보다?~~

> 2020-01-09 추가 설명, 

> Spring Boot chooses a default value for you based on whether it thinks your database is embedded (default create-drop) or not (default none).

```
Hibernate:
    insert
    into
        member
        (age, name, id)
    values
        (?, ?, ?)
Hibernate:
    select
        member0_.id as id1_0_0_,
        member0_.age as age2_0_0_,
        member0_.name as name3_0_0_
    from
        member member0_
    where
        member0_.id=?
```

쿼리를 확인해 보면, 1) 등록 쿼리, 2) 조회 쿼리 두개가 발생하는 것을 알 수 있습니다.