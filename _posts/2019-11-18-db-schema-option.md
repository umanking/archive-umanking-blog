---
layout: post
title: "MySQL 데이터가 ???? 이렇게 들어가는 상황"
date: 2019-11-18 12:03 +0900
---

### 문제상황
DB에 한글로 데이터 INSERT 했는데,  ??? 로 들어가는 상황

### 해결방법
application.properties의 `db url` 설정부분에 뒤에
`useUnicode=true&characterEncoding=utf8`해당 부분을 추가해주면 됩니다.

### 결과
`localhost:3306/{db명}?useUnicode=true&characterEncoding=utf8`
