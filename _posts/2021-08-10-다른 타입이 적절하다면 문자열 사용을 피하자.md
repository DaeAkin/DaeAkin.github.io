---
layout: post
title: 다른 타입이 적절하다면 문자열 사용을 피하자
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---



**문자열은 다른 값 타입을 대신하기에 적합하지 않다.**

많은 사람이 파일, 네트워크, 키보드 입력으로부터 데이터를 받을 때 주로 문자열을 사용한다.

자연스러워 보이지만, 입력받을 데이터가 진짜 문자열일 때만 그렇게 하는게 좋다.

- 받은 데이터가 수치형이라면 int, float, BigInteger 등 적당한 수치 타입으로 변환해야 한다.
- '예/아니오' 질문의 답이라면 적절한 열거 타입이나 boolean으로 변환해야 한다.

일반화해 이야기하자면, 기본 타입이든 참조 타입이든 적절한 값 타입이 있다면 그것을 사용하고, 없다면 새로 하나 작성하자.



# 문자열은 열거타입을 대신하기에 적합하지 않다.

[int 상수 대신 열거 타입을 사용하자](https://donghyeon.dev/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C%EC%9E%90%EB%B0%94/2021/05/08/int-%EC%83%81%EC%88%98-%EB%8C%80%EC%8B%A0-%EC%97%B4%EA%B1%B0-%ED%83%80%EC%9E%85%EC%9D%84-%EC%82%AC%EC%9A%A9%ED%95%98%EC%9E%90/) 에서 말했듯이, 상수를 열거할 때는 문자열보다 열거 타입이 월등히 낫다. 



# 문자열은 혼합 타입을 대신하기에 적합하지 않다.

다음과 같이 여러 요소가 혼합된 데이털르 하나의 문자열로 표현하는 것은 대체로 좋지 않은 생각이다.

```java
String compoundKey = className = "#" + i.next();
```

이 방식의 단점은

- 두 요소를 구분해주는 문자 `#` 이 두 요소 중 하나에 쓰였다면 혼란스러운 결과를 초래 한다.
- 각 요소를 개별로 접근하려면 문자열을 파싱해야 해서 느리고, 귀찮고, 오류 가능성도 커진다.
- 적절한 equals, toString, compareTo 메서드를 제공할 수 없으며, String이 제공하는 기능에만 의존해야 한다.

그래서 차라리 전용 클래스를 새로 만드는 편이 낫다.

이런 클래스는 보통 private 정적 멤버 클래스로 선언한다.



# 문자열은 권한을 표현하기에 적합하지 않다.

권한(capacity)을 문자열료 표현하는 경우가 종종 있다.

예를 들어 스레드 지역변수 기능을 설계한다고 해보자.

이름처럼 각 스레드가 자신만의 변수를 갖게 해주는 기능이다.

자바가 이 기능을 지원하기 시작한 때는 자바 2부터로 그 전에는 프로그래머가 직접 구현해야 했다.

그 당시 이 기능을 설계해야 했던 여러 프로그래머가 독립적으로 방법을 모색하다가 결국에는 똑같은 설계에 이르렀다.

**바로 클라이언트가 제공한 문자열 키로 스레드별 지역변수를 식별한 것이다.**

**잘못된 예 - 문자열을 사용해 권한을 구분했다.**

```java
public class ThreadLocal {
  //객체 생성 불가
  private ThreadLocal() { }
  
  //현 스레드의 값을 키로 구분해서 저장한다.
  public static void set(String key, Object value);
  
  //키가 가리키는 현 스레드의 값을 반환한다.
  public static Object get(String key);
}
```

이 방식의 문제점은 스레드 구분용 문자열 키가 전역 이름 공간에서 공유된다는 점이다.

이 방식이 의도대로 동작하려면 **각 클라이언트가 고유한 키를 제공해야 한다.**

그런데 만약 **두 클라이언트가 서로 소통하지 못해 같은 키를 쓰기로 결정한다면**, 의도지 찮게 같은 변수를 공유하게 된다.

결국 두 클라이언트는 모두 제대로 기능하지 못하며, 보안도 취약하고, 악의적인 클라이언트라면 의도적으로 같은 키를 사용하여 다른 클라이언트의 값을 가져올 수 있다.

**이 API는 문자열 대신 위조할 수 없는 키를 사용하면 해결 된다. 이 키를 권한(capacity)이라고 한다.**

**Key 클래스로 권한을 구분했다.**

```java
public class ThreadLocal {
  //객체 생성 불가
  private ThreadLocal() { }
  
  public static class Key { //(권한)
    Key() {}
  }
  
  // 위조 불가능한 고유 키를 생성한다.
  public static Key getKey() {
    return new Key();
  }
  
  public static void set(Key key, Object value);
  public static Object get(Key key);
}
```

이 방법은 앞서의 문자열 기반 API의 문제 두 가지를 모두 해결해주지만, 개선할 여지가 있다.

set과 get은 이제 정적 메서드일 이유가 없으니 Key 클래스의 인스턴스 메서드로 바꾸자.

**이렇게 하면 Key는 더 이상 스레드 지역변수를 구분하기 위한 키가 아니라, 그 자체가 스레드 지역변수가 된다.**

결과적으로 지금의 톱레벨 클래스인 ThreadLocal은 별달리 하는 일이 없어지므로, 치워버리고, 중첩 클래스 Key의 이름을 ThreadLocal로 바꾸자.

```java
public final class ThreadLocal {
  public ThreadLocal();
  public void set(Object value):
  public Object get();
}
```

이 API는 get으로 얻은 Object를 실제 타입으로 형변환해서 써야 해서 타입에 안전하지 않다.

처음의 문자열 기반 API는 타입안전하게 만들 수 없으며, Key를 사용한 API도 타입안전하게 만들기 어렵다.

하지만 ThreadLocal을 매개변수화 타입으로 선언하면 문제가 해결 된다.

```java
public final class ThreadLocal<T> {
  public ThreadLocal();
  public void set(T value):
  public T get();
}
```

이제 자바의 java.lang.ThreadLocal과 흡사해졌다.

문자열 기반 API의 문제를 해결해주며, 키 기반 API보다 빠르고 우아하다.
