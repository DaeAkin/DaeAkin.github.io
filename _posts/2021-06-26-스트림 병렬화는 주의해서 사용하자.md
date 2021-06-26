---
layout: post
title: 스트림 병렬화는 주의해서 사용하자
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---



# 동시성 프로그래밍

자바는 다음과 같이 동시성 프로그래밍이 진화해왔다.

**Java5**

- java.util.concurrent 라이브러리 지원
- Executor 프레임워크 지원

**Java7**

- 고성능 병렬분해 프레임워크인 fork-join 추가

**java8**

- 병렬 스트림 실행 parallel메서드



# parallel 메서드 

**스트림을 사용해 처음부터 20개의 메르센 소수를 생성**

```java
public class Main {
    // 메르센 수는 2의 거듭제곱에서 1이 모자란 숫자를 가리킨다
    // 1, 3, 7, 15, 31
    // 메르센 수 중에서 소수인 수이다
    // 3, 7, 31, 127, 8191, 131071, 524287, 2147483647, 2305843009213693951.
    public static void main(String[] args) {
        long beforeTime = System.currentTimeMillis();

        primes().map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
                .filter(mersenne -> mersenne.isProbablePrime(50))
                .limit(20)
                .forEach(System.out::println);

        long afterTime = System.currentTimeMillis();
        System.out.println("실행시간 : "+(afterTime - beforeTime)/1000 + "초");
    }

    static Stream<BigInteger> primes() {
        return Stream.iterate(TWO, BigInteger::nextProbablePrime);
    }
}
```

이 코드를 실행하면 실행시간 7초가 걸린다. 

여기서 `parallel()` 메서드를 사용하면 실행시간이 단축이 될지 실험을 하게 되면 아예 실행이 되지 않는다.

그 이유는 **스트림 라이브러리가 이 파이프라인을 병렬화하는 방법을 찾아내지 못했기 때문이다.**

환경이 아무리 좋더라도 데이터 소스가 `Stream.iterate` 거나 중간 연산으로 limit를 쓰면 파이프라인 병렬화로는 성능 개선을 기대할 수 없다.

또한 파이프라인 병렬화는 limit를 다룰 때 CPU 코어가 남는다면 원소를 몇 개 더 처리한 후 제한된 개수 이후의 결과를 버려도 아무런 해가 없다고 가정한다. 

그런데 이 코드의 경우 새롭게 메르센 소수를 찾을 때마다 그 전 소수를 찾을 때보다 두 배 정도 걸린다. 

그 말은, 원소 하나를 계산하는 비용이 대략 그 이전까지의 원소 전부를 계산한 비용을 합친 것만큼 든다는 뜻이다. 



# 병렬화 하기 좋은 자료구조

스트림의 소스가 다음과 같으면 병렬화 하기 좋다.

- ArrayList
- HashMap
- HashSet
- ConcurrentHashMap
- 배열
- int 범위
- long 범위

위 자료 구조들은 모두 데이터를 원하는 크기로 정확하고 손쉽게 나눌 수 있어서 **일을 다수의 스레드에 분배하기 좋다는 특징이 있다.**

나누는 작업은 Spliterator가 담당하며, Spliterator 객체는 Stream이나 Iterable의 spliterator 메서드로 얻어올 수 있다.

## 참조 지역성(locality of reference)

위에 자료구조들은 이웃한 메모리에 연속적으로 저장되 있어서 참조 지역성이 좋다.

참조 지역성이 낮으면 스레드는 데이터가 주 메모리에서 캐시 메모리로 전송되어 오기를 기다리며 멍때리게 된다.

참조 지역성이 가장 뛰어난 자료구조는 기본타입의 배열이다.



# 스트림 종단 연산

스트림 종단 연산도 병렬화에 영향을 미치는데, 족단 연산 중 병렬화에 가장 적합한 것은 축소(reduction)다.

축소는 파이프라인에서 만들어진 모든 원소를 하나로 합치는 작업으로

- min
- max
- count
- sum

을 사용하거나

- anyMatch
- allMatch
- noneMatch

처럼 조건에 맞으면 바로 반환되는 메서드가 병렬화에 적합하다.

반면 가변 축소(mutable reduction)을 수행하는 Stream의 collect 메서드는 컬렉션들을 합치는 부담이 있어 병렬화게 적합하지 않다.



# spliterator 메서드를 재정의 고려

직접 구현한 Stream, Iterable, Collection이 병렬화 이점을 제대로 누리고 싶으면 spliterator 메서드를 재정의하고 테스트를 해보자.



# stream 함수 규약

Stream 명세는 사용되는 함수 객체에 관한 엄중한 규약을 정의해놨다.

예를 들어 Stream의 reduce 연산에 건네지는 accmulator(누적기)와 combiner(결합기) 함수는 반드시 결합법칙을 만족하고 간서받지 않고, 상태를 갖이 않아야 한다.

즉 (a op b) op c == a op (b op c)를 만족해야 하며, 파이프라인이 수행되는 동안 데이터 소스가 변경되지 않아야 한다.

위의 3개의 규칙을 지키지 않았더라도, 파이프라인이 순차적으로 실행된다면 올바른 결과를 얻을 수 있지만, 병렬로 실행하면 실패할 수도 있다.



# 스트림 병렬은 최후의 수단이다.

스트림 병렬은 최후의 수단이다. 다른 최적화와 마찬가지로 변경 전후로 반드시 성능을 테스트하여 병렬화를 사용할 가치가 있는지 확인해야 한다.

반드시 운영 시스템과 흡사한 환경에서 테스트하는 것이 좋으며, 보통은 병렬 스트림 파이프라인도 공통의 포크-조인 풀에서 수행된다. 

같은 스레드 풀을 사용하기 때문에 잘못된 파이프라인 하나가 시스템의 다른 부분의 성능에 악영향을 끼칠 수 있다.



# 무작위 수 스트림 병렬화

무작위 수들로 이뤄진 스트림을 병렬화하거든 `ThreadLocalRandom` , `Random` 보다는 `SplittableRandom` 인스턴스를 사용하자.

SplittableRandom은 정확히 이럴 때 쓰고자 설계된 것이라, 병렬화하면 성능이 선형으로 증가하지만

ThreadLocalRandom은 단일 스레드에서 사용하기위해 만들어져서 SplittableRandom 보다는 빠르지 않다.

Random은 모든 연산을 동기화 하기 때문에 병렬처리하면 최악의 성능을 보인다.





# 참조

[Stream API Docs ](https://docs.oracle.com/javase/8/docs/api/java/util/stream/package-summary.html#package.description)
