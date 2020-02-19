---
layout: post
title: "[Spring] MVC 만들어 보기"
date: 2019-04-17 09:21:01
categories: [Spring]
tags: [Spring]
redirect_from: 
- 2019/04/17/spring-mvc-introduce-1/
- spring/spring-mvc-introduce-1/
---

> Spring MVC 에 대해서 알아보자


MVC는 Model, View, Controller로 구성되어 있다. 

- Model : 데이터를 담을 도메인 객체, DTO
- View: 화면에 보여주는 부분, jsp, thymeleaf, React … 
- Controller: 사용자의 입력을 받아서 모델 데이터를 검증/변경 하고, View단으로 전달하는 역할을 한다. 



### 예제코드

Boot 기반의 프로젝트를 만든다.

```xml
		<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        
	    <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
```

`web`, `thymeleaf` , `lombok` 를 추가한다. 



```java
// Event 도메인 클래스
@Getter
@Setter
@AllArgsConstructor
@NoArgsConstructor
@Builder
public class Event {
    
    private String name;
    private int limitOfEnrollment;
    private LocalDateTime startDateTime;
    private LocalDateTime endDateTime;
}

```



```java
@Service
public class EventService {

    public List<Event> getEvents() {
        Event event1 = Event.builder()
                .name("effective java")
                .limitOfEnrollment(3)
                .startDateTime(LocalDateTime.of(2019, 4, 17, 10, 0))
                .endDateTime(LocalDateTime.of(2019, 4, 17, 12, 0))
                .build();

        Event event2 = Event.builder()
                .name("spring boot")
                .limitOfEnrollment(3)
                .startDateTime(LocalDateTime.of(2019, 6, 17, 10, 0))
                .endDateTime(LocalDateTime.of(2019, 6, 17, 12, 0))
                .build();

        List<Event> list = new ArrayList<>();
        list.add(event1);
        list.add(event2);
        return list;
    }
```

EventService에는 2개의 이벤트를 미리 넣어놓고(사실은 데이터베이서에 접근해서 데이터를 가지고 와야함), 이벤트의 리스트를 가지고 오는 코드를 작성한다. 



```java
@Controller
public class EventController {

    @Autowired
    private EventService eventService;

    @GetMapping("/events")
    public String events(Model model) {
        model.addAttribute("events", eventService.getEvents());
        return "events";
    }
}
```

`/events` 라는 경로로 요청이 들어왔을 때, 모델에 eventService에서 가지고 온 이벤트 목록을 `events` 모델에 담는다. `return "events"; ` 이 부분은 events 라는 뷰(여기서는 thymeleaf)로 전달하겠다는 의미. 



`src/main/resources/templates` 하위에 `events.html` (컨트롤러에서 return하는 view name과 동일하게) 파일을 만든다.



```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
</body>
</html>
```

`xmlns:th="http://www.thymeleaf.org"` 타임리프를 사용할 xml name space를 지정해준다. 



```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<table>
    <tr>
        <th>이름</th>
        <th>참가인원</th>
        <th>시작</th>
        <th>종료</th>
    </tr>
    <tr th:each="event: ${events}">
        <td th:text="${event.name}"></td>
        <td th:text="${event.limitOfEnrollment}"></td>
        <td th:text="${event.startDateTime}"></td>
        <td th:text="${event.endDateTime}"></td>
    </tr>
</table>

</body>
</html>
```

타임리프의 each구문을 사용해서 events 모델로 전달받은 객체를 event로 명명하고, event.name 이런식으로 접근해서 사용한다. 

애플리케이션을 띄우고 `localhost:8080/events` 로 요청하면 원하는 데이터가 화면에 잘 보여진다. 

