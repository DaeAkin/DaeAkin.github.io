---
layout: post
title: 공유 중인 가변 데이터는 동기화해 사용하자
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---



# synchronized

`synchronized` 키워드는 해당 메서드나 블록을 한번에 한 스레드씩 수행하도록 한다.

많은 개발자들이 동기화를 배타적 실행, 즉 한 스레드가 변경하는 중이라서 상태가 일관되지 않은 순간의 객체를 다른 스레드가 보지 못하게 막는 용도다.

한 객체가 일관된 상태를 가지고 생성되고, 이 객체에 접근하는 메서드는 그 객체에 락을 건다.

락을 건 메서드는 객체의 상태를 확인하고 필요하면 수정한다.

즉, 객체를 하나의 일관된 상태에서 다른 일관된 상태로 변화시킨다.

**동기화를 제대로 사용하면 어떤 메서드도 이 객체의 상태가 일관되지 않은 순간을 볼 수 없을 것이다.**

# 변화 감지

동기화에는 중요한 기능이 하나 더 있다.

동기화 없이는 한 스레드가 만든 변화를 다른 스레드에서 확인하지 못할 수 있다.

동기화는 일관성이 깨진 상태를 볼 수 없게 하는 것은 물론, 

**동기화된 메서드나 블록에 들어간 스레드가 같은 락의 보호하에 수행된 모든 이전 수정의 최종 결과를 보게 해준다.**

`long` 타입과 `double` 외의 변수를 읽고 쓰는 동작은 원자적이다.

이 말은 여러 스레드가 같은 변수를 **<u>동기화 없이</u>** 수정하는 중이라도, 스레드 안전성을 갖는다는 말이다.

**그러면 원자적 동작을 수행하는 변수는 동기화가 필요 없다고 생각할 수 있는데, 아주 위험한 발상이다.**

**자바 언어 명세는 스레드가 필드를 읽을 때 항상 <u>'수정이 완전히 반영된'</u> 값을 얻는다고 보장하지만,**

**한 스레드가 저장한 값이 다른 스레드에게 <u>'보이는가'</u>는 보장하지 않는다.**

**그 말은 동기화는 배타적 실행뿐 아니라 스레드 사이의 안정적인 통신에 꼭 필요하다.**

이는 한 스레드가 만든 변화가 다른 스레드에게 언제 어떻게 보이는지를 규정한 자바의 메모리 모델 때문이다.

**이 코드는 얼마나 오래 실행 될까?**

```java
public class StopThread {
  private static boolean stopRequested;
  
  public static void main(String[] args) throws InterruptedException {
    Thread backgroundThread = new Thread(() -> {
      int i = 0;
      while(!stopRequested)
        i++;
    });
    backgroundThread.start();
    
    TimeUnit.SECONDS.sleep(1);
    stopRequested = true;
  }
}
```

이 코드는 1초 후에 종료될까?

메인 스레드가 1초 후 `stopRequested` 를 true로 설정하면 backgroundThread는 반복문을 빠져나올 것 처럼 보인다.

하지만 무한루프가 실행된다.

원인은 동기화에 있다.

동기화하지 않으면 메인 스레드가 수정한 값을 백그라운드 스레드가 언제쯤에나 보게 될지 보증할 수 없다.

동기화가 빠지면 가상 머신이 다음과 같은 최적화를 수행할 수도 있는 것이다.

```java
// 원래 코드
while (!stopRequested)
  i++;

// 최적화한 코드
if (!stopRequested)
  while (true)
    i++;
```

OpenJDK 서버 VM이 실제롤 적용하는 끌어올리기(hoisting)라는 최적화 기법이다.

이 결과 프로그램은 응답 불가 상태가 되어 더 이상 진전이 없다.

stopRequested 필드를 동기화해 접근하면 이 문제를 해결할 수 있다.

```java
public class StopThread {
    private static boolean stopRequested;

    private static synchronized void requestStop() {
        stopRequested = true;
    }

    private static synchronized boolean stopRequested() {
        return stopRequested;
    }


    public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while(!stopRequested())
                i++;
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        requestStop();
    }
}
```

쓰기 메서드(requestStop)와 읽기 메서드(stopRequested) 모두를 동기화했음에 주목하자.

쓰기 메서드만 동기화해서는 안된다.

쓰기와 읽기 모두가 동기화되지 않으면 동작을 보장하지 않는다.

**아까 전에 이야기 했듯이 이 두 메서드는 동기화 없어도 원자적으로 동작한다.**

하지만 동기화는 배타적 수행과 스레드 간 통신이라는 두 가지 기능을 수행하는데, 

