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
- @NullAndEmptySource : @NullSource 와 @EmptySource 기능들을 합친 어노테이션이다.

만약 파라미터화 테스트로 다양한 빈 스트링을 제공해주고 싶다면 다음과 같이 사용하면 된다.

```java
@ValueSource(Strings = {" ", "    ", "\t", "\n"})
```

또한 null , 빈 값 , 빈 입력을 테스트 하고 싶으면, @NullSource, @EmptySoucre, @ValueSouce를 혼합해서 사용하면 된다.

```java
@ParameterizedTest 
@NullSource 
@EmptySource 
@ValueSource(strings = { " ", " ", "\t", "\n" }) 
void nullEmptyAndBlankStrings(String text) {
  assertTrue(text == null || text.trim().isEmpty()); 
}
```

이걸 더 요약하면 다음과 같이 작성하면 된다.

```java
@ParameterizedTest 
@NullAndEmptySource
@ValueSource(strings = { " ", " ", "\t", "\n" }) 
void nullEmptyAndBlankStrings(String text) {
  assertTrue(text == null || text.trim().isEmpty()); 
}
```

> 위에 있는 nullEmptyAndBlankString(String) 파라미터화 테스트는 6번의 호출이 이루어진다. null과 빈 스트링, 그리고 @ValueSoucre에 적힌 4개의 String들이다.

**@EnumSource**

@EnumSource는 Enum을 편리하게 사용하게 해 준다.

```java
@ParameterizedTest
@EnumSource(ChronoUnit.class)
void testWithEnumSource(TemporalUnit unit) {
  assertNotNull(unit);
}
```

value 속성은 선택이다. 생략하면 메소드에 선언된 첫번 째 타입 파라미터가 사용된다. enum 타입을 참조하지 않으면 테스트는 실패 한다. 위에 예제는 메소드 파라미터가 `TemporalUnit` 으로 선언되어 있기 때문에 value 속성이 필요하다. i.e. the interface implemented by ChronoUnit, which isn’t an enum type. Changing the method parameter type to ChronoUnit allows you to omit the explicit enum type from the annotation as follow

```java
@ParameterizedTest 
@EnumSource 
void testWithEnumSourceWithAutoDetection(ChronoUnit unit) {
	assertNotNull(unit); 
}
```



이 어노테이션은 어떤 상수를 사용할지 지정해주는 추가적인 name 속성을 사용할 수 있다. 생략하면 모든 상수가 사용된다.

```java
@ParameterizedTest
@EnumSource(names = { "DAYS", "HOURS" })
void testWithEnumSourceInclude(ChronoUnit unit) {
  assertTrue(EnumSet.of(ChronoUnit.DAYS, ChronoUnit.HOURS).contains(unit)); 
}
```

@EnumSource 어노테이션은 mode 라는 추가적인 속성을 제공하는데, 이 속성은 어떤 상수를 테스트 메소드에 넘길지, 섬세하게 조절할 수 있는 기능이 있다. 예를 들어, enum 상수 풀 이나 특정 정규식에서 다음과 같이 특정 name만을 제외할 수 있다.

```java
@ParameterizedTest 
@EnumSource(mode = EXCLUDE, names = { "ERAS", "FOREVER" })
void testWithEnumSourceExclude(ChronoUnit unit) { 
  assertFalse(EnumSet.of(ChronoUnit.ERAS, ChronoUnit.FOREVER).contains(unit)); 
}
```



```java
@ParameterizedTest 
@EnumSource(mode = MATCH_ALL, names = "^.*DAYS$")
void testWithEnumSourceRegex(ChronoUnit unit) {
  assertTrue(unit.name().endsWith("DAYS")); 
}
```



**@MethodSource**

@MethodSource는 하나 이상의 테스트 클래스 또는 외부 클래스 팩토리 메소드를 참조할 수 있다.

 테스트 클래스 안에 있는 팩토리 메서드는 `@TestInstance(Lifecycle.PER_CLASS)` 어노테이션을 테스트 클래스에 붙이지 않았으면, 반드시 static을 붙여줘야 하며 외부 클래스에 있는 팩토리 메서드는 반드시 static을 붙여줘야 한다. 게다가 이런 팩토리 메서드는 어떠한 인자도 갖지 않는다.

각각의 팩토리 메서드는 반드시 인자의 stream을 만들며, stream안의 각각의 인자들은 @ParameterizedTest 가 붙은 메소드의 독립적인 호출에 대하여 물리적인 인자를 제공한다. 일반적으로 말하면 인자의 Stream으로 번역해주는 것이지만, 실제 구체적인 반환 타입은 다양한 형태를 가질 수 있다. 여기서의 stream은 JUnit이 Stream으로 변환할 수 있는 어떤 것이든 해당한다, 예를 들어, Stream, DoubleStream, LongStream, IntStream, Collection, Iterator, Iterable 객체배열, 원시배열 등이다. stream안에 "인자"는 인자의 인스턴스로 제공될 수 있다. an array of objects (e.g., Object[]), or a single value if the parameterized test method accepts a single argument.

만약 하나의 파라미터만 필요하다면 다음과 같이 파라미터 타입의 인스턴스의 Stream을 리턴한다.

```java
@ParameterizedTest 
@MethodSource("stringProvider") 
void testWithExplicitLocalMethodSource(String argument) {
  assertNotNull(argument); 
}

static Stream<String> stringProvider() {
  return Stream.of("apple", "banana"); 
}
```

@MethodSource를 통해서 팩토리 메소드 이름을 명시적으로 제공해주지 않으면, @ParamterizedTest 메소드가 붙은 현재 테스트 메소드 이름을 기준으로 팩토리 메소드를 찾는다.

```java
@ParameterizedTest 
@MethodSource void testWithDefaultLocalMethodSource(String argument) { 	
  assertNotNull(argument); 
}

static Stream<String> testWithDefaultLocalMethodSource() { 
  return Stream.of("apple", "banana"); 
}
```

다음과 같이 DoubleStream, IntStream, LongStream의 프리미트 타입의 Stream도 지원한다

```java
@ParameterizedTest 
@MethodSource("range") void testWithRangeMethodSource(int argument) {
  assertNotEquals(9, argument); 
}

static IntStream range() {
  return IntStream.range(0, 20).skip(10); 
}
```

파라미터화 테스트 메소드가 여러개의 파라미터를 갖고 있으면 collection , stream , 인자인스턴스의 배열, 객체 배열을 리턴해야 한다. 

arguments(Obejct)는 Arguments 인터페이스에 정의된 정적 팩토리 메소드 이다. arguments(Object)의 대안으로 Arguments.of(Object)를 사용할 수도 있다.

```java
@ParameterizedTest 
@MethodSource("stringIntAndListProvider")
void testWithMultiArgMethodSource(String str, int num, List<String> list) {
	assertEquals(5, str.length());
	assertTrue(num >=1 && num <=2);
	assertEquals(2, list.size()); 
}

static Stream<Arguments> stringIntAndListProvider() { 
  return Stream.of(
    arguments("apple", 1, Arrays.asList("a", "b")),
    arguments("lemon", 2, Arrays.asList("x", "y")) 
  ); 
}
```

외부에 있는 정적 팩토리 메소드를 사용하려면 메소드 이름을 구별할 수 있게 다음과 같이 전체를 적어줘야한다.

