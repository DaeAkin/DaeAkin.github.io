---
layout: post
title: 옵셔널 반환은 신중히 하자
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---



자바 8전에는 메서드가 특정 조건에서 값을 반환할 수 없을 때는 선택지가 두 가지 있었다.

- 예외를 던진다
- null을 반환한다.

두 방법 모두 허점이 있다.



# 예외와 null

 예외는 진짜 예외적인 상황에서만 사용해야 하며 예외를 생성할 때 스택 추적 전체를 캡처하므로 비용이 크다.

null을 반환하면 이런 문제가 생기진 않지만, 별도의 null 처리 코드를 추가해야 한다.

null 처리를 무시하면 언젠가는 NullPointerException이 일어날 수 있다. 그것도 근본적인 원인에서 멀리 떨어진 곳에서 말이다.





## Optional<T\>

자바 8부터 또 하나의 선택지가 생겼다. 

Optional<T\> 은 null이 아닌 T타입 참조를 하나 담거나, 혹은 아무것도 담지 않을 수 있다.

아무것도 담지 않은 옵셔널은 비었다라고 말하고, 반대로 어떤 값을 담은 옵셔널은 비지 않았다라고 한다.

옵셔널은 원소를 최대 1개 가질 수 있는 불변 컬렉션이다. Optional이 Collection을 구현하지는 않았지만, 원칙적으로는 그렇게 부른다.

보통은 T를 반환해야 하지만, 특정 조건에서는 아무것도 반환하지 않아야 할 때 T 대신 Optional<T\>을 반환하도록 선언하면 된다.

그러면 유효한 반환값이 없을 때는 빈 결과를 반환하는 메서드가 만들어진다.

옵셔널을 반환하는 메서드는 예외를 던지는 메서드보다 유연하고 사용하기 쉬우며, null을 반환하는 메서드보다 오류 가능성이 작다.

**컬렉션에서 최대값을 구하는데, 컬렉션이 비어 있으면 예외를 던짐**

```java
public static <E extends Comparable<E>> E max(Collection<E> c) {
  if (c.isEmpty())
    throw new IllegalArgumentException("빈 컬렉션");
  
  E result = null;
  for (E e : c)
    if (result == null || e.compareTo(result) > 0)
      result = Objects.requireNonNull(e);
  return result;
}
```



**컬렉션에서 최대값을 구해 Optional로 반환한다.**

```java
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
  if (c.isEmpty())
    throw Optional.empty();
  
  E result = null;
  for (E e : c)
    if (result == null || e.compareTo(result) > 0)
      result = Objects.requireNonNull(e);
  return Optional.of(result);
}
```

옵셔널을 만들기 위해 두 가지 정적 팩터리 메서드를 사용 했다.

- Optional.empty() : 빈 옵서녈 생성
- Optional.of(value) : 값이 든 옵셔널 생성
  - value에 null을 넣으면 NullPointerException이 발생한다.
  - Optional.ofNullable(value)를 사용하면 null도 반환 가능하지만, 추천하진 않다.



# 옵셔널 반환의 기준

옵셔널은 검사 예외와 취지가 비슷하다.

즉 반환값이 없을 수도 있음을 API 사용자에게 명확히 알려준다.

비검사 예외를 던지거나 null을 반환한다면 API 사용자가 그 사실을 인지하지 못해 끔찍한 결과로 이어질 수 있다.

하지만 검사 예외를 던지면 클라이언트에서는 반드시 이에 대한 대처하는 코드를 작성해 넣어야 한다.

비슷하게, 메서드가 옵셔널을 반환한다면 클라이언트는 값을 받지 못했을 때 취할 행동을 선택해야 하는데,

그중 하나는 기본값을 설정하는 방법이다.

```java
String lastWordInLexicon = max(words).orElse("단어 없음..")
```

또는 상황에 맞는 예외를 던질 수 있다.

다음 코드는 실제 예외가 아니라 예외 팩터리를 건넨 것에 주목하자.

이렇게 하면 예외가 실제로 발생하지 않는 한 예외 생성 비용은 들지 않는다.

```java
Toy myToy = max(toys).orElseThrow(TemperTantrumException::new);
```

옵셔널이 항상 값이 채워져 있다고 확신하면 그냥 바로 값을 꺼낼 수도 있다.

하지만 값이 없다면 NoSuchElementException이 발생 한다.

```java
Element lastNobleGas = max(Elements.NOBLE_GASES).get();
```



### 캐싱

기본값을 설정하는 비용은 아주 커서 부담이 될 때가 있다.

