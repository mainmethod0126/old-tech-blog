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

---

## HTTP 요청 메세지 - 단순 텍스트

**HTTP message body**에 데이터를 직접 담아서 요청
  - HTTP API에서 주로 사용, JSON, XML, TEXT
  - 데이터 형식은 주로 JSON 사용
  - POST, PUT, PATCH

HTTP 메세지 바디에 그냥 ROW로 넣어서 요청되는 Request Body의 경우 **@RequestParam ,
@ModelAttribute** 를 사용할 수 없음!!
(HTML Form 은 사용 가능!)

### Input, Output 스트림, Reader

```java
/**
* InputStream(Reader): HTTP 요청 메시지 바디의 내용을 직접 조회
* OutputStream(Writer): HTTP 응답 메시지의 바디에 직접 결과 출력
*/
@PostMapping("/request-body-string-v2")
public void requestBodyStringV2(InputStream inputStream, Writer responseWriter) throws IOException {
  String messageBody = StreamUtils.copyToString(inputStream,StandardCharsets.UTF_8);
  log.info("messageBody={}", messageBody);
  responseWriter.write("ok");
}
```

### HttpEntity

```java

/**
* HttpEntity: HTTP header, body 정보를 편리하게 조회
* - 메시지 바디 정보를 직접 조회(@RequestParam X, @ModelAttribute X)
* - HttpMessageConverter 사용 -> StringHttpMessageConverter 적용
*
* 응답에서도 HttpEntity 사용 가능
* - 메시지 바디 정보 직접 반환(view 조회X)
* - HttpMessageConverter 사용 -> StringHttpMessageConverter 적용
*/
@PostMapping("/request-body-string-v3")
public HttpEntity<String> requestBodyStringV3(HttpEntity<String> httpEntity) {
  String messageBody = httpEntity.getBody();
  log.info("messageBody={}", messageBody);
  return new HttpEntity<>("ok");
}

```

스프링 MVC는 다음 파라미터를 지원한다.

**HttpEntity**: HTTP header, body 정보를 편리하게 조회

- 메시지 바디 정보를 직접 조회
- 요청 파라미터를 조회하는 기능과 관계 없음 @RequestParam X, @ModelAttribute X

**HttpEntity**는 응답에도 사용 가능

- 메시지 바디 정보 직접 반환
- 헤더 정보 포함 가능
- view 조회X

**HttpEntity** 를 상속받은 다음 객체들도 같은 기능을 제공한다.

**RequestEntity**

- HttpMethod, url 정보가 추가, 요청에서 사용

**ResponseEntity**

- HTTP 상태 코드 설정 가능, 응답에서 사용
- return new ResponseEntity<String>("Hello World", responseHeaders,
HttpStatus.CREATED)

### @RequestBody

```java
/**
* @RequestBody
* - 메시지 바디 정보를 직접 조회(@RequestParam X, @ModelAttribute X)
* - HttpMessageConverter 사용 -> StringHttpMessageConverter 적용
*
* @ResponseBody
* - 메시지 바디 정보 직접 반환(view 조회X)
* - HttpMessageConverter 사용 -> StringHttpMessageConverter 적용
*/
@ResponseBody
@PostMapping("/request-body-string-v4")
public String requestBodyStringV4(@RequestBody String messageBody) {
  log.info("messageBody={}", messageBody);
  return "ok";
}
```

@RequestBody 를 사용하면 HTTP 메시지 바디 정보를 편리하게 조회할 수 있다. 참고로 헤더 정보가필요하다면 HttpEntity 를 사용하거나 @RequestHeader 를 사용하면 된다

**이렇게 메시지 바디를 직접 조회하는 기능은 요청 파라미터를 조회하는 @RequestParam ,
@ModelAttribute 와는 전혀 관계가 없다**

### requestBodyJsonV2 - @RequestBody 문자 변환

```java
/**
* @RequestBody
* HttpMessageConverter 사용 -> StringHttpMessageConverter 적용
*
* @ResponseBody
* - 모든 메서드에 @ResponseBody 적용
* - 메시지 바디 정보 직접 반환(view 조회X)
* - HttpMessageConverter 사용 -> StringHttpMessageConverter 적용
*/
@ResponseBody
@PostMapping("/request-body-json-v2")
public String requestBodyJsonV2(@RequestBody String messageBody) throws IOException {
  HelloData data = objectMapper.readValue(messageBody, HelloData.class);
  log.info("username={}, age={}", data.getUsername(), data.getAge());
  return "ok";
}
```

문자로 된 JSON 데이터인 messageBody 를 objectMapper 를 통해서 자바 객체로 변환한다.