```java
import java.util.stream.Stream; 
import org.junit.jupiter.params.ParameterizedTest; 
import org.junit.jupiter.params.provider.MethodSource;
class ExternalMethodSourceDemo {

@ParameterizedTest 
@MethodSource("example.StringsProviders#tinyStrings") 
  void testWithExternalMethodSource(String tinyString) { 
    // test with tiny string 
  }
}

class StringsProviders {
  
  static Stream<String> tinyStrings() {
    return Stream.of(".", "oo", "OOO"); 
  } 
}
```



**@CsvSource**

@CsvSource는 리스트를 콤마(,)로 구분 해 준다.

```java
@ParameterizedTest
@CsvSource({
  "apple, 1",
  "banana, 2",
  "'lemon, lime', 0xF1" 
})
void testWithCsvSource(String fruit, int rank) {
  assertNotNull(fruit);
	assertNotEquals(0, rank); 
}
```

기본 구분자는 콤마를 사용하지만, delimiter 속성을 이용해서 다른 문자를 기본 구분자로 사용할 수도 있다. 또한 대안적으로 delimiterString 속성을 쓰면 문자 대신 문자열로 구분자를 사용할 수 있다. 그러나 delimiter 속성과 delimiterString을 동시에 사용하면 안된다.

@CsvSource uses a single quote ' as its quote character. See the 'lemon, lime' value in the example above and in the table below. An empty, quoted value '' results in an empty String unless the emptyValue attribute is set; whereas, an entirely empty value is interpreted as a null reference. By specifying one or more nullValues, a custom value can be interpreted as a null reference (see the NIL example in the table below). An ArgumentConversionException is thrown if the target type of a null reference is a primitive type.

| Example Input                                                | Resulting Argument List |
| ------------------------------------------------------------ | ----------------------- |
| @CsvSource({ "apple, banana" })                              | "apple", "banana"       |
| @CsvSource({ "apple, 'lemon, lime'" })                       | "apple", "lemon, lime"  |
| @CsvSource({ "apple, ''" })                                  | "apple", ""             |
| @CsvSource({ "apple, " })                                    | "apple", null           |
| @CsvSource(value = { "apple, banana, NIL" }, nullValues = "NIL") | "apple", "banana", null |

**@CsvFileSource**

@CsvFileSource는 클래스패스나 로컬 파일 시스템에 있는 CSV 파일을 사용합니다. CSV 파일에 있는 각각의 라인의 결과로 파라미터화 테스트에서는 오직 한번만 호출됩니다.

기본 구분자는 콤마(,)지만, delimiter 속성을 설정해서 다른 문자를 사용할 수 있습니다. 또한 대안적으로 delimiterString 속성을 쓰면 문자 대신 문자열로 구분자를 사용할 수 있다. 그러나 delimiter 속성과 delimiterString을 동시에 사용하면 안된다.

> CSV 파일안에 있는 #기호는 주석으로 처리됩니다.



```java
@ParameterizedTest 
@CsvFileSource(resources = "/two-column.csv", numLinesToSkip = 1) 
void testWithCsvFileSourceFromClasspath(String country, int reference) {
  assertNotNull(country);
	assertNotEquals(0, reference); 
}

@ParameterizedTest 
@CsvFileSource(files = "src/test/resources/two-column.csv", numLinesToSkip = 1) 
void testWithCsvFileSourceFromFile(String country, int reference) {
	assertNotNull(country);
	assertNotEquals(0, reference); 
}
```

**two-column.csv**

```
Country, reference
Sweden, 1 
Poland, 2
"United States of America", 3
```

In contrast to the syntax used in @CsvSource, @CsvFileSource uses a double quote " as the quote character. See the "United States of America" value in the example above. An empty, quoted value "" results in an empty String unless the emptyValue attribute is set; whereas, an entirely empty value is interpreted as a null reference. By specifying one or more nullValues, a custom value can be interpreted as a null reference. An ArgumentConversionException is thrown if the target type of a null reference is a primitive type.

> An unquoted empty value will always be converted to a null reference regardless of any custom values configured via the nullValues attribute.



**@ArgumentsSource**

@ArgumentsSource는 커스텀 또는 재사용가능한 ArgumentsProvider를 지정해 줄 때 사용한다. ArgumentsProvider의 구현은 반드시 클래스 탑 레벨에 선언하거나, 정적 중첩 클래스에 선언해야 한다. 

```java
@ParameterizedTest 
@ArgumentsSource(MyArgumentsProvider.class) 
void testWithArgumentsSource(String argument) {
  assertNotNull(argument); 
}
```



```java
public class MyArgumentsProvider implements ArgumentsProvider {

	@Override 
  public Stream<? extends Arguments> provideArguments(ExtensionContext context) { 		
    return Stream.of("apple", "banana").map(Arguments::of); 
	}
}
```



#### **인자 변환**

**Widening 변환**

@ParaterizedTest에 제공된 인자들은 Widening Primitive 변환을 지원한다. 예를 들어 @ValueSoue(ints = {1, 2, 3})이 적힌 파라미터화 테스트에서 int 타입 뿐만 아니라, long, float, double 타입들을 받을 수 있다. 

**묵시적 변환**

@CsvSource 같은 경우를 지원하기 위해 JUnit Jupiter는 내장된 많은 묵시적 변환기를 지원한다. 변환 프로세스는 각각의 메소드 파라미터에 선언된 타입에 달려있다. 

예를 들어 만약 @ParameterizedTest가 TimeUnit 파라미터가 선언되어 있고, 외부에서 String 값으로 파라미터를 공급해준다고 했을 때, 이 공급되는 String은 자동적으로 일치하는 TimeUnit enum 상수로 변환된다. 

```java
@ParameterizedTest 
@ValueSource(strings = "SECONDS") 
void testWithImplicitArgumentConversion(ChronoUnit argument) { 						   			             assertNotNull(argument.name()); 
}   
```

String 인스턴스는 묵시적으로 ChronoUnit인 타켓 타입으로 묵시적으로 변환된다.

> 10진법, 16진법, 8진법 String 문자들은,byte,short,int,long 등 대응하는 타입으로 변환 된다.



