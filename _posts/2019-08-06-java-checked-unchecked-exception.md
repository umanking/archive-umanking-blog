---
layout: post
title: "[Java] Exception"
categoires: [java]
date: 2019-08-06 11:07:14
tags: 
- Java
redirect_from: 
- 2019/08/06/java-checked-unchecked-exception/
---
> Java에서 Exception의 종류 Checked Exception, Unchecked Exception(Runtime Exception)에 대해서 알아보고 Exception Handling은 어떻게 하는지 알아보자!



## 1. 에러(Error)랑 예외(Exception)의 차이? 

에러는 시스템에 비정상적인 상황이 생겼을때 발생한다. 시스템 레벨에서 발생하기 때문에 심각한 수준의 오류이다. 개발자가 직접 접근해서 처리할 수 없는 영역이다. 반면에 예외(Exception)는 발생할 상황을 미리 예측하여 처리할 수 있다.
Exception에는 CheckedException과 UnceckedException으로 나뉜다. 

## 2. Throwable 클래스 계층구조
![Image result for java throwable hierarchy](https://www.cis.upenn.edu/~bcpierce/courses/629/papers/Java-tutorial/java/exceptions/images/throwableHierarchy_trans.gif)

Throwable 클래스는 Error 클래스와 Exception 클래스의 최상위 클래스고, Throwable은 Object 클래스의 자손이다. Exception은 `CheckedException` 과`UncheckedException(RuntimeException)`으로 나뉜다. 


## 3. Checked Exception과 Unchecked Exception 차이


|                           | checked                                                      | Unchecked(Runtime)                                           |
| ------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 처리여부                  | 반드시 예외 처리해야함<br />(try-catch, throw exception)     | 처리를 강제 하지 않는다                                      |
| 확인시점                  | 컴파일                                                       | 런타입                                                       |
| 예외 발생시 트랜잭션 처리 | 롤백 X                                                       | 롤백O                                                        |
| Example                   | Exception의 상속받는 하위 클래스 중 RuntimeException을 제외한 모든 예외<br />- IOException<br />- SQLException | RuntimeException 하위 예외<br />- NullPointerException<br />-IllegalArgumentException<br/>-IndexOutOfBoundException<br />- SystemException |


가장 명확한 기준은 **“꼭 처리해야 하는지 여부”** CheckedException은 발생할 가능성이 있는 메서드라면 반드시 try/catch 나 throw로 던져서 처리해야한다. 반면에 UncheckedException은 명시적으로 예외를 처리하지 않아도 된다. 이 예외는 피할 수 있지만 **개발자의 부주의에 의해서 발생하는 경우가** 대부분, 그렇기 때문에 굳이 명시적으로 예외를 처리하지 않아도 된다.


## 4. 예외 처리방법 

1. 예외 복구: 다른 작업 흐름으로 유도
2. 예외 처리 회피: 처리하지 않고 호출한 쪽으로 throw

   ```java
   public void method() throws IOException {
   	// 구현로직
   }
   ```

   비교적 단순하지만, 신중해야하는 로직. 호출한 쪽으로 예외를 던지기 때문에, 호출한 쪽에서 다시 예외를 받아서 처리하도록 해야하거나, 해당 메서드가 예외를 던지는 것이 최선의 방법이라는 확신이 있을때만 사용해야 한다. 

3. 예외 전환: 명확한 의미의 예외로 전환 후 throw 

   ```java
   catch (SQLException e){
     ...
       throw DuplicatedUserIdException();
   }
   ```

   예외를 잡아서 다른 예외를 던지는 것이다. 호출한 쪽에서 예외를 받아서 처리할 때 좀더 명확하게 인지할 수 있도록 돕기 위한 방법이다. 예를 들어 checkException 중 복구가 불가능한 예외가 잡혔다면  이를 UncheckedException으로 전환해서 메서드를 호출하는 곳 마다(다른 계층)에서 일일이 예외를 선언할 필요가 없도록 할 수 있다. 