---
layout: post
title: "[Spring] SpringApplication를 통한 코딩"
date: 2019-04-12 09:11:51
categories: [Spring]
tags: [spring]
redirect_from: 
- 2019/04/12/spring-springapplication/
---
## SpringBoot - SpringApplication ( applicationContext 등록 이전의 이벤트 Listener 등록 하기)

> applicationContext가 등록되기 전에 발생한 이벤트를 Listen하는 리스너를 등록하려면 어떻게 해야할까? 





```java
@Component
public class SampleListener implements ApplicationListener<ApplicationStartingEvent> {

    @Override
    public void onApplicationEvent(ApplicationStartingEvent applicationStartingEvent) {
        System.out.println("===================");
        System.out.println("application start ");
        System.out.println("===================");
    }
}
```

SampleListner를 만들고, `ApplicationListenr`를 구현한다. Generic으로 발생할 이벤트를 명시해 준다. 여기에서는 `ApplicationStartingEvent` 클래스는 스프링부트에서 제공하는 Event 클래스이다. 문제는 해당 이벤트가 applicationContext가 띄기전에 발생하기 때문에, 해당 리스너가 캐치하지 못하는 문제가 발생한다. 



### 해결책

```java
@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
		
    //  SpringApplication.run(DemoApplication.class, args);

        SpringApplication springApplication = new SpringApplication(DemoApplication.class);
        springApplication.addListeners(new SampleListner());
        springApplication.run(args);

    }

}
```

기존의 SpringBoot 애플리케이션을 만든 Main메서드를 보면, `SpringApplication.run()` 이라는 static 메서드를 통해서 만든다. 그렇게 되면, SpringApplication을 커스터마이징 할 수 없기 때문에, 새로운 SpringApplication 객체를 만들고 리스너를 추가한다. 이렇게 되면 이전에 작성했던 SampleListner는 빈으로 등록할 필요가 없다. (어차피 해당 이벤트가 applicationContext가 등록되기 전에 발생한 이벤트이니까) 



