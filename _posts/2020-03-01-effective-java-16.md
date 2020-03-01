---
layout: post
title: "[이펙티브 자바 3판] 아이템16.public클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라"
description: 자바 접근자 메서드를 활용한다.
categories: [Effective Java]
date: 2020-03-01 09:02 +0900
---

![effective java image](https://user-images.githubusercontent.com/28615416/75598228-81ca1c00-5add-11ea-9319-e949af4e07cd.png){:.postImage}

```java
class Point {
    public double x;
    public double y;
}
```
이런 클래스는 데이터 필드에 직접 접근할 수 있으니 캡슐화의 이점을 제공하지 못한다. API를 수정하지 않고는 내부 표현을 바꿀 수 없고, 불변식을 보장할 수 없으며, 외부에서 필드에 접근할 때 부수 작업을 수행할 수도 없다. 
철저한 객체 지향 프로그래머는 이런 클래스를 상당히 싫어해서 필드를 모두 private으로 바꾸고 public 접근자를(getter)를 추가한다.

```java
class Point {
    private double x;
    private double y;

    // 생성자 생략

    public double getX(){
        return x;
    }
    public double getY(){
        return y;
    }
    // setter 생략
}
```
public 클래스라면 이 방식이 맞다. 패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공함으로써 클래스 내부 표현 방식을 언제든 바꿀 수 있는 유연성을 얻을 수 있다. public 클래스가 필드를 공개하면 이를 사용하는 클라이언트가 생겨날 것이므로 내부 표현 방식을 마음대로 바꿀 수 없게 된다.

**하지만 package-private 클래스 혹은 private 중첩 클래스라면 데이터 필드를 노출한다 해도 하등의 문제가 없다.** 그 클래스가 표현하려는 추상 개념만 올바르게 표현해주면 된다. 이 방식은 클래스 선언 면에서나 이를 사용하는 클라이언트 코드 면에서나 접근자 방식보다 훨씬 깔끔하다. 클라이언트 코드가 이 클래스 내부 표현에 묶이기는 하나, 클라이언트도 어차피 이 클래스를 포함하는 패키지 안에서만 동작하는 코드일 뿐이다. 따라서 패키지 바깥 코드는 전혀 손대지 않고도 데이터 표현 방식을 바꿀 수 있다. private 중첩 클래스의 경우라면 수정 범위가 더 좁아져서 이 클래스를 포함하는 외부 클래스까지로 제한된다. 

## 핵심정리
public 클래스는 절대 가변 필드를 직접 노출해서는 안된다. 불변 필드라면 노출해도 덜 위험하지만 완전히 안심할 수 없다. 하지만 `package-private 클래스`나 `private 중첩 클래스`에서는 종종(불변이든 가변이든) 필드를 노출하는 편이 나을 때도 있다.

## 참고 
- [https://gyrfalcon.tistory.com/entry/JAVAJ-Nested-Class](https://gyrfalcon.tistory.com/entry/JAVAJ-Nested-Class)