---
layout: post
title: 탑레벨 클래스는 한 파일에 하나만 담자
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---

소스 파일 하나에 탑레벨 클래스를 여러 개 선언하더라도 컴파일러 에러가 나진 않는다. 하지만 아무런 득도 없고, 심각한 위험을 감수해야 할 수도 있다. 이렇게 하면 한 클래스를 여러 가지로 정의할 수 있으며, 그 중 어느것을 사용할지는 어느 소스 파일을 먼저 컴파일 하냐에 따라 달라지기 때문이다.

다음과 같은 Main 클래스가 있다고 가정해보자.

```java
public class Main {
     public static void main(String[] args) {
       System.out.println(Utensil.NAME + Dessert.NAME);
     }
}
```

이 클래스는 Utensil 클래스에 있는 정적 변수인 NAME과 Dessert 클래스에 있는 정적 변수인 NAME을 조합해서 출력한다.

**Utensil.java**

```java
class Utensil {
    static final String NAME = "pan";
}

class Dessert {
    static final String NAME = "cake";
}
```

**Utensil 클래스**를 만들어서 위와 같이 탑레벨 클래스를 두개 만들었다.



**Dessert.java**

```java
class Utensil {
    static final String NAME = "pot";
}

class Dessert {
    static final String NAME = "pie";
}
```

**Dessert 클래스**를 만들어서 위와 같이 탑레벨 클래스를 두개 만들었다.



## 문제점

위와 같이 `Utensil.java` 와  `Dessert.java` 를 두개 만들면, Intellij에서는 중복된 클래스가 있다고 컴파일 에러가 뜬다.

하지만 javac로 컴파일한다면 문제가 생긴다.

운좋게 `javac Main.java Deesert.java `  명령으로 컴파일한다면 중복 정의가 있다고 컴파일 오류가 난다. <u>컴파일러는 가장 먼저 Main.java를 컴파일 한 후 그 안에서 Utensil을 만나면, Utensil.java에 가서 Utensil과 Dessert를 모두 찾아내고, 그런 다음 Dessert를 처리하려고 할 때 같은 클래스의 정의가 이미 있음을 알게 된다.</u>

그러나, `javac Main.java` 나 `javac Main.java Utensil.java` 컴파일하면 **pancake**를 출력하고, `javac Dessert.java Main.java` 명령으로 컴파일하면 **potpie**이 출력된다. 

이처럼 컴파일러에 어느 소스 파일을 먼저 건네느냐에 따라 동작이 달라지므로 반드시 잡아야할 문제다.

## 해결하기

해결법은 단순히 탑레벨 클래스들을 서로 다른 소스 파일로 분리하면 된다. 굳이 한 파일에 담고 싶으면 정적 멤버 클래스를 만드는게 일반적으로 더 나은 방법이다. 읽기도 좋고, private으로 선언하면 접근 범위도 최소로 관리할 수 있기 때문이다.

```java
public class Main {
     public static void main(String[] args) {
          System.out.println(Utensil.NAME + Dessert.NAME);
     }
     private static class Utensil {
          static final String NAME = "pan";
     }
     private static class Dessert {
          static final String NAME = "cake";
     }     
}
```

## 정리

**소스 파일 하나에는 반드시 탑레벨 클래스(혹은 탑레벨 인터페이스)를 하나만 담자.** 