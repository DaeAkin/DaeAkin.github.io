# ‼️ 스프링 부트의 Autoconfiguration

Spring Legacy를 사용하다가 Spring Boot를 사용하게 되면, Legacy에 비해 설정을 따로 해주지 않아도 자동으로 해주기 때문에 엄청 간편하다는 생각을 많이 해보셨을 겁니다. 

이번 시간에는 스프링 부트를 쓰면 왜 설정들을 따로 해주지 않아도 어떻게 자동으로 그 원리를 알아보겠습니다.

## ❓ Auto-configuration이 뭘까?

Spring Boot의 auto-configuration은 추가한 **jar 파일**에 따라 자동적으로 설정을 해줍니다. 예를 들어 HSQLDB가 <u>클래스패스에</u> 존재하고, **데이터베이스의 커넥션을 맺는 Bean을 수동으로 구성해주지 않았다면**, **자동으로 인메모리 DB로 자동 구성** 됩니다.

만약 Spring Legacy이었다면 Connection 오류가 떠서 애플리케이션이 실행이 되지 않습니다.

Auto-configuration을 사용하고 싶다면 **@EnableAutoConfiguration** 또는 **@SpringBootApplication** 주석을 @Configuration 클래스 중 하나에 추가하면 됩니다.

보통 Spring Boot 어플리케이션을 만들면 다음과 같은 보일러플레이트 코드를 만날 수 있습니다.

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class TestApplication {

	public static void main(String[] args) {
		SpringApplication.run(TestApplication.class, args);
	}

}
```

위에 나와 있는 `@SpringBootApplication` 이 자동구성을 해주는 어노테이션이였습니다.!


> @SpringBootApplication 또는 @EnableAutoConfiguration 주석을 하나만 추가해야합니다.



## 🧐 Auto-configuration을 사용하기 싫다면?

언제든지 특정 부분을 Auto-configuration으로 변경할 수 있습니다. 예를 들어 DataSource 빈을 있다면, **디폴트로 사용되는 임베디드 데이터베이스는 더 이상 사용되지 않습니다.**

만약 현재 어느 부분에 Auto-configuration이 적용되어있는지 알고 싶다면 애플리케이션을 `--debug` 과 함께 실행시키면 됩니다.  이렇게 함으로써 logger의 debug를 활성화 시킬 수 있어, 콘솔로 확인할 수 있습니다.

```
$ java -jar TestApplication.jar --debug

============================
CONDITIONS EVALUATION REPORT
============================
Positive matches:
-----------------
   AopAutoConfiguration matched:
      - @ConditionalOnClass found required classes 'org.springframework.context.annotation.EnableAspectJAutoProxy', 'org.aspectj.lang.annotation.Aspect', 'org.aspectj.lang.reflect.Advice', 'org.aspectj.weaver.AnnotatedElement' (OnClassCondition)
      - @ConditionalOnProperty (spring.aop.auto=true) matched (OnPropertyCondition)

   AopAutoConfiguration.CglibAutoProxyConfiguration matched:
      - @ConditionalOnProperty (spring.aop.proxy-target-class=true) matched (OnPropertyCondition)

   CodecsAutoConfiguration matched:
      - @ConditionalOnClass found required class 'org.springframework.http.codec.CodecConfigurer' (OnClassCondition)

....
Exclusions:
-----------
    None
Unconditional classes:
----------------------
    org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration
    org.springframework.boot.autoconfigure.context.PropertyPlaceholderAutoConfiguration
    org.springframework.boot.autoconfigure.info.ProjectInfoAutoConfiguration

```



## 특정 클래스 Auto-configuration 비활성화하기 

Auto-configuration을 적용하고 싶지 않는 클래스가 있다면, `@EnabelAutoConfiguration`의 **exclude** 속성을 사용하여 비활성화 할 수 있습니다.

##### DataSourceAutoConfiguration의 자동구성을 제외하는 예제

```java
import org.springframework.boot.autoconfigure.*;
import org.springframework.boot.autoconfigure.jdbc.*;
import org.springframework.context.annotation.*;

