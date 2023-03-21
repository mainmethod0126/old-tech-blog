---
layout: post
title: spring mvc 3장 서블릿,jsp,mvc 패턴
tags: [spring, note]
skills: [java, spring]
author: mainmethod0126
---

# 서블릿,jsp,mvc 패턴

## 테스트 코드 작성시 나온 키워드

### @AfterEach

테스트 코드 작성 후 자동으로 호출됩니다.

만약 테스트 함수가 3개일 경우 각각의 테스트 함수가 완료된 후 @AfterEach 가 호출됩니다.
결과적으로 총 3번의 호출이 이뤄집니다.

## 템플릿 엔진

HTML 문서에서 동적으로 넣어야하는 부분만 java 코드를 넣어서 동적으로 변경하게 해줌

유명한 템플릿 엔진으로는 JSP가 있다

## JSP

### JSP 문법

- `<%@ page contentType="text/html;charset=UTF-8" language="java" %>` 가 문서 최상단에 꼭 들어가야함
- 클래스를 쓰기 위해서는 import를 작성해야함
`<%@ page import="hello.servlet.domain.member.Member" %>`
- `request, response` 는 import없이 jsp 에서 지원해줌
- `<% java code %>` jsp에서는 `<% %>` 로 감싸여있지 않은 부분은 그냥 html text로 인식한다
- `<%= ~~ %>` 와 같이 `=` 을 붙여 사용할 경우 자바 코드를 그대로 Text출력합니다.
- `out` 이라는 예약어를 통하여 `writer.write()` 와 같이 `out.write()` 가 가능하다
- jsp는 서버 내부에서 `서블릿`으로 변환됩니다  서블릿을 구현한 java 코드로 변화하는것으로 예상됨

### JSP의 한계

비즈니스 로직과 View 렌더링 하는 부분이 한 페이지에 섞여있다.
