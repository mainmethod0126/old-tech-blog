---
layout: post
title: equals()와 hashCode()
tags: [java, note, temp]
skills: [java]
author: mainmethod0126
excerpt_separator: <!--more-->
---

# 서론

java에서 String 비교시 `equals()` 라는 함수를 통하여 비교하는 것을 많이들 보았고 또는 많이들 사용해보셨을 것 입니다.
이 `equals()` 는 java 최상위 객체인 `Object` 에 정의되어있는데, 직접 만든 커스텀 객체에서 이 `equals()` 를 재정의할 상황이 생겨 좀 더 자세히 알아보았습니다.

<!--more-->

## 재정의 전 기본 Object.equals()

재정의 되기전에 기본으로 존재하는 Object.equals() 의 코드는 아래와 같습니다.

```java
    public boolean equals(Object obj) {
        return (this == obj);
    }
```

간단합니다. 그냥 비교 연산자 `==` 통하여 현재 객체와 매개변수로 들어온 객체를 비교 후 결과를 반환합니다.
그런데 만약 이 객체가 `사람` 을 의미하는 객체라고 생각해보십다.

```java

public class Human {
    
    public Human(        
        long id,
        String name,
        String age,
        String gender,
        String bloodType) {
            this.id = id;
            this.name = name;
            this.age = age;
            this.gender = gender;
            this.bloodType = bloodType;
    }
    
    long id;
    String name;
    String age;
    String gender;
    String bloodType;
}

public class Test {
    public void humanEqualsExample() {
        Human gilDong = new Human("111", "홍길동", "30", "man", "o");
        Human wuSub = new Human("222", "신우섭", "30", "man", "o");

        // result는 과연..?
        boolean result = gilDong.equals(wuSub);
    }
}
```

위와 같은 샘플 코드 상황이 있을 때 `boolean result = gilDong.equals(wuSub);` 의 결과는 어떻게 나올까요?
둘은 엄연히 다른 사람이기 때문에 결과는 `false` 가 나올겁니다.

하지만 이 사람이 같고 다름의 기준에 대하여 컴퓨터는 모르는 상태입니다.
그럼 어떤 값을 비교해서 다르다고 알려줬을 까요?

처음으로 돌아가 Object.equals() 의 구현을 다시 보면

```java
return (this == obj);
```

단순히 비교연산자만을 사용합니다.
이때 비교 값은 `각 객체의 주소값` 입니다.
두 객체는 서로 다른 Heap 영역에 저장되어있기 때문에 서로 주소값이 달라 `false`를 반환하게 됩니다.

그럼 만약 `gilDong` 의 `id` 와 `이름` 이 바뀌어 `wuSub` 과 동일해지면 어떻게 될까요?

```java
public class Test {
    public void humanEqualsExample() {
        Human gilDong = new Human("222", "신우섭", "30", "man", "o");
        Human wuSub = new Human("222", "신우섭", "30", "man", "o");

        // result는 과연..?
        boolean result = gilDong.equals(wuSub);
    }
}
```

이때의 결과도 `false`가 나올겁니다. 왜냐면 실제로 객체가 가지고 있는 값(필드의 값)만 바뀌었을 뿐 여전히 다른 Heap 영역에 존재하는 별도의 객체이기 때문입니다.

하지만! `Human` 클래스를 설계한 개발자가 의도적으로 **내가 생각하는 `Human` 의 같음은 객체가 지닌 모든 값이 동일할 경우 같은 객체라고 판단할래!** 라고 설계할 수 있습니다.

이럴 경우 보통 `equals()` 함수를 재정의하게 되는대 개발자가 자신의 생각대로 아래와 같인 코드를 변경했다고 생각해 봅시다.


```java
    @Getter
   public class Human {
    
    public Human(        
        long id,
        String name,
        String age,
        String gender,
        String bloodType) {
            this.id = id;
            this.name = name;
            this.age = age;
            this.gender = gender;
            this.bloodType = bloodType;
    }
    
    long id;
    String name;
    String age;
    String gender;
    String bloodType;

    @Override
    public boolean equals(Object obj) {

        if (obj == null) {
           return false;
        }

        Human human = (Human) obj;

        return human.

    }
}
    

```