@Configuration
@EnableAutoConfiguration(exclude={DataSourceAutoConfiguration.class})
public class MyConfiguration {
}
```

클래스패스에 클래스가 존재하지 않다면, **excludeName** 속성을 사용하여 이름을 전부 적어주면 됩니다.



## 💡 Auto-configuration 원리

Spring Boot가 실행될 때, 클래스패스에 있는 spring.factories 파일을 찾습니다. 이 파일은 **resources/META-INF/spring.factories**에 있습니다. [spring-boot-autoconfigure](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/resources/META-INF/spring.factories) 프로젝트의 spring.factories의 일부분은 다음과 같습니다.

```
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.MongoAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
```

 이 파일들은 스프링 부트가 실행할 설정 클래스들의 이름들을 담고 있습니다. 위에 일부분을 해석해보면, AOP , RabbitMQ , MongoDB, Spring Batch의 설정 클래스를 실행한다고 해석할 수 있습니다.

그러나, 이 설정 클래스가 클래스패스에 존재해야 실행이 됩니다. 만약 MongoDB가 클래스패스에 있으면, [MongoAutoConfiguration이](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/mongo/MongoAutoConfiguration.java) 실행되며, mongo와 관련된 빈들이 초기화가 됩니다.

MongoAutoConfiguration을 보면 @ConditionalOnClass 어노테이션을 사용하여, 이 클래스가 언제 실행될지 **<u>조건</u>**을 걸어주었습니다.

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass(MongoClient.class)
@EnableConfigurationProperties(MongoProperties.class)
@ConditionalOnMissingBean(type = "org.springframework.data.mongodb.MongoDatabaseFactory")
public class MongoAutoConfiguration {
			...설정들
}
```

MongoAutoConfiguration 클래스는 클래스패스에 MongoClient가 존재할 때만 실행 됩니다.



## 💡 Properties의 원리

데이터베이스 접속 관련 정보를 입력할 땐 /resources/application 파일에 다음과 같이 작성하게 됩니다.

```yaml
spring:
  data:
    mongodb:
      host: 
			....
```

이렇게 사전에 설정된 값으로 빈들을 초기화 하고 싶을 때가 있습니다.

이 값들은 [MongoProperties](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/resources/META-INF/spring.factories) 클래스에 @ConfigurationProperties 어노테이션과 연관 있습니다.

```java
@ConfigurationProperties(prefix = "spring.data.mongodb")
public class MongoProperties {
	public static final int DEFAULT_PORT = 27017;
	public static final String DEFAULT_URI = "mongodb://localhost/test";

	private String host;
	private Integer port = null;
	private String uri;
}
```

@ConfigurationProperties의 prefix와 MongoProperties의 필드이름을 합친 값을 property file에서 사용하게 됩니다.

```yaml
spring:
	data:
		mongodb:
			host:
			port:
			uri :
```



# 🎯 커스텀 Auto-Configuration 만들기

이 내용을 토대로 커스텀 Auto-Configuration을 한 번 만들어 보겠습니다.

- custom 설정을 위한 properties 클래스와 함께 auto-configuration 제공
- pom 또는 gradle로 우리가 만든 custom auto-configuration 의존성을 작성 해, 프로젝트 자동구성 적용해보기

그 전에 앞서서 용어 정리를 해보겠습니다.

