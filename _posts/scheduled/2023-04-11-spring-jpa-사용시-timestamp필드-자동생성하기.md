---
layout: post
title: spring jpa 사용시 timestamp필드 자동생성하기
tags: [spring, note, temp]
skills: [java, spring]
author: mainmethod0126
---

## 공통적으로 entity들이 영속화되는 순간의 시간정보를 저장하도록 할 수 없을까?

운영을 하다보면 이 데이터가 언제 db에 들어간건지 확인하고 싶을때가 있습니다. 이럴때 마땅한 time 정보가 없을 경우 굉장히 골치 아파집니다.

그렇기 때문에 애초에 모든 entity가 공통적으로 timestamp를 포함하여 저장되게 만들면 좋을 것 같기에 방법을 알아보았습니다.

### time 값을 가지는 공통 부모 클래스를 생성한 후 모든 entity들이 해당 클래스를 상속받아 사용하는 방법

공통적으로 모든 entity들이 영속화되는 순간의 시간정보를 저장하도록 할려면, 해당 기능을 포함하고 있는 공통되는 부모 클래스를 사용하면 될 것 같은 느낌입니다.

만약, user, item 이라는 두개의 entity가 존재할 때를 예로 들어어보겠습니다.

```java
@Entity
public class BaseEntity {
    // 생성 시간
    private LocalDateTime createdAt;
    
    // 변경 시간
    private LocalDateTime updatedAt;
}
```

```java
@Entity
public class User extends BaseEntity {
    private String name;
    private String age;
}
```

```java
@Entity
public class Item extends BaseEntity {
    private String name;
    private Long price;
}
```

자 위 예시처럼 user와 item 클래스가 공통적으로 baseEntity 를 상속받습니다.

이 baseEntity에는 `createdAt` 는 생성 시간을, `updatedAt` 변경 시간을 나타내는 필드들이 존재합니다.

그럼 이제 user와 item 객체는 영속화 될 때 createdAt과 updatedAt을 포함하여 영속화 되겠죠?

하지만 매번 사용자가 `setCreatedAt(), setUpdatedAt()` 을 명시하기 보다는 자동으로 값이 들어가면 좋을 것 같습니다.

이때 `@CreatedDate, @LastModifiedDate` 를 사용할 수 있습니다.

---

### @CreatedDate, @LastModifiedDate 과 @EnableJpaAuditing

`@CreatedDate`, `@LastModifiedDate` 두 어노테이션은 변수에 직접 시간을 할당해주지 않아도 자동으로 시간이 할당되도록 해주는 어노테이션입니다.

예를 들어 보겠습니다.

```java
@Entity
public class BaseEntity {
    // 생성 시간
    @CreatedDate
    private LocalDateTime createdAt;
    
    // 변경 시간
    @LastModifiedDate
    private LocalDateTime updatedAt;
}
```

user와 item의 부모클래스인 BaseEntity 의 각 필드 createdAt 와 updatedAt 를 위와 같이 수정해주면

- `@CreatedDate` 가 붙은 createdAt은 entity 가 db에 저장되는 시점의 시간정보가 자동으로 들어가고,

- `@LastModifiedDate` 가 붙은 updatedAt은 마찬가지로 db에 저장되는(db의 값이 변경되는) 시점의 시간정보가 자동으로 들어갑니다.

그럼 이렇게만 해주면 끝이냐? 그건 아닙니다.

추가적으로 사용해야할 어노테이션이 몇개 더 있습니다.

---

### @MappedSuperclass

JPA에서 @Entity 가 붙은 클래스가 부모 클래스를 상속 받기 위한 조건이 있습니다.

- **부모 클래스가 @Entity 을 사용한 클래스여야 한다**
- **부모 클래스가 @MappedSuperclass 을 사용한 클래스여야 한다**

둘 조건간에는 명확한 차이가 존재합니다.

@Entity 클래스의 경우 실제 DB 테이블과 무조건 매핑되어야합니다.
그런데 저희가 만든 BaseEntity는 공통 매핑정보만을 제공할 뿐 DB에 **BaseEntity** 라는 테이블이 존재해서는 안됩니다.

그럼 **DB에 테이블은 만들지 않고, @Entity 클래스의 부모 클래스로 사용하기 위해서는 어떤 방법이 있을까요?**

이때 등장하는게 바로 **@MappedSuperclass** 입니다.

이 **@MappedSuperclass** 어노테이션은 **매핑정보만을 제공하는 부모 클래스** 라는걸 명시해주는 어노테이션입니다.
다시 한번 어노테이션 이름을 보면 딱 목적에 맞는 이름이라는 걸 알 수 있습니다.

> 주의! @MappedSuperclass 을 사용하는 클래스는 추상클래스로 선언할 것을 권장합니다 그 이유는 **BaseEntity** 클래스를 별도의 객체로 생성해서 사용하는 것을 막기 위함입니다.

