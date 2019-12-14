---
layout: post
title: Spring MVC 궁금한 거
date: 2019-11-02 21:14 +0900
redirect_from: 
- 2019/11/02/spring-mvc/
---
최근 진행했던 프로젝트는 `Spring MVC`로 되어있는 프로젝트로, View단은 thymeleaf를 사용했다. 실제 진행하면서 궁금했던 점들이 생겼다. 궁금했던 점들을 뚜렷한 해결을 제시하지는 못한다. 다만 그 문제들에 대해서 생각해 보고자 한다.

```java
@Data
public class DailyMission {
    
    private String dailyMission1;
    private String dailyMission2;
    private String dailyMission3;

    private int status;
}
```

다음과 같은 유저가 DailyMission을 완료해야만 하는 비즈니스 로직이라고 가정해 보자. 보통의 Spring MVC에서 DailyMission에 대한 정보를 모델에 담아서 뷰단에 넘기는 로직은 다음과 같다. 

```java
@Controller
@RequestMapping(value = "/dailyMission")
public class DailyMissionViewController {

    @Autowired
    private DailyMissionService dailyMissionService;

    @GetMapping(value ="")
    public String list(Model model, HttpServletRequest req, HttpServletResponse resp){
        
        // Session에서 유저 정보를 가져와서, 
        List<DailyMissionDto> dailyMissionList = dailyMissionService.findDailyMissionList(userId);
        model.addAttribute("dailyMissionDto", dailyMissionList);
    }
}
```
서비스단에서 작성한 DailyMission 목록을 받아와서 model정보에 담아서 넘긴다. 
이렇게 간단한 예제에서는 심플하지만, 

## Q1. 만약에 dailyMission1, dailyMission2 만 완료한지를 체크하는 로직이 있을때 어떻게 처리를 해야할까?
1. 위의 로직은 Model정보에 반환하는게 아니라, `DailyMissionDto`를 만들어서 반환하기 때문에 해당 Dto에서 `isCompleteDailyMissionOneAndTwo()`를 작성하면 된다. 이렇게 하면 장점은 기존의 `Controller`의 코드는 손댈필요 없이 Dto 클래스에서 해당 메서드만 추가해주면 된다. 이게 바로 OCP인가? 

사실, 이렇게 model정보에 public 메서드를 넘겨도 View단에서 사용할 수 있는지 몰랐다. 하지만 소스상에서 엥? 메서드를 호출하길래 따라가 보니 모델 객체에 담아서 사용할 수 있다는 것을 알게 되었다. 

2. isCompleteDailyMissionOneAndTwo() 조건을 충족하는 쿼리를 반드는 것이다. 이 경우에, Repository에 쿼리를 추가하고, Controller단에 `model.addAttribute("isCompleteDailyMissionOneAndTwo", dailyMissionService.isCompleteDailyMissionOneAndTwo());` 이런 코드를 삽입해야 할 것이다. 일단 select 쿼리가 늘어났고, 불필요한 코드도 증가했고, 해당 체크하는 쿼리를 여러 도메인에서 많이 사용하면 (적어도 3번 이상 ) 모를까. 코드 중복이 많이 발생한다고 생각된다. 

## Q2. 메서드, 변수명은 어떻게 가져가야 하나? 
이건 서버단 로직을 작성하면서 매번 마주했던 문제들(?) 인데, 결론은 나지 않은 상황. 일단은 어떤 조직에 속해있던 회사의 컨벤션을 따르는 게 가장 중요한 첫번째 이고, 두번째는 개인 프로젝트 혹은 새로운 프로젝트를 들어갈 때 어떻게 네이밍을 할지가 매우 많이 궁금하다. 난 아직 잘 모르기 때문에..

예를 들어 Spring Data Jpa에서 제공하는 SimpleCRUDRepository에서는 findById, findAll, findDailyMissionByUserId 와 같은 `쿼리 메서드`를 제공한다. 처음부터 여기에 익숙해서 이게 뭔가 표준이라고 생각했었는데, 지금 있는 프로젝트에서는 DB에 저장할때 진짜 insert(), select(), update() 이라는 쿼리할때 쓰는 키워드를 메서드명으로 사용한다. 

또한, 특정 리스트를 조회할때 findDailyMissionList vs findDailyMissions 어떤게 더 직관적이고 맞을까? 궁금했다. 🤔
도메인이 아닌, Dto로 응답객체를 받을 때도 dailyMissionDto vs dailyMissionDto 어떤게 더 맞을까 ?