그럴 때는 Supplier<T\> 를 인수로 받는 orElseGet을 사용하면, 값이 처음 필요할 때 Supplier<T\> 를 사용해 생성하므로 초기 설정 비용을 낮출 수 있다.

```java
public class Main {
    //캐싱
    private static final Supplier<Cheese> defaultCheese = Cheese::new;

    public static void main(String[] args) {
        getCheese().orElseGet(defaultCheese);
    }

    public static Optional<Cheese> getCheese() {
        return Optional.empty();
    }

}

class Cheese {

}
```



# Optional.stream()

자바 9에서는 Optional에 stream() 메서드가 추가 되었다.

이 메서드는 Optional을 Stream으로 변환해주는 어댑터다. 

옵셔널에 값이 있으면 그 값을 원소로 담은 스트림으로, 값이 없다면 빈 스트림으로 변환한다.

이를 Stream의 flatMap 메서드와 조합하면 앞의 코드를 다음처럼 명료하게 바꿀 수 있다.

```java
streamOfOptionals
  .flatMap(Optional::Stream)
```





# 옵셔널을 사용해야 할 곳

반환값을 옵셔널을 사용한다고 해서 무조건 득이 되는 건 아니다.

**컬렉션, 스트림, 배열, 옵셔널 같은 컨테이너 타입은 옵셔널로 감싸면 안 된다.**

빈 Optional<List<T\>> 를 반환하기보다는 빈 List<T\> 를 반환하는게 좋다.

빈 컨테이너를 그대로 반환하면 클라이언트에서 옵셔널 처리 코드를 넣지 않아도 된다.

참고로 ProcessHandle.Info 인터페이스의 arguments 메서드는 Optional<String[]\> 를 반환 하는게 예외적인 경우니 따라 하지 말자.

그렇다면 어떤 경우에 메서드 반환 타입을 T 대신 Optional<T\> 로 선언해야 할까?

- 결과가 없을 수 있으며
- 클라이언트가 이 상황을 특별하게 처리해야 할 때

이 경우 Optional을 반환한다.

그런데 이렇게 하더라도 Optional을 반환하는 데는 대가가 따른다.

Optional도 새로 할당하고 초기화해야 하는 객체이고, 그 안에서 값을 꺼내려면 메서드를 호출해야 하니 한 단계를 더 거치는 셈이다.

그래서 성능이 중요한 상황에서는 옵셔널이 맞지 않을 수 있다



# 원시타입 Optional

박싱된 기본 타입을 담는 옵셔널은 기본 타입 자체보다는 값을 두 겹이나 감싸기 때문에 무거울 수 밖에 없다.

그래서 int, long, double 전용 옵셔널 클래스들인 OptionalInt, OptionalLong, OptionalDouble이 있다.

이 옵셔널들도 Optional<T\> 가 제공하는 메서드를 거의 다 제공한다.

이렇게 대체제까지 있으니 박싱된 기본 타입을 담은 옵셔널을 반환하는 일도 없도록 하자.

단 '덜 중요한 기본 타입'용인 Boolean, Byte, Character, Short, Float은 예외일 수도 있다.



# Map의 키로 옵셔널?

지금까지 옵셔널을 반환하고 반환된 옵셔널을 처리하는 이야기를 했고, 다른 쓰임에 관해서는 논하지 않았다.

왜냐하면 대부분 적절치 않기 때문이다.

예를 들어 옵셔널을 맵의 값으로 사용하면 절대 안 된다.

만약 그렇게 하면 맵 안의 키가 없다는 사실을 나타내는 방법이 두 가지가 된다.

- 키 자체가 없는 경우
- 키 는 있는데 그 키가 속이 빈 옵셔널 일 경우

쓸데 없이 복잡성만 높이게 된다.



# 인스턴스 변수로 옵셔널?

옵셔널을 인스턴스 필드에 저장해두는게 필요할 때가 있을까?

이런 상황 대부분은 필수 필드를 갖는 클래스와, 이를 확장해 선택적 필드를 추가한 하위 클래스를 따로 만들어야 함을 암시하는 나쁜 냄새다.

이전에 살펴봤던 영양소 관련 클래스인 NutritionFacts 클래스의 필드 대부분은 필수 값이 아니다.

또한 그 필드들은 기본 타입이라 값이 없음을 나타낼 방법이 마땅치 않다.

이런 경우라면 선택적 필드의 게터 메서드들이 옵셔널을 반환하게 해주는 것도 좋은 방법이기도 하다.
