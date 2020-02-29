---
layout: post
title: "[이펙티브 자바 3판] 아이템5.자원을 직접 명시하지 말고 의존 객체 주입을 사용하라"
description: 이펙티브 자바, 아이템5.자원을 직접 명시하지 말고 의존 객체 주입을 사용하라
categories: [Effective Java]
date: 2020-02-21 20:52 +0900
---
<!-- TOC -->

- [정적 유틸리티를 잘못 사용한 예 - 유연하지 않고 테스트하기 어렵다.](#정적-유틸리티를-잘못-사용한-예---유연하지-않고-테스트하기-어렵다)
- [싱글턴을 잘못 사용한 예 - 유연하지 않고 테스트하기 어렵다.](#싱글턴을-잘못-사용한-예---유연하지-않고-테스트하기-어렵다)
- [의존 객체 주입 방식](#의존-객체-주입-방식)
- [생성자에 자원 팩터리를 넘겨주는 방식 (변형)](#생성자에-자원-팩터리를-넘겨주는-방식-변형)
- [핵심정리](#핵심정리)

<!-- /TOC -->
많은 클래스가 하나 이상의 자원에 의존한다. 가령 SpellChecker는 사전(dictornary)에 의존한다. 

## 정적 유틸리티를 잘못 사용한 예 - 유연하지 않고 테스트하기 어렵다.
```java
public class SpellChecker {
    private static final Lexicon dictionary = ...;

    private SpellChecker() {
    }

    public static boolean isValid(String word){...};
    public static List<String> suggestioons(String type){...}
    
}
```

## 싱글턴을 잘못 사용한 예 - 유연하지 않고 테스트하기 어렵다.
```java
public class SpellChecker {
    private static final Lexicon dictionary = ...;

    private SpellChecker() {
    }
    public static SpellChecker INSTANCE = new SpellChecker(...);
    public static boolean isValid(String word){...};
    public static List<String> suggestioons(String type){...}
    
}
```

## 의존 객체 주입 방식
**사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않다.**
대신 클래스(SpellChecker)가 여러 자원 인스턴스를 지원해야 하며, 클라이언트가 원하는 자원(dictionary)을 사용해야 한다. 이 조건을 만족하는 간단한 패턴이 바로 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식.(의존 객체 주입 방식)

```java
public class SpellChecker {
    private static final Lexicon dictionary;

    private SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requiredNonNull(dictionary);
    }

    public static SpellChecker INSTANCE = new SpellChecker(...);
    public static boolean isValid(String word){...};
    public static List<String> suggestioons(String type){...}
    
}
```
예에서는 dictionary 딱 하나의 자원만 사용하지만, 자원이 몇개든 의존 관계가 어떻든 상관없이 잘 작동한다. 또한 불변을 보장하여 (같은 자원을 사용하려는) 여러 클라이언트가 의존 객체들을 안심하고 공유할 수 있기도 하다. 의존 객체 주입은 `생성자`, `정적 팩터리`, `빌더` 모두에 똑같이 응용할 수 있다.

## 생성자에 자원 팩터리를 넘겨주는 방식 (변형)
이 패턴의 쓸만한 변형으로, 생성자에 자원 팩터리를 넘겨주는 방식이 있다. 
> 팩터리란 호출할 때마다 특정 인스턴스를 반복해서 만들어 주는 객체를 말한다. 즉, 팩터리 메서드 패턴을 구현한 것이다.

자바 8에서 소개한 Supplier<T> 인터페이스가 팩터리를 표현한 완벽한 예이다. Supplier<T>를 입력으로 받는 메서드는 일반적으로 한정적 와일드 카드 타입을 사용해 팩터리의 타입 매개변수를 제한해야 한다. 이 방식을 사용해 클라이언트는 자신이 명시한 타입의 하위 타입이라면 무엇이든 생성할 수 있는 팩터리를 넘길 수 있다. 예컨대 다음 코드는 클라이언트가 제공한 팩터리가 생성한 타일(Tile)들로 구성된 모자이크(Mosaic)를 만든느 메서드다.
```java
Mosaic create(Supplier<? extends Tile> tileFactory){...}
```

의존 객체 주입이 유연성과 테스트 용이성을 개선해주지만, 의존성이 수천개나 되는 큰 프로젝트에서는 코드를 어지럽게 만든다. 대거, 주스, 스프링 같은 의존 객체 주입 프레임워크를 사용하면 이를 해소할 수 있다. 

## 핵심정리
클래스가 내부적으로 하나 이상의 자원에 의존하고, 그 자원이 클래스 동작에 영향을 준다면 싱글턴과 정적 유틸리티 클래스는 사용하지 않는 것이 좋다. 이 자원들은 클래스가 직접 만들게 해서도 안된다. 대신 필요한 자원을 (혹은 그 자원을 만들어 주는 팩터리를) 생성자에 (혹은 정적 팩터리나 빌더에) 넘겨주자. 의존 객체 주입이라 하는 이 기법은 클래스의 유연성, 재사용성, 테스트 용이성을 기막히게 개선해 준다.