| Target Type      | Example                           |
| ---------------- | --------------------------------- |
| boolean/ Boolean | "true" → true                     |
| byte/Byte       | "15", "0xF", or "017" → (byte) 15 |
| char/Character  | "o" → 'o'                         |
| short/Sh ort               | "15", "0xF", or "017" → (short) 15                           |
| int/Inte ger               | "15", "0xF", or "017" → 15                                   |
| long/Lon g                 | "15", "0xF", or "017" → 15L                                  |
| float/Fl oat               | "1.0" → 1.0f                                                 |
| double/D ouble             | "1.0" → 1.0d                                                 |
| Enumsubclass               | "SECONDS" → TimeUnit.SECONDS                                 |
| java.io. File              | "/path/to/file" → new File("/path/to/file")                  |
| java.lan g.Class           | "java.lang.Integer" → java.lang.Integer.class (use $ for nested classes, e.g. "java.lang.Thread$State") |
| java.lan g.Class           | "byte" → byte.class (primitive types are supported)          |
| java.lan g.Class           | "char[]" → char[].class (array types are supported)          |
| java.mat h.BigDec imal     | "123.456e789" → new BigDecimal("123.456e789")                |
| java.mat h.BigInt eger     | "1234567890123456789" → new BigInteger("1234567890123456789") |
| java.net .URI              | "https://junit.org/" → URI.create("https://junit.org/")      |
| java.net .URL              | "https://junit.org/" → new URL("https://junit.org/")         |
| java.nio .charset .Charset | "UTF-8" → Charset.forName("UTF-8")                           |
| java.nio .file.Pa th       | "/path/to/file" → Paths.get("/path/to/file")                 |
| java.tim e.Durati on       | "PT3S" → Duration.ofSeconds(3)                               |
| java.tim e.Instan t        | "1970-01-01T00:00:00Z" → Instant.ofEpochMilli(0)             |
| java.tim e.LocalD ateTime  | "2017-03-14T12:34:56.789" → LocalDateTime.of(2017, 3, 14, 12, 34, 56, 789_000_000) |
| java.tim e.LocalD ate      | "2017-03-14" → LocalDate.of(2017, 3, 14)                     |
| java.tim e.LocalT ime      | "12:34:56.789" → LocalTime.of(12, 34, 56, 789_000_000)       |
| java.tim e.MonthD ay       | "--03-14" → MonthDay.of(3, 14)                               |
| java.tim e.Offset DateTime | "2017-03-14T12:34:56.789Z" → OffsetDateTime.of(2017, 3, 14, 12, 34, 56, 789_000_000, ZoneOffset.UTC) |
| java.tim e.Offset Time     | "12:34:56.789Z" → OffsetTime.of(12, 34, 56, 789_000_000, ZoneOffset.UTC) |
| java.tim e.Period          | P2M6D" → Period.of(0, 2, 6)"                                 |
| java.tim e.YearMo nth      | "2017-03" → YearMonth.of(2017, 3)                            |
| java.tim e.Year            | "2017" → Year.of(2017)                                       |
| java.tim e.ZonedD ateTime  | "2017-03-14T12:34:56.789Z" → ZonedDateTime.of(2017, 3, 14, 12, 34, 56, 789_000_000, ZoneOffset.UTC) |
| java.tim e.ZoneId          | Europe/Berlin" → ZoneId.of("Europe/Berlin")"                 |
| java.tim e.ZoneOf fset     | "+02:30" → ZoneOffset.ofHoursMinutes(2, 30)                  |
| java.uti l.Curren cy       | "JPY" → Currency.getInstance("JPY")                          |
| java.uti l.Locale          | en" → new Locale("en")"                                      |
| java.uti l.UUID            | "d043e930-7b3b-48e3-bdbe-5a3ccfb833db" → UUID.fromString("d043e930-7b3b-48e3-bdbe- 5a3ccfb833db") |

**Fallback String to-Object 변환** 

또한 위에 있는 테이블에 있는 것 처럼 String 타입을 변환하려는 대상 타입으로 묵시적으로 변환할 수 있다. JUnit Jupiter는 만약 대상 타입이 아래에 적힌 것 처럼 정확히 하나의 팩토리 메소드 또는 팩토리 생성자에 알맞은 경우를 위해 String 타입을 자동적으로 변환해주는 fallback 메카니즘을 제공한다.

- factory method : 접근자가 private가 아니여야함, static method declared in the target type that accepts a single String argument and returns an instance of the target type. The name of the method can be 55 arbitrary and need not follow any particular convention.
- factory constructor : a non-private constructor in the target type that accepts a single String argument. Note that the target type must be declared as either a top-level class or as a static nested class

> If multiple factory methods are discovered, they will be ignored. If a factory method and a factory constructor are discovered, the factory method will be used instead of the constructor.

아래 예제에서, @ParameterizedTest 메소드에서, Book 인자는 Book.fromTitle(String) 팩토리 메소드가 호출될 때 생성되며, title의 값으로 "42 Cats" 전달 된다.

```java
@ParameterizedTest 
@ValueSource(strings = "42 Cats") 
void testWithImplicitFallbackArgumentConversion(Book book) { 
	assertEquals("42 Cats", book.getTitle()); 
}
```

**Book.java**

```java
public class Book {

  private final String title;

  private Book(String title) { 
    this.title = title; 
  } 
  
  public static Book fromTitle(String title) {
		return new Book(title); 
  }

  public String getTitle() {
    return this.title; 
  }
}
```



**명시적 변환**

묵시적 인자 변환에 의존하는 대신, 아래의 예제와 같이 특정 파라미터에 @ConvertWith 어노테이션을 사용하여 ArgumentConverter를 명시적으로 지정해주면 된다. ArgumentConverter의 구현은 반드시 클래스 최상위 레벨에 선언하거나, 정적 중첩 클래스로 선언되야한다.

```java
@ParameterizedTest 
@EnumSource(ChronoUnit.class) 
void testWithExplicitArgumentConversion( 
  @ConvertWith(ToStringArgumentConverter.class) String argument) {
	assertNotNull(ChronoUnit.valueOf(argument));
}
```

**ToStringArgumentConverter.java**

```java
public class ToStringArgumentConverter extends SimpleArgumentConverter {

	@Override 
  protected Object convert(Object source, Class<?> targetType) {
    assertEquals(String.class, targetType, "Can only convert to String");
    if (source instanceof Enum<?>) {
      return ((Enum<?>) source).name(); 
    }
    return String.valueOf(source); 
  }
}
```

컨버터가 오직 타입을 다른 타입으로 변경하는거라면, 보일러플레이트 타입 체크를 줄이기 위해서 TypedArgumentConverter를 상속하면 된다.

**ToLengthArgumentConverter.java**

```java
public class ToLengthArgumentConverter extends TypedArgumentConverter<String, Integer> 
{

  protected ToLengthArgumentConverter() {
    super(String.class, Integer.class); 
  }

	@Override 
  protected Integer convert(String source) {
    return source.length(); 
  }

}
```

명시적 인자 컨버터는 테스트를 구현하거나, authors ?를  확장할 때 사용한다, 그러므로 junit-jupiter-params는 참조 구현(reference implementation)을 제공하는 JavaTimeArgumentConverter 오직 하나의 명시적 인자 컨버터만 제공한다. JavaTimeConversionPattern 어노테이션을 사용할 때 사용된다. 



```java
@ParameterizedTest 
@ValueSource(strings = { "01.01.2017", "31.12.2017" }) 
void testWithExplicitJavaTimeConverter( 
  @JavaTimeConversionPattern("dd.MM.yyyy") LocalDate argument) {

	assertEquals(2017, argument.getYear());
}
```



#### **인자 수집**

기본적으로 @ParameterizedTest 메소드에 제공되는 각각의 인자들은 하나의 메소드 파라미터와 일치한다. 결과적으로 많은 인자가 제공될 것이 예상 되는 인자 소스는 많은 메소드 시그니쳐를 낳게 된다.

이런 경우때문에 `ArgumentsAccessor` 는 여러개의 파라미터를 대신하여 사용한다. 이 API를 사용하기 위해 테스트 메소드에 제공된 하나의 인자를 통해서 접근할 수 있다. 게다가 위에서 묵시적 변환에서 얘기한 것 처럼 타입 변환이 지원 된다.

```java
@ParameterizedTest 
@CsvSource({
  "Jane, Doe, F, 1990-05-20",
  "John, Doe, M, 1990-10-22" 
}) 
void testWithArgumentsAccessor(ArgumentsAccessor arguments) {
	Person person = new Person(arguments.getString(0),
                             arguments.getString(1),
                             arguments.get(2, Gender.class),
                             arguments.get(3, LocalDate.class));

	if (person.getFirstName().equals("Jane")) {
    assertEquals(Gender.F, person.getGender()); 
  } else {
		assertEquals(Gender.M, person.getGender()); 
  }
  assertEquals("Doe", person.getLastName()); 
  assertEquals(1990, 	person.getDateOfBirth().getYear());
}
```

ArgumentsAccessor 인스턴스는 자동으로 ArgumentsAccessor 타입의 파라미터에 주입된다.



**커스텀 수집**

