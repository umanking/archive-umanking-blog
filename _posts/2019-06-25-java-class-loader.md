---
layout: post
title: "[Java] 클래스로더란?" 
date: 2019-06-25 10:32:52
categories: [Java]
tags: [Java]
redirect_from: 
- 2019/06/25/java-class-loader/
---
> 자바에서 클래스 로더에 대해서 알아보자. java8 기준

자바 프로그램은 한개 혹은 그 이상의 클래스들의 조합으로 실행된다. 실행시 모든 클래스 파일이 한 번에 JVM 메모리에 로딩되지 않고 , 요청 순간 로딩된다. 자바의 클래스로더가 이런 역할을 수행함. 



![자바 프로그램의 실행 순서](http://tcpschool.com/lectures/img_java_programming.png)

> 자바가 동작하는 법
>
> - .java 코드를 작성하고, `javac`  (자바 컴파일러가) .class 파일로 변환해준다. 
> - 해당 클래스파일은 한 번에 메모리에 올라가지 않고, 요청 순간 로딩된다. 이러한 역할을 해주는게 클래스 로더









## java에 3가지 기본 클래스 로더가 존재

### Bootstrap class loader

- 3가지 기본 클래스로더 중 최상위 클래스로더로서, `jre/lib/rt.jar` 에 담긴 JDK 클래스 파일을 로딩한다. 
- [Primordial ClassLoader(원시 클래스로더)](https://docs.oracle.com/javase/8/docs/technotes/guides/security/spec/security-spec.doc5.html#a28414) 라고 불린다.  - 가장 최상위 클래스로더이기 때문에



> ※ rt.jar 파일이란? 
>
> rt는 RunTime를 의미한다. rt.jar는 코어 자바클래스(자바 런타임 환경에서)의 콜렉션이다. JVM에서 rt.jar파일에 담긴 클래스 파일들을 런타임시에 메모리에 올린다. 아무렇지 않게 사용했던 String 클래스, System 클래스 (System.out.println()) 같은 클래스가 rt.jar 안에 속해 있다.



### Extension class loader 

- 익스텐션 클래스로더는 `jre/lib/ext` 폴더나 `java.ext.dirs` 환경 변수로 지정된 폴더에 있는 클래스 파일을 로딩한다. 



```java
public class Main {
  public static void main(String args[]){
    System.out.println(System.getProperty("java.ext.dirs"));
  }
}


// console
/Users/andrew/Library/Java/Extensions:/usr/local/Cellar/adoptopenjdk-openjdk8/jdk8u172-b11/jre/lib/ext:/Library/Java/Extensions:/Network/Library/Java/Extensions:/System/Library/Java/Extensions:/usr/lib/java

```



### Application class loader 

- 애플리케이션 클래스로더는 `-classpath` 나 jar파일 안에 있는 Manifest 파일의 Class-path 속성값으로 지정된 폴더에 있는 클래스를 로딩한다. 
 자가 애플리케이션 구동을 위해 직접 작성한 대부분 클래스는 이 애플리케이션 클래스로더에 의해 로딩된다. 



## 클래스 로더 동작에는 3가지 원칙

1. 위임원칙: 클래스로딩 작업을 상위 클래스로더에 위임한다. 
2. 가시범위 원칙: 하위 클래스로더는 상위 클래스 로더가 로딩한 클래스를 볼 수 있지만, 상위 클래스 로더는 하위 클래스 로더가 로딩한 클래스를 볼 수 없다.
3. 유일성 원칙: 하위 클래스로더는 상위 클래스 로더가 로딩한 클래스를 다시 로딩하지 않게 해서 로딩된 클래스의 유일성을 보장한다. 







## 참고

- [https://homoefficio.github.io/2018/10/13/Java-%ED%81%B4%EB%9E%98%EC%8A%A4%EB%A1%9C%EB%8D%94-%ED%9B%91%EC%96%B4%EB%B3%B4%EA%B8%B0/](https://homoefficio.github.io/2018/10/13/Java-클래스로더-훑어보기/)

- [http://tcpschool.com/java/java_intro_programming](http://tcpschool.com/java/java_intro_programming)

- https://futurists.tistory.com/43
- https://www.quora.com/What-is-rt-jar-in-Java