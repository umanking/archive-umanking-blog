---
layout: post
title: "Garbage Collection - #2부 GC에 대한 설명"
date: 2020-01-09 20:22 +0900
categories: [Java]
tags: [java]
---

## 목표
이전 시간에 [Garbage Collection - #1부 JVM HotSpot](https://umanking.github.io/java/java-garbage-collector/)에 대해서 알아보았습니다. 오늘은 Garbage Collection이 heap 영역에서 어떻게 이루어 지는 지 알아 보도록 하겠습니다. 

## Automatic Garbage Collection? 
Automatic(자동으로 동작하는) gc는 heap메모리를 바라보고, 어떤 객체가 사용되는지, 사용되지 않는지 판단하고, 사용하지 않는 객체는 삭제합니다. 객체를 사용한다는 의미는 객체에 대한 Reference참조를 가지고 있다는 뜻입니다. 반대로 사용하지 않는 다는 의미는 여러분의 프로그램에서 해당 객체를 더이상 참조하지 않는다라는 뜻입니다. 그래서 사용하지 않는 객체들을 통해서 메모리를 회수 할 수 있습니다.

## Step 1: Marking
이 단계는 `마킹`이라고 부릅니다. gc(garbege collector를 줄여서)가 어떤 메모리를 사용하는지 하지 않는지 판단합니다.
![](https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/images/gcslides/Slide3.png)

오렌지색깔이 참조되지 않은 객체들, 파란색은 참조된 객체들 입니다. 이러한 결정을 하기 위해서, 마킹 단계에서 모든 객체를 전체 스캔합니다. 모든 객체가 시스템상에서 전부 스캔해야 하기 때문에 이건 매우 시간 소모가 큰 프로세스 입니다.

## Step 2: Noramal Deletion
일반적인 삭제는 참조되지 않은 객체들을 삭제 하고, 남은 빈 공간과 참조한 객체들에 대한 포인터를 남깁니다. 
이 포인터로 무엇을 할까요? 

![](https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/images/gcslides/Slide1b.png)

메모리 할당자(allocator)라는 놈은 새로운 객체가 할당 될 수 있는 여유공간 블록에 대한 참조 값을 가지고 있습니다.

## Step 2a: Deletion with Compacting 
추가적으로 성능을 향상시키기 위해서, 남은 참조된 객체들도 압축할 수 있습니다. 참조된 객체를 함께 이동하면 새 메모리 할당이 훨씬 쉽고 빨라집니다.

![](https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/images/gcslides/Slide4.png)


## 왜 ? Generational Garbage Collection ?
위에 언급했듯이, JVM상의 모든 객체에 대해서 mark하고 compact 하는 과정은 비효율적입니다. 점점 더 많은 객체가 메모리에 할당되고, 객체 목록이 커지면서 GC가 일어나는 시간이 점점 더 길어집니다. 그에 반해 응용 프로그램은 경험적 분석에 의하면 대부분의 객체는 수명이 짧습니다.

한 가지 예를 들면, Y축은 할당된 바이트의 수이고, X축은 시간동안 할당된 바이트의 수 입니다.

![](https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/images/ObjectLifetime.gif)

보시는 것처럼, 시간이 지남에 따라 할당된 객체들이 남아있는게 점점 적어집니다. 사실, 대부분의 객체는 매우 짧은 수명주기를 가지고 있습니다. 특히나 그래프의 왼쪽의 큰 값을 통해서


## JVM Generations 
객체 할당의 일련의 패턴을 통해서 배운 정보로 JVM의 퍼포먼스를 향상 시킬 수 있습니다. 그러므로 힙영역을 작은 부분으로 쪼개거나 genrations 해야 합니다. heap영역은 Young Generation, Old 또는 Tenured Generation, Permanent Generation 으로 나뉩니다. 

![](https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/images/gcslides/Slide5.png)

Young Generation은 새로운 객체가 할당되고, 나이를 먹습니다. yong generation 영역이 채우지면 minor garbage collection이 일어납니다. 죽은 객체들로 가득찬 young genration은 매우 빠르게 수집 됩니다. 사라남은 객체들은 노화되고, 결국에 old generation으로 이동합니다.

Stop the World Event - 모든 minor garbage collection은 "Stop the World" 이벤트 입니다. 이게 의미하는 바는, 모든 애플리케이션의 스레드가 운영이 완료 될 때 까지 멈춥니다. minor gc는 항상 Stop the World 이벤트 입니다.


Old Generation은 오랫동안 살아 남은 객체들을 저장하는데 사용하곤 합니다. 일반적으로, young generation 객체의 임계값(한계치)를 설정하고, 객체들이 노화가 되고 충족을 시킨다면, 객체들은 old generation으로 이동합니다. 마침내, old generation도 수집되야 할 필요성을 느낍니다. 이 때 이벤트는 major garbage collection이라고 부릅니다.

Major garbage collection 또한 Stop the World 이벤트 입니다. 종종 major collection은 조금 더 느립니다. 이유는 모든 살아있는 객체에 대해서 동작하기 때문 입니다. 따라서 반응형 프로그램에서는 major gc는 반드시 최소화 해야 합니다. 
또한, major gc의 stop the world 이벤트의 길이는 old generation 공간에 사용되는 가비지 콜렉터의 유형에 영향을 받습니. (old generation에서 어떤 gc를 사용하느냐에 따라서 영향을 받습니다)

Permanent Generation은 애플리케이션의 클래스, 메서드를 설명하는 메타 데이터를 포함하고 있습니다. permanent generation은 애플리케이션에서 사용중인 클래스를 기반으로 JVM에 채워집니다. 

JVM이 더 이상 필요하지 않고, 다른 클래스에 공간이 필요할 경우, 클래스가 수집(unloaded) 될 수 있습니다. permanent generation은 full garbage collection에 포함됩니다.

