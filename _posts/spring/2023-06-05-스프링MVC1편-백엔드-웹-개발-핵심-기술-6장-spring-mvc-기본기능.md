---
layout: post
title: 스프링MVC1편 백엔드 웹 개발 핵심 기술 6장 spring mvc 기본 기능
tags: [spring, note]
skills: [java, spring]
author: mainmethod0126
---

## 프로젝트 생성

### JAR or WAR

- JAR : 항상 내장 서버 사용, webapp 경로도 사용하지 않음, 내장 서버 사용에 최적화되어 있음
- WAR : 외부 서버에 배포하는 목적 (톰캣이 따로 동작할 때)

### Welcome 페이지

Spring boot에서 `jar`를 사용하면 `/resources/static/index.html` 파일을 Welcome페이지로 처리함

---

## 로깅

이제 `system.out.println()` 안씀

### 로깅 라이브러리

- SLF4J : 여러 로그 라이브러리를 사용하기 편리하게 해주는 인터페이스, Logback, log4j, log4j2 등의 여러 로깅 라이브러리를 편리하게 쓸 수 있게 해줌
- Logback : 로깅 라이브러리

보통 Spring boot 기본 제공하는 `Logback` 사용

#### 로그 선언

`private Logger log = LoggerFactory.getLogger(getClass());`
`private static final Logger log = LoggerFactory.getLogger(Xxx.class)`
`@Slf4j : 롬복 사용 가능` 보통은 이거 많이 씀

> **@Slf4j** 사용 시 자동으로 `private static final org.slf4j.Logger log =
org.slf4j.LoggerFactory.getLogger(RequestHeaderController.class);` 코드가 작성됨
> 컴파일 타임 어노테이션

#### 외부 설정 파일로 로그 레밸 설정

`application.properties` 파일에 아래와 같이 정의하여 새로 빌드하지 않고 로깅 레벨을 지정할 수 있음

```yaml

#전체 로그 레벨 설정(기본 info)
logging.level.root=info

#hello.springmvc 패키지와 그 하위 로그 레벨 설정
logging.level.hello.springmvc=debug

```

#### 올바른 로그 사용법

`log.debug("data="+data)`
로그 출력 레벨을 info로 설정해도 해당 코드에 있는 "data="+data가 실제 실행이 되어 버린다.
결과적으로 문자 더하기 연산이 발생한다.

`log.debug("data={}", data)`
로그 출력 레벨을 info로 설정하면 아무일도 발생하지 않는다. 따라서 앞과 같은 의미없는 연산이
발생하지 않는다 불필요한 연산을 막기 위해 아래와 같은 방식을 사용해야한다.

---

## 요청 매핑

### MappingController

**`@Controller`**

- 반환값이 `String` 이면 뷰 이름으로 인식됩니다. **뷰 이름을 찾고 뷰가 랜더링** 된다
- `@RestController` : 반환 값으로 뷰를 찾는 것이 아니라 **HTTP 메세지 바디에 바로 입력** `@ResponseBody` 와 연관이 있다

**`@RequestMapping("/hello-basic")`**

- `/hello-basic` URL 호출이 오면 이 메서드가 실행되도록 매핑한다.
- 대부분의 속성을 `배열[]`로 제공하므로 다중 설정이 가능하다. `{"/hello-basic", "/hello-go"}`

> 잠깐! URL 요청: /hello-basic , /hello-basic/ 와 같이 뒤에 '/' 가 붙은 상태로 끝나도 매핑을 시켜준다

### HTTP 메서드

@RequestMapping 에 method 속성으로 HTTP 메서드를 지정하지 않으면 HTTP 메서드와 무관하게
호출된다. 모두 허용 GET, HEAD, POST, PUT, PATCH, DELET

`Method Not Allowed` 라는 에러가 발생하는데 허용되지 않은 Method라는 의미

### HTTP 메서드 매핑 축약

