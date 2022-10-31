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

결론만 먼저 말씀 드리면 **@Configuration** 사용하지 않고 **@Bean** 어노테이션만 사용하여 **Bean 등록** 시 **싱글턴이 보장되지 않습니다** 즉, 단일 빈 하나만 생성하고 재활용되는게 아니라 생성 요청 (new 를 이용한 객체 생성 호출) 시 그대로 객체가 생성되어 버립니다.

그 이유로는 Spring의 싱글턴 객체를 생성하는 방식에 있는데, Spring은 **빈 생성** 시 **CGLIB(Byte Code Generation Library)** 를 이용하여 **프록시 객체** 라는 것을 생성하여 실제 **@Configuration** 가 적용된 클래스를 이용하여 객체를 생성하는 것이 아닌, **JAVA 바이트코드 조작** 을 통하여 **실제 Bean으로 등록될 별도의 클래스를 생성** 합니다.
그 후 이 **생성된 별도의 클래스를 이용해 생성한 객체를 빈으로 등록** 합니다.

이게 무슨 말이냐.. 설명을 돕기 위해 코드를 예를들겠습니다.

##### 사용자가 정의한 Bean 설정 클래스

```java
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

##### CGLIB 의 바이트코드 조작을 통하여 자동으로 생성된 Bean 설정 클래스 (예시를 위한 의사코드입니다)

```java
/**
 * CGLIB가 ApplicationContextConfig 를 상속받은 하위 클래스를 생성합니다.
 */
@Configuration
public class CglibApplicationContextConfig extends ApplicationContextConfig {
    @Bean
    @Override
    public MemberService memberService() {

        // Bean 이미 등록되었는지 확인 후 등록되어있지 않을 경우에만 생성하도록 합니다.
        if (memberService is already registered bean) {
            return memberService;
        } else {
            return new MemberServiceImpl(memoryMemberRepository());
        }
    }

    @Bean
    @Override
    public MemberRepository memoryMemberRepository() {

        // Bean 이미 등록되었는지 확인 후 등록되어있지 않을 경우에만 생성하도록 합니다.
        if (memoryMemberRepository is already registered bean) {
            return memoryMemberRepository;
        } else {
            return new MemoryMemberRepository();
        }
    }

