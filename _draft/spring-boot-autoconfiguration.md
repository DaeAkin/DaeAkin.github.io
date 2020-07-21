# ‼️ 스프링 부트의 Autoconfiguration

Spring Legacy를 사용하다가 Spring Boot를 사용하게 되면, Legacy에 비해 설정을 따로 해주지 않아도 자동으로 해주기 때문에 엄청 간편하다는 생각을 많이 해보셨을 겁니다. 

이번 시간에는 스프링 부트를 쓰면 왜 설정들을 따로 해주지 않아도 어떻게 자동으로 그 원리를 알아보겠습니다.

## 🚗 Auto-configuration

Spring Boot의 auto-configuration은 추가한 jar 파일에 따라 자동적으로 설정을 해줍니다. 예를 들어 HSQLDB가 클래스패스에 존재하고, 데이터베이스의 커넥션을 맺는 Bean을 수동으로 구성해주지 않았다면, **자동으로 인메모리 DB로 자동 구성** 됩니다.

Spring Legacy에서는 Connection 오류가 떠서 애플리케이션이 실행이 되지 않습니다.

Auto-configuration을 사용하고 싶다면 @EnableAutoConfiguration 또는 @SpringBootApplication 주석을 @Configuration 클래스 중 하나에 추가하면 됩니다.

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

> @SpringBootApplication 또는 @EnableAutoConfiguration 주석을 하나만 추가해야합니다.



## Auto-configuration으로 점진적으로 변경하기

언제든지 특정 부분을 auto-configuration으로 변경할 수 있습니다. 예를 들어 DataSource 빈을 있다면, **디폴트로 사용되는 임베디드 데이터베이스는 더 이상 사용되지 않습니다.**

만약 현재 어느 부분에 auto-configuration이 적용되어있는지 알고 싶다면 애플리케이션을 `--debug` 과 함께 실행시키면 됩니다.  이렇게 함으로써 logger의 debug를 활성화 시킬 수 있어, 콘솔로 확인할 수 있습니다.

```
$java -jar TestApplication.jar --debug
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



## 특정 클래스 Auto-configuration  비활성화하기 

auto-configuration을 적용하지 않은 클래스가 있다면, `@EnabelAutoConfiguration`의 exclude 속성을 사용하여 비활성화 할 수 있습니다.

```java
import org.springframework.boot.autoconfigure.*;
import org.springframework.boot.autoconfigure.jdbc.*;
import org.springframework.context.annotation.*;

@Configuration
@EnableAutoConfiguration(exclude={DataSourceAutoConfiguration.class})
public class MyConfiguration {
}
```

클래스패스에 클래스가 존재하지 않다면, excludeName 속성을 사용하여 이름을 전부 적어주면 됩니다.



## 커스텀 Auto-Configuration 만들기

회사에서 라이브러리를 만들거나, 오픈소스 또는 상업적 라이브러리를 만들 때 auto-configuration을 적용하고 싶을 때가 있습니다. 

## auto-configuration 원리

auto-configuration은 내부적으로 표준 @Configuration 클래스로 구현됩니다. 거기에 `@Conditional` 어노테이션을 사용하면 auto-configuration의 조건을 걸 수 있습니다. 주로 auto-configuration 클래스는 @`ConditionalonClass` 과 `@ConditionalOnMissingBean` 어노테이션을 사용합니다. 이 어노테이션을 사용하면 auto-configuration이 관련 클래스를 찾은 경우, 그리고 @Configuration 어노테이션이 없는 경우에 적용되게 할 수 있습니다.



## auto-configuration 후보자 설정하기

Spring Boot는 `META-INF/spring.factories` 파일의 유무를 검사합니다. 이 파일은 EnableAutoConfiguration 

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.mycorp.libx.autoconfigure.LibXAutoConfiguration,\
com.mycorp.libx.autoconfigure.LibXWebAutoConfiguration
```

`@AutoConfigureAfter` 또는 `@AutoConfigureBefore` 어노테이션을 이용하면 사용자가 원하는 순서대로 설정이 적용됩니다. 예를 들어, 만약 web 관련 설정을 한다면, `WebMvcAutoConfiguration` 이 된 후 에 설정이 적용되도록 하는 것이 좋습니다.



## Condition 어노테이션

거의 항상 auto-configuration 클래스에는 하나 이상의 @Conditional 어노테이션이 있습니다. @ConditionalOnMissingBean은 개발자가 기본 값에 만족하지 않는 경우 auto-configuration을 오버라이드 할 수 있도록 해줍니다.



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



## 네이밍 컨벤션 

acme 이라는 auto-configure 모듈을 만든다면 이 모듈의 이름은 acme-spring-boot-autoconfigure 으로 만들어야 하며, starter는 acme-spring-boot-starter로 만들어야 합니다. 만약 이 모듈 두개를 하나로 만든다면, acme-spring-boot-starter 가 되어야 합니다.

게다가, starter가 설정에 대한 키를 지원한다면, 적절한 이름을 지어줘야 합니다. Spring Boot가 사용하고 있는(server,management, spring 등)을 사용하지 말아야 합니다. 



## 블

커스텀된 Auto-Configuration을 만들기 위해서는 @Configuration 어노테이션이 붙은 클래스가 필요하며, 이 클래스를 등록해줘야 합니다.

Mysql 데이터소스를 커스텀 설정을 해보겠습니다.

```java
@Configuration
public class MySQLAutoconfiguration {
  //..
}
```

그 다음으로 반드시 해야할 작업은 이 클래스를 auto-configuration의 후보자로 등록해야 합니다. @EnableAutoConfiguration의 후보자 목록의 파일은 resources/META-INF/spring.factories에 있습니다.



When SpringBoot app is starting, it will not scan all the classes in jars, So SpringBoot starter should specify which classes are auto-configured.



## 참고자료

https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/#using-boot-auto-configuration

https://www.baeldung.com/spring-boot-custom-auto-configuration

https://docs.spring.io/autorepo/docs/spring-boot/2.0.0.M3/reference/html/boot-features-developing-auto-configuration.html

https://www.baeldung.com/spring-boot-custom-starter

