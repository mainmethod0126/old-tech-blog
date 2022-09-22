---
layout: post
title: Spring MVC
tags: [java, spring, springboot, mvc]
author: mainmethod0126
excerpt_separator: <!--more-->
---

MVC 패턴을 추상적으로 사용하였기에 MVC에 대해 간략하게 짚고 넘어가려합니다.

<!--more-->


# MVC(Model View Controller) 패턴
![spring-mvc.png](/assets/img/feature-img/spring/spring-mvc.png)

## Controller
Controller 란 HTTP 요청을 받아서 **파라미터를 검증**하고, **비즈니스 로직을 실행**하며, 뷰에 전달할 **결과 데이터를 조회 및 가공하여 모델에 담아주는** 역할을 말합니다.
### Controller의 역할
- 요청의 파라미터 검증
- 비즈니스 로직 실행
- 모델에 결과를 담아 View에 전달

## Model
Model이란 **뷰에 출력할 데이터를 담아두는 역할**이며 뷰가 필요한 데이터를 모두 모델에 담아서 전달해주는 덕분에 뷰는 비즈니스 로직이나 데이터 접근을 몰라도 되고, 화면을 렌더링 하는 일에 집중할 수 있습니다.

## View
**모델에 담겨있는 데이터를 사용해서 화면을 그리는 역할**입니다.

# Spring Boot에서의 MVC
## Controller
디자인 패턴 MVC에서의 Controller는 **비즈니스 로직을 전담**하지만, Spring Boot에서는 **Service 클래스를 별도로** 두어 비즈니스 로직을 **Service 클래스가 전담**하도록 하고 Controller는 단순히 이 Service 클래스 객체를 통해 **비즈니스 로직을 실행하라는 명령을 진행**하고 **결과값을 반환**해주기만 합니다.
Spring Boot에서는 이 **Controller 역할**을 부여하기 위하여 Controller로 사용될 클래스에 **@Controller 또는 @RestController 어노테이션을 적용**합니다.
일반적으로는 @Controller를 사용하기보단 @RestController를 사용하며 둘의 차이를 간단하게 알아보고 가겠습니다.
### @Controller 와 @RestController의 차이
#### @Controller
Controller 어노테이션은 전형적인 Spring MVC에서 **View를 반환**하기 위한 어노테이션입니다.
~~~java
@Controller
public class UserController {

    @Autowired
    private UserService userService;


    @GetMapping(value = "/users/view")
    public String view(Model model, @RequestParam("name") String name) {
        User user = userService.findUser(name);
        model.addAttribute("user", user);
        return "/users/view";
    }
}
~~~
#### @RestController
RestController 어노테이션 이름 부터 RestAPI를 지원할 것 같은 느낌이 물씬 풍깁니다.
그렇기 때문에 @Controller와 달리 **JSON을 반환**합니다.
이 RestController을 분석해보면 @Controller와 달리 단순하게 **@ResponseBody** 라는 어노테이션이 하나 더 추가되있는걸 확인할 수 있습니다.
이 **@ResponseBody** 어노테이션이 JSON을 반환할 수 있게 **HttpMessageConverter** 라는 것을 동작시켜줍니다. **HttpMessageConverter** 는 반환되는 **객체의 타입(Type)에 따라** 사전에 등록된 **MessageConverter** 중 알맞은 것을 찾아서 동작시키며, **객체를 JSON 형식으로 직렬화**하여 Response 시켜줍니다.

#### 
~~~java
@RestController
public class UserController {

    @Autowired
    private UserService userService;

    @GetMapping(value = "/users")
    public ResponseEntity<User> findUser(@RequestParam("name") String name) {
        return ResponseEntity.ok(userService.findUser(name));
    }
}
~~~
## Model
최근 대세는 **RestAPI**이기 때문에 대부분 View와 Model을 사용하지 않고 HTTP Response에 바로 **JSON을 반환**하는 **@RestController** 를 사용합니다.
하지만 만약 View 반환해야한다면 아래와 같이 **model.addAttribute()을** 이용할 수 있습니다.
~~~java
@Controller
public class UserController {

    @Autowired
    private UserService userService;


    @GetMapping(value = "/users/view")
    public String view(Model model, @RequestParam("name") String name) {
        User user = userService.findUser(name);
        model.addAttribute("user", user);
        return "/users/view";
    }
}
~~~
## View
Spring MVC에서 View는 **JSP**를 이용하거나 최근 대세에 따라 프론트 엔드, 백엔드 방식으로 View.js, React.js 등을 이용한 **별도의 View(프론트 엔드)단**을 이용합니다.