    @Bean
    @Override
    public MemberRepository mariaDBMemberRepository() {

        // Bean 이미 등록되었는지 확인 후 등록되어있지 않을 경우에만 생성하도록 합니다.
        if (mariaDBMemberRepository is already registered bean) {
            return mariaDBMemberRepository;
        } else {
            return new MariaDBMemberRepository();
        }
    }
}
```

CGLIB 가 생성하는 클래스는 기존의 사용자가 정의한 **ApplicationContextConfig** 를 상속받아 **빈 등록 함수들을 재정의** 합니다. 그 이후 실제로 CGLIB를 통하여 생성된 **CglibApplicationContextConfig(가칭)** 클래스가 **빈으로 등록됩니다**
사용자는 디버깅을 해보지 않는 이상 내가 정의한 **ApplicationContextConfig** 가 Bean으로 등록되었다고 착각하게 됩니다.

다시 정리해보면, Bean 객체 생성 시점에서 중간에 프로그램의 흐름을 가져와 **CglibApplicationContextConfig** 라는 별도의 클래스를 생성하고 해당 객체를 등록될 Bean과 갈아 끼워버리는 일종의 도둑질(?)을 하는 것 입니다. (해당 부분 실제로 코드를 분석한 것이 아니라 다를 수 있습니다. 내용이 달라질 경우 글을 수정하도록 하겠습니다.)

실제로 저희가 **AnnotationConfigApplicationContext** 의 생성자 파라미터로 주입한 클래스 정보는 **ApplicationContextConfig.class** 이니 **ApplicationContextConfig** 객체가 빈으로 등록됬어야하는데, 실제로는 **CglibApplicationContextConfig** 객체가 빈으로 등록되는 것 입니다.

실제로 **getBean(ClassType)** 을 통하여 **getBean(ApplicationContextConfig.class)** 을 꺼내와서 객체 자체 toString() 해보면 웬걸? **CGLIB**이라는 이상한 텍스트가 낑겨있는 걸 확인할 수 있습니다. 바로 **CGLIB 라이브러리** 가 만든 **별도의 클래스**를 말하는 것이죠.
> **여기서 잠깐! 별도의 클래스가 등록되었는데 ApplicationContextConfig Type으로 조회가 되네?**
스프링 컨테이너(ApplicationContext)에서 **Type으로 빈 조회 시** 해당 Type의 하위 클래스는 모두 조회되는 특징이 있습니다.
> **CGLIB 라이브러리** 가 생성한 별도의 클래스는 **ApplicationContextConfig** 을 부모로하는 하위 클래스이기 때문에 부모 클래스인 **ApplicationContextConfig** 로 조회가 가능합니다

이렇게 어떤 기능 앞, 뒤로 뜬금없는 다른 기능을 끼워넣어서 실행시키는 것을 **프록시 패턴(Proxy Pattern)** 이라고 하며
이 개념을 도입하여 개발하는 것을 **AOP(Aspect Oriented Programming, 관점 지향 프로그래밍)** 이라고 합니다.

> **여기서 잠깐! 이게 말이되는건가?!**
> 우리가 보고있는 이 행위는 소스코드가 컴파일 된 후 실행 단계인 **런타임 단계**에서 이미 컴파일 단계에서 생성되어진 **java 바이트코드** 에 접근하여 수정하는 마법 같은 작업입니다. 이게 가능한 이유는 **java의 특징인 리플렉션(reflection)** 때문인데 이에 대한 더 자세한 정보는 나중에 시간을 마련하여 공부를 한 후 포스팅을 진행하겠습니다.

이 **프록시 패턴** 이 **@Configuration** 이 존재해야지만 작동하게 되는데, 실제로 **@Configuration** 가 있고, 없고의 차이를 동일한 클래스로 두번 이상 **Bean 등록을 시도**하여 등록된  **동일한 Bean 객체** 인지, 아니면 **별도의 객체**인지 비교를 통하여 확인해보겠습니다.

우선 **ApplicationContextConfig** 클래스를 약간 수정해보겠습니다.


```java
/**
 * MemberService.java
 */
public interface MemberService {

    /**
     * 멤버 정보 저장
     * 
     * @param memberId   : 멤버 고유값
     * @param memberName : 멤버 이름
     */
    public void save(String memberId, String memberName);

    /**
     * 멤버 정보 반환
     * 
     * @param memberId
     * @return : 멤버 이름
     */
    public String findById(String memberId);

    public MemberRepository getMemberRepository();

}
```

```java
/**
 * MemberServiceImpl.java
 */
public class MemberServiceImpl implements MemberService {

    private final MemberRepository memberRepository;

    public MemberServiceImpl(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }

    @Override
    public void save(String memberId, String memberName) {

        memberRepository.save(memberId, memberName);
    }

    @Override
    public String findById(String memberId) {

        return memberRepository.findById(memberId);
    }

    @Override
    public MemberRepository getMemberRepository() {
        return this.memberRepository;
    }

}
```

```java
/**
 * OrderService.java
 */
public class OrderService implements MemberService {

    private final MemberRepository memberRepository;

    public OrderService(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }

    @Override
    public void save(String memberId, String memberName) {

        memberRepository.save(memberId, memberName);
    }

    @Override
    public String findById(String memberId) {

        return memberRepository.findById(memberId);
    }

    @Override
    public MemberRepository getMemberRepository() {
        return this.memberRepository;
    }

}
```

```java
/**
 * ApplicationContextConfig.java
 */
@Configuration
public class ApplicationContextConfig {
    @Bean
    public MemberService memberService() {
        System.out.println("call memberService");
        return new MemberServiceImpl(memoryMemberRepository());
    }

    @Bean
    public MemberService orderService() {
        System.out.println("call orderService");
        return new MemberServiceImpl(memoryMemberRepository());
    }