- [greeter-library](https://github.com/DaeAkin/greeter-library) : greeter의 코어 로직이 있는 라이브러리 입니다.
- [greeter-spring-boot-autoconfigure](https://github.com/DaeAkin/greeter-spring-boot-autoconfigure) : greeter 라이브러리를 사용하기 위한 설정을 해줘야하는데, **<u>설정을 안할 경우</u>**, 이 라이브러리가 자동설정을 해줍니다.
- [greeter-spring-boot-starter](https://github.com/DaeAkin/greeter-spring-boot-starter) : greeter-library + greeter-spring-boot-autoconfigure 가 합쳐진 라이브러리 입니다.
- [greeter-client](https://github.com/DaeAkin/greeter-client) : greeter을 테스트하기 위한 클라이언트 입니다.

> ## 🎫 네이밍 컨벤션 
>
> acme 이라는 auto-configure 모듈을 만든다면 이 모듈의 이름은 acme-spring-boot-autoconfigure 으로 만들어야 하며, starter는 acme-spring-boot-starter로 만들어야 합니다. 만약 이 모듈 두개를 하나로 만든다면, acme-spring-boot-starter 가 되어야 합니다.
>
> 게다가, starter가 설정에 대한 키를 지원한다면, 적절한 이름을 지어줘야 합니다. Spring Boot가 사용하고 있는(server,management, spring 등)을 사용하지 말아야 합니다. 



## Greeter 라이브러리 만들기 

먼저 Greeter 라이브러리를 간단하게 만들어 보겠습니다.

```java
public class Greeter {

    private GreetingConfig greetingConfig;

    public Greeter(GreetingConfig greetingConfig) {
        this.greetingConfig = greetingConfig;
    }

    public String greet(LocalDateTime localDateTime) {

        String name = greetingConfig.getProperty(USER_NAME);
        int hourOfDay = localDateTime.getHour();

        if (hourOfDay >= 5 && hourOfDay < 12) {
            return String.format("Hello %s, %s", name, greetingConfig.get(MORNING_MESSAGE));
        } else if (hourOfDay >= 12 && hourOfDay < 17) {
            return String.format("Hello %s, %s", name, greetingConfig.get(AFTERNOON_MESSAGE));
        } else if (hourOfDay >= 17 && hourOfDay < 20) {
            return String.format("Hello %s, %s", name, greetingConfig.get(EVENING_MESSAGE));
        } else {
            return String.format("Hello %s, %s", name, greetingConfig.get(NIGHT_MESSAGE));
        }
    }

    public String greet() {
        return greet(LocalDateTime.now());
    }

}
```

Greeter 라이브러리의 주요 클래스입니다. 이 클래스를 간단히 설명하자면, Greeter.greet()를 호출 될 때 properties의 정의된 userName을 가져와서 현재 시간에 맞게 콘솔에 출력하는 간단한 라이브러리 입니다.

이 포스팅의 주된 내용은 라이브러리 구현이 아니기 때문에 간단히 넘어가겠습니다.

좀 더 자세한 내용은 [여기](https://github.com/DaeAkin/greeter-library)를 참조해주세요.

## Greeter-spring-boot-autoconfigure 만들기

그 다음으로greeter-spring-boot-autoconfigure 라는 모듈을 만들어 보겠습니다. 

이 모듈은 2개의 클래스로 이루어져 있는데, GreeterProperties 클래스는 application.yaml(또는 .properties)를 통해 커스텀 설정을 하는 클래스이고, 다른 하나인 GreeterAutoConfiguration 클래스는 greeter 라이브러리를 위한 설정 빈들을 생성하는 클래스입니다.

##### GreeterAutoConfiguration.java

```java
@Configuration
@ConditionalOnClass(Greeter.class)
@EnableConfigurationProperties(GreeterProperties.class)
public class GreeterAutoConfiguration {

    @Autowired
    private GreeterProperties greeterProperties;

    @ConditionalOnMissingBean
    public GreetingConfig greeterConfig() {

        String userName = greeterProperties.getUserName() == null
                ? System.getProperty("user.name")
                : greeterProperties.getUserName();

        // ..

        GreetingConfig greetingConfig = new GreetingConfig();
        greetingConfig.put(USER_NAME, userName);
        // ...
        return greetingConfig;
    }

    @Bean
    @ConditionalOnMissingBean
    public Greeter greeter(GreetingConfig greetingConfig) {
        return new Greeter(greetingConfig);
    }
}
```

애플리케이션이 실행될 때, Greeter 클래스가 클래스패스에 존재하면 GreeterAutoConfiguration 클래스를 실행합니다.

[@ConditionalOnMissingBean](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/condition/ConditionalOnMissingBean.html) 어노테이션을 사용하여, GreeterConfig의 bean이 없을 경우 GreeterConfig bean을 자동 생성합니다. 개발자가 자동생성된 GreeterConfig bean을 사용하고 싶다면, @Configuration 어노테이션을 클래스에 붙여서 동일하게 bean을 만들어주면 됩니다.

##### /resource/META-INF/spring.factories

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  dev.donghyeon.autoconfiguration.GreeterAutoConfiguration
```

다음으로 spring.factories라는 파일이 존재하는데, 이 파일은 애플리케이션이 실행될 때, 시작할 AutoConfiguration 목록을 추가해주는 역할을 합니다.

##### GreeterProperties.java

```java
@ConfigurationProperties(prefix = "donghyeon.greeter")
public class GreeterProperties {

    private String userName;
    private String morningMessage;
    private String afternoonMessage;
    private String eveningMessage;
    private String nightMessage;

    //..getter setter
}
```

@ConfigurationProperties는 설정된 prefix + 필드 이름으로 property를 만들 수 있습니다.
다음과 같이 사용 됩니다.

##### application.yaml

```yaml
donghyeon :
  greeter :
    userName :
    morningMessage :
    afternoonMessage :
    ...
```



## autoconfigure 테스트 코드 작성하기

테스트 코드는 좋은 프로그램을 만드는 좋은 습관이기 때문에, 테스트 코드도 같이 작성해 보겠습니다.

```java
class AutoconfigurationApplicationTests {

    private final ApplicationContextRunner contextRunner = new ApplicationContextRunner()
            .withConfiguration(AutoConfigurations.of(GreeterAutoConfiguration.class));

    @Test
    public void greeterConfigExists() {
        this.contextRunner.run((context -> assertThat(context).hasSingleBean(GreetingConfig.class)));
    }

    @Test
    public void settingsAdded() {
        this.contextRunner.withUserConfiguration(MyGreeterConfig.class)
                .run((context -> assertThat(context.getBean(GreetingConfig.class).getProperty(USER_NAME))
                .isEqualTo("testUserName")));
    }

    @Test
    public void noSettingsAdded() {
        this.contextRunner.run((context ->
                assertThat(context.getBean(GreetingConfig.class).getProperty(USER_NAME))
                        .isEqualTo(System.getProperty("user.name"))));
    }

    //no runtime-generated subclass is necessary.
    @Configuration(proxyBeanMethods = false)
    static class MyGreeterConfig {

        @Bean
        public GreetingConfig myGreeterConfig() {
            GreetingConfig greetingConfig = new GreetingConfig();
            greetingConfig.put(USER_NAME, "testUserName");
            return greetingConfig;
        }

    }
}
```

[테스트코드](https://github.com/DaeAkin/greeter-spring-boot-autoconfigure/blob/master/src/test/java/dev/donghyeon/autoconfiguration/AutoconfigurationApplicationTests.java)는 여기서 보실 수 있습니다.

## starter 사용 해보기

이렇게 만든 라이브러리를 사용 해보겠습니다.

**greeter-library**와 greeter-libarary의 설정을 도와주는 **greeter-spring-boot-autoconfigure** 총 두개의 모듈을 만들었습니다.

이 모듈 2개를 합친 모듈인 **[greeter-spring-boot-starter](https://github.com/DaeAkin/greeter-spring-boot-starter)** 를 이용해서 사용 해보겠습니다.

##### build.gradle

```
plugins {
	id 'org.springframework.boot' version '2.3.2.RELEASE'
	id 'io.spring.dependency-management' version '1.0.9.RELEASE'
	id 'java'
}

group = 'dev.donghyeon'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '1.8'

repositories {
	mavenCentral()
	maven { url 'https://jitpack.io' }
}

dependencies {
	implementation('com.github.DaeAkin:greeter-spring-boot-starter:v1.0.1')
	
	implementation 'org.springframework.boot:spring-boot-starter'
	testImplementation('org.springframework.boot:spring-boot-starter-test') {
		exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
	}
}

test {
	useJUnitPlatform()
}

```



##### GreeterClienApplication.java

```java
@SpringBootApplication
public class GreeterClientApplication implements CommandLineRunner {
	@Autowired
	private Greeter greeter;

	public static void main(String[] args) {
		SpringApplication.run(GreeterClientApplication.class, args);
	}

	@Override
	public void run(String... args) throws Exception {
        String message = greeter.greet();
        System.out.println(message);
	}
}
```

이 클래스를 실행시키면 다음과 같은 결과를 얻습니다.

```
Hello donghyeonmin, null
```



## application.yaml을 이용한 프로퍼티 주입

##### application.yaml


```yaml
donghyeon :
  greeter :
    userName : Hello Donghyeon
```

application.yaml(또는 .properties)를 다음과 같이 수정한 후 다시 실행시키면 다음과 같은 결과를 얻습니다.

##### 결과

```
Hello Hello Donghyeon, null
```









## Condition 어노테이션 정리

거의 항상 Auto-configuration 클래스에는 하나 이상의 @Conditional 어노테이션이 있습니다. @ConditionalOnMissingBean은 개발자가 기본 값에 만족하지 않는 경우 Auto-configuration을 오버라이드 할 수 있도록 해줍니다.



### Class Conditions

`@ConditionalOnClass` 와 `@ConditionalOnMissingClass` 어노테이션은 특정 클래스의 존재 여부에 따라 설정을 해줍니다. Due to the fact that annotation metadata is parsed using [ASM](http://asm.ow2.org/) you can actually use the `value` attribute to refer to the real class, even though that class might not actually appear on the running application classpath. You can also use the `name` attribute if you prefer to specify the class name using a `String` valu



>  If you are using `@ConditionalOnClass` or `@ConditionalOnMissingClass` as a part of a meta-annotation to compose your own composed annotations you must use `name` as referring to the class in such a case is not handled.



### Bean conditions

`@ConditionalOnBean` 과 `@ConditionalOnMissingBean` 어노테이션은 특정 빈의 존재 여부에 따라 설정해 줍니다. value 속성을 이용하면, 빈의 타입을 정해줄수 있고, name 속성을 이용하여 빈의 이름을 정해줄수 있습니다. search 속성을 사용하며, 빈을 검색할 범위에 제한을 걸 수 있습니다.

```java
@Configuration
public class MyAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean
    public MyService myService() { ... }

}
```

위에 예제는 MyService 타입의 빈이 ApplicationConext에서 갖고 있지 않으면, 생성을 해줍니다.



> You need to be very careful about the order that bean definitions are added as these conditions are evaluated based on what has been processed so far. For this reason, we recommend only using `@ConditionalOnBean` and `@ConditionalOnMissingBean` annotations on auto-configuration classes (since these are guaranteed to load after any user-defined beans definitions have been added).



> `@ConditionalOnBean` and `@ConditionalOnMissingBean` do not prevent `@Configuration` classes from being created. Using these conditions at the class level is equivalent to marking each contained `@Bean` method with the annotation.



### property Conditions

`@ConditionalOnProperty` 어노테이션은 특정 스프링 환경 프로퍼티 파일의 존재에 여부에따라 설정을 해주는 어노테이션 입니다. prefix와 name 속성을 이용하여 특정한 propery가 있는지 검사 할 수 있습니다. `havingValue` 와 `matchIfMissing` 속성을 이용하며 좀 더 세밀하게 검사할 수 있습니다.

```
@PropertySource("classpath:mysql.properties")
public class MySQLAutoconfiguration {
    //...
}
```

### Resource Conditions

`@ConditionalOnProperty` 어노테이션은 특정 리소스가 존재할 때 설정 됩니다. 리소스는 주로 스프링 컨벤션을 사용하여 `file:/home/user/test.dat` 처럼 작성합니다.



### Web Application Conditions

`@ConditionalOnWebApplication` 과 `@ConditionalOnNotWebApplication` 어노테이션은 애플리케이션이 웹 어플리케이션인지에 따라 설정 됩니다. 웹 어플리케이션은 스프링 WebApplicationContext를 사용하며, session 범위를 정의하거나, StandardServletEnvironment를 갖는 애플리케이션을 말합니다.



### SpEL expression conditions 

`@ConditionalOnExpression` 어노테이션은 [SpEL expression](https://docs.spring.io/spring/docs/5.0.0.RC3/spring-framework-reference/core.html#expressions) 결과 값에 따라 설정 됩니다.







## 참고자료

https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/#using-boot-auto-configuration

https://www.baeldung.com/spring-boot-custom-auto-configuration

https://docs.spring.io/autorepo/docs/spring-boot/2.0.0.M3/reference/html/boot-features-developing-auto-configuration.html

https://www.baeldung.com/spring-boot-custom-starter

https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-autoconfigure





auto-configuration은 내부적으로 표준 @Configuration 클래스로 구현됩니다. 거기에 `@Conditional` 어노테이션을 사용하면 auto-configuration의 조건을 걸 수 있습니다. 주로 auto-configuration 클래스는 @`ConditionalonClass` 과 `@ConditionalOnMissingBean` 어노테이션을 사용합니다. 이 어노테이션을 사용하면 auto-configuration이 관련 클래스를 찾은 경우, 그리고 @Configuration 어노테이션이 없는 경우에 적용되게 할 수 있습니다.

## 

커스텀된 Auto-Configuration을 만들기 위해서는 @Configuration 어노테이션이 붙은 클래스가 필요하며, 이 클래스를 등록해줘야 합니다.

Mysql 데이터소스를 커스텀 설정을 해보겠습니다.

```java
plugins {
	id 'org.springframework.boot' version '2.3.2.RELEASE'
	id 'io.spring.dependency-management' version '1.0.9.RELEASE'
	id 'java'
}

group = 'dev.donghyeon'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '8'


repositories {
	mavenCentral()
	maven { url 'https://jitpack.io' }
}

dependencies {
  annotationProcessor "org.springframework.boot:spring-boot-autoconfigure-processor"
	implementation 'com.github.DaeAkin:greeter-library:v1.0.0'
	implementation 'org.springframework.boot:spring-boot-starter'
	testImplementation('org.springframework.boot:spring-boot-starter-test') {
		exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
	}
}

bootJar{enabled=false}
jar{enabled=true}
```