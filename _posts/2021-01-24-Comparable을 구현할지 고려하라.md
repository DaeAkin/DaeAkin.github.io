---
layout: post
title: Comparable을 구현할지 고려하라
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---

이번에는 Comparable 인터페이스의 유일한 메서드인 compareTo를 알아보자. 

Object의 equals와 comareTo의 차이점은, 단순 동치성 비교를 더해 순서까지 비교할 수 있으며 제너릭하다는 장점이 있다. Comparable을 구현했다는 것은 그 클래스의 인스턴스들에는 자연적인 순서가 있음을 뜻한다. 그래서 Comparable을 구현한 객체들의 배열은 다음처럼 손쉽게 정렬할 수 있다.

`Arrays.sort(a);`

검색, 극단값 계산, 자동 정렬되는 컬렉션 관리도 역시 쉽게 할 수 있다. 예컨대 다음 프로그램은 명령줄 인수들을 (중복을 제거하고) 알파벳순으로 출력한다. String이 Comparable을 구현한 덕분이다. 

```java
public class WordList {
  public static void main(String[] args) {
    Set<String> s = new TreeSet<>();
    Collections.addAll(s,args);
    System.out.println(s):
  }
}
```

사실상 자바 플랫폼 라이브러리의 모든 값 클래스와 열거타입이 Comparable을 구현했다. 알파벳, 숫자, 연대 같이 순서가 명확한 값 클래스를 작성한다면 반드시 Comparable 인터페이스를 구현하자.

```java
public interface Comparable<T> {
  int compareTo(T t);
}
```



