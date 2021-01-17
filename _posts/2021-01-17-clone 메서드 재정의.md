---
layout: post
title: clone 메서드 재정의
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---

Cloneable 인터페이스는 복제해도 되는 클래스임을 명시한다.

**Cloneable 인터페이스**

```java
public interface Cloneable {
}
```

그런데 인터페이스 안에 메소드가 한개도 없다. 이 인터페이스는 대체 무슨 일을 할까?

**이 인터페이스는 Object 클래스에 있는 protected 메서드인 clone의 동작 방식을 결정한다.** Cloneable을 구현한 클래스의 인스턴스에서 clone을 호출하면 그 객체의 필드들을 하나하나 복사한 객체를 반환하며, 그렇지 않은 클래스의 인스턴스에서 호출하면 CloneNotSupportedException을 던진다.

**이는 인터페이스를 상당히 이례적으로 사용한 예이니 따라하지 않는게 좋다.** 인터페이스를 구현한다는 것은 일반적으로 해당 클래스가 그 인터페이스에서 정의한 기능을 제공한다고 선언하는 행위다. **<u>그런데 Cloneable의 경우에는 상위 클래스에 정의된 protected 메서드의 동작 방식을 변경한 것이다.</u>**

## clone 메서드의 일반 규약

다음은 Object 명세에서 가져온 clone 메서드의 일반 규약이다.

> 이 객체의 복사본을 생성해 반환한다. '복사'의 정확한 뜻은 그 객체를 구현한 클래스에 따라 다를 수 있다. 일반적인 의도는 다음과 같다. 어떤 객체 x에 대해 다음 식은 참이다.
>
> x.clone() != x  (새로운 객체를 반드시 만들어야 되는 것 같다.)
>
> 또한 다음 식도 참이다.
>
> x.clone().getClass() == x.getClass()
>
> 하지만 이상의 요구를 반드시 만족해야 한는 것은 아니다.
>
> 한편 다음 식도 일반적으로 참이지만, 역시 필수는 아니다.
>
> x.clone().equals(x)
>
> 관례상, 이 메서드가 반환하는 객체는 super.clone을 호출해 얻어야 한다. 이 클래스와 (Object를 제외한) 모든 상위 클래스가 이 관례를 따른다면 다음 식은 참이다.
>
> x.clone().getClass() == x.getClass()
>
> 관례상, 반환된 객체와 원본 객체는 독립적이어야 한다. 이를 만족하려면 super.clone으로 얻은 객체의 필드 중 하나 이상을 반환 전에 수정해야 할 수도 있다.

제대로 동작하는 clone 메서드를 가진 상위 클래스를 상속해 Cloneable을 구현하고 싶다고 해보자. 먼저 super.clone을 호출한다. 그렇게 얻은 객체는 원본의 완벽한 복제본일 것이다. 클래스에 정의된 모든 필드는 원본 필드와 똑같은 값을 갖는다. **모든 필드가 기본 타입이거나 불변 객체를 참조한다면 이 객체는 완벽히 우리가 원하는 상태라 더 손볼 것이 없다.**

## 가변 상태를 참조하지 않는 클래스용 clone 메서드

```java
public class PhoneNumber implements Cloneable{
    private short areaCode;
    private short prefix;
    private short lineNum;

    @Override
    protected PhoneNumber clone() {
        try {
            return (PhoneNumber) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```

## 가변 상태를 참조하는 Stack 클래스의 예제

**Stack 클래스**

```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16 ;

    public Stack() {
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY]; 
    }
    
    public void push(Object e) {
        //...
    }
  
    @Override
    protected Stack clone() {
        try {
            return (Stack) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```

이 클래스에서 PhoneNumber 클래스에서 clone 메서드를 만든 것처럼 똑같이 하게 되면 반환된 Stack 인스턴스의 size 필드는 올바른 값을 갖겠지만, **<u>elements 필드는 원본 Stack 인스턴스와 똑같은 배열을 참조할 것이다.</u>** 원본이나 복제본 중 하나를 수정하면 다른 하나도 수정되어 불변식을 해치게 된다. 따라서 프로그램이 이상하게 동작하거나 NullPointerException을 던질 것이다.