**불~편 @ModelAttribute처럼 한번에 객체로 변환할 수는 없을까?**

### requestBodyJsonV3 - @RequestBody 객체 변환

```java
/**
* @RequestBody 생략 불가능(@ModelAttribute 가 적용되어 버림)
* HttpMessageConverter 사용 -> MappingJackson2HttpMessageConverter (contenttype: application/json)
*
*/
@ResponseBody
@PostMapping("/request-body-json-v3")
public String requestBodyJsonV3(@RequestBody HelloData data) {
  log.info("username={}, age={}", data.getUsername(), data.getAge());
  return "ok";
}
```

#### @RequestBody 객체 파라미터

- `@RequestBody HelloData data`
- `@RequestBody` 에 직접 만든 객체를 지정할 수 있다.


`HttpEntity` , `@RequestBody` 를 사용하면 **`HTTP 메시지 컨버터가`** HTTP 메시지 바디의 내용을 우리가
원하는 문자나 객체 등으로 변환해준다.

**@RequestBody는 생략 불가능**
**불가능한 이유 : 생략 시 @ModelAttribute 가 적용됨**

> 잠깐! 
> HTTP 요청시에 content-type이 application/json인지 꼭! 확인해야 한다. 그래야 JSON을 처리할 수
있는 HTTP 메시지 컨버터가 실행된다

---

## HTTP 응답 - 정적 리소스, 뷰 템플릿

스프링(서버)에서 응답 데이터를 만드는 방법은 크게 3가지

**정적 리소스**

- 예) 웹 브라우저에 정적인 HTML, css, js를 제공할 때는, 정적 리소스를 사용한다.

**뷰 템플릿 사용**

- 예) 웹 브라우저에 동적인 HTML을 제공할 때는 뷰 템플릿을 사용한다.

**HTTP 메시지 사용**

- HTTP API를 제공하는 경우에는 HTML이 아니라 데이터를 전달해야 하므로, HTTP 메시지 바디에
  JSON 같은 형식으로 데이터를 실어 보낸다

### 정적 리소스

스프링 부트는 클래스패스의 다음 디렉토리에 있는 정적 리소스를 제공한다.
`/static` , `/public` , `/resources` , `/META-INF/resources`

`src/main/resources` 는 리소스를 보관하는 곳이고, 또 클래스패스의 시작 경로이다.
따라서 다음 디렉토리에 리소스를 넣어두면 스프링 부트가 정적 리소스로 서비스를 제공한다.

#### 정적 리소스 경로

`src/main/resources/static`

다음 경로에 파일이 들어있으면
`src/main/resources/static/basic/hello-form.html`

웹 브라우저에서 다음과 같이 실행하면 된다.
`http://localhost:8080/basic/hello-form.html`

정적 리소스는 해당 파일을 변경 없이 그대로 서비스하는 것이다.

### 뷰 템플릿

뷰 템플릿을 거쳐서 HTML이 생성되고, 뷰가 응답을 만들어서 전달한다.
일반적으로 HTML을 동적으로 생성하는 용도로 사용하지만, 다른 것들도 가능하다. 뷰 템플릿이 만들 수
있는 것이라면 뭐든지 가능하다.

스프링 부트는 기본 뷰 템플릿 경로를 제공한다.

#### 뷰 템플릿 경로

`src/main/resources/templates`

#### 뷰 템플릿 생성

`src/main/resources/templates/response/hello.html`

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
<meta charset="UTF-8">
<title>Title</title>
</head>
<body>
<p th:text="${data}">empty</p>
</body>
</html>
```

#### ResponseViewController - 뷰 템플릿을 호출하는 컨트롤러

```java
package hello.springmvc.basic.response;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;
@Controller
public class ResponseViewController {

  // String을 반환하는 경우
  @RequestMapping("/response-view-v2")
  public String responseViewV2(Model model) {
    model.addAttribute("data", "hello!!");
    return "response/hello";
  }

  //void를 반환하는 경우
  @RequestMapping("/response/hello")
  public void responseViewV3(Model model) {
    model.addAttribute("data", "hello!!");
  }

  @RequestMapping("/response-view-v1")
  public ModelAndView responseViewV1() {
    ModelAndView mav = new ModelAndView("response/hello").addObject("data", "hello!");
    return mav;
  }

}
```

---

## HTTP 응답 - HTTP API, 메시지 바디에 직접 입력

### @RestController

`@RestController` 는 `@ResponseBody`를 포함하고 있어서 반환값을 직접 HttpResponseBody 쓰겠다는 말임

### HTTP 메시지 컨버터

JAVA 객체를 String으로, String을 JAVA 객체로 알아서 변환해주는 친구

**스프링 MVC는 다음의 경우에 HTTP 메시지 컨버터를 적용한다**

- **HTTP 요청:** `@RequestBody` , `HttpEntity(RequestEntity)`
- **HTTP 응답:** `@ResponseBody`, `HttpEntity(ResponseEntity)`

### HTTP 메시지 컨버터 인터페이스

`org.springframework.http.converter.HttpMessageConverter`

```java
package org.springframework.http.converter;

