---
layout: post
title: Java 동시성 프로그래밍
date: 2019-12-30 23:06 +0900
categories: [Java]
tags: [Java]
---

java는 처음 만들어질 때부터 cocurrent(동시성) 프로그래밍을 지원했다. java 5버전 이후로 부터 high level concurrent api를 제공한다.
java에서 동시성 프로그래밍을 말할 때 주로 고려하는 대상은 `thread`이다 하지만 그에 못지 않게 process도 중요하다. 

# 프로세스란?
각 process는 자신의 메모리 공간을 가진다. 프로세스간의 통신하기 위해서는 IPC resources를 지원한다. pipe, socket IPC와 같은
JVM은 single process로 동작한다. java application은 `ProcessBuilder` 오브젝트를 통해서 추가로 프로세스를 구성할 수 도 있지만, `멀티 프로세싱`은 동시성에 대한 문제를 들어다 보기에 충분히 범위를 넘어선다. 

# 쓰레드란?
Thread는 lightweight processes라는 말이 있을 정도로, 프로세스나 쓰레드 둘다 실행 환경을 제공한다. 하지만 쓰레드는 프로세스에 비해서 덜 적은 자원을 요구한다. 이 말인 즉슨, 쓰레드는 프로세스의 하위개념이고, 프로세스는 반드시 하나의 쓰레드를 가진다. 쓰레드는 프로세스의 자원을 공유하기 때문에 비교적 적은 자원을 요구한다.

동시성 프로그래밍을 할려면 Thread 오브젝트를 생성해야 한다. 
두 가지 방법이 존재한다.
- 직접적으로 컨트롤하고 관리하는 방법
- executor를 통해서 (추상화)를 통해서 쓰레드를 관리하는 방법


쓰레드를 애플리케이션 단에서 실행하는 2가지 방법이 존재한다. 
첫번째는 Runnable 인터페이스를 구현 run()메서드 한개만 존재함, 두번째는 Thread 클래스를 상속받아서 run()메서드를 구현하는 것
당연히 전자가 훨씬 유연하고, high-level 쓰레드 매니지먼트 API에도 적용할 수 있다. Runnable을 구현하자!!!!

```java
public class HelloRunnable implements Runnable {

    @Override
    public void run() {
        System.out.println("hello from a thread");
    }

    public static void main(String[] args) {
        new Thread(new HelloRunnable()).start();
    }
}
```

```java
public class HelloThread extends Thread {

    @Override
    public void run() {
        System.out.println("hello from thread");
    }

    public static void main(String[] args) {
        new HelloThread().start();
    }
}
```

# 쓰레드 멈춤 
Thread.sleep은 현재 쓰레드가 지정된 시간동안 실행을 일시 중단한다. sleep 메서드는 InterupptedException을 발생한다. 현재 쓰레드에 다른 쓰레드가 끼어들어와서 interrupt를 하는 순간 해당 exception을 발생한다. 
```java
public static void main(String args[]) throws InterruptedException {
        String message[] = {
                "Hello world",
                "My Name is Andrew",
                "My age is secret",
                "Bye world"
        };

        for (int i = 0; i < message.length; i++) {
            //Pause for 4 seconds
            Thread.sleep(4000);
            //Print a message
            System.out.println(message[i]);
        }
    }
```