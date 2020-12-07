## JUnit5

이전 JUnit 버전과 다르게, JUnt5는 세개의 서브 프로젝트로 이루어져 있다.

JUnit5 = JUnit Platform + JUnit Jupiter + JUnit Vintage

JUnit Platform은 JVM에서 테스팅 프레임워크를 실행하는데 기초를 제공한다. 또한 TestEngine API를 제공해 테스팅 프레임워크를 개발할 수 있다.

JUnit Jupiter는 JUnit 5에서 테스트를 작성하고 확장을 하기 위한 새로은 프로그래밍 모델과 확장 모델의 조합이다.

JUnit Vintage는 하위 호완성을 위해 JUnit3과 JUnt4를 기반으로 돌아가는 플랫폼에 TestEngine을 제공해준다.

## 요구사항

JUnit5은 java 8부터 지원하며, 이전 버전으로 작성된 테스트 코드여도 컴파일이 정상적으로 지원된다.



## 테스트 작성해보기

다음은 JUnit Jupiter에서 테스트를 작성하기 위한 최소 조건으로 테스트를 작성한 것이다.

```java
import static org.junit.jupiter.api.Assertions.assertEquals;
import example.util.Calculator; import org.junit.jupiter.api.Test; 
class MyFirstJUnitJupiterTests {

	private final Calculator calculator = new Calculator();
  @Test 
  void addition() {
    assertEquals(2, calculator.add(1, 1)); 
  }
}
```



### 어노테이션

테스트를 구성하고, 프레임워크를 상속하기 위해서 다음과 같은 어노테이션을 지한다.

따로 명시하지 않으면 대부분 junit-jupiter-api 모듈 안의 `org.junit.jupiter.api` 패키지안에 존재한다.

- @DisplayName : 테스트 클래스나 테스트 메소드에 이름을 붙여줄 때 사용한다.

  ![]({{site.url}}/images/junit5/@displayName.png)

  기본적으로 테스트 이름은 메소드이름을 따라간다. 하지만 테스트 이름을 바꾸면 이 어노테이션을 사용한다.

  ![]({{site.url}}/images/junit5/@displayNameEx.png)

- @DisplayNameGeneration : 클래스에 해당 애노테이션을 붙이면 @Test 메소드 이름에 `_`로 표시한 모든 부분은 space로 처리된다.

  ```java 
  @DisplayNameGeneration(DisplayNameGenerator.ReplaceUnderscores.class)
  class SomeTest {}
  ```
  
- @BeforeEach : 각각 테스트 메소드가 실행되기전에 실행되어야 하는 메소드를 명시해준다. `@Test` , `@RepeatedTest` , `@ParameterizedTest` , `@TestFactory` 가 붙은 테스트 메소드가 실행하기 전에 실행된다. JUnit4의 `@Before` 와 같은 역할을 한다. 개인적으로 테스트 하기전에 필요한 목업 데이터를 미리 세팅해주기 위해 주로 사용한다.

- @AfterEach : 위와 같이 해당 어노테이션이 붙은 테스트 메소드가 실행되고 난 후 실행된다. JUnit4의 `@After` 어노테이션과 같은 역할을 한다. 

- @BeforeAll : `@BeforeEach` 는 각 테스트 메소드 마다 실행되지만, 이 어노테이션은 테스트가 시작하기 전 딱 한번만 실행 된다. 

- @AfterAll : 이것도 위와 같지만, 테스트가 완전히 끝난 후 딱 한번만 실행 된다.

- @Nested : test 클래스안에 Nested 테스트 클래스를 작성할 때 사용한다. Denotes that the annotated class is a non-static nested test class. @BeforeAll and @AfterAll methods cannot be used directly in a @Nested test class unless the "per-class" test instance lifecycle is used. Such annotations are not inherited.

- @Tag : 테스트를 필터링할 때 사용한다. 클래스또는 메소드레벨에 사용한다. 

- @Disabled : 테스트 클래스나, 메소드의 테스트를 비활성화 한다. JUnit4의 @Ignore와 같다.

- @Timeout : 주어진 시간안에 테스트가 끝나지 않으면 실패한다.

- @ExtendWith : Used to register extensions declaratively. Such annotations are inherited.

- @RegisterExtension : Used to register extensions programmatically via fields. Such fields are inherited unless they are shadowed.

- @TempDir : 필드 주입이나 파라미터 주입을 통해 임시적인 디렉토리를 제공할 때 사용한다. 

### 

### 메타 어노테이션과 컴포즈 어노테이션

JUnit Jupiter 어노테이션은 메타 어노테이션처럼 사용된다. 그 말은 즉슨 자동으로 메타 어노테이션을 상속하는 자기만의 컴포즈 어노테이션을 정의할 수 있다.

예를 들어 코드에다가 `@Tag("fast")` 를 복사 붙여 넣기 하기보다, 커스텀 컴포즈 어노테이션 `@Fast` 를  만든 다음 `Tag("fast")` 를 대체하여 사용하는 것이다.

```java
@Target({ ElementType.TYPE, ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME) 
@Tag("fast") 
public @interface Fast { }
```

다음과 같이 사용할 수 있다.

```java
@Fast
@Test
void myFastTest() {}
```

더 나아가서 `@Tag("fast")` 와 `@Test` 를 합쳐 @FastTest 로 만들어도 좋다.



## 테스트 클래스와 메소드

