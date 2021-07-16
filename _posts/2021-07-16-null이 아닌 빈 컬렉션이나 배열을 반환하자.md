---
layout: post
title: null이 아닌 빈 컬렉션이나 배열을 반환하자
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---



# 컬렉션이 비어 있으면 null을 반환하는 메서드

```java
private final List<Cheese> cheesesInStock = new ArrayList<>();

public List<Cheese> getCheeses() {
  return cheesesInStock.isEmpty() ? null : new ArrayList<>();
}
```

치즈의 재고가 없다고 null을 반환한다면, 클라이언트는 이 null 상황을 처리하는 코드를 추가로 작성 해야 한다.

```java
List<Cheese> cheeses = shop.getCheeses();
if(cheeses != null && cheeses.contains(Chesses.STILTON)) {
  ...
}
```

컬렉션이나 배열 같은 컨테이너가 비었을 때 null을 반환하는 메서드를 사용할 때면 이와 같은 방어 코드를 항상 넣어줘야 한다.

클라이언트에서 방어 코드를 빼먹으면 오류가 발생할 수 있다.



## 빈 컨테이너를 할당 비용 vs null 반환

빈 컨테이너를 할당하는 비용 때문에 null을 반환 하는게 나을까?

- 성능 분석 결과 이 할당이 성능 저하의 주범이라고 확인되지 않는 한, 이 정도의 성능 차이는 신경 쓸 수준이 못 된다.
- 빈 컬렉션과 배열은 굳이 새로 할당하지 않고도 반환할 수 있다.

**빈 컬렉션을 반환하는 올바른 예**

```java
public List<Cheese> getCheeses() {
  return new ArrayList<>();
}
```

가능성은 작지만 사용 패턴에 따라 빈 컬렉션은 할당 하는 것은 성능을 떨어뜨릴 위험이 있다.

**해법은 매번 똑같은 빈 불변 컬렉션을 반환하는 것이다.**

Collections.emptyList 메서드나 Collections.emptySet 등을 사용하면 된다.

**매번 빈 컬렉션을 할당하지 않도록 한 코드**

```java
public List<Cheese> getCheeses() {
  return cheesesInStock.isEmpty() ? Collections.emptyList() : new ArrayList<>();
}
```

배열을 사용할 때도 마찬가지로, 절대 null을 반환하지 말고 길이가 0인 배열을 반환 하자.

**길이가 0일 수도 있는 배열을 반환하는 올바른 방법**

```java
public Cheese[] getCheeses() {
  return cheesesInStock.toArray(neww Cheese[0]);
}
```

이 방식도 최적화를 할 수 있다.

길이 0짜리 배열을 미리 선언해두고 매번 그 배열을 반환하면 된다. 길이 0인 배열은 모두 불변이기 때문이다.

**빈 배열을 매번 새로 할당하지 않는 코드**

```java
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses() { 
  return cheeseInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```



## 잘못된 최적화 방법

단순히 성능을 개선할 목적이라면 toArray에 넘기는 배열을 미리 할당하는 건 추천하지 않는다.

오히려 성능이 떨어 질 수 있다.

```java
return cheesesInStock.toArray(new Cheese[cheesesInStock.size()]);
```

`List.toArray(T[] a)` 메서드는 주어진 배열 a가 충분히 크면 a 안에 원소를 담아 반환하고, 그렇지 않으면 **T[]** 타입 배열을 새로 만들어 그 안에 원소를 담아 반환한다. 

따라서 원소가 하나라도 있다면 Cheese[] 타입의 배열을 새로 생성해 반환하고, 원소가 0개면 EMPTY_CHEESE_ARRAY를 반환한다.

