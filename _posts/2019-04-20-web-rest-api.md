---
layout: post
title: RESTful API 가이드
date: 2019-04-20 20:46:14
categories: [Web]
tags: [web]
redirect_from: 
- 2019/04/20/web-rest-api/
---
![](https://cdn.crunchify.com/wp-content/uploads/2012/10/Crunchify.com-RESTful-Introduction.png)
> HTTP REST API에 대해서 알아보자


### REST의 기본 요소 

- 리소스
- 메서드
- 메세지 

예제를 하나 들어보자

```
POST /api/v1/users

{
  "users":{
    "name":"andrew"
  }
}
```

POST 방식으로 `api/v1/users`  요청하고 바디로 해당 데이터를 넘긴다. 

- 리소스: /api/v1/users  (URI)
- 메서드 : POST 
- 메세지: users의 Body 정보





### HTTP 메서드

| 메서드 | 의미   | Idempotent |
| ------ | ------ | ---------- |
| POST   | create | No         |
| GET    | select | Yes        |
| PUT    | update | Yes        |
| DELETE | delete | Yes        |

HTTP 메서드가 각각 행위를 정의한다. idempotent는 같은 동작을 수행했을 때 결과가 같은 경우를 Idempotent 하다고 말한다. POST는 매번 바디정보로 새롭게 만들기 때문에 Idempotent 하지 않고, 매번 조회를 하지만 count++ 를 증가시키는 로직이 추가된 API라면 이 경우도 Idempotent하지 않다고 한다. 



> Idempotent설명은 왜? (Stateless)
>
> REST는 각 개별 API를 상태 없이 수행하게 된다. 상태를 HttpSession와 같은 컨텍스트 저장소에 상태 정보를 저장하지 않음을 의미한다. 



### URL 동사 보다 명사를 사용한다. 

REST API는 리소스에 대해서 행동을 정의함

```
GET /teams/{id}/groups/{groupId}
```

- 단수형보다는 복수형 명사를 사용하라. 



```
// 잘못된 예제
GET /getTeams/
POST /setTeam/{id}
```



|          | POST               | GET            | PUT                      | DELETE              |
| -------- | ------------------ | -------------- | ------------------------ | ------------------- |
| /teams   | 새로운 팀을 등록함 | 팀 목록을 리턴 | 팀 정보를 업데이트(Bulk) | 모든 팀 정보를 삭제 |
| /teams/1 | x                  | 1번 팀을 리턴  | 1번 팀 정보를 업데이트   | 1번 팀을 삭제       |



### 리소스간의 관계 표현하는 방법

위에서도 예제를 통해서 들었지만 `/resource/{identifier}/other-related-resources` 형태로 정의한다. 

```
GET /teams/3/groups/4
```

3번 팀의 4번 그룹을 조회하는 API





### 검색 

일반적으로 HTTP GET은 QueryString을 사용한다. 이 경우 검색조건이 다른 Query String과 섞일 수 있다. 

결론적으로 검색조건은 URL Encoder를 사용해서 하나의 검색조건으로 만드는 것이 좋다. 

```
// 여러개의 QueryString이기 때문에 검색조건과 페이징 조건이 헷갈림 (X)
/users?name=andrew&region=seoul&offset=20&limi=10

// 검색조건은 하나의 QueryString에서 끝내고, 나머지는 페이징 조건 (O)
/users?q=name%3Dandrew,region%3Dseoul&offset=20&limit=10
```



#### 전역 검색과 리소스 검색

전역 검색은  /search?q=name%3Dandrew

리소스검색은 /users?q=name%3Dandrew



### 단일 API URL 

API 서버가 물리적으로 분리된 경우, 여러개의 서버에서 동작하고 있을 때, `account.apiserver.com`, `appapiserver.com`  과 같을 때 HAProxy나 nginx reverse proxy를 사용할 수 있다.

HAProxy 설정에서 

```
api.apiserver.com/account 는 account.apiserver.com로 라우팅 하고
api.apiserver.com/app 는 app.apiserver.com으로 라우팅하도록 구현
```
