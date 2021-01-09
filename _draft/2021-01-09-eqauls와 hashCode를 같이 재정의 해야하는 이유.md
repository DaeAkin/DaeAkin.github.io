---
layout: post
title: eqauls와 hashCode를 같이 재정의 해야하는 이유
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---

이전 글에서 [equals를 재정의 하는 방법](https://donghyeon.dev/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C%EC%9E%90%EB%B0%94/2021/01/04/eqauls%EB%A5%BC-%EC%9E%AC%EC%A0%95%EC%9D%98-%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95/) 에 대해 다뤘다.

이번 장에서는 equals와 hashCode를 같이 재정의 해야 하는 이유에 대해서 설명 해 보려고 한다.

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

**왜 그럴 까?**

위에서 생성한 PhoneNumber 클래스는 논리적으론 서로 동일하지만, hashCode를 재정의하지 않았기 때문에 **논리적 동치인 두 객체가 서로 다른 해시코드를 반환하기 때문이다.**

HashMap은 내부적으로 hashCode를 이용해서, 해시 버킷에서 꺼내오기 때문이다.

이 문제를 해결하기 위해선 PhoneNumber의 hashCode를 구현해줘야 한다.



```

```

