---
layout: post
title: Java 11 New Features
category:
date: 2019-07-25 14:57:43
tags: 
- Java
redirect_from: 
- 2019/07/25/java-jdk11-feature/
---
# 1. String 메서드 

### strip() - 벗기다 

문자열의 공백을 앞/뒤/ 앞뒤 제거를 제공합니다. 

```java
    public static void main(String[] args) {
        String andrew = " ANDREW ";
        System.out.println(andrew.strip()); // "ANDREW"
        System.out.println(andrew.stripLeading()); // "ANDREW "
        System.out.println(andrew.stripTrailing()); // " ANDREW"
				System.out.println(andrew.stripTrailing().stripLeading()); // "ANDREW"
    }
```



### isBlank()

String이 비어있거나, 공백만을 포함하는 경우 true를 반환, 그렇지 않으면 false를 반환하빈다.

```java
String emptyWhiteSpace = "";
System.out.println(andrew.isBlank());
System.out.println(andrew.isEmpty());
```



#### ※ isEmpty vs isBlank 어떤 차이가 있나요? 

얼핏보면 기존에 String 클래스의 isEmpty랑 별 차이가 없어 보이지만, 한 칸 띄어쓴 문자열을 담는 변수를 생각해봅시다.

```java
String emptyStr = " ";
System.out.println(emptyStr.isBlank()); //true
System.out.println(emptyStr.isEmpty()); // false
```

JDK11에서 isBlank() 메서드는 **<u>말 그대로 비어있는, 공백을 포함 하는 경우</u>** (띄어쓰기가 존재 하더라도 비어 있기 때문에) 값이 true가 반환되고, **<u>isEmpty() 메서드는 띄어쓰기 자체를 빈 문자열로 인식하기때문에</u>** isEmpty()가 false를 반환한다. 



### lines()

이 메서드는 문자열 스트림을 반환합니다. 

```java
var lines = "hello world\nI'm developer";
System.out.println(lines); 
//hello world
//I'm developer

System.out.println(lines.lines().collect(Collectors.toList()));
// [hello world, I'm developer]
```



### repeat(n)

n번 문자열의 concat(연결하다)

```java
String hello = "hello";
System.out.println(hello.repeat(3)); //hellohellohello
```



# 2.Files 유틸리티 메서드

### writeString 

Files의 static 메서드로 정의되어있습니다. 컨텐츠를 file로 쓰는 메서드입니다. 문자는 지정된 charset을 사용하여 byte로 인코딩되며 기본값은 UTF-8 charset입니다. 

### ReadString

```java
Files.writeString(Path.of("test.txt"), "hello");
System.out.println(Files.readString(Path.of("test.txt")));
```



### isSameFile

```java
System.out.println(Files.isSameFile(Path.of("test.txt"), Path.of("1.txt"))); //false
System.out.println(Files.isSameFile(Path.of("test.txt"), Path.of("test.txt"))); //true
```

첫번째, file명만 다르고 안에 내용은 같은데 값이 false가 나왔습니다. 이름을 통해서만 같은 파일인지 비교 하는 듯 보입니다. 



# 3. Optional.isEmpty()

Optional타입으로 반환하는데, 값이 존재하지 않으면 true, 그렇지 않으면 false를 반환합니다.

```java
Optional<String> name = Optional.empty();
System.out.println(name.isEmpty()); //JDK11 //true
System.out.println(name.isPresent()); //JDK8 //false
```

사실, isPresent 메서드 하나로 처리할 수 있지만, true, false 한 값에 따라서, 메서드 명이 직관적이지 않아서 추가한게 아닐까 생각해봅니다.. 





# 4. TimeUnit covert(Duration duration)

기간(duration)을 유닛으로 변환해주는 메서드입니다.  Duration.toNanos() 와는 다릅니다. toNanos()메서드는 숫자가 overflow될시, ArithmeticException을 발생시킵니다 . 

```java
TimeUnit timeUnit = TimeUnit.DAYS;
System.out.println(timeUnit.convert(Duration.ofHours(24)));
System.out.println(timeUnit.convert(Duration.ofHours(36)));// 하루가 지나도, 반환은 하루로 반환

System.out.println(timeUnit.convert(Duration.ofMinutes(60 * 24)));
System.out.println(timeUnit.convert(Duration.ofMinutes(60 * 24 + 10)));
```

