---
layout: post
title: "Spring, Thymeleaf 를 이용한 List 화면에 뿌리기"
date: 2020-01-19 14:20 +0900
categories: [Spring]
---

# 개요
오늘은 SpringBoot와 Thymleaf를 통한 Item 목록을 화면에 뿌리는 방법에 대해서 알아보겠습니다.


[전체 소스코드는 이곳](https://github.com/umanking/spring-workspace/tree/master/spring-thymleaf)에서 확인 가능합니다.
> [thymleaf](https://en.wikipedia.org/wiki/Thymeleaf)는 template 엔진으로써 Spring MVC에서 모델에 담은 데이터 정보를 View단에 렌더링(뿌리는) 역할을 쉽게 해주는 템플릿 엔진입니다.

# 간단한 Item 클래스 
```java
@Data
@AllArgsConstructor
public class Item {

    private String name;
    private int price;

}
```
요즘 집꾸미기에 푹 빠져있어서, 이케아의 가구들을 담을 수 있는 Item POJO를 예로 들어 보겠습니다.

# ItemController
```java
@Controller
public class ItemController {

    @GetMapping("/item")
    public String findItem(Model model) {
        // 원래는 DB에서 정보를 가지고 와서 화면에 뿌려야 한다.
        // 화면에 뿌릴 때, 검색, 페이징을 고려해야 한다.
        List<Item> items = Arrays.asList(
                new Item("Chair", 30),
                new Item("Lug", 20),
                new Item("Desk", 50)
        );


        model.addAttribute("items", items);

        return "item_list";
    }
}
```
REST API를 작성하는 컨트롤러와 다른 점은 `@Controller` 와 메서드 선언부에 `@ResponseBody` 가 없습니다. 이런 경우에 스프링에서는 ViewNameResolver 라는 놈이 return하는 `item_list`를 `item_list.html` 파일로 매핑해서 해당 화면에 대한 정보를 렌더링합니다. 렌더링할 때 model정보에 `items` 라는 이름으로 해당 목록정보를 담았습니다.

# item_list.html 
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
해당 파일은 `src/main/resources/templates/` 디렉토리 하위에 `item_list.html`파일을 위치시켜야 합니다.
물론 이 path정보를 변경할 수 있지만, 기본 default path가 SpringBoot를 통해서 이루어 지므로 딱히 수정할 것은 없습니다.

### 1. thymleaf 네임태그 달기
html 태그 옆에 `xmlns:th` thymleaf를 사용하기 위해서는 해당 네임태그를 반드시 달아줘야 합니다. 

### 2. th:each 구문 사용
그리고 간단한 table 태그를 만들어서 thymleaf에서 목록을 렌더링하는 each 구문을 사용했습니다. 첫번째 라인에서 item은 각 개별 건들에 대한 alias 라고 생각하면 됩니다. 이 부분은 자기가 원하는 어떤 이름이라도 지을 수가 있습니다. 기왕이면 해당 item에 무엇을 의미하는지 정확하게 명시하는게 좋을 것 같습니다. 
컨트롤러에서 넘긴 첫번째 파라미터 `model.addAttribute("items", xxx)` `items`가 바로 `${items}`이 부분에 들어오게 됩니다. 


# 결과
프로젝트를 실행하고 `localhost:8080/item`를 호출하면 다음과 같이 나옵니다.
```
Item List
아이템	가격
Chair	30
Lug	20
Desk	50
```


# 맺음말
오늘은 간단한 thymleaf를 통한 아이템을 화면에 뿌리는 방법에 대해서 알아보았습니다. 이 외에도 자주 사용하는 thymeleaf 태그(?)들과 html내 파일에서 자바스크립트를 활용한 이벤트들을 다루는 방법은 추후에 포스팅 하도록 하겠습니다.