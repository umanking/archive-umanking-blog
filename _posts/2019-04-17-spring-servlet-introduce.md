---
layout: post
title: "[Spring] 서블릿에 대해서 알아보자"
date: 2019-04-17 09:37:42
categories: [Spring]
tags: [Spring]
tages: [spring]
redirect_from: 
- 2019/04/17/spring-servlet-introduce/
- spring/spring-servlet-introduce/
---

> Servlet에 대해서 알아보자



### 서블릿이 뭐냐?

- 자바엔터프라이즈에서 웹애플리케이션 개발용 스팩과 API를 제공
  - HttpServelt이 중요함



### 서블릿 특징

- 서블릿은 프로세스 하나당 동작(CGI)하는게 아니라, 쓰레드 기반으로 자원을 공유한다. 
  - 빠르고, 플랫폼 독립적(OS와 별개), 보안, 이식성이 좋다
- 서블릿은 우리가 실행할 수 없다. 서블릿 컨테이너가 실행함



### 서블릿 컨테이너 

- tomcat, jetty, undertow 등..
- 세션관리
- 네트워크 서비스
- MIME 기반 메세지 인코딩/ 디코딩
- 서블릿 생명주기 관리



### 서블릿 라이프 사이클 



- 서블릿 컨테이너가 서블릿 인스턴스의 init()메서드를 최초에 호출하여 초기화 한다. 
  - 그 다음부터 이 과정 생략
- 서블릿 초기화 후에 클라이언트의 요청을 처리 한다. 요청은 별도의 쓰레드로 처리한다. 서블릿 인스턴스의 service()메서드를 호출한다. 
  - HTTP요청을 받고 클라이언트로 보낼 HTTP응답을 만든다. 
  - service() 메서드는 HTTP 메서드의 따라 doGet(), doPost()등으로 처리를 위임한다.
  - 고로 doGet(), doPost()를 구현한다. 
- 서블릿 컨테이너 판단에 따라 해당 서블릿을 메모리에 내려야 할 시점에 destroy() 메서드를 호출한다. 

