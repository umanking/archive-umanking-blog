---
layout: post
title: "[Java] Reflection 예제"
categories: [Java]
tags: [java]
date: 2019-08-24 23:36:46
redirect_from: 
- 2019/08/24/java-reflection/
- java/java-reflection
---
> 자바에서 리플랙션은 하나의 기능으로, 내부 properties를 변경하거나 정보를 직접 얻어올 수 있는 manual적인 프로그래밍을 말합니다. 대표적으로 애플리케이션의 모든 클래스 정보, 혹은 메서드 정보를 직접 가져올 수 있습니다.


## 1. 클래스 정보 가져오는 3가지 방법

```java
public static void main(String[] args){
  
  // 인스턴스에서 가져오기
  Member member = new Member();
  Class<? extends Member> m1Class = member.getClass();
  
  // 타입으로 접근하는 방법
  Class<Member> m2Class = Member.class;
  
  // full package name 
  Class<?> m3Class = Class.forName("com.example.demo.member");
}
```

자바의 Class라는 API를 통해서 클래스 정보를 가지고 올 수 있다. 클래스 정보를 통해서 `필드`, `메서드`, `구현한 클래스`, `인터페이스` 를 가지고 올 수 있다. 특히나, 메서드, 필드에 접근할 때는 접근제한자가 private인 경우에는 `getDeclaredxxx()` 시리즈의 메서드가 전부다 가지고 올 수 있다. 또한, 접근이 불가능한 필드에 대해서는 각각의 필드에 대해서 `setAccesible(true)` 값을 주면 접근이 가능해 진다. 



## 2.자바 @애노테이션 사용하기

```java
public @interface CustomAnnotation {
  
}

// 사용할때
@CustomAnnotation 
public class Member {
  
  ..//생략
}
```

`@interface` 타입으로 클래스를 만들어서 사용할 수 있다. 하지만 `CustomAnnotation` 은 아무 기능도 하지 않는다. 그저 주석의 역할일 뿐, java파일이 compile이 되고 class파일까지 갔을때, 해당 어노테이션이 남아있긴 하지만, 바이트코드가 JVM에서 로딩이 되어 동작할때 까지 유지되지 않는다. 애노테이션의 기본이 `@Retention(RetentionPolicy.CLASS)` 이기 때문에 클래스파일 까지만 유지(Retaion)된다. `@Retention(RetentionPolicy.RUNTIME)` 으로 바꾸면 그제서야, 리플렉션을 통해서 접근할 수 있다. 

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface MyAnnotation{
 	 
}

@MyAnnotaion
public class Member{
  
}
```



```java
public static void main(String[] args){
  Class<Member> memberClass = Member.class;
  Arrays.stream(memberClass.getAnnotations()).forEach(System.out::println);
}
```

이렇게 클래스에서 어노테이션 정보를 가지고 와서 사용할 수 있다. 


## 3.객체 직접 만들기
class에서 `getConstructor()` 메서드를 통해서 생성자를 가지고와서, `constructor.newInstance()` 를 통해서 만든다. 


## 추가 학습
- 기본 생성자를 가지고 와서 인스턴스 만들기
- 파미터가 있는 생성자를 가지고 와서 인스턴스 만들기(값을 넘겨줘야 함)
- 만든 인스턴스의 필드에 get(), set()을 통해서 값을 변경하기
- 만든 인스턴스의 메서드 호출하기



