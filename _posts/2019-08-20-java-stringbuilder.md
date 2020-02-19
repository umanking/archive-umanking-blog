---
layout: post
title: "[Java] StringBuffer, StringBuilder의 차이"
categories: [Java]
tags: [Java]
date: 2019-08-20 17:40:48
redirect_from: 
- 2019/08/20/java-stringbuilder/
- java/java-stringbuilder
---
> String, StringBuffer, StringBuilder 차이점에 대해서 알아보겠습니다. 

자바에서 String을 만드는 방법은 2가지가 있습니다.

```java
public static void main(String[] args){
  String name = "andrew";
  String email = new String("umanking@gmail.com");
}
```

이렇게 만들고, String은 `Immutable` 한 객체 입니다. 그렇기 때문에 멀티 쓰레드 환경에서 공유하는 것이 상관이 없습니다.  String에서 동일한 값인 "andrew"값을 새로운 변수에 할당하면, JVM은 String풀에서 해당 String 객체의 값("andrew")를 가지고 오고, 동일한 값이 존재 하지 않는다면 새로운 객체로 생성해서 리턴합니다. 이 말인 즉슨 

```java
public static void main(String[] args){
  String name1 = "andrew";
  String name2 = "andrew";
  String name3 = new String("andrew");

  System.out.println(name == name2); //true
  System.out.println(name == name3); //false
}
```

이런 식으로 참조값(동등성) 비교를 하게되면 같은 String Pool에서 관리하는 "andrew"값이 같기 때문에 true를 리턴합니다. 반면에 new를 통한 새로운 객체를 만드는 경우에는 당연히 주소값이 다르기 때문에 동등성 비교에서 false가 나옵니다. 

String은 concat(), substring() 메서드나 `+` 연산자를 통한 문자열을 조작할 때, 새로운 메모리 공간을 할당하고, 예전에 사용하지 않는 String은 GC에 의해서 수거를 합니다. 만약에 엄청난 수의 문자열을 조작한다고 생각해보십시요. 사용하지도 않는 힙 메모리 영역들을 차지하게 되고(나중에 GC에 의해 수거하더라도) 메모리 Leak이 발생할 수 있습니다. 그래서 자바에서는 StringBuffer, StringBuilder를 제공하고 있습니다. 

## StringBuilder, StringBuffer 는 mutable 합니다.
StringBuffer, StringBuilder는 String과 다르게 `mutable` 한 객체입니다.  append(), insert(), delete() and substring() 이런 메서드들을 통해서 메모리에서 새롭게 할당하는 것이 아닌, 가변적으로 메모리를 늘려나갑니다. 

## 그렇다면 StringBuffer와 StringBuilder의 차이는 무엇일까요 ?
StringBuffer는 모든 public 메서드에 `syncronized` 키워드가 붙어있습니다. 말인 즉슨, Thread-safe하다는 말입니다. 동시에 Thread-safe하다는 말은 자원을 선점할때 Lock를 걸기 때문에 그만큼의 성능 효율이 매우 안 좋다는 것을 의미합니다. 그래서 1.4때까지는 StringBuffer를 사용했지만, 그 이후 버전에는 StringBuilder를 사용하는 것입니다. StringBuilder는 그에 반해 당연히 성능이 좋습니다. 



## 정리
- String은 `imutable`클래스 StringBuffer, StringBuilder는 `mutable`클래스 
- StringBuffer는 thread-safe 하지만 성능이 느리다.
- StringBuilder는 thread-safe 하지 않지만, 성능은 빠르다. 
- 그래서 멀티스레드 환경이 아닌, 문자열 연산이 많은 경우에는 → StringBuilder를 사용한다. 
- 문자열 연산이 많이 없고, 단순히 값들만을 가져오는 경우에는  → String Pool에서 가져오므로 String 사용한다. 

