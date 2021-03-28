---
layout: post
title: 비검사 경고를 제거하자
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---



제너릭을 사용하기 시작하면 수많은 컴파일러 경고를 보게 된다.

- 비검사 형변환 경고
- 비검사 메서드 호출 경고
- 비검사 매개변수화 가변인수 타입 경고
- 비검사 반환 경고

등이 있다

```java
Set<String> set = new HashSet();
```

이렇게 잘못작성한 코드를 javac -Xlint:uncheck 명령어로 컴파일하면 `unchecked conversion` 에러가 나온다. 

그럼 컴파일러가 올바른 실제 타입 매개변수를 추론해준다.

이 외에도 제거하기 훨씬 어려운 경고도 있다. 하지만 포기하지말고 **할 수 있는 한 모든 비검사 경고를 제거하라.** 모두 제거한다면 그 코드는 타입 안전성이 보장된다. 즉 **런타임에 ClassCastException이 발생할 일이 없고, 개발자가 의도한 대로 잘 동작하리라 확신할 수 있다.**

## @suppressWarnings

**경고를 제거할 수는 없지만 타입이 안전하다고 확신할 수 있다면 @SuppressWarnings("unchecked") 어노테이션을 달아 경고를 숨기자.** 하지만 타입 안전함을 검증하지 않은 채 경고를 숨기면 잘못된 보안 인식을 심어준다. 그 코드는 경고 없이 컴파일되겠지만, 런타임에는 여전히 ClassCastException을 던질 수 있다. 한편, 안전하다고 검증된 비검사 경고를 그대로 두면, **<u>진짜 문제를 알리는 새로운 경고가 나와도 눈치 채지 못할 수 있다</u>**. 제거하지 않는 수많은 거짓 경고 속에 새로운 경고가 파묻힐 수 있다. 

@SuppressWarnings 어노테이션은 개별 지역변수 선언부터 클래스 전체까지 어떤 선언에도 달 수 있다. 하지만 **@SuppressWarnings 어노테이션은 항상 가능한 한 좁은 범위에 적용하자.** 보통 변수 선언, 아주 짧은 메서드, 혹은 생성자가 될 것이다. 자칫 심각한 경고를 놓칠 수 있으니 절대로 클래스 전체에 적용해서는 안 된다.

**ArrayList.toArray()**

```java
public <T> T[] toArray(T[] a) {
    if (a.length < size)
        return (T[]) Arrays.copyOf(elementData, size, a.getClass());
    System.arraycopy(elementData, 0, a, 0, size);
    if (a.length > size)
        a[size] = null;
    return a;
}
```

위에 코드는 변환된 T타입의 타입 검증을 하지 않아서 `unchecked cast` 경고가 발생한다. 이 경고를 없애기 위해  @SuppressWarnings를 메소드 전체에 달 수 있지만, 범위가 필요 이상으로 넓어지므로 좁은 범위에 적용하는 것이 좋다.

어노테이션은 선언에만 달 수 있기 때문에, 반환값을 담을 지역변수를 하나 선언하고 그 변수에 어노테이션을 달아주자.

```java
public <T> T[] toArray(T[] a) {
    if (a.length < size)
      	// 생성한 배열과 매개변수로 받은 배열의 타입이 모두 T[]로 같으므로 
      	// 올바른 형변환이다.
        return (T[]) Arrays.copyOf(elementData, size, a.getClass());
    System.arraycopy(elementData, 0, a, 0, size);
    if (a.length > size)
        a[size] = null;
    return a;
}
```

이 코드는 깔끔하게 컴파일되고 비검사 경고를 숨기는 범위도 최소로 좁혔다.

**@SuppressWarnings("unchecked")어노테이션을 사용할 때면 그 경고를 무시해도 안전한 이유를 항상 주석으로 남겨야 한다.** 다른 사람이 그 코드를 이해 하는 데 도움이 되며, 더 중요하게는, 다른 사람이 그 코드를 잘못 수정하여 타입 안전성을 잃는 상황을 줄여 준다.



## 정리

비검사 경고는 중요하니 무시하지 말자. 모든 비검사 경고는 런타임에 ClassCastException을 일으킬 수 있는 잠재적 기능성을 뜻하니 최선을 다해 제거하자.