---
layout: post
title: spring jpa 사용시 계층형 데이터 조회하기
tags: [spring]
skills: [spring, java]
author: mainmethod0126
excerpt_separator: <!--more-->
---

"조직" 이라는 정보의 경우 계층형 구조를 갖을 가능성이 높다.
예를 들어 "OOO사업본부 000부서 000팀" 과 같이 사업본부 > 부서 > 팀과 같이 데이터가 존재할 수 있다.
이 데이터를 Spring jpa를 이용하여 db에 저장,조회 등을 할 때 어떻게 해야하는지 알아보았습니다.

<!--more-->

## with recursive (재귀쿼리)

### 재귀쿼리가 실행되는 순서를 그림으로 확인해보자

![picture 0](../../images/b96db62f057327dcc7cc2fe91c62358aa47a3595af5bf03ce964109eacc41810.png)  

![picture 1](../../images/7181ac8c0b7f2ff630ac43b9cd152f584eac71f91901ed5ba62168d6c396c6f5.png)  

![picture 2](../../images/e52e87f7b1fff3871bc03f400cf67409ce93b6a360fe7c35e97b35fd5627e609.png)  
