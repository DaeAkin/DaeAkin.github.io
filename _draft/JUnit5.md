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

특정 JRE 버전에 따라 테스트를 활성화 비활성화 할 수 있습니다. `@EnabledOnJre` 와 `@DisabledOnJre` 어노테이션을 이용해서 활성화할 JRE 버전을 명시하고, `@EnabledForJreRange` 와 `DisabledForJreRange`  어노테이션을 이용하면 여러개으 ㅣJRE 버전을 

## 테스트 LifeCycle

사이드 이펙트를 줄이고, 테스트 메서드를 격리된 환경에서 독립적으로 실행시키기 위해 JUnit은 테스트 메소드를 실행시키기 전에 각각의 테스트 클래스의 새로운 인스턴스를 만든다.  

