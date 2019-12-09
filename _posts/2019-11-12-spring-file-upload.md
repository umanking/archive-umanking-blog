---
layout: post
title: 내가 스프링을 학습하는 방법
date: 2019-11-12 21:04 +0900
---

뭐든 시작은 실무에서 적용할 때, 그때 비로소 동기부여가 되서 찾아보게 된다(가장 중요한 동기 라고 생각한다).
예를 들면 fileUpload 대충~ 아는 듯 하지만 정확하게 그리고 pivotal에서 어떻게 구현했는지에 대해 궁금해서 구글에 검색해봤다. 

[https://spring.io/guides/gs/uploading-files/](https://spring.io/guides/gs/uploading-files/)

해당 사이트에서 아주 친절하게 소스코드를 제공해 주면서 설명해준다. 

나는 이 프로젝트를 clone를 통해서 안에 소스 코드를 확인했다. 실제 프로젝트를 띄우면서 Debugging 하면서 무슨 기능을 하고, 어떻게 Spring MVC 단에서 처리를 하는지 살펴 보았다. 바로 여기가 가장 중요한 부분이다. 

예를 들면, 자기가 짜왔던 코드 스타일(사용하는 라이브러리, 패턴)을 고집하면 새로운 코드(남이 작성한 코드)를 읽는데 시간이 많이 걸리고, 잘 읽히지도 않는다. 

바로 모르는 부분을 찾아보고, 이렇게 짤 수도 이렇게 처리도 하는 구나! 정도를 파악한다. 이 코드를 보면서 잘 몰랐던 것들은 다음과 같다. 

- java.nio pacakge의 Path API
- Spring MVC redirectAttribute, flashAttribute
- @RequestMapping 의 정규식을 통한 매핑
- MvcUriComponentBuilder
- List로 받지 않고, Stream<?> 으로 받아서 처리하는 부분

역시나 공부할 주제는 정말 많은 것 같다.