**<u>테스트 클래스란</u>** 최상위 클래스나, 스태틱 멤버 클래스나, @Nested 클래스에 적어도 한개의 test 메소드가 포함되있는 걸 말한다. 테스트 클래스는 `abstract` 이면 안되고, 하나의 생성자가 있어야 한다.

**<u>테스트 메소드란</u>** `@Test` ,`@RepeatedTest` ,`@ParamterizedTest`,`@TestFactory` ,`@TestTemplate`  같은 메타 어노테이션이 메소드에 붙여진 메소드를 말한다.

**<u>라이플사이클 메소드란</u>** `@BeforeAll` , `@AfterAll` , `@BeforeEach` , `@AfterEach` 같은 메타 어노테이션이 메소드에 붙여진 메소드를 말한다.

테스트 메소드와 라이플사이클 메소드는 테스트 할 클래스나, 상속한 부모클래스 또는 인터페이스에 선언된다. 추가로 테스트 메소드와 라이프사이클 메소드는 `abstract` 선언하면 안되고, 어떠한 값도 리턴되선 안된다.

> 테스트 클래스, 테스트 메소드, 라이플사이클 메소드는 접근제어자를 `public` 으로 선언을 꼭 안해줘도 됩니다. 그러나 `private` 으로 선언하면 안됩니다.



## Display Names 

테스트 클래스와 테스트 메소드는 `@DisplayName` 을 이용해서 테스트 이름을 변경해줄 수 있다. 공백이나 특수문자나, 이모지도 가능하다.



### Display Name Generators

JUnit Jupiter는 `@DisplayNameGeneration` 어노테이션을 통해 테스트이름을 어떻게 보여줄지 결정할 수 있다.



- Standard : 메소드이름과 그 뒤에 붙는 괄호 그대로 보여준다.
- Simple : 메소드이름만 보여준다.
- ReplaceUnderscores : 언더스코어를 제거한다. `게시판_테스트 ` 를 `게시판 테스트` 로 보여준다.
- IndicativeSentences :  테스트 클래스 이름과 테스트 메소드이름 + 괄호를 보여준다.

### 기본 DisplayName Generator 세팅하기

```
junit.jupiter.displayname.generator.default = \ org.junit.jupiter.api.DisplayNameGenerator$ReplaceUnderscores
```

테스트를 시작할 때 DisplayName 우선순위가 있다.

1. `@DisplayName` 어노테이션 안의 value 값 
2. `DisplayNameGeneration` 어노테이션 안에 있는  `DisplayNameGenerator`  값
3. 설정 파라미터를 통해 설정해준 디폴트 `DisplayNameGenerator` 값



## Assertions

JUnit Jupiter는 JUnit4로부터 온 assertion 메소드와 새롭게 자바 8 람다 표현식으로 추가된 메소드들이 있다. 모든 JUnit Jupiter assertion은 static 메소드이며, `org.junit.jupiter.api.Assertions` 클래스 안에 있다.



```java
class AssertionsTest {

    private final Calculator calculator = new Calculator();
    private final Person person = new Person("동현", "민");

    @Test
    void standardAssertions() {
        assertEquals(2, calculator.add(1, 1));
        assertEquals(4, calculator.multiply(2, 2),
                "추가적인 실패 메세지는 마지막 파라미터에 넣는다.");
        assertTrue('a' < 'b', () -> "Assertion messages는 지연로딩과 비슷하게 동작한다. -- "
                + "불필요하게 메세지를 만드는 일을 피하기 위해서.");
    }

    @Test
    void groupedAssertions() {
        assertAll("person",
                () -> assertEquals("Jane", person.getFirstName()),
                () -> assertEquals("Doe", person.getLastName()));
    }

    @Test
    void exceptionTesting() {
        Exception exception = assertThrows(ArithmeticException.class, () ->
                calculator.divide(1, 0));

        assertEquals("/ by zero", exception.getMessage());
    }

    @Test
    void timeoutNotExceeded() {
        // 아래의 assertion은 성공.
        assertTimeout(ofMinutes(2), () -> {
            // 2분 미만으로 실행되는 동작 실행
        });
    }

    @Test
    void timeoutNotExceededWithResult() {
        //아래의 assertion은 성공하면서 supplied 객체를 반환한다.
        String actualResult = assertTimeout(ofMinutes(2), () -> {
            return "a result";
        });
        assertEquals("a result", actualResult);
    }

    @Test
    void timeoutExceeded() {
        //이 테스트는 fail이 난다.
        assertTimeout(ofMillis(10), () -> {
            // 10ms 이상 걸리는 작업
            Thread.sleep(100);
        });
    }

    @Test
    void timeoutExceededWithPreemptiveTermination() {
        //이 테스트는 fail이 난다.
        assertTimeoutPreemptively(ofMillis(10), () -> {
            // 10ms 이상 걸리는 작업
            new CountDownLatch(1).await();
        });
    }
}
```



> Preemptive Timeout(선점적인 시간초과) 테스트 assertTimeoutPreemptively()
>
> p67쪽 본다음에 정리해야할듯.

### 써드 파티 Assertion 라이브러리

