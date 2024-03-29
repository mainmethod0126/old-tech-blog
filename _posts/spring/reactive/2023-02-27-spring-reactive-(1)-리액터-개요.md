---
layout: post
title: spring 리액터 개요
tags: [spring, note, temp]
skills: [spring, java]
author: mainmethod0126
excerpt_separator: <!--more-->
---


## 리액티브 스프링 이해하기

<!--more-->

일반적인 프로그래밍 방법은 두가지로 분리된다.

- 명령형 : 작업이 완료될때까지 처리 결과를 쌓아놨다가 작업이 완료되면 한번에 반환하는 방식
- 리액티브 : 처리가 끝난 데이터를 즉석으로 반환하고 다른 작업을 계속해서 이어서 하는 방식

**명령형의 예시 :** **`물풍선`** 을 이용한 물놀이, 물풍선에 물을 가득 담아서 한번에 상대방을 적신다. **단점으로는 한번에 하나의 상대만 적실 수 있으며** 다른 상대를 적시기 위해서는 **다시 물풍선에 물을 가득 담는 동안 기다려야한다.** 즉 물을 담는 동안 물놀이가 불가하다.

**리액티브 예시 :** **`물총`**, 한번 물을 채워놓으면 채워놓은 양이 전부 소비되기전까지 상대방을 지속적으로 적실 수 있다. 장점으로는 쏘면서 상대를 변경할 수 있어 **동시에 여러명을 적실 수 있다.** 만약 물총의 물탱크가 빅사이즈면 오랫동안 지속적으로 **끊김없이** 물놀이가 가능하다

## 리액티브 스트림 정의하기

### 탄생 배경

리액티브 스트림은 차단되지 않는 `백 프레셔(backpressure)`를 갖는 **비동기 스트림 처리의 표준**을 제공하기 위하여 넥플릭스, 라이트밴드, 피보탈의 엔지니어들이 합심하여 제작

그 전까지는 표준적인 비동기 스트림 방식이 존재하지 않았음

### 백 프레셔

백 프레셔는 데이터를 소비하는(읽는) **소비자(컨슈머)가 처리할 수 있는 만큼으로 데이터를 제한함**으로써 데이터 생산자가 지나치게 빠르게 데이터를 컨슈머에게 전달하여 데이터 폭주가 발생할 수 있는 상황이 발생하지 않게 함

> **자바 스트림 vs 리액티브 스트림**
> **자바 스트림**은 대개 동기화되어 있고 한정된 데이터로 작업을 수행함
> **리액티브 스트림**은 무한 데이터셋을 비롯해서 어떤 크기의 데이터셋이건 비동기 처리를 지원함, 실시간 데이터 처리, 백 프레션을 이용한 데이터 전달의 폭주를 막음
>

### 리액티브 스트림의 인터페이스

- **Publisher(발행자, 데이터 생산자)**
- **Subscriber(구독자, 데이터 소비자)**
- **Subscription(구독)**
- **Processor(프로세서)**

**Publisher**는 하나의 Subscription당 하나의 Subscriber에 전송할 데이터를 생성한다. P
![picture 1](..//images/aca3b33bf802388f6a1dda08dc008af8bcc875ebd6b67c5513dfaeaa1d8d881b.png)  
