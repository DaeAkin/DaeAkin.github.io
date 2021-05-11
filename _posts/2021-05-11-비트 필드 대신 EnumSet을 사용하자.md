---
layout: post
title: 비트 필드 대신 EnumSet을 사용하자
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---



# 비트 필드란?

```java
public class Text {
    public static final int STYLE_BOLD = 1 << 0; // 1
    public static final int STYLE_ITALIC = 1 << 1; // 2
    public static final int STYLE_UNDERLINE = 1 << 2; // 4
    public static final int STYLE_STRIKETHROUGH = 1 << 3; // 8

  // 매개변수 styles는 0개 이상의 STYLE_ 상수를 비트별 OR한 값이다.
    public void applyStyle(int style) {
      ...
    }

}
```



```java
public class Main {
     public static void main(String[] args) {
         Text text = new Text();
         text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
      }
}
```

위와 같이 비트별 OR를 사용해 여러 상수를 하나의 집합으로 모을 수 있으며, 이렇게 만들어진 집합을 비트 필드라고 한다.

- 비트별 연산을 사용해 합집합과 교집합 같은 집합 연산을 효율적으로 수행가능
  - 비트 연산은 매우 빠르다.
- 하지만 [정거 열거 상수](https://donghyeon.dev/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C%EC%9E%90%EB%B0%94/2021/05/08/int-%EC%83%81%EC%88%98-%EB%8C%80%EC%8B%A0-%EC%97%B4%EA%B1%B0-%ED%83%80%EC%9E%85%EC%9D%84-%EC%82%AC%EC%9A%A9%ED%95%98%EC%9E%90/)의 단점이 그대로 있음
- 비트 필드 값은 해석하기가 어렵다.
- 최대 몇 비트(32비트 or 64비트)가 필요한지 API 작성시 미리 예측해야 한다.
- 비트의 결과가 어떤 비트로 이루어져있는지 다시 찾아봐야 한다.

## 비트 필드의 대안

정수 상수보다 열거 타입을 선호하는 프로그래머 중에도 상수 집합을 주고 받아야 할 때 여전히 비트 필드를 사용하기도 한다.

**하지만** 더 나은 대안은 `java.util` 패키지의 EnumSet 클래스를 사용하면 열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현해준다. 

```java
public class Text {

    public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH}
    
    //어떤 Set을 넘겨도 된다.
    public void applyStyles(Set<Style> styles) {

    }
}
```



```java
public class Main {
     public static void main(String[] args) {
         Text text = new Text();
         text.applyStyles(EnumSet.of(Text.Style.BOLD, Text.Style.ITALIC));
         text.applyStyles(Set.of(Text.Style.BOLD, Text.Style.ITALIC));
      }
}
```

**Set 인터페이스를 구현한 모든 구현체를 사용해도 된다.**

하지만 그 중에서 <u>EnumSet</u>을 사용하면 좋은데, 그 이유는 다음과 같다.

- 원소가 64개 이하라면, 대부분의 경우 EnumSet 전체를 long 변수 하나로 표현하여 비트 필드에 비견되는 성능을 보여준다.
  - removeAll과 retainAll같은 대량 작업은 비트를 효율적으로 처리할 수 있는 산술 연산을 써서 구현했다.
- EnumSet이 다 처리해주기 때문에 비트를 직접 다룰 때 흔히 겪는 오류들에서 해방 된다.



# 정리

열거할 수 있는 타입을 한데 모아 집합 형태로 사용한다고 해도 비트 필드를 사용할 이유는 없다. EnumSet 클래스가 비트 필드 수준의 명료함과 성능을 제공하고 열거 타입의 장점까지 주기 때문이다.

EnumSet의 유일한 단점은 불변 EnumSet을 만들 수는 없지만, `Collections.unmodifiableSet`으로 EnumSet을 감싸 사용할 수 있다.