아쉽게도 JUnit Jupiter가 제공해주는 assertion이 많은 테스트 시나리오에서 부족할 수 있다. 그럴 경우 JUnit 팀은 [AssertJ](https://joel-costigliola.github.io/assertj/) , [Hamcrest](https://hamcrest.org/JavaHamcrest/) , [Truth](https://truth.dev/) 등 써드 파티 라이브러리를 쓰는걸 추천해 한다. 

```java
import static org.hamcrest.CoreMatchers.equalTo;
import static org.hamcrest.CoreMatchers.is;
import static org.hamcrest.MatcherAssert.assertThat;

public class HamcrestAssertionsTest {

    private final Calculator calculator = new Calculator();

    @Test
    void HamcrestMatcher_이용하기() {
        assertThat(calculator.subtract(4, 1), is(equalTo(3)));
    }
}
```



## Assumptions

JUnit Jupiter는 JUnit4에서 제공해주던 assumption 메소드와, Java 8 람다 표현식과, 메소드 레퍼런스를 이용해 몇 개를 더 추가 했다.  모든 assumptions 메소드는 `org.junit.jupiter.api.Assumptions` 클래스안에 있다.



```java
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assumptions.assumeTrue;
import static org.junit.jupiter.api.Assumptions.assumingThat;

public class AssumptionTest {
    private final Calculator calculator = new Calculator();

    @Test
    void CI서버에서만_테스트() {
        assumeTrue("CI".equals(System.getenv("ENV")));
    }

    @Test
    void 개발환경에서만_테스트() {
        assumeTrue("DEV".equals(System.getenv("ENV")),
                () -> "Aborting test: not on developer workstation");
    }

    @Test
    void 모든환경_테스트() {
        assumingThat("CI".equals(System.getenv("ENV")), () -> {
            // CI 서버에서만 실행하는 테스트
            assertEquals(2, calculator.divide(4, 2));
        });
        // 모든 환경에서 실행하는 테스트
        assertEquals(42, calculator.multiply(6, 7));
    }
}
```



## 특정 테스트 비활성화하기

테스트 클래스나, 테스트 메소드에 `@Disabled` 어노테이션을 이용해서 비활성화 할 수 있다. 

**클래스에 선언**

```java
@Disabled("이슈 #103 가 해결될 때 까지 비활성화")
public class DisabledClassTest {

    @Test
    void testWillBeSkipped() {
    }
}
```

**메소드에 선언**

```java
public class DisabledTest {
    @Disabled("#42 버그가 해결될 때 까지 비활성화")
    @Test
    void testWillBeSkipped() {
    }

    @Test
    void testWillBeExecuted() {
    }
}
```

`@Disabled` 는 이유를 명시하지 않고 사용해도 되지만, JUnit 팀에서는 왜 이 테스트가 비활성화 되었는지에 대한 짧은 설명을 추가하는 걸 권장하고 있다. 예를 들어 위에 예제에서는 #42 의 버그가 해결될 떄 까지 비활성화 처럼 말이다. 



## 테스트 실행 조건

JUnit Juptier에 있는 `ExecutionCondition` API는 개발자가 특정 조건에 따라 테스트를 실행할지 말지 결정한다.심플한 예로  `@Disabled` 어노테이션을 지원하는 내장된 `DisabledCondition` 이다. JUnit Jupiter는 또한 `@Disabled` 이외에도  개발자가 선언적으로 테스트를 활성화하거나 비활성화 하기 위해 `org.junit.jupiter.api.condition` 패키지에 있는 어노테이션 기반 조건을 지원한다. 여러 개의 `ExecutionCondition` 이 등록되면 여러개 조건중 하나라도 비활성화 조건에 걸리면, 테스트를 비활성화 한다. 이 테스트가 왜 비활성화 되었는지 알려주고 싶다면, 모든 어노테이션은 `disabledReson` 속성을 지정할 수 있으므로, 이걸 이용하며 된다.

### OS 따라 테스트 실행하기

```java
public class ConditionalTest {
    @Test
    @EnabledOnOs(MAC)
    void onlyOnMacOs() {
        // ...
    }

    @TestOnMac
    void testOnMac() {
        // ...
    }

    @Test
    @EnabledOnOs({LINUX, MAC})
    void onLinuxOrMac() {
        // ...
    }

    @Test
    @DisabledOnOs(WINDOWS)
    void notOnWindows() {
        // ...
    }
}

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Test
@EnabledOnOs(MAC)
@interface TestOnMac {
}
```

### 자바 환경변수에 따라 실행하기

특정 JRE 버전에 따라 테스트를 활성화 비활성화 할 수 있습니다. `@EnabledOnJre` 와 `@DisabledOnJre` 어노테이션을 이용해서 활성화할 JRE 버전을 명시하고, `@EnabledForJreRange` 와 `@DisabledForJreRange`  어노테이션을 이용하면 여러개의JRE 버전을 테스트 할 수 있다. 

### 시스템 속성 조건

`@EnabledIfSystemProperty` 와 `@DisabledIfSystemProperty` 어노테이션을 사용하면 named에 작성된 JVM 시스템 속성에 따라 테스트를 활성화 또는 비활성화 할 수 있다. 

```java
public class SystemPropertyConditionalTest {
    @Test
    @EnabledIfSystemProperty(named = "os.arch", matches = ".*64.*")
    void onlyOn64BitArchitectures() {
        // ...
    }

    @Test @DisabledIfSystemProperty(named = "ci-server", matches = "true")
    void notOnCiServer() {
        // ...
    }
}
```

> Junit Jupiter 5.6부터는 `EnabledIfSystemProperty` 와 `DisabledIfSystemProperty` 는 반복가능한 어노테이션으로 변경되었다. 그 결과로, 테스트를 할 때 여러 개를 중첩해서 사용할 수 있다. 

### 환경 변수 조건

`@EnabledIfEnvironmentVariable` 와 `@DisabledIfEnvironmentVariable` 어노테이션을 사용하여 운영체제 시스템의 환경 변수에 따라 테스트를 활성화 또는 비활성화 할 수 있다.

```java
public class EnvironmentVariableConditionalTest {
    @Test
    @EnabledIfEnvironmentVariable(named = "ENV", matches = "staging-server")
    void onlyOnStagingServer() {
        // ...
    }

    @Test
    @DisabledIfEnvironmentVariable(named = "ENV", matches = ".*development.*")
    void notOnDeveloperWorkstation() { 
        // ...
    }
}
```

> JUnit Jupiter 5.6부터 두개의 어노테이션은 반복가능한 어노테이션으로 변경되었다. 그 결과로 테스트 할 때 여러개를 중첩해서 사용할 수 있다.

### 사용자 커스텀 조건

`@EnabledIf` 와 `@DisabledIf` 어노테이션을 사용하여 지정해준 메소드가 반환하는 boolean의 값에 따라 테스트를 활성화 또는 비활성화 할 수 있다. 어노테이션 안에 메소드 이름을 작성주면 되고, 만약 테스트 클래스 밖에 있는 메소드라면, 클래스 까지 써줘야 한다. 필요하다면 메소드는 하나의 파라미터를 갖을 수 있다.

```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.condition.DisabledIf;
import org.junit.jupiter.api.condition.EnabledIf;

public class CustomConditionalTest {
    @Test
    @EnabledIf("customCondition")
    void enabled() {


    }

    @Test
    @DisabledIf("customCondition")
    void disabled() {

    }

    boolean customCondition() {
        return true;
    }
}
```

> `@EnabledIf` 나 `@DisabledIf` 를 클래스 레벨에 사용할 때 컨디션 메소드는 반드시 `static` 으로 선언되어야 한다. 외부 클래스에 위치한 컨디션 메소드도 `static` 으로 선언되어야 한다.

external 클래스에 위치한 메소드는 안되나?



## 태그와 필터링

클래스와 메소드는 `@Tag` 어노테이션을 통해 태그할 수 있다. 이 태그들은 나중에 테스트를 필터링 할때 사용 된다.

### Tags를 사용하기 위한 규칙

- 태그는 공백이나 `null` 이 있으면 안됨
- ISO 제어문자가 있으면 안됨
- 다음과 같은 문자열이 있으면 안됨
  - , 
  - ( 
  - ) 
  - |
  - !
  - &

## 테스트 실행 순서 바꾸기

일반적으로 단위테스트는 테스트 순서에 영향을 받지 않지만, 통합 테스트를 작성할 때나, 테스트의 순서가 중요한 함수형 테스트를 할 때 테스트 실행 순서를 바꾸고 싶을 때가 있다. 

테스트 메소드가 실행되는 순서를 바꾸고 싶으면 테스트 클래스나 메소드에 `@TestMethodOrder` 를 이용하여 `MethodOrderer` 를 원하는대로 구현하면 된다. `MethodOrderer` 를 커스텀해서 구현하거나, 이미 내장된 `MethodOrderer` 구현 중 하나를 사용하면 된다.

- DisplayName : displayName 기반으로 정렬한다.

- MethodName : 메소드 이름으로 정렬한다.

- OrderAnnotation : `@Order` 어노테이션에 명시된 순서대로 정렬한다.

- Random : 랜덤으로 정렬한다.

- Alphanumeric : sorts test methods alphanumerically based on their names and formal parameter

  28 lists.

> MethodName은 6.0에 Deprecated 될 예정임.

```java
import org.junit.jupiter.api.MethodOrderer;
import org.junit.jupiter.api.Order;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.TestMethodOrder;

@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
public class OrderedTest {
    @Test
    @Order(1)
    void nullValues() {
    }

    @Test
    @Order(2)
    void emptyValues() {
    }

    @Test
    @Order(3)
    void validValues() {
    }
}
```



### 메소드 실행 순서 디폴트 설정하기

`junit.jupiter.testmethod.order.default` 설정 파라미터를 이용하여 디폴트로 사용할 `MethodOrderer` 를 설정해줄 수 있다. 테스트 클래스에 `@TestMethodOrder` 가 없으면 모든 테스트들은 디폴트 순서를 사용한다.

예를 들어 `OrderAnnotation ` 클래스를 디폴트로 사용하고 싶다면 클래스 이름 전체를 설정 파라미터에 설정하면 된다.

**src/test/reousrces/junit-platform.properties**

```yaml
junit.jupiter.testmethod.order.default = \
	org.junit.jupiter.api.MethodOrder$OrderAnnotation
```



## 테스트 LifeCycle

사이드 이펙트를 줄이고, 테스트 메서드를 격리된 환경에서 독립적으로 실행시키기 위해 JUnit은 테스트 메소드를 실행시키기 전에 각각의 테스트 클래스의 새로운 인스턴스를 만든다.  이렇게 메소드마다 테스트 라이프사이클을 갖는다.

> 테스트 메소드에 `@Disabled` ,`@DisabledOnOs` 를 붙여 테스트 메소드를 비활성화 해도 똑같은 테스트 라이플 사이클을 가진다

만약 같은 인스턴스에서 모든 테스트 메소드를 모두 실행하고 싶다면 테스트 클래스에 `@TestInstance(Lifecycle.PER_CLASS)` 어노테이션을 사용하면 된다. 이 어노테이션을 사용하면 **<u>테스트 클래스 단위로</u>** 새로운 테스트 인스턴스가 생기게 된다. 그러므로 그 안에 있는 인스턴스 변수를 테스트 메소드들이 공유를 하므로 `@BeforeEach` 나 `@AfterEach` 를 사용하여 내부 상태를 리셋을 시켜줘야 한다.

**PER-CLASS** 는 테스트 인스턴스의 기본 생성 방식인**PER-METHOD** 를 사용할 때와 같은 이점을 얻는다. 특히 PER-CLASS는 `@BeforeAll` 과 `@AfterAll` 를 붙인 메소드에 static을 사용하지 않아도 되고 인터페이스의 `deafult` 메소드에서도 사용하지 않아도 된다. 또한   **PER-CLASS** 는`@Nested` 테스트 클래스에서 `@BeforeAll` 과 `@AfterAll` 메소드를 사용할 수 있게 해준다.

### 디폴트 테스트 인스턴스 라이플사이클 변경하기

테스트 클래스에 `@TestInstance` 어노테이션이 없으면 기본 라이플사이클을 사용한다. 기본 모드는 PER_METHOD 이다. 그러나 전체 테스트에 디폴트 라이프사이클을 변경할 수 있다. 변경하기 위해선 `junit.jupiter.testinstnace.lifecycle.default` 설정 파라미터에 `TestInstnace.LifeCycle` enum클래스를 써주면 된다. JUnit 설정파일이나, LauncherDiscoveryRequest를 Launcher로 전달한 설정 파라미터를 이용해 JVM 시스템 변수로 제공해줄 수 있다. 

예를 들어 라이플사이클 모드를 `LifeCycle_PER_CLASS` 로 변경하고 싶으면 JVM을 실행할 때 다음과 같이 실행한다.

**JVM 시스템 변수 설정**

```
-Djunit.jupiter.testinstance.lifecycle.default=per_class
```

그러나 이것 보단, JUnit 설정파일을 통해 라이플사이클 모드를 변경하는 것이 프로젝트의 버전 관리를 할 수 있기 때문에 좀 더 권장하는 방법이다.

JUnit 설정 파일을 통해 설정하는 방법은, `junit-platform.properties` 이름의 파일을 클래스패스 만들고 다음과 같이 작성한다.

**JUnit 설정 파일 설정**

```
junit.jupiter.testinstance.lifecycle.default = per_class
```

> 디폴트 테스트 인스턴스 라이플사이클 변경이 일관적으로 적용되지 않으면 예상치 못한 결과를 초래 한다. 예를 들어, 빌드 설정엔 "per-class"를 디폴트로 설정했지만, IDE 설정에서는 "per-method"로 설정되어 실행될 수 있다. 이렇게 되면, 빌드 서버에서 오류가 난다. 이런 현상을 해결하기 위해서는 JVM 시스템 변수 대신, JUnit 설정 파일을 사용하는 걸 추천 한다.

## Nested Tests

`@Nested` 테스트는 테스트 그룹 사이의 관계를 표현할 수 있게 해준다.

```java
@DisplayName("A stack")
class TestingStack {

    Stack<Object> stack;

    @Test
    @DisplayName("is instantiated with new Stack()")
    void isInstantiatedWithNew() {
        new Stack<>();
    }

    @Nested
    @DisplayName("when new")
    class WhenNew {
        @BeforeEach
        void createNewStack() {
            stack = new Stack<>();
        }

        @Test
        @DisplayName("is empty")
        void isEmpty() {
            assertTrue(stack.isEmpty());
        }

        @Test
        @DisplayName("throws EmptyStackException when popped")
        void throwsExceptionWhenPopped() {
            assertThrows(EmptyStackException.class, stack::pop);
        }

        @Test
        @DisplayName("throws EmptyStackException when peeked")
        void throwsExceptionWhenPeeked() {
            assertThrows(EmptyStackException.class, stack::peek);
        }

        @Nested
        @DisplayName("after pushing an element")
        class AfterPushing {
            String anElement = "an element";

            @BeforeEach
            void pushAnElement() {
                stack.push(anElement);
            }

            @Test
            @DisplayName("it is no longer empty")
            void isNotEmpty() {
                assertFalse(stack.isEmpty());
            }

            @Test
            @DisplayName("returns the element when popped and is empty")
            void returnElementWhenPopped() {
                assertEquals(anElement, stack.pop());
                assertTrue(stack.isEmpty());
            }

            @Test
            @DisplayName("returns the element when peeked but remains not empty")
            void returnElementWhenPeeked() {
                assertEquals(anElement, stack.peek());
                assertFalse(stack.isEmpty());
            }
        }
    }
}
```

오직 non-static인 nested 클래스(예를 들어 inner 클래스)만 `@Nested` 를 붙일 수 있다. 

Only non-static nested classes (i.e. inner classes) can serve as @Nested test classes. Nesting can be arbitrarily deep, and those inner classes are considered to be full members of the test class family with one exception: @BeforeAll and @AfterAll methods do not work by default. The reason is that Java does not allow static members in inner classes. However, this restriction can be circumvented by annotating a @Nested test class with @TestInstance(Lifecycle.PER_CLASS) (see Test Instance Lifecycle).

## 생생자와 메소드 의존성 주입

이전 JUnit 버전들에서는 테스트 생성자나 메소드에 파라미터를 갖지 못하게 했다. JUnit juptier의 주요 변화로 테스트 생성자와 메소드가 이제는 파라미터를 갖을 수 있도록 변경되었다. 이런 변화는 코드의 유연성과 생성자와 메소드에 의존성 주입을 가능하게 해준다. 

**ParameterResolver** 는 런타임시 동적으로 파라미터를 결정하는 테스트 익스텐션에 관한 API가 정의되어 있다. 테스트 클래스 생성자나, 테스트메소드나, 라이플사이클 메소드가 파라미터를 받고 싶다면, 파라미터는 `ParameterResolver` 를 등록함으로써 런타임시 결정을 해야 한다.

현재 자동적으로 등록되는 3개의 내장된 리졸버들이 있다.

- [TestInfoParameterResolver](https://github.com/junit-team/junit5/blob/r5.7.0/junit-jupiter-engine/src/main/java/org/junit/jupiter/engine/extension/TestInfoParameterResolver.java) : 생성자나 메소드 파라미터가 TestInfo의 타입이면 TestInfoParameterResolver가  현재 컨테이너나, 테스트에 일치하는 값을 TestInfo 인스턴스로 제공해준다. 제공받은 TestInfo는 현재 컨테이너 또는 테스트에 관한 display name, 테스트클래스, 테스트메소드, 관련된 태그들의 테스트 정보를 가져올 때 사용한다. 

  다음의 예제는 테스트 생성자와, `@BeforeEach` 메소드, `@Test` 메소드에 어떻게 **TestInfo** 가 주입되는지 보여준다.

  ```java
  @DisplayName("TestInfo Demo Test")
  class TestInfoTest {
      TestInfoTest(TestInfo testInfo) {
          assertEquals("TestInfo Demo Test", testInfo.getDisplayName());
      }
  
      @BeforeEach
      void init(TestInfo testInfo) {
          String displayName = testInfo.getDisplayName();
          assertTrue(displayName.equals("TEST 1") || displayName.equals("test2()"));
      }
  
      @Test
      @DisplayName("TEST 1")
      @Tag("my-tag")
      void test1(TestInfo testInfo) {
          assertEquals("TEST 1", testInfo.getDisplayName());
          assertTrue(testInfo.getTags().contains("my-tag"));
      }
  
      @Test
      void test2() {
      }
  
  }
  ```

- RepetitionInfoParameterResolver : `@RepeatedTest` ,`@BeforeEach` ,`@AfterEach` 의 어노테이션이 붙은 메소드 파라미터는 RepetitionInfo의 타입이며, RepetitionInfoParameterResolver가 RepetitionInfo 인스턴스를 제공해준다. **RepetitionInfo**는 현재 반복하고 있는 정보나, `@RepeatedTest` 와 관련된 반복의 총 갯수의 정보를 가져올 때 사용한다. 그러나 현재 컨텍스트 외부에 있는 `@RepeatedTest` 를 찾아내진 못한다.

- TestReporterParameterResolver : TestReporter 타입의 파라미터를 사용해야할 때 사용한다. TestReporter는 현재 실행중인 테스트에 관한 추가적인 데이터를 발행해야할 때 사용한다.  데이터는 TestExecutionListener안에 있는 reportingEntryPublished() 메소드를 이용해 컨슘되며, IDE나 리포트에서 볼 수 있다. -- 안보인다. 나중에 다시한번확인

다른 리졸버를 사용하고 싶으면, `@ExtendWith` 을 통해서 상속을 하면 된다.

```java
ExtendWith(RandomParametersExtension.class)
class MyRandomParameterTest {

    @Test
    void injectsInteger(@Random int i, @Random int j) {

        assertNotEquals(i, j);
    }

    @Test
    void injectsDouble(@Random double d) {
        assertEquals(0.0, d, 1.0);
    }

}
```

>   RandomParametersExtension은 아직 정식출시가 안되었다.

실제 사용 사례로는 MockitoExtension과 SpringExtension을 많이 쓴다.



## 테스트 인터페이스와 디폴트 메소드

@Test, @RepeatedTest, @ParameterizedTest , @TestFactory, @TestTemplate, @BeforeEach, @AfterEach는 인터페이스 디폴트 메소드에 선언을 해도 된다.만약 테스트 인터페이스나 테스트 클래스에 @TestInstnace(Lifecycle.PER_CLASS)로 되어 있다면  @BeforeAll과 @AfterAll은 static으로 인터페이스 디폴트 메소드와 테스트 인터페이스에 선언해도 된다. 

```java
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
interface TestLifecycleLogger {

    static final Logger logger = Logger.getLogger(TestLifecycleLogger.class.getName());

    @BeforeAll
    default void beforeAllTests() {
        logger.info("Before all tests");
    }

    @AfterAll
    default void afterAllTests() {
        logger.info("After all tests");
    }

    @BeforeEach
    default void beforeEachTest(TestInfo testInfo) {
        logger.info(() -> String.format("About to execute [%s]",
                testInfo.getDisplayName()));
    }

    @AfterEach
    default void afterEachTest(TestInfo testInfo) {
        logger.info(() -> String.format("Finished executing [%s]",
                testInfo.getDisplayName()));
    }
}
```



```java
interface TestInterfaceDynamicTestsDemo {
    @TestFactory
    default Stream<DynamicTest> dynamicTestsForPalindromes() {
        return Stream.of("racecar", "radar", "mom", "dad")
                .map(text -> dynamicTest(text, () -> assertTrue(text,equals(text)))); }
}
```

@ExtendWith과 @Tag는 테스트 인터페이스에 선언할 수 있어서 이 테스트 인터페이스를 선언한 구현체는 자동으로 tag와 extension을 상속 받는다.

나중에 다시정리해야겠따. 40p.

## @Repeated Test 

@RepeatedTest 어노테이션에 명시된 숫자로 테스트를 얼마나 반복적으로 실행할지 지정해줄 수 있다. 반복 테스트의 호출은 보통의 @Test 메소드들이랑 똑같이 동작한다.

다음의 예제는 repeatedTest()가 10번 호출된다.

```java
@RepeatedTest(10) 
void repeatedTest() { 
}
```

또한 반복의 수를 지정해 줄 때, 각각의 반복되는 테스트에 대해서 보여줄 display name도 설정할 수 있다. 게다가, display name은 스트링과 정적 placeholder로 조합해서 패턴을 만들 수 있다.  다음은 지원되는 placeholder 이다.

- DisplayName : @RepeatedTest 메소드의 보여줄 이름
- {currentRepetition} : 현재 반복 횟수
- {totalRepetitions} : 반복할 총 횟수

반복을 돌 때 보여지는 기본 display name은 다음과 같은 패턴으로 만들어진다.

**"repetition {currentRepetition} of {totalRepetitions}"**

그래서 위에 있는 repeatedTest()는 "repetition 1 of 10, repetition 2 of 10" 이렇게 보여진다. 메소드 이름을 포함하고 싶으면 사전 정의된, RepeatedTest.LONG_DISPLAY_NAME을 사용하면 된다. 

현재 반복 정보와 반복의 수를 구하기 위해서 개발자가 @RepeatedTest ,@BeforeEach ,@AfterEach 메소드에 **RepetitionInfo의** 인스턴스를 주입시켜야 한다.

### 반복 테스트 예제

```java
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.*;

import static org.junit.jupiter.api.Assertions.assertEquals;

@Slf4j
public class RepeatTest2 {

    @BeforeEach
    void beforeEach(TestInfo testInfo, RepetitionInfo repetitionInfo) {
        int currentRepetition = repetitionInfo.getCurrentRepetition();
        int totalRepetitions = repetitionInfo.getTotalRepetitions();
        String methodName = testInfo.getTestMethod().get().getName();
        log.info(String.format("About to execute repetition %d of %d for %s",
                currentRepetition, totalRepetitions, methodName));
    }

    @RepeatedTest(10)
    void repeatedTest() {
    }

    @RepeatedTest(5)
    void repeatedTestWithRepetitionInfo(RepetitionInfo repetitionInfo) {
        assertEquals(5, repetitionInfo.getTotalRepetitions());
    }

    @RepeatedTest(value = 1, name = "{displayName} {currentRepetition}/{totalRepetitions}")
    @DisplayName("Repeat!")
    void customDisplayName (TestInfo testInfo) {
        assertEquals("Repeat! 1/1", testInfo.getDisplayName());
    }

    @RepeatedTest(value = 1, name = RepeatedTest.LONG_DISPLAY_NAME)
    @DisplayName("Details...")
    void customDisplayNameWithLongPattern(TestInfo testInfo) {
        assertEquals("Details... :: repetition 1 of 1", testInfo.getDisplayName());
    }

    @RepeatedTest(value = 5, name = "Wiederholung {currentRepetition} von {totalRepetitions}")
    void repeatedTestInGerman() { // ...

    }
}
```



Console 결과

```
INFO: About to execute repetition 1 of 10 for repeatedTest
INFO: About to execute repetition 2 of 10 for repeatedTest
INFO: About to execute repetition 3 of 10 for repeatedTest
INFO: About to execute repetition 4 of 10 for repeatedTest
INFO: About to execute repetition 5 of 10 for repeatedTest
INFO: About to execute repetition 6 of 10 for repeatedTest
INFO: About to execute repetition 7 of 10 for repeatedTest 
INFO: About to execute repetition 8 of 10 for repeatedTest
INFO: About to execute repetition 9 of 10 for repeatedTest 
INFO: About to execute repetition 10 of 10 for repeatedTest
INFO: About to execute repetition 1 of 5 for repeatedTestWithRepetitionInfo
INFO: About to execute repetition 2 of 5 for repeatedTestWithRepetitionInfo
INFO: About to execute repetition 3 of 5 for repeatedTestWithRepetitionInfo
INFO: About to execute repetition 4 of 5 for repeatedTestWithRepetitionInfo
INFO: About to execute repetition 5 of 5 for repeatedTestWithRepetitionInfo 
INFO: About to execute repetition 1 of 1 for customDisplayName 
INFO: About to execute repetition 1 of 1 for customDisplayNameWithLongPattern 
INFO: About to execute repetition 1 of 5 for repeatedTestInGerman 
INFO: About to execute repetition 2 of 5 for repeatedTestInGerman 
INFO: About to execute repetition 3 of 5 for repeatedTestInGerman 
INFO: About to execute repetition 4 of 5 for repeatedTestInGerman 
INFO: About to execute repetition 5 of 5 for repeatedTestInGerman
```



테스트 이름 결과

```
├─ RepeatedTestsDemo ✔
│ ├─ repeatedTest() ✔
│ │ ├─ repetition 1 of 10 ✔
│ │ ├─ repetition 2 of 10 ✔
│ │ ├─ repetition 3 of 10 ✔
│ │ ├─ repetition 4 of 10 ✔
│ │ ├─ repetition 5 of 10 ✔
│ │ ├─ repetition 6 of 10 ✔
│ │ ├─ repetition 7 of 10 ✔
│ │ ├─ repetition 8 of 10 ✔ 
│ │ ├─ repetition 9 of 10 ✔ 
│ │ └─ repetition 10 of 10 ✔
│ ├─ repeatedTestWithRepetitionInfo(RepetitionInfo) ✔
│ │ ├─ repetition 1 of 5 ✔
│ │ ├─ repetition 2 of 5 ✔
│ │ ├─ repetition 3 of 5 ✔
│ │ ├─ repetition 4 of 5 ✔
│ │ └─ repetition 5 of 5 ✔
│ ├─ Repeat! ✔
│ │ └─ Repeat! 1/1 ✔
│ ├─ Details... ✔
│ │ └─ Details... :: repetition 1 of 1 ✔
│ └─ repeatedTestInGerman() ✔
│ ├─ Wiederholung 1 von 5 ✔
│ ├─ Wiederholung 2 von 5 ✔
│ ├─ Wiederholung 3 von 5 ✔
│ ├─ Wiederholung 4 von 5 ✔
│ └─ Wiederholung 5 von 5 ✔
```

## 파라미터화 테스트

파라미터화 테스트는 각각 다른 인자로 여러 번 테스트를 돌린다. @Test 대신 @ParameterizedTest 어노테이션을 사용하면 된다. 추가적으로 호출 시 사용될 인자를 적어도 하나는 적어줘야 한다. 

```java
@ParameterizedTest
@ValueSource(strings = {"racecar", "radar", "able was I ere I saw elba"})
void palindromes(String candidate) {
	assertTrue(StringUtils.isPalindrome(candidate));
}
```

이 테스트코드를 실행하면 콘솔에는 다음과 같이 찍힌다.

```
palindromes(String) ✔
├─ [1] candidate=racecar ✔
├─ [2] candidate=radar ✔
└─ [3] candidate=able was I ere I saw elba ✔
```

> 파라미터화 테스트는 현재 실험단계다.

### 필수 셋업

파라미터화 테스트를 사용하려면 [junit-jupiter-params](https://mvnrepository.com/artifact/org.junit.jupiter/junit-jupiter-params) 를 추가해야한다.

### 인자 사용하기

파라미터화 테스트 메소드는 일반적으로 미리 정해놓은 파라미터 소스들과 1:1 매칭이 된다. 그러나 

Parameterized test methods typically consume arguments directly from the configured source (see Sources of Arguments) following a one-to-one correlation between argument source index and method parameter index (see examples in @CsvSource). However, a parameterized test method may also choose to aggregate arguments from the source into a single object passed to the method (see Argument Aggregation). Additional arguments may also be provided by a ParameterResolver (e.g., to obtain an instance of TestInfo, TestReporter, etc.). Specifically, a parameterized test method must declare formal parameters according to the following rules.

- 인덱스화된 인자들은 반드시 처음에 선언되어야 한다.
- aggregator는 그 다음에 선언되어야 한다.
- ParameterResolver에 의해 제공되는 인자는 반드시 마지막에 선언되어야 한다.

In this context, an indexed argument is an argument for a given index in the Arguments provided by an ArgumentsProvider that is passed as an argument to the parameterized method at the same index in the method’s formal parameter list. An aggregator is any parameter of type ArgumentsAccessor or any parameter annotated with @AggregateWith.

### 인자 제공하기

JUnit Jupiter는 몇 개의 source 어노테이션을 제공한다. 

**@ValueSource**

@ValueSource는 가장 심플한 소스다. 간단하게 하나의 배열로 값을 정의하며, 하나의 인자만 받는 파라미터화 테스트에만 적용할 수 있다.

다음은 @ValueSource가 지원하는 타입의 목록이다.

- Short
- byte
- int
- long
- float
- double
- char
- boolean
- java.lang.String

예를 들어 다음의 @ParameterizedTest 메소드는 3번 호출 된다.

```java
@ParameterizedTest 
@ValueSource(ints = { 1, 2, 3 }) 
void testWithValueSource(int argument) {
	assertTrue(argument > 0 && argument < 4); 
}
```

**Null and Empty Sources**

잘못된 인풋이 들어올 수 있는 코너케이스(corner 케이스가 뭐지?) 확인하고 적절한 행동을 검증하기 위해서 파라미터화 테스트에 null 또는 빈 값을 제공해주는 것이 유용하다. 다음의 어노테이션은 null 과 빈 값을 제공해준다.

- @NullSource : @ParameterizedTest 메소드에 null을 제공한다.
- @EmptySource : 다음과 같은 타입 String ,List ,Set, Map, 프리미티브 배열 예) int[] ,char[] ..., 객체 배열 예) String[] ,Intger[] ... 같은 인자에 빈값을 제공한다.
  - 지원되는 타입중에 서브타입들은 지원하지 않는다.
- @NullAndEmptySource : 