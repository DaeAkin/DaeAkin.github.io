---
layout: post
title: 과도한 동기화는 피하자
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---



# 과도한 동기화

과도한 동기화는 성능을 떨어뜨리고, 교착상태에 빠뜨리고, 심지어 예측할 수 없는 동작을 낳기도 한다.

**응답 불가와 안전 실패를 피하려면 동기화 메서드나 동기화 블록안에서는 제어를 절대로 클라이언트에 양도하면 안 된다.**

예를 들어 동기화된 영역 안에서는 재정의하라 수 있는 메서드는 호출하면 안 되며, 클라이언트가 넘겨준 함수 객체를 호출해서도 안 된다.

동기화된 영역을 포함한 클래스 관점에서 이런 메서드는 모두 바깥 세상에서 온 외계인이다.

그 메서드가 무슨 일을 할지 알지 못하며 통제할 수도 없다는 뜻이다.

**외계인 메서드가 하는 일에 따라 동기화된 영역은 예외를 일으키거나, 교착상태에 빠지거나 데이터 훼손할 수도 있다.**

구체적인 예를 보자.

다음은 어떤 집합(Set)을 감싼 래퍼 클래스이고,

이 클래스의 클라이언트는 집합에 원소가 추가되면 알림을 받을 수 있다.

바로 관찰자(옵저버) 패턴이다. 

```java
public class ObservableSet<E> extends ForwardingSet<E>{
    public ObservableSet(Set<E> s) {
        super(s);
    }

    private final List<SetObserver<E>> observers = new ArrayList<>();

    public void addObserver(SetObserver<E> observer) {
        synchronized (observers) {
            observers.add(observer);
        }
    }

    public boolean removeObserver(SetObserver<E> observer) {
        synchronized (observers) {
            return observers.remove(observer);
        }
    }

    private void notifyElementAdded(E element) {
        synchronized (observers) {
            for (SetObserver<E> observer : observers)
                observer.added(this, element);
        }
    }

    @Override
    public boolean add(E e) {
        boolean added = super.add(e);
        if (added)
            notifyElementAdded(e);
        return added;
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        boolean result = false;
        for ( E element :  c)
            result |= add(element); //notifyElementAdded를 호출한다.
        return result;
    }
}
```

관찰자들은 `addObserver` 와 `removeObserver` 메서드를 호출해 구독을 신청하거나 해지 한다.

두 경우 모두 다음 콜백 인터페이스의 인스턴스를 메서드에 건넨다.

```java
@FunctionalInterface
public interface SetObserver<E> {

    //ObservableSet에 원소가 더해지면 호출된다.
    void added(ObservableSet<E> set, E element);
}
```

눈으로 보기에 ObservableSet은 잘 동작할 것 같다.

다음의 코드는 0부터 99까지 출력한다.

```java
public static void main(String[] args) {

  ObservableSet<Integer> set =
    new ObservableSet<>(new HashSet<>());

  set.addObserver((s,e)-> System.out.println(e));

  for(int i = 0; i < 100; i++)
    set.add(i);

}
```

잘 된다.

```
0
1
2
3
...
99
```

코드를 좀 더 수정해서, 정수값을 출력하다가 그 값이 23이면 자기 자신을 제거(구독해지)하는 관찰자를 추가해보자.

```java
public static void main(String[] args) {

  ObservableSet<Integer> set =
    new ObservableSet<>(new HashSet<>());

  set.addObserver(new SetObserver<Integer>() {
    @Override
    public void added(ObservableSet<Integer> s, Integer e) {
      System.out.println(e);
      if (e == 23)
        s.removeObserver(this);
    }
  });
  for(int i = 0; i < 100; i++)
    set.add(i);

}
```

> 람다는 자기 자신을 참조할 수단이 없어서, 익명클래스를 사용했다.

이 코드를 실행하면 23까지 출력한 후 자신을 구독해지한 다음 `ConcurrentModificationException` 을 던진다.

관찰자의 added 메서드 호출이 일어난 시점이 `notifyElementAdded`가 관찰자들의 리스트를 순회하는 도중이기 때문이다.

