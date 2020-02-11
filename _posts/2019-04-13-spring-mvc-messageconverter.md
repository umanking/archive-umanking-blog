---
layout: post
title: "[Spring] 메세지 컨버터"
date: 2019-04-13 22:07:05
categories: [Spring]
tags: [spring]
redirect_from: 
- 2019/04/13/spring-mvc-messageconverter/
---
> spring web mvc의  HttpMessageConverter에 대해서 알아보자!



### 메세지 컨버터란? 

http 요청 본문을 객체로, 객체를 http 응답 본문으로 변경해 주는 것

> 예제에서는 요청/응답 본문은 json 형식으로, 객체는 User를 통해서 살펴보자!



### 종류

- JsonMessageConverter
- StringMessageConverter 
- 등등...



```java
@Data
public class User {

    private Long id;
    private String username;
    private String password;

}
```



```java
@Controller
public class UserController {

    @PostMapping("/user/create")
  	@ResponseBody
    public User create(@RequestBody User user) {
        return user;

    }
}
```

- @ResponseBody가 없으면 -> BeanNameViewResolver를 탄다
- @ResponseBody가 있으면 -> MessageConverter를 탄다.



```java
@RunWith(SpringRunner.class)
@WebMvcTest
public class UserControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void createUser_JSON() throws Exception {
        String user = "{\"username\":  \"andrew\",\n" +
                "  \"password\": 1234" + "}";

      	mockMvc.perform(post("/user/create")
                .contentType(MediaType.APPLICATION_JSON_UTF8)
                .accept(MediaType.APPLICATION_JSON_UTF8)
                .content(user))
                 
                .andDo(print())
                .andExpect(jsonPath("$.username", Matchers.is(equalTo("andrew"))))
                .andExpect(status().isOk());
    }
}
```

content-type과 accept타입을 json으로 설정하고, user라는 json데이터를 요청본문에 담아서 해당 url를 호출해서 나온 결과값을 jsonpath 를 통해서 검증해 보자! 



