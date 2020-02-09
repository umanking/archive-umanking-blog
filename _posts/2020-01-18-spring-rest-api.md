---
layout: post
title: "Spring, JPA를 이용한 REST API 만들기"
date: 2020-01-18 10:33 +0900
categories: [spring]
tags: [spring]
---
> 모든 소스코드는 **[여기](https://github.com/umanking/blog-code-workspace)🍎**에서 확인 가능합니다.  
> 잘못된 정보나 기술적 피드백은 적극 환영합니다.🙆‍♂️

오늘은 Spring, JPA, H2를 이용한 RESTful API를 만들겠습니다.  
도메인은 유저들 가입하고, 조회하고, 수정,삭제하는 Account 입니다.

## 1. 요구사항 분석 (Account API)
- Account 생성 - POST /api/account/
- Account 목록 - GET /api/account/
- Account 조회 - GET /api/account/{id}
- Account 수정 - PUT /api/account/{id}
- Account 삭제 - DELETE /api/account/{id} 


## 2. 의존성 추가
pom.xml 파일에 web, lombok외에 spring-data-jpa, h2 모듈을 추가했습니다. 
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
    <version>2.2.2.RELEASE</version>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>1.4.197</version>
    <scope>runtime</scope>
</dependency>
```

## 3. application.properties 설정
```properties
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect

spring.h2.console.enabled=true
spring.h2.console.path=/h2-console
```

## 4. AccountRepsotiroy 추가

```java
@Repository
public interface AccountRepository extends JpaRepository<Account, Long> {
}
```
JpaRepository 인터페이스를 상속받는 AccountRepository를 만들어 줍니다.
> JpaRepository는 인터페이스로 구현체는 SimpleJpaRepository 입니다. 이 말인 즉슨, 기본 CRUD는 이미 구현을 해 놨습니다. 그렇기 때문에 기본 엔티티에 대한 CRUD는 따로 만들어주지 않아도 됩니다. 


## 5. AccountController 추가
```java
@RestController
@RequestMapping("/api")
@AllArgsConstructor
public class AccountApiController {

    private final AccountRepository accountRepository;

    @PostMapping("/account")
    public ResponseEntity<?> saveAccount(@RequestBody Account account) {
        Account savedAccount = accountRepository.save(account);
        return ResponseEntity.ok(savedAccount);
    }

    // Aggregate root
    @GetMapping("/account")
    public List<Account> all() {
        return accountRepository.findAll();
    }

    // Single item
    @GetMapping("/account/{id}")
    public Account getAccount(@PathVariable Long id) {
        return accountRepository.findById(id)
                .orElseThrow(() -> new IllegalArgumentException("illegal argument :" + id));
    }

    @PutMapping("/{id}")
    public void updateAccount(@PathVariable Long id, @RequestBody Account newAccount) {
        accountRepository.findById(id)
                .map(account -> {
                    account.setName(newAccount.getName());
                    account.setPassword(newAccount.getPassword());
                    return accountRepository.save(account);
                })
                .orElseGet(() -> {
                    newAccount.setId(id);
                    return accountRepository.save(newAccount);
                });
    }

    @DeleteMapping("/{id}")
    public void deleteAccount(@PathVariable Long id) {
        accountRepository.deleteById(id);
    }

```
- `@RestController` : `@Controller` + `@ResponseBody` 입니다. 기존의 @Controller는 return 값을 ViewResolverName을 통해서 View단의 이름이랑 매핑했습니다. 하지만, RESTful API는 응답형태를 (보통은 Json) 으로 내보내야 하기 때문에 간편하게 @RestController를 통해서 API관련 코드를 작성할 수 있습니다. 이름에서도 AccountApiController라고 지었습니다. 

- `@AllArgsConstructor`: 롬복의 애노테이션으로, 모든 필드에 대한 클래스 생성자를 자동으로 해줍니다. 이것을 사용하는 이유는 보통은 `@Autowired` 를 통해서 AccountRepository 빈을 주입 받아서 사용하는데, 스프링에서는, IntelliJ에서도 Field Injection은 추천하지 않습니다. 여러 가지 이유가 있지만 Circular dependencies(빈 순환 참조) 이슈가 발생할 수 있기 때문입니다. 나중에 자세한 포스팅은 따로 하겠습니다. 그런 이유에서 필드 주입이 아닌, 생성자 주입을 통해서 빈을 사용합니다. 롬복은 그것을 조금 더 쉽게 만들어 줍니다.



- POST Account 는 @RequestBody를 메서드 파라미터에 붙여주고, save()메서드를 호출합니다. 보통은 저장한 엔티티 그 자체를 return 받아서 ResponseEntity의 body에 담아서 응답을 합니다. 


## 6. PostMan API 테스트

![](/assets/images/post.png)

PostMan으로 POST요청을 날리면 Status 200 과 저장에 성공한 Account정보를 Body에 담아서 보내줍니다. 
요청을 보낼때는 Id에 값을 따로 입력하지 않습니다. 자동으로 @GeneratedValue 를 통해서 자동증감으로 값을 할당해 줍니다. 반면에 ResponseBody에는 해당 Id값이 1로 할당되어 돌아오는 것을 확인했습니다. 

## 정리
지금까지 Spring, JPA, H2 셋팅을 통한, 간단한 RESTful API를 만드는 방법에 대해서 알아봤습니다.