compareTo 메서드의 일반 규약은 [equals의 규약](https://donghyeon.dev/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C%EC%9E%90%EB%B0%94/2021/01/04/eqauls%EB%A5%BC-%EC%9E%AC%EC%A0%95%EC%9D%98-%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95/)과 비슷하다. 

## compareTo 메서드의 일반 규약

이 객체와 주어진 객체의 순서를 비교한다. 이 객체가 **주어진 객체보다 작으면 음의 정수를, 같으면 0을, 크면 양의 정수를 반환한다.** 이 객체와 비교할 수 없는 타입의 객체가 주어지면 ClassCastException을 던진다.

다음 설명에서 sgn(표현식) 표기는 수학에서 말하는 부호 함수(signum function)를 뜻하며, 표현식의 값이 음수, 0, 양수 일 때 -1,0,1을 반환하도록 정의했다.

- Comparable을 구현한 클래스는 모든 x,y에 대해 sgn(x.compareTo(y)) == -sgn(y.compareTo(x))여야 한다. 
  - 따라서  x.compareTo(y)는 y.compareTo(x)가 예외를 던질 때에 한해 예외를 던져야 한다.
- Comparable을 구현한 클래스는 추이성을 보장해야 한다. 즉, x.compareTo(y) > 0 && y.compareTo(z) 이면 x.compareTo(z) 여야 한다.
- x.compareTo(y) == 0 이면 sgn(x.compareTo(z)) == sng(y.compareTo(z)) 이다.
- 이번 권고가 필수는 아니지만 꼭 지키는게 좋다. (x.compareTo(y) == 0) == (x.equals(y)) 여야 한다. Comparable을 구현하고 이 권고를 지키지 않는 모든 클래스는 그 사실을 명시해야 한다. 다음과 같이 명시하면 적당할 것이다. 
  "주의 : 이 클래스의 순서는 equals 메서드와 일관되지 않다."

모든 객체에 대해 전역 동치관계를 부여하는 equals 메서드와 달리, compareTo는 타입이 다른 객체를 신경 쓰지 않아도 된다. 타입이 다른 객체가 주어지면 간단히 ClassCastException을 던져도 되며, 대부분 그렇게 한다. 물론, 이 규약에서는 다른 타입 사이의 비교도 허용하는데, 보통은 비교할 객체들이 구현할 공통 인터페이스를 매개로 이뤄진다.

hashCode 규약을 지키지 못하면 해시를 사용하는 클래스와 어울리지 못하듯, compareTo 규약을 지키지 못하면 비교를 활용하는 클래스와 어울리지 못한다. 비교를 활용하는 클래스의 예로는 정렬된 컬렉션인 TreeSet과 TreeMap, 검색과 정렬 알고리즘을 활용하는 유틸리티 클래스인 Collections와 Arrays가 있다.

**<u>위의 세 규약은</u>** compareTo 메서드로 수행하는 동치성 검사도 [equals](https://donghyeon.dev/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C%EC%9E%90%EB%B0%94/2021/01/04/eqauls%EB%A5%BC-%EC%9E%AC%EC%A0%95%EC%9D%98-%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95/) 규약과 똑같이 반사성, 대칭성, 추이성을 충족해야 함을 뜻한다. 그래서 주의사항도 똑같다. 기존 클래스를 확장한 구체 클래스에서 새로운 값 컴포넌트를 추가했다면 compareTo 규약을 지킬 방법이 없다. 객체 지향적 추상화의 이점을 포기할 생각이 아니라면 말이다. 우회법도 같다. Comparable을 구현한 클래스를 확장해 값 컴포넌트를 추가하고 싶다면, 확장하는 대신 독립된 클래스를 만들고 이 클래스에 원래 클래스의 인스턴스를 가리키는 필드로 두자 (equals 포스팅의 ColorPoint처럼 말이다.)

**<u>compareTo의 마지막 규약은 필수는 아니지만 꼭 지키길 권한다.</u>** 마지막 규약은 간단히 말하면 compareTo 메서드로 수행한 동치성 테스트의 결과가 equals와 같아야 한다는 것이다. 이를 잘 지키면 compareTo로 줄지은 순서와 equals의 결과가 일관되게 된다. compareTo의 순서와 equals의 결과가 일관되지 않은 클래스도 여전히 동작은 한다. **단 이 클래스의 객체를 정렬된 컬렉션에 넣으면 해당 컬렉션이 구현한 인터페이스(Collection, Set 혹은 Map) 에 정의된 동작과 엇박자를 낼 것이다.** 이 인터페이스들은 equals 메서드의 규약을 따른다고 되어 있지만, 놀랍게도 정렬된 컬렉션들은 동치성을 비교할 때 equals대신 comapreTo를 사용하기 때문이다. 아주 큰 문제는 아니지만, 주의해야 한다.

**예제**

```java
BigDecimal bigDecimal1 = new BigDecimal("1.0");
BigDecimal bigDecimal2 = new BigDecimal("1.00");

Set<BigDecimal> hashSet = new HashSet<>(); // eqauls로 구분
hashSet.add(bigDecimal1);
hashSet.add(bigDecimal2);

Set<BigDecimal> treeSet = new TreeSet<>(hashSet); // compareTo로 구분

System.out.println(bigDecimal1.equals(bigDecimal2)); // false
System.out.println(bigDecimal1.compareTo(bigDecimal2)); // 0
System.out.println(hashSet.size()); // 2
System.out.println(treeSet.size());  // 1
```



## CompareTo 구현

compareTo 메서드는 각 필드가 동치인지를 비교하는게 아니라 그 순서를 비교한다. 객체 참조 필드를 비교하려면 compareTo 메서드를 재귀적으로 호출한다. Comparable을 구현하지 않은 필드나 표쥰이 아닌 순서로 비교해야 한다면 비교자(Comparator)를 대신 사용한다. 비교자는 직접 만들거나 자바가 제공하는 것 중에 골라 쓰면 된다. 

우리가 [eqauls](https://donghyeon.dev/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C%EC%9E%90%EB%B0%94/2021/01/04/eqauls%EB%A5%BC-%EC%9E%AC%EC%A0%95%EC%9D%98-%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95/)에서 다뤘던 예제 인 **CaseInsensitiveString** 을 예로 들어 보겠다.

```java
public class CaseInsensitiveString implements Comparable<CaseInsensitiveString>{
    private final String s;

    @Override
    public int compareTo(CaseInsensitiveString cis) {
        return String.CASE_INSENSITIVE_ORDER.compare(s,cis.s);
    }
}
```

CaseInsensitiveString은 자바가 제공하는 비교자를 사용하고 있다.



#### 기본 타입 필드가 여러개 일 때

기본 타입 필드가 여럿일 때는 어떻게 해야 할까? 다음과 같이 3개의 필드를 갖고 있는 PhoneNumber 클래스는 다음과 같이 구현 한다.

**PhoneNumber**

```java
package test;

public class PhoneNumber implements Comparable<PhoneNumber>{
    private short areaCode;
    private short prefix;
    private short lineNum;

    @Override
    public int compareTo(PhoneNumber pn) {
        int result = Short.compare(areaCode,pn.areaCode); // 가장 중요한 필드
        if(result == 0) {
            result = Short.compare(prefix,pn.prefix); //두 번째로 중요한 필드
            if(result == 0) 
                result = Short.compare(lineNum,pn.lineNum); // 세 번째로 중요한 필드
        }
        return result;
    }
}
```

그러나 자바 8부터 메서드 연쇄 방식으로 비교자를 생성할 수 있게 되어서 더욱 간결하게 코드를 짤 수 있다.

```java
private static final Comparator<PhoneNumber> COMPARATOR =
  Comparator.comparingInt((PhoneNumber pn) -> pn.areaCode)
  .thenComparingInt(pn -> pn.prefix)
  .thenComparingInt(pn -> pn.lineNum);

@Override
public int compareTo(PhoneNumber pn) {
  return COMPARATOR.compare(this,pn);
}
```

그러나 이 방식은 간결하지만, 10%정도의 성능이 저하가 존재한다.

## 주의 할점

예전에는 compareTo 메서드에서 정수 기본 타입 필드를 비교 할 때는 관계 연산자인 < 와 >를, 실수 기본 타입 필드를 비교할 때는 정적 메서드인 Double.comprae와 Float.compare를 사용하라고 권했다. 그런대 자바 7 부터는 상황이 변했다. 박싱된 기본 타입 클래스들에 새로 추가된 정적 메서드인 compare를 이용하면 된다. **compareTo 메서드에서 관계 연산자 <와 >을 사용하는 이전 방식은 거추장스럽고 오류를 유발하니, 이제는 추천하지 않는다.**

## 정리

순서를 고려해야 하는 값 클래스를 작성한다면 꼭 Comparable 인터페이스를 구현하여, 그 인스턴스들을 쉽게 정렬하고, 검색하고, 비교 기능을 제공하는 컬렉션과 어우러지도록 해야 한다. compareTo 메서드에서 필드의 값을 비교할 때 < 와 > 연산자는 쓰지 말아야 한다. 그 대신 박싱된 기본 타입 클래스가 제공하는 정적 compare 메서드나 Comparetor 인터페이스가 제공하는 비교자 생성 메서드를 사용하자.