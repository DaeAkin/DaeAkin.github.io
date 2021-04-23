---
layout: post
title: 이왕이면 제너릭 메서드로 만들자
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---



클래스와 마찬가지로, 메서드도 제너릭으로 만들 수 있다. `List<E>` 처럼 매개변수화 타입을 받는 정적 유틸리티 메서드는 보통 제너릭이다.

Collections의 binarySeach , sort 등 알고리즘 메서드는 모두 제너릭이다.

**Collections.sort()**

```java
public static <T extends Comparable<? super T>> void sort(List<T> list) {
  list.sort(null);
}
```



## 로우타입으로 만든 제너릭

다음은 두 집합의 합집합을 반환하는데, 문제가 있는 메서드다.

**raw 타입을 사용한 메서드**

```java
public static Set union(Set s1, Set s2) {
  Set result = new HashSet(s1);
  result.addAll(s2);
  return result;
}
```

이 메서드는 정상적으로 컴파일은 되지만 컴파일 경고가 두 개 발생한다.

- **unchecked call to HashSet(Collection<? extends E) as a member of raw type HashSet Set result = new HashSet(s1):**
- **unchecked call to addAll(Collection<? extends E) as a member of raw type Set result.addAll(s);**

첫번 째 경고는 타입 매개변수에 타입을 명시하지 않아 옳바른 타입인지 체크를 할 수 없어서 컴파일러가 경고를 한다.

두번 째 경고는 타입을 알수없는 raw타입인 Set에 원소를 추가하고 있어, 옳바른 타입인지 체크할 수 없어서 컴파일 경고가 난다.

경고를 모두 없애려면 타입을 안전하게 만들어야 한다. 

메서드 선언에서의 세 집합(s1, s1, result) 의 원소타입을 타입 매개변수로 명시하고, 메서드 안에서도 이 타입 매개변수만 사용하도록 수정하면 된다.

**타입 매개변수 목록은 메서드의 제한자와 반환 타입 사이에 온다.**

다음 코드에서 타입 매개변수 목록은 `<E> ` 이고 반환 타입은 `Set<E>` 이다. 

```java
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
  Set<E> result = new HashSet<>(s1);
  result.addAll(s2);
  return result;
}
```

단순히 제너릭 메서드라면 이 정도면 충분하다



# 제너릭 싱글턴 팩터리

때로는 불변 객체를 여러 타입으로 활용할 수 있게 만들어야 할 때가 있다. **제너릭은 런타임에 타입 정보가 소거가 되므로 하나의 객체를 어떤 타입으로든 매개변수화할 수 있다.**

제네릭 싱글턴 팩터리란 요청한 타입 매개변수에 맞게 그 객체의 타입을 바꿔주는 정적 팩터리 메서드 이다.

하지만 이렇게 하려면 요청한 타입 매개변수에 맞게 매번 그 객체의 타입을 바꿔주는 정적 팩터리를 만들어야 한다. 이 패턴을 제너릭 싱글턴 팩터리라 하며, `Collections.reverseOrder` 같은 함수 객체나 `Collections.emptySet` 같은 컬렉션용으로 사용 한다.

```java
@SuppressWarnings("unchecked")
public static <T> Comparator<T> reverseOrder() {
  return (Comparator<T>) ReverseComparator.REVERSE_ORDER;
}

private static class ReverseComparator
  implements Comparator<Comparable<Object>>, Serializable {

  static final ReverseComparator REVERSE_ORDER
    = new ReverseComparator();

  public int compare(Comparable<Object> c1, Comparable<Object> c2) {
    return c2.compareTo(c1);
  }

  @java.io.Serial
    private Object readResolve() { return Collections.reverseOrder(); }

  @Override
  public Comparator<Comparable<Object>> reversed() {
    return Comparator.naturalOrder();
  }
}
```

`(Comparator<T>) ReverseComparator.REVERSE_ORDER;` 처럼 형변환을 하면 비검사 형변환 경고가 발생한다. 

하지만 코드를보면 어떤 수정도 없이 형변환만 하기 때문에, 타입에 안전하다고 할 수 있다. 그래서 `@SuppressWarnings` 어노테이션을 추가해서 경고를 제거해줘도 된다.

```java
public class Main {

    public static void main(String[] args) {
        Set<String> stringSet = Collections.emptySet();
        Set<Integer> integerSet = Collections.emptySet();
    }
}
```



## 재귀적 타입 한정(recursive type bound)

드물긴 하지만, **자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용 범위를 한정할 수 있다.** 이런 개념을 재귀적 타입 한정이라고 한다.

재귀적 타입 한정은 주로 타입의 자연적 순서를 정하는 Comparable 인터페이스와 함께 쓰인다.

Comparable을 구현한 원소의 컬렉션을 입력받는 메서드들은 주로 그 원소들을 정렬 혹은 검색하거나, 최솟값이나 최대값을 구하는 식으로 사용된다. 이 기능을 수행하려면 컬렉션에 담긴 모든 원소가 상호 비교될 수 있어야 한다.

**재귀적 타입 한정을 이용해 상호 비교할 수 있음을 표현한 코드**

```java
public static <E extends Comparable<E>> E max(Collection<E> c);
```

**이 코드를 해석하자면, E는 Comparable을 구현한 타입이여야 한다는 타입한정을 정의한 것이다.**  E타입이 Comparable을 구현하지 않았다면, 컴파일 에러가 난다.

# 정리

제너릭 타입과 마찬가지로, 클라이언트에서 입력 매개변수와 반환값을 명시적으로 형변환해야 하는 메서드보다 제너릭 메서드가 더 안전하며 사용하기도 쉽다.

**타입과 마찬가지로, 메서드도 형변환 없이 사용할 수 있는 편이 좋으며, 많은 경우 그렇게 하려면 제너릭 메서드가 되어야 한다. 역시 타입과 마찬가지로, 형변환을 해줘야 하는 기존 메서드는 제너릭하게 만들자.** 

기존 클라이언트는 그대로 둔 채 새로운 사용자의 삶을 훨씬 편하게 만들어줄 것이다.