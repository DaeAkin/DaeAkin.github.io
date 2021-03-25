---
layout: post
title: raw 타입은 사용하지 말자
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---



클래스와 인터페이스 선언에 타입 매개변수가 쓰이면 이를 **제너릭 클래스** 혹은 **제너릭 인터페이스**라고 한다. List 인터페이스로 예를 들자면, List 인터페이스는 원소의 타입을 나타내는 타입 매개변수 E를 받는다. 

`List<String>` 에서는 String이 정규 타입 매개변수 E에 해당하는 실제 타입 매개 변수이다.

## 단어 정리

| 한글용어                 | 영문용어                | 예                          |
| ------------------------ | ----------------------- | --------------------------- |
| 매개변수화 타입          | parameterized type      | List<String\>               |
| 실제 타입 매개변수       | actual type parameter   | String                      |
| 제너릭 타입              | generic type            | List<E\>                    |
| 정규 타입 매개변수       | formal type parameter   | E                           |
| 비한정적 와일드카드 타입 | unbounded wildcard type | List<?\>                    |
| 로 타입                  | raw type                | List                        |
| 한정적 타입 매개 변수    | bounded type parameter  | <E extends Number\>         |
| 재귀적 타입 한정         | recursive type bound    | <T extends Comparable<T\>\> |
| 한정적 와일드카드 타입   | bounded wildcard type   | List<? extends Number\>     |



## Raw 타입

raw 타입이란, **제너릭 타입에서 타입 매개 변수를 전혀 사용하지 않을 때**를 말한다. 예를 들어 List를 선언할 때 `List` 만 선언해 놓은 경우를 말한다. 

**컬렉션의 Raw 타입**

```java
private final Collection stamps = ...;
```

이 코드를 사용하면 실수로 도장(Stamp) 대신 동전(Coin)을 넣어도 아무 오류 없이 컴파일 되고 실행된다.

```java
//실수로 동전을 넣는다.
stamps.add(new Coin(...)); // "unchecked call" 경고를 내뱉는다.
```

**매개변수화된 컬렉션 타입 - 타입 안전성 확보!**

```java
private final Collection<Stamp> stamps = ...;
```

이렇게 선언하면 컴파일러는 stamps에는 Stamp의 인스턴스만 넣어야 함을 컴파일러가 인지하게 된다. 따라서 아무런 경고 없이 컴파일된다면 의도대로 동작할 것임을 보장해준다.



## 제너릭 소거(Generic erasure)

Raw 타입을 쓰는 걸 언어 차원에서 막아 놓지는 않았지만 절대로 써서는 안 된다. **Raw 타입을 쓰면 제너릭이 안겨주는 안전성과 표현력을 모두 잃게 된다.**

그렇다면 Raw 타입을 왜 없애지 않았을까? 바로 호환성 때문이다. 자바가 제너릭을 받아들이기까지 거의 10년이 걸린 탓에 제너릭 없이 짠 코드가 이미 많아졌다. 그래서 기존 코드를 모두 수용하면서 제너릭을 사용하는 새로운 코드와도 맞물려 돌아가게 해야만 했다.

Raw 타입을 사용하는 메서드에 매개변수화 타입의 인스턴스를 넘겨도 동작해야만 해야했다. 이 마이그레이션 호환성을 위해 Raw 타입을 지원하고 제너릭 구현에는 소거 방식을 사용하기로 했다.



### List vs List<Object\>의 차이

List 같은 Raw 타입은 사용해서는 안되지만, List<Object\> 처럼 임의 객체를 허용하는 매개변수화 타입은 괜찮다. 

이 둘의 차이점은 List는 **제너릭 타입에서 완전히 발을 뺀 것이고** List<Object\>는 모든 타입을 허용한다는 의사를 컴파일러에게 명확히 전달한 것이다.

