---
layout: post
title: 쿠키란? 
category:
  - Web
date: 2019-08-06 12:08:19
tags: [Web]
---
> Web 쿠키와 세션에 대해서 알아보자. 



## HTTP 쿠키

서버가 웹브라우저에 전송하는 작은 데이터 조각이다. 쿠키를 통해서 사용자는 같은 웹사이트 방문시, 새롭게 읽히고 수시로 새로운 정보로 바뀔 수 있다. Http프로토콜은 Stateless(무상태)이기 때문에 상태정보를 유지해야할 필요성 때문에 쿠키라는 개념이 등장했다.



주로 세가지 용도로 사용된다. 

- **세션관리**: 서버에 저장해야할 로그인, 장바구니 등의 정보관리
- **개인화**: 사용자 선호, 테마등의 셋팅 
  ex) MDN 사이트에서 언어 설정을 바꿀때마다 Cookies의 `django_language` 값이 변동됨. 다음번에 사이트 들어올때마다 바뀐언어로 유지된다. 
- **트래킹**: 사용자 행동을 기록하고 분석하는 용도



## 쿠키의 구성요소

- key: 쿠키의 이름
- value: 쿠키의 저장된 값 
- expires: 만료일 + 시간
- domain: 쿠키가 사용되는 도메인을 지정 | 어떤 domain에서 동작할 것인가에 대해 제한
  `domain=umanking.github.io`
  이렇게 함으로써, 다른 도메인에 대한 쿠키 사용을 하지 못하게 함
- path: 어떤 path에서 동작할 것인가를 제한하는 것
- secure & HttpOnly: secure 는 웹브라우저와 웹서버가 https통신하는 경우만 웹브라우저가 쿠키를 서버로 전송하는 옵션. HttpOnly는 자바스크립트의 document.cookie를 이용해서 쿠키에 접속하는 것을 막는 옵션입니다. 쿠키를 훔쳐가는 행위를 막기 위한 방법입니다.



## 쿠키 프로세스 (Set-Cookie) 

1. 클라이언트는 서버로 request 요청을 보낸다.
2. `응답의 헤더`에 `Set-Cookie` 를 key,value로 설정해서 클라이언트로 보낸다.(클라이언트에게 쿠키를 저장하라고 전달한다.)
3. 이제 서버로 전송되는 모든 요청과 함께, 브라우저는 `Cookie 헤더`를 사용하여 서버로 이전에 저장했떤 모든 쿠키들을 회신한다. 



## 쿠키의 종류

- Session Cookie: 브라우저를 종료하게 사라진다.
- Prmanent Cookie: Expires(만료 날짜)나 명시한 시간(Max-age) 이후에 사라진다. 브라우저를 종료해도 해당 만료날짜가 되지 않았다면 살아있다. 



## 쿠키의 단점 

- 쿠키에 대한 정보를 매 헤더에 추가하여 보내기 때문에 트래픽이 상당함
- 결제 정보등을 쿠키에 저장하였을때 쿠키가 유출되면 보안 문제 발생
  ex) 크롬 [mdn 사이트](https://developer.mozilla.org/ko/)에서 로그인 한 후에 - sessionid 쿠키값을 복사하고, 파이어폭스에서 해당 사이트의 sessionid 쿠키값을 셋팅하면 그대로 로그인 됨(userId, Password 어떤 것도 알지 못하지만 로그인에 성공한 sessionid값을 알게되면 로그인이 가능하다.)