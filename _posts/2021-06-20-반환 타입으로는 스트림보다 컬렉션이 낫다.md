---
layout: post
title: 반환 타입으로는 스트림보다 컬렉션이 낫다
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---



일련의 원소를 반환하는 메서드는 자바 7까지 반환 타입으로 `Collection` , `Set` , `List` 같은 컬렉션 인터페이스나, `Iterable` 이나 `배열` 을 사용 했다.

그런데 자바 8에서 스트림이라는 개념이 들어오게 되었는데, 스트림은 반복(iteration)을 지원하지 않는다. 

따라서 스트림과 반복을 알맞게 조합해야 좋은 코드가 나온다.

만약 API를 스트림만 반환하도록 짜놓으면 반한된 스트림을 for-each로 반복하길 원하는 사용자는 불만이 생기게 된다. 

하지만 Stream 인터페이스는 Iterable 인터페이스가 정의한 추상 메서드를 전부 포함할 뿐만 아니라, Iterable 인터페이스가 정의한 방식대로 동작하는데, 그럼에도 불구하고 for-each로 스트림을 반복할 수 없는 이유는 Iterable을 확장하지 않아서다.

## Stream에서 iterator 사용하기

### Stream의 iterator 사용

```java
public static void main(String[] args) {

     List<Object> testList = List.of();

    for (Object obj: (Iterable<Object>)testList.stream()::iterator) {

    }
}
```

위에 코드는 타입추론의 한계로 개발자가 형변환을 명시적으로 해줘야한다.



### 어댑터 메서드 사용

```java
public class Main {
    public static void main(String[] args) {

         List<Object> testList = List.of();
        for (Object obj: iterableOf(testList.stream())) {

        }
    }
    
    public static <E> Iterable<E> iterableOf(Stream<E> stream) {
        return stream::iterator;
    }
}
```

어댑터 메서드를 사용하면 타입추론이 문맥을 잘 파악하여 형변환을 따로하지 안해도 된다.



## iterator에서 Stream 사용하기

반대로 API가 Iterable만 반환하면 스트림을 사용하려는 사용자는 불만이 생기게 된다.

하지만 어댑터 메서드를 구현해 손쉽게 구현할 수 있다.

```java
public static <E> Stream<E> streamOf(Iterable<E> iterator) {
    return StreamSupport.stream(iterator.spliterator(),false);
}
```



# 그러므로 Collection을 반환하자

- Collection 인터페이스는 Iterable 하위타입이다.
- Collection 인터페이스는 stream 메서드도 제공한다.

그러므로 Collection을 반환하는 것이 가장 좋다.



# 정리

- 원소 시퀀스를 반환하는 메서드를 작성할 때, 스트림으로 처리하길 원하는 사용자와 반복으로 처리하길 원하는 사용자 둘다 만족 시키려고 노력하자.
- 반환 전부터 이미 원소들을 컬렉션에 담아 관리하고 있거나 컬렉션을 하나 더 만들어도 될 정도로 원소 개수가 적다면 ArrayList 같은 표준 컬렉션에 담아 봔한하자.
  - 그렇지 않으면 전용 컬렉션을 구현할지 고민하자
- 컬렉션을 반환하는게 불가능하면 스트림과 Iterable 중 더 자연스러운 것을 반환하자.
