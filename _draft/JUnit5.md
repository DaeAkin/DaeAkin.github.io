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

- @DisplayName : 테스트 클래스나테스트 메소드에 이름을 붙여줄 때 사용한다.

- @DisplayNameGeneration : 클래스에 해당 애노테이션을 붙이면 @Test 메소드 이름에 `_`로 표시한 모든 부분은 space로 처리된다.

  ```java 
  @DisplayNameGeneration(DisplayNameGenerator.ReplaceUnderscores.class)
  class SomeTest {
  
  }
  ```

  

