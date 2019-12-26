---
layout: post
title: javascript event loop
date: 2019-12-19 00:46 +0900
---

> 자바스크립트가 브라우저에서 어떻게 비동기 처리를 하는지 알아보자

먼저 자바스크립트는 싱글스레드이다!
싱글스레드임에도 불구하고 어떻게 비동기 처리를 하나? (Ajax, setTimeout, ....)

이것들을 이해하기 위해서는 `heap` `call stack` `WEB APIs` `event loop` `callback queue`를 이해해야 한다. 


![](https://miro.medium.com/max/752/1*7GXoHZiIUhlKuKGT22gHmA.png)

먼저 `heap`은 객체가 생성이 되면 heap영역에 올라간다. 
`call stack`은 함수가 호출이 되면 stack영역에 호출한 순서대로 하나씩 쌓이고, return을 하게되면 스택에서 하나씩 사라진다. 


## 동작 방법
자바스크립트는 싱글쓰레드로 함수를 호출하게 되면 call stack에 차곡차곡 쌓이게된다. 
그러면 비동기적으로 처리는 어떻게 하나? 가장 흔한 예제가 바로 `setTimeout()` 이다.

setTimeout은 Web API에서 Timer를 통해서 처리하게 되고, 기존에 call stack에 있던 setTimeOut은 사라지게 된다. 
Timer를 통해서 deplay 처리가 되면, event loop를 통해서 `callback queue`에 쌓이게 된다. 
(이름에서도 알 수 있듯이 큐라는 자료구조이기 때문에, 순차적으로 쌓이고, 쌓인 순서대로 stack에 재할당되서 실행이 된다 - 이것 또한 event loop가 처리한다)