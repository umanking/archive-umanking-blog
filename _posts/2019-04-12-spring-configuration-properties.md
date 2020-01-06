---
layout: post
title: "[Spring] @ConfigurationProperties를 이용한 외부 설정 정보 클래스 만들기"
date: 2019-04-13 22:04:37
category: 
 
- Spring
tags: 
- Spring
redirect_from: 
- 2019/04/12/spring-configuration-properties/
---

> 스프링부트에서 외부설정을 Configuration 클래스로 만들어서 가지고 오는 상황을 살펴 보자.



application.properties 에 다음과 같이 정의했다. 

```java
andrew.name=andrew
andrew.age=${random.int(0,100)}
andrew.fullName=${andrew.name} Han
```

`andrew` 라는 prefix로 설정하고, `name`, `age`, `fullName` 속성의 값을 설정했다. 실제 코드에서 사용할 때는 다음과 같다. 

```java
		@Value("${andrew.name}")
    private String name;

    @Value("${andrew.age}")
    private int age;

    @Value("${andrew.fullName}")
    private String fullName;

// 생략
```



### 하지만, 가지고 와야 하는 변수가 많은 경우에는 어떻게 할까? 

관리할 property를 정의할 class를 하나 만든다. 

```java
@ConfigurationProperties(prefix = "andrew")
@Component
public class AndrewConfiguration {
    String name;
    int age;
    String fullName;
  
  	// getter, setter
}
```

`@ConfigurationProperties` 를 클래스레벨에 선언해 주고, 빈으로 등록하기 위해 `@Component` 를 달아준다. `application.properties` 에 정의했던 prfix명을 그대로 입력해준다. 



`Spring Boot Configuration Annotaition Processor not found in classpath`  문구가 떴다. 문서를 열어서 해당 어노테이션 프로세서를 추가하자 

```xml
<dependency>
  	<groupId>org.springframework.boot</groupId>
  	<artifactId>spring-boot-configuration-processor</artifactId>
  	<optional>true</optional>
</dependency>
```



이제 기존의 `@Value(${andrew.name})` 이런 코드들 대신, 해당 `AndrewConfiguration` 빈을 주입받아서 getter를 통해서 접근할 수 있다. 



```java
@Component
public class Runner implements ApplicationRunner {

	 // 불필요한 코드
   /* @Value("${andrew.name}")
    private String name;

    @Value("${andrew.age}")
    private int age;

    @Value("${andrew.fullName}")
    private String fullName;*/

    @Autowired
    private AndrewConfiguration andrew;


    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("=========================================");
        System.out.println("name: " + andrew.getName());
        System.out.println("age: " + andrew.getAge());
        System.out.println("fullName: " + andrew.getFullName());
        System.out.println("=========================================");
    }
}
```





## 정리

`application.properties`에서 관리하는 값들이 많을 때, 기존의 `@Value`  가 아닌, 클래스로 properties에 속성을 정의해서 빈으로 등록하고 사용하는 방법에 대해서 알아보았다. 