---
layout: post
title: SpringBoot Validation(JSR-303)
date: 2019-12-12 21:19 +0900
categories: [Spring]
---

## 1. Overview
**Spring Boot를 통한 REST API 샘플예제를 통해서, 도메인 객체를 validation 하는 방법을 알아보자.**
<u>기본 전제 조건은, 유저가 넘기는 모든 데이터는 신뢰할수 없다.</u> 그렇기 때문에 모든 데이터를 반드시 검증해야 한다.
대게 validation은 Post요청시, @RequestBody에 담길 객체에 대해서 검증하는게 일반적이다.

## 2. User Domain 
```java
@Entity
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    @NotBlank(message = "name is required")
    private String name;

    @NotBlank(message = "email is required")
    private String email;

}
```

## 3. Implement RestControllor
```java
@RestController
public class UserController {

    @PostMapping("/users")
    public ResponseEntity<String> addUser(@Valid @RequestBody User user) {
        return ResponseEntity.ok("user is valid");
    }
}
```

POST 요청시, 클라이언트가 넘기는 값들은 기본적으로 신뢰할 수 없다. 그렇기 때문에 validation을 반드시 해야 한다. `@Valid` 어노테이션만 붙이면, Spring Boot는 JSR303 구현체(Hibernate Validator)를 사용해서 argument값을 validate를 진행한다. 


## 4. Exception Handling
```java
@ResponseStatus(HttpStatus.BAD_REQUEST)
@ExceptionHandler(MethodArgumentNotValidException.class)
public Map<String, String> handleValidationExceptions(MethodArgumentNotValidException ex) {
    Map<String, String> errors = new HashMap<>();
    ex.getBindingResult().getAllErrors().forEach(
            (error) -> {
                String fieldName = ((FieldError) error).getField();
                String errorMessage = error.getDefaultMessage();
                errors.put(fieldName, errorMessage);
            }
    );

    return errors;
}
```
`@ExceptionHandler`를 통해서 `MethodArgumentNotValidException`를 핸들링 한다. errors map에 담아서 반환하면, `@RestController`의 `@ResponseBody`를 통해서 JSON 값으로 리턴된다. 

## 5. TEST
PostMan으로 `POST /users`의 `name`과 `email` 필드 없이 빈 Json을 넘기면 다음과 같이 나온다.

```json
{
    "name": "name is required",
    "email": "email is required"
}

```

## 생각해볼 것들
- [https://beanvalidation.org/2.0/](https://beanvalidation.org/2.0/) - `@NotBlank` 외의 Bean validation spec
- 단순 field validation은 JSR303을 통해서 간단하게 할 수 있다. 하지만, 비즈니스 로직이 필요한 경우라면?
- POST 요청(도메인 객체)만을 검증하나? 아니면 **RequestParam, @Pathvariable 도 검증해야 하나?** (아시는 분들은 댓글좀)
