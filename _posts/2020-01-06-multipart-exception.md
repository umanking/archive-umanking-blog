---
layout: post
title: "[Spring] Tomcat Multipart Exception 디버깅"
date: 2020-01-06 20:59 +0900
categories: [Spring]
tags: [Spring]
---

## 개요
몇 일째 간헐적으로 발생한 FileUpload 관련 이슈 사항을 공유하며, 어떤 것들을 느끼고 알게 되었는지 공유합니다.
(반전은, 글을 쓰는 지금도 문제의 해결책을 찾지는 못했습니다. ㅠㅠ)

## Server Exception Message
아래는 해당 Exception 메세지이다. 매번 발생하는 이슈가 아닌, 간헐적으로 발생하는 이슈다. 

```
Request processing failed; nested exception is org.springframework.web.multipart.MultipartException: 
Could not parse multipart servlet request; nested exception is java.io.IOException: 
org.apache.tomcat.util.http.fileupload.FileUploadBase$IOFileUploadException: 
Processing of multipart/form-data request failed. null
```

## 구글링!
`org.apache.tomcat.util.http.fileupload.FileUploadBase$IOFileUploadException` 이 키워드를 통해서 구글링을 했을 때 몇가지 해결 방안들이 있었다. 
1. form 태그의 `enctype="multipart/form-data"`
2. CommonMultipartResolver로 빈 등록 


첫번째 방법은 form 태그를 서버에 전송할때, 어떻게 encoding할지를 정하는 attribute이다.
두번째 방법은 기존의 SpringBoot에서는 여러 가지 빈들을 자동으로 등록해 준다. 그 중에 MultipartAutoConfiguration클래스를 통해서 기본 MultipartResolver 구현체를 StandardServletMultipartResolver를 쓰고 있다.

```java
    // MultipartAutoConfiguration 클래스 파일내 메소드 
    @Bean(name = DispatcherServlet.MULTIPART_RESOLVER_BEAN_NAME)
	@ConditionalOnMissingBean(MultipartResolver.class)
	public StandardServletMultipartResolver multipartResolver() {
		StandardServletMultipartResolver multipartResolver = new StandardServletMultipartResolver();
		multipartResolver.setResolveLazily(this.multipartProperties.isResolveLazily());
		return multipartResolver;
	}
```

## 그럼 StandardServletMultipartResolver랑 CommonsMultipartResolver 는 뭐가 다를까?🤔

`MultipartResolver` [자바 문서](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/multipart/MultipartResolver.html)를 보니까 아주 자세하게 잘 설명이 되어있다.
```
There are two concrete implementations included in Spring, as of Spring 3.1:

- CommonsMultipartResolver for Apache Commons FileUpload
- StandardServletMultipartResolver for the Servlet 3.0+ Part API
```
MultipartResolver는 인터페이스고, Commons~, Standard~ 각각이 구현체이다. `CommonsMultipartResolver`는 Apache 재단의 Commons 파일업로드 라이브러리를 위한 것이고, `StandardServletMultipartResolver`는 서블릿 3.0 이상의 API를 위한 것이였다. 그렇다면 합리적인 의심은 운영중인 서버가 Servlet 3.0 버전이 안맞는 걸까? 

## tomcat 버전과 서블릿 버전 스펙을 확인해 보니? 
![](/assets/images/tomcat.png)
tomcat8 버전을 사용하기 때문에 서블릿 스펙도 3.0 이상이다. 이것도 아니다.


## 답을 찾아야지
몇 일 더 디버깅을 해보고, 문제의 답을 찾아서 다시 posting을 수정해야겠다.