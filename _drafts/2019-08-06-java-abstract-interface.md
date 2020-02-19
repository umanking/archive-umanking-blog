---
layout: post
title: Java 인터페이스와 추상클래스
categories: [Java]
date: 2019-08-06 08:36:29
tags: [Java]
redirect_from: 
- 2019/08/06/java-abstract-interface/
---
> Java에서 abstract vs Interface의 차이 



추상: 내부 구현은 감추고, 무엇인지만을 보여준다.



1. 메서드 타입: 인터페이스는 오직 추상메서드만을 가진다 . **추상클래스는 추상 메서드와 추상 메서드가 아닌 것을 가진다. 자바8부터 default와 static 메서드도 가능**하다.
2. Final Variables: 자바 인터페이스에서는 변수 선언 기본이 final이다. 추상 클래스에서는 non-final 변수들도 포함한다. 
3. 변수 타입: 추상 클래스는 final, non-final, static, non-static 변수를 가지고, 인터페이스는 오직 static과 final 변수를 가진다. 
4. 구현: 추상클래스는 인터페이스를 구현 할 수 있다. 그 반대는 안됨 
5. 상속 vs 추상: 자바 인터페이스는 `implements`라는 키워드로 구현을 하고 추상 클래스는 `extends`라는 키워드로 상속을 한다. 
6. 다중 구현: 상속은 한번, 인터페이스 구현은 다중 가능
7. Accessibility of Data Members: 자바 인터페이스 맴버들은 기본이 public이다. 추상클래스는 member들이 private, protected, etc 가질 수 있다. 



> 정리하자면
>
> 자바에서 상속은 한번만, 인터페이스로 `다중 구현` 가능 하고, 인터페이스는 빈껍데기인 추상메서드로만 구성되어있는 반면에, 추상클래스는 클래스의 성격도 지니고 있기 때문에 접근제어자, 변수 타입이 다채롭다. 





