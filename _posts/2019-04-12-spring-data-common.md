---
layout: post
title: "[Spring] Spring Data Common 프로젝트 살펴보기"
date: 2019-04-12 09:03:09
category: 
 
- Spring
tags: 
- Spring
---
> Spring Data는 여러 가지 프로젝트로 되어 있다. 그중에서 Common 프로젝트에 대해서 알아보자.

[##_Image|kage@dkdxVK/btqt838FRZS/rNf2fK7Nikct474OtbIxH1/img.png|alignCenter|data-filename="spring-data.png"|_##]

Spring Data는 여러개의 프로젝트로 구성되어 있다. 그 중에서 JPA에서 공통으로 사용하는 Common 프로젝트에 대해서 알아보자. (물론 다른 Redis,MongoDb 저장소에서도 공통으로 사용하니까 Common으로 따로 뺀 것)

## Repository 상속 관계

```java
@Repository
public interface PostRepository extends JpaRepository<Post, Long> {
}
```

실제 특정 엔티티의 Repository를 구성할 때 다음과 같이 JpaRepository 인터페이스를 상속받아서 사용했다.

```java
@NoRepositoryBean
public interface JpaRepository<T, ID> extends PagingAndSortingRepository<T, ID>, QueryByExampleExecutor<T> {
    List<T> findAll();

    List<T> findAll(Sort var1);

    List<T> findAllById(Iterable<ID> var1);
    ...

```

```java
@NoRepositoryBean
public interface PagingAndSortingRepository<T, ID> extends CrudRepository<T, ID> {
    Iterable<T> findAll(Sort var1);

    Page<T> findAll(Pageable var1);
}
```

```java
@NoRepositoryBean
public interface CrudRepository<T, ID> extends Repository<T, ID> {
    <S extends T> S save(S var1);

    <S extends T> Iterable<S> saveAll(Iterable<S> var1);

    Optional<T> findById(ID var1);

    boolean existsById(ID var1);

    Iterable<T> findAll();

    ...

```

```java
@Indexed
public interface Repository<T, ID> {

```

## Repository 구조

```
Repository
CrudRepository
PagingAndSortRepository
=========================  ▲ 위에 부분이 Common 프로젝트 부분
JpaRepository
PostRepository
```

다음과 같이 상속 구조를 가지고 있다. 최상위 `Repository`는 마커용으로 아무 의미도 없다.  
`CrudRepository`는 엔티티의 기본적인 crud를 담당. 페이징관련된 부분은 `PagingAndSortRepository`가 담당한다.

## 예제 코드

Post 엔티티를 만들고 `@DataJpaTest` 슬라이싱 테스트를 해보자. 슬라이싱 테스트를 하는 이유는 Repository 관련 된 빈만 등록하기 때문에 시간을 절약한다.

```java
@Data
@Entity
@Table
@ToString
public class Post {

    @Id
    @GeneratedValue
    private Long id;
    private String title;

    ...
}
```

```xml
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>test</scope>
    </dependency>
```

테스트용 h2 데이터베이스를 사용한다.

## 테스트 케이스

```java
@RunWith(SpringRunner.class)
@DataJpaTest //레포지토리 관련된 빈만 등록하는 슬라이싱 테스트 (통합테스트와 다름)
public class JpaTest {

    @Autowired
    private PostRepository postRepository;

    @Test
    public void crudTest() {

        Post post = new Post();
        post.setTitle("new Post");

        Post retrivedPost = postRepository.save(post);
        Assert.assertNotNull(retrivedPost);

        List<Post> allPost = postRepository.findAll();

        Page<Post> page = postRepository.findAll(PageRequest.of(0, 10));

        Assert.assertEquals(page.getTotalElements(), 1); // 전체 데이터 개수
        Assert.assertEquals(page.getTotalPages(), 1); //전체 페이지
        Assert.assertEquals(page.getNumber(), 0); //시작 페이지 0부터 시작
        Assert.assertEquals(page.getSize(), 10); //페이지의 컨텐츠 사이즈 10개 단위로 끊음
        System.out.println(page.getContent()); // [Post(id=1, title=new Post, comments=[])]

```

findAll() 메서드는 파라미터가 없는 경우에 전체 엔티티를 조회하지만(실제 업무에서 거의 쓰이지 않음, 테스트 용도를 제외하고) 인자로 Pageable를 넘기는 경우에는 페이징에 맞게 데이터를 조회할 수 있다.

```java
    Page<T> findAll(Pageable pageable);
```

## 참고자료

- [백기선님 인프런 - spring-data-jpa](