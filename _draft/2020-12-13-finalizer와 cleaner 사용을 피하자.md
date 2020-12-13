---
layout: post
title: finalizer와 cleaner 사용을 피하자
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---







**객체 소멸자**

자바는 두 가지 객체 소멸자를 제공한다. 그 중 **finalizer는 예측할 수 없고, 상황에 따라 위험할 수 있어 일반적으로 불필요하다.** 오동작, 낮은 성능, 이식성 문제의 원인이 되기도 한다. finalizer는 나름의 쓰임새가 몇 가지 있긴 하지만, 기본적으로 쓰지 말아야 한다. 그래서 자바 9에서 부터 finalizer는 deprecated 되었으며, **그 대안으로 cleaner를 제공하지만, cleaner는 finalizer보단 덜 위험하지만, 여전히 예측할 수 없고, 느리고, 일반적으로 불 필요하다.**

**자원 회수**

C++에서의 파괴자(destructor)는 특정 개체와 관련된 자원을 회수하는 보편적인 방법이다. 자바에서는 접근할 수 없게 된 객체를 회수하는 역할을 가비지 컬렉터가 담당하고, 개발자는 아무런 작업을 하지 않는다.

C++의 파괴자는 비메모리 자원을 회수하는 용도로 사용하며, 자바에서는 try-wtih-resources와 try-finally를 사용하여 회수한다.

**문제점**

finalizer와 cleaner는 즉시 수행된단느 보장이 없다.  객체에 접근할 수 없게 된 후 finalizer나 cleaner가 실행되기까지 얼마나 걸릴지 알 수 없다. **즉 finalizer와 cleaner로는 제때 실행되어야 하는 작업은 절대 할 수 없다.** 예를 들어 파일 닫기를 finalizer와 cleaner에 맡기면 중대한 오류를 일으킬 수 있다. 시스템이 동시에 열 수 있는 파일 개수에 한계가 있어, 시스템이 finalizer나 cleaner 실행을 게을리해서 파일을 계속 열어둔다면, 새로운 파일을 열지 못해 프로그램이 실패할 수 있다.

finalizer나 cleaner를 얼마나 신속히 수행할지는 전적으로 [가비지 컬렉터](https://donghyeon.dev/java/2020/03/31/%EC%9E%90%EB%B0%94%EC%9D%98-JVM-%EA%B5%AC%EC%A1%B0%EC%99%80-Garbage-Collection/) 알고리즘에 달려있으며, 이는 가비지 컬렉터 구현마자 천차 만별이다.

자바 언어 명세는 finalizer나 cleaner의 수행 시점뿐 아니라 수행 여부조차 보장하지 않는다. 접근할 수 없는 일부 객체에 딸린 종료 작업을 전혀 수행하지 못한 채 프로그램이 중단될 수도 있다는 얘기다. 따라서 프로그램 생애주기와 상관없는, **상태를 영구적으로 수정하는 작업에서는 절대 finalizer나 cleaner에 의존해서는 안 된다.**  예를 들어 데이터베이스 같이 공유 자원의 영구 락을 해제하는 작업을 finalizer나 cleaner에게 맡겨 놓으면, 시스템 전체가 서서히 멈출 것이다.

System.gc나 System.runFinalization 메서드에 현혹되지 말자. finalizer와 cleaner가 실행될 가능성을 높여줄 수는 있으나, 보장해주진 않는다. 

또한 finalizer 동작 중 발생한 예외는 무시되며, 처리할 작업이 남았더라도 그 순간 종료되어, 잡지 못한 예외 때문에 해당 객체는 마무리가 덜 된 상태로 남을 수 있다. 그리고 다른 스레드가 이 처럼 훼손된 객체를 사용하려 한다면 어떻게 동작할지 예측 할 수도 없다. 보통의 경우엔 잡지 못한 예외가 스레드를 중단시키고 스택 추적 내역을 출력하겠지만, 같은 일이 finalizer에서 일어난다면 경고조차 출력하지 않는다. 그나마 cleaner를 사용하는 라이브러리는 자신의 스레드를 통제하기 때문에 이러한 문제가 발생하지 않는다.

**finalizer와 cleaner는 심각한 성능 문제도 동반한다**. AutoCloseable 인터페이스를 구현한 객체를 생성하고 가비지 컬렉터가 수거하기까지 12ns가 걸린 반면(try-with-resource 사용), finalizer와 cleaner를 사용하면 50배 가까이 느려진다. 

**그렇다면 어디에 쓰는걸까?**

적절한 쓰임새는 두 가지 정도 있다. 하나는 자원의 소유자가 close 메서드를 호출하지 않는 것에 대비한 안전망 역할이다. cleaner나 finalizer가 즉시 호출되리라는 보장은 없지만, 클라이언트가 하지 않은 자원 회수를 늦게라도 해주는 것이 아예 안하는 것보다 낫다. 이런 안전망 역할의 finalizer를 작성할 때는 그럴만한 값어치가 있는지 심사숙고하자. 자바 라이브러리 중 FileInputStream, FileOutputStream, ThreadPoolExecutor가 대표적이다.

두 번째 예로 네이티브 피어와 연결된 객체에서다.



> 네이티브 피어란?
>
> **자바 네이티브 인터페이스**(Java Native Interface, JNI)는 [자바 가상 머신](https://ko.wikipedia.org/wiki/자바_가상_머신)(JVM)위에서 실행되고 있는 자바코드가 네이티브 응용 프로그램(하드웨어와 운영 체제 플랫폼에 종속된 프로그램들) 그리고 C, C++ 그리고 어샘블리 같은 다른 언어들로 작성된 라이브러리들을 호출하거나 반대로 호출되는 것을 가능하게 하는 프로그래밍 프레임워크이다.
>
> 예를 들어 JFrame은 자바 피어이다. 그러나 실제로 그래픽을 그릴 때, 네이티브 피어가 필요하다. 
>
> 그래서 JFrame에서 전부 삭제 할 때 dispose()를 호출하는 것이다.  네이티브 피어는 가비지 컬렉터가 손 댈 수 없는 영역이기 때문에, 명시적으로 네이티브 컴포넌트를 삭제해야한다.



```java
package demo;

import java.lang.ref.Cleaner;

public class Room implements AutoCloseable{
    private static final Cleaner cleaner = Cleaner.create();

    private static class State implements Runnable {
        int numJunkPiles; // 방(Room) 안의 쓰레기 수

        public State(int numJunkPiles) {
            this.numJunkPiles = numJunkPiles;
        }

        // close 메소드나 cleaner가 호출한다.
        @Override
        public void run() {
            System.out.println("방 청소");
             numJunkPiles = 0;
        }
    }

    //방의 상태. cleanable와 공유한다.
    private final State state;

    // cleanable 객체. 수거 대상이 되면 방을 청소한다.
    private final Cleaner.Cleanable cleanable;

    public Room(State state, Cleaner.Cleanable cleanable) {
        this.state = new State(state.numJunkPiles);
        this.cleanable = cleaner.register(this,state);
    }

    @Override
    public void close() throws Exception {
        cleanable.clean();
    }
}

```