- 축약 전 : `@RequestMapping(value = "/mapping-get-v1", method = RequestMethod.GET)`
- 축약 후 : `@GetMapping(value = "/mapping-get-v1")` **그냥 이 방식 쓰면 됩니다**

### PathVariable(경로 변수) 사용

요즘은 URL 에 식별자는 넣는게 유행
`/mapping/userA`
`/users/1`
이러한 식별자를 코드상에서 변수로 접근할 수 있게 해줌

```java

/**
 * PathVariable 사용
 * 변수명이 같으면 생략 가능
 * @PathVariable("userId") String userId -> @PathVariable userId
 */
@GetMapping("/mapping/{userId}")
public String mappingPath(@PathVariable("userId") String data) {
    log.info("mappingPath userId={}", data);
    return "ok";
}

```

`@RequestMapping` 은 URL 경로를 템플릿화 할 수 있는데, @PathVariable 을 사용하면 매칭 되는 부분을
편리하게 조회할 수 있다.

**`@PathVariable` 의 이름과 파라미터 이름이 같으면 생략할 수 있다**

### PathVariable 사용 - 다중

```java
/**
 * PathVariable 사용 다중
 */
@GetMapping("/mapping/users/{userId}/orders/{orderId}")
public String mappingPath(@PathVariable String userId, @PathVariable Long
orderId) {
 log.info("mappingPath userId={}, orderId={}", userId, orderId);
 return "ok";
}
```

### 특정 파라미터 조건 매핑

그냥 패스해버림 안씀

### 특정 헤더 조건 매핑

그냥 패스해버림 안씀

### 미디어 타입 조건 매핑 - HTTP 요청 Content-Type, consume

이건 많이 쓴다

Request header의 Content-Type에 따라서 호출 여부 결정

```java

/**
 * Content-Type 헤더 기반 추가 매핑 Media Type
 * consumes="application/json"
 * consumes="!application/json"
 * consumes="application/*"
 * consumes="*\/*"
 * MediaType.APPLICATION_JSON_VALUE
 */
@PostMapping(value = "/mapping-consume", consumes = "application/json")
public String mappingConsumes() {
 log.info("mappingConsumes");
 return "ok";
}

```

`consumes(소비하다)` 컨트롤러가 요청을 소비하는데, `"Content-Type : application/json"` 인 Request 만 소비하겠다! 라는 의미

### 미디어 타입 조건 매핑 - HTTP 요청 Accept, produce

```java
/**
 * Accept 헤더 기반 Media Type
 * produces = "text/html"
 * produces = "!text/html"
 * produces = "text/*"
 * produces = "*\/*"
 */
@PostMapping(value = "/mapping-produce", produces = "text/html")
public String mappingProduces() {
 log.info("mappingProduces");
 return "ok";
}
```

`produces(생산하다)` 컨트롤러가 응답을 생성하는데, 요청하는 클라이언트가 `"Accept : application/json"` 이라고 하면 API가 응답하는 형식이 `application/json` 인 API를 호출하겠다는 의미!

---

## HTTP 요청 - 기본, 헤더 조회

`HttpServletRequest`
`HttpServletResponse`
`HttpMethod`

- HTTP 메서드를 조회한다. `org.springframework.http.HttpMethod`
  `Locale`
- Locale 정보를 조회한다.
  `@RequestHeader MultiValueMap<String, String> headerMap`
- 모든 HTTP 헤더를 MultiValueMap 형식으로 조회한다.

`@RequestHeader("host") String host`

- 특정 HTTP 헤더를 조회한다.
- 속성
  - 필수 값 여부: `required`
  - 기본 값 속성: `defaultValue`

`@CookieValue(value = "myCookie", required = false) String cookie`

- 특정 쿠키를 조회한다.
- 속성
  - 필수 값 여부: `required`
  - 기본 값: `defaultValue`

`MultiValueMap`

- MAP과 유사한데, 하나의 키에 여러 값을 받을 수 있다.
- HTTP header, HTTP 쿼리 파라미터와 같이 하나의 키에 여러 값을 받을 때 사용한다.
  - keyA=value1&keyA=value2

