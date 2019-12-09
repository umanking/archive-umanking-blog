---
layout: post
title: "[JPA] transactional 살펴보기"
date: 2019-04-12 09:09:13
 
---
> Spring의 Transaction에 대해서 알아보자.

JpaRepository를 상속받아서 사용하는 Repository인 경우에, 기본 @Transactional 어노테이션이 적용되어 있다.

```java
public interface CommentRepository extends JpaRepository<Comment, Long> 
```

```java
@Repository
@Transactional(readOnly = true)
public class SimpleJpaRepository<T, ID> implements JpaRepositoryImplementation<T, ID> {
```

JpaRepository의 구현체인 SimpleJpaRepositry를 살펴보면, 클래스 레벨에 `@Transactional(readOnly= ture)` 라고 붙어 있다. 모든 메서드는 기본은 `readOnly=ture` 로 하고, 변경사항이 있는 메서드들은 추가로 `@Transactional` 을 붙였다.

```java
... //생략

@Transactional
    public void delete(T entity) {

        Assert.notNull(entity, "The entity must not be null!");
        em.remove(em.contains(entity) ? entity : em.merge(entity));
    }

... //생략
```

즉, 데이터의 변경이 일어나지 않는 조회하는 메서드인 경우에 `@Transactional(readOnly=true)` 이렇게 설정한다. 이렇게 설정하면 좋은 점은 Flush모드(데이터 싱크)의 NEVER(필요없음)로 설정하고, Dirty Checking을 하지 않기 때문에 성능이 좋아진다.