---
layout: post
title: "[Spring] Profile 설정하기"
date: 2019-04-13 22:02:17
categories: [Spring]
tags: [Spring]
redirect_from: 
- 2019/04/13/spring-profile/
- spring/spring-profile/
---
> 스프링부트에서 각 stage마다 profile 설정을 알아보자

### `@Profile` 애노테이션은 어디에 설정?

- 빈으로 등록되어있는 클래스 레벨에 설정 가능
- @Configuration, @Component



```java
// TestConfiguration 
@Profile("test")
@Configuration
public class TestConfiguration {

    @Bean
    public String hello() {
        return "hello test";
    }
}

// ProdConfiguration
@Profile("prod")
@Configuration
public class BaseConfiguration {

    @Bean
    public String hello() {
        return "hello prod";
    }
}
```

2개의 Configuration클래스를 만들고, `hello` 라는 빈을 등록했다. 각각 테스트용, 프로덕트용 으로 다른 결과를 return 한다. 

```java
@Component
public class Runner implements ApplicationRunner {

    @Autowired
    private String hello;

    @Override
    public void run(ApplicationArguments args) {
        System.out.println(hello);
    }
}
```

hello 빈을 주입받아 출력한다. 당연히 어떤 profile로 띄울지에 대한 설정이 없기 때문에 예외가 발생한다. 

```
*************************
APPLICATION FAILED TO START
***************************

Description:

Field hello in com.example.demo.Runner required a bean of type 'java.lang.String' that could not be found.

The injection point has the following annotations:
	- @org.springframework.beans.factory.annotation.Autowired(required=true)


Action:

Consider defining a bean of type 'java.lang.String' in your configuration.
```



### 프로파일 활성화

1) application.properties의 propertiy값을 변경하는 방법 

```properties
spring.profiles.active=prod
```

prod 환경으로 springBoot를 띄우면 `hello prod`가 찍히고  test로 실행 시키면, `hello test`가 콘솔창에 찍힌다. 

```
hello prod
```



2) 패키징을 하고난 jar 파일을 실행할 때 arg로 넘겨서 실행하는 방법

```
$ mvn clean package -DskipTests
$ java -jar target/*.jar --spring.profiles.active=prod
```

이 경우에 application.properties의 spring.profiles.active=test로 해놓고 패키징을 해도, [Java] jar를 통해서arg로 넘기는 경우 오버라이딩이 되서, prod 프로파일이 적용됨을 알 수 있다. 



### 프로파일용 프로퍼티를 분리하기

application-{stage}.properties 형식의 프로파일용 프로퍼티를 분리할 수 있다. 



### 프로파일 Include

```properties
# application.properties 파일
spring.profile.include = realdb
```

이런 식으로 작성하게 되면, application-realdb.properties 정보를 포함하는 application.properties 파일이 만들어 진다. 



### Best Practice 는 무엇? 

- application-common.properties
- application-test.properties
- application-local.properties
- application-alpha.properties
- application-prod.properties

이런식으로 properties파일을 각 스테이징 별로 나누고, 공통으로 사용하는 app에 대한 정보나, 외부 라이브러리 연동하는 app-key같은 정보를 적용할 수 있다. 또한, 각 stage별로 common의 properties 파일을 includ하여서 공통인 정보를 포함하게 할 수 있다. 



## 정리

profile은 각 Stage환경에 따라 구성해야 하는 빈설정, 정보들이 상이하게 다른 경우에 적용할 수 있다. 각 스테이징환경에 맞는 Configuration파일을 구성, profile을 실행할 때는 spring.profiles.actives 옵션을 통해서 설정했다. 각 스테이징 별 정보들이 많을 때는 프로파일용 프로퍼티를 각각 만들어서 분리했다. 