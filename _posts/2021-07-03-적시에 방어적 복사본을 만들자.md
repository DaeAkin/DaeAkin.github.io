---
layout: post
title: 적시에 방어적 복사본을 만들자
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---



## 불변

어떤 객체든 그 객체의 허락 없이는 외부에서 내부를 수정하는 일은 불가능하다.

하지만 주의를 기울이지 않으면 자기도 모르게 내부를 수정하도록 허락 하는 경우가 생긴다.

**기간을 표현하는 클래스 - Period - 불변식을 지키지 못함**

```java
public final class Period {
  private final Date start;
  private final Date end;
  
  public Period(Date start, Date end) {
    if(start.compareTo(end) > 0)
      throw new IllegalArgumentException(start + " after " + end);
    this.start = start;
    this.end = end;
  }
  
  public Date start() {
    return start;
  }
  
  public Date end() {
    return end;
  }
}
```

이 클래스는 불변 처림 보이지만 Date가 가변이라는 사실을 이용하면 불변식을 깨 뜨릴 수 있다. 

```java
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
end.setYear(78); //p의 내부를 수정함
```

하지만 자바 8 이후로 Date 대신 Instant, LocalDateTime, ZonedDateTime을 사용하면 된다.



## Period를 불변 클래스로 만들어보기

외부 공격으로부터 Period 인스턴스의 내부를 보호하려면 생성자에서 받은 가변 매개변수 각각을 방어적으로 복사 해야 한다.

그런 다음 Period 인스턴스 안에서는 원본이 아닌 복사본을 사용한다.



**매개변수의 방어적 복사본을 만든다.**

```java
public Period(Date start, Date end) {
  this.start = new Date(start.getTime());
  this.end = new Date(end.getTime());
  
  if(this.start.compareTo(this.end) > 0)
    throw new IllegalArgumentException(this.start + "after" + this.end)
}
```



이렇게 작성하면 앞전에 살펴본 공격은 더 이상 위협이 되지 않는다.

###  순서

방어적 복사본 코드를 보면 동작을 2가지 한다.

- Date 객체 방어적 복사
- 매개변수 유효검사

우리가 작성한 코드는 

1. Date 객체 방어적 복사를 먼저 한 후
2. 매개 변수 유효 검사를 한다

이렇게 작동하는데, 만약 이 순서를 바꾸게 된다면 멀티스레딩 환경에서 원본 객체의 유효성 검사한 후 복사본을 만드는 그 찰나 취약한 순간에 다른 스레드가 원본 객체를 수정할 위험이 있어서 반드시 순서를 잘 조정해야 한다.

이런 공격을 검사시점/사용시점 공격 혹은 영어 표기로 줄여서 TOCTOU라고 한다.



## Date.clone()

방어적 복사에 Date의 clone 메서드를 사용하지 않았다.

그 이유는 Date는 fianl이 아니므로 clone이 Date가 정의한게 아닐 수 있다. 

즉 clone이 악의를 가진 하위 클래스의 인스턴스를 반환할 수도 있다.

예를 들어 이 하위 클래스는 start와 end 필드의 참조를 private 정적 리스트에 담아뒀다가 공격자에게 이 리스트에 접근하는 길을 열어줄 수도 있다.

결국 공격자에게 Period 인스턴스 자체를 송두리째 맡기게 된다.

이런 공격을 막기 위해 매개변수가 제3자에 의해 확장될 수 있는 타입이라면 방어적 복사본을 만들 때 clone을 사용해서는 안 된다.

```java
public class Main {

     public static void main(String[] args) {
         MaliciousDate someDate = new MaliciousDate();
         Date copyOfMaliciousDate = someDate;
         Date anotherDate = (Date)copyOfMaliciousDate.clone();
     }

}
 class MaliciousDate extends Date {

     @Override
     public Object clone() {
         System.out.println("MailciousDate");
         return super.clone();
     }
 }
```



## 또 다른 Period 문제점

위에서 매개변수의 방어적 복사본을 만들 었지만,

getter 메서드를 이용하면 Date를 가져와서 변경할 수 있기 때문에 문제가 발생한다.

이를 막기 위해서 getter가 가변 필드의 방어적 복사본을 반환하면 된다.

```java
public Date start() {
  return new Date(start.getTime()); 
}

public Date end() {
  return new Date(end.getTime());
}
```



## 정리

방어적 복사에는 성능 저하가 따르고 또 항상 쓸수 있는 것도 아니다.(같은 패키지에 속하는 등의 이유로) 

호출자가 컴포넌트 컴포넌트 내부를 수정하지 않으리라 확신하면 방어적 복사를 생략할 수 있다. 

이러한 상황이라도 호출자에서 해당 매개변수나 반환값을 수정하지 말아야 함을 명확히 문서화하는 게 좋다.

다른 패키지에서 사용한다고 해서 넘겨받은 가변 매개변수를 항상 방어적으로 복사해 저장해야 하는 것은 아니다. 때로는 메서드나 생성자의 매개변수로 넘기는 행위가 그 객체의 통제권을 명백히 이전함을 뜻하기도 한다.

이처럼 통제권을 이전하는 메서드를 호출하는 클라이언트는 해당 객체를 더 이상 직접 수정하는 일이 없다고 약속해야 한다.

클라이언트가 건네주는 가변 객체의 통제권을 넘겨받는다고 기대하는 메서드나 생성자에서도 그 사실을 확실히 문서에 기재해야 한다. 

통제권을 넘겨받기로 한 메서드나 생성자를 가진 클래스들은 악의적인 클라이언트의 공격에 취약하다. 따라서 방어적 복사를 생략해도 되는 상황은 해당 클래스와 그 클라이언트가 상호 신뢰할 수 있을 때, 혹은 불변식이 깨지더라도 그 영향이 오직 호출한 클라이언트로 국한될 때로 한정해야 한다.
