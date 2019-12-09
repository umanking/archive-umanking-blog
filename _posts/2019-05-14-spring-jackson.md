---
layout: post
title: "[Spring]Jackson 어노테이션 사용하기"
category: 
- Spring
date: 2019-05-14 22:30:23
tags: 
- Spring
---

> 오늘은 Jackson 어노테이션에  대해서 알아보도록 하겠습니다. 



실무에서 아무래도 가장 많이 사용하지만, 잘 몰라서 자칫 헤멜 수 있는 부분에 대해서 정리차원에 글을 씁니다. 기본적으로 스프링 부트를 사용한다면, spring-boot-starter-web의 의존성에 jackson 라이브러리가 들어 있음을 확인할 수 있습니다. 

실제 객체를 통해서 Json을 생성하는 코드를 살펴보겠습니다. 



### Account 도메인 

```java
@Getter
@Setter
public class Account {
  
    private Long id;
    private String name;
    private String age;
  
}
```

회원 정보를 나타내는 Account 클래스에 다음과 같은 필드들이 있다고 봅시다. 해당 객체를 생성하고, Json String으로 변환하는 테스트 코드를 짜보겠습니다. 



### 기본 Json형 변환

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class DemoApplicationTests {

    @Autowired
    ObjectMapper objectMapper;

    @Test
    public void objectTest() throws JsonProcessingException {

        Account account = new Account();
        account.setId(1L);
        account.setName("andrew");
        account.setAge("32");

        String s = objectMapper.writeValueAsString(account);
        System.out.println(s);

        Assert.assertThat(s, containsString("andrew"));
        Assert.assertThat(s, containsString("name"));

    }
}
```

1. `@SpringBootTest `:  빈을 다 등록하는 통합테스트
2. `ObjectMapper` 를 `@Autowired`를 통해서 주입받아서 바로 사용 가능합니다. 이런게 가능했던 것은 SpringBoot Starter 의존성 때문에 
3. Account 객체를 만들어서 objectMapper의 `writeValueAsString()` 를 통해서 객체를 String형인 json으로 변환해 줍니다. 



```java
{"id":1,"name":"andrew","age": "32"}
```

다음과 같이 예쁘게 결과같이 나옵니다. 



### @JsonProperty 

해당 어노테이션은 Json 프로퍼티명을 선언한 필드명과 달리 내 마음대로 바꾸고 싶을 때 사용합니다. 예를 들면 json를 내려줄때, 그냥 name이 아닌 userName이라는 프로퍼티로 내려주고 싶은 경우를 생각해봅시다. 

```java
@Getter
@Setter
public class Account {
  
    private Long id;
  	@JsonProperty("userName")
    private String name;
    private String age;
  
}
```

요렇게 설정하면, 다음과 같이 프로퍼티가 변경되어서 내려가는 것을 확인 할 수 있습니다. 

```java
{"id":1,"userName":"andrew","age": "32"}
```



### @JsonIgnore

해당 어노테이션은 도메인의 특정 필드를 무시하고 싶을 때 사용합니다. 

```java
@Getter
@Setter
public class Account {
  
    private Long id;
  	@JsonProperty("userName")
    private String name;
  	private String password;
    private String age;
  
}
```

password 라는 필드가 추가되었습니다.

```java
				// 테스트 케이스 			
				Account account = new Account();
        account.setId(1L);
        account.setName("andrew");
        account.setAge("32");
				account.setPassword("123456");

				//결과
        String s = objectMapper.writeValueAsString(account);
        System.out.println(s);

        Assert.assertThat(s, containsString("andrew"));
        Assert.assertThat(s, containsString("name"));

```

이렇게 콘솔에 찍게되면, password 값이 그대로 노출이 됩니다. 물론 암호화를 하고 DB에 저장하는게 당연하겠지만, API를 만들어서 내려줄 때, password 항목을 굳이 노출할 필요는 없습니다. 이 경우에는 `@JsonIgnore` 를 통해서 해당 필드를 무시 할 수 있습니다. 



```java
@Getter
@Setter
public class Account {
  
    private Long id;
  	@JsonProperty("userName")
    private String name;
  	
  	@JsonIgnore
  	private String password;
    private String age;
  
}
```

이렇게 테스트를 하면 원래 기대하던 Password 프로퍼티값이 보이지 않게 됩니다. 



### @JsonPropertyOrder

내려주는 json의 프로퍼티의 순서를 지정하고 싶을 때

```java
@Getter
@Setter
@JsonPropertyOrder({"id", "age", "name"})
public class Account {


    private Long id;
    private String name;
  	@JsonIgnore
  	private String password;
    private String age;
}
```

클래스에 선언해주고, 필드명을 객체 형태로 대입한다.



### @JsonRawValue

json String갑을 넘겼을 때, Json형식으로 Raw한 형태로 변환시켜 준다. 

```java
@Getter
@Setter
@JsonPropertyOrder({"id", "age", "name"})
public class Account {


    private Long id;
    private String name;
  	@JsonIgnore
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
