---
layout: post
title: "Spring, RestTemplate, WebClient 차이"
date: 2020-01-20 22:11 +0900
categories: [Spring]
tags: [Spring]
---

# 개요
오늘은 RestTemplate 과 WebClient의 차이에 대해서 알아보겠습니다. 
둘다 다른 서비스에 HTTP 콜을 합니다. 

`RestTemplate`는 자바 서블릿API 기반으로, 요청당 쓰레드가 생성되는 모델입니다. 
반면에 `WebClient`는 논블록킹으로 비동기로 동작하고, Spring Reactive 프레임워크를 통해서 제공합니다. 
참고로 블록킹과 논블록킹에 관한 매우 직관적인 설명이 있어서 아래 동영상을 첨부합니다. 

> ※ 블록킹과 논블록킹에 관한 동영상 (자막 켜고 보세요!)
{% include youtubePlayer.html id='zXBTTOQ_iSQ?t=1185' %}



# 실습
- GET /items 아이템 목록을 가져오는 HTTP 콜을 하는 경우를 생각합니다.
- 하나는 일반 RestTemplate를 통한 콜을 할 것이고
- 다른 하나는 WebClient를 통한 콜을 할 것입니다.


