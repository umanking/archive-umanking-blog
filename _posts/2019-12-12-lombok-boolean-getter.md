---
layout: post
title: "[Lombok] boolean 필드"
date: 2019-12-12 22:59 +0900
categories: [Library]
redirect_from: 
- 2019/12/12/lombok-boolean-getter/
---

## 1. Overview
Lombok를 사용해서 boolean 필드를 정의할때, `is`라는 prefix를 붙여야 하는지? 알아보겠습니다.

## 2. Sample example
```java
@Getter
public class Account {
    private String name;

    private boolean signUp = true;
    private boolean isSignUp = false;
}
```

```java
public class MainTest {
    public static void main(String[] args) {
        Account account = new Account();

        System.out.println(account.isSignUp()); // 뭔가 나올까요? 
    }
}
```
isSignUp 이나 signUp 이나 결국 => 롬복이 `isSignUp()` Getter를 생성해준다.

그런데 만약에 위에 처럼 2개를 하나의 클래스에서 선언하게 되면? 어떻게 될까?

롬복은 포인팅을 signUp를 향한다. (true)값이 나옴 

## 3. Boolean 타입인 경우
`boolean` 타입이 아닌 `Boolean` 타입인 경우는 원시 타입이 아니라, Object Wrapper 라는 사실!

그래서 롬복은 is가 아닌 get을 만들어 준다.
```java
@Getter
public class Account {
    private String name;
    
    private Boolean running;

}
```

```java
public class MainTest {
    public static void main(String[] args) {
        Account account = new Account();
        System.out.println(account.getRunning()); 
    }
}
```