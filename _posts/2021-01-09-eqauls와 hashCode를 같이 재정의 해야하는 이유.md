---
layout: post
title: eqauls와 hashCode를 같이 재정의 해야하는 이유
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---

이전 글에서 [equals를 재정의 하는 방법](https://donghyeon.dev/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C%EC%9E%90%EB%B0%94/2021/01/04/eqauls%EB%A5%BC-%EC%9E%AC%EC%A0%95%EC%9D%98-%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95/) 에 대해 다뤘다.

이번 장에서는 **equals와 hashCode를 같이 재정의 해야 하는 이유**에 대해서 설명 해 보려고 한다.

## Object 명세에서 발췌한 규약

- equals 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 hashCode 메서드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야 한다. 단, 애플리케이션을 다시 실행한다면 이 값이 달라져도 상관 없다.
- equals(Object)가 두 객체를 같다고 판단했다면(equals), **두 객체의 hashCode는 똑같은 값을 반환해야 한다.**
- equals(Object)가 두 객체를 다르게 판단했더라고, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다. 단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

**PhoneNumber 클래스**

```java
public class PhoneNumber {
    private int areaCode;
    private int prefix;
    private int lineNum;

    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = areaCode;
        this.prefix = prefix;
        this.lineNum = lineNum;
    }
}
```

다음과 같이 전화번호(02-444-7777)를 입력받는 PhoneNumber 클래스가 있다.

이 클래스를 활용하여 HashMap 안에 key를 PhoneNumber로 사용하고, value를 String으로 사용하는 Map 클래스를 만들어, put과 get을 해보자.

```java
Map<PhoneNumber,String> map = new HashMap<>();

map.put(new PhoneNumber(02,444,7777),"우리집");
String myHome = map.get(new PhoneNumber(02, 444, 7777));

System.out.println(myHome); // ?? 뭐가 출력 될까
```

이 코드의 실행 결과는 myHome의 null 값이 담긴다.

**왜 그럴까?**

위에서 생성한 PhoneNumber 클래스는 논리적으론 서로 동일하지만, **hashCode를 재정의하지 않았기 때문에** **논리적 동치인 두 객체가 서로 다른 해시코드를 반환하기 때문이다.**

HashMap은 내부적으로 <u>hashCode()</u>를 이용해서, 해시 버킷에서 꺼내오기 때문이다.

이 문제를 해결하기 위해선 PhoneNumber의 hashCode를 구현해줘야 한다.

**잘못된 hashCode 구현**

```java
@Override
public int hashCode() {
  return 50;
}
```

이렇게 구현하면 될까? 이렇게 구현하면 Object 명세에서 발췌한 규약을 모두 만족하기는 한다.

**그러나** 매번 같은 hashCode를 반환하기 때문에, 결국 한개의 버킷에만 쌓여, 링크드리스트하고 다를게 없게 된다. 

**<u>즉 map.get()의 걸리는 시간이 선형시간이 되버린다.</u>**

## hashCode를 구현하는 방법

1. int 변수 result를 선언한 후 값 c로 초기화 한다. c는 해당 객체의 **<u>첫번 째 핵심 필드를 단계</u>** 2.a 방식으로 계산한 해시코드다. 여기서 핵심 필드란 equals 비교에 사용되는 필드를 말한다.

2. 해당 객체의 나머지 핵심 필드 f 를 각각에 대해 다음 작업을 수행한다.

   a. 해당 필드의 해시코드 c를 계산 한다.

   - 기본 타입 필드라면 Type.hashCode(f)를 수행한다. 여기서 Type은 해당 기본 타입의 박싱 클래스다.

     ```java
     Double.doubleToLongBits(prefix); // Dobule 일 경우
     lineNum.hashCode(); // String 일경우
     ```

   - 참조 타입 필드면서 이 클래스의 equals 메서드가 이 필드의 equals를 재귀적으로 호출해 비교한다면, 이 필드의 hashCode를 재귀적으로 호출한다. 계산이 더 복잡해질 것 같으면, 이 필드의 표준형을 만들어 그 표준형의 hashCode를 호출한다. 필드의 값이 null이면 0을 사용한다. (다른 상수도 괜찮지만 전통적으로 0을 사용한다.)

   - 필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 다룬다. 이상의 규칙을 재귀적으로 적용해 각 핵심 원소의 해시코드를 계산한 다음, 단계 2.b 방식으로 갱신한다. 배열에 핵심 원소가 하나도 없다면, 단순히 상수(0을 추천)를 사용한다. 모든 원소가 핵심 원소라면 Arrays.hashCode를 사용한다.

   b. 단계 2.a에서 계산한 해시코드 c로 result를 갱신한다. `result = 31 * result + c; `

3. result를 반환한다.

**잘 구현한 hashCode**

```java
@Override
public int hashCode() {
    int result = areaCode;
    result = 31 * result + prefix;
    result = 31 * result + lineNum;
    return result;
}
```

파생 필드는 해시코드 계산에서 제외해도 된다. 즉, 다른 필드로부터 계산해 낼 수 있는 필드는 모두 무시해도 된다. **또한 equals 비교에 사용되지 않은 필드는 <u>반드시</u> 제외해야 한다.** 그렇지 않으면 hashCode 규약 두 번째를 어기게 될 위험이 있다.

**단계 2.b의 곱셈 31 * result는 필드를 곱하는 순서에 따라 result 값이 달라져서, 비슷한 필드가 여러 개 일 때 해시 효과를 높여 준다.** 예를 들면, String의 hashCode를 곱셈없이 구현한다면, 아나그램(구성하는 철자가 같고, 그 순서만 다른 문자열)의 경우 해시코드가 같아진다. 또한 31로 정한 이유는 홀수이면서 소수(prime)이기 때문이다. 만약 이 숫자가 짝수이고 오버플로우가 발생한다면 정보를 잃는다. 결과적으로 31을을 곱하면 시프트 연산과 뺄셈을 이용해서 최적화 할 수 있다. 

**또한 해시성능을 높이자고 해시코드를 계산할 때 핵심 필드를 생략하면 안된다.** 속도는 빨라지겠지만, 해시 품질이 나빠져 해시테이블의 성능을 심각하게 떨어뜨릴 수도 있다. 특히 어떤 필드는 특정 영역에 몰린 인스턴스들의 해시코드를 넓은 범위로 고르게 퍼트려주는 효과가 있을지도 모른다.

**마지막으로 hashCode가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 말자. 그래야 클라이언트가 이 값에 의지하지 않게 되고, 추후에 계산 방식을 바꿀 수도 있다.** String과 Integer를 포함해, 자바 라이브러리의 많은 클래스에서 hashCode 메서드가 반환하는 정확한 값을 알려준다. 이렇게 되면, 향후 릴리스에서 해시 기능을 개선항 여지를 없애버렸기 때문에 좋지 않은 방법이다. 자세한 규칙을 공표하지 않는다면, 해시 기능에서 결함을 발견했거나 더 나은 해시 방식을 알아낸 경우 다음 릴리스에서 수정할 수 있다.

## Objects.hash()

Objects 클래스는 임의의 개수만큼 객체를 받아 해시코드를 계산해주는 정적 메서드 hash를 제공하지만, 입력 인수를 담기 위한 배열이 만들어지고, 입력 중 기본 타입이 있다면 언박싱도 거쳐야 하기 때문에 **속도는 더 느리다**.

**Objects.hash()**

```java
@Override
public int hashCode() {
  return Objects.hash(lineNum, prefix, areaCode);
}
```



## Hash 캐싱

클래스가 불변이고 해시코드 계산하는 비용이 크다면, 매번 새로 계산하기 보다는 캐싱하는 방법이 있다.

```java
public int hashCode;

@Override
public int hashCode() {
  	int result = hashCode;
  	if (result == 0) {
      result = areaCode;
      result = 31 * result + prefix;
      result = 31 * result + lineNum;
      hashCode = result;
    }
  return result;
}
```

그렇지만 hashCode가 **스레드 안전성**을 갖도록 잘 구현해야 할 것이다.