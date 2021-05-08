---
layout: post
title: ordinal 메서드 대신 인스턴스 필드를 사용하자
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---



# Enum의 ordinal() 메서드

열거 타입은 해당 상수가 그 열거 타입에서 몇 번재 위치인지를 반환하는 ordinal 이라는 메서드를 제공한다.

```java
public enum Ensemble {
  SOLO, DUET, TRIO, QUARTET;
  
  public int numberOfMusicians() { return ordinal() + 1}
}
```

동작은하지만 유지보수하기에는 좋지 않다.

**<u>상수 선언 순서를 바꾸는 순간</u>** `numberOfMusicians()` 메서드는 오동작하며, 이미 사용 중인 정수와 값이 같은 상수를 추가할 수 없다.

또한 중간에 숫자를 건너띄고 싶어도 할 수가 없어서, 더미 상수를 추가를 해야 한다.

이와 비슷한 문제로 [JPA에서 ENUM을 저장할 때](https://donghyeon.dev/jpa/2019/10/27/%ED%95%84%EB%93%9C%EC%99%80-%EC%BB%AC%EB%9F%BC-%EB%A7%A4%ED%95%91-%ED%95%84%EB%93%9C%EC%99%80-%EC%BB%AC%EB%9F%BC-%EB%A7%A4%ED%95%91/), ENUM 순서로 저장할지, ENUM의 값을 저장할지 결정할 수 있다.



# 해결법

열거 타입 상수에 연결된 값은 ordinal 메서드로 얻지 못하게 하고, 인스턴스 필드에 저장하면 된다.

```java
public enum Ensemble {
  SOLO(1), DUET(2), TRIO(3), QUARTET(4);
  
  private final int numberOfMusicians;
  Ensemble(int size) {this.numberOfMusicians = size;}
  public int numberOfMusicians() {return numberOfMusicians;}
}
```



## ordinal() 존재 이유

Enum API 문서에 보면 ordinal에 대해 이렇게 쓰여 있다.

> 대부분 프로그래머는 이 메서드를 쓸 일이 없다. 이 메서드는 EnumSet과 EnumMap 같이 열거 타입 기반의 범용 자료구조에 쓸 목적으로 설계 되었다.

따라서 이런 용도가 아니라면 ordinal 메서드는 절대 사용하지 말자.