- added 메서드는 ObservableSet의 removeObserver 메서드를 호출
- removeObserver 메서드는 observer.remove 메서드를 호출

여기서 문제가 발생한다.

리스트에서 원소를 제거하려 하는데, 마침 지금은 `notifyElementAdded` 메서드가 이 리스트를 순회하는 도중이다.

```java
private void notifyElementAdded(E element) {
  synchronized (observers) {
    for (SetObserver<E> observer : observers)
      observer.added(this, element);
  }
}
```

notifyElementAdded 메서드에서 수행하는 순회는 **동기화 블록** 안에 있으므로 동시에 수정이 일어나지 않도록 보장하지만,

정작 자신이 콜백을 거쳐 되돌아 수정하는 것 까지 막지는 못한다.

> 동기화 블록으로 인해 한번에 한 스레드가 공유자원을 사용한다.
>
> 하지만 obersers.added() 에서 list가 수정되는것 까지는 막지 못한다.
>
> 그 결과 `ConcurrentModificationException` 을 던진다.



# 교착 상태

이번에는 구독해지를 하는 관찰자를 작성하는데,

removeObserver를 직접 호출하지 않고 실행자 서비스(ExecutorService)를 사용해 다른 스레드한테 부탁해보자.

```java
set.addObserver(new SetObserver<Integer>() {
  @Override
  public void added(ObservableSet<Integer> s, Integer e) {
    System.out.println(e);
    if (e == 23) {
      ExecutorService exec = Executors.newSingleThreadExecutor();
      try {
        exec.submit(() -> s.removeObserver(this)).get();
      } catch (ExecutionException | InterruptedException ex) {
        throw new AssertionError(ex);
      } finally {
        exec.shutdown();
      }
    }
  }
});
```

이 코드를 실행하면 예외는 나지 않지만 교착상태에 빠진다.

백그라운드 스레드가 `s.removeObserver` 를 호출하면 잠그려고 시도하지만 락을 얻을 수 없다.

```java
public boolean removeObserver(SetObserver<E> observer) {
  synchronized (observers) {  //같은 observers 락을 사용 중
    return observers.remove(observer);
  }
}
```

메인 스레드가 이미 락을 선점하고 있기 때문이다.

**그와 동시에 메인스레드는 백그라운드 스레드가 관찰자를 제거하기만을 기다리는 중이다.**

**바로 교착상태다.**

# 교착 상태 해결

교착 상태를 해결하려면 외계인 메서드 호출을 동기화 블록 바깥으로 옮기면 된다.

notifyElementAdded에서 관찰자 리스트를 복사해서 쓰면 락 없이도 안전하게 순회할 수 있다.

이 방식을 적용하면 앞서의 두 예제에서 예외 발생과 교착상태 증상이 사라진다.

```java
private void notifyElementAdded(E element) {
  List<SetObserver<E>> snapshot;
  synchronized (observers) {
    snapshot = new ArrayList<>(observers);
  }
  for (SetObserver<E> observer : snapshot)
    observer.added(this, element);
}
```

외계인 메서드 호출을 동기화 블록 바깥으로 옮기는 더 나은 방법이 있다.

자바의 동시성 컬렉션 라이브러리의 `CopyOnWriteArrayList` 가 정확히 이 목적으로 설계 된 것이다.

이름이 말해주듯이 ArrayList를 구현한 클래스로,

내부를 변경하는 작업은 항상 깨끗한 복사본을 만들어 수행하도록 구현했다.

내부의 배열은 절대 수정되지 않으니 순회할 때 락이 필요 없어 매우 빠르다.

다른 용도로 쓰인다면 CopyOnWriteArrayList는 느리겠지만, 

수정할 일은 드물고 순회만 빈번히 일어나는 관찰자 리스트 용도로는 최적이다.

**동기화 코드가 다 사라진 ObservableSet 클래스**

