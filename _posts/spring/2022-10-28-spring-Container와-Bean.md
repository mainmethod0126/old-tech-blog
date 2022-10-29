# BeanFactory 와 ApplicationContext

## BeanFactory

**BeanFactory** 는 스프링 컨테이너의 **최상위 인터페이스** 이며, 빈을 **관리**, **조회** 하는 기능을 제공합니다.

실제 BeanFactory의 코드를 보면 getBean(), containsBean() 과 빈 조회, 빈이 존재하는지 확인 등의 Bean관리적인 기본 함수 인터페이스들을 제공하는 것을 볼 수 있습니다.
![beanFactory.png](/assets/img/feature-img/spring/beanFactory.png)

하지만 위 정의된 함수만으로 방대한 SpringFramework에서 핵심을 담당하기에는 뭔가 기능이 많이 부족해보입니다. 그렇기 때문에 **사용자가 이 BeanFactory를 직접 사용할 경우가 거의 존재하지 않으며** 실제로는 **ApplicationContext라는 인터페이스들의 구현체**를 이용하는 경우가 대부분입니다.

그럼 그 ApplicationContext가 뭔지 알아보겠습니다.

## ApplicationContext

**ApplicationContext**는 쉽게 말하자면 **BeanFactory에 부가적인 여러 기능을 붙여 확장한 인터페이스**입니다. 당연히 BeanFactory의 확장이기에 BeanFactory가 지원하는 기능들을 포함하고 있습니다. 그렇기 때문에 사용자들이 BeanFactory를 사용하지 않고 ApplicationContext의 구현체를 사용하는 이유입니다.

실제 **ApplicationContext** 코드를 보면 상당히 많은 상속을 받고있는 걸 확인할 수 있는데, 위에서 알아본 **BeanFactory**는 **ApplicationContext** 의 여러 부모 인터페이스중에 하나일 뿐이고 심지어 BeanFacotry 가 직접 상속된 것이 아닌 BeanFactory 를 확장한  **ListableBeanFactory, HierarchicalBeanFactory** 를 상속받을걸 확인 할 수 있습니다. 이 부분을 통하여 **ApplicationContext** 가 실제로 **BeanFactory** 의 기능 뿐만이 아닌 무수히 많은 기능들을 지원하는걸 확인할 수 있습니다.

![ApplicationContext-02.png](/assets/img/feature-img/spring/ApplicationContext-02.png)

간략하게 ApplicationContext 이 상속받은 각 요소들의 기능을 알아보면 아래와 같습니다

### MessageSource (메세지 소스)

언어에 대한 국제화 기능을 지원, 예를 들어 한국에서 들어오면 한국어로, 영어권에서 들어오면 영어로 출력

### EnvironmentCapable (환경 변수)

로컬, 개발, 운영 등의 환경을 구분해서 처리할 수 있게하는 기능을 지원

### ApplicationEventPublisher (애플리케이션 이벤트)

이벤트를 발행하고 구독하는 모델을 편리하게 지원

### ResiurcePatternResolver (편리한 리소스 조회)

파일, 클래스패스, 외부 등에서 리소스를 편리하게 조회

이렇게 많은 기능이 추가되어 확장된 **ApplicationContext**도 결국은 **인터페이스**입니다 실제로 이 기능들을 단순히 추상 메소드로 실제로는 구현되지 않은 메소드들입니다, 쉽게 말해 **껍데기만 존재하는 깡통**입니다.
그렇다면 실제로 이 많은 기능들을 구현하게되는 **ApplicationContext 구현체**들에 대하여 알아보겠습니다

## ApplicationContext 구현체 (대표적인 3개)

보통 스프링컨테이너 즉, 빈을 관리하기 위해 사용하게 될 ApplicationContext 의 구현체를 알아보기 위해서 소스코드를 본 결과 생각보다 여러가지가 존재했습니다

![applicationContextImpls.png](/assets/img/feature-img/spring/applicationContextImpls.png){: .align-center}
*캡션 테스트*

위 사진을 보면 ApplicationContext를 최상위 인터페이스로하는 여러 클래스 및 인터페이스가 존재합니다.
모든 클래스를 다 알아보기에는 시간과 정신력이 부족함으로 대표적으로 사용되는 **3개의 클래스** 를 알아보겠습니다.

