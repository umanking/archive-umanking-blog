---
layout: post
title: "[이펙티브 자바] 2장. 객체 생성과 파괴"
date: 2020-02-16 11:16 +0900
categories: [Effective Java]
published: false
---
> 이번 장은 객체의 생성과 파괴를 다룬다. 객체를 만들어야 할 때와 만들지 말아야 할때를 구분하고 올바른 객체 생성 방법을 알아 본다.
<!-- TOC -->

- [1. 아이템 1.생성자 대신 정적 팩터리 메서드를 고려하라](#1-아이템-1생성자-대신-정적-팩터리-메서드를-고려하라)
    - [1.1. 정적 팩터리 메서드의 장점](#11-정적-팩터리-메서드의-장점)
- [2. 아이템 2 - 생성자 인자가 많을 때는 Builder패턴 적용을 고려하라.](#2-아이템-2---생성자-인자가-많을-때는-builder패턴-적용을-고려하라)
- [3. 아이템 3 - private 생성자나 enum 자료형은 싱글턴 패턴을 따르도록 설계하라](#3-아이템-3---private-생성자나-enum-자료형은-싱글턴-패턴을-따르도록-설계하라)
- [4. 아이템 4. 인스턴스화를 막으려거든 private 생성자를 사용하라](#4-아이템-4-인스턴스화를-막으려거든-private-생성자를-사용하라)

<!-- /TOC -->

# 1. 아이템 1.생성자 대신 정적 팩터리 메서드를 고려하라
클라이언트가 클래스의 인스턴스를 얻는 전통적인 수단은 `public 생성자`다.
하지만 클래스는 생성자와 별도로 `정적 팩터리 메서드`를 제공할 수 있다.

다음 코드는 boolean 기본 타입의 박싱 클래스인 `Boolean`에서 발췌한 코드다. 
```java
public static Boolean valueOf(boolean b){
    return b ? Boolean.TRUE : Boolean.FALSE;
}
```

## 1.1. 정적 팩터리 메서드의 장점

### 장점 1. 이름을 가질 수 있다. 
생성자에 넘기는 매개변수와 생성자 자체만으로는 반환 될 객체의 특성을 제대로 설명하지 못한다. 반면 정적 팩터리 는 이름만 잘 지으면 반환될 객체의 특성을 쉽게 묘사할 수 있다. 

예컨대 BigInteger(int, int, Random)과 정적 팩터리 메서드인 BigInteger.probablePrime 중 어느쪽이 `값이 소수인 BigInteger를 반환한다`는 의미를 더 잘 설명할 것 같은지 생각해 보라.

### 장점 2. 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.
이 덕분에 불변 클래스는 인스턴스를 미리 만들어 놓거나 새로 생성한 인스턴스를 캐싱하여 재활용 하는 식으로 불필요한 객체 생성을 피할 수 있다. 대표적인 예는 Boolean.valueOf(boolean) 메서드는 객체를 아예 생성하지 않는다. 따라서 **(특히 생성 비용이 큰) 같은 객체가 자주 요청되는 상황이라면 성능을 상당히 끌어 올려 준다.** [플라이웨이트 패턴](https://ko.wikipedia.org/wiki/%ED%94%8C%EB%9D%BC%EC%9D%B4%EC%9B%A8%EC%9D%B4%ED%8A%B8_%ED%8C%A8%ED%84%B4)도 이와 비슷한 기법이라 할 수 있다.


반복되는 요청에 같은 객체를 반환하는 식으로 **정적 팩터리 방식의 클래스는 언제 어느 인스턴스를 살아 있게 할지를 철저히 통제할 수 있다**. 이런 클래스를 인스턴스 통제(instance-controlled) 클래스라 한다. 이렇게 통제하는 이유는 무엇일까?

인스턴스를 통제하면 클래스를 싱글턴으로 만들 수도, 인스턴스화 불가로 만들 수도 있다. 또한 불변 값 클래스에서 동치인 인스턴스가 단 하나뿐 임을 보장(a == b일때만 a.eqauls(b)가 성립)할 수 있다. 인스턴스 통제는 플라이웨이트 패턴의 근간이 되며, 열거 타입은 인스턴스가 하나만 만들어짐을 보장한다. 

### 장점 3. 반환 타입의 하위 타입 객체를 반환 할 수 있다.
반환할 객체의 클래스를 자유롭게 선택할 수있게 하는 `엄청난 유연성`을 가진다. API를 만들 때 이 유연성을 응용하면 구현 클래스를 공개하지 않고도 그 객체를 반환할 수 있어 API를 작게 유지 할 수 있다. 이는 인터페이스를 정적 팩터리 메서드의 반환 타입으로 사용하는 인터페이스 기반 프레임워크를 만드는 핵심 기술이기도 하다.





### 장점 4. 입력 매개 변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다. 

### 장점 5. 정적 팩터리 메서드를 작성하는 시점에 반환할 객체의 클래스가 존재하지 않아도 된다. 



```java
public class Account {
    private Long id;
    private String name;
    private int age;
    
    // 기본 생성자
    public Account(){
    }
    
    // 인자가 있는 생성자 
    public Account(String name, int age){
        this.name = name;
        this.age = age;
    }
    
    // 정적 팩터리 메서드
    public Account of(String name, int age){
        this.name = name;
        this.age = age;
    }
}
```




### 정적 팩터리 메서드의 단점 
1. 상속하려면 public 이나 protected 생성자가 필여하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.
2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다. 

### 흔히 사용한는 정적 팩터리 명명 방식 
```
// from: 매개변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드
Date d = Date.from(instant);

// of: 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드
Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);


// instance 혹은 getInstance: (매개변수를 받는 다면) 매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스 임을 보장하지지는 않는다. 
StackWalker luke = StackWalker.getInstance(options);

// create 혹은 newInstance: instance혹은 getInstance와 같지만, 매번 새로운 인스턴스를 생성해 반환함을 보장
Object newArray = Array.newInstance(classObject, arrayLen);
```



# 2. 아이템 2 - 생성자 인자가 많을 때는 Builder패턴 적용을 고려하라. 
1. 점층적인 생성자 패턴
2. 자바빈 패턴(setter 위주의) 
3. Builder패턴 

1번의 경우에 더 많은 인자 개수에 잘 적응 하지 못한다. 코드가 너무 방대해 지는 단점이 있다. 

2번의 경우에는 일관성이 훼손(setter로 변경) 되고, 항상 변경 가능하다. 

1,2번의 문제를 빌더 패턴으로 해결!  😃



실제 실무에서는 `Lombok` 를 사용해서 클래스 위가 X 해당 생성자 위에(O) `@Builder` 를 달아준다. 

```java
public class Member {

    private String name;
    private int age;
    private String address;
    
    @Builder
    public Member(String name, int age) {
        this.name = name;
        this.age = age;
    }
}

```



실제 사용하는 코드는 다음과 같다.

```java
public static void main(String[] args) {
        Member member = Member.builder()
                .name("andrew")
                .age(32)
                .build();
    }
```





# 3. 아이템 3 - private 생성자나 enum 자료형은 싱글턴 패턴을 따르도록 설계하라

싱글턴 패턴을 만들 때는 private 생성자를 통해서 해당 클래스 내에서 만들고(유일하게 만듬) 외부에서 접근할 때는 getInstance()를 통해서 만들어진 객체의 참조값으로 접근하고 사용한다. 싱글턴 패턴을 사용하는 케이스는 파일 시스템, 단 하나만을 요구하는 그런 상황에서 쓰인다. 

enum 자료형을 이용해서 싱글턴 패턴을 만들 수 있는 것 자체를 몰랐다. (기능적으로 public 필드를 사용해서 접근하는 방법과 동일) 하지만 다음과 같은 장점이 존재한다.  

- 좀더 간결하다는 것

    ```jav
    public enum Elvis {
        INSTANCE;
    }
    ```

    

- 직렬화가 자동으로 처리 된다는 것 

- 리플렉션을 통한 공격에도 안전

    ```java
    public class PrivateInvoker {
    
        public static void main(String[] args) throws Exception {
            Constructor<?> constructor = Private.class.getDeclaredConstructors()[0];
            constructor.setAccessible(true); //접근할 수 있는 권한을 획득 함
            Private p = (Private) constructor.newInstance();
    
        }
    
        class Private {
           	private Private() {
                System.out.println("hello world");
            }
        }
    }
    
    ```

그래서 원소가 하나 뿐인 enum 자료형이야 말로 싱글턴을 구현하는 가장 좋은 방법이다.

# 4. 아이템 4. 인스턴스화를 막으려거든 private 생성자를 사용하라

<b>단순히 정적 메서드와 정적 필드만을 담은 클래스</b>를 만들어야 하는 경우가 있다. 이 방식은 객체 지향적으로 좋은 않은 클래스라고 여길 수 있지만, 이러한 클래스도 나름의 쓰임새가 있다.

예를 들어 ```java.lang.Math```와 ```java.util.Arrays``` 클래스처럼 기본 타입 값이나 배열 관련 메서드를 모아 놓을 수 있다. 또한, ```java.util.Collections```처럼 특정 인터페이스를 구현하는 객체를 생성해주는 정적 메서드를 가지고 있는 클래스가 있을 수 있다.

다음 코드는 Math 클래스의 일부분이다.

```java
public final class Math {

    private Math() {}

    public static final double E = 2.7182818284590452354;

    public static final double PI = 3.14159265358979323846;


    public static double sin(double a) {
        return StrictMath.sin(a); // default impl. delegates to StrictMath
    }

    public static double cos(double a) {
        return StrictMath.cos(a); // default impl. delegates to StrictMath
    }

    public static double tan(double a) {
        return StrictMath.tan(a); // default impl. delegates to StrictMath
    }
    
    // 생략 ...
}
```

<br/>

<b>정적 멤버만 담은 유틸리티 클래스는 사실 인스턴스로 만들어 쓰려고 설계한 것이 아니다.</b> Math 클래스는 ```final``` 클래스이기 때문에 상속해서 하위 클래스에 메서드를 넣을 수도 없다. 그리고 private 생성자를 만들었기 때문에 컴파일러가 자동으로 기본 생성자를 만들지도 않는다.

유틸리티 클래스를 추상 클래스로 만들면 인스턴스 생성하는 것을 막을 수 없다. 이러한 종류의 클래스는 애초에 다른 클래스에서 상속 받아 사용하도록 설계한 것이 아니기 때문에 인스턴스가 생성되는 것을 막는 것이 좋다.

인스턴스 생성을 막는 방법은 간단한다. 개발자가 클래스의 기본 생성자를 빼먹으면 컴파일러가 기본 생성자를 추가해준다. 그렇기 때문에 명시적으로 <b>private 생성자</b>를 클래스에 추가하면 클래스의 인스턴스 생성을 막을 수 있다.

다음 코드는 Utils 클래스에 private 생성자를 넣는 예제이다.

```java
public class Utils {

    private Utils() {
        throw new Exception("Utils 클래스는 인스턴스화를 할 수 없습니다.");
    }

    public static int plus(int a, int b) { 
        return a + b
    }

    // 정적 멤버 선언...
}
```

<br/>

만약 Utils 클래스를 상속 받아서 사용하는 코드를 작성하면, 하위 클래스 생성자에서는 에러가 발생한다. 하위 클래스에서는 상위 클래스의 생성자를 반드시 호출해야 하는데, 이를 private 접근자로 선언했으므로 상위 클래스의 생성자에 접근할 수 없다.