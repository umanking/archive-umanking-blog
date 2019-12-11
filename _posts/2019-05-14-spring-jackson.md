---
layout: post
title: "Jackson 어노테이션 사용"
category: 
- Spring
date: 2019-05-14 22:30:23
tags: 
- Spring
---

> 오늘은 Jackson 어노테이션에  대해서 알아보도록 하겠습니다. 

다음과 같이 스프링 부트를 사용하면 `spring-boot-starter-web`의존성에  `jackson-databind`라이브러리가 추가 되어 있다.

![](/assets/images/jackson.png)

## Account 도메인
```java
@Getter
@Setter
public class Account {  
    private Long id;
    private String name;
    private String age;
}
```


## ObjectMapper 기본적인 사용

```java
  Account account = new Account();
  account.setId(1L);
  account.setName("andrew");
  account.setAge("32");

  String accountString = objectMapper.writeValueAsString(account);
  console.log(accountString);

}
```

결과 값
```java
{"id":1,"name":"andrew","age": "32"}
```

## 1. @JsonProperty 
**Json 프로퍼티명을 선언한 필드명과 달리 내 마음대로 바꾸고 싶을 때 사용합니다.** 

```java

  @JsonProperty("userName") // Json 응답을 name이 아닌 userName으로 만들어 주고 싶을 때
  private String name;
  private String age;

```

결과 값
```java
{"id":1,"userName":"andrew","age": "32"}
```

## 2. @JsonIgnore
해당 어노테이션은 도메인의 특정 필드를 무시하고 싶을 때 사용합니다. 


```java
  @JsonIgnore
  private String password;
  
  private String age;
```

이렇게 테스트를 하면 원래 기대하던 Password 프로퍼티값이 보이지 않게 됩니다. 



## 3. @JsonPropertyOrder
내려주는 json의 프로퍼티의 순서를 지정하고 싶을 때

```java
@Getter
@Setter
@JsonPropertyOrder({"id", "age", "name"})
public class Account {
    private Long id;
    private String name;
    private String age;
}
```
## 4. @JsonRawValue
json String값을 넘겼을 때, Json형식으로 Raw한 형태로 변환시켜 준다. 

```java
@Getter
@Setter
@JsonPropertyOrder({"id", "age", "name"})
public class Account {
  
  private Long id;
  private String name;
  private String password;
  private String age;

  @JsonRawValue
  private String type;
}
```

type은 Account가 어디를 통해서 접속했는지를 판단해주는 필드 입니다. 

```java
  @Test
  public void objectTest() throws JsonProcessingException {

      Account account = new Account();
      account.setId(1L);
      account.setName("andrew");
      account.setAge("32");
      account.setPassword("1234");

      account.setType("{\"device\":\"mobile\"}");
      String s = objectMapper.writeValueAsString(account);
      System.out.println(s);

      Assert.assertThat(s, containsString("andrew"));
      Assert.assertThat(s, containsString("Name"));
  }
```

 만약에 @JsonRawValue를 사용하지 않았다면 결과는 다음과 같습니다 

```java
{"id":1,"Name":"andrew","type":"{\"device\":\"mobile\"}"}
```

우리가 원하지 않는 이스케이프 문자까지 다 포함되어서 나오는 것을 확인할 수 있습니다. `@JsonRawValue` 를 넣고 다시 실행하면

```java
{"id":1,"Name":"andrew","type":{"device":"mobile"}}
```



### 정리

오늘은 실무에서 API 스펙에 따라서 결과를 만들어 내는 Json(Jackson)의 기본적인 사용법을 알아 봤습니다. [jackson 어노테이션 예제 ](https://www.baeldung.com/jackson-annotations) 를 참고 했으며, 분량이 많은 관계로, 다음 2편에서 더 정리하도록 하겠습니다. 
