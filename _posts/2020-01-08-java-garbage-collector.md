---
layout: post
title: "Java Garbage Collection - #1 JVM HotSpot"
date: 2020-01-08 21:33 +0900
categories: [Java]
---

## 목표
오늘은 Java의 `Garbage Collection`에 대해서 알아보도록 하겠습니다.
이 글은 제가 이해하기 위함을 목적으로 작성된 글로써, 시리즈 형태로 작성해서 올리겠습니다. 한 번에 많은 양을 커버하기가 쉽지 않아서 잘게 나눠서 올립니다. 틀린 내용 댓글로 환영합니다.🐶

- JVM의 HotSpot의 아키텍쳐
- 무엇을 퍼포먼스라고 부르는지
- 실제 퍼포먼스 튜닝을 하기 위해서 어떤 부분을 수정해야 하는지


[원문 링크](https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html) 영어가 편하신 분들은 직접 읽는게 훨씬 좋을 것 같습니다.


# Hotspot 아키텍쳐

Hotspot JVM 프로세스는 하나의 아키텍쳐 입니다. 높은 퍼포먼스와 대량의 확장성을 지원하는 특징을 가지고 있습니다.
예를 들면 HotSpot JVM JIT 컴파일러는 동적 최적화를 만듭니다. 다른 말로 하면 자바 프로그램이 런타임에 최적화 결정을 하고, 고성능 네이티브 컴퓨터 명령어를 생성합니다. 높은 확장성을 갖고 있습니다.


![](https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/images/gcslides/Slide1.png)
JVM의 메인 컴포넌트들로 구성되어 있고, JVM은 클래스 로더, runtime data areas, 실행 엔진을 포함하고 있습니다.

## Key Hotspot Components
JVM의 핵심 컴포넌트들은 다음 이미지에서 하이라이트로 부분을 표시합니다.

![](https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/images/gcslides/Slide2.png)

퍼포먼스 튜닝할 때, 우리가 집중해야 하는 것이 저 3개의 컴포넌트 입니다. heap 영역은 객체들이 저장되는 곳입니다. heap 영역은 gc(처음 시작할때 선택된 garbage collector)에 의해서 관리됩니다. **대부분의 튜닝 옵션은 1)heap사이즈와 2)적절한 gc를 선택하는 것이 중요 합니다.** JIT 컴파일러 또한 퍼포먼스에 매우 대단히 중요한 영향을 끼치지만, 거의 드물게 새로운 버전의 JVM에서는 요구되지 않습니다.(고려 대상 아님)

# Performance Basics 
그렇다면 퍼포먼스라는 것은 무엇을 의미할까요? responsiveness(응답성)과 throughput(처리량)을 이야기 합니다. 보통 자바 application에서는 2가지 중 하나에 집중을 합니다.


## 반응성
반응성은 얼마나 빨리 애플리케이션이 응답을 하는지, 요청에 대한 시스템 응답은 어떤지? 이런 것들을 봅니다. 예를 들면 다음과 같습니다.

- 이벤트에 대해서 얼마나 빨리 desktop UI가 응답하는지
- 얼마나 빨리 웹사이트가 페이지를 리턴하는지
- 얼마나 빨리 database 쿼리가 리턴하는지

긴 시간을 두고 반응성을 살피는게 아니라, __얼마나 짧은 시간안에 반응이 오는지를 살펴보는게 핵심입니다.__

## 처리량
처리량은 특정 기간 동안에 얼만큼 최대치의 일을 할 수 있느냐를 판단합니다. 예를 들면 다음과 같습니다.

- 주어진 시간안에 얼만큼의 트랜잭션이 완료가 되는지
- 한 시간안에 얼만큼의 배치 프로그램의 job이 완료 되는지
- 한 시간안에 얼만큼의 database 쿼리가 완료 되는지 

반응성이 긴 시간이 아닌, 짧은 시간에 집중했던 것에 반해, **처리량은 긴 시간에 집중 합니다.** 


## 맺음말
다음에는 GC가 메모리 상에서 참조가 없는 객체들을 어떻게 회수해 가는지 세부적인 부분을 그림을 통해서 하나씩 살펴보도록 하겠습니다.