ArgumentsAccessor를 @ParameterizedTest 인자로 직접적으로 접근하는걸 제외하고, 커스텀하고, 재사용 가능한 수집기를 사용할 수 있다.

커스텀 수집기를 사용하기 위해서 ArgumentsAggregator 인터페이스를 구현해서, @AggregateWith 어노테이션을 통해서 등록해야 한다. 

Note that an implementation of ArgumentsAggregator must be declared as either a top-level class or as a static nested class.

```java
@ParameterizedTest 
@CsvSource({
  "Jane, Doe, F, 1990-05-20",
  "John, Doe, M, 1990-10-22" 
}) void testWithArgumentsAggregator(@AggregateWith(PersonAggregator.class) Person person) 
{
// perform assertions against person 
}
```



```java
public class PersonAggregator implements ArgumentsAggregator { 
  @Override 
  public Person aggregateArguments(ArgumentsAccessor arguments, ParameterContext context) {
		return new Person(arguments.getString(0),
                      arguments.getString(1),
                      arguments.get(2, Gender.class),
                      arguments.get(3, LocalDate.class));
  }
}
```



만약 @AggregateWith() 어노테이션이 파라미터화 테스트 메소드 여러 개에서 사용하고 있다면, 다음과 같이 커스텀 어노테이션을 만들어서 사용해 볼 수도 있다.

```java
@ParameterizedTest 
@CsvSource({
	"Jane, Doe, F, 1990-05-20",
	"John, Doe, M, 1990-10-22" 
}) 
void testWithCustomAggregatorAnnotation(@CsvToPerson Person person) {
// perform assertions against person 
}
```



```java
@Retention(RetentionPolicy.RUNTIME) 
@Target(ElementType.PARAMETER) 
@AggregateWith(PersonAggregator.class)
public @interface CsvToPerson { }
```



#### DispalyName 커스텀 하기

기본적으로 파라미터화 테스트 호출의 display name은 호출된 index와 특정 호출에 관련한 모든 인자를 나타내는 String으로 이루어져 있다. 각각 앞에 파라미터 변수 이름이 붙는다. 

그러나 다음과 같이 name 속성을 이용해서 display name을 커스텀할 수 있다.

```java
@DisplayName("Display name of container") 
@ParameterizedTest(name = "{index} ==> the rank of ''{0}'' is {1}") 
@CsvSource({ "apple, 1", "banana, 2", "'lemon, lime', 3" }) 
void testWithCustomDisplayNames(String fruit, int rank) { 
}
```

이 테스트를 실행해보면 콘솔에 다음과 같이 찍힌다.

```
Display name of container ✔ 
├─ 1 ==> the rank of 'apple' is 1 ✔ 
├─ 2 ==> the rank of 'banana' is 2 ✔ 
└─ 3 ==> the rank of 'lemon, lime' is 3 ✔
```

