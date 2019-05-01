---
layout: post
title: DI
categories: [Spring]
comments: true

---
# Dependecy Injection ( 의존성 주입 ) 

Dependect Injection ( DI) 의존성 주입은 스프링의 특징이자 장점중 하나이며 구성요소간 의존 관계가 소스코드 내부에 아닌 외부 설정파일 등을 통해서 정의되는 디자인 패턴중에 하나이다.

의존성 주입을 함으로써 얻게되는 이점은 의존 관계 설정이 컴파일시 아닌 실행시에 이루어져 모듈간의 결합도를 낮출수 있고, 코드의 재사용을 높이고 소스코드들이 한 곳에 모여 있어 유지보수에 편리하다는 장점이 있다.

## DI설정하기

일단 먼저 사용할 패키지를 생성해보겠다.

필자는 com.donghyeon.blog.dipratice를 만들었다.

[![img](https://4.bp.blogspot.com/-e6Av_POj9d8/W95ZbCwg5rI/AAAAAAAABAo/Oaio-E90f_g80Qnlt4cPlYumiS5kAtnaACEwYBhgL/s1600/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA%2B2018-11-04%2B%25E1%2584%258B%25E1%2585%25A9%25E1%2584%258C%25E1%2585%25A5%25E1%2586%25AB%2B11.28.48.png)](https://4.bp.blogspot.com/-e6Av_POj9d8/W95ZbCwg5rI/AAAAAAAABAo/Oaio-E90f_g80Qnlt4cPlYumiS5kAtnaACEwYBhgL/s1600/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%2B2018-11-04%2B%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%2B11.28.48.png)

여기에 DI를 위한 XML 파일을 하나 만들어보겠다.

이름은 `DI-context.xml`로 만들겠다.

테스트를 위해 SayHello 를 만들어보자

> #### SayHello.java

```java
package com.donghyeon.blog.dipractice;

public class SayHello {
    String hello;
    
    public void setHello(String hello) {
        this.hello = hello;
    }

    public void sayHello() {
        System.out.println(hello);
    }
}
```

필드로 String인 hello 변수가 선언되어있고,

hello의 값을 설정해주는 setHello() 함수가 있고,

hello 변수의 값을 콘솔로 찍어주는 sayHello() 함수가 있다.

우리가 해볼 것은 hello의 변수에 <u>직접적으로 값을 주지 않고</u> xml 파일로 값을 주입해볼 것이다.

Main 함수를 만들어 테스트 해보겠다.

> #### DIMain.java

```java
package com.donghyeon.blog.dipractice;

public class DIMain {
    public static void main(String[] args) {
        SayHello hello = new SayHello();
        hello.sayHello();
    }
}
```

### Output

[![img](https://1.bp.blogspot.com/-b_XejLEL6ac/W95bEA6wqqI/AAAAAAAABAw/DE-pPp7SodIil4bXZCpdzXrASTuN1imKwCLcBGAs/s1600/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA%2B2018-11-04%2B%25E1%2584%258B%25E1%2585%25A9%25E1%2584%258C%25E1%2585%25A5%25E1%2586%25AB%2B11.35.52.png)](https://1.bp.blogspot.com/-b_XejLEL6ac/W95bEA6wqqI/AAAAAAAABAw/DE-pPp7SodIil4bXZCpdzXrASTuN1imKwCLcBGAs/s1600/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%2B2018-11-04%2B%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%2B11.35.52.png)

<u>null이 나오는 이유</u>는 아직 SayHello 클래스에 있는 hello 변수가 선언만 되어있고,

값이 초기화 되어있지 않기 때문이다.

이 초기화를 DI-context.xml을 통해서 해보겠다.



> #### DI-context.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">
        
    <bean id="hello" class="com.donghyeon.blog.dipractice.SayHello">
        <property name="hello" value="Donghyeon"/>
    </bean>    
</beans>
```

bean에서 id는 bean들을 구별하기 위한 속성이며 <u>유일</u>해야한다.

class는 주입을 해주고싶은 클래스를 패키지포함해서 적어주면 된다.

<property>태그는 클래스안에 있는 필드들을 말하며,

name속성은 변수의 이름이며, value는 초기화할 값을 적어주면 된다.

자신이 만든 패키지내용과 동일하게 SayHello를 등록해보자.



다시 DIMain 클래스로 가보겠다.

> #### DIMain.java

```java
package com.donghyeon.blog.dipractice;
import org.springframework.beans.factory.xml.XmlBeanDefinitionReader;
import org.springframework.context.support.GenericApplicationContext;
public class DIMain {
    public static void main(String[] args) {
        
        GenericApplicationContext ac =
                new GenericApplicationContext();
        
        XmlBeanDefinitionReader reader =
                new XmlBeanDefinitionReader(ac);
        
        reader.loadBeanDefinitions("com/donghyeon/blog/dipractice/DI-context.xml");
        
        ac.refresh();
                
        SayHello hello = ac.getBean("hello",SayHello.class);
        
        hello.sayHello();
    }
}
```

일단 XML을 가져와서 읽어 오기전에 XML의 내용들을 가져와서 그 <u>내용들을 적을 도화지</u>를 하나 만든다고 생각하면 된다.

[![img](https://1.bp.blogspot.com/-aa_o7JE-zpQ/W95cBARpa3I/AAAAAAAABA8/tVZhf1O015Qir3yY9Yhjemi7P29OqFr4gCLcBGAs/s1600/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA%2B2018-11-04%2B%25E1%2584%258B%25E1%2585%25A9%25E1%2584%258C%25E1%2585%25A5%25E1%2586%25AB%2B11.39.47.png)](https://1.bp.blogspot.com/-aa_o7JE-zpQ/W95cBARpa3I/AAAAAAAABA8/tVZhf1O015Qir3yY9Yhjemi7P29OqFr4gCLcBGAs/s1600/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%2B2018-11-04%2B%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%2B11.39.47.png)

그 다음으로 할건 <u>XML의 값들을 불러</u>와야 될 것이다.

XML의 값을 읽을 수 있게 해주는 클래스가 있는데 XmlBeanDefinitionReader이다.

클래스 이름 앞에 XML이 붙은걸 보아 외부에서 값을 주입해줄 때는 꼭 XML이 아니고 다른 타입의 형식이여도 상관 없다.

그리고 생성자는 BeanDefinitionRegistry 인터페이스를 구현한 클래스면 된다.

여기에서는 위에서 만든 도화지인 GenericApplicationContext는BeanDefinitionRegistry 인터페이스를 구현했다는 사실을 알 수 있다.

이제 선언한 reader로 우리가 xml에서 적어줬던  bean들을 도화지에 옮겨 적을 시간이다.

XmlBeanDefinitionReader안에 있는 reader 메소드를 사용한다.

매개변수 값으로는 xml 파일의 위치를 적어준다.

그 다음에 도화지를 새로고침 해주면 주입이 정상적으로 된 것이다.

확인해보자.

#### Output

[![img](https://1.bp.blogspot.com/-cZ76kW_ScMA/W95cm2C582I/AAAAAAAABBE/B8YLhkwB690pIb7z6lrrdxgXxYNgLy82gCLcBGAs/s1600/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA%2B2018-11-04%2B%25E1%2584%258B%25E1%2585%25A9%25E1%2584%258C%25E1%2585%25A5%25E1%2586%25AB%2B11.42.24.png)](https://1.bp.blogspot.com/-cZ76kW_ScMA/W95cm2C582I/AAAAAAAABBE/B8YLhkwB690pIb7z6lrrdxgXxYNgLy82gCLcBGAs/s1600/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%2B2018-11-04%2B%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%2B11.42.24.png)

 xml에서 Donghyeon으로 값을 적어줬던게 정상적으로 출력된 것을 볼 수 있다.

 이번엔 Hello로 바꿔서 출력해보겠다.

> #### DI-context.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">
        
    <bean id="hello" class="com.donghyeon.blog.dipractice.SayHello">
        <property name="hello" value="hello"/>
    </bean>    
</beans>
```

DIMain 함수는 변경없이 실행해보자.

#### Output

[![img](https://4.bp.blogspot.com/-ACL9qCOfPJI/W95dO2b-gUI/AAAAAAAABBM/7a14G_-29Q03KG8FjLhXRpDcSUZqA5g6ACLcBGAs/s1600/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA%2B2018-11-03%2B%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE%2B2.55.09.png)](https://4.bp.blogspot.com/-ACL9qCOfPJI/W95dO2b-gUI/AAAAAAAABBM/7a14G_-29Q03KG8FjLhXRpDcSUZqA5g6ACLcBGAs/s1600/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%2B2018-11-03%2B%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%2B2.55.09.png)   

## DI를 왜 사용할까?

DI를 사용하는 이유는 이렇게 외부에서 값을 넣어줌으로써 유지보수가 편리하고 한눈에 볼수 있다는 장점이 있다.

예를 들어 interface를 하나 정의한 뒤 그 interface를 구현한 클래스들을 바꾸면서 사용할 수 있다.

> #### Hello 인터페이스

```java
package com.donghyeon.blog.dipractice;
public interface Hello {
    void sayHello();
}
```

Hello 인터페이스를 두어서 클래스간의 관계를 느슨하게 할 수 있다.

우리가 해볼 것은 영어로 인사하기와 한글로 인사하기를 해볼 것이다.

> #### KoreanHello.java

```java
package com.donghyeon.blog.dipractice;
public class KoreanHello implements Hello {
    @Override
    public void sayHello() {
        System.out.println("안녕하세요!!");  
    }
}
```

> #### EnglishHello.java

```java
package com.donghyeon.blog.dipractice;
public class EnglishHello implements Hello{
    @Override
    public void sayHello() {
        System.out.println("Hello World !! ");
        
    }
}
```

> #### DI-context.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">
     
    <bean id="hello" class="com.donghyeon.blog.dipractice.KoreanHello">
    </bean>    
</beans>
```

bean의 <u>id는</u> hello, <u>class는</u> com.donghyeon.blog.dipractice.KoreanHello를 적어줬다.

안녕하세요!!가 출력될 것으로 예상된다.

> #### DIMain.java

```java
package com.donghyeon.blog.dipractice;
import org.springframework.beans.factory.xml.XmlBeanDefinitionReader;
import org.springframework.context.support.GenericApplicationContext;
public class DIMain {
    public static void main(String[] args) {
        
        GenericApplicationContext ac =
                new GenericApplicationContext();
        
        XmlBeanDefinitionReader reader =
                new XmlBeanDefinitionReader(ac);
        
        reader.loadBeanDefinitions("com/donghyeon/blog/dipractice/DI-context.xml");
        
        ac.refresh();
        
        // interface
        Hello hello = ac.getBean("hello",Hello.class);
        
        hello.sayHello();
    }
}
```

먼저 Hello 인터페이스 타입인 hello 변수에 xml에 선언해둔 bean의 id가 hello인 bean을 가져와서

Hello 로 리턴해 hello 변수에 넣어준다.

그 다음  hello.sayHello()함수를 호출해본다.

<u>과연 어떤 값이 출력 될까?</u>

#### **Output**

```
안녕하세요!!
```



이제 DI의 장점이 나온다.

만약 영어로 인사하고 싶다면 DI-contextr.xml에 선언된 hello 빈의 클래스를

<u>EnglishHello</u>로 변경해주면 된다.

```java
<bean id="hello" class="com.donghyeon.blog.dipractice.EnglishHello">
 </bean>    
```

**그런다음 DIMain의 소스는 아무것도 건드리지 않고 실행해보자**.

#### Ouput

[![img](https://1.bp.blogspot.com/-OhOhJlPOJqI/W95n0UDDpeI/AAAAAAAABBY/N1W5OGv089AO2B6nOGHjkAkNo5IohgIGwCLcBGAs/s1600/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA%2B2018-11-04%2B%25E1%2584%258B%25E1%2585%25A9%25E1%2584%258C%25E1%2585%25A5%25E1%2586%25AB%2B11.07.01.png)](https://1.bp.blogspot.com/-OhOhJlPOJqI/W95n0UDDpeI/AAAAAAAABBY/N1W5OGv089AO2B6nOGHjkAkNo5IohgIGwCLcBGAs/s1600/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%2B2018-11-04%2B%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%2B11.07.01.png)

이렇게 중간에 Hello라는 인터페이스를 두면 xml에서 그 인터페이스를 구현한 클래스만 주입해주면 된다.

이렇게 해서 클래스간에 느슨한 관계가 형성이 된다는 장점이 있다.



## DI할때 주의할점.

DI를 해줄때는 <u>DI해줄 객체에 관한 setter 함수</u>가 반드시 정의되어 있어야한다.

```java
String hello;
public void setHello(String hello) {
    this.hello = hello;
}
```

아니면 <u>스프링 프레임워크를 사용하는사람</u>은 @Autowired 어노테이션만 붙이면 자동으로 주입이 된다.

```java
@Autowired
 String hello;
```



## 스프링에서의 DI

> #### web.xml

```xml
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/spring/root-context.xml</param-value>
</context-param>
```

web.xml에 가보면 \<context-name>으로 contextConfigLocation이 적힌 부분이 있다.

이부분의 value 값을 따라가보면 기본적으로 생성되는 root-context.xml이 선언되어있다.

> #### root-context.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!-- Root Context: defines shared resources visible to all other web components -->
</beans>
```

현재는 빈 설정이 아무것도 안된 빈 xml 파일이지만, 여기에 bean 설정을 해주면

서비스 시작시 자동으로 빈 주입이 된다.

DIMain에서 사용했던 도화지를 만드는 작업인,

```java
GenericApplicationContext ac =
                new GenericApplicationContext();
        
        XmlBeanDefinitionReader reader =
                new XmlBeanDefinitionReader(ac);
        
        reader.loadBeanDefinitions("com/donghyeon/blog/dipractice/DI-context.xml");
        
        ac.refresh();
        
        // interface
        Hello hello = ac.getBean("hello",Hello.class);
```

##### 이런 코드를 생략해도 자동으로 프레임워크가 빈을 주입해줘서 편리한 장점이 있다.
