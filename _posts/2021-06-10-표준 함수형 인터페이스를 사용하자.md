---
layout: post
title: 표준 함수형 인터페이스를 사용하자
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---

자바가 람다를 지원하면서 API를 작성하는 모범 사례도 크게 바뀌었다.

상위 클래스의 기본 메서드를 재정의해 원하는 동작을 구현하는 템플릿 메서드 패턴의 매력이 크게 줄었다.

이를 대체하는 현대적인 해법은 같은 효과의 **<u>함수 객체</u>**를 받는 정적 팩터리나 생성자를 제공하는 것이다.

# LinkedHashMap

LinkedHashMap의 protected 메서드인 `removeEldestEntry` 를 재정의하면 캐시로 사용할 수 있다.

맵에 새로운 키를 추가하는 put 메서드는 이 메서드를 호출하여 true가 반환되면 맵에서 가장 오래된 원소를 제거한다.

```java
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
  return size() > 100;
}
```

위와 같이 removeEldestEntry를 재정의하게 되면, map에 새로운 키를 추가하는 **put 메서드가 호출 될 때마**다, 위에 코드가 실행되어, 원소가 100개가 될 때 까지 커지다가, 그 이상 되면 가장 오래된 원소를 제거한다.



# removeEldestEntry를 람다로 변경 해 보기

removeEldestEntry() 메서드를 구현할 때 필요한 값은 

- 현재 map에 있는 원소의 개수

이다. 

위에 코드에서는 map의 **<u>인스턴스 메서드인</u>** `size()` 를 이용했다.

**하지만 람다를 이용하면, Context가 달라지기 때문에 인스턴스 메서드를 사용하지 못하기 때문에 size를 알아낼 때 어려움이 있다.**

그래서 size를 알아내기 위해서, map 객체 자신도 같이 넘겨줘야 한다.

```java
@FunctionalInterface
interface EldestEntryRemovalFunction<K, V> {
    boolean remove(Map<K, V> map, Map.Entry<K, V> eldest);
}
```

**<u>하지만 이렇게 직접 만드는 것보단 표준 함수형 인터페이스를 활용하는 것이 좋다.</u>**

위에 코드는 2개의 인자를 받아서 boolean을 리턴하는 로직이므로 EldestEntryRemovalFunction대신 BiPredicate를 사용할 수 있다.

```java
@FunctionalInterface
public interface BiPredicate<T, U> {
    boolean test(T t, U u);
}
```

**기존 EldestEntryRemovalFunction을 사용하는 방법**

```java
public class Main {
    public static void main(String[] args) {
        Map<String, Integer> map = CacheMap.of((map1, eldest) -> map1.size() > 2);

        map.put("1", 1);
        map.put("2", 2);
        map.put("3", 3);

        System.out.println(map); // 2 , 3 출력
    }
}

@FunctionalInterface
interface EldestEntryRemovalFunction<K, V> {
    boolean remove(Map<K, V> map, Map.Entry<K, V> eldest);
}

class CacheMap<K, V> extends LinkedHashMap<K, V> {

    private EldestEntryRemovalFunction<K,V> function;

    private CacheMap(EldestEntryRemovalFunction<K,V> function) {
        this.function = function;
    }

    public static <K,V> CacheMap<K,V> of(EldestEntryRemovalFunction<K,V> function) {
        return new CacheMap<K,V>(function);
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return function.remove(this,eldest);
    }
}
```

**BiPredicate을 사용하는 방법**

```java
public class Main {
    public static void main(String[] args) {
        Map<String, Integer> map = CacheMap.of((map1, eldest) -> map1.size() > 2);

        map.put("1", 1);
        map.put("2", 2);
        map.put("3", 3);

        System.out.println(map); // 2 , 3 출력
    }
}

class CacheMap<K, V> extends LinkedHashMap<K, V> {

    private BiPredicate<Map<K,V>,Map.Entry<K,V>> function;

    private CacheMap(BiPredicate<Map<K,V>,Map.Entry<K,V>> function) {
        this.function = function;
    }

    public static <K,V> CacheMap<K,V> of(BiPredicate<Map<K,V>,Map.Entry<K,V>> function) {
        return new CacheMap<>(function);
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return function.test(this,eldest);
    }
}
```



# 표준 함수형 인터페이스의 장점

- 표준이기 때문에 개발자들이 개념을 익히기 쉬움
- 유용한 디폴트 메서드를 많이 제공해준다.



## java.util.function 패키지에 있는 기본 인터페이스 6개

- Operator : 반환값과 인수의 타입이 같은 함수를 뜻한다.
  => UnaryOperator : 인수가 1개
  => BinaryOperator : 인수가 2개
- Predicate : 인수 하나를 받아 boolean을 반환하는 함수를 뜻한다.
- Function : 인수와 반환 타입이 다른 함수를 뜻한다.
- Supplier : 인수를 받지 않고 반환하는 함수를 뜻한다.
- Consumer : 인수를 하나 받고 반환값은 없는 함수를 뜻한다.

| **인터페이스**    | **함수 시그니처**   | **예**              |
| ----------------- | ------------------- | ------------------- |
| UnaryOperator<T>  | T apply(T t)        | String::toLowerCase |
| BinaryOperator<T> | T apply(T t1, T t2) | BigInteger::add     |
| Predicate<T>      | boolean test(T t)   | Collection::isEmpty |
| Function<T, R>    | R apply(T t)        | Array::asList       |
| Supplier<T>       | T get()             | Instant::now        |
| Consumer<T>       | void accept(T t)    | System.out::println |

**표준 함수형 인터페이스는 대부분 기본 타입만 지원하지만, 기본 함수형 인터페이스에 박싱된 기본 타입을 넣어 사용하는 것은 좋지 않다.** (특히 계산량이 많을 때는 성능이 처참히 느려진다.)

### 직접 만든 함수형 인터페이스에는 항상 @FunctionalInterface 애너테이션을 사용하자

- 해당 클래스의 코드나 설명 문서를 읽을 이에게 그 인터페이스가 람다용으로 설계된 것임을 알려준다.
- 해당 인터페이스가 추상 메서드를 오직 하나만 가지고 있어야 컴파일되게 해준다.
- 그 결과 그 결과 유지보수 과정에서 누군가 실수로 메서드를 추가하지 못하게 막아준다.



# Comparator는 왜 표준 인터페이스를 사용 안했을까

우리가 자주본 Comparator<T\> 인터페이스는 구조적으로 ToIntBiFunction<T,U\> 와 동일하다.

하지만 `Comparator<T>` 가 독자적인 인터페이스로 살아남은 이유는 

- 자주 쓰이며 이름자체가 용도를 설명해 준다.
- 반드시 따라야 하는 규약이 있다.
- 유용한 디폴트 메서드를 제공할 수 있다.

이다. 

이중 하나 이상을 만족한다면 전용 함수형 인터페이스를 구현해야 할지 고민해야 한다.



# 오버로딩

함수형 인터페이스를 API에서 사용할 때는 오버로딩을 사용하면 안된다.

클라이언트에게 불필요한 모호함만 안겨줄 뿐이며, 이 모호함으로 인해 실제로 문제가 일어난다.

ExecutorService의 submit 메서드는 Callable<T\> 를 받는 것과 Runnable을 받는 것을 다중정의 했다.

그래서 올바른 메서드를 알려주기 위해 형변환해야 할 때가 생긴다.
