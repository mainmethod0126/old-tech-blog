---
layout: post
title: JAVA MOCKITO SPY 객체에서 변수값이 바뀌지 않는 현상
tags: [java, temp, note]
skills: [java]
author: mainmethod0126
excerpt_separator: <!--more-->
---

Spy(원본객체)로 생성된 Mockito.spy 객체는 실제 객체와는 다른 객체입니다. 실제 객체의 메소드를 똑같이 동작할 뿐 Spy객체의 변수값을 바꾼다고 원본객체의 변수값이 변하는건 아닙니다.

만약 원본 객체의 함수내부에 this.myName 이라는 변수의 값을

```java
this.myName = "길동"
```

위 코드와 같이 바꾸길 시도하였을 때 실제로 this.myName 객체는 변경되지 않습니다.

자세하는거는 더 알아봐야합니다.
