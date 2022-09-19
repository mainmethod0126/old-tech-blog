---
layout: post
title: public final class 란 무엇인가?
tags: [java]
author: mainmethod0126
excerpt_separator: <!--more-->
---


# public final class ??
가끔 자바 소스코드를 열어보다보면 **public final class** 로 선언된 class를 볼 수 있다
무심코 넘어갔지만 무엇인지 확인해보려합니다.
<!--more-->

# 변수에서의 final
우리가 흔히 final을 쓰는 경우는 클래스 필드 즉, 변수를 선언할 때 자주 사용합니다.
변수에서 final 키워드를 붙여 선언하면 해당 변수는 **불변** 을 뜻하게 됩니다.
한번 값이 초기화되면 다신 바꿀 수 없는 상태가 되죠.

그렇다면 class의 final은 무엇일까요?

# 클래스에서의 final
간단합니다. 클래스에서 final은 더 이상 확장을 할 수 없는 클래스, 즉 상속을 받아서 사용할 수 없는 상태를 말합니다.
상속을 염두하여 설계된 클래스가 아닌대 이것을 무작정 상속하여 확장하는 경우를 막기 위해 존재하는 키워드입니다.