    @Bean
    public MemberRepository memoryMemberRepository() {
        System.out.println("call memoryMemberRepository");
        return new MemoryMemberRepository();
    }
}
```

위와 같은 코드들을 보면 **memberService** 와 **orderService** 는 둘다 동일하게 생성자를 통해 **MemberRepository** 를 주입받으며 위 예시 코드의 **ApplicationContextConfig** 에서 두 빈의 생성자를 보면 똑같이 **memoryMemberRepository()** 함수의 반환값을 인자로하여 생성하게되는대 **memoryMemberRepository()** 함수를 살펴보면 **MemoryMemberRepository** 의 생성자를 호출하여 객체를 생성하고 생성된 객체를 반환하고 있습니다.

```java
@Bean
public MemberRepository memoryMemberRepository() {
    System.out.println("call memoryMemberRepository");
    return new MemoryMemberRepository();
}
```

결과적으로 **memberService** 와 **orderService** 은 동일한 클래스 객체를 주입받는 모습으로 보입니다.

특별한 점은 호출되는 **memoryMemberRepository()** 는 **@Bean** 이라는 어노테이션을 통하여 Bean을 생성하는 메소드로 정의되었으니,
스프링 컨테이너의 **싱글톤 패턴**에 따라 **memoryMemberRepository()** 로 생성되는 **MemoryMemberRepository** 객체는 **단 하나** 만 존재해야합니다.

그렇기 때문에 **memberService** bean이 갖고 있는 **memoryMemberRepository** 와
**orderService** bean이 갖고 있는 **memoryMemberRepository** 는 동일한 개체(객체 말고 개체입니다. 개체는 고유한 객체를 의미합니다.)여야할 것이 확실해보입니다.

이를 검증하기 위하여 **memberService** 과 **orderService** 두 빈을 **getBean()**하여 꺼내온 후 각각 가지고있는 **memoryMemberRepository** 를 꺼내와 객체 
객체 자체를 표준출력으로 보내면 toString() 이 호출되며 해당 개체의 해시값이 표시되는데
이 해시값이 동일하다면 동일한 **개체(객체와 개체는 다른 의미입니다)** 일 것이고
다르다면 서로 다른 개체일 것 입니다.

##### @Configuration 적용 시

먼저 **@Configuration** 이 적용된 **ApplicationContextConfig** 를 이용하여 아래와 같이 테스트해보겠습니다.

```java
public class AnnotationConfigApplicationContextTest {

    @Test
    @DisplayName("@Configuration 을 사용하는 Config 파일로 annotationConfigApplicationContext Test")
    public void annotationConfigApplicationContextTest_use_Configuration_annotation() {

        try (AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(
                ApplicationContextConfig.class)) {
            MemberService memberService = annotationConfigApplicationContext.getBean("memberService",
                    MemberService.class);
            MemberService orderService = annotationConfigApplicationContext.getBean("orderService",
                    MemberService.class);

            System.out.println("getBean : " + memberService.getMemberRepository());
            System.out.println("getBean : " + orderService.getMemberRepository());

            System.out.println("found member : " + memberService.findById(memberId));
        }
    }
}

```

위 테스트를 실행 전 결과를 미리 예상해보면 아래의 코드 **System.out.println("getBean : " + memberService.getMemberRepository()** 과 **System.out.println("getBean : " + orderService.getMemberRepository()** 의 결과가 완전히 동일할 것이 예상됩니다.

실제로 테스트 코드를 돌려보면 출력되는 **두 emberRepository 객체의 해시값이 동일**하여 서로 **동일한 개체** 인 것을 확인할 수 있습니다.

##### @Configuration 미적용 시

그리고 나서 다음 테스트로는 **@Configuration** 을 제거한 후 테스트 코드를 실행해보겠습니다.

실행 결과를 확인해보면 **두 emberRepository 객체의 해시값이 다른것** 임을 확인할 수 있습니다.

결과적으로 저희가 예상했던대로 **@Configuration** 를 적용하면 **싱글톤 객체임이 보장** 되었고, **@Configuration** 를 사용하지 않을 경우 **Bean** 으로 등록되지만 같은 객체의 **Bean** 이 여러개 등록되는걸 확인했습니다.

위에 설명했던 이유와 같이 **@Configuration** 을 사용할 경우 **프록시 패턴** 이 적용되며 **CGLIB** 가 동작하여 새로운 클래스를 생성하고 해당 클래스가 새로운 객체가 생성되는것을 방지해줍니다.

### GenericGroovyApplicationContext

### GenericXmlApplicationContext
