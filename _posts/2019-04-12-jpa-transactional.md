---
layout: post
title: "Spring Data JPA - @Transactional를 통한 Optimization"
categories: [JPA]
tags: [jpa]
date: 2019-04-12 09:09:13
redirect_from: 
- 2019/04/12/jpa-transactional/
---
Spring Data Jpa에서 제공하는 `JpaRepository`의 구현체인 `SimpleJpaReposity`의 `@Transactional`을 어떻게 사용하는지 살펴보고, 실제 우리 서비스에서 어떻게 적용하면 좋을지에 대해서 생각해보자. 

## 1. JpaRepository의 구현체인 SimpleJpaRepository를 열어보면
JpaRepository의 구현체인 SimpleJpaRepositry를 살펴보면, 클래스 레벨에 `@Transactional(readOnly= ture)` 라고 붙어 있다. 모든 메서드는 기본은 `readOnly=ture` 로 하고, 변경사항이 있는 메서드들은 추가로 `@Transactional` 을 붙였다.

```java
@Repository
// 클래스 레벨은 기본 읽기 옵션으로 설정하고
@Transactional(readOnly = true)
public class SimpleJpaRepository<T, ID> implements JpaRepositoryImplementation<T, ID> {

    
    // 데이터의 변경사항이 있는 메서드 레벨에는 직접 @Transactional 어노테이션을 붙였다. 
    // default가 readOnly = false 이기 때문에
    @Transactional
    public void delete(T entity) {
        Assert.notNull(entity, "The entity must not be null!");
        em.remove(em.contains(entity) ? entity : em.merge(entity));
    }
```

## 2. `readOnly=true` 가 의미하는 것은? 
즉, 데이터의 변경이 일어나지 않는 조회하는 메서드인 경우에 `@Transactional(readOnly=true)` 이렇게 설정한다. 이렇게 설정하면 좋은 점은 Flush모드(데이터 싱크)의 NEVER(필요없음)로 설정하고, Dirty Checking을 하지 않기 때문에 성능이 좋아진다.

> @Transactional의 readOnly 속성은 현재 Session의 flush모드를 FlushType.MANUAL 로 해서 dirty checking하는 것은 disable 시킨다.


## 참고
- [Spring read-only transaction Hibernate optimization](https://vladmihalcea.com/spring-read-only-transaction-hibernate-optimization/)