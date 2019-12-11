---
layout: post
title: "[JPA] projection 살펴보기"
categories: JPA
date: 2019-04-12 09:08:56
---
> JPA를 사용해서 엔티티의 일부 데이터만 가져오는 Projection에 대해서 알아보자.

```java
@Data
@Entity
@Table
public class Comment {

    @Id
    @GeneratedValue
    private Long id;
    private String comment;

    @ManyToOne(fetch = FetchType.LAZY)
    private Post post;

    private int up;
    private int down;
    private boolean best;

    private String votes;

}
```

다음과 같은 Comment 엔티티가 존재한다.

```java
public interface CommentSummary {

    String getComment();

    int getUp();

    int getDown();
}
```

필요한 일부 데이터만을 담은 `CommentSummary` 인터페이스를 만든다.

```java
@Repository
public interface CommentRepository extends JpaRepository<Comment, Long> {

    List<Comment> findByPost_Id(Long id);
}
```

먼저 comment의 id를 통해서 Post를 찾는 쿼리메서드를 만들고 테스트를 돌려보자.

```
Hibernate: 
    select
        comment0_.id as id1_0_,
        comment0_.best as best2_0_,
        comment0_.comment as comment3_0_,
        comment0_.down as down4_0_,
        comment0_.post_id as post_id7_0_,
        comment0_.up as up5_0_,
        comment0_.votes as votes6_0_ 
    from
        comment comment0_ 
    left outer join
        post post1_ 
            on comment0_.post_id=post1_.id 
    where
        post1_.id=?
```

일반적으로 `Comment` 엔티티에 정의한 모든 필드를 다 가지고 온다.

```java
    List<CommentSummary> findByPost_Id(Long id); // List 타입을 CommentSummary로 변경
```

이렇게 바꾸고 테스트 케이스를 돌리면 다음과 같이 interface에서 정의한 필드 3개만 쿼리문이 실행됨을 알 수 있다.(최적화)

```
Hibernate: 
    select
        comment0_.comment as col_0_0_,
        comment0_.up as col_1_0_,
        comment0_.down as col_2_0_ 
    from
        comment comment0_ 
    left outer join
        post post1_ 
            on comment0_.post_id=post1_.id 
    where
        post1_.id=?
```

## Open Projection, Close Projection

OpenProjection은 @Value (롬복꺼 아님)와 SpEL를 사용해서 target(여기서는 Comment entity)의 필드를 새롭게 가공한 값을 가져다가 사용할 수 있다. 하지만 전체 필드(그래서 Open Projection임) 에 대한 쿼리가 날아가기 때문에 최적화는 되지 않는다.

```java
public interface CommentSummary {

        String getComment();

    int getUp();

    int getDown();

    @Value("#{target.up + ' ' +target.down}")
    String getVotes();

}
```

앞에서 살펴본 CommentSummary에 getVotes()를 추가하고, @Value (`import org.springframework.beans.factory.annotation.Value;`) 와 SpEL를 사용해서 target의 up과 + target의 down을 문자열과 합치는 연산을 하도록 만들었다.

```java
        @Test
    public void getComment() {

        Post post = new Post();
        post.setTitle("jpa");
        Post savedPost = postRepository.save(post);

        Comment comment = new Comment();
        comment.setComment("hello jpa");
        comment.setUp(10);
        comment.setDown(1);
        comment.setPost(savedPost);
        commentRepository.save(comment);

        commentRepository.findByPost_Id(1L).stream().forEach(c -> {
                    System.out.println(" ============================= ");
                    System.out.println(c.getVotes());
                }
        );
    }
```

Post와 Comment를 각각 만들어서 저장하고(연관관계 설정함), 그리고 `commentId` 로 조회해서 getVotes()의 어떤 값이 들어가는지 살펴보자.

```
Hibernate: 
    select
        comment0_.id as id1_0_,
        comment0_.best as best2_0_,
        comment0_.comment as comment3_0_,
        comment0_.down as down4_0_,
        comment0_.post_id as post_id7_0_,
        comment0_.up as up5_0_,
        comment0_.votes as votes6_0_ 
    from
        comment comment0_ 
    left outer join
        post post1_ 
            on comment0_.post_id=post1_.id 
    where
        post1_.id=?
 ============================= 
10 1

```

결과를 보면, 일단 쿼리가 전체 필드에 대해서 날아간다. 그리고 원했던 연산의 결과가 찍혔다. 그럼 기존의 Closed (제한적인) Projection을 하면서 연산할 수 있는 방법이 있지 않을까? 🤔

### Java8 부터 도입된 default Method를 활용하는 것

```java
public interface CommentSummary {

        ...생략

    default String getVotes() {
        return getUp() + " " + getDown();
    }

}

```

```
Hibernate: 
    select
        comment0_.comment as col_0_0_,
        comment0_.up as col_1_0_,
        comment0_.down as col_2_0_ 
    from
        comment comment0_ 
    left outer join
        post post1_ 
            on comment0_.post_id=post1_.id 
    where
        post1_.id=?
 ============================= 
10 1

```

위의 결과와 다르게, 딱 3개의 필드를 조회하는 쿼리와, 아래에 연산 결과까지 완벽하다!

## 다이나믹 프로젝션

> 만약에 요구 사항이 CommentSummary 뿐만 아니라, CommentOnly(오직 코멘트만 찾는 쿼리를 날리도록 한다면)

```java
@Repository
public interface CommentRepository extends JpaRepository<Comment, Long> {

    List<CommnetSummary> findByPost_Id(Long id);
    List<CommnetOnly> findByPost_Id(Long id);
}

```

이렇게 하고 싶지만, 사용할 수 없다. 그래서 클래스의 타입을 파라미터로 넘기는 Generic를 활용해서 변경할 수 있다.

```java
@Repository
public interface CommentRepository extends JpaRepository<Comment, Long> {

    <T> List<T> findByPost_Id(Long id, Class<T> type);
}

```

## 정리

엔티티의 특정 데이터만을 조회하는(최적화) Projection에 대해서 알아봤다. 예제에서는 Interface를 통해서 Projection을 했지만 클래스기반(DTO)를 통해서도 할 수 있다. 클래스기반은 코드의 양이 많아진다.(롬복으로 줄일 수 있지만). 특정 쿼리에서 연산한 결과를 받을 수도 있었다. (Open Projection vs Closed Projection) Open은 말 그대로 target.field를 하기 때문에 target에 해당하는 엔티티 전체를 조회한다. 그래서 최적화는 무의미. 그래서 default 메서드를 통해서 대체 할 수 있었다.

마지막으로, 여러 요구 사항에 맞는 DTO클래스, 혹은 인터페이스를 만들었을 때 중복이 발생하기 때문에 클래스의 타입을 파라미터로 넘겨서 제네릭으로 만들어서 사용하는 방법을 알아봤다.