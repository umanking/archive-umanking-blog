---
layout: post
title: "디자인패턴 - Composite 패턴"
date: 2020-01-05 14:02 +0900
categories: [Java]
---

## 개요
- 이번에는 디자인 패턴중에 Composite 패턴에 대해서 알아보자. 
- 컴포지트 패턴은 **부분과 전체의 관계인** 경우 적용할 수 있다.
- Computer, Keyboard, Monitor 클래스를 통한 제품 가격 정보를 출력하는 간단한 프로그램을 통해서 부분과 전체의 설계 관점에서 어떻게 변하는지 알아보자.

```java
public class Keyboard {
    int price;
    // 생성자 & getter 생략
}

public class Monitor {
    int price;
    // 생성자 & getter 생략
}
```

```java
public class Computer {
    Keyboard keyboard;
    Monitor monitor;
    int price;

    public Computer(Keyboard keyboard, Monitor monitor) {
        this.keyboard = keyboard;
        this.monitor = monitor;
    }

    public int getPrice() {
        this.price = keyboard.getPrice() + monitor.getPrice();
        return price;
    }
}
```
각각 Keyboard, Monitor클래스가 있고, Computer 클래스에서 가격을 합산한다. 


```java
public class Client {
    public static void main(String[] args) {
        Monitor monitor = new Monitor(320);
        Keyboard keyboard = new Keyboard(100);
        Computer computer = new Computer(keyboard, monitor);
        int price = computer.getPrice();

        System.out.println("computuer price: " + price );
    }
}
```
실제 클라이언트 코드에서는 Monitor, Keyboard 가격을 통해서 컴퓨터의 총 비용을 환산한다.

## 새로운 컴퓨터의 부품을 추가한다면?
만약에 컴퓨터 부품(Speaker)가 추가했을 때 어떤 작업을 해야 할까? 

```java
public class Computer {
    Keyboard keyboard;
    Monitor monitor;
    // 스피커를 추가한다.
    Speaker speaker;
    int price;

    // 생성자에서 스피커를 추가한다.
    public Computer(Keyboard keyboard, Monitor monitor, Speaker speaker) {
        this.keyboard = keyboard;
        this.monitor = monitor;
        this.speaker = speaker;
    }

    public int getPrice() {
        // 가격을 합산하는데도, 추가해 준다. 
        this.price = keyboard.getPrice() + monitor.getPrice() + speaker.getPrice();
        return price;
    }
}
```
- Computer 클래스에서 Speaker 라는 클래스를 필드에 추가하고
- Computer 클래스의 생성자에 Speaker를 파라미터로 추가하고
- Computer 클래스의 getPrice 메서드를 수정해야 한다.
  
OCP를 위반한다. 스피커 제품 하나를 추가했을 뿐인데, Computer 클래스를 계속 수정해야 한다. 또한, 가격 정보에서 speaker.getPrice() 코드를 깜빡하게 된다면 실제 결제하는 금액에서 스피커 가격이 포함되지 않으니 이건 엄청난 손해를 일으킬 수 있다. (물론 클라이언트들은 좋아하겠지만...)


## 해결책 - 컴포지트 패턴을 통해서 구조를 변경
`ComputerDevice` 라는 추상 클래스를 만들고, 기존의 Computer에서는 여러개의 ComputerDevice를 가질 수 있는 포함하는 구조(Composite)로 가야한다.

```java
public abstract class ComputerDevice {

    abstract int getPrice();
}

public class Keyboard extends ComputerDevice {
    int price;
    // 생성자 & getter 생략
}

public class Monitor extends ComputerDevice {
    int price;
    // 생성자 & getter 생략
}

```

```java
public class Computer extends ComputerDevice{

    static List<ComputerDevice> components = new ArrayList<>();

    int price;

    public List<ComputerDevice> addComponent(ComputerDevice computerDevice) {
        components.add(computerDevice);
        return components;
    }

    public void removeComponent(ComputerDevice computerDevice) {
        components.remove(computerDevice);
    }

    @Override
    int getPrice() {
        for (ComputerDevice computerDevice : components) {
            this.price += computerDevice.getPrice();
        }
        return this.price;
    }
}
```
이제 Computer클래스는 복수개의 ComputerDevice를 갖는 components 리스트를 만들고, 해당 components에 추가하고, 삭제할 수 있도록 메서드를 구현했다. 마지막으로 getPrice는 components의 순회하면서 price정보를 합산해서 리턴한다. 

```java
public class Client {
    public static void main(String[] args) {
        ComputerDevice monitor = new Monitor(320);
        ComputerDevice keyboard = new Keyboard(100);
        ComputerDevice speaker = new Speaker(50);

        Computer computer = new Computer();
        computer.addComponent(monitor);
        computer.addComponent(keyboard);
        computer.addComponent(speaker);


        int price = computer.getPrice();

        System.out.println("computuer price: " + price );


    }
}
```
실제 클라이언트 코드에서 호출하는 것을 보면, Speaker라는 새로운 제품이 추가되었을 때, 클라이언트 코드에서 addComponent로 ComputerDevice만 추가해 주면 된다. 실제 Computer에서 어떤 코드도 수정이 필요없다. 이 후에도 어떤 컴퓨터 디바이스를 추가해도 쉽게 확장 가능하게 변경되었다. 