List 매개변수에는 List<String\>을 전달할 수는 있지만, List<Object\> 매개변수에는 List<String\>을 전달할 수 없다. **즉 타입 안정성을 잃게 된다.**



### 제너릭 소거 더 알아보기

다음과 같은 메서드가 있다.

```java
public static <E> void printArray(E[] array) {
    for (E element : array) {
        System.out.printf("%s ", element);
    }
}
```

이 메서드를 컴파일 하면 다음과 같이 된다.

```java
public static void printArray(Object[] array) {
    for (Object element : array) {
        System.out.printf("%s ", element);
    }
}
```

컴파일 시 E타입이 Object 타입으로 변경이 된다.

이렇게 E 타입만 있는 상황을 unbound 되었다라고 한다.

이번엔 이렇게 해보자.

```java
public static <E extends Comparable<E>> void printArray(E[] array) {
    for (E element : array) {
        System.out.printf("%s ", element);
    }
}
```

이 메서드를 컴파일하면 다음과 같이 된다.

```java
public static void printArray(Comparable[] array) {
    for (Comparable element : array) {
        System.out.printf("%s ", element);
    }
}
```

아까랑은 다르게 E가 부모타입인 Comparable로 되었다.

이렇게 부모 타입으로 변경되는 상황을 bound 되었다라고 한다.



## Raw 타입을 써야 할 때

제너릭 타입을 쓰고 싶지만 실제 타입 매개변수가 무엇인지 신경 쓰고 싶지 않다면 물음표(?)를 사용하자. 제너릭 타입인 Set<E\>의 비한정적 와일드 카드(Unbound wildcard type)은 Set<?\> 이다. 이것이 어떤 타입이라도 담을 수 있는 가장 범용적인 매개변수화 Set이다.

```java
static int numElementsInCommon(Set<?> s1, Set<?> s2) {...}
```

### Set vs Set<?\>

이 둘의 특징을간단히 말하자면 와일드카드 타입은 안전하고, Raw타입은 안전하지 않다. Raw 타입 컬렉션은 아무 원소나 넣을 수 있으니 타입 불변식을 훼손하기 쉽다. **반면 Collection<?\> 에는 어떤 원소도 넣을 수 없다.** 다른 원소를 넣으려고 하면 오류메세지가 나온다.

즉 컬렉션의 타입 불변식을 훼손하지 못하게 막았다. 구체적으로는 어떤 원소도 Collection<?\>에 넣지 못하게 했으며 컬렉션에서 꺼낼 수 있는 객체의 타입도 전혀 알 수 없게 했다. 이러한 제약을 받아들일 수 없다면 제너릭 메서드나 한정적 와일드카드 타입을 사용하면 된다.

---

로 타입을 쓰지 말라는 규칙에도 예외가 몇 개 있다.

**class 리터럴에는 Raw 타입을 써야 한다.** 자바 명세는 class 리터럴에 매개변수화 타입을 사용하지 못하게 했다.예를 들어 List.class, String[].class, int.class는 허용하고 LIst<String\>.class와 List<?\>.class는 허용하지 않는다.

또 다른 예로 instanceof 연산자와 관련이 있다. 런타임에는 제너릭 타입 정보가 지워지므로 instanceof 연산자는 비한정적 와일드카드 타입 이외의 매개변수화 타입에는 적용할 수 없다. 그리고 Raw 타입이든 비한정적 와일드카드 타입이든 instanceof는 완전히 똑같이 동작한다. 비한정적 와일드카드 타입의 꺽쇠괄호와 물음표는 아무런 역할 없이 코드만 지저분하게 만드므로, 차라리 raw 타입을 쓰는 편이 깔끔하다.

```java
if(o instanceof Set) {
  Set<?> s = (Set<?)o;
}
```



## 정리

Raw 타입을 사용하면 런타임에 예외가 일어날 수 있으니 사용하면 안 된다. Raw 타입은 제너릭이 도입되기 이전 코드와의 호환성을 위해 제공될 뿐이다. 