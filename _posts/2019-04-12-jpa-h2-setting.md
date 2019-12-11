---
layout: post
title: "[JPA] H2 데이터베이스 세팅"
categories: JPA
date: 2019-04-12 09:05:02
---
# 1\. 들어가며

Spring Boot, JPA, H2 db로 프로젝트를 구성하고, 간단한 Member엔티티를 만들고, 테스트 케이스로 검증하는 튜토리얼

# 2\. 예제 코드

## Dependecy 추가

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>test</scope>
</dependency>
```

스프링 부트 프로젝트에서 해당 라이브러리 등록하고 애플리케이션을 띄우면

```js
***************************
APPLICATION FAILED TO START
***************************

Description:

Failed to configure a DataSource: 'url' attribute is not specified and no embedded datasource could be configured.

Reason: Failed to determine a suitable driver class
```

DataSource: url 속성을 찾지 못한다는 메세지가 뜹니다. `h2 application properties` 구글에서 검색해서 `application.properties` 에 다음과 같이 설정합니다.

## Datasource Properties 추가

```java
# H2
spring.h2.console.enabled=true
spring.h2.console.path=/h2
# Datasource
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.url=jdbc:h2:mem:test
spring.datasource.username=sa
spring.datasource.password=
```

다시 애플리케이션을 띄우면

```
Failed to instantiate [com.zaxxer.hikari.HikariDataSource]: Factory method 'dataSource' threw exception; nested exception is java.lang.IllegalStateException: Cannot load driver class: org.h2.Driver
```

드라이버를 로드할 수 없다는 메세지가 뜹니다. 눈치 채셨나요? 우리가 처음에 dependency를 추가할 때, H2 Database 스코프를 Test로 두었기 때문에 RunTime에 해당 Driver를 로드 할 수 없는 것입니다.

```xml
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <!-- <scope>test</scope> -->  //이 부분을 주석으로 달거나, runtime으로 바꿔 줍니다.
</dependency>
```

물론 H2 Database를 테스트하는 경우에만 사용하는 경우에는 스코프를 test로 해줍니다. 이제 다시 애플리케이션을 띄웁니다.  
잘 뜹니다!

`localhost:8080/h2` 주소로 접속.

## Member Entity, Repository

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

코드의 가독성을 위해서 Lombok을 사용 `@Data` 는 `@Getter`, `@Setter`, `@AllArgsConstructor`, `@ToString`, 등등 엄청 많은 것들을 커버하기 때문에 실무에서는 저렇게 사용하지 않음.

- @Entity는 Member 클래스가 엔티티임을 명시함
- @Table은 Member 엔티티와 매핑할 테이블을 나타낸다.
- @Id는 Persiscontext에서 식별할 수 있는 값이 필요하기 때문에 나타냄
- @GeneratedValue는 Id가 생성되는 전략을 나타내는데, 기본 전략은 AUTO로 설정되어 있기 때문에 각각 다른 Database의 Id 생성 전략을 유연하게 대응 할 수 있다. 예를 들어 Oracle은 Sequence라는 개념이 들어가지만, Mysql, Mariadb에서는 그렇지 않다.

```java
@Repository
public interface MemberRepository extends JpaRepository<Member, Long> {
}
```

기본적인 CRUD 만을 할 것이기 때문에, Service 레이어를 만들지 않고 바로 Repository를 인터페이스로 만들었다. JpaRepository(인터페이스)를 상속받음으로써 Jpa가 구현해 놓은 CRUD를 바로 사용할 수 있다.

실제 테스트 케이스를 작성하기 전에, DB가 만들어 지고, SQL 포맷팅을 하는 properties를 추가한다.

```
## Hibernate
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```

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
        Assert.assertEquals(retrivedMember.getAge(), Integer.valueOf(33));
    }
```

이름과 나이를 갖는 Member를 저장하고, DB로 부터 불러오고 조회 한 데이터를 기존의 데이터와 비교하는 테스트 케이스

- @SpringBootTest는 전체 빈을 다 등록하고, 모든 application.properties를 반영해서 테스트를 하기 때문에 통합테스트용으로 많이 사용합니다. (그 외에 컨트롤러, 서비스, 레포지 토리 테스트는 Slicing 테스트로 단위 테스트적인 성격을 갖음)
- @Rollback(false) - 우리는 H2 DB이기 때문에 크게 상관이 없지만, 실제 물리적인 테스트용 데이터베이스를 구축하고 테스트 하는 경우에, 매 테스트마다 DataBase가 오염 되지 않기를 바랄 수도 있고, 실제 테스트 결과를 DB에 쌓음으로써 확인하고 싶은 경우도 있기 때문에 해당 옵션을 통해서 제어 할 수 있음
- 테스트 케이스 작성 요령이 많겠지만, 기본 적으로 given, when, then을 통해서 어떤 상황이 주어지고(given), 언제 그 상황을 가지고오고(when) 그리고 그 결과를 비교한다(then)의 주석을 달아주면 가독성이 좋음

```js
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

처음에 member테이블이 존재하면 테이블을 삭제하고 다시 만드는 쿼리가 발생한다. 신기한게 application.properties에 `spring.jpa.hibernate.ddl-auto=create-drop` 이런 옵션을 주지 않았음에도 H2 db특성상(인메모리) 기존의 테이블을 전부 날리고 다시 생성하는 가 보다?

```js
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

쿼리를 확인해 보면, 1) 등록 쿼리, 2) 조회 쿼리 두개가 발생하는 것을 알 수 있다.

```js
java.lang.AssertionError:
Expected :32
Actual   :33
 <Click to see difference>
```

위의 테스트 결과는 실패 한다. 나이 값을 잘 못 넣었기 때문에, 성공만 하는 테스트 케이스보다 일부러 실패하는 테스트 케이스를 작성하는 게 좋다.