![GenericApplicationContextChildrens.png](/assets/img/feature-img/spring/GenericApplicationContextChildrens.png)
![ApplicationContextImpls-02.png](/assets/img/feature-img/spring/ApplicationContextImpls-02.png)

그전에 잠시 Ann

### AnnotationConfigApplicationContext

**AnnotationConfigApplicationContext** 는 **어노테이션 기반 자바 코드**를 이용하여 **빈 의존관계**를 설정할 수 있도록 되어있는 스프링 컨텍스트입니다.

그렇다면 어떤 어노테이션을 이용하여 **빈 의존관계** 를 설정할 수 있을까요?

이때 사용되는 어노테이션은 **@Configuration** 입니다.

대략적인 사용방법을 알아보자면 먼저 **@Configuration** 이 적용된 **Bean설정을 위한 클래스** 가 존재해야합니다.

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import com.example.demo.member.MemberService;
import com.example.demo.member.MemberServiceImpl;
import com.example.demo.member.repo.MariaDBMemberRepository;
import com.example.demo.member.repo.MemberRepository;
import com.example.demo.member.repo.MemoryMemberRepository;

@Configuration
public class ApplicationContextConfig {
    @Bean
    public MemberService memberService() {
        return new MemberServiceImpl(memoryMemberRepository());
    }

    @Bean
    public MemberRepository memoryMemberRepository() {
        return new MemoryMemberRepository();
    }

    @Bean
    public MemberRepository mariaDBMemberRepository() {
        return new MariaDBMemberRepository();
    }
}
```

예를 들어 위와 같은 **Bean설정을 위한 클래스** 가 존재할 때 이를 이용하여 **AnnotationConfigApplicationContext** 객체를 생성할 수 있습니다.

```java
public class AnnotationConfigApplicationContextTest {

    @Test
    @DisplayName("@Configuration 을 사용하는 Config 파일로 annotationConfigApplicationContext Test")
    public void annotationConfigApplicationContextTest_use_Configuration_annotation() {

        try (AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(
                ApplicationContextConfig.class)) {
            MemberService memberService = annotationConfigApplicationContext.getBean("memberService",
                    MemberService.class);

            String memberId = "abc";
            String memberName = "wusubshin";

            memberService.save(memberId, memberName);

            System.out.println("found member : " + memberService.findById(memberId));
        }
    }
}
```

> **여기서 잠깐!**
> AnnotationConfigApplicationContext 클래스의 상속 관계를 살펴보면 ConfigurableApplicationContext 라는 인터페이스를 찾을 수 있는데, 이 인터페이스는 **Closeable 인터페이스** 를 상속받았습니다.
> 그렇기 때문에 try () {} 문법을 통하여 **close() 호출**이 보장되지 않을 경우
> **Resource leak: 'annotationConfigApplicationContext' is never closedJava(536871799)** 라는 경고가 발생합니다.
> 좀 더 자세한 정보는 **Closeable 인터페이스** 에 대하여 별도 검색을 부탁드리겠습니다.

위와 예시 코드의 **AnnotationConfigApplicationContext 생성자 호출** 부분을 보면 사전에 작성해놓은 **Bean설정을 위한 클래스 ApplicationContextConfig** 의 클래스 타입을 매개변수로 받는 걸 확인할 수 있습니다.

```java
AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(
                ApplicationContextConfig.class)
```

이렇게 **ApplicationContextConfig.class** 를 매개변수로 하여 객체를 생성하게 되면,
ApplicationContextConfig 내부에 정의되어있던 **@Bean 어노테이션** 이 붙은 함수의 반환 객체들을 모두 **Bean 으로 등록** 됩니다.

정말 Bean등록이 잘 이루어졌는지 확인하기 위해서 **@Bean** 이 붙어있던 **메소드의 이름** 으로 **getBean()** 을 호출하면 생성된 **Bean 객체** 를 얻어올 수 있습니다.

```java
MemberService memberService = annotationConfigApplicationContext.getBean("memberService",MemberService.class);
```

여기서 약간 호기심이 더 많으신 분이라면 ApplicationContextConfig 클래스에서 **@Configuration** 을 사용하지 않으면 Bean 등록이 불가능한가? 라는 궁금증을 갖으실 수 있습니다.
#### @Configuration 을 사용하지 않는다면?




### GenericGroovyApplicationContext

### GenericXmlApplicationContext
