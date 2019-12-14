---
layout: post
title: "[JPA] 도메인 클래스 컨버터란?"
categories: JPA
redirect_from: 
- 2019/04/12/jpa-domain-class-converter/
date: 2019-04-12 09:04:25
---
> 도메인 클래스 컨버터에 대해서 알아보자.

도메인 클래스 컨버터는 Spring Data Jpa를 사용하게 되면 Common 프로젝트에 있는 클래스 인데, 간단하게, id-> 엔티티 타입으로 변환 해주거나, 엔티티 -> id 로 변환해 주는 컨버터를 제공해준다. 예제를 통해서 동작하는 지 살펴보자.

## 예제 코드

```java
@Data
@Entity
@Table
public class Post {

    @Id
    @GeneratedValue
    private Long id;
    private String title;
}

```

간단한 id, Title만 있는 Post 엔티티를 정의한다.

```java
public interface PostRepository extends JpaRepository<Post, Long> {
}
```

JpaRepository를 상속받는 레포지토리를 만든다.

```java
@RestController
public class PostController {

    @Autowired
    private PostRepository postRepository;

    @GetMapping("/post/{id}")
    public String getPost(@PathVariable Long id) {
        Optional<Post> byId = postRepository.findById(id);
        Post post = byId.get();
        return post.getTitle();
    }
}
```

/post/{id} 로 요청이 왔을때, DB에서 id로 post데이터를 조회해서, title 을 return해 주는 코드

```java
@SpringBootTest
@RunWith(SpringRunner.class)
@AutoConfigureMockMvc
public class PostControllerTest {

    @Autowired
    private PostRepository postRepository;
    @Autowired
    MockMvc mockMvc;

    @Test
    public void getPost() throws Exception {
        Post post = new Post();
        post.setTitle("jpa");
        postRepository.save(post);

        mockMvc.perform(get("/post/" + post.getId()))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(content().string("jpa"));

    }
}
```

해당 컨트롤러에 맞는 테스트 케이스를 작성한다. Post 하나를 만들고 저장한 후에, /post/{id} 로 요청했을 때 응답 상태와 응답 컨텐츠의 타이틀이 저장했던 것과 일치하는지를 테스트 한다.

여기 까지는 성공한다.

## 도메인 클래스 컨버터

이제 도메인 클래스 컨버터를 살펴보자.

```java
public class DomainClassConverter<T extends ConversionService & ConverterRegistry>
        implements ConditionalGenericConverter, ApplicationContextAware {

    private final T conversionService;
    private Repositories repositories = Repositories.NONE;
    private Optional<ToEntityConverter> toEntityConverter = Optional.empty();
    private Optional<ToIdConverter> toIdConverter = Optional.empty();
...

```

여기에서 ToEntityConverter와 ToIdConverter 클래스가 위에 컨트롤러에서 작성했던 역활과 동일한 일을 한다. 그렇기 때문에 아까전의 컨트롤러 코드에서 다음과 같이 바꿀 수 있다.

```java
...// 생략
    @GetMapping("/post/{id}")
    public String getPost(@PathVariable("id") Post post) {
        return post.getTitle();
    }
```

id를 받아서 -> Post엔티티로 바꿔주는 컨버터가 실행되기 때문에, 이렇게 만들어도 테스트케이스를 돌려도 성공한다.

> 주의: @PathVariable에 프로퍼티로 "id"를 꼭 명시해줘야 한다.