name 속성은 `MessageFormat` 패턴을 사용한다. 그래서 작은따옴표(')를 표현하기 위해 작은따옴표를 두번 썼다.

다음은 display name을 커스텀 하기 위한 지원되는 placeholder 들이다.

| Placeholder          | Description                                                  |
| -------------------- | ------------------------------------------------------------ |
| DisplayName          | 메소드의 이름을 나타낸다.                                    |
| {index}              | 현재 호출된 인덱스를 나타낸다.                               |
| {arguments}          | the complete, comma-separated arguments list                 |
| {argumentsWithNames} | the complete, comma-separated arguments list with parameter names |
| {0}, {1}, …          | 0번째 인자 1번째 인자..                                      |



> When including arguments in display names, their string representations are truncated if they exceed the configured maximum length. The limit is configurable via the junit.jupiter.params.displayname.argument.maxlength configuration parameter and defaults to 512 characters.



#### 생명주기와 상호작용성?(Interoperability) 

@BeforeEach 메소드는 호출 전에 실행 되듯이, 파라미터화 테스트는 호출마다 @Test 메소드와 동일한 생명주기를 갖는다. 다이나믹 테스트와 비슷하게, 호출마다 테스트이 트리가 하나씩 IDE에 보여진다. 일반적인 @Test 메소드와 @ParamterizedTest 메소드를 같은 테스트 클래스안에 혼합해서 사용할 수 있다.

@ParameterizedTest 메소드를 ParamterResolver를 확장해서 사용하지만, argument source에 의해 제공되는 메소드 파라미터들은 인자 리스트에 먼저 와야한다, 테스트 클래스가 



ou may use ParameterResolver extensions with @ParameterizedTest methods. However, method parameters that are resolved by argument sources need to come first in the argument list. Since a test class may contain regular tests as well as parameterized tests with different parameter lists, values from argument sources are not resolved for lifecycle methods (e.g. @BeforeEach) and test class constructors.

```java
@BeforeEach void beforeEach(TestInfo testInfo) {

// ...

} @ParameterizedTest @ValueSource(strings = "apple") void testWithRegularParameterResolver(String argument, TestReporter testReporter) {

testReporter.publishEntry("argument", argument); } @AfterEach void afterEach(TestInfo testInfo) {

// ... }
```



### 테스트 템플릿

@TestTemplate 메소드는 일반적인 테스트 케이스보단 테스트 케이스에 대한 템플릿 이다. 등록된 프로바이더가 리턴하는 컨텍스트 호출 수에 따라 여러번 호출 되도록 디자인 되었다. 그래서 반드시 등록된 TestTemplateInvocationContextProvider와 함 께 사용해야 한다. 테스트 템플릿 메소드의 각각의 호출은 일반적인 @Test 메소드처럼 실행된다. 

> Repared Test와 Parameterized Test는 내장된 테스트 템플릿 중 하나이다.



### Dynamic Test (동적 테스트)

JUnit Jupiter의 @Test 어노테이션은 Junit 4의 @Test 어노테이션과 굉장히 유사하다. 둘다 테스트 케이스에 대한 내용이 메소드 안에 있다. 이런 테스트 케이스는 그 상태로 정적이며, 컴파일 시간에 정해지며, 런타임 시에도 아무 변화가 없다.  

Junit Jupiter는 이 기본 테스트들에 대해 완전히 새로운 테스트 프로그래밍 모델을 소개 했다. 이 새로운 종류의 테스트는 @TestFactory 어노테이션이 붙은 팩토리 메소드에 의해 런타임시 만들어지는 동적 테스트이다.

@Test 어노테이션을 사용하는 메소드와 반대로, @TestFactory 메소드는 그 자체로 테스트는 아니며, 팩토리 메소드가 테스트 케이스다. 그래서, 동적 테스트는 팩토리의 제품이다. 기술적으로 말하자면, @TestFactory 메소드는 반드시 하나의 DynamicNode나, Stream,Collection,Interable,Interator 나 DynamicNode 인스턴스의 배열을 리턴해야 한다. DynamicContainer 인스턴스는 dispalyName과랜덤으로 dynamic node의 중첩 계층을 만들 수 있는 동적 자식 노드의 리스트로 이루어져 있다. DynamicTest 인스턴스는 lazy 실행되어 테스트 케이스의 동적 생성과 비 결정적(non-deterministic) 생성이 가능하다.

@TestFactory 가 리턴하는 Stream은 stream.close()을 호출해서 닫아줘야 Files.lines() 같은 리소스를 안전하게 사용할 수 있다.

@Test 메소드를 같이 쓰려면,  @TestFactory를 반드시 private나  static으로 선언하면 안되며, 선택적으로 ParameterResolver에서 파라미터를 제공해주는 걸 사용할 수 있다.

그래서 DynamicTest는 런타임시 만들어지는 테스트케이스를 말한다. displayname과  A DynamicTest is a test case generated at runtime. It is composed of a display name and an Executable. Executable is a @FunctionalInterface which means that the implementations of dynamic tests can be provided as lambda expressions or method references.

> 동적 테스트 생명주기
>
> 동적 테스트의 생명주기는 @Test 어노테이션과는 좀 다르다. 특히 콜백 라이플 사이클이 존재하지 않는데, @BeforeEach 와 @AfterEach 메소드는 @TestFactory 메소드에서는 실행하는데, 각각의 동적테스트에 대해서는 실행하지 않는다. 다른 말로, 동적테스트 관한 람다 표현식안의 테스트 인스턴스의 필드에 접근하기위해서 해당 필드는 초기화 되지 않는 다는 말이다. 

JUnit Jupiter 5.7.0이 되면서 동적 테스트는 반드시 항상 factory 메소드가 만들어야 한다. however, this might be complemented by a registration facility in a later release.

#### 동적 테스트 예제

다음의 DynamicTestsDemo 클래스는 동적 테스트의 예제를 보여준다.

첫번 째 메소드는 유효하지 않은 타입을 반환한다. 유효하지 하지 않는 타입을 리턴하는건 컴파일 시에 알수 없고 런타임시에 알 수 있기 때문에, 발견되면 JUnitException을 던진다.

다음 5개의 메소드는 DynamicTest 인스턴스의 Collection, Iterable, Iterator, Stream을 만드는 간단한 예제이다. 대부분의 예제는 동적 동작을 제대로 보여주진 않고, 그저 규칙에 맞게 지원되는 리턴 타입을 반환하기만 한다. 그러나 dynamicTestFromStream() 과 dynamicTestsFromIntStream()은 주어진 string의 set과 인풋 숫자의 범위로 동적 테스트를 얼마나 쉽게 만드는지 보여준다.

그 다음 메소드는 정말로 다이나믹하다. generateRandomNumberOfTests()는 랜덤 숫자를 만드는 Iterator, display name generator, test executor를 만들어, 이 세개를 DynamicTest.stream()에게 제공한다.  generateRandomNumberOfTests()의 비 결정적(non-deterministic) 동작 때문에 반복되는 테스트와 충돌은 날수 있기 때문에 사용에 주의 해야 한다. 동적 테스트는 표현력이 뛰어나다.

그 다음 메소드는 유연셩 측면에서 generateRandomNumberOfTests()와 비슷하다. 그러나 dynamicTestsFromStreamFactoryMethod()는  DynamicTest.stream 팩토리 메소드를 통해서 다이나믹 테스트의 stream을 만들어낸다.

증명하기 위해서, dynamicNodeSingleTest()메소드는 stream 대신 하나의 DynamicTest를 만든다. the dynamicNodeSingleContainer() method generates a nested hierarchy of dynamic tests utilizing DynamicContainer.

```java
package dev.donghyeon.junitstudy.dynamic;

import static dev.donghyeon.junitstudy.StringUtils.isPalindrome;
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertFalse;
import static org.junit.jupiter.api.Assertions.assertNotNull;
import static org.junit.jupiter.api.Assertions.assertTrue;
import static org.junit.jupiter.api.DynamicContainer.dynamicContainer;
import static org.junit.jupiter.api.DynamicTest.dynamicTest;

import java.util.Arrays;
import java.util.Collection;
import java.util.Iterator;
import java.util.List;
import java.util.Random;
import java.util.function.Function;
import java.util.stream.IntStream;
import java.util.stream.Stream;


import dev.donghyeon.junitstudy.Calculator;
import org.junit.jupiter.api.DynamicNode;
import org.junit.jupiter.api.DynamicTest;
import org.junit.jupiter.api.Tag;
import org.junit.jupiter.api.TestFactory;
import org.junit.jupiter.api.function.ThrowingConsumer;

class DynamicTestsDemo {

    private final Calculator calculator = new Calculator();

    // This will result in a JUnitException!
    @TestFactory
    List<String> dynamicTestsWithInvalidReturnType() {
        return Arrays.asList("Hello");
    }

    @TestFactory
    Collection<DynamicTest> dynamicTestsFromCollection() {
        return Arrays.asList(
                dynamicTest("1st dynamic test", () -> assertTrue(isPalindrome("madam"))),
                dynamicTest("2nd dynamic test", () -> assertEquals(4, calculator.multiply(2, 2)))
        );
    }

    @TestFactory
    Iterable<DynamicTest> dynamicTestsFromIterable() {
        return Arrays.asList(
                dynamicTest("3rd dynamic test", () -> assertTrue(isPalindrome("madam"))),
                dynamicTest("4th dynamic test", () -> assertEquals(4, calculator.multiply(2, 2)))
        );
    }

    @TestFactory
    Iterator<DynamicTest> dynamicTestsFromIterator() {
        return Arrays.asList(
                dynamicTest("5th dynamic test", () -> assertTrue(isPalindrome("madam"))),
                dynamicTest("6th dynamic test", () -> assertEquals(4, calculator.multiply(2, 2)))
        ).iterator();
    }

    @TestFactory
    DynamicTest[] dynamicTestsFromArray() {
        return new DynamicTest[]{
                dynamicTest("7th dynamic test", () -> assertTrue(isPalindrome("madam"))),
                dynamicTest("8th dynamic test", () -> assertEquals(4, calculator.multiply(2, 2)))
        };
    }

    @TestFactory
    Stream<DynamicTest> dynamicTestsFromStream() {

        return Stream.of("racecar", "radar", "mom", "dad")
                .map(text -> dynamicTest(text, () -> assertTrue(isPalindrome(text))));
    }

    @TestFactory
    Stream<DynamicTest> dynamicTestsFromIntStream() {

        // Generates tests for the first 10 even integers.
        return IntStream.iterate(0, n -> n + 2).limit(10).mapToObj(n -> dynamicTest("test" + n, () -> assertTrue(n % 2 == 0)));

    }

    @TestFactory
    Stream<DynamicTest> generateRandomNumberOfTestsFromIterator() {

        // Generates random positive integers between 0 and 100 until
        // a number evenly divisible by 7 is encountered.
        Iterator<Integer> inputGenerator = new Iterator<Integer>() {
            Random random = new Random();
            int current;

            @Override
            public boolean hasNext() {
                current = random.nextInt(100);
                return current % 7 != 0;
            }

            @Override
            public Integer next() {
                return current;
            }

        };
        // Generates display names like: input:5, input:37, input:85, etc.
        Function<Integer, String> displayNameGenerator = (input) -> "input:" + input;
        // Executes tests based on the current input value.
        ThrowingConsumer<Integer> testExecutor = (input) -> assertTrue(input % 7 != 0);

        // Returns a stream of dynamic tests.

        return DynamicTest.stream(inputGenerator, displayNameGenerator, testExecutor);

    }

    @TestFactory
    Stream<DynamicTest> dynamicTestsFromStreamFactoryMethod() {

        // Stream of palindromes to check
        Stream<String> inputStream = Stream.of("racecar", "radar", "mom", "dad");

        // Generates display names like: racecar is a palindrome
        Function<String, String> displayNameGenerator = text -> text + " is a palindrome";

        // Executes tests based on the current input value.
        ThrowingConsumer<String> testExecutor = text -> assertTrue(isPalindrome(text));

        // Returns a stream of dynamic tests.
        return DynamicTest.stream(inputStream, displayNameGenerator, testExecutor);

    }

    @TestFactory
    Stream<DynamicNode> dynamicTestsWithContainers() {

        return Stream.of("A", "B", "C")
                .map(input -> dynamicContainer("Container " + input, Stream.of(
                        dynamicTest("not null", () -> assertNotNull(input)),
                        dynamicContainer("properties", Stream.of(
                                dynamicTest("length > 0", () -> assertTrue(input.length() > 0)),
                                dynamicTest("not empty", () -> assertFalse(input.isEmpty()))
                        ))
                )));
    }

    @TestFactory
    DynamicNode dynamicNodeSingleTest() {
        return dynamicTest("'pop' is a palindrome", () -> assertTrue(isPalindrome("pop")));
    }

    @TestFactory
    DynamicNode dynamicNodeSingleContainer() {
        return dynamicContainer("palindromes",
                Stream.of("racecar", "radar", "mom", "dad")
                        .map(text -> dynamicTest(text, () -> assertTrue(isPalindrome(text)))
                ));
    }

}

```



#### 다이나믹 URI 테스트 

2.17.2. URI Test Sources for Dynamic Tests

The JUnit Platform provides TestSource, a representation of the source of a test or container used to navigate to its location by IDEs and build tools.

The TestSource for a dynamic test or dynamic container can be constructed from a java.net.URI which can be supplied via the DynamicTest.dynamicTest(String, URI, Executable) or DynamicContainer.dynamicContainer(String, URI, Stream) factory method, respectively. The URI will be converted to one of the following TestSource implementations.

ClasspathResourceSource

If the URI contains the classpath:/test/foo.xml?line=20,column=2.

classpath

scheme — for

example,

DirectorySource

If the URI represents a directory present in the file system.

66 FileSource

If the URI represents a file present in the file system.

MethodSource

If the URI contains the method scheme and the fully qualified method name (FQMN) — for example, method:org.junit.Foo#bar(java.lang.String, java.lang.String[]). Please refer to the Javadoc for DiscoverySelectors.selectMethod(String) for the supported formats for a FQMN.

UriSource

If none of the above TestSource implementations are applicable.

### Timeouts

@Timeout 어노테이션은 테스트, 테스트 팩토리, 테스트 템플릿 , 라이프사이클 메소드에 선언하며, 주어진 시간 동안 실행시간이 초과하면 실패하게 된다. 기본 단위는 seconds(초)이지만, 따로 설정도 가능하다.

다음이 예제는 @Timeout이 라이프사이클 메서드와, 테스트 메서드에 사용된 모습이다.

```java
class TimeoutDemo {

    @BeforeEach
    @Timeout(5)
    void setUp() {
        // fails if execution time exceeds 5 seconds
    }

    @Test
    @Timeout(value = 100, unit = TimeUnit.MILLISECONDS)
    void failsIfExecutionTimeExceeds100Milliseconds() {
        // fails if execution time exceeds 100 milliseconds
    }

}
```

assertTimeoutPreemptively() 검증과 대조적으로 @Timeout 어노테이션이 붙은 메소드는 테스트의 메인스레드 안에서 진행된다. 만약 시간초과가 되면, 다른 스레드에 의해 메인스레드는 인터럽트 된다. 이는 현재 실행중인 스레드에 민감한 메커니즘을 사용하는 Spring과 같은 프레임 워크와의 상호 운용성을 보장하기 위해 수행된다. ThreadLocal의 트랜잭션 관리가 예로 있다.

테스트 클래스안에 모든 테스트 메소드 또는 모든 @Nested 클래스 안에  같은 timeout을 적용하고 싶으면 클래스 레벨에 @Timeout 어노테이션을 선언하면 된다. 이렇게하면 특정 메소드나 @Nested 클래스 안에 @Timeout 어노테이션을 오버라이드만 하지 않으면, 모든 테스트, 테스트팩토리와 테스트템플릿 메소드에 일괄 적용된다.

Declaring @Timeout on a @TestFactory method checks that the factory method returns within the specified duration but does not verify the execution time of each individual DynamicTest generated by the factory. Please use assertTimeout() or assertTimeoutPreemptively() for that purpose.

@Timeout이 @RepeatedTest 나 @ParameterizedTest 같은 @TestTemplate 메서드에 선언되어 있다면, 각각의 호출마다 timeout이 적용 된다.

다음의 설정 파라미터는 특정 카테고리에 있는 모든 메소드의 전체 타임아웃을 지정하는 방법이다. 이 방법은 @Timeout이 붙지 않은 테스트 클래스에 일괄 적용 된다.

junit.jupiter.execution.timeout.default

Default timeout for all testable and lifecycle methods

junit.jupiter.execution.timeout.testable.method.default

Default timeout for all testable methods

junit.jupiter.execution.timeout.test.method.default

Default timeout for @Test methods

junit.jupiter.execution.timeout.testtemplate.method.default

Default timeout for @TestTemplate methods

junit.jupiter.execution.timeout.testfactory.method.default

Default timeout for @TestFactory methods

junit.jupiter.execution.timeout.lifecycle.method.default

Default timeout for all lifecycle methods

junit.jupiter.execution.timeout.beforeall.method.default

Default timeout for @BeforeAll methods

junit.jupiter.execution.timeout.beforeeach.method.default

Default timeout for @BeforeEach methods

junit.jupiter.execution.timeout.aftereach.method.default

Default timeout for @AfterEach methods

junit.jupiter.execution.timeout.afterall.method.default

Default timeout for @AfterAll methods

More specific configuration parameters override junit.jupiter.execution.timeout.test.method.default junit.jupiter.execution.timeout.testable.method.default junit.jupiter.execution.timeout.default.

less

specific

ones.

For

which

example, overrides overrides

The values of such configuration parameters must be in the following, case-insensitive format: <number> [ns|μs|ms|s|m|h|d]. The space between the number and the unit may be omitted. Specifying no unit is equivalent to using seconds.

Table 1. Example timeout configuration parameter values

| Parameter value | Equivalent annotation                     |
| --------------- | ----------------------------------------- |
| 42              | @Timeout(42)                              |
| 42 ns           | @Timeout(value = 42, unit = NANOSECONDS)  |
| 42 μs           | @Timeout(value = 42, unit = MICROSECONDS) |
| 42 ms           | @Timeout(value = 42, unit = MILLISECONDS) |
| 42 s            | @Timeout(value = 42, unit = SECONDS)      |
| 42 m            | @Timeout(value = 42, unit = MINUTES)      |
| 42 h            | @Timeout(value = 42, unit = HOURS)        |
| 42 d            | @Timeout(value = 42, unit = DAYS)         |

#### Polling Test에 @Timeout 사용하기

비동기 코드를 다뤄야 할 때  검증을 하기 전에, 어떤 일이 생기기를 기다리는 poll 테스트를 작성하는 것이 일반적이다. 어떤 경우에는 CountDownLatch나 다른 동기화 메카니즘으로 로직을 재작성 해야 한다. 그러나 때때로 불가능할 수도 있다. 예를 들어, 테스트에 있는 대상이 외부에 있는 메세지 브로커로 채널에 메세지를 전송하면, 메세지가 성공적으로 채널로 전달될 때 까지 검증을 수행하지 못한다. 이와 같은 비동기 테스트에는 비동기 메시지가 성공적으로 전달되지 않는 경우와 같이 무기한 실행하여 테스트 모음이 중단되지 않도록 특정 형태의 시간 제한이 필요하다.

폴링하는 비동기 테스트에 대한 제한 시간을 구성하여 테스트가 무기한 실행되지 않도록 할 수 있다. 다음의 예제는 @Timeout 어노테이션을 이용한 예제다. 

```java
@Test
@Timeout(5)  // 최대 5초동안 실행
void pollUntil() throws InterruptedException {
  while (asynchronousResultNotAvailable()) {
    Thread.sleep(250); // 폴링 시간 설정
  }
  // 비동기 결과를 얻어서 검증을 실행
}
```

> 폴링 간격을 컨트롤하거나, 비동기 테스트를 유연하게 하고 싶으면 [Awaitility](https://github.com/awaitility/awaitility) 를 사용해보는 걸 추천한다.

#### 전체적으로 @Timout 비활성화하기

테스트를 디버그모드로 돌릴 때, 고정된 타임아웃 제한은 테스트결과에 영향이 있을 수 있다. 예를 들어 모든 검증이 성공했음에도 테스트가 실패할 수 있다.

Junit jupiter는 junit.jupiter.execution.timeout.mode 설정 파라미터를 지원해서 원할 때 enabled, disabled, disabled_on_debug 세가지 모드로 변경할 수 있다. 기본 값은 enabled 이다. VM 런타임이 인풋 파라미터가 -agentlib:jdwp로 시작하는 파라미터가 있으면 디버그 모드로 실행된다. 

### 병렬적 실행

> 병렬적 실행은 experimental 기능이다.

기본적으로 테스트들은 싱글스레드안에서 순차적으로 실행된다. 테스트를 병렬적으로 실행하면, 테스트 실행 속도가 빨라진다. 이 기능은 5.3부터 지원한다. 병렬적 실행을 활성화 하려면 junit.jupiter.execution.parallel.enabled 설정 파라미터를 true로 설정하면 된다. 

위에서 파라미터를 true로 설정한 것은 첫번 째 단계이며, true로 설정해도 테스트는 기본적으로 순차적으로 실행 될 것이다. 이는 테스트 트리의 노드가 동시에 실행되는지 여부에 따라 실행 모드가 제어 된다. 다음의 두 가지 모드가 사용 가능 하다.

**SAME_THREAD**

부모가 사용하던 스레드를 사용한다. 예를 들어, 테스트 메소드를 사용할 때, 테스트 메소드는 @BeforeAll이나 @AfterAll 메소드가 사용하던 스레드를 같이 사용 한다..

**CONCURRENT**

리소스 잠금이 동일한 스레드에서 강제 실행하지 않는 한 동시에 실행한다.

기본적으로 테스트 트리에 있는 노드는 SAME_THREAD를 사용한다. junit.jupiter.execution.parallel.mode.default 설정 파라미터를 사용하면 기본값을 바꿀 수 있다. 아니면, @Execution 어노테이션을 사용해서 독립적으로 해당 테스트 또는 클래스만 실행모드를 변경할 수 있다.

모든 테스트를 병렬적으로 실행하는 파라미터 설정이다.

```
junit.jupiter.execution.parallel.enabled = true junit.jupiter.execution.parallel.mode.default = concurrent
```

기본 실행 모드는 Lifecycle.PER_CLASS 모드 또는 MethodOrderer를 사용하는 테스트 클래스르 제외하고 테스트 트리의 모든 노드에 적용 된다. Lifecycle.PER_CLASS의 경우 테스트 작성자는 테스트 클래스가 스레드로부터 안전한지 확인해야 한다. MethodOrderer 경우, 동시 실행이 구성된 실행순서와 충돌할 수 있다. 따라서 두 경우 모두 이러한 테스트 클래스의 테스트 메서드는 @Execution(CONCURRENT)주석이 테스트 클래스 또는 메서드에 있는 경우에만 병렬적으로 실행된다.

CONCURRENT 실행 모드로 구성된 테스트 트리의 모든 노드는 선언적 동기화 메커니즘을 관찰하면서 제공된 구성에 따라 완전히 병렬로 실행된다. Please note that Capturing Standard Output/Error needs to be enabled separately.

추가적으로 junit.jupiter.execution.parallel.mode.classes.default 설정 파라미터를 클래스 최상단에 설정할 수 있다. 이 설정을 위에 설정해준거랑 같이 써주면 병렬적으로 실행되지만, 메소드들은 같은 스레드에서 실행된다.

```
junit.jupiter.execution.parallel.enabled = true junit.jupiter.execution.parallel.mode.default = same_thread junit.jupiter.execution.parallel.mode.classes.default = concurrent
```

이와 반대적인 설정은, 하나의 클래스의 모든 메소드는 병렬적으로 실행되지만, 최상위 클래스는 순차적으로 실행된다.

```
junit.jupiter.execution.parallel.enabled = true junit.jupiter.execution.parallel.mode.default = concurrent junit.jupiter.execution.parallel.mode.classes.default = same_thread
```

![스크린샷 2021-01-07 오전 8.44.54](/Users/donghyeonmin/Library/Application Support/typora-user-images/스크린샷 2021-01-07 오전 8.44.54.png)

If the junit.jupiter.execution.parallel.mode.classes.default configuration parameter is explicitly set, the value for junit.jupiter.execution.parallel.mode.default will be used instead.



#### 설정

`ParallelExecutionConfigurationStrategy` 을 이용하여 원하는 병렬과, 최대 풀 사이즈의 프로퍼티들을 설정할 수 있다. dynamic과 fixed 두개의 구현을 제공한다. 아니면 커스텀해서 사용해도 된다.

사용하고 싶은 구현을 선택하려면 junit.jupiter.execution.parallel.config.strategy 설정 파라미터를 사용해야 한다.

**dynamic**

junit.jupiter.execution.paralle.config.dynamic.factor 설정 파라미터에 설정된 값과 사용가능한 프로세스/ 코어 수를 곱하여 원하는 병렬 처리를 계산한다. (기본 값은 1)

**fixed**

필수적인 junit.jupiter.execution.parallel.config.fixed.parallelism 설정 파라미터를 원하는 병렬 처리로 사용 한다.

**custom**

필수적인 junit.jupiter.execution.parallel.config.custom.class 설정 파라미터를 통해 사용자 지정 ParallelExecutionConfigurationStrategy 구현을 지정하여 원하는 설정을 결정한다.

만약 어떠한 전략도 설정되지 않았으면, factor 1을 가진 dynamic 설정을 이용한다. 그렇게 되면, 병렬 구성은 프로세서/코어의 사용 가능한 수로 사용된다.

> 병렬처리는 최대 동시 스레드 수를 의미 하지 않는다.
>
> Junit은 동시에 실행되는 테스트의 수가 설정된 병렬 처리를 초과하지 않을 것이라고 보장하지 않는다. 예를 들어 다음 세션에서 살펴 볼 동기화 메카니즘 중 하나인 ForkJoinPool을 사용할 때 그 뒤에는 충분한 병렬 처리로 실행이 계속 되도록 추가 스레드를 생성 한다. 따라서 테스트 클래스에서 이러한 보장이 필요한 경우 동시성을 제어하는 자체 수단을 사용해야 한다.



#### 동기화(Synchronization)

@Execution 어노테이션을 이용해서 실행 모드를 컨트롤 하기 위해, Junit은 또 다른 어노테이션 기반 선언적 동기화 메카니즘을 제공한다. @ResourceLock 어노테이션은 테스트 클래스나 메서드에 선언할 수 있으며, 안정적인 테스트 실행 보장하기 위해 동기화된 접근이 필요한 특정 공유 자원에 사용한다.

공유 자원은 String타입으로 유일한 이름을 갖도록하여 식별한다. 이름은 사용자가 정의하거나, Resources 안에 미리 선언된 SYSTEM_PROPERTIES, SYSTEM_OUT, SYSTEMERR, LOCALE, TIME_ZONE을 사용할 수 있다.

만약 아래의 테스트가 @ResourceLock 어노테이션 없이 병렬하게 실행된다면 테스트가 이상해진다. 가끔은 테스트가 패스되고,  가끔은 race condition 때문에 실패하기도 한다. 

@ResourceLock 어노테이션이 붙은 공유 자원에 접근하려고 할 때 Junit은 병렬적으로 실행되는 테스트에 충돌이 없게 보장한다.

> 격리된 테스트 실행
>
> 대부분의 테스트가 병렬적으로 실행되는데, 어떠한 동기화도 없이 실행되는 클래스라면,@Isolated 어노테이션을 이용하여 격리된 상태로 테스트를 실행할 수 있다. 이런 테스트 클래스는 다른 테스트와 동시에 실행되지 않고, 순차적으로 실행 된다.

공유 자원을 고유하게 식별하는 String 타입외에도 접근 모드를 지정해줄 수 있다. 공유 자원에 대한 READ 접근이 필요한 두 테스트는 서로 병렬로 실행 될 수 있지만, 공유 자원에 대한 READ_WRITE 접근이 필요한 다른 테스트가 실행되는 동안에는 실행되지 않는다. 즉 READ_WRTIE의 테스트가 전부 끝날 때 까지 대기한다.

```java
@Execution(CONCURRENT)
class SharedResourcesDemo {

    private Properties backup;

    @BeforeEach
    void backup() {
        backup = new Properties();
        backup.putAll(System.getProperties());
    }

    @AfterEach
    void restore() {
        System.setProperties(backup);
    }

    @Test
    @ResourceLock(value = SYSTEM_PROPERTIES, mode = READ)
    void customPropertyIsNotSetByDefault() {
        assertNull(System.getProperty("my.prop"));
    }

    @Test
    @ResourceLock(value = SYSTEM_PROPERTIES, mode = READ_WRITE)
    void canSetCustomPropertyToApple() {
        System.setProperty("my.prop", "apple");
        assertEquals("apple", System.getProperty("my.prop"));
    }

    @Test
    @ResourceLock(value = SYSTEM_PROPERTIES, mode = READ_WRITE)
    void canSetCustomPropertyToBanana() {
        System.setProperty("my.prop", "banana");
        assertEquals("banana", System.getProperty("my.prop"));
    }

}
```



### Built-in Extensions

Junit 팀은 재사용 가능한 extension이 패키징되고 유지되도록 권장하지만, Junit Jupiter API 아티팩트는 일반적으로 유옹하여 사용자가 다른 의존성을 추가할 필요 없이 몇 가지의 extension이 포함되어 있다.

#### TempDirectory Extension

> @TempDir는 experimental 기능이다.

TempDirectory extension은 테스트클래스 안에 있는 독립적인 테스트 또는 모든 테스트에 대해 임시 디렉토리를 생성하거나, 정리를 할 때 사용한다. 이 기능을 사용하려면 접근 제어자가 private이 아닌 `java.nio.file.Path` 나 `java.io.File` 필드에 @TempDir 어노테이션을 붙이거나, 파라미터에 붙여준다.  

```java
@Test 
void writeItemsToFile(@TempDir Path tempDir) throws IOException {
  Path file = tempDir.resolve("test.txt");
  new ListWriter(file).write("a", "b", "c");
  assertEquals(singletonList("a,b,c"), Files.readAllLines(file)); 
}
```

> @TempDir는 생성자 파라미터에는 지원하지 않는다. 만약 라이프 사이클 메서드와 현재 테스트 메서드를 넘어서 임시 디렉토리를 유지하기 싶으면 접근제어자가 private이 아닌 인스턴스 필드에 @TempDir 어노테이션을 이용해 필드 인젝션을 이용해야 한다.



다음의 예제는 static field에 있는 공유 임시 디렉토리에 저장한다. This allows the same sharedTempDir to be used in all lifecycle methods and test methods of the test class.

```java
class SharedTempDirectoryDemo {

    @TempDir
    static Path sharedTempDir;

    @Test
    void writeItemsToFile() throws IOException {

        Path file = sharedTempDir.resolve("test.txt");
        new ListWriter(file).write("a", "b", "c");
        assertEquals(singletonList("a,b,c"), Files.readAllLines(file));
    }

    @Test
    void anotherTestThatUsesTheSameTempDir() {
        // use sharedTempDir 
    }
}
```



## Junit4에서 마이그레이션 하기

JUnit Juptier 프로그래밍 모델과 extensiom 모델은 Junit4 특징인 Rules과 Runners에 네이티브하게 지원되지는 않지만, 반드시 버전업을 해야되는건 아니다.

대신 JUnit은 Junit 플랫폼 인프라를 이용해  JUnit3와 와  JUnit4 기반의 테스트를 실행시켜주는  Junit Vintage 테스트 엔진을 통해 마이그레이션을 지원해준다. 모든 클래스와 어노테이션들은 새로운 패키지인 org.junit.jupiter 베이스 안에 존재하기 때문에 JUnit4와  JUnit Jupiter는 클래스패스에서 충돌날 일이 없다. 게다가 JUnit 팀은 Junit4에 대하여 지속적인 유지보수와 버그수정 릴리즈 진행하고 있다.

### JUnit 플랫폼에서  JUnit4 테스트 실행하기

먼저`junit-vintage-engine` 이 테스트 런타임 패스에 있는지 확인하자. 

#### Categories Support

@Category 어노테이션이 붙은 테스트 클래스나 메서드를 실행하기 위해, JUnit Vintage 테스트 엔진에 해당 테스트 식별자의 태그로 카테고리의 패키지를 포함한 클래스 이름을 적어야 한다. 예를 들어 `@Category(Example.class)` 어노테이션이 붙은 테스트 메서드가 있으면  JUnit 4에서는 Categories 러너에  `com.acme.Example`  태그와 같다. 

### 마이그레이션 팁

다음의 주제들은 JUnit4 에서 JUnit Jupiter로 마이그레이션할 때 조심해야할 것들이다.

- 어노테이션은 `org.junit.jupiter.api` 패키지에 있다.
- Assertion은 `org.junit.jupiter.api.Assertions` 에 있다.
  - `org.junit.Asssert` 에 있는 assertion 메서드나 다른 assertion 라이브러리인 AssertJ, Hamcrest, Truth 등을 사용해도 된다.
- Assumption은 `org.junit.jupiter.api.Assumptions` 에 있다.
  - JUnit 5.4 버전이나 그 후 버전은JUnit 4의 `org.junit.Assume` 클래스의 assumption을 지원한다.  Specifically, JUnit Jupiter supports JUnit 4’s AssumptionViolatedException to signal that a test should be aborted instead of marked as a failure.
- @Before 와 @After는 더 이상 없다. 대신 @BeforeEach와 @AfterEach를 사용해야 한다.
- @BeforeClass와 @AFterClass는 더 이상 없다. 대신 @BeforeAll와 @AfterAll를 사용해야 한다.
- @Ignore는 더이상 없다. 대신 @Disabled나 내장된 조건실행(execution condition)을 사용하면 된다.
- @Category는 더 이상 없다. 대신 @Tag를 사용하면 된다.
- @RunWith는 더 이상 없다. 대신 @ExtendWith을 사용하면 된다.
- @Rule과 @ClassRule은 더 이상 없다. 대신 @ExtendWith과 @RegisterExtenstion을 사용하면 된다.

### 제한된 JUnit 4 Rule 지원

위에 명시한 것 처럼 JUnit Jupiter에서 JUnit4의 rule은 더 이상 네이티브하게 지원하지 않는다. 