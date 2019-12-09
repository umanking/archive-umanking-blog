---
layout: post
title: "[JPA] save()메서드로 살펴보는 persist와 merge 개념"
date: 2019-04-12 09:08:33
category: 
 
- JPA
---
> JpaRepository의 구현체 SimpleJpaRepository에서 save()메서드가 내부적으로 어떻게 동작하는지 알아보자.

`SimpleJpaRepository`의 save() 메서드

```java
    @Transactional
    public <S extends T> S save(S entity) {
        if (this.entityInformation.isNew(entity)) {
            this.em.persist(entity);
            return entity;
        } else {
            return this.em.merge(entity);
        }
    }
```

entityInformation에서 새로운 entity이면 `persist()` 를 그게 아니면 `merge()`를 호출한다.  
merge는 한번 persist 상태였다가 detached 된 상태에서 그 다음 persist 상태가 될 때, merge 한다고 한다.

## 테스트

실제 Post 엔티티를 만들고, 테스트를 해보자

```java
@Data
@Entity
public class Post{

    @Id
    @GeneratedValue
    private Long id;
    private String title;

    @OneToMany
    private List<Comment> comments = new ArrayList<>();

    public Post publish() {
        this.registerEvent(new Event(this));
        return this;
    }
```

```java
// PostRepository
public interface PostRepository extends JpaRepository<Post, Long> {
}
```

```java
    @Test
    public void saveAndMergeTest() {
        // 1
        Post post = new Post();
        post.setTitle("jpa");
        Post savedPost = postRepository.save(post); // persist

        // 2
        assertThat(entityManager.contains(post)).isTrue();
        assertThat(entityManager.contains(savedPost)).isTrue();
        assertThat(post == savedPost);

        // 3
        Post post2 = new Post();
        post2.setId(post.getId());
        post2.setTitle("hibernate");
        Post updatedPost = postRepository.save(post2);// merge

        // 4
        assertThat(entityManager.contains(post2)).isFalse();
        assertThat(entityManager.contains(updatedPost)).isTrue();
        assertThat(post2 == updatedPost);

        postRepository.findAll();
    }
```

```java
        // 1
        Post post = new Post();
        post.setTitle("jpa");
        Post savedPost = postRepository.save(post); // persist
```

처음에 post 객체를 만들고 save()를 하게 되면 내부적으로 persist()를 호출한다. 그 이유는 Post엔티티에서 @Id 어노테이션을 붙인 id를 통해서 엔티티의 상태를 관리한다. 여기에서는 당연히 처음 객체를 만들었으므로 persist 상태가 된다.

```java
        // 2
        assertThat(entityManager.contains(post)).isTrue();
        assertThat(entityManager.contains(savedPost)).isTrue();
        assertThat(post == savedPost);
```

2번째 테스트 케이스는 entityManager를 통해서 앞에서 저장한 post엔티티가 포함되어있는지를 확인하는 코드다. 마지막 동일성을 비교하는 코드는 save()메서드의 반환된 결과와 파라미터에 넘어간 객체의 동일성을 보장한다.

```
        // 3
        Post post2 = new Post();
        post2.setId(post.getId());
        post2.setTitle("hibernate");
        Post updatedPost = postRepository.save(post2);// merge
```

이번에는 id를 새롭게 생성하는 게 아니라, 기존의 post.getId()를 통해서 가지고 온다. 그렇기 때문에 save()를 호출하게 되면, persist()가 아닌 merge()를 호출하게 된다.

```java
        // 4
        assertThat(entityManager.contains(post2)).isFalse();
        assertThat(entityManager.contains(updatedPost)).isTrue();
        assertThat(post2 == updatedPost);
```

여기서 중요한 것이. 파라미터로 넘긴 post2객체는 entityManger에 존재하지 않는 것이다. return 된 결과만을 entittyManager가 관리한다. 객체를 새로 post2로 만들었어도, save()메서드는 하이버네이트에서 id를 통해서 엔티티의 상태를 관리하기 때문에, 여기에서는 엔티티의 id가 이전에 만들어둔 값을 가지고 와서 넣어준다.

## 정리

persist()를 호출하는 경우에는 리턴한 객체 필드를 사용하지 않아도 된다.  
merge()를 호출하는 경우에는 **반드시** 리턴된 결과를 통해서 다음 작업을 진행해야지, entityManager가 관리하는 상태를 추적할 수 있다.

결론, 왠만하면 persist(), merge()를 매번 코딩할 때마다 구분하기 힘드니 무조건 반환받고 그 값을 통해서 코딩하자!