```java
@MappedSuperclass // @Entity 를 삭제하고 @MappedSuperclass 를 추가~!
public abstract class BaseEntity { // abstract 를 추가하여 추상 클래스임을 명시합니다! 
    // 생성 시간
    @CreatedDate
    private LocalDateTime createdAt;
    
    // 변경 시간
    @LastModifiedDate
    private LocalDateTime updatedAt;
}
```

그럼 이제 끝이냐? 아직 남았습니다ㅠ..

---

### @EntityListeners

`@EntityListeners` 는 JPA의 이벤트 리스너를 기반으로 한 어노테이션입니다.

`이벤트 리스너` 라는 쉽게 말해 **특정 이벤트가 발생했을 때 실행될 콜백 함수를 등록해놓고 해당 이벤트를 감지하고 있다가 이벤트가 발생하면 콜백 함수를 실행시켜주는 친구** 입니다.

그 개념을 활용한 것이 `@EntityListeners` 인데 이름 답게 `Entity`에게 어떤 이벤트가 발생했을 때를 감지할 수 있는 이벤트 리스너를 등록할 수 있게 해줍니다

기본적으로 아래와 같은 Event들에 대하여 쉽게 감지할 수 있도록 되어있습니다.

- **@PrePersist** : 엔티티가 영속화되기 전에 호출됩니다.
- **@PostPersist** : 엔티티가 영속화된 후에 호출됩니다.
- **@PreUpdate** : 엔티티가 업데이트되기 전에 호출됩니다.
- **@PostUpdate** : 엔티티가 업데이트된 후에 호출됩니다.
- **@PreRemove** : 엔티티가 삭제되기 전에 호출됩니다.
- **@PostRemove** : 엔티티가 삭제된 후에 호출됩니다.
- **@PostLoad** : 엔티티가 로딩된 후에 호출됩니다.

그럼 결론적으로 저희가 사용할 `@CreatedDate` 과 `@LastModifiedDate` 는 Entity가 DB에 **영속화되기 직전의 시간정보**를 얻어야하니까
**@PrePersist 가 붙은 메소드를 구현하는 클래스를 하나 만들면 되겠습니다**

하지만, 전세계에는 많은 개발자가 있고 이미 누군가 만들어 놓은 클래스가 있습니다.
**AuditingEntityListener** 라는 클래스인데.

요놈의 구현부를 보면

```java
@PrePersist
public void touchForCreate(Object target) {
    // ...
}
```

```java
@PreUpdate
public void touchForUpdate(Object target) {
    // ...
}
```

이런식으로 `@PrePersist, @PreUpdate` 두 이벤트에 대해서 이미 처리를 해놨습니다.

그럼 저희는 요놈을 아래와 같이 사용하면 됩니다.

```java
@EntityListeners(AuditingEntityListener.class)
@MappedSuperclass // @Entity 를 삭제하고 @MappedSuperclass 를 추가~!
public abstract class BaseEntity { // abstract 를 추가하여 추상 클래스임을 명시합니다! 
    // 생성 시간
    @CreatedDate
    private LocalDateTime createdAt;
    
    // 변경 시간
    @LastModifiedDate
    private LocalDateTime updatedAt;
}
```

자 이렇게되면 `BaseEntity` 를 상속받은 `user`와`item` Entity에서 `@PrePersist` 또는 `@PreUpdate` 이벤트가 발생하는것을 **AuditingEntityListener** 가 감지 후 값 변경을 위한 콜백이 동작할 거입니다.

> 어떻게 동작하나 대충 **AuditingHandlerSupport.class** 까지 들어가봤더니
리플랙션을 통하여 **AuditableBeanWrapper** 라는 **감사 가능한 래핑 객체**를 만들어내고, 해당 랩핑 객체를 통하여 원본 Bean객체(여기서는 BaseEntity를 상속받은 다른 Entity들)의 **createdAt** 과 **updatedAt** 의 값을 바꿔버리는 것으로 예상됩니다.

---

### @EnableJpaAuditing

자.. 그럼 마지막입니다!

최종적으로 entity에 대한 감사 기능을 **활성화!!** 해줘야합니다.

`@EnableJpaAuditing` 이라는 어노테이션을 선언함으로 이 **활성화** 를 시킬 수 있으며,

`@EnableJpaAuditing` 이 붙어야하는 위치는 `@Configuration` 이 선언된 클래스 어디서나 하면되는데 가시성을 높이기 위해서 **@SpringBootApplication** 가 선언된 클래스에 붙여주시면 좋습니다.

> @SpringBootApplication 도 @Configuration 을 내포합니다.

자 이제 Entity를 db에 실제 넣어보고, 업데이트해보면 **createdAt** 과 **updatedAt** 의 값이 잘 저장되는 걸 확인하실 수 있을겁니다.

## 끝