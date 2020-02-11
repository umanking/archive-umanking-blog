---
layout: post
title: "[Java] System.out.println을 사용하면 안되는 이유"
date: 2019-12-30 20:43 +0900
categories: [Java]
tags: [java]
---
# 개요
흔히 자바에서 간단하게 로그를 찍을 때, System.out.println()을 사용한다. 하지만 실제 운영상의 소스 코드로 배포를 할 때는 Sysout보다는 로깅 프레임워크를 사용한다. 아주 당연하게.
오늘은 System.out.println을 자바 소스 코드 상에서 사용하면 안되는 이유에 대해서 알아보자.

# Sysout vs 로깅 프레임워크의 차이에 대해서 알아보자. 
먼저 System.out.println()은 IO operation이기 때문에 여러분의 애플리케이션이 print메서드를 통해서 결과물을 출력할 때 까지 기다리게 된다.

```java
    /**
     * Prints a String and then terminate the line.  This method behaves as
     * though it invokes <code>{@link #print(String)}</code> and then
     * <code>{@link #println()}</code>.
     *
     * @param x  The <code>String</code> to be printed.
     */
    public void println(String x) {
        synchronized (this) {
            print(x);
            newLine();
        }
    }
```
실제로 코드를 까보면, 문자를 출력하는 `println`메서드에 `synchronized`키워드가 있는 것을 볼 수 있다.
`synchronized` 키워드는 하나의 객체에서 하나의 스레드만 (동일시간에) 실행됨을 의미한다. 그러니까 하나의 스레드만 그 해당 시간을 점유해버린다.

# 로깅 프레임 워크의 장점?
- 로깅 프레임워크는 다른 출력 형태가 없다면 메세지큐를 사용한다. 
- 로깅 프레임워크는 다른 운영상의(모니터링상의) 목적을 가지고 log파일을 분리하고, 주기적으로 파일을 관리할 수 있다. 
- 로깅 레벨을 적절하게 설정해줌으로써 원하는 레벨에 맞는 로그만을 볼 수 있다. 
