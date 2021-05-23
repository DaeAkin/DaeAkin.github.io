---
layout: post
title: 명명 패턴보다 어노테이션을 사용하자
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---



# 명명 패턴

전통적으로 도구나 프레임워크가 특별히 다뤄야 하는 요소에는 **딱 구분되는 명명 패턴을** 적용해왔다.

예를 들어 테스트 프레임워크인 JUnit의 3버전 경우 테스트 메서드 이름이 **<u>test</u>** 로 시작해야 된다.



## 명령 패턴의 단점

- 실수로 이름을 **tsetSafety** 로 지으면 JUnit 3이 이 메서드를 무시하고 지나간다.
- 프로그램 요소를 매개변수로 전달할 마땅한 방법이 없다.



# 어노테이션

이런 명명패턴의 단점을 해결하기 위해서 어노테이션을 이용할 수 있다.

JUnit4도 버전 4부터 명명패턴의 단점을 해결하기 위해서 어노테이션을 도입 했다.

어노테이션의 동작 방식을 위해서, 테스트 프레임워크를 직접 제작해보자.

### Test 어노테이션 정의

자동으로 수행되는 간단한 테스트용 어노테이션인 `@Test` 어노테이션을 만들어보자. 이 어노테이션은, 예외가 발생하면 해당 테스트를 실패로 처리한다.

```java
/**
 * 테스트 메서드임을 선언하는 어노테이션이다.
 * 매개변수 없는 정적 메서드 전용이다.
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test {
}
```

`@Test` 어노테이션 타입 선언 자체에는 두 가지의 다른 어노테이션인 `@Retention` 과 `@Target` 이 달려 있다. 

이처럼 어노테이션 선언에 다른 어노테이션을 **<u>메타 어노테이션</u>** 이라고 한다.

#### @Retention(RentenionPolicy.RUNTIME)

`@Retention` 메타 어노테이션은 `@Test` 가 런타임에도 유지되어야 한다는 뜻이다.

#### @Target(ElemenType.METHOD)

이 메타 어노테이션은, `@Test` 가 반드시 메서드 선언에만 사용돼야 한다고 알려준다. 

테스트 어노테이션의 주석을 보면  "매개변수 없는 정적 메서드 전용이다"라고 쓰여 있지만, 이 제약을 컴파일러가 강제로 할 수는 없다. 강제로 하려면 적절한 어노테이션 처리기를 구현해야 한다.



# @Test 어노테이션 적용

```java
public class Sample {
    @Test public static void m1() {} // 성공해야 한다.
    public static void m2() {}
    @Test public static void m3() {  // 실패해야 한다.
        throw new RuntimeException("실패");
    }
    public static void m4() { }
    @Test public void m5() { } //잘못 사용한 예 : 정적 메서드가 아님.
    public static void m6() { }
    @Test public static void m7() { // 실패해야 한다.
        throw new RuntimeException("실패");
    }
    public static void m8() {}
}
```

위에 코드는 `@Test` 어노테이션을 실제로 적용한 코드인데, `@Test`  은 아무 매개 변수 없이, 단순히 대상에 마킹 한다는 뜻에서 **마커 어노테이션** 이라고 한다.

위에 코드에서는 `@Test` 어노테이션이 Sample 클래스의 의미에 직접적인 영향을 주지 않는다. 그저 이 어노테이션에 관심 있는 프로그램에게 추가 정보를 제공할 뿐이다.

더 넓게 이야기 하면, **대상 코드의 의미는 그대로 둔 채 그 어노테이션에 관심 있는 도구에서 특별한 처리를 할 기회를 준다.**

## 어노테이션에 관심있는 클래스인 RunTests

```java
public class RunTests {
     public static void main(String[] args) throws Exception{
          int tests = 0;
          int passed = 0;
          Class<?> testClass = Class.forName(Sample.class.getName());
          for (Method m : testClass.getDeclaredMethods()) {
              if(m.isAnnotationPresent(Test.class)) {
                  tests++;
                  try {
                      m.invoke(null);
                      passed++;
                  } catch (InvocationTargetException wrappedExc) {
                      Throwable exc = wrappedExc.getCause();
                      System.out.println(m + "실패 : "+ exc);
                  } catch (Exception exc) {
                      System.out.println("잘못 사용한 @Test :" + m);
                  }
              }
          }
          System.out.printf("성공 : %d, 실패 : %d%n ",
                  passed, tests - passed);
      }
}
```



```
public static void test.Sample.m3()실패 : java.lang.RuntimeException: 실패
잘못 사용한 @Test :public void test.Sample.m5()
public static void test.Sample.m7()실패 : java.lang.RuntimeException: 실패
성공 : 1, 실패 : 3
```



# 특정 예외를 던져야만 성공하는 테스트

```java
/**
 * 명시한 예외를 던져야만 성공하는 테스트 메서드용 어노테이션
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}
```

매개변수 타입이 제한되어 있는데, "Throwable을 확장한 클래스의 Class 객체"라는 뜻이며, 따라서 모든 예외 타입만 들어갈 수 있다.

```java
public class Sample2 {
    @ExceptionTest(ArithmeticException.class)
    public static void m1() { // 성공해야 한다.
        int i = 0;
        i = i / i;
    }
    @ExceptionTest(ArithmeticException.class)
    public static void m2() { // 실패해야 한다. (다른 예외 발생)
        int[] a = new int[0];
    }
    @ExceptionTest(ArithmeticException.class)
    public static void m3() { // 실패해야 한다. (예외가 발생하지 않음)
    }
}
```

