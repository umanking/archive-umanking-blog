---
layout: post
title: "[Spring]HandlerMethodArgumentResolver 인터페이스"
date: 2019-07-25 19:08:12
categories: [Spring]
tags: [spring]
redirect_from: 
- 2019/07/25/spring-handlerMethodArgumentResolver/
- spring/spring-handlerMethodArgumentResolver/
---
## HandlerMethodArgumentResolver 

HandlerMethodArgumentResolver 인터페이스는 컨트롤러에서 파라미터 바인딩 해주는 역할을 합니다.

> Strategy Interface for resolving method parameters into argument values in the context of a given request.

요청의 메서드 파라미터를 아규먼트의 값으로 resolve하는 인터페이스입니다. 흔히, 컨트롤러단에서 @RequestParam , @RequestBody, @ModelAttribute 와 같은 어노테이션 요청이나 클래스/인터페이스로 파라미터로 선언한 경우에 바인딩 해줍니다. 



## 실제 구현체 예제- PageableHandlerMethodArgumentResolver

```java
@RestController("/api/v1/post")
public Class PostController {
  
  @Autowired
  private PostService postService;
  
  @GetMapping("/list")
  public List<Post> list(Pageable pageable){
  	return postService.list(pageable);
	}
 
}
```

게시판의 Post목록 데이터를 불러오는 list메서드를 생각해보면, 메서드의 인자로 Pageable(`package org.springframework.data.domain` 패키지에 있는)를 넘겼을 때 아무것도 한게 없는데 페이징이 되는 이유를 아시나요 ?



앞에서 설명했던 PageableHandlerMethodArgumentResolver 클래스가HandlerMethodArgumentResolver를 구현했기 때문에 메서드의 파라미터로 들어온 요청에 대한 Binding을 해당 구현클래스인 PageableHandlerMethodArgumentReoslver가 해줍니다. 그렇기 때문에 아무렇지 않게 사용할 수 있었던 것입니다. 





## 커스텀하게 구현하기 - MyCustomHandlerMethodArgumentResolver

이번에는 앞에서 설명한 HandlerMethodArgumentResolver를 내 입맛에 맞는 구현을 직접 해봅시다. 

```java
@Data
public class User{
  private final String name;
  private final String email;
}
```

이름과 이메일을 받는 User라는 클래스를 만듭니다. 



```java
public class MyCustomHandlerMethodArgumentResolver implements HandlerMethodArgumentResolver {
  @Override
  public boolean supportsParameter(MethodParameter parameter) {
    return User.class.isAssignableFrom(parameter.getParameterType());
  }

  @Override
  public User resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {
    // 바인딩할 객체를 조작할 수 있다. 
    return new User("andrew han", "umanking@gmail.com");
  }
}
```

HandlerMethodArgumentResolver를 구현을 합니다. 첫번째 supportsParameter메서드는 바인딩할 클래스를 지정해줍니다. 두번째, resolveArgument메서드는 바인딩할 객체를 조작할 수 있다. 여기에서는 새로운 User 클래스를 만들어서 이름과 이메일정보를 넘겼습니다. 



내 입맛에 맞게 구현한 MyCustomHandlerMethodArgumentResolver를 빈으로 등록합니다. 

```java
@Configuration 
public class WebMvcConfigure implements WebMvcConfigurer { 

  @Bean 
  public MyCustomHandlerMethodArgumentResolver myCustomHandlerMethodArgumentResolver() { 			return new MyCustomHandlerMethodArgumentResolver(); 
	} 
  
  @Override 
  public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
    resolvers.add(myCustomHandlerMethodArgumentResolver()) 
	} 
}
```



실제 요청을 날리게 되면 MyCustomHandlerMethodArgumentResolver에서 정의한 resolveArgument 메서드를 호출함으로써 `new User(“andrew”, “umanking@gmail.com”)` 의 값을 리던하게 되고 원하는 결과를 얻게 됩니다.

```java
@RestController("/api/v1/user")
public class UserController{
  
  @GetMapping
  public User getUser(User user){
    return user;
  }
}
```

## 참고

- http://wonwoo.ml/index.php/post/1092
- https://lelecoder.tistory.com/107
