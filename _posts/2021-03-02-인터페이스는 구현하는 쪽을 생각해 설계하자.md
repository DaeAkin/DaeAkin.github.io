---
layout: post
title: 인터페이스는 구현하는 쪽을 생각해 설계하자
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---



자바 8 이전에는 기존 구현체를 깨뜨리지 않고는 인터페이스에 메서드를 추가할 방법이 없었다. 만약 메서드를 추가했다면, 그 인터페이스를 구현하고 있는 클래스들은 반드시 추가된 메서드를 구현해야 한다. 구현 클래스가 1개면 괜찮겠지만, 100개가 넘어가버리면, 100개 전부 메서드를 구현을 해줘야 한다. 

하지만 자바 8에 와서 기존 인터페이스에 메서드를 추가할 수 있도록 디폴트 메서드를 소개했지만, 위험이 완전히 사라진 것은 아니다.

인터페이스에 디폴트 메서드를 선언하면 **그 인터페이스를 구현한 클래스에서 디폴트 메서드를 재정의 하지 않았다면,** 디폴트 구현이 사용된다. 이렇게 자바에도 기존 인터페이스에 메서드를 추가하는 길이 열렸지만 모든 기존 구현체들과 매끄럽게 연동되리라는 보장은 없다. 자바 7 까지의 세상에서는 모든 클래스가 "현재의 인터페이스에 새로운 메서드가 추가될 일은 영원히 없다"고 가정하고 작성이 됐기 때문이다. 여기에 무작정 디폴트 메서드를 넣어버리면 디폴트 메서드는 구현 클래스에 대해 아무것도 모른 채 하바의 없이 무작정 삽입 될 뿐이다.

## Collection의 디폴트 메서드 removeIf

다음은 자바 8에서 Collection 인터페이스에 새롭게 추가 된 removeIf 메서드다.

```java
default boolean removeIf(Predicate<? super E> filter) {
    Objects.requireNonNull(filter);
    boolean removed = false;
    final Iterator<E> each = iterator();
    while (each.hasNext()) {
        if (filter.test(each.next())) {
            each.remove();
            removed = true;
        }
    }
    return removed;
}
```

이 메서드는 주어진 불리언 함수가 true를 반환하는 모든 원소를 제거 한다. 

반복자를 이용해 순회하면서 각 원소를 인수로 넣어 predicate를 호출해서, predicate가 true를 반환하면 iterator의 remove() 메서드를 호출해 원소를 제거 한다.

이렇게 Collection에 새롭게 들어간 디폴드 메서는 과연 모든 Collection에 대해 정상작동 할까? 

## apache가 만든 SynchronizedCollection

apache에서 만든 [SynchronizedCollection](https://github.com/apache/commons-collections/blob/master/src/main/java/org/apache/commons/collections4/collection/SynchronizedCollection.java)이 있다.

이 클래스는 스레드 안정성을 위해서 동기화가 되어있는 list의 wrapper 클래스이다.

그래서 다음과 같이 코드를 보면 동기화 코드가 있는걸 확인할 수 있다.

**SynchronizedCollection.add()**

```java
    @Override
    public boolean add(final E object) {
        synchronized (lock) {
            return decorated().add(object);
        }
    }
```

그러나 이 클래스는 removeIf 메서드를 재정의하고 있지 않다. 그래서 자바8과 함께 쓴다면, **스레드 안전성**을 갖지 못하게 된다.

removeIf의 구현은 동기화에 관해 아무것도 모르므로 락 객체를 사용할 수 없다. 따라서 SynchronizedCollection 인스턴스를 여러 스레드가 공유하는 환경에서 한 스레드가 removeIf를 호출하면 ConcurrentModificationException이 발생하거나 다른 예기치 못한 결과로 이루어 질 수 있다.

JDK에서는 이런 문제를 예방하기 위해서 구현 클래스에서 removeIf() 디폴트메서드를 재정의를 했다. 대표적으로 Collections.synchronizedCollection이 그렇다.

**Collections.synchronizedCollection 클래스의 removeIf()**

```java
@Override
public boolean removeIf(Predicate<? super E> filter) {
  synchronized (mutex) {return c.removeIf(filter);}
}
```

하지만 JDK에 속하지 않은 제3의 기존 컬렉션 구현체들은 이런 언어 차원의 인터페이스 변화에 발맞춰 수정될 기회가 없었으며, 그 중 일부는 여전히 수정되지 않고 있다.

## 그럼 어떻게 할까?

기존 인터페이스에 디폴트 메서드로 새 메서드를 추가하는 일은 꼭 필요한 경우가 아니면 피해야 한다. 추가하려는 디폴트 메서드가 기존 구현체들과 충돌하지 않은지 심사숙고해 한다. 반면 **<u>새로운 인터페이스를 만드는 경우라면</u>** 디폴트 메서드 구현을 제공하는 것은 아주 유용한 수단이며, 그 인터페이스를 더 쉽게 구현해 활용할 수 있게끔 해준다

디폴트 메서드라는 도구가 생겼더라도 인터페이스를 설계할 때는 여전히 세심한 주의를 기울여야 한다. 디**<u>폴트 메서드로 기존 인터페이스에 새로운 메서드를 추가하면 커다란 위험도 딸려 온다.</u>** 

새로운 인터페이스라면 릴리스 전에 반드시 테스트를 거쳐야 한다. 수 많은 다른 개발자 들이 그 인터페이스를 나름의 방식으로 구현하기 때문에, 서로 다른 방식으로 최소한 세 가지는 구현해보기를 권장한다. 또한 각 인터페이스의 인스턴스를 다향한 작업에 활용하는 클라이언트도 여러 개 만들어 봐야 한다. 새 인터페이스가 의도한 용도에 잘 부합하는지를 확인하는 길은 이처럼 험난하다. 이런 작업들을 거치면 여러분은 인터페이스를 릴리스하기 전에, 즉 바로잡을 기회가 아직 남았을 때 결함을 찾아낼 수 있다. **인터페이스를 릴리스한 후라도 결함을 수정하는게 가능한 경우도 있겠지만, 절대 그 가능성에 기대서는 안된다.**