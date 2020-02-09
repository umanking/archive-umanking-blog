---
layout: post
title: "[디자인패턴] 커맨드 패턴 (Command Pattern)"
date: 2020-01-12 13:25 +0900
categories: [java]
tags: [java]
---
## 개요
이번 시간에는 **기능을 캡슐화** 하는 커맨드 패턴에 대해서 알아 보겠습니다.

## 예제 
Button 을 누르면 Lamp가 켜지는 간단한 애플리케이션을 만들어 보겠습니다.
```java
public class Lamp {
    
    public void turnOn() {
        System.out.println("turn on");
    }
}
```

```java
public class Button {

    private final Lamp lamp;

    public Button(Lamp lamp) {
        this.lamp = lamp;
    }

    public void pressed() {
        this.lamp.turnOn();
    }
}
```
Lamp클래스를 생성자를 통해서 주입해주고, pressed()를 호출하면 lamp의 trunOn메서드를 호출합니다. 

```java
public class Client {
    public static void main(String[] args) {

        Lamp lamp = new Lamp();
        Button button = new Button(lamp);
        button.pressed();

    }
}
``` 

## Button이 Lamp를 키는 것 외에 Alarm을 켜는 기능을 추가한다면 ?
버튼의 pressed()를 호출했을 때, 지금은 lamp를 키는 기능으로 코딩 되어 있기 때문에 Alarm과 구분하기 위해서 
Mode라는 클래스를 추가합니다.

```java
public enum Mode {
    LAMP, ALARM
}
```

```java
public class Alarm {

    public void start() {
        System.out.println("alarm start......");
    }
}
```

```java
public class Button {

    private final Lamp lamp;

    // Alarm 클래스 추가
    private final Alarm alarm;
    private Mode mode;

    // 생성자 추가
    public Button(Lamp lamp, Alarm alarm) {
        this.lamp = lamp;
        this.alarm = alarm;
    }

    public void setMode(Mode mode) {
        this.mode = mode;
    }

    // switch 추가
    public void pressed() {
        switch (this.mode) {
            case LAMP:
                this.lamp.turnOn();
                break;
            case ALARM:
                this.alarm.start();
                break;
        }
    }
}
```
Button 클래스에 Alarm 클래스를 추가했고, pressed() 메서드가 mode에 의해서 어떤 메서드를 호출 할지 switch 문을 타게 됩니다. 

## 문제점은 무엇일까요 ?
- 새로운 기능을 추가할 때마다, Mode, Button 클래스를 코드를 수정합니다. (OCP 위반)
- 또한, swtich 케이스가 좋지 못하다는 것은 모두가 아실 겁니다. (코딩 관점)


## 해결방법
위의 문제에서도 살펴봤듯이, 문제점은 새로운 기능을 추가할 때 마다 버튼 클래스를 수정하는 것에 문제가 있었습니다. 버튼 클래스를 직접 수정하는 것이 아닌, 최초의 디자인 했을 때의 `버튼은 누른다.` 이 디자인은 그대로 두고, 새롭게 추가되는 기능들을 Button 클래스 외부에서 제공받아 캡슐화 해 pressed 메서드를 호출할 수 있습니다. 

```java
public interface Command {
    void execute();
}
```
Command 인터페이스를 선언하고, execute() 메서드를 선언합니다. 나중에 Button의 생성자로 오로지 이 Command 인터페이스만 넘길 겁니다. 
그리고 LampOnCommand 클래스와 AlaramOnCommand 클래스는 Command 인터페이스를 구현해서, 각각 execute()는 lamp.turnOn() 과 alarm.start() 메서드를 호출할 것입니다.
```java
public class LampOnCommand implements Command {

    private final Lamp lamp;

    public LampOnCommand(Lamp lamp) {
        this.lamp = lamp;
    }

    @Override
    public void execute() {
        lamp.turnOn();
    }
}
```

```java
public class AlarmOnCommand implements Command {

    private final Alarm alarm;

    public AlarmOnCommand(Alarm alarm) {
        this.alarm = alarm;
    }

    @Override
    public void execute() {
        alarm.start();
    }
}
```

```java
public class Button {
    
    private final Command command;

    public Button(Command command) {
        this.command = command;
    }
    
    public void pressed() {
        command.execute();
    }

}
```
이제 Button 클래스는 기존의 덕지덕지 여러 가지 부분이 혼재 했던 클래스가 아니라, 오로지 Command 클래스만을 생성자로 주입 받아서 execute() 메서드를 호출합니다. 


```java
public class Client {
    public static void main(String[] args) {

        LampOnCommand lampOnCommand = new LampOnCommand(new Lamp());
        AlarmOnCommand alarmOnCommand = new AlarmOnCommand(new Alarm());
        
        // lampOnCommand setting
        Button button = new Button(lampOnCommand);
        button.pressed();

        // alarmOnCommand setting
        Button button1 = new Button(alarmOnCommand);
        button1.pressed();

    }
}
```
클라이언트 코드를 보면, 원하는 Command를 셋팅하고 button은 그냥 pressed()만 호출 합니다. 

지금까지 보면, 복잡해보일 수도 있지만, 여기에 만약에 램프를 끄는 기능인 LampOff기능을 추가 한다고 생각하면, 오로지 할 일은 LampOffOnCommand 클래스를 만들어서 구현만 하면 됩니다. Button클래스에 어떤 것도 손댈 필요가 없습니다. 이것이 바로 기능의 캡슐화, OCP를 위반하지 않는 커맨드 패턴 입니다. 