---
layout: post
title: "[Spring] 스프링 부트 특징 3가지"
date: 2019-04-12 09:11:26
categories: [Spring]
tags: [spring]
redirect_from: 
- 2019/04/12/spring-concept/
- spring/spring-concept/
---

## 스프링 부트의 특징
- 의존성관리
- 자동설정
- 내장 웹서버



## 1. 의존성 관리 

스프링 부트는 starter- 를 통해서 의존성관리를 한다. `pom.xml` 파일에 상위에 정의되어있는 `spring-boot-starter-parent` 위에 `spring-boot-dependencies`  를 보면 다음과 같이 수많은 버전들을 관리 해준다. 

```xml
<properties>
	<activemq.version>5.15.9</activemq.version>
	<antlr2.version>2.7.7</antlr2.version>
	<appengine-sdk.version>1.9.73</appengine-sdk.version>
	//... 생략
<properties>
```


그래서 `pom.xml`에 `version` 를 명시하지 않아도, parent -> dependencies에 명시했던 버전 관리를 통해서 자동으로 버전이 등록된다. 인텔리J에서 클릭하면 해당 버전으로 이동하게 된다. 물론 다른 버전을 적용하고 싶으면 명시적으로 version을 명시적으로 적었을 때 오버라이딩이 된다. 



## 2. 자동설정

스프링 부트는 빈을 등록하는게 크게 2가지로 이루어진다. 

1. @ComponentScan을 통해서 @Component, @Serveice, @Repository .. 와 같은 컴포넌트 기반을 스캐닝해서 우리가 정의한 빈들을 등록한다. 
2. @EnableAutoConfiguration을 통해서, `META-INF` 하위에 있는 `spring.factories`  파일에 정의된 클래스들에 의해서 빈으로 등록된다. (@ConditionalOnMissingBean 같은 경우, 빈으로 등록되어있지 않은 경우, 저기에서 정의한 대로 빈으로 등록한다.) 





## 3. 내장 웹서버

스프링 부트는 내장 웹서버를 가지고(?) 있다. 메인 애플리케이션으로 띄울 수 있고, jar로 패키징된 파일 하나로 만들고, 해당 jar를 실행해서도 애플리케이션을 띄울 수 있다. 

스프링 부트는 != 웹서버는 아니다. 

여러 가지 설정을 할 수 있다. 

- server.port = 0 (random) 
- spring.main.web-application-type=none  -> 내장 웹서버로 실행하지 않음
- https 설정, http2.0 설정