그 다음 이전에 작성했던 RunTests 클래스를 `@ExceptionTest` 를 지원하도록 수정해야 한다.

```java
public class RunTests {
     public static void main(String[] args) throws Exception{
          int tests = 0;
          int passed = 0;
          Class<?> testClass = Class.forName(Sample.class.getName());
          for (Method m : testClass.getDeclaredMethods()) {
              if(m.isAnnotationPresent(Test.class)) {
                  tests++;
                  try {
                      m.invoke(null);
                      System.out.printf("테스트 %s 실패 : 예외를 던지지 않음 \n",m);
                  } catch (InvocationTargetException wrappedExc) {
                      Throwable exc = wrappedExc.getCause();
                      Class<? extends Throwable> exeType =
                              m.getAnnotation(ExceptionTest.class).value();
                      if(exeType.isInstance(exc)) {
                          passed++;
                      } else {
                          System.out.printf(
                                  "테스트 %s 실패 : 기대한 예외 %s, 발생한 예외 %s%n",
                                  m, exeType.getName(), exc);
                      }
                  } catch (Exception exc) {
                      System.out.println("잘못 사용한 @Test :" + m);
                  }
              }
          }
          System.out.printf("성공 : %d, 실패 : %d%n ",
                  passed, tests - passed);
      }
}
```

# 배열 매개변수를 받는 어노테이션 

여러 개의 Exception을 받고 싶다면 배열을 받게 수정하면 된다.

```java
/**
 * 명시한 예외를 던져야만 성공하는 테스트 메서드용 어노테이션
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Throwable>[] value();
}
```



```java
@ExceptionTest({ IndexOutOfBoundsException.class ,
               NullPointerException.class})
public static void doublyBad() {
}
```



그 다음 테스트러너도 수정해주면 된다.

```java
public class RunTests {
    public static void main(String[] args) throws Exception {
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName(Sample.class.getName());
        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(Test.class)) {
                tests++;
                try {
                    m.invoke(null);
                    System.out.printf("테스트 %s 실패 : 예외를 던지지 않음 \n", m);
                } catch (InvocationTargetException wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    int oldPassed = passed;
                    Class<? extends Throwable>[] excTypes =
                            m.getAnnotation(ExceptionTest.class).value();
                    for (Class<? extends Throwable> excType : excTypes) {
                        if (excType.isInstance(exc)) {
                            passed++;
                            break;
                        }
                    }
                    if(passed == oldPassed)
                        System.out.printf("테스트 %s 실패 : %s %n", m,  exc);

                }
            }
        }
        System.out.printf("성공 : %d, 실패 : %d%n ",
                passed, tests - passed);
    }
}
```



# @Repeatable

자바 8에서는 여러 개의 값을 어노테이션을 새로운 방식으로 만들 수 있다.

배열 매개변수를 사용하는 대신 `@Repeatable` 어노테이션을 이용하면, 하나의 프로그램 요소에 여러 번 달 수 있다.

하지만 주의할 점이 있는데,

- `@Repeatable` 을 단 어노테이션을 반환하는 컨테이너 어노테이션을 하나 더 정의하고, `@Repeatable`에 이 컨테이너 어노테이션의 class 객체로 매개변수로 전달해야 한다.
- 컨테이너 어노테이션은 내부 어노테이션 타입의 배열을 반환하는 value 메서드를 정의해야 한다.
- 마지막으로 컨테이너 어노테이션 타입에는 적절한 `@Retention` 과 `@Target` 을 명시해야 한다.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class)
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}
```

**컨테이너 어노테이션**

```java
// 컨테이너 어노테이션
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
    ExceptionTest[] value();
}
```

**반복 가능 어노테이션을 두 번 단 코드**

```java
@ExceptionTest(IndexOutOfBoundsException.class)
@ExceptionTest(NullPointerException.class)
public static void doublyBad() {}
```

반복 가능 어노테이션을 처리할 때는 주의를 요한다. 

**반복 가능 어노테이션을 여러 개 달면 하나만 달았을 때와 구분하기 위해 해당 컨테이너 어노테이션 타입이 적용 된다.**

`getAnnotationsByType` 메서드는 이 둘을 구분하지 않아서, 반복 가능 어노테이션과, 그 컨테이너 어노테이션을 모두 가져오지만, isAnnotationPresent 메서드는 둘을 명확히 구분 한다.

그래서 둘을 따로따로 확인 해야 한다.

```java
public class RunTests {
    public static void main(String[] args) throws Exception {
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName(Sample.class.getName());
        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(ExceptionTest.class)
                    || m.isAnnotationPresent(ExceptionTestContainer.class)) {
                tests++;
                try {
                    m.invoke(null);
                    System.out.printf("테스트 %s 실패 : 예외를 던지지 않음 \n", m);
                } catch (InvocationTargetException wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    int oldPassed = passed;
                    ExceptionTest[] excTests =
                            m.getAnnotationsByType(ExceptionTest.class);
                    for (ExceptionTest excTest : excTests) {
                        if (excTest.value().isInstance(exc)) {
                            passed++;
                            break;
                        }
                    }
                    if (passed == oldPassed) {
                        System.out.printf("테스트 %s 실패 :%s%n", m, exc);
                    }
                }
                System.out.printf("성공 : %d, 실패 : %d%n ",
                        passed, tests - passed);
            }
        }
    }
}
```

- 반복 가능 어노테이션을 사용하면 코드 가독성을 개선할 수 있다.
- 하지만 어노테이션을 처리하는 부분의 코드 양이 늘어 난다.
- 어노테이션으로 할 수 있는 일을 명명 패턴으로 처리하지 말자.