---
layout: post
title: 이왕이면 제너릭 타입으로 만들자
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---



우리가 아이템7에서 만들었던 Stack 클래스를 살펴보자.

**아이템7의 Stack 클래스**

```java
public class Stack {
  private Object[] elements;
  private int size = 0;
  private static final int DEFAULT_INITIAL_CAPACTIY = 16;
  
  public Stack() {
    elements = new Object[DEFAULT_INITIAL_CAPACTIY];
  }
  
  public void push(Object e) {
    ensureCapacity();
    elements[size++] = e;
  }
  
  public Object pop() {
    if(size == 0)
      throw new EmptyStackException();
    return elements[--size];
  }
  
  private void ensureCapacity() {
    if(elements.length == size)
      elements = Arrays.copyOf(elements, 2 * size + 1);
  }
}
```

우리가 만든 Stack 클래스는 원소를 담는 타입이 Object 타입이다. 그래서 이 Stack을 쓰는 클라이언트 입장에서는 스택에서 꺼낸 객체를 형변환을 해야 하는데, 이 때 런타임 오류가 발생할 위험이 있다. 

이런 클래스는 제너릭으로 만들어야 한다.

일반 클래스를 제너릭 클래스를 만드는 첫 번째 단계는 클래스 선언에 타입 매개변수(E)를 추가하는 일이다.

**타입 매개변수가 추가 된 Stack 클래스**

```java
public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACTIY = 16;

    public Stack() {
        elements = new E[DEFAULT_INITIAL_CAPACTIY];
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if(size == 0)
            throw new EmptyStackException();
        return elements[--size];
    }

    private void ensureCapacity() {
        if(elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
    ... // isEmpty와 ensureCapacity 메서드는 그대로
}
```

이렇게 코드를 작성하면 생성자 부분인 `new E[DEFAULT_INITIAL_CAPACTIY]` 에서 오류가 난다. 오류가 나는 이유는 E와 같은 실체화 불가 타입으로 배열을 만들 수 없기 때문이다. 



### 해결법 1 

```java
elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
```

**이 방법은 제너릭 배열 생성을 금지하는 제약을 대놓고 우회하는 방법이다.** 이렇게 작성하면 컴파일러는 unchekced cast 경고를 발생시킨다. 

하지만, 컴파일러는 이 코드가 타입이 안전하지는 모르지만, **<u>elements 배열은 private 접근자</u>**를 가지며, **<u>클라이언트로 반환되거나 다른 메서드에 전달되는 일이 전혀 없다</u>**. push 메서드를 통해 배열에 저장되는 원소의 타입은 항상 E다. 따라서 이 비검사 형변환은 확실히 **안전하다.**

비검사 형변환이 안전함을 직접 증명했으면 [아이템 27](https://donghyeon.dev/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C%EC%9E%90%EB%B0%94/2021/03/26/%EB%B9%84%EA%B2%80%EC%82%AC-%EA%B2%BD%EA%B3%A0%EB%A5%BC-%EC%A0%9C%EA%B1%B0%ED%95%98%EC%9E%90/)처럼 범위를 최소로 좁혀 @SuppressWarning 어노테이션을 달아주자.

### 해결법 2

두번 째 방법은 **elements 필드의 타입을 E[]에서 Object[]로 바꾸는 것이다.**

```java
public class Stack<E> {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACTIY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACTIY];
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if(size == 0)
            throw new EmptyStackException();
        return elements[--size];
    }

    private void ensureCapacity() {
        if(elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
    ... // isEmpty와 ensureCapacity 메서드는 그대로
}
```

이렇게 코드를 작성하게 되면, pop() 메서드의 return 문에서 **incompatible types** 에러가 뜬다. 

그래서 return문을 `(E) elements[--size]` 로  변환하게 되면  오류 대신 unchekced cast 경고가 뜬다. E는 실체화 불가 타입이기 때문에 컴파일러는 **런타임에 이뤄지는 형변환이 안전한지 증명할 방법이 없다.** 

그래서 아까와 같이 형변환이 안전한지 검증이 되면 `@SuppressWarnings("unchecked")`로 경고를 없애주자.

```java
public E pop() {
  if(size == 0)
	  throw new EmptyStackException();
  @SuppressWarnings("unchecked")
  E result = (E) elements[--size];
  return result;
}
```



## 두 해결법의 차이점 

첫 번째 방법은 가독성이 더 좋다. 배열의 타입을 E[]로 선언해서 오직 E 타입만 받는다는걸 알 수 있고 코드도 더 짧다. 보통 제너릭 클래스라면 코드 이곳저곳에서 이 배열을 자주 사용하는데, **첫 번째 방식에서는 형변환을 배열 생성시 한번만 해주면 되는데, 두 번째는 배열에서 원소를 읽을 때 마다 해줘야 한다.** 그래서 현업에서는 첫 번째 방법을 더 선호하며 자주 사용한다.

하지만 배열의 런타임 타입이 컴파일타임 타입과 달라 힙 오염을 일으킬 수 있어, 힙 오염이 마음에 걸리는 개발자는 두 번째 방식을 사용하기도 한다.

> Heap Pollution
>
> Java generic type에서 변수화 된 타입이 가리키는 오브젝트의 타입이 해당 변수화 된 타입의 타입과 다를 때를 의미한다.
>
> 즉 제너릭 소거 때문에 컴파일시 타입이 변경되어, 런타임과 컴파일 타임때 서로 타입이 다를 때를 의미한다.

## 정리

클라이언트에서 직접 형변환해야 하는 타입보다 제너릭 타입이 더 안전하고 쓰기 편하다. 그러나 새로운 타입을 설계할 때는 형변환 없이도 사용할 수 있도록 하자. 그렇게 하려면 제너릭 타입으로 만들어야 할 경우가 많다. 기존 타입 중 제너릭이었어야 하는 게 있다면 제너릭 타입으로 변경하자. 기존 클라이언트에는 아무 영향을 주지 않으면서, 새로운 사용자를 훨씬 편하게 해준다. 