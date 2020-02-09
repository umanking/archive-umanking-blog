---
layout: post
title: "Thymeleaf Collection 정보 화면에 렌더링하기"
date: 2020-01-19 14:20 +0900
categories: [spring]
tags: [spring]
---

> 모든 소스코드는 **[여기](https://github.com/umanking/blog-code-workspace)🍎**에서 확인 가능합니다.  
> 잘못된 정보나 기술적 피드백은 적극 환영합니다.🙆‍♂️

Thymleaf를 통한 Item 목록을 화면에 렌더링 하는 방법에 대해서 알아보자.  
[thymleaf](https://en.wikipedia.org/wiki/Thymeleaf)는 template view 엔진으로써 Spring MVC에서 모델에 담은 데이터 정보를 View단에 렌더링(뿌리는) 역할을 한다.

## 1. Item 클래스 정의
```java
@Data
@AllArgsConstructor
public class Item {

    private String name;
    private int price;

}
```


## 2. Mvc Controller 작성
REST API를 작성하는 컨트롤러와 다른 점은 `@Controller` 와 메서드 선언부에 `@ResponseBody` 가 없다. 이런 경우에 스프링에서는 ViewNameResolver 라는 놈이 return하는 `item_list`를 `item_list.html` 파일로 매핑해서 해당 화면에 대한 정보를 렌더링한다. 렌더링할 때 model정보에 `items` 라는 이름으로 해당 목록정보를 담았다.
```java
@Controller
public class ItemController {
    
    private static List<Item> ITEMS = Arrays.asList(
            new Item("Chair", 30),
            new Item("Lug", 20),
            new Item("Desk", 50)
        );

    @GetMapping("/item")
    public String findItem(Model model) {
        model.addAttribute("items", ITEMS);
        return "item_list";
    }

}
```


## 3. item_list.html 작성
해당 파일은 `src/main/resources/templates/` 디렉토리 하위에 `item_list.html`파일을 위치시켜야 합니다.
물론 이 path정보를 변경할 수 있지만, 기본 default path가 SpringBoot를 통해서 이루어 지므로 딱히 수정할 것은 없습니다.
```html
<!DOCTYPE html>
<!-- 반드시 네임태그를 달아줍니다. -->
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Item List</title>
</head>
<body>
<h1>Item List</h1>
<table>
    <thead>
    <th>아이템</th>
    <th>가격</th>
    </thead>
    <tbody>
    <tr th:each="item:${items}">
        <td th:text="${item.name}"></td>
        <td th:text="${item.price}"></td>
    </tr>
    </tbody>
</table>

</body>
</html>
```


- thymleaf 네임태그 달기
- th:each 구문 사용: 컨트롤러에서 넘긴 첫번째 파라미터 `model.addAttribute("items", xxx)` `items`가 바로 `${items}`이 부분에 들어오게 됩니다. 


## 4.결과확인
프로젝트를 실행하고 `localhost:8080/item`를 호출하면 Model에 담은 데이터 정보가 화면에 제대로 뿌려진다.


## 정리
오늘은 간단한 thymleaf를 통한 아이템을 화면에 뿌리는 방법에 대해서 알아보았습니다. 이 외에도 자주 사용하는 thymeleaf 태그(?)들과 html내 파일에서 자바스크립트를 활용한 이벤트들을 다루는 방법은 추후에 포스팅 하도록 하겠습니다.
