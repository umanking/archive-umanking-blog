---
layout: post
title: "[이펙티브 자바] 9.try-finally보다는 try-with-resouce를 사용하라"
categories: [Effective Java]
date: 2020-02-25 20:10 +0900
---

자바 라이브러리에는 close메서드를 호출해 직접 닫아줘야 하는 자원이 많다,
InputStream, OutputStream, java.sql.Connection등이 좋은 예다,

전통적인 방식으로 try-finally 가 쓰였다. 
```java
static String firstLineOfFile(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try{
        return br.readLine();
    }finally{
      br.close();  
    }
}
```
나쁘지는 않지만, 자원을 하나 더 사용한다면 어떨까? 

```java
static void copy(String src, String dst) throws IOException {
    InputStream in = new FileOuuputStream(dst);
    try{
        OutputStream out = new FileOutputStream(dst);
        try{
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >=  0) 
                out.write(buf, 0, n);
        }finally {
        out.close();
    } finally {
        in.close();
    }
}
```
예외는 try블록과 finally 블록 모두에서 발생할 수 있다. 만약에 firstLineOfFile 메서드 안의 readLine 메서드가 예외를 던지고, 같은 이유로 close메서드도 실패할 것이다. 이런 상황이라면 두 번째 예외가 첫 번째 예외를 완전히 집어삼켜 버린다. 그러면 스택 추적 내역에 첫 번째 예외에 관한 정보는 남지 않게 되어, 실제 시스템에서 디버깅을 몹시 어렵게 한다. 

## try-with-resources 로 해결!
이러한 문제들은 자바7이 투척한 try-with-resources 덕에 모두 해결되었다.
이 구조를 사용하면 해당 자원이 AutoCloseable 인터페이스를 구현해야 한다.
자바 라이브러리와 서드파티 라이브러리들의 수많은 클래스와 인터페이스가 이미 AutoCloseable을 구현하거나 확장했다.
다음은 try-with-resources를 사용한 코드이다.

```java
static String firstLineOfFile(String path) throws IOException {
    try(BufferedReader br = new BufferedReader(new FileReader(path))){
        return br.readLine();
    }
}
```

```java
static void copy(String src, String dst) throws IOException {
    try (InputStream in = new FileOuuputStream(dst);
    OutputStream out = new FileOutputStream(dst)){
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while ((n = in.read(buf)) >=  0) 
            out.write(buf, 0, n);    
    }
}
```
try-with-resources 읽기 수월할 뿐 아니라 문제를 진단하기도 훨씬 좋다.
firstLineOfFile 메서드를 생각해보자. readLine과 close 호출 양쪽에서 예외가 발생하면, close에서 발생한 예외는 숨겨지고 readLine에서 발생한 예외가 기록된다. 
실전 프로그래머에게 보여줄 예외 하나만 보존 되고 여러 개의 다른 예외가 숨겨질 수도 있다.
이렇게 숨겨진 예외들도 그냥 버리지 않고, 스택 추적 내역에 '숨겨졌다(suppressed)' 는 꼬리표를 달고 출력된다. 

> 또한, 자바7에서 Throwable에 추가된 getSuppressed 메서드를 이용하면 프로그램 코드에서 가져올 수도 있다.

try-with-resources 에서도 catch절을 쓸 수 있다. 그 덕분에 try문을 더 중첩하지 않고도 다수의 예외를 처리할 수 있다.

```java
static String firstLineOfFile(String path) throws IOException {
    try(BufferedReader br = new BufferedReader(new FileReader(path))){
        return br.readLine();
    } catch (IOException e){
        // 기본 값 반환, 예제가 살짝 어색하지만 
        return defaultVal;
    }
}
```

## 핵심정리
꼭 회수해야 하는 자원이 있을때, try-with-resources 를 사용하자.
쉽고 정확하고, 쉽게 자원을 회수할 수 있다.