Stack 클래스의 하나뿐인 생성자를 호출한다면 이러한 상황은 절대 일어나지 않는다. **clone 메서드는 사실상 생성자와 같은 효과를 낸다. 즉, clone은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 한다.** 그래서 Stack의 clone 메서드는 제대로 동작하려면 스택 내부 정보를 복사해야 하는데, 가장 쉬운 방법은 elements 배열의 clone을 재귀적으로 호출해주는 것이다. **한마디로 말하자면, 내부변수들이 참조하고 있는 객체들을 Deepy Copy를 해줘야 한다는 뜻이다.**

```java
@Override
protected Stack clone() {
  try {
		Stack result = (Stack) super.clone();
    result.elements = elements.clone();
    return result;
  } catch (CloneNotSupportedException e) {
    throw new AssertionError();
  }
}
```

**그러나 만약 elements final로 선언되었다면, 이 방식은 작동하지 않는다. final 필드에는 새로운 값을 할당할 수 없기 때문이다.** 이는 근본적인 문제로, 직렬화와 마찬가지로 Cloneable 아키텍처는 '가변 객체를 참조하는 필드는 final로 선언하라'는 일반 용법과 충돌한다. 그래서 복제할 수 있는 클래스를 만들기 위해 일부 필드에서 final 한정자를 제거해야 할 수도 있다.



## 적절한 동기화가 필수

Cloneable을 구현한 클래스가 스레드세이프를 가지려면, clone 메서드를 적절히 동기화 해야한다. Object의 clone 메서드는 동기화를 신경 쓰지 않았다. 그러니 **super.clone 호출 외에 다른 할 일이 없더라고 clone을 재정의하고 동기화해줘야 한다.**



## 객체 복사를 위해 Cloneable이 최선일까?

Cloneable을 이미 구현한 클래스를 확장한다면 어쩔 수 없이 clone을 잘 작동하도록 구현해야 한다. **그렇지 않은 상황에서는 복사 생성자와 복사 패터리라는 더 나은 객체 복사 방식을 제공할 수 있다.** 복사 생성자란 단순히 자신과 같은 클래스의 인스턴스를 인수로 받는 생성자를 말한다.

**복사 생성자**

```java
public Yum(Yum yum){...};
```

**복사 팩터리**

```java
public static Yum newInstance(Yum yum){...};
```

복사 생성자와 그 변형인 복사 패터리는 Cloneable/clone 방식보다 나은 면이 많다. 언어 모순적이고 위험천만한 객체 생성 메커니즘(**생성자를 쓰지 않는 방식**)을 사용하지 않으며, 엉성하게 문서화된 규약에 기대지 않고, 정상적인 final 필드 용법과도 충돌하지 않으며, 불필요한 검사 예외를 던지지도 않고, 형변환도 필요하지 않다.

또한 이 방식을 사용하면 클라이언트는 원본의 구현 타입에 얽매이지 않고 복제본의 타입을 직접 선택할 수 있다. 예를 들어 HashSet 객체 s를 TreeSet 타입으로 복제할 수 있는 기능을 제공해 줄 수 있다. clone으로는 불가능한 이 기능을 간단히 처리 할 수 있다.



## 정리

Cloneable이 몰고 온 모든 문제를 되짚어봤을 때, 새로운 인터페이스를 만들 때는 절대 Cloneable을 확장해서는 안 되며, 새로운 클래스도 이를 구현해서는 안 된다. 기본 원칙은 복제 기능은 생성자와 팩터리를 이용하는 것이 최고 라는 것이다. 단 배열만은 clone 메서드 방식이 가장 깔끔한, 이 규칙의 합당한 예외라 할 수 있다.