---
layout: post
title: "[이펙티브 자바 3판] 아이템 2.생성자 인자가 많을 때는 Builder패턴 적용을 고려하라."
categories: [Effective Java]
date: 2020-02-16 18:53 +0900
---

<!-- TOC -->

- [점층적인 생성자 패턴 (안정성)](#점층적인-생성자-패턴-안정성)
- [자바빈즈 패턴(JavaBeans pattern) (가독성)](#자바빈즈-패턴javabeans-pattern-가독성)
- [빌더 패턴(Builder pattern) (안정성 + 가독성)](#빌더-패턴builder-pattern-안정성--가독성)
- [롬복을 사용해서 Builder 패턴 사용 하기](#롬복을-사용해서-builder-패턴-사용-하기)

<!-- /TOC -->




## 점층적인 생성자 패턴 (안정성)
정적 팩터리와 생성자에는 똑같은 제약이 하나 있다. 선택적 매개변수가 많을 때 적절히 대응하기 어렵다는 점이다. 식품 포장의 영양 정보를 표현하는 클래스를 생각해 보자. 이런 클래스용 생성자 혹은 정적 팩터리는 어떤 모습일까? 프로그래머들은 이럴 때 점층적 생성자 패턴(telescoping constructor pattern)을 즐겨 사용한다. 
> 점층적 생성자 패턴은 필수 매개변수만 받는 생성자, 필수 매개변수와 선택 매개변수 1개를 받는 생성자, 선택 매개변수를 2개까지 받는 생성자 ,... 형태로 선택 매개변수를 전부다 받는 생성자까지 늘려가는 방식이다.
**결국, 점층적 생성자 패턴도 쓸 수 있지만, 매개변수 개수가 많아지면 클라이언트 코드를 작성하거나 읽기 어렵다.** 

## 자바빈즈 패턴(JavaBeans pattern) (가독성)
매개 변수가 없는 생성자로 객체를 만든 후, 세터(Setter) 메서드들을 호출해 원하는 매개변수의 값을 설정하는 방식이다. 점층적인 생성자 패턴의 단점들이 자바빈즈 패턴에서 더 이상 보이지 않는다. 하지만 불행히도 자바빈즈는 자신만의 심각한 단점을 지니고 있다. 자바빈즈 패턴에서는 객체 하나를 만들려면 메서드를(setter) 여러개 호출해야 하고, **객체가 완전히 생성되기 전 까지는 일관성(consitenct)이 무너진 상태에 놓이게 된다.** 점층적인 생성자 패턴에서는 매개변수들이 유효한지를 생성자에서만 확인하면 일관성을 유지할 수 있었는데, 그 장치가 완전히 사라진 것이다. 이처럼 일관성이 무너지는 문제 때문에 자바빈즈 패턴에서는 클래스를 불변(아이템 17)으로 만들 수 없으며 스레드 안전성을 얻으려면 프로그래머가 추가 작업을 해줘야만 한다. 

## 빌더 패턴(Builder pattern) (안정성 + 가독성)
점층적 생성자 패턴의 안전성과 자바 빈즈 패턴의 가독성을 겸비한 빌더 패턴(Builder pattern)이다. 클라이언트는 필요한 객체를 직접 만드는 대신, 필수 매개 변수만으로 생성자(혹은 정적 팩터리)를 호출해 빌더 객체를 얻는다. 그런 다음 빌더 객체가 제공하는 세터 메서드들로 원하는 선택 매개변수들을 설정한다. 마지막으로 매개 변수가 없는 build 메서드를 호출해 우리에게 필요한 (보통은 불변인) 객체를 얻는다. 

```java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;

    public static class Builder {
        //필수 매개변수
        private final int servingSize;
        private final int servings;

        //선택 매개변수 - 기본값으로 초기화
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int val) {
            calories = val;
            return this;
        }

        public Builder fat(int val) {
            fat = val;
            return this;
        }

        public Builder sodium(int val) {
            sodium = val;
            return this;
        }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
    }
```
NutritionFacts 클래스는 불변이며, 모든 매개변수의 기본 값들을 한곳에 모아 뒀다. 빌더의 세터 메서드들은 빌더 자신을 반환하기 때문에 연쇄적으로 호출 할 수 있다.


다음은 실제 클라이언트에 사용하는 코드이다.

```java
NutritionFacts nutritionFacts = new NutritionFacts.Builder(240, 8)
                .calories(0).fat(2).sodium(2).build();
```

잘못된 매개변수를 최대한 일찍 발견하려면 빌더의 생성자와 메서드에서 입력 매개변수를 검사하고, build 메서드가 호출하는 생성자에서 여러 매개 변수에 걸친 불변식(invariant)를 검사하자. 
검사해서 잘못된 점을 발견하면 어떤 매개변수가 잘 못되었는지를 자세히 알려주는 메세지를 담아 IllegalArgumentExceptiond을 던지면 된다. 


> 불변(immutable or immutablity)는 어떠한 변경도 허용하지 않는다는 뜻으로, 주로 변경을 허용하는 가변(mutable) 객체와 구분하는 용도로 쓰인다. 대표적으로 String 객체는 한번 만들어지면 절대 값을 바꿀 수 없는 불변 객체다.

> 한편, 불변식(invariant)는 프로그램이 실행되는 동안, 혹은 정해진 기간 동안 반드시 만족해야 하는 조건을 말한다. 다시 말해 변경을 허용할 수는 있으나 주어진 조건 내에서만 허용한다는 뜻이다. 예컨대 리스트의 크기는 반드시 0 이상이어야 하니, 만약 한 순간이라도 음수 값이 된다면 불변식이 깨진 것이다.

## 롬복을 사용해서 Builder 패턴 사용 하기
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

