---
layout: post
title: 익명 클래스보다는 람다를 사용하자
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---



# 익명클래스

예전에는 자바에서 함수 타입을 표현할 때 추상 메서드를 하나만 담은 인터페이스를 사용했다. 이런 인터페이스의 인스턴스를 함수 객체라고 하여, 특정 함수나 동작을 나타내는 데 썼다.

1997년 JDK1.1 부터 함수 객체를 만드는 주요 수단은 익명 클래스를 많이 사용 했다.

```java
Collections.sort(words, new Comparator<String>() {
  public int compare(String s1, String s2) {
    return Integer.compare(s1.length(), s2.length());
  }
});
```

[전략 패턴](https://donghyeon.dev/design%20pattern/2020/05/17/%EC%A0%84%EB%9E%B5-%ED%8C%A8%ED%84%B4/) 처럼, 함수 객체를 사용하는 과거 객체 디자인 패턴에는 익명 클래스면 충분했지만, 익명 클래스의 방식은 코드가 너무 길기 때문에 자바는 함수형 프로그래밍에 적합하지 않았다.



# 람다

자바 8에 와서 추상 메서드 하나짜리 인터페이스는 특별한 의미를 인정받아 특별한 대우를 받게 되었다.

지금은 함수형 인터페이스라 부르는 이 인터페이스들의 인스턴스를 람다식을 사용해 만들 수 있게 된 것이다.

람다는 함수나 익명 클래스와 개념은 비슷하지만 코드는 훨씬 간결하다.

**익명클래스 대신 람다를 사용한 코드**

```java
Collections.sort(words,
                (s1, s2) -> Integer.compare(s1.length(), s2.length()));
```

여기서 람다와 매개변수(s1,s2) 반환값의 타입은 람다는 `Comparator<String>` 을, 매개변수들은, `String` 과 `int` 를 반환하지만 코드에서는 언급이 없다.

우리 대신 컴파일러가 문맥을 살펴 타입을 추론해준 것이며, 상황에 따라 컴파일러가 타입을 결정하지 못하면, 개발자가 직접 명시해야 한다.

> 타입 추론에 필요한 정보는 대부분 제너릭에서 얻기 때문에, 제너릭을 잘 활용해야 한다.

람다 자리에 비교자 생성 메서드를 사용하면 코드를 더 간결하게 만들 수 있다.

```java
Collections.sort(words, comparingInt(String::length));
```

더 나아가 자바 8 때 List 인터페이스에 추가 된 sort 메서드를 이용하면 더욱 짧아진다.

```java
words.sort(comparingInt(String::length));
```





# 열거타입 리팩토링

이전 장에서 살펴봤던 Operation 열거 타입을 리팩토링 해보자.

```java
public enum Operation {
    PLUS {
        @Override
        public double apply(double x, double y) {
            return x + y;
        }
    },
    MINUS {
        @Override
        public double apply(double x, double y) {
            return x - y;
        }
    }, 
    TIMES {
        @Override
        public double apply(double x, double y) {
            return x * y;
        }
    },
    DIVIDE {
        @Override
        public double apply(double x, double y) {
            return x / y;
        }
    };

    public abstract double apply(double x, double y);
}
```

이 열거타입은 apply 메서드의 동작이 상수마다 달라야 해서 상수별 클래스 몸체를 사용해 각 상수에서 apply 메서드를 재정의 했다.

람다를 이용하면 더욱 축약이 가능해진다. 

**람다를 인스턴스 필드에 저장해 상수별 동작을 구현한 열거 타입**

```java
public enum Operation {
    PLUS("+", (x, y) -> x + y),
    MINUS("-", (x, y) -> x - y),
    TIMES("*", (x, y) -> x * y),
    DIVIDE("/", (x, y) -> x / y);

    private final String symbol;
    private final DoubleBinaryOperator op;

    Operation(String symbol, DoubleBinaryOperator op) {
        this.symbol = symbol;
        this.op = op;
    }

    @Override
    public String toString() {
        return symbol;
    }

    public double apply(double x, double y) {
        return op.applyAsDouble(x, y);
    }
}
```



## 람다의 단점

- 메서드나 클래스와 달리, 람다는 이름이 없고 문서화도 못한다. 따라서 코드 자체로 동작이 명확히 설명되지 않거나 코드 줄 수가 많아지면 람다를 쓰지 말아야 한다. 
- 람다는 한 줄 일때 가장 좋고, 길어야 세줄 안에 끝내면 좋다. 세줄이 넘어가면 가독성이 떨어진다. 세줄이 넘어가면 람다를 안쓰는쪽으로 생각해보자.
- 열거 타입 생성자에 넘겨지는 인수들의 타입도 컴파일타임에 추론된다. 따라서 열거 타입 생성자 안의 람다는 열거 타입의 인스턴스 멤버에 접근할 수 없다 (인스턴스는 런타임에 만들어지기 때문이다.)
  - 따라서 상수별 동작을 단 몇 줄로 구현하기 어렵거나, 인스턴스 필드나 메서드를 사용해야만 하는 상황이라면 상수별 클래스 몸체를 사용해야 한다.
  - 위에 예제코드에서 람다안에서 smybol을 사용할 수 없다.
- 람다는 함수형 인터페이스에서만 쓰인다.
  - 추상클래스의 인스턴스를 만들 때 람다를 쓸 수 없으니, 익명 클래스를 사용해야 한다.
  - 추상 메서드가 여러 개인 인터페이스의 인스턴스를 만들 때도 익명 클래스로 쓸 수 있다.
  - 람다도 익명 클래스처럼 직렬화 형태가 구현별로 다를 수 있으니, 직렬화 하는 일을 극히 삼가해야 한다.
    - 직렬화 하려면 private 정적 중첩 클래스의 인스턴스를 사용하자.



## 람다의 this

람다는 자신을 참조할 수 없다. 람다에서의 this 키워드는 바깥 인스턴스를 가리킨다. 반면 익명 클래스에서의 this는 익명 클래스의 인스턴스 자신을 가리킨다.

그래서 함수 객체가 자신을 참조해야 한다면 반드시 익명 클래스를 써야 한다.

```java
public class Main {

     public static void main(String[] args) {
          new LambdaTest().doLambda().doSomething(); //2가 출력 
          new AnonyTest().doAnony().doSomething(); // 10이 출력
      }
}

interface EffectiveTest {

    void doSomething();
}

class LambdaTest {
    int a = 2;

    public EffectiveTest doLambda() {
        return () -> {
            int a = 10;
            System.out.println(this.a);
        };
    }
}

class AnonyTest {
    int a = 2;

    public EffectiveTest doAnony() {
        return new EffectiveTest() {
            int a = 10;
            @Override
            public void doSomething() {
                System.out.println(this.a);
            }
        };
    }
}
```

