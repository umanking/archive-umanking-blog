---
layout: post
title: "Thymeleaf 실무에서 자주 사용하는 것들"
date: 2020-01-19 15:59 +0900
categories: [spring]
tags: [spring, thymeleaf]
---

# 개요
오늘은 저번시간에 다룬 간단한 [Spring, Thymleaf로 예제](https://umanking.github.io/spring/spring-thymeleaf/)를 중심으로 실무에서 자주 사용하는 Thymeleaf를 알아보도록 하겠습니다. 
[소스코드는](https://github.com/umanking/spring-workspace/tree/master/spring-thymleaf)이곳 에서 확인 가능 합니다.

## th:text, th:value
```html
<div th:text="${message}"></div>
```
모델에 데이터를 담고, 화면에 뿌릴때, 가장 많이 사용하는 `th:text`과 `th:value` 
## th:each 
```html
<tr th:each="item, iter:${items}">
    <td th:text="${item.name}" th:value="${iter.index}">이름값이 없습니다.</td>
    <td th:text="${item.price}" th:value="${iter.index}">가격정보 없습니다.</td>
</tr>
```

items 라는 list를 받아서, item 각각의 정보를 text로 화면에 뿌립니다. 종종 index정보가 필요한 경우가 있습니다.
iter를 통해서 index정보도 얻을 수 있습니다. 

## th:if, th:unless

```html
<!--if else -->
<th:block th:if="${isPurchased}">
    <div>구매함</div>
</th:block>
<th:block th:if="!${isPurchased}">
    <div>구매하지 않음</div>
</th:block>
```
`th:if` 나 `th:unless` 를 통해서 if else를 나타낼 수 있습니다.

## th:switch
```html
<!--switch case-->
<th:block th:switch="${itemStatus}">
    <div th:case="1" th:text="success"></div>
    <div th:case="2" th:text="fail"></div>
    <div th:case="3" th:text="pending"></div>
</th:block>
```
`th:switch` 로 switch case 구문도 나타낼 수 있습니다. 

## th:object 객체의 attribute에 접근하기
```html
<div th:object="${userInfo.user}">
    <div th:text="*{firstName}"></div>
    <div th:text="*{lastName}"></div>
    <div th:text="*{city}"></div>
</div>
```
userInfo 객체 안에 user 객체의 attribute에 접근할 때는 다음과 같이 사용할 수 있습니다. 
```html
    <div th:text="${userInfo.user.firstName}"></div>
    <div th:text="${userInfo.user.lastName}"></div>
    <div th:text="${userInfo.user.city}"></div>
```
위에랑 정확히 동일한 표현 방법입니다.


## 계산식
### 1. numbers.sequence
```html
<ul th:each="num: ${#numbers.sequence(1,5)}">
    <li th:text="${num}"></li>
</ul>
```
sequential 하게 숫자를 만들고 싶으면, 다음과 같이 #numbers라는 이미 정의해놓은 함수들을 사용할 수 있습니다. 
### 2. Collection
```html
<span>총 아이템: <div th:text="${#lists.size(items)}"></div>
```
아이템의 총 size도 측정할 수 있는 utils를 제공해줍니다. 

### 3. 합계(통계)
```html
<td th:text="${#aggregates.sum(o.orderLines.{purchasePrice * amount})}">23.32</td>
```


### 4. 날짜 정보
```html
<div th:text="${#calendars.format(#calendars.createNow(), 'yyyy-MM-dd HH:mm')}"></div>
```
`#calendars`를 통해서 날짜도 화면에 렌더링할 수 있습니다. 

> Expression Utility Objects에 관한 더 많은 정보는 [여기](https://www.docs4dev.com/docs/en/thymeleaf/3.0/reference/using_thymeleaf.html#expression-utility-objects)에서 확인 바랍니다.


## 정리 
오늘은 view template engine인 Thymeleaf 기본적으로 실무에서 많이? 자주 사용하는 것들에 대해서 알아보았습니다. 
