---
layout: post
title: 토비의 봄 스프링 리액티브 프로그래밍(1)-Reactive Streams
tags: [spring, temp, note]
skills: [spring]
author: mainmethod0126
excerpt_separator: <!--more-->
---

# 토비의 봄 스프링 리액티브 프로그래밍(1)-Reactive Streams

토비님의 유튜브 강의를 보고 내용을 정리합니다.

<!--more-->

## Duality
`Iterable` 과 `Observable` 은 쌍대성이다.

```java
Iterable <---> Observable
```

Iterable 은 `Pull` 방식이다.

```java
for(Iterator<Integer> it = iter.iterator(); it.hasNext();) {
    sysout(it.next());
}
```

위 코드에서 `it.next()` 가 `Pull` 방식이다

Observable 은 `Push` 방식이다.
이거 가져가라~ 하고 밀어넣어주는 방식이 Push 방식이다.



## Observer Pattern





## Reactive Streams - 표준 - Java9 API에 포함됨