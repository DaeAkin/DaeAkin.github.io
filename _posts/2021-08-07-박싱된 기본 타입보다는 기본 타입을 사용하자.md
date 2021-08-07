---
layout: post
title: 박싱된 기본 타입보다는 기본 타입을 사용하자,
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---



# 기본타입과 박싱된 기본 타입의 차이점

- 기본 타입은 값만 가지고 있으나, 박싱된 기본 타입은 값에 더해 식별성이란 속성을 갖는다.
  - 박싱된 기본 타입의 두 인스턴스는 값이 같아도 서로 다르다고 식별될 수 있다.
- 기본 타입의 값은 언제나 유효하나, 박싱된 기본 타입은 null을 가질 수 있다.
- 기본 타입이 박싱된 기본 타입보다 시간과 메모리 사용면에서 더 효율적이다.

# 문제점

```java
Comparator<Integer> naturalOrder = 
  (i, j) -> (i < j) ? -1 : (i == j ? 0 : 1);
```

> 비교자의 compare 메서드는 
>
> 첫 번째 원소가 두 번째 원소보다
>
> - 작으면 음수
> - 같으면 0
> - 크면 양수
>
> 를 반환한다. 

이 코드는 별다른 문제를 찾기 어렵고, 실제로 이것저것 테스트해봐도 잘 통과한다.

하지만 `naturalOrder.compare(new Integer(42),new Integer(42))` 의 값을 출력하면 0이 나와야 하는데 1이 나온다.

즉 첫 번째 Integer가 두 번째보다 크다고 주장한다.

## 원인

- i < j 를 비교할 때 기본 타입 값으로 변환되어 42 < 42 를 비교한다.
- i == j 를 비교할 때는 박싱된 기본타입을 이용하기 때문에 같지 않다고 판단한다.



## 해결

이 문제를 해결하려면 == 연산에도 기본 타입을 사용하게 하면 된다.

```java
Comparator<Integer> naturalOrder = (iBoxed, jBoxed) -> {
  int i = iBoxed, j = jBoxed;
  return i < j ? -1 : (i == j ? 0 : 1);
}
```



# null 문제점

```java
public class Unbelievable {
  static Integer i;
  
  public static void main(String[] args) {
    if (i == 42)
      System.out.println("hi");
  }
}
```

이 코드는 `hi` 를 출력하지 않지만, `i == 42 ` 를 검사할때 NullPointerException을 던진다.

## 원인

원인은  i가 int가 아닌I nteger이며, **거의 예외 없이 기본 타입과 박싱된 기본 타입을 혼용한 연산에서는 박싱된 기본 타입의 박싱이 자동으로 풀린다.**

그리고 null 참조를 언박싱하면 NullPointerException이 발생한다.

## 해결

간단히 i를  int로 선언해주면 끝이다. 



# 성능 문제점

```java
public static void main(String[] args) {
  Long sum = 0L;
  for (long i =0; i <= Integer.MAX_VALUE; i++) {
    sum += i;
  }
  System.out.println(sum);
}
```

이 코드는 실수로 지역변수sum을 박싱된 기본 타입으로 선언해서 느려졌다.

오류나 경고 없이 컴파일되지만, 박싱과 언박싱이 반복해서 일어나 체감될 정도로 성능이 느려진다.



# 박싱된 기본 타입을 쓰는 곳

- 컬렉션의 원소, 키 , 값으로 쓴다. 컬렉션은 기본 타입을 담을 수 없으므로 어쩔 수 없이 박싱된 기본 타입을 써야만 한다.
- 매개변수화 타입이나 매개변수화 메서드의 타입 매개변수로는 박싱된 기본 타입을 써야 한다. 자바는 타입 매개변수로 기본 타입을 지원하지 않는다.
  - `ThreadLocal<int>` 대신 `ThreadLocal<Integer>` 
- 리플렉션을 통해 메서드를 호출할 때도 박싱된 기본 타입을 사용해야 한다.
