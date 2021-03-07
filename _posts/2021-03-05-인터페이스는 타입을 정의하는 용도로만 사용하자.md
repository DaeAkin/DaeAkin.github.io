---
layout: post
title: 인터페이스는 타입을 정의하는 용도로만 사용하자
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---



인터페이스는 자신을 구현한 클래스의 인스턴스를 참조할 수 있는 타입 역할을 한다. 달리 말해, 클래스가 어떤 인터페이스를 구현한다는 것은 자신의 인스턴스로 무엇을 할 수 있는지를 클라이언트에 얘기해주는 것이다. 인터페이스는 오직 이 용도로만 사용해야 한다.

## 상수 인터페이스

상수 인터페이스란 메서드 없이, static final 필드로만 가득찬 인터페이스이다.

**상수인터페이스**

```java
public interafce PhysicalConstants {
  static final double AVOGADROS_NUMBER = 6.022_140_857e23;
  
  static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
}
```

**이 방법은 인터페이스를 잘못 사용한 예다.**

클래스 내부에서 사용하는 상수는 내부 구현에 해당한다. 따라서 이 내부 구현을 클래스의 API로 노출하는 행위다. 이렇게 사용하면 캡슐화가 깨지며, 클라이언트 코드가 내부 구현에 해당하는 이 상수들에 종속된다. 그래서 다음 릴리스에서 이 상수들은 더 이상 사용하지 않음에도 불구하고 호환성을 위해 지속적으로 제공해줘야 한다. 또한 상수가 final 아니라면 모든 하위 클래스의 동일한 변수명을 사용하고 있는 공간에 인터페이스가 정의한 상수들로 오염이 된다.

## 상수가 필요하다면 이렇게

상수가 필요하다면 다음과 같이 하자.

특정 클래스나 인터페이스와 강하게 연관된 상수라면 그 클래스나 인터페이스 자체에 추가해야 한다. 우리가 자주쓰는 Integer와 Double에 선언된 MIN_VALUE와 MAX_VALUE 상수가 이런 예다. 

아니면 열거 타입으로 나타내기 적합하다면 열거 타입으로 만들면 된다.

그것도 아니라면 private 생성자를 이용하여 인스턴스화를 막아버린 유틸리티 클래스를 만들자.

**상수 유틸리티 클래스**

```java
public class PhysicalConstants {
  private PhysicalConstants() {}
  
  public static final double AVOGADROS_NUMBER = 6.022_140_857e23;
  
  public static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
}
```



## 정리

인터페이스는 타입을 정의하는 용도로만 사용해야 한다. 상수 공개용 수단으로 사용하지 말자.