---
layout: post
title: Jackson, null 필드 무시하기
date: 2019-12-11 22:25 +0900
categories: [Spring]
tags: [spring]
redirect_from:
- /2019/12/11/jackson-null-property/
---
백단에서 데이터를 프론트 단에 넘길 때, 객체에 대한 property값이 널인 경우까지 넘어 간다. 
## 예제 코드 
```java 
public class Account {
    private String name;
    private int age;
}
```

```java
public void create_account_null_test(){
    Account account = new Account();
    account.setName(null);
    account.setAge(32);
    
    System.out.println(objectMapper.writeValueAsString(account));
}
```
다음과 같이 null 값의 해당하는 `name` 프로퍼티가 결과로 나온다.  
```java
{"name":null,"age":32}
```


## 1. 지역적으로 처리 
```java 
@JsonInclude(JsonInclude.Include.NON_NULL)
public class Account {

    //@JsonInclude(JsonInclude.Include.NON_NULL) //클래스 또는 해당 프로퍼티에 붙여준다.
    private String name;
    private int age;
}
```

## 2. 전역적으로 설정하는 방법 
```java
@Configuration
public class JacksonConfiguration {

    @Bean
    public ObjectMapper objectMapper() {
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);
        return objectMapper;
    }
}
```


## Ref
- [practical jackson configuration](stubbornjava.com/posts/practical-jackson-objectmapper-configuration)