---
layout: post
title: "[이펙티브 자바 3판] 아이템17.변경 가능성을 최소화하라"
description: 불변 클래스란 무엇인가? 어떤 장점들이 있나? 만드는 방법에 대해서 알아본다.
categories: [Effective Java]
date: 2020-03-01 09:17 +0900
---
![effective java image](https://user-images.githubusercontent.com/28615416/75598228-81ca1c00-5add-11ea-9319-e949af4e07cd.png){:.postImage}

## 불변클래스란? 
그 인스턴스의 내부 값을 수정할 수 없는 클래스다. 불변 인스턴스에 간직된 정보는 고정되어 객체가 파괴되는 순간까지 절대 달라지지 않는다. 

자바에도 다양한 불변 클래스가 있다. String, 기본타입의 박싱된 클래스들, BigInteger, BIgDecimal 이 여기 속한다.
이 클래스들을 불변으로 설계한 데는 그만한 이유가 있다. 불변 클래스는 가변 클래스 보다 설계하고 구현하고 사용하기 쉬우며, 오류가 생길 여지도 적고 훨씬 안전하다.

## 불변 클래스로 만드는 5가지 규칙
- 객체의 상태를 변경하는 메서드(변경자)를 제공하지 않는다.
- 클래스를 확장 할 수 없도록 한다. 하위 클래스에서 부주의하게 혹은 나쁜 의도로 객체의 상태를 변하게 맘ㄴ드는 사애트를 막아준다. 상속을 막는 대표적인 방법은 클래스를 final 로 선언하는 것이지만, 다른 방법도 뒤에 살펴보자.
- 모든 필드를 final로 선언한다. 시스템이 강제하는 수단을 이용해 설계자의 의도를 명확히 드러내는 방법이다. 새로 생성된 인스턴스를 동기화 없이 다른 스레드로 건네도 문제없이 동작하게끔 보장하는데도 필요하다.
- 모든 필드를 private로 선언한다. 필드가 참조하는 가변 객체를 클라이언트에서 직접 접근해 수정하는 일을 막아준다. 기술적으로는 기본 타입 필드나 불변 객체를 참조하는 필드를 public final로 선언해도 불변 객체가 되지만, 이렇게 하면 다음 리리스에서 내부 표현을 바꾸지 못하므로 권하지는 않는다.
- 자신외에는 내부의 가변 컴포넌트에 접근할 수 없도록한다. 클래스에 가변 객체를 참조하는 필드가 하나라도 있다면 클라이언트에서 그 객체의 참조를 얻을 수 없도록 해야 한다. 이런 필드는 절대 클라이언트가 제공한 객체 참조를 가리키게 해서는 안되며, 접근자 메서드가 그 필드를 그대로 반환해서도 안된다. 생성자, 접근자, readObject메서드 모두에서 방어적 복사를 수행하라.



사칙연산 메서드들이 인스턴스 자신은 수정하지 않고 새로운 Complex 인스턴스를 만들어 반환하는 모습에 주목하자. **이처럼 피연산자에 함수를 적용해 그 결과를 반환하지만, 피 연산자 자체는 그대로인 프로그래밍 패턴을 `함수형 프로그래밍`이라 한다.** 이와 달리, `절차적 혹은 명령형 프로그래밍`에서는 메서드에서 피연산자인 자신을 수정해 자신의 상태가 변하게 된다. 또한 메서드 이름으로(add 같은) 동사 대신 (plus 같은) 전치사를 사용한 점에도 주목하자. 이는 해당 메서드가 객체의 값을 변경하지 않는다는 사실을 강조하려는 의도다. 

함수형 프로그래밍에 익숙하지 않다면 조금 부자연스러워 보일 수도 있지만, 이 방식은 프코드에서 불변이 되는 영역의 비율이 높아지는 장점을 누릴 수 있다. 불변 객체는 단순하다. **불변 객체는 생성된 시점의 상태를 파괴 될 때 까지 그대로 간직한다.** 모든 생성자가 클래스 불변식(class invariant)를 보장한다면 그 클래스를 사용하는 프로그래머가 다른 노력을 들이지 않더라도 영원히 불변으로 남는다. 

**불변 객체는 근본적으로 스레드 안전하여 따로 동기화 할 필요 없다.** 여러 스레드가 동시에 사용해도 절대 훼손 되지 않는다. 사실 클래스를 스레드 안전하게 만드는 가장 쉬운 방법이기도 하다. 불변 객체에 대해서는 그 어떤 스레드도 다른 스레드에 영향을 줄 수 없으니, 불변 객체는 안심하고 공유할 수 있다. 따라서 불변 클래스라면 한번 만든 인스턴스를 최대한 재활용하기를 권한다. 가장 쉬운 재활용 방법은 자주 쓰이는 값들을 상수(public static final)로 제공하는 것이다. 예컨대 Complex 클래스는 다음 상수들을 제공할 수 있다.
```java
public static final Complex ZERO = new Complex(0,0);
public static final Complex ONE = new Complex(1,0);
public static final Complex I = new Complex(0,1);
```

불변 클래스는 자주 사용되는 인스턴스를 캐싱하여 같은 인스턴스를 중복 생성하지 않게 해주는 정적 팩터리를 제공할 수 있다. 박싱된 기본 타입 클래스 전부와 BigInteger가 여기 속한다. 이런 정적 팩터리를 사용하면 여러 클라이언트가 인스턴스를 공유하여 메모리 사용량과 가비지 컬렉션 비용이 줄어든다. 

불변 객체를 자유롭게 공유할 수 있다는 점은 방어적 복사도 필요 없다는 결론으로 이른다. 아무리 복사해봐야 원본과 똑같으니 복사 자체가 의미 없다. 그러니 불변 클래스는 clone메서드나 복사 생성자를 제공하지 않는게 좋다. 

불변 객체는 자유롭게 공유할 수 있음은 물론, 불변 객체끼리는 내부 데이터를 공유할 수 있다. 

**객체를 만들 때 다른 불변 객체들을 구성요소로 사용하면 이점이 많다.** 값이 바뀌지 않는 구성요소들로 이뤄진 객체라면 그 구조가 아무리 복잡하더라도 불변식을 유지하기 훨씬 수월하기 때문이다. 좋은 예로 불변 객체는 맵의 키와 집합의 원소르 쓰기에 안성 맞춤이다. 맵이나 집합은 안에 담긴 값이 바뀌면 불변식이 허물어지는데, 불변 객체를 사용하면 그런 걱정은 하지 않아도 된다.

**불변 객체는 그 자체로 실패 원자성을 제공한다.** 상태가 절대 변하지 않으니 잠깐이라도 불일치 상태에 빠질 가능성이 없다.
> 실패원자성이란 메서드에서 예외가 발생한 후에도 그 객체는 여전히(메서드 호출전과 똑같은)유효한 상태여야 한다는 성질이다.

**불변 클래스에도 단점은 있다. 값이 다르면 반드시 독립된 객체로 만들어야 한다는 것이다.** 값의 가짓수가 많다면 이들을 모두 만드는 데 큰 비용을 치러야 한다.

이상으로 불변 클래스를 만드는 기본적인 방법과 불변 클래스의 장/단점을 알아 보았다.

## 불변 클래스를 만드는 또 다른 설계 방법 
클래스가 불변임을 보장하려면 자신을 상속하지 못하게 해야 한다. 자신을 상속하지 못하게 하는 가장 쉬운 방법은 final클래스로 선언하는 것이지만, 더 유연한 방법이 있다. 모든 생성자를 private 혹은 package-private으로 만들고, public 정적 팩터리를 제공하는 방법이다. 
```java
public class Complex {
    private final double re;
    private final double im;

    private Complex(double re, double im){
        this.re = re;
        this.im = im;
    }

    public static Complex valueOf(double re, double im) {   
        return new Complex(re, im);
    }
}
```
패키지 바깥의 클라이언트에서 바라본 이 불변 객체는 사실상 final이다. public이나 protected 생성자가 없으니 다른 패키지에서는 이 클래스