---
layout: post
title: 정확한 답이 필요하다면 float와 double은 피하자
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---



# float와 double의 문제점

float와 double 타입은 과학과 공학 계산용으로 설계되었다.

이진 부동소수점 연산에 쓰이며, 넓은 범위의 수를 빠르게 정밀한 **<u>근사치</u>** 로 계산하도록 세심하게 설계되었다.

따라서 정확한 결과가 필요할 때는 사용하면 안 된다.

**float와 double 타입은 특히 금융 관련 계산과는 맞지 않는다.**

0.1 혹은 10의 음의 거듭제곱 수를 표현할 수 없기 때문이다.

```java
System.out.println(1.03 - 0.42);
```

이 코드의 결과는 **0.6100000000001** 을 출력한다.

실제로 10진수의 **0.1**을 2진수로 바꾸면 **0.00011001100....** 처럼 순환소수가 된다.

이 때문에 **0.1을 100번 더한다고 해도 10이 되지 않는다.**  



# BigDecimal

이 문제를 올바로 해결하려면 BigDecimal, int 혹은 long을 사용해야 한다.

```java
BigDecimal a = new BigDecimal("1.03");
BigDecimal b = new BigDecimal("0.42");
System.out.println(a.subtract(b));
```

BigDecimal의 생성자 중 문자열을 받는 생성자를 사용했다. 

계산시 부정확한 값이 사용되는 걸 막기 위해 필요한 조치다.

## BigDecimal의 단점

기본 타입보다 쓰기가 불편하고 느리다. 단발성 계산이라면 느리다는 문제는 무시할 수 있지만, 쓰기 불편하다는 점은 아쉽다.

BigDecimal의 대안으로 int 혹은 long 타입을 쓸 수도 있지만, 그럴 경우 다룰수 있는 값의 크기가 제한되고, 소수점을 직접 관리해야 한다.



## BigDeciamal의 원리

`BigDecimal("10.25")` 가 있다면 먼저 10의 제곱을 곱해서, 정수로 만든다.



# 정리

정확한 답이 필요한 계산에는 float나 double을 피하자

코딩 시 불편함이나 성능저하를 신경쓰지 않겠다면 BigDecimal을 사용하자

반면 성능이 중요하고 소수점을 직접 추적할 수 있고 숫자가 너무 크지 않다면 int나 long을 사용하자.

숫자를 아홉자리 십진수로 표현햘 수 있다면 int로, 열여덟 자리 십진수로 표현할 수 있다면 long을 사용하자.

그 이상은 BigDecimal을 사용해야 한다. 
