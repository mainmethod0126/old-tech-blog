

평상시에 사용하던 IOC와 DI에 대하여 좀 더 깊이 이해하는 시간을 가져볼까 합니다.
<!--more-->

# OOP에서의 IOC (제어의 역전)

저희가 객체지향 언어을 이용하여 개발을 할때는 **흐름**이라는 것이 존재합니다.
먼저 객체를 **생성** 하고, 필요에 따라 **호출** 하고, 더이상 사용할 필요가 없다하면 **소멸** 시키죠
JVM과 같이 별도의 VM이 존재하지 않는 언어의 경우 이 **생성, 호출, 소멸** 을 개발자가 원하는 시점에서 진행하게됩니다.
개발자가 이 **흐름을 제어**하게되죠
예를들어 봅시다.

~~~JAVA
// 사용자 정보 클래스
public class User {
    
    private String name;
    private String address;


    public User(String name, String address) {
        this.name = name;
        this.address = address;
    }

    public String getName() {
        return this.name;
    }
}
~~~

~~~JAVA
public class MyApplication {
	public static void main(String[] args) {
        // 개발자가 원하는 시점에 new 키워드를 통한 객체 생성        
        User user = new User("신우섭", "의왕시");

        // 개발자가 원하는 시점에서 생성된 user 객체의 getName 함수를 호출
        String userName = user.getName();
        
        sysout("userName");

        // 만약 user객체를 더 이상 사용하지 않는다 하면
        // 소멸자 호출등을 진행 (JAVA 같은 경우 JVM이 대신하기 때문에 별도의 호출이 필요하지 않습니다.)
	}
}
~~~

위와 같이 **User** 클래스가 존재하고 이를 main 함수에
서 사용한다고 예를들어 보겠습니다.

개발자가 User 객체가 필요한 시점에서 User 클래스를 통해 **인스턴스화** 즉 객체를 **생성***하고, 원하는 시점에서 getName() 과 같이 함수를 **호출**합니다.
사용한 객체가 더 이상 불필요하다고 생각되면 **소멸** 도 원하는 시점에 진행하게됩니다.

이처럼 모든 **흐름**을 **개발자**가 관리하죠 즉 **제어권**이 개발자에게 존재합니다.

그럼 다시 **제어의 역전** 이라는 키워드를 살펴보겠습니다.
좀전에 살펴보았던 **제어**가 역전되었다는 말로 해석할 수 있는데, 이것이 어떻게 되었길래 **역전** 되었다고 할 수 있을까요?

보통 개발에서 개발자가 직접적으로 코드를 수정하지 않는 부분들이 존재합니다.
바로 **프레임 워크 또는 라이브러리** 등이 그것이며 사전에 필요한 기능들이 만들어져있고 개발자는 이를 수정하지 않고 가져다 쓰기만 합니다. **프레임 워크 또는 라이브러리** 사용에 필요한 입력과 입력에 대한 출력만 보장되면 내부 코드가 어떻든 신경쓰지 않습니다.

바로 이 **프레임 워크와 라이브러리** 가 개발의 흐름을 제어하게되면 이를 제어의 역전이라합니다.

이해를 돕기 위해 위 User 클래스와 User를 사용하던 main 함수를 약간 변형하여 예시를 들어보겠습니다.
~~~JAVA
public class MyApplication {
	
    public static void main(String[] args) {


        Container container = new Container(ContainerConfig.class); 

        User user = container.getInstance("user");


        // 개발자가 원하는 시점에서 생성된 user 객체의 getName 함수를 호출
        String userName = user.getName();
        
        sysout("userName");

        // 만약 user객체를 더 이상 사용하지 않는다 하면
        // 소멸자 호출등을 진행 (JAVA 같은 경우 JVM이 대신하기 때문에 별도의 호출이 필요하지 않습니다.)
	}
}
~~~
위 코드를 보면 기존에 User 객체를 생성하던 **new** 호출 부분이 사라지고
**Container** 라는 객체를 생성하여 User 객체를 얻어오고 있습니다.
new 를 이용한 user를 **생성** 하는 부분이 사라졌습니다.
이미 어딘가에서 생성된 User 객체를 가져와서 사용하기만 하는거죠.

여기서 사용된 **Container** 가 **프레임 워크**에서 지원하는 클래스가 되는 것입니다.

그러면 User라는 클래스 자체는 개발자가 정의한 클래스인데 **Container** 라는 녀석이 어떻게 알고 객체를 생성하고 돌려주는 것 일까요?

예제 코드를 보면 **Container** 객체를 생성할 때 **ContaierConfig.class** 를 인자로 받아 생성합니다.

우리는 이 **ContaierConfig.class** 에 사용될 **객체 생성에 대한 정보**를 **미리 정의**해놓습니다.

**ContaierConfig** 의 예시는 아래와 같습니다.

~~~java
public class ContainerConfig() {
    
    public User user() {
        return new User("신우섭", "의왕시");
    }

}
~~~

이렇게 ContainerConfig에 user 생성애 대한 정보를 미리 정의하고 **Container** 객체가 위 설정 정보를 참고하도록 설정해놓으면

개발자가 **user** 객체를 요구할때 이 정보를 토대로 **user** 객체를 생성하여 반환해줍니다.

객체 생성에 대한 권한을 **Container** 에게 넘긴거죠. 개발자는 **user** 생성 방식만을 정해놨을 뿐 실제로 **생성**을 호출하진 않습니다.

이렇게 기존에는 개발자가 갖고 있던 **생성** 에 대한 **권한** 을 **프레임 워크** 에게 넘기는 것 과 같이 **제어권**을 개발자가 아닌 누군가가 갖는 것을 **IOC 제어의 역전** 이라고 하며 **객체에 대한 생성과 의존관계 주입** 해주는 예시의 **Container 클래스** 와 같은 객체를 **IOC 컨테이너** 라고 부릅니다.

## IOC 컨테이너
IOC 컨테이너에 대하여 좀 더 자세히 알아보겠습니다.
IOC 컨테이너의 주 목적은 **객체 생성, 의존관계 설정** 이며
