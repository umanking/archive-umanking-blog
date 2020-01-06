---
layout: post
title: "[Java]  Timer로 1초간격으로 문자열 출력하기"
category:
  - Java
date: 2019-08-01 10:42:08
tags: 
- Java
redirect_from: 
- 2019/08/01/java-timer/
---

> 자바에서 1ms 마다 문자열을 출력하는 상황에 대해서 알아보겠습니다. 



스레드를 이용하면 하나 부터 열까지 직접 다뤄야 하지만, Timer클래스를 이용하면 비교적 쉽게 작성할 수 있습니다.  먼저 어떤 요소들이 필요한지 생각 + 파악 해봅시다. 

1. 먼저 Timer 객체를 생성합니다. 
2. 해당 Timer가 어떤 작업을 할 건지 TimerTask를 정의해 줍니다. 
3. Timer의 schedule를 이용해서 반복주기를 설정할 수 있습니다. 

```java
public static void main(String[] args){
  // timer 객체를 하나 생성합니다.
  Timer timer = new Timer();
  
  TimerTask task = new TimerTask(){
    @Override
    public void run(){
      // 어떤 Task를 할지를 정의해 줍니다.
      System.out.println("Hello 세상아!");
    }
  }
  
  //해당 timer객체에 task를 할당해주고, 반복주기를 설정합니다.
  timer.shedule(task, 1000, 1000);
  
}
```



schedule() 메서드의 1첫번째 인자는 할당될 task(작업), 2번째 인자는 delay(ms), 세번째가 반복주기 period(ms) 입니다. 두번째 인자까지만 넘기게 되면 단발성 작업이고, 세번째 인자까지 넘기면 반복 주기를 통해서 우리가 원래 하려고 했던 작업이 됩니다. 



API문서를 보니 1.3이후로 등장했던 클래스였네요! 