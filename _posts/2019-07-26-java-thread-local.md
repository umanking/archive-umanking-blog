---
layout: post
title: "[Java] ThreadLocal"
categories: [Java]
date: 2019-07-26 11:58:39
tags: 
- Java
redirect_from: 
- 2019/07/26/java-thread-local/
- java/java-thread-local/
---

일반 변수는 특정 코드 블록내의 범위에서만 유효합니다. 반면에 ThreadLocal은 쓰레드 영역에 변수를 설정할 수 있기 때문에 특정 쓰레드가 실행하는 모든 코드에서 그 쓰레드에 설정된 변수 값을 사용할 수 있습니다. 



## 언제쓰일까요 ?

ThreadLocal은 한 쓰레드에서 실행되는 코드가 동일한 객체를 사용할 수 있도록 해주기 때문에 쓰레드와 관련된 코드에서 파라미터를 사용하지 않고 객체를 전파하기 위한 용도로 주로 사용합니다. 

- 사용자의 인증정보
- 트랜잭션 컨텍스트 전파 
- 쓰레드에 안전해야 하는 데이터 보관





## 사용방법

1. ThreadLocal 객체를 생성
2. set()메서드를 이용해 **<u>현재 쓰레드의</u>** 로컬 변수에 값을 저장
3. get()메서드를 이용해 **<u>현재쓰레드의</u>** 로컬 변수 값을 읽어옴
4. remove()를 이용해 **<u>현재 쓰레드의</u>** 로컬 변수 값을 삭제한다.



※주의사항 - 쓰레드 풀 환경에서 ThreadLocal을 사용하는 경우, ThreadLocal에 보관된 데이터의 사용이 끝나면 반드시 해당 데이터를 삭제해야합니다. 그렇지 않으면 쓰레드가 올바르지 않은 데이터를 참조 할 수 있습니다. 







