---
layout: post
title: 가변인수는 시중히 사용하자
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---



# 가변인수

가변인수 메서드는 명시한 타입의 인수를 0개 이상 받을 수 있다.

**가변인수 메서드를 호출하면, 가장 먼저 인수의 개수와 길이가 같은 배열을 만들고 인수들을 이 배열에 저장하여 가변인수 메서드에 건네 준다.**

```java
static int sum(int... args) {
  int sum = 0;
  for (int arg : args) 
    sum += arg;
  return sum;
}
```



## 인수가 1개 이상이어야 하는 가변인수 메서드

가변인수에 인수가 1개 이상을 받아야 한다는 제약 조건을 넣어보자.

예를 들면 최솟값을 찾는 메서드일 때는 인수 1개이상을 받도록 설계해야한다.

인수 개수는 런타임에 배열의 길이로 알 수 있다.

```java
static int min(int... args) {
  if (args.length == 0)
    throw new IllegalArgumentException("인수가 1개 이상 필요합니다.");
  int min = args[0];
  for (int i = 1; i < args.length; i++)
    if (args[i] < min)
      min = args[i];
  return min;
}
```

이 방식에는 문제가 몇 개 있다.

- 인수를 0개만 넣어 호출하면, 컴파일타임 에러가아닌, 런타임 에러가 떨어진다.
- args 유효성 검사를 명시적으로 해야 한다.

이 방법 보다는, 매개변수를 2개 받고록 하면 된다.

첫 번째로는 평범한 매개변수를 받고, 가변인수는 두 번째로 받으면 앞의 문제가 깔끔히 사라진다.

```java
static int min(int firstArg, int... remainingArgs) {
  int min = firstArg;
  for (int arg : remainingArgs)
    if (arg < min)
      min = arg;
  return min;
}
```



# 가변인수의 성능이슈

가변인수 메서드는 호출될 때마다 배열을 새로 하나 할당하고 초기화 한다.

다행히, 이 비용을 감당할 수는 없지만 가변인수의 유연성이 필요할 때 선택할 수 있는 패턴이 있다.

예를 들어 해당 메서드 호출의 95%가 인수를 3개 이하로 사용한다고 해보자.

그렇다면 다음처럼 인수가 0개인 것 부터 4개인 것까지, 총 5개 다중정의하자.

마지막 다중정의 메서드가 인수 4개 이상인 5%의 호출을 담당하게 된다.

```java
public void foo() {}
public void foo(int a1) {}
public void foo(int a1, int a2) {}
public void foo(int a1, int a2, int a3) {}
public void foo(int a1, int a2, int a3, int... rest) {}
```

따라서 메서드 호출 중 단 5%만이 배열을 생성한다. 

대다수의 성능 최적화와 마찬가지로 이 기법도 보통 때는 별 이득이 없지만, 꼭 필요한 특수 상황에서는 쓸만하다.

EnumSet의 정적 팩터리도 이 기법을 사용해 열거 타입 집합 생성 비용을 최소화 한다.

EnumSet은 비트 필드를 대체하면서 성능까지 유지해야 하므로 아주 적절하게 활용한 예라 할 수 있다.

또한 java 9에서 나온 List 정적팩토리 메서드인 List.of() 도 비슷하게 사용하고 있다.

```java
static <E> List<E> of() {
  return (List<E>) ImmutableCollections.EMPTY_LIST;
}
static <E> List<E> of(E e1) {
  return new ImmutableCollections.List12<>(e1);
}


static <E> List<E> of(E e1, E e2) {
  return new ImmutableCollections.List12<>(e1, e2);
}


static <E> List<E> of(E e1, E e2, E e3) {
  return new ImmutableCollections.ListN<>(e1, e2, e3);
}


static <E> List<E> of(E e1, E e2, E e3, E e4) {
  return new ImmutableCollections.ListN<>(e1, e2, e3, e4);
}


static <E> List<E> of(E e1, E e2, E e3, E e4, E e5) {
  return new ImmutableCollections.ListN<>(e1, e2, e3, e4, e5);
}
```