여기에서는 통신 목적으로만 사용된 것이다.



# volatile

반복문에서 매번 동기화하는 비용은 크진 않지만 속도가 더 빠른 대안이 있다.

다음과 같이 stopRequested 필드를 `volatile` 으로 선언하면 동기화를 생략해도 된다.

volatile 한정자는 배타적 수행과는 상관없지만 항상 가장 최근에 기록된 값을 읽게 됨을 보장한다.

```java
public class StopThread {
    private static volatile boolean stopRequested;


    public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while(!stopRequested)
                i++;
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```



# volatile은 원자적 로직에만 사용해야 한다.

volatile은 주의해서 사용해야 한다.

예를 들어 다음은 일련번호를 생성할 의도로 작성한 메서드다.

**잘못된 코드 - 동기화 필수**

```java
private static volatile int nextSerialNumber = 0;

public static int generateSerialNumber() {
  return nextSerialNumber++;
}
```

이 메서드는 매번 고유한 값을 반환할 의도로 만들어졌다.

이 메서드의 상태는 `nextSerialNumber`라는 단 하나의 필드로 결정되는데,

원자적으로 접근할 수 있고 어떤 값이든 허용한다.

따라서 굳이 동기화하지 않더라도 불변식을 보호할 수 있어 보인다.

**하지만 이 역시 동기화 없이는 올바로 동작하지 않는다.**

문제는 증가 연산자(++) 때문이다.

이 연산자는 코드상으로는 하나지만 실제로는 `nextSerialNumber` 필드에 두 번 접근한다.

먼저 값을 읽고, 그런 다음 1증가한 새로운 값을 저장하는 것이다.

만약 두 번째 스레드가 이 두 접근 사이를 비집고 들어와 값을 읽어가면 

첫 번째 스레드와 똑같은 값을 돌려받게 된다.

프로그램이 잘못된 결과를 계산해내는 이런 오류를 안전 실패라고 한다.

generateSerialNumber 메서드에 synchronized 한정자를 붙이면 이 문제가 해결 된다.

동시에 호출해도 서로 간섭하지 않으며 이전 호출이 변경한 값을 읽게 된다는 뜻이다.

메서드에 synchronized를 붙였다면 nextSerialNumber 필드에서는 volatile을 제거 해야 한다.



# 락 프리(lock-free) 객체

`java.util.concurrent.atomic` 패키지는 락이 없어도 스레드 안전한 프로그래밍을 지원하는 클래스들이 담겨 있다.

**volatile은 동기화 두 효과 중 통신 쪽만 지원하지만 이 패키지는 원자성(배타적 실행)까지 지원한다.**

우리가 generateSerialNumber에 원하는 바로 그 기능이다.

더구나 성능도 동기화 버전보다 우수하다.

**락-프리 동기화**

```java
private static final AtomicLong nextSerialNum = new AtomicLong();

public static long generateSerialNumber() {
  return nextSerialNum.getAndIncrement();
}
```



# 정리

이런 동기화 문제를 피하는 가장 좋은 방법은 애초에 가변 데이터를 공유하지 않는 것이다.

불변 데이터만 공유하거나 아무것도 공유하지 말자.

**다시 말해 가변 데이터는 단일 스레드에서만 쓰도록 하자.**

또한 사용하려는 프레임워크와 라이브러리를 깊이 이해하는 것도 중요하다.

이런 외부 코드가 우리가 인지하지 못한 스레드를 수행하는 복병으로 작용하는 경우도 있기 때문이다.

한 스레드가 데이터를 다 수정한 후 다른 스레드에 공유할 때는 해다아 객체에서 공유하는 부분만 동기화해도 된다.

그러면 그 객체를 다시 수정할 일이 생기기 전까지 다른 스레드들은 동기화 없이 자유롭게 값을 읽어갈 수 있다. 

이런 객체를 사실상 불변(effectively immutable)이라고 하고, 다른 스레드에 이런 객체를 건네는 행위를 안전 발행(safe publication) 이라고 한다.

객체를 안전하게 발행하는 방법은 많다.

클래스 초기화 과정에서 객체를 정적 필드, volatile 필드, final 필드, 혹은 보통의 락을 통해 접근하는 필드에 저장해도 된다.

그리고 동시성 컬렉션에 저장하는 방법도 있다.



# 공부해야 할것

이 글의 내용에서 자바 메모리 모델이 나온다.

자바 메모리 모델을 더 공부해보자.

- [오라클 공식 문서](https://docs.oracle.com/javase/specs/jls/se8/html/jls-17.html#jls-17.4.5)
- 자바 병렬 프로그래밍 책의 자바 메모리 모델 챕터 부분
