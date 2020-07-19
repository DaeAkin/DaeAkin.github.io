# ‼️ 스프링 부트의 autoconfiguration



## Auto-configuration

Spring Boot의 auto-configuration은 추가한 jar 파일에 따라 자동적으로 설정을 해줍니다. 예를 들어 HSQLDB가 클래스패스에 존재하면, 데이터베이스의 커넥션을 맺는 bean을 수동으로 구성해주지 않았다면, 자동으로 인메모리 DB로 자동 구성 됩니다.

@EnableAutoConfiguration 또는 @SpringBootApplication 주석을 @Configuration 클래스 중 하나에 추가하면 됩니다.

> @SpringBootApplication 또는 @EnableAutoConfiguration 주석을 하나만 추가해야합니다. 일반적으로 기본 @Configuration 클래스에만 하나를 추가하는 것이 좋습니다.



## Auto-configuration으로 점진적으로 변경하기

언제든지 특정 부분을 auto-configuration으로 변경할 수 있습니다. 예를 들어 DataSource 빈을 사용하고 있다면, 디폴트로 사용되는 임베디드 데이터베이스는 더 이상 사용되지 않습니다.

만약 현재 어느 부분에 auto-configuration이 적용되어있는지 알고 싶다면 애플리케이션을 `--debug` 과 함께 실행시키면 됩니다.  이렇게 함으로써 logger의 debug를 활성화 시킬 수 있어, 콘솔로 확인할 수 있습니다.



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

`@ConditionalOnProperty` 어노테이션은 특정 스프링 환경 프로퍼티 파일의 존재에 여부에따라 설정을 해주는 어노테이션 입니다. prefix와 name 속성을 





```
@PropertySource("classpath:mysql.properties")
public class MySQLAutoconfiguration {
    //...
}
```





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