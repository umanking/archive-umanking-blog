---
layout: post
title: "[Java]  TimeStamp, LocalDateTime, Instant 비교"
category:
   
  - Java
date: 2019-08-01 16:51:29
tags: 
- Java
redirect_from: 
- 2019/08/01/java-timestamp/
---

> 자바에서 사용하는 TimeStamp, Unix TimeStamp, LocalDateTime, Instant 는 무엇인지 알아 보겠습니다. 



## 유닉스 타임스탬프(Unix TimeStamp)

- 유닉스 타임 스탬프는 1970년 1월 1일 0시 0분 0초 부터 몇초가 지났는지를 나타내는 정수이다.
- 배경? 
  - 유닉스는 19070년대 초에 개발되었기 때문에, 최대한 많은 숫자를 타임스탬프로 구성하기 위해서 기준일자(epoch - 시대, 시기)를 1970년 1월 1일로 정한 것
- 윤초(leap second) 까지 반영하지 못함





## 유닉스 타임스탬프 -> LocalDateTime 변환

아래 예제는 [Import 토큰을 획득 API](https://docs.iamport.kr/tech/access-token)를 요청하고, 응답받은 JSON입니다. 만료시간이 유닉스 타임 스탬프이기 때문에 LocalDateTime으로 변환시켜야 합니다.

```json
  {
    "code": 0,
    "message": null,
    "response":{
      "access_token": "a9ace025c90c0da2161075da6ddd3492a2fca776",
      "now": 1512446940,
      "expired_at": 1512448740,
    },
  }
```



```java
long expiredAt = 1512448740;
LocalDateTime result = LocalDateTime.ofInstant(Instant.ofEpochSecond(expiredAt), TimeZone.getDefualt().toZoneId());

System.out.println(result); //2017-12-05T13:39
```





## LocalDateTime vs Instant 비교

**LocalDateTime**

- 타임존이 없는 date-time  ex) `2007-12-03T10:15:30`



**Instant**

- Instant 객체는 UTC기준의 ISO 포맷으로 출력됩니다. 
- Instant는 타임스탬프를 표현하는 클래스 답게 **<u>초나 밀리 초 단위로</u>** 시간을 더하거나 빼는 메소들을 제공합니다. 

```java
LocalDateTime now = LocalDateTime.now();
Instant instantNow = Instant.now();

System.out.println(now);  //2019-08-01T16:41:28.027
System.out.println(instantNow); //2019-08-01T07:41:28.027Z

```

2번째 Instant 객체는 뒤에 Z가 붙어있는 것을 확인할 수 있습니다. 
