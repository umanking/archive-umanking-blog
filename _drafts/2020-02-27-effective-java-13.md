---
layout: post
title: "[이펙티브 자바] 13.clone 재정의는 주의해서 진행하라."
categories: [Effective Java]
date: 2020-02-27 21:24 +0900
---
Cloneable은 복제해도 되는 클래스임을 명시하는 용도의 믹스인 인터페이스이지만, 아쉽게도 의도한 목적을 제대로 이루지 못했다. 가장 큰 문제는 clone 메서드가 선언된 곳이 Object이고 그마저도 protected라는데 있다. 그래서 Cloneable을 구현하는 것만으로는 외부 객체에서 ㅊlone 메서드를 호출할 수 없다. 

이번 아이템에서는 clone메서드를 잘 동작하게끔 해주는 구현 방법과 언제 그렇게 해야 하는지
그리고 가능한 다른 선택지에 관해 논의 하자.

메서드가 하나 없는 Cloneable 인터페이스는 대체 무슨일을 할까? 이 인터페이스는 놀랍게도 Object의 protected메서드인 clone의 동작 방식을 결정한다. Cloneable을 구현한 클래스의 인스턴스에서 clone을 호출하면 그 객체의 필드들을 하나하나 복사한 객체를 반환하며, 그렇지 않은 클래스의 인스턴스에서 호출하면 CloneNotSupportedException을 던진다. 

이는 인터페이스를 상당히 이례적으로 사용한 예이니 따라 하지말자.
인터페이스를 구현한다는 것은 일반적으로 해당 클래스가 그 인터페이스에서 정의한 기능을 제공한다고 선언하는 행위다. 그런데 Cloneable의 경우에는 상위 클래스에 정의된 protected 메서드의 동작 방식을 변경한 것이다.
