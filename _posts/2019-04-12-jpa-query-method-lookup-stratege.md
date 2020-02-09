---
layout: post
title: "[JPA] 쿼리메서드(Lookup 전략)"
categories: [jpa]
tags: [jpa]
redirect_from: 
- 2019/04/12/jpa-query-method-lookup-stratege/
date: 2019-04-12 09:07:40
---
> JPA에서 쿼리 메서드를 정의하는 방법에 대해서 알아보자.

# QueryLookup 전략

- `CREATE` 메서드 이름을 분석해서 만듬
- `USE_DECLARED_QUERY` 미리 정의해둔 쿼리를 찾아서 사용
- `CREATE_IF_NOT_FOUND` 기본 전략 - 미리 정의해둔 쿼리 찾아서 없으면 메서드 이름 분석해서 만듬

CREATE는 메서드 이름을 통해서 쿼리를 매핑해 준다. 예를 들면 `findById` `findTop10ByTitleContainsOrderByIdDesc` 와 같이 실제 쿼리를 작성할 때 처럼 명명 규칙이 존재한다.

두번째는 @Query 어노테이션을 활용한 방법이다. 일반적으롱 JPQL 을 사용해서 작성하지만, navtive Query를 원하는 경우에는 nativeQuery 프로퍼티의 값을 true로 설정하면 된다.  
`@Query(“select p from Post”, nativeQuery =true)`

세번째가 쿼리를 만들어내는 기본 전략이다. 1번 2번을 합친것이다. 2번에서 정의한 쿼리가 없으면 1번을 통해서 만든다.

해당 QueryLookUp 전략은 `@EnableJpaRepository`에 `queryLookupStrategy` 프로퍼티를 통해서 변경할 수 있지만, 굳이 하지 않아도 된다. 관련 소스를 보는 것만으로 족함(예를 들면, @Query, @NamedQuery를 쿼리 메서드 위에 정의했을 때 우선순위가 어떻게 될까? )

# 예제 코드

```java
@Data
@Entity
@Table
public class Post {

    @Id
    @GeneratedValue
    private Long id;
    private String title;
```

id, title만 있는 Post 엔티티

```java
public interface PostRepository extends JpaRepository<Post, Long> {

    Post findByTitl(String title);
}
```

title을 통해서 Post를 찾는 쿼리 메서드 작성

## 테스트

```java
    @Test
    public void queryMethodTest() {
        // just context load ...
    }
```

아무 것도 작성하지 않고 테스트를 돌려보자. 쿼리 메서드가 잘 만들었으면, 빈을 초기화 하고 등록할 때 문제없이 잘 동작해야 한다.

```
Caused by: org.springframework.data.mapping.PropertyReferenceException: No property titl found for type Post! Did you mean 'title'?

```

예외가 난다. 쿼리 메서드를 만들 때 Post의 property명을 titl로 기입해서 `PropertyReferenceException`이 발생했다. `findByTitle()` 로 수정하고 테스트를 돌리면 성공

```java
    @Test
    public void queryMethodTest() {
        Post post = new Post();
        post.setTitle("post1");
        postRepository.save(post);

        Post result = postRepository.findByTitle("post1");
        Assert.assertEquals(result.getTitle(), "post1");
    }
```

쿼리 메서드만 잘 만들어 지는지 말고, 잘 동작하는지 테스트 코드 작성하고 테스트  
성공! 😀

## 참고

[JPA-QueryMethod](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.query-methods.details)

