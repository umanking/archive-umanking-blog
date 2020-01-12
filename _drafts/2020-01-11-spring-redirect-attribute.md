---
layout: post
title: ". spring-redirect-attribute"
date: 2020-01-11 00:09 +0900
categories: [Spring]
---
# RedirectAttribute 에 대해서 알아 보겠습니다.
이번 시간에는 Spring MVC 에서 특정 페이지로 `redirect`를 보낼 때 임시적으로 값을 저장할 수 있는 RedirectAttribute에 대해서 알아 보겠습니다.

## 예제 

```java
@Data
public class Account {

    private Long id;
    private String name;
    private int age;
}
```
간단한 Account 클래스를 정의합니다.

```java
@Controller
public class AccountController {

    @PostMapping("/account")
    public String saveAccount(@RequestBody Account account, RedirectAttributes redirectAttributes) {
        // save account 작업 

        redirectAttributes
                .addAttribute("id", account.getId())
                .addFlashAttribute("message", "메세지는..");

        return "redirect:/account/{id}";
    }
}   
```
account를 POST방식으로 DB에 저장하고, 저장한 유저에 대한 페이지로 이동한다고 가정해 보겠습니다. 
리다이렉트를 account의 id값을 url로 전달해야 할때 ?
이럴때, RedirectAttributes를 이용하면 가능합니다. 


- 그렇다면 addFlashAttribute는 무엇일까요? 
- 그냥 addAttribute와 무엇이 다를까요? 

attFlashAttribute는 map에 임시적인 맵에 담겨있고, redirect 이후에는 사라지는 값입니다. 
브라우저의 header에 저장하는 것이 아니고, 서버쪽의 세션에 저장했다가 바로 사라지는 놈입니다. 

```java
    @GetMapping("/account/{id}")
    public String findAccount(Model model, @PathVariable Long id) {
        model.addAttribute("id", id);

        return "account";
    }
}
```

