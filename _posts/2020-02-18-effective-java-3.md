---
layout: post
title: "[이펙티브 자바 3판] 아이템 3.private 생성자나 열거 타입으로 싱글턴임을 보증하라."
categories: [Effective Java]
date: 2020-02-18 22:03 +0900
---

<!-- TOC -->

- [싱글턴을 만드는 첫번째 방법](#싱글턴을-만드는-첫번째-방법)
- [싱글턴을 만드는 두번째 방법](#싱글턴을-만드는-두번째-방법)
    - [장점 비교](#장점-비교)
    - [직렬화 문제](#직렬화-문제)
- [싱글턴을 만드는 세번째 방법](#싱글턴을-만드는-세번째-방법)

<!-- /TOC -->
클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어려워 질 수 있다. 
> 싱글턴이란? 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다. 전형적인 예로는 함수와 같은 무상태 객체나 설계상의 유일해야 하는 시스템 컴포넌트를 들 수 있다.
타입을 인터페이스로 정의한 다음 그 인터페이스를 구현해서 만든 싱글턴이 아니라면 싱글턴 인스턴스를 가짜(mock) 구현으로 대체할 수 없기 때문이다.

## 싱글턴을 만드는 첫번째 방법
```java
public class Elvis {
    public static final Elvis INSTANCE = NEW Elvis();
    
    private Elvis(){
    }
}
```
private 생성자는 public statkc final 필드인 `Elvis.INSTANCE`를 초기화 할 때 딱 한 번만 호출된다.
public이나 protected 생성자가 없으므로 Elivis 클래스가 초기화될 때 만들어진 인스턴스가 전체 시스템에서 하나 뿐임을 보장한다.
예외는 단 한가지, 리플렉션 API AccessibleObject.setAccessible을 사용해서 private 생성자를 호출 할 수 있다.
이러한 공격을 방어하려면 생성자를 수정하여 두 번째 객체가 생성되려 할 때 예외를 던지게 하면 된다.


## 싱글턴을 만드는 두번째 방법
```java
public class Elvis {
    private static final Elvis INSTANCE = NEW Elvis();
    
    private Elvis(){
    }

    public static Elvis getInstance(){
        return INSTANCE;
    }
}
```
두번째 방법은 정적팩터리 메서드를 public static 멤버로 제공한다. `Elvis.getInstance`는 항상 같은 객체의 참조를 반환하므로 제2의 Elvis 인스턴스란 결코 만들어지지 않는다. (리플렉션을 통한 예외는 똑같이 적용된다.)


### 장점 비교
첫번째 방식의 장점
- 해당 클래스가 싱글턴임이 API에 명백히 드러난다는 것이다. public static 필드가 final 이니 절대로 다른 객체를 참조할 수 없다.
- 간결함

두번째 방식의 장점 
- API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다는 점이다. 유일한 인스턴스를 반환하던 팩터리 메서드가 (예컨대) 호출하는 스레드별로 다른 인스턴스를 넘겨주게 할 수 있다.
- 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다는 점(아이템 30)
- 정적 팩터리의 메서드 참조를 공급자(supplier)로 사용할 수 있다는 점. 가령 Elivs::getInstance를 Supplier<Elvis>로 사용하는 식이다. **이러한 장점이 필요하지 않다면 public 필드 방식이 좋다.**


### 직렬화 문제
둘 중 하나의 방식으로 만든 싱글턴 클래스를 직렬화하려면 단순히 Serializable을 구현하고 선언하는 것 만으로는 부족하다. 모든 인스턴스필드를 일시적(transient)라고 선언하고 readResolve 메서드를 제공해야 한다. 이렇게 하지 않으면 직렬화된 인스턴스를 역직렬화할 때 마다 새로운 인스턴스가 만들어진다. 위의 2번째 코드의 예에서 가짜 Elvis가 탄생한다는 뜻이다. 이를 예방하고 싶다면 Elvis 클래스에 다음의 readResolve 메서드를 추가하자.

```java
// 싱글턴임을 보장해 주는 readResovle 메서드
private Object readResovle(){
    // 진짜 Elvis를 반환하고, 가짜 Elvis는 가비지 컬렉터에 맡긴다.
    return INSTANCE;
}
```


## 싱글턴을 만드는 세번째 방법
원소가 하나인 열거 타입을 선언하는 것이다.
```java
public enum Elvis {
    INSTANCE;
}
```
public 필드 방식과 비슷하지만, **더 간결하고**, 추가 노력 없이 직렬화 할 수 있고, 심지어 아주 복잡한 **직렬화 상황**이나 **리플렉션 공격**에서도 제2의 인스턴스가 생기는 일을 완벽히 막아준다. 대부분 상황에서 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다. 단, 만들려는 싱글턴이 Enum외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없다.(열거 타입이 다른 인터페이스를 구현하도록 선언할 수 는 있다)