```java
public class ObservableSet<E> extends ForwardingSet<E> {
    public ObservableSet(Set<E> s) {
        super(s);
    }

    private final List<SetObserver<E>> observers = new CopyOnWriteArrayList<>();

    public void addObserver(SetObserver<E> observer) {
        observers.add(observer);
    }

    public boolean removeObserver(SetObserver<E> observer) {
        return observers.remove(observer);

    }

    private void notifyElementAdded(E element) {
        for (SetObserver<E> observer : observers)
            observer.added(this, element);
    }   
}
```



# 열린호출

```java
private void notifyElementAdded(E element) {
  List<SetObserver<E>> snapshot;
  synchronized (observers) {
    snapshot = new ArrayList<>(observers);
  }
  for (SetObserver<E> observer : snapshot)
    observer.added(this, element);
}
```

이전에 작성했던 `notifyElementAdded` 처럼 동기화 영역 바깥에서 호출되는 외계인 메서드를 **<u>열린 호출(open call)</u>** 이라고 한다.

외계인 메서드(observer.added)는 얼마나 오래 실행될지 알 수 없는데,

동기화 영역 안에서 호출된다면 그동안 다른 스레드는 보호된 자원을 사용하지 못하고 대기해야만 한다.

따라서 열린 호출은 실패 방지 효과 외에도 동시성 효율을 크게 개선해준다.

> 외계인 메서드를 동기화 블록안에 사용한다고 가정해보자.
>
> 그리고 외계인 메서드 안에 Thread.sleep(10000); 코드를 추가해서 10초동안 스레드를 대기시킨다.
>
> 이 외계인 메서드를 호출한 메소드에는 동기화가 걸려 있기 때문에 
>
> 이 외계인 메서드 때문에 다른 스레드가 이 공유자원을 10초동안 락을 얻지 못하게 되어 성능이 떨어진다.



# 동기화 성능

자바의 동기화 비용은 빠르게 낮아져 왔지만, 과도한 동기화를 피하는 일은 오히려 과거 어느 때보다 중요하다.

멀티코어가 일반화된 오늘날, 과도한 동기화가 초래하는 진짜 비용은 락을 얻는 데 드는 CPU 시간이 아니다.

**바로 경쟁하느 낭비하는 시간, 즉 병렬로 실행할 기회를 잃고, 모든 코어가 메모리를 일관되게 보기 위한 지연시간이 진짜 비용이다.**

VM의 코드 최적화를 제한한다는 점도 과도한 동기화의 또 다른 숨은 비용이다.



# 가변 클래스 작성법

가변 클래스를 작성하려거든 다음 두 선택지 중 하나를 따르자.

- 동기화를 전혀 하지 말고, 그 클래스를 동시에 사용해야 하는 클래스가 외부에서 알아서 동기화하게 하자.
- 동기화를 내부에서 수행해 스레드 안전한 클래스로 만들자. 
  - 단, 클라이언트가 외부에서 객체에 락을 거는 것보다 동시성을 월등히 개선할 수 있을 때만

java.util 패키지의 클래스들은 Vector와 Hashtable을 제외하고 첫 번째 방식을 취했고

java.util.concurrent는 두 번째 방식을 취했다.

StringBuffer 인스턴스는 거의 항상 단일 스레드에서 쓰였음에도 내부적으로 동기화를 수행했다.

뒤늦게 StringBuilder가 등장한 이유이기도 하다. (StringBuffer의 동기화하지 않은 버전)

비슷환 이유로, 스레드 안전한 의사 난수 발생기인 java.util.Random은 동기화하지 않는 버전인 java.util.concurrent.ThreadLocalRandom으로 대체되었다.

선택하기 어려우면 동기화하지 말고, 대신 문서에 "스레드 안전하지 않음"이라고 명시하자.



# 동시성 성능 높이기

클래스를 내부에서 동기화하기로 했다면,

- 락 분할(lock splitting)
- 락 스트라이핑(lock striping)
- 비차단 동시성 제어(nonblocking )

등 다양한 기법으로 동시성을 높여 줄 수 있다.



# 정리

교착상태와 데이터 훼손을 피하려면 동기화 영역 안에서 외계인 메서드를 절대 호출하지말자.
