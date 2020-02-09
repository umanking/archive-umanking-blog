---
layout: post
title: "[JPA] Casecade 옵션"
categories: [jpa]
tags: [jpa]
redirect_from: 
- 2019/04/12/jpa-cascade/
date: 2019-04-12 09:01:48
---


> 멤버(Member)와 사물함(Locker) 1:1 매핑 상황을 살펴보고, cascade 옵션이 어떤 상황에서 쓰이는지 알아보자.

# 예제코드

### Member, Locker 엔티티

```java
@Data
@Table(name = "MEMBER")
@Entity
public class Member {

    @Id
    @GeneratedValue
    private Long id;

    @Column(name = "NAME")
    private String name;

    @Column(name = "AGE")
    private Integer age;

    @OneToOne
    @JoinColumn(name = "LOCKER_ID")
    private Locker locker;

}
```

```java
@Data
@Entity
@Table
public class Locker {

    @Id
    @GeneratedValue
    private Long id;

    @Column(name = "NAME")
    private String name;
}
```

Member -> Locker 단방향 1:1 매핑에 대해서 살펴보자.

중요한 것은, `@OneToOne` 과 `@JoinColumn`을 사용해서 관계 매핑을 했고, 참조하는(방향성을 가지는) Locker를 해당 멤버 필드의 Type으로 지정했다. 명시적으로 `@JoinColumn`의 `name 프로퍼티`를 참조하는 LOCKER\_ID로 설정했다.

### 테스트 케이스

```java
    @Test
    public void 단방향테스트_Memmber_Locker_SaveTest() {

        //given
        Member member = new Member();
        member.setName("andrew");
        member.setAge(32);


        Locker locker = new Locker();
        locker.setName("1번 사물함");

        member.setLocker(locker);

        memberRepository.save(member);
        lockerRepository.save(locker);

        //when
        Member existMember = memberRepository.findById(member.getId()).get();

        //then
        assertEquals(existMember.getLocker().getName(), "1번 사물함");

    }
```

Member, Locker 객체를 만들고 저장 한후에, member.getLocker().getName() 으로 객체를 탐색하는 테스트 케이스.  
**이 테스트 케이스는 실패 한다.**

```js
`save the transient instance before flushing` : com.example.demo.spring.jpa.member.Member.locker -> com.example.demo.spring.jpa.locker.Locker;
```

다음과 같은 exception이 발생한다. flusing 하기 전에 transient 인스턴스를 save하라!

테스트 코드를 다시 살펴보면, locker 객체를 저장하기 전에, 객체 생성만 하고, member의 setLocker의 인자로 넘기는 것을 알 수 있다.

즉, locker가 save()를 통해서 db에 저장 되기도 전에, memberRepository.save()를 호출 하니 문제가 발생한 것이다. (자세히 완벽히 이해 못함)

> DB에 플러쉬 되는 시점
>
> - @Transational 메소드가 종료 되는 시점
> - 강제로 PersistContext를 통해서 persist()

위와 같은 문제가 발생했을 때, 2가지 방법이 있다.

### 해결방법 1. 프로그래밍적으로

논리적 흐름이 맞지 않기 때문에, locker를 먼저 저장하고, member를 저장하도록 로직을 수정하면 테스트 케이스가 성공한다.

```java
    ...
    lockerRepository.save(locker);
    memberRepository.save(member);

```

하지만 매번 로직을 작성할 때마다 고려해야 하기 때문에 그리 쉬운 방법은 아니다.  
나중에 양방향 관계에서는 **편의 메소드** 라는 것을 통해서 실수 할 수 있는 부분을 줄여 주기는 한다.

### 해결방법 2. cascade 옵션을 주는 것

`@OneToOne`에 `cascade = CascadeType.ALL` 옵션을 줌으로써 해결 할 수 있다.  
이 casecade는 전파되는 속성을 나타낸다. 즉, member가 save()를 할 때, 연관관계에 있는 locker도 저장함으로써 위와 같은 문제를 해결 할 수 있다. 처음 테스트 케이스는 그대로 두고, Member 엔티티에서 해당 옵션만을 주면 테스트는 성공한다.

```java
    ...
    @OneToOne(cascade = CascadeType.ALL)
    @JoinColumn(name = "LOCKER_ID")
    private Locker locker;

```

### 🤔 여기서 갑자기 궁금해진 점

그럼 casecade옵션으로 연관 엔티티도 저장된다고 했는데, 테스트 케이스에서 locker의 객체를 저장하는 `lockerRepository.save(locker);` 을 주석 처리하고 테스트 케이스를 돌리면 어떻게 될까요?

```
Hibernate:
    insert
    into
        locker
        (name, locker_id)
    values
        (?, ?)
Hibernate:
    insert
    into
        member
        (age, locker_id, name, member_id)
    values
        (?, ?, ?, ?)
```

놀랍게도(?), 당연하게도 결과는 성공한다. locker insert 쿼리문과 member insert 쿼리문 동시에 두개가 날라감

정리: casecade 옵션을 활용하면, 등록, 삭제 시에 불필요한 save() 메소드를 하지 않아도 참조하는 쪽의 하나의 엔티티만 저장/삭제 해도 같이 반영된다. **(특히, 연관관계의 DB 데이터를 지울 때, 데이터베이스의 정합성을 고려하면 좋음)**

## 이번엔 양방향 관계로 설정!!

위에는 단방향 관계 였으니, 이번에는 양방향 관계 설정을 마무리 해보자 💪

```java
@Data
@Table(name = "MEMBER")
@Entity
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column(name = "MEMBER_ID")
    private Long id;

    @Column(name = "NAME")
    private String name;

    @Column(name = "AGE")
    private Integer age;

    @OneToOne(cascade = CascadeType.ALL)
    @JoinColumn(name = "LOCKER_ID")
    private Locker locker;

}
```

Member 엔티티는 변함이 없음

```java
@Data
@Entity
@Table
public class Locker {

    @Id
    @GeneratedValue
    @Column(name = "LOCKER_ID")
    private Long id;

    @Column(name = "NAME")
    private String name;

    // 이 부분이 추가됨
    @OneToOne(mappedBy = "locker")
    private Member member;
}

```

Locker 엔티티에서는 역시 1:1 연관관계 어노테이션인 `@OneToOne`을 선언해줌.

중요한 것은 `외래키의 관리`가 남아 있어서  
mappedBy 속성을 통해서 마무리 지었다.

이번에 테스트 케이스는 양방향 관계가 잘 매핑되었는지를 확인한다.

- member -> getLocker()
- locker -> getMember()

```java
    @Test
    public void 양방향관계_테스트() {

        //given
        Member member = new Member();
        member.setName("andrew");
        member.setAge(32);

        Locker locker = new Locker();
        locker.setName("1번 사물함");
        member.setLocker(locker);


        memberRepository.save(member);

        //when
        Member exsitMember = memberRepository.findById(member.getId()).get();
        Locker existLocker = lockerRepository.findById(locker.getId()).get();

        //then
        assertEquals(exsitMember.getLocker().getName(), "1번 사물함");
        assertEquals(existLocker.getMember().getName(), "andrew");

    }

```

테스트 성공!