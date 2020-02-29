---
layout: post
title: "[이펙티브 자바 3판] 아이템10.equals는 일반 규약을 지켜 재정의하라"
description: 이펙티브 자바, 아이템10.equals는 일반 규약을 지켜 재정의하라
categories: [Effective Java]
date: 2020-02-26 05:18 +0900
---
![effective java image](https://user-images.githubusercontent.com/28615416/75598228-81ca1c00-5add-11ea-9319-e949af4e07cd.png){:.postImage}

# 3장 모든 객체의 공통 메서드
Object는 객체를 만들 수 있는 구체 클래스지만 기본적으로는 상속해서 사용하도록 설계되었다.
Object에서 final이 아닌 메서드(equals, hashcode, toString, clone, finalize)는 모두 재정의를 염두에 두고 설계된 것이라 재정의 시 지켜야 하는 일반 규약이 명확히 정의되어 있다. 그래서 Object를 상속하는 클래스, 즉 모든 클래스는 이 메서드들을 일반 규약에 맞게 재정의해야 한다. 이 메서드들을 잘못 구현하면 대상 클래스가 이 규약을 준수한다고 가정하는 클래스(HashMap과 HashSet)를 오동작하게 만들 수 있다. 

다음과 같은 상황중 하나에 해당하면 재정의 하지 말라 
- 각인스턴가 본질적으로 고유하다. 값 객체가 아닌경우
- 인스턴스의 '논리적 동치성'을 검사할 일이 없다. 
- 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다. 예컨대 대부분의 Set구현체는 AbstractSet이 구현한 equals를 상속받아 쓰고, List구현체들은 AbstractList로부터 상속받아 쓴다.
- 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다. 실수로라도 equals 가 호출되는 걸 막고 싶다면 다음처럼 구현해두자.
    ```java
    @Override public boolean equals(Object o){
        throw new AssertionError(); // 호출 금지
    }
    ```

## 언제 정의할까? 
주로 값 클래스들이다. 객체 식별성이 아니라 논리적 동치성을 확인해야 하는데, 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때다. 주로 값 클래스들이 여기 해당한다.

값 클래스란 Integer와 String처럼 값을 표현하는 클래스를 말한다.
두 값 객체를 equals로 비교하는 프로그래머는 객체가 같은지가 아니라, 값이 같은지를 알고 싶어 할 것이다. 

값 클래스라 해도, 값이 같은 인스턴스가 둘 이상 만들어지지 않음을 보장하는 인스턴스 통제 클래스라면 equals를 재정의하지 않아도 된다. Enum도 여기에 해당한다.

## equals 메서드를 재정의 일반 규약 
다음은 Object 명세에 적힌 규약이다.
equals메서드는 동치관계를 구현하며, 다음을 만족한다.
- 반사성: null아닌 아닌 모든 참조값 x에 대해, x.equals(x)는 true다.
- 대칭성: null이 아닌 모든 참조값 x,y에 대해 x.equals(y)가 true이면 y.eqauls(x)도 true
- 추이성: null이 아닌 모든 참조값 x,y,z 에 대해 x.equals(y)가 true이고, y.equals(z)도 true면, x.equals(z)도 true다
- 일관성: null이 아닌 모든 참조값 x,y에 대해 x.equals(y)를 반복해서 호출하면 항상 ㅅrue를 반환하거나 항상 false를 반환한다.
- null-아님: null이 아닌 모든 참조 값 x에 대해 x.equals(null)은 false다.


> John Donne의 말처럼 세상에 홀로 존재하는 클래스는 없다. 한 클래스의 인스턴스는 다른곳으로 빈번히 전달된다. 그리고 컬렉션 클래스들을 포함해 수많은 클래스는 전달받은 객체가 equals 규약을 지킨다고 가정하고 동작한다.

## 대칭성 위배 
```java
public final class CaseInsesitiveString {
    private final String s;

    public CaseInsesitiveString(String s){
        this.s = Objects.requiredNonNull(s);
    }

    @Override public boolean equals(Object o){
        if (o instanceof CaseInsensitiveString){
            return s.equalsIgnoreCase(
                ((CaseInsensitiveString) o).s);
        }
        // 한 방향으로만 작동한다.
        if (o instanceof String){
            return s.equalsIgnoreCase((String) o);
        }
        return false;
    }
}
```
```java
CaseInsensitiveString cis = new CaseInsensitiveString("Andrew")
String s = "andrew";
```
cis.equals(s)는 true를 반환한다. 문제는 String의 equals는 CaseInsensitiveString의 존재를 모른다. 따라서 s.equals(cis)는 false를 반환하여, 대칭성을 명백히 위반한다. 
equals 규약을 어기면 그 객체를 사용하는 다른 객체들이 어떻게 반응할지 알 수 없다. 

이를 해결하려면 CaseInsensitiveString의 equals를 String과도 연동하겠다는 허황한 꿈을 버려야 한다. 
```java
 @Override public boolean equals(Object o){
     return o instanceof CaseInsensitiveString &&
            ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
```

## 추이성
상위 클래스에는 없는 새로운 필드를 하위 클래스에 추가하는 상황을 생각해보자.
Point 클래스이고 이 클래스를 확장해서 점에 색상을 더하는 ColorPoint를 생각해보자.

**결국 구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족 시킬 방법은 존재하지 않는다.**
구체 클래스의 하위 클래스에서 값을 추가할 방법은 없지만 괜찮은 우회 방법이 하나 있다. "상속 대신 컴포지션을 사용하라"는 아이템 18의 조언을 따른다. 

또한, 추상 클래스의 하위 클래스에서라면 equals 규약을 지키면서도 값을 추가할 수 있다. 예컨대 아무런 값을 갖지 않는 추상 클래스인 Shpae를 위에 두고, 이를 확장하여 radius필드를 추가한 Circle 클래스와 length와 width 필드를 추가한 Rectagnle 클래스를 만들 수 있다. 상위 클래스를 직접 인스턴스로 만드는 게 불가능하다면 지금까지 이야기한 문제들은 일어나지 않는다. 


![스크린샷 2020-02-26 오전 6 13 37](https://user-images.githubusercontent.com/28615416/75288226-3c002000-585f-11ea-8959-42c0e687057f.png)

## 일관성
두 객체가 같다면 앞으로도 영원히 같아야 한다. 가변객체는 비교 시점에 따라 서로 다를 수도 혹은 같을 수도 있다. 반면, 불변 객체는 한번 다르면 끝까지 달라야 한다. 클래스를 작성할때 불변 클래스로 만드는게 나을지 심사숙고 하자(아이템 17)

클래스가 불변이든 가변이든 equals의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안된다. 예컨데 java.net.URL의 equals는 주어진 URL과 매핑된 호스트의 IP 주소를 이용해 비교한다. 호스트 이름을 IP주소로 바꾸려면 네트워크를 통해야 하는데, 그 결과가 항상 같다고 보장할 수 없다. 
이런 문제를 피하려면 equals는 항상 메모리에 존재하는 객체만을 사용한 결정적 계산만 수행해야 한다.

## null-아님
이름 처럼 모든 객체가 null과 같지 않아야 한다는 뜻이다. 수 많은 클래스가 다음 코드 처럼 입력이 null인지 확인해 자신을 보호한다.
```java
// 명시적 null 검사 - 필요없다.
@Override public boolean equals(Object o){
    if (o == null)
        return false;
    ...
}
```
이런 검사는 필요치 않다. 동치성 검사하려면 equals는 건네받은 객체를 적절히 형변환한 후 필수 필드들의 값을 알아야내야 한다. 그러려면 형변환에 앞서 instanceof 연산자로 입력 매개변수가 올바른 타입인지 검사해야 한다.
```java
// 묵시적 null 검사 - 이쪽이 낫다
@Override public boolean equals(Object o){
    if (!(o instanceof MyType))
        return false;
    MyType mt = (MyType) o;
    ...
}
```
equals가 타입을 확인하지 않으면 잘못된 타입이 인수로 주어졌을 때 ClassCastException을 던져서 일반 규약을 위배하게 된다. 그런데 instancef는 첫 번째 피연산자가 null이면 false를 반환한다. 따라서 입력이 null이면 타입 확인 단계에서 이미 false를 반환하기 때문에 **null검사를 명시적으로 하지 않아도 된다.**

## 양질의 equals메서드 구현방법을 단계별 정리 
1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다. 
2. instanceof 연산자로 입력이 올바른 타입인지 확인한다. 
3. 입력을 올바른 타입으로 형변환한다. 
4. 입력 객체와 자기 자신의 대응되는 핵심 필드들이 모두 일치하는지 하나씩 검사한다. 2단계에서 인터페이스를 사용했다면 입력의 필드 값을 가져올 때도 그 인터페이스의 메서드를 사용해야 한다. 타입이 클래스라면 (접근 권한에 따라) 해당 필드에 직접 접근할 수도 있다.

float와 double을 제외한 기본 타입 필드는 == 연산자로 비교하고, 참조 타입필드는 각각의 equals메서드로 , float와 double 필드는 각각 정적 메서드인 Float.compare(float, float)와 Double.compare(double, double)로 비교한다.

Float.equals와 Double.equals 메서드를 사용할 수 있지만, 이 메서드들은 오토박싱을 수반할 수 있으니 성능상 좋지 않다. 

배열의 모든 원소가 핵심 필드라면 Arrays.equals 메서드들 중 하나를 사용하자.

때론 null도 정상 값으로 취급하는 참조 타입 필드들도 있다. 이런 필드는 정적 메서드인 Objects.equals(Object, Object)로 비교해 NPE 발생을 예방하자.

어떤 필드를 먼저 비교하느냐가 equals의 성능을 좌우하기도 한다. 최상의 성능을 바란다면 다를 가능성이 더 크거가 비교하는 비용이 싼 필드를 먼저 비교하자. 동기화용 락 필드 같이 객체의 논리적 상태와 관련 없는 필드는 비교하면 안된다. 핵심 필드로 부터 계산해 낼 수 있는 파생필드 역시 굳이 비교할 필요는 없지만, 비교하는 쪽이 더 빠를 때도 있다. 파생 필드가 객체 전체의 상태를 대표하는 상황이 그렇다.


equals를 다 구현했다면 세가지만 자문해보자. 대칭적인가? 추이성이 있는가? 일관적인가? 
자문에서 끝내지 말고 단위 테스트를 작성해 돌려보자. 단, autovalue를 이용해 작성했다면 테스트는 생략가능

## 실제 예
```java
public final class PhoneNumber {
    private final short areaCode, prefix, lineNum;
    
    public PhoneNumber (int areaCode, prefix, lineNum){
        this.areaCode = rangeCheck(areaCode, 999, "지역코드");
        this.prefix = rangeCheck(prefix, 999, "프리픽스");
        this.lineNum = rangeCheck(lineNum, 999, "가입자번호");
    }

    private static short rangeCheck(int val, int max, String arg){
        if(val < 0 || val > max){
            throw new IllegalArgumentException(arg + ":" + val);
        }
        return (short) val;
    }

    @Override public boolean equals(Object o){
        if (o == this){
            return true;
        }
        if (!(o instanceof PhoneNumber)){
            return false;
        }

        PhoneNumber pn = (PhoneNumber) o;
        return pn.lineNum == linNum && pn.prefix == prefix 
            && pn.areaCode = areaCode;
    }
}
```

드디어 마지막 주의 사항
- equals를 재정의할 땐 hashcode도 반드시 재정의하자
- 너무 복잡하게 해결하지 말자. 필드들의 동치성만 검사해도 equals규약을 어렵지 않게 지킬 수 있다.
- Object외의 타입을 매개변수로 받는 equals 메서드는 선언하지 말자.
    ```java
    // 잘못된 예 - 입력 타입은 반드시 Object이어야 한다.
    public boolean equals(MyClass o)
    ```


equals(hashCode도 마찬가지)를 작성하고 테스트하는 일은 지루하고 이를 테스트하는 코드도 항상 뻔하다. 다행히 이 작업을 대신해줄 오픈소스가 있으니 구글이 만든 AutoValue 프레임워크다. 클래스에 애너테이션 하나만 추가하면 AutoValue가 이 메서드들을 알아서 작성해주며, 여러분이 직접 작성하는 것과 근본적으로 똑같은 코드를 만들어 준다.

## 핵심정리
꼭 필요한 경우가 아니면 equals 재정의하지 말자. (값객체인 경우에는 필요함, 논리적 동치성이 요구되는 경우) 많은 경우에 Object의 equals가 여러분이 원하는 비교를 정확히 수행해준다. 재정의 할 때는 클래스의 핵심 필드를 모두 비교하고 다섯가지 규약을 지켜가며 비교해야 한다.