```java

MultiValueMap<String, String> map = new LinkedMultiValueMap();
map.add("keyA", "value1");
map.add("keyA", "value2");
//[value1,value2]
List<String> values = map.get("keyA");

```

요런식으로 리스트로 꺼낼 수 있음

---

## HTTP 요청 파라미터 - 쿼리 파라미터, HTML Form

클라이언트에서 서버로 요청 데이터를 전달할 때는 주로 다음 3가지 방법을 사용한다.

**GET - 쿼리 파라미터**

- /url?username=hello&age=20
- 메시지 바디 없이, URL의 쿼리 파라미터에 데이터를 포함해서 전달
- 예) 검색, 필터, 페이징등에서 많이 사용하는 방식

**POST - HTML Form**

- content-type: application/x-www-form-urlencoded
- 메시지 바디에 쿼리 파리미터 형식으로 전달 username=hello&age=20
- 예) 회원 가입, 상품 주문, HTML Form 사용

**HTTP message body에 데이터를 직접 담아서 요청**

- HTTP API에서 주로 사용, JSON, XML, TEXT
- 데이터 형식은 주로 JSON 사용
- POST, PUT, PATCH

### HTTP 요청 파라미터 - @RequestParam

```java
/**
 * @RequestParam 사용
 * - 파라미터 이름으로 바인딩
 * @ResponseBody 추가
 * - View 조회를 무시하고, HTTP message body에 직접 해당 내용 입력
 */
@ResponseBody
@RequestMapping("/request-param-v2")
public String requestParamV2(
    @RequestParam("username") String memberName,
    @RequestParam("age") int memberAge) {
    log.info("username={}, age={}", memberName, memberAge);
    return "ok";
}
```

### HTTP 요청 파라미터 - @ModelAttribute

@RequestParm -> Dto Mapping 해주는 녀석

```java

package hello.springmvc.basic;
import lombok.Data;

@Data
public class HelloData {
    private String username;
    private int age;
}

```

- 롬복 `@Data`
  `@Getter` , `@Setter` , `@ToString` , `@EqualsAndHashCode` , `@RequiredArgsConstructor` 를
  자동으로 적용해준다

### @ModelAttribute 적용 - modelAttributeV1

```java
/**
 * @ModelAttribute 사용
 * 참고: model.addAttribute(helloData) 코드도 함께 자동 적용됨, 뒤에 model을 설명할 때
자세히 설명
 */
@ResponseBody
@RequestMapping("/model-attribute-v1")
public String modelAttributeV1(@ModelAttribute HelloData helloData) {
    log.info("username={}, age={}", helloData.getUsername(),
    helloData.getAge());
    return "ok";
}
```

스프링MVC는 `@ModelAttribute` 가 있으면 다음을 실행한다.

1. HelloData 객체를 생성한다.
2. 요청 파라미터의 이름으로 HelloData 객체의 프로퍼티를 찾는다. 그리고 해당 프로퍼티의 setter를
   호출해서 파라미터의 값을 입력(바인딩) 한다.
3. 예) 파라미터 이름이 username 이면 setUsername() 메서드를 찾아서 호출하면서 값을 입력한다.

**요약 : Parameter 와 동일한 이름의 필드의 Setter 호출**

`@ModelAttribute` 생략도 가능

```java
/**
 * @ModelAttribute 생략 가능
 * String, int 같은 단순 타입 = @RequestParam
 * argument resolver 로 지정해둔 타입 외 = @ModelAttribute
 */
@ResponseBody
@RequestMapping("/model-attribute-v2")
public String modelAttributeV2(HelloData helloData) {
    log.info("username={}, age={}", helloData.getUsername(),
    helloData.getAge());
    return "ok";
}

```

스프링은 해당 생략시 다음과 같은 규칙을 적용한다.
`String` , `int` , `Integer` 같은 단순 타입 = `@RequestParam`
나머지 = `@ModelAttribute` (argument resolver 로 지정해둔 타입 외)
