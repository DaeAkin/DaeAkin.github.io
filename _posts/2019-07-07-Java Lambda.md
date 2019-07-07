---
layout: post
title: Java Lamda
categories: [Java]
comments: true

---

# 람다
[예제소스](https://github.com/DaeAkin/Java8Lambda)

람다를 사용함으로써 얻는 이득을 알기위해서는 람다를 이해해야 합니다.

### 람다의 이점

- 함수형 프로그래밍 사용
- 코드의 가독성과 간결함 상승
- API와 라이브러리를 사용하기 쉬움 
- 병렬 처리 지원 가능

### OOP의 코드

객체지향프로그래밍에서는 모든 것이 오브젝트입니다.

- 모든것이 객체로 이루어짐
- 모든코드의 블록은 클래스와 객체로 연관되어있다.

만약 Hello World! 코드를 찍는다면, Greet 클래스를 만들고, 함수하나를 만들어야합니다.

만약 "난 클래스가 필요없고 그냥 로직 하나가 필요해"라고 한다면 이때 람다를 사용합니다.

람다는 Java 8버전부터 사용가능합니다. 

보통 변수에 값을 할당할 때 다음과 같이 작성합니다.

```java
String name = "foo"
double pi = 3.14
```

하지만 이 변수에 값을 할당하는게 아니라, <u>코드블록</u>을 할당하고 싶다면 어떻게해야할까요?

## 람다함수로 변경하기

> #### 변경전

다음과 같이 aBlockOfCode에 HelloWorld! 출력하는 함수를 할당해보도록 하겠습니다.

```java
aBlockOfCode	= public void perform() {
  System.out.print("Hello World!");
}
```

### 람다의 특징

- public 접근자가 필요 없음
- 함수이름도 필요없음. 
- 람다가 알아서 추측하기 때문에 리턴값도 필요없음.


> #### 변경후

```java
aBlockOfCode = () -> {
	 System.out.print("Hello World!");
};
```

> 이 소스를 그대로 IDE에 작성하시면 컴파일 오류가 납니다. 이는 타입을 지정해주지 않아서 생기는 오류인데요,
>
> 메소드로 작성하는 방법과 람다로 작성하는 방법의 차이를 명확히 보여드리기 위해서 작성되었습니다.
>
> 컴파일 오류를 해결하는 예제는 계속해서 나옵니다.

> #### 람다코드가 한줄이라면 다음과 같이 사용할 수 있습니다.

```java
aBlockOfCode = () -> System.out.print("Hello World!");
```

람다가 한줄만에 끝난다면 `{ }` 을 사용하지 않아도 됩니다.

## 파라미터가 있는 람다함수

> #### 변경 전

```java
doubleNumberFunction = public int double(int a) {
  return a * 2;
}
```

> #### 변경 후

```java
doubleNumberFunction = (int a) -> return a * 2;
```



> #### 람다코드가 한줄이라면 return 문장도 없앨 수 있다.

```java
doubleNumberFunction =(int a) -> a * 2;
```



## 람다의 장점

Hello Wolrd를 출력하는 인터페이스와 클래스를 이용해서 람다의 장점을 더 알아보겠습니다.

> #### Greeting 인터페이스 

```java
public interface Greeting {
    public void perform();
}
```

Greeting 인터페이스를 이용해서 람다를 좀 더 이용해 보겠습니다.

> #### HelloWorldGreeting 클래스

```java
public class HelloWorldGreeting implements Greeting {

    @Override
    public void perform() {
        System.out.println("Hello World!!");
    }
}
```

Java 8 이전을 사용했다면 인터페이스를 구현해서 이용했습니다.

### 인터페이스로 구현한 Hello World

> #### Greeter 클래스 

```java
public class Greeter {

     public static void main(String[] args) {
        Greeting helloWorldGreeting = new HelloWorldGreeting();
        helloWorldGreeting.perform();
      }
}

```

> #### Output

```
Hello World!!
```

### 내부 익명 함수로 구현한 Hello World

> #### Greeter 클래스 

```java
public class Greeter {
     public static void main(String[] args) {

        Greeting helloWorldGreeting = new HelloWorldGreeting();

        helloWorldGreeting.perform();

        Greeting anonymousHello = new Greeting() {
              @Override
              public void perform() {
                  System.out.println("anonymous Hello!");
              }
          };
        anonymousHello.perform();
      }
}

```

> #### Output

```
Hello World!!
anonymous Hello!
```

### 람다로 구현한 Hello World

그러나 Java 8 부터는 람다를 이용할 수 있습니다.

> #### Greeter 클래스 

```java
public class Greeter {

     public static void main(String[] args) {

        Greeting helloWorldGreeting = new HelloWorldGreeting();

        helloWorldGreeting.perform();

        Greeting anonymousHello = new Greeting() {
            @Override
            public void perform() {
                System.out.println("anonymous Hello!");
            }
        };

        anonymousHello.perform();

        Greeting lambdaHelloWorld = () -> System.out.println("lambda Hello!");

        lambdaHelloWorld.perform();

      }
}
```

> #### Output

```
Hello World!!
anonymous Hello!
lambda Hello!
```



## 람다의 타입

람다의 타입은 무슨타입일까요?

문자를 가지는 변수라면 String의 타입이고, 숫자를 가지는 변수라면 int타입일 것입니다.

람다가 어떻게 타입을 가지는지 알아보겠습니다.

위에 있는 예제를 보면

```java
Greeting lambdaHelloWorld = () -> System.out.println("lambda Hello!");
```

lambdaHelloWorld의 타입은 <u>Greeting 타입</u>입니다.

```java
public interface Greeting {
    public void perform();
}
```

람다의 타입을 가질 인터페이스를 만드는데는 몇가지 조건이 있습니다.

```java
public interface Greeting {
    public void perform();
  	public int getSomething();
}
```

이렇게 인터페이스에 두개의 메소드가 정의되어있으면, 컴파일러는 람다가 어떤 메소드를 구현했는지,

헷갈리기 때문에 <u>메인메소드에</u> 오류가나게 됩니다.

하지만 다른프로그래머가 이 인터페이스는 람다를 위해 만든것이며,

새로운 메소드를 실수로 추가하지 않도록 하려면 어떻게 해야할까요?

그럴땐 `@FunctionalInterace` 어노테이션을 작성하면 됩니다.

```java
@FunctionalInterface
public interface Greeting {
    public void perform();
}

```

만약 인터페이스안에 새로운 메소드를 작성한다면, 컴파일러가 인터페이스에 오류를 냅니다.



## Collection 활용

> #### Person 클래스

```java
public class Person {

    String firstName;
    String lastName;
    int age;

    Person() {

    }

    Person(String firstName,String lastName,int age) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.age = age;
    }

    public String getFirstName() {
        return firstName;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

    public String getLastName() {
        return lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person [firstName=" + firstName + ", lastName=" + lastName + ", age=" + age + "]";
    }
}
```



> #### StandardFunctionalPerson 클래스 

```java
import java.util.*;

public class StandardFunctionalPerson {
     public static void main(String[] args) {
          List<Person> people = Arrays.asList(
                  new Person("동현","민",24),
                  new Person("인규","박",22),
                  new Person("석호","조",26),
                  new Person("경훈","민",23)
          );

          //성으로 정렬하기
         Collections.sort(people, new Comparator<Person>() {
             @Override
             public int compare(Person o1, Person o2) {
                 return o1.getFirstName().compareTo(o2.getLastName());
             }
         });
         System.out.println("printAll");
         // 전부다 출력하기
            perform(people, new Condition() {
                @Override
                public boolean test(Person p) {
                    return true;
                }
            });

            System.out.println("printAll Conditionally");
         // 성이 민씨인 사람만 출력하기
            perform(people, new Condition() {
                @Override
                public boolean test(Person p) {
                    return p.getLastName().startsWith("민");
                }
            });

      }

      public static void perform(List<Person> people,Condition condition) {
          for (Person p : people) {
              if(condition.test(p))
                System.out.println(p);
          }
      }
}

interface Condition {
    boolean test(Person p);
}
```



> #### Output

```java
printAll
Person [firstName=경훈, lastName=민, age=23]
Person [firstName=동현, lastName=민, age=24]
Person [firstName=인규, lastName=박, age=22]
Person [firstName=석호, lastName=조, age=26]
printAll Conditionally
Person [firstName=경훈, lastName=민, age=23]
Person [firstName=동현, lastName=민, age=24]
```

위에있는 예제는 Collection API를 이용한 정렬입니다.

이 API를 람다식으로 바꿔보겠습니다.

### 1단계 : 이름의 성으로 정렬하기

```java
 // 1단계 : 성으로 정렬하기
Collections.sort(people, (p1,p2) -> p1.getLastName().compareTo(p2.getLastName()));
```

sort 함수는 두개의 인자를 갖습니다.`sort(List<T> list,Comparator<? super T> c)`

첫번째 인자로는 정렬할 리스트, 두번째로는 Comparator 인터페이스를 구현해서 어떻게 정렬할건지 알려줘야합니다.

람다변수가 타입을 가지기 위해서는 인터페이스를 만들고, 그 인터페이스는 <u>하나의 메소드만</u> 정의되어 있어야합니다.

인터페이스를 람다로 구현하려면 한개의 구현되지 않은 메소드만 있어야 합니다.

> #### Comparator 인터페이스의 문서

![](https://github.com/DaeAkin/DaeAkin.github.io/blob/master/img/blog/lambda/image1.png?raw=true)

Comparator의 문서를 보면 다른 메소드는 default로 이미 구현이 되어 있습니다.

Java8부터는 인터페이스에 구현체를 넣을 수 있는데 앞에 default를 붙여줘야합니다.

구현이 안된 메소드는 `compare` 메소드 하나 이므 람다사용에 만족합니다.

### 2단계 : Collection에 있는 요소들을 전부 출력하기

```java
// 2단계 : 전부다 출력하기
perform(people, p -> true);
```





### 3단계 : 특정 조건을 가진사람만 출력하기

```java
 // 3단계 : 성이 민씨인 사람만 출력하기
    perform(people, p -> p.getLastName().startsWith("민"));
```



> #### 전체코드

```java
public class StandardFunctionalPerson {
     public static void main(String[] args) {
          List<Person> people = Arrays.asList(
                  new Person("동현","민",24),
                  new Person("인규","박",22),
                  new Person("석호","조",26),
                  new Person("경훈","민",23)
          );

          // 1단계 : 성으로 정렬하기
         Collections.sort(people, (p1,p2) -> p1.getLastName().compareTo(p2.getLastName()));

         System.out.println("printAll");
         // 2단계 : 전부다 출력하기
            perform(people, p -> true);

            System.out.println("printAll Conditionally");
         // 3단계 : 성이 민씨인 사람만 출력하기
            perform(people, p -> p.getLastName().startsWith("민"));

      }
    public static void perform(List<Person> people,Condition condition) {
        for (Person p : people) {
            if(condition.test(p))
                System.out.println(p);
        }
    }

}
interface Condition{
    boolean test(Person p);
}
```



이렇게 람다를 이용해서 코드를 리팩토링해봤습니다. 

그러나 이렇게 인터페이스를 만들고 람다를 만들고 하는 일을 내부 익명함수랑 비교해서 큰 이점을 나타나질 않습니다.

인터페이스를 반드시 만들어야할까요?

java 8부터 java.util.function 패키지가 생겼습니다.

이 패키지는 람다를 위한 패키지인데요, 람다를 위한 인터페이스들이 정의되어 있습니다.

```java
interface Condition{
    boolean test(Person p);
}
```

우리가 만든 Condition 인터페이스안에 있는 `test` 메소드는 리턴값을 boolean 이고 인자를 하나만 갖습니다.

이렇게 똑같은 리턴값을 가지고 인자를 하나만 가지는 인터페이스가 java.util.function 패키지안에 존재합니다.

`java.util.function.Predicate<T>`  메소드를 이용하면 리턴값이 boolean이고 인자를 한개받는 인터페이스를 만들지

않아도 됩니다.

> #### Condition 인터페이스를 삭제한 후 다시 리팩토링 한 모습

```java
import java.util.Arrays;
import java.util.Collections;
import java.util.List;
import java.util.function.Predicate;

public class StandardFunctionalPerson {
     public static void main(String[] args) {
          List<Person> people = Arrays.asList(
                  new Person("동현","민",24),
                  new Person("인규","박",22),
                  new Person("석호","조",26),
                  new Person("경훈","민",23)
          );

          // 1단계 : 성으로 정렬하기
         Collections.sort(people, (p1,p2) -> p1.getLastName().compareTo(p2.getLastName()));

         System.out.println("printAll");
         // 2단계 : 전부다 출력하기
            perform(people, p -> true);

            System.out.println("printAll Conditionally");
         // 3단계 : 성이 민씨인 사람만 출력하기
            perform(people, p -> p.getLastName().startsWith("민"));

      }

      public static void perform(List<Person> people, Predicate<Person> predicate) {
          for (Person p : people) {
              if(predicate.test(p))
                System.out.println(p);
          }
      }
}






```

Predicate 인터페이스말고 다른 인터페이스들도 정의되어있으니 문서를 보시는걸 추천드립니다.

[java.util.function 문서](https://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html)

## 예외처리

> #### ExceptionHandling 클래스

```java
public class ExceptionHandling {

     public static void main(String[] args) {
          int[] numbers = {1,2,3,4};
          int divide = 2;
        process(numbers,divide,(v,k) -> System.out.println(v/k));
      }

      public static void process(int[] number, int divide,
      	BiConsumer<Integer,Integer> consumer) {
         System.out.println("-- process --");
          for (int num: number) {
              consumer.accept(num,divide);
          }
      }
}
```

이 예제는 numbers의 값들을 divide 변수의 값으로 나누는 예제입니다.

process함수에서 BiConsumer 인터페이스를 사용했는데,

이 인터페이스는 두개의 인자를 갖고, 리턴값이 없는 람다를 이용할때 사용됩니다.

> #### Output

```java
-- process --
0
1
1
2
```

하지만 <u>divde의 값을 0으로</u> 두면 어떻게 될까요? 

> #### ArthmeticExcpetion  예외발생

```con
-- process --
Exception in thread "main" java.lang.ArithmeticException: / by zero
	at com.donghyeon.ex.tutorial2.tutorial3.ExceptionHandling.lambda$main$0(ExceptionHandling.java:10)
	at com.donghyeon.ex.tutorial2.tutorial3.ExceptionHandling.process(ExceptionHandling.java:16)
	at com.donghyeon.ex.tutorial2.tutorial3.ExceptionHandling.main(ExceptionHandling.java:10)
```

0으로 나눌수 없으니 예외를 던지게됩니다. 이 예외를 다루는법을 살펴보겠습니다.

> #### process 함수안에 try~catch 사용하기

```java
public static void process(int[] number, int divide, BiConsumer<Integer,Integer> consumer) {
   System.out.println("-- process --");
    for (int num: number) {
        try {
            consumer.accept(num, divide);
        } catch (ArithmeticException e) {
            System.out.println("Exception occur");
        }
    }
}
```

process 메소드 안에 try ~ catch를 이용해서 예외를 처리해줬습니다.

> #### try~catch 전용 함수 만들기 

```java
public class ExceptionHandling {

     public static void main(String[] args) {
          int[] numbers = {1,2,3,4};
          int divide = 0;
        process(numbers,divide, wrapperLambda((v,k) -> System.out.println(v/k)));
      }

      public static void process(int[] number, int divide, BiConsumer<Integer,Integer> consumer) {
         System.out.println("-- process --");
          for (int num: number) {
                  consumer.accept(num, divide);
          }
      }

    private static BiConsumer<Integer, Integer> wrapperLambda(BiConsumer<Integer, Integer> consumer) {
        return (v, k) ->  {
            try {
                consumer.accept(v, k);
            }
            catch (ArithmeticException e) {
                System.out.println("Exception caught in wrapper lambda");
            }

        };
    }
}
```

wrapperLamda 함수를 작성해서 예외처리를 해줄수도 있습니다.

## 병렬처리

병렬처리 추가예정.
