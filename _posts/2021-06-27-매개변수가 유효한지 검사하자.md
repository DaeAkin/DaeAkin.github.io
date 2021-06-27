---
layout: post
title: 매개변수가 유효한지 검사하자
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---



# 매개변수의 특정 조건

메서드와 생성자 대부분은 입력 매개변수의 값이 특정 조건을 만족하기를 바란다. 

예를 들어 인덱스 값은 음수이면 안 되며, 객체 참조는 null이 아니어야 하는 식이다.

이런 제약은 반드시 문서화해야 하며 메서드 몸체가 시작되기 전에 검사해야 한다.

이런 오류를 발생한 즉시 잡지 못하면 오류를 감지하기 어려워지고, 감지하더라도 오류의 발생 지점을 찾기 어려워 진다.



# 매개변수 조건 검사의 필요성

메서드가 실행되기전 매개변수를 확인한다면 잘못된 값이 넘어왔을 때 즉각적이고 깔끔한 방식으로 예외를 던질 수 있다.

하지만 매개변수를 제대로 하지 못하면 몇 가지 문제가 생길 수 있다.

- 수행되는 중간에 모호한 예외를 던지며 실패할 수 있다.
- 메서드가 잘 수행되지만 잘못된 결과를 반환할 수 있다.
- 메서드는 문제없이 수행됐지만, 어떤 객체를 이상한 상태로 만들어놓아서 미래의 알 수 없는 시점에 이 메서드와는 관련 없는 오류를 낼 수 있다.
- 실패 원자성을 어기는 결과를 낳을 수 있다.



# public과 protected 메서드

public과 protected 메서드는 매개변수 값이 잘못됐을 때 던지는 예외를 문서화 해야 한다.

매개변수의 제약을 문서화한다면 그 제약을 어겼을 때 발생하는 예외도 함께 기술해야 한다.

이런 간단한 방법으로 API 사용자가 제약을 지킬 가능성을 크게 높일 수 있다.

**BigInteger 클래스의 mod() 메서드**

```java
    /**
     * Returns a BigInteger whose value is {@code (this mod m}).  This method
     * differs from {@code remainder} in that it always returns a
     * <i>non-negative</i> BigInteger.
     *
     * @param  m the modulus.
     * @return {@code this mod m}
     * @throws ArithmeticException {@code m} &le; 0
     * @see    #remainder
     */
    public BigInteger mod(BigInteger m) {
        if (m.signum <= 0)
            throw new ArithmeticException("BigInteger: modulus not positive");

        BigInteger result = this.remainder(m);
        return (result.signum >= 0 ? result : result.add(m));
    }}
```

이 메서드는 m이 null이면 m.signum() 호출 때 NullPointerException을 던지는데, 그런 설명이 없다.

그 이유는 이 설명을 BigInteger 클래스 수준에서 설명했기 때문이다.

**클래스 수준 주석**은 그 클래스의 모든 public 메서드에 적용되므로 각 메서드에 일일이 기술하는 것보다 깔끔하다.

@Nullable이나 이와 비슷한 어노테이션을 사용해 특정 매개변수는 null이 될 수 있다고 알려줄 수 있지만 표준적인 방법은 ㅇ니다.



# Objects.requireNonNull()

자바 7에 추가된 `java.util.Objects.requireNonNull` 메서드는 유연하고 사용하기도 편하니, 더 이상 null 검사를 수동으로 하지 않아도 된다.

원하는 예외 메세지도 지정할 수 있고, 입력을 그대로 반환하므로 값을 사용하는 동시에 null 검사도 수행할 수 있다.

```java
this.strategy = Objects.requireNonNull(strategy, "전략");
```



# 자바9의 범위 검사

자바 9부터는 범위검사도 가능하다

- checkFromIndexSize
- checkFromToIndex
- checkIndex

를 사용하면 된다.

null 검사 메서드만큼 유연하지는 않다. 예외 메세지를 지정할 수 없고, 리스트와 배열 전용으로 설계됐으며, 닫힌범위는 다루지 못하지만, 유용할 때가 있다.



# private 메서드

private 메서드라면 패키지 제작자인 우리가 메서드가 호출되는 상황을 통제할 수 있다.

따라서 오직 유효한 값만이 메서드에 넘겨지는 것을 보증해야 한다.

```java
private static void sort(long a[], int offset, int length) {
  assert a != null;
  assert offset >= 0 && offset <= a.length;
  assert length >= 0 && length <= a.length - offset;
  ..// 로직
}
```

assert 단언문은 일반적인 유효성 검사와 다르다.

- 실패하면 AssertionError를 던진다
- 런타임에 아무런 효과도 성능 저하도 없다.
  - 단 java를 실행할 때 명령줄에서 -ea 혹은 --enableassertions 플래스 설정을 하면 영향을 준다.



# 유효성 검사 예외

가끔은 유효성 검사를 하지 않아도 될 때가 있다.

- 유효성 검사 비용이 지나치게 높거나 실용적이지 않을 때
- 계산 과정에서 암묵적으로 수행될 때

예를 들어 `Collections.sort(List)` 처럼 정렬하는 메서드는 리스트 안의 객체들은 모두 상호 비교될 수 있어야 하며, 정렬 과정에서 이 비교가 이뤄진다.

만약 상호 비교될 수 없는 타입의 객체가 들어 있다면, 그 객체와 비교할 때 `ClassCastException` 을 던진다. 

따라서 비교하기 앞서 리스트 안의 모든 객체가 상호 비교될 수 있는지 검사해봐야 별다른 이득이 없다.

하지만 암묵적 유효성 검사에 너무 의존했다가는 실패 원자성을 해칠 수 있으니 주의하자.



# 마무리

매개변수에 제약을 두는게 좋다라고 해석하면 안 된다. 사실은 그 반대다. 

메서드는 최대한 범용적으로 설계해야 한다. 메서드가 건네받은 값으로 무언가를 제대로 된 일을 할 수 있다면 매개변수 제약은 적을수록 좋다.

하지만 구현하려는 개념 자체가 특정한 제약을 내재한 경우도 많긴 하다.