public interface HttpMessageConverter<T> {
  boolean canRead(Class<?> clazz, @Nullable MediaType mediaType);
  boolean canWrite(Class<?> clazz, @Nullable MediaType mediaType);

  List<MediaType> getSupportedMediaTypes();

  T read(Class<? extends T> clazz, HttpInputMessage inputMessage) throws IOException, HttpMessageNotReadableException;

  void write(T t, @Nullable MediaType contentType, HttpOutputMessage outputMessage) throws IOException, HttpMessageNotWritableException;
}
```

**HTTP 메시지 컨버터는 HTTP 요청, HTTP 응답 둘 다 사용된다**

- `canRead()` , `canWrite()` : 메시지 컨버터가 해당 클래스, 미디어타입을 지원하는지 체크
- `read()` , `write()` : 메시지 컨버터를 통해서 메시지를 읽고 쓰는 기능

### 스프링 부트 기본 메시지 컨버터

(일부 생략)
`0 = ByteArrayHttpMessageConverter`
`1 = StringHttpMessageConverter`
`2 = MappingJackson2HttpMessageConverter`

**대상 클래스 타입과 미디어 타입 둘을 체크해서
사용여부를 결정한다. 만약 만족하지 않으면 다음 메시지 컨버터로 우선순위가 넘어간다.**

몇가지 주요한 메시지 컨버터들

`ByteArrayHttpMessageConverter` : byte[] 데이터를 처리한다.
  
  - 클래스 타입: byte[] , 미디어타입: */* ,
  - 요청 예) @RequestBody byte[] data
  - 응답 예) @ResponseBody return byte[] 쓰기 미디어타입 application/octet-stream

`StringHttpMessageConverter` : String 문자로 데이터를 처리한다.

  - 클래스 타입: String , 미디어타입: */*
  - 요청 예) @RequestBody String data
  - 응답 예) @ResponseBody return "ok" 쓰기 미디어타입 text/plain

`MappingJackson2HttpMessageConverter` : application/json
  
  - 클래스 타입: 객체 또는 HashMap , 미디어타입 application/json 관련
  - 요청 예) @RequestBody HelloData data
  - 응답 예) @ResponseBody return helloData 쓰기 미디어타입 application/json 관련

#### HTTP 요청 데이터 읽기

- HTTP 요청이 오고, 컨트롤러에서 @RequestBody , HttpEntity 파라미터를 사용한다.
- 메시지 컨버터가 메시지를 읽을 수 있는지 확인하기 위해 canRead() 를 호출한다.
  - 대상 클래스 타입을 지원하는가.
    예) @RequestBody 의 대상 클래스 ( byte[] , String , HelloData )
  - HTTP 요청의 Content-Type 미디어 타입을 지원하는가.
    예) text/plain , application/json , */*
  - canRead() 조건을 만족하면 read() 를 호출해서 객체 생성하고, 반환한다.

#### HTTP 응답 데이터 생성

- 컨트롤러에서 @ResponseBody , HttpEntity 로 값이 반환된다.
- 메시지 컨버터가 메시지를 쓸 수 있는지 확인하기 위해 canWrite() 를 호출한다.
  - 대상 클래스 타입을 지원하는가.
    예) return의 대상 클래스 ( byte[] , String , HelloData )
  - HTTP 요청의 Accept 미디어 타입을 지원하는가.(더 정확히는 @RequestMapping 의 produces )
    예) text/plain , application/json , */*
  - canWrite() 조건을 만족하면 write() 를 호출해서 HTTP 응답 메시지 바디에 데이터를 생성한다

---

## 요청 매핑 헨들러 어뎁터 구조

그렇다면 HTTP 메시지 컨버터는 스프링 MVC 어디쯤에서 사용되는 것일까?
다음 그림에서는 보이지 않는다.

![picture 1](/images/e2a54eca31006ce5a975b1a84a0bd796cd3122ba123c6fff804154c3c89a0134.png)  

모든 비밀은 애노테이션 기반의 컨트롤러, 그러니까 @RequestMapping 을 처리하는 핸들러 어댑터인
RequestMappingHandlerAdapter (요청 매핑 헨들러 어뎁터)에 있다

### RequestMappingHandlerAdapter 동작 방식

![picture 2](/images/7d84bc2cd8c0bd03a796bf1366cfbd5023c4c617ec825008dff1b9bffa9371c0.png)  

### ArgumentResolver

`HttpServletRequest` , `Model` 은 물론이고, `@RequestParam` , `@ModelAttribute` 같은 애노테이션
그리고 `@RequestBody` , `HttpEntity` 같은 HTTP 메시지를 처리하는 부분까지 매우 큰 유연함을 보일 수 있는 것은 바로 **`ArgumentResolver`** 덕분이다.

애노테이션 기반 컨트롤러를 처리하는 `RequestMappingHandlerAdapter` 는 바로 이
`ArgumentResolver` 를 호출해서 컨트롤러(핸들러)가 필요로 하는 다양한 파라미터의 값(객체)을 생성한다.
그리고 이렇게 파리미터의 값이 모두 준비되면 컨트롤러를 호출하면서 값을 넘겨준다.

스프링은 30개가 넘는 ArgumentResolver 를 기본으로 제공한다.
어떤 종류들이 있는지 살짝 코드로 확인만 해보자.

정확히는 HandlerMethodArgumentResolver 인데 줄여서 ArgumentResolver 라고 부른다.
```java
public interface HandlerMethodArgumentResolver {
boolean supportsParameter(MethodParameter parameter);

@Nullable
Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception;

}
```

#### 동작 방식

`ArgumentResolver` 의 `supportsParameter()` 를 호출해서 해당 파라미터를 지원하는지 체크하고,
지원하면 `resolveArgument()` 를 호출해서 실제 객체를 생성한다. 그리고 이렇게 생성된 객체가 컨트롤러
호출시 넘어가는 것이다.

그리고 원한다면 여러분이 직접 이 인터페이스를 확장해서 원하는 `ArgumentResolver` 를 만들 수도 있다

### ReturnValueHandler

`HandlerMethodReturnValueHandler` 를 줄여서 `ReturnValueHandler` 라 부른다.
`ArgumentResolver` 와 비슷한데, 이것은 응답 값을 변환하고 처리한다.

컨트롤러에서 String으로 뷰 이름을 반환해도, 동작하는 이유가 바로 `ReturnValueHandler` 덕분이다.
어떤 종류들이 있는지 살짝 코드로 확인만 해보자.

스프링은 10여개가 넘는 ReturnValueHandler 를 지원한다.
예) `ModelAndView` , `@ResponseBody` , `HttpEntity` , `String`

---

## HTTP 메시지 컨버터

### HTTP 메시지 컨버터 위치

![picture 3](/images/8b569d1b2b63824ca93c2c7e2a2ddd31d7d01fbeca752cab8d7d7ed6adea83c7.png)  

HTTP 메시지 컨버터는 어디쯤 있을까?
HTTP 메시지 컨버터를 사용하는 `@RequestBody` 도 컨트롤러가 필요로 하는 파라미터의 값에 사용된다.
`@ResponseBody` 의 경우도 컨트롤러의 반환 값을 이용한다.

**요청의 경우** `@RequestBody` 를 처리하는 `ArgumentResolver` 가 있고, `HttpEntity` 를 처리하는
`ArgumentResolver` 가 있다. 이 `ArgumentResolver` 들이 HTTP 메시지 컨버터를 사용해서 필요한
객체를 생성하는 것이다. (어떤 종류가 있는지 코드로 살짝 확인해보자)

**응답의 경우** `@ResponseBody` 와 `HttpEntity` 를 처리하는 `ReturnValueHandler` 가 있다. 그리고
여기에서 HTTP 메시지 컨버터를 호출해서 응답 결과를 만든다.

스프링 MVC는 `@RequestBody` `@ResponseBody` 가 있으면
`RequestResponseBodyMethodProcessor (ArgumentResolver)`

`HttpEntity` 가 있으면 `HttpEntityMethodProcessor (ArgumentResolver)`를 사용한다.

> 참고 : HttpMessageConverter 를 구현한 클래스를 한번 확인해보자

---

## 확장

스프링은 다음을 모두 인터페이스로 제공한다. 따라서 필요하면 언제든지 기능을 확장할 수 있다.

- `HandlerMethodArgumentResolver`
- `HandlerMethodReturnValueHandler`
- `HttpMessageConverter`

기능 확장은 `WebMvcConfigurer` 를 상속 받아서 스프링 빈으로 등록하면 된다.

### WebMvcConfigurer 확장

```java
@Bean
public WebMvcConfigurer webMvcConfigurer() {
  return new WebMvcConfigurer() {
    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
      //...
    }

    @Overridepublic void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
      //...
    }
  };
}
```