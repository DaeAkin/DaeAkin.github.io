---
layout: post
title: public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하자
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---

다음과 같이 필드를 모아 놓은 일 외에는 아무 목적도 없는 클래스를 작성하려고 할 때가 있다.

**안좋은 클래스**

```java
class Point {
  public double x;
  public double y;
}
```

이런 클래스들은 필드에 직접 접근할 수 있어 캡슐화의 이점을 제공하지 못한다. API를 수정하지 않고는 내부 표현을 바꿀 수 없고 **불변식**을 보장 할 수 없으며,  외부에서 필드에 접근할 때 **부수 작업**을 수행할 수도 없다. 철저한 객체 지향 프로그래머는 이런 클래스를 상당히 싫어해서 필드들을 모두 private으로 바꾸고 public 접근자인 getter를 추가하자.

```java
public class Point {
    private double x;
    private double y;

    public double getX() {
        return x;
    }

    public void setX(double x) {
        this.x = x;
    }

    public double getY() {
        return y;
    }

    public void setY(double y) {
        this.y = y;
    }
}
```

이렇게 하면 Point 내부 표현 방식을 언제든지 바꿀 수 있는 유연성을 얻을 수 있다. public 클래스가 필드를 공개하면 이를 사용하는 클라이언트가 생겨날 것이므로 내부 표현 방식을 마음대로 바꿀 수 있는 장점이 있다. 

**하지만 package-private 클래스 혹은 private 중첩 클래스라면 데이터 필드를 노출해도 상관이 없다.** 그 클래스가 표현하려는 추상 개념만 올바르게 표현해주면 된다. 이 방식은 클래스 선언 면에서나 이를 사용하는 클라이언트 코드 면에서나 접근자 방식보다 훨씬 깔끔하다. 클라이언트 코드가 이 클래스 내부 표현에 묶이기는 하나, **클라이언트도 어차피 이 클래스를 포함하는 패키지 안에서만 동작하는 코드일 뿐이다.** 따라서 패키지 바깥 코드는 전혀 손대지 않고도 데이터 표현 방식을 바꿀 수 있다. private 중첩 클래스의 경우라면 수정 범위가 더 좁아져서 이 클래스를 포함하는 외부 클래스까지로 제한 된다.

## 정리

public 클래스는 절대 가변 필드를 직접 노출해서는 안 된다. 불변 필드라면 노출해도 덜 위험하지만 완전히 안심할 수는 없다. 하지만 package-private 클래스나 private 중첩 클래스에서는 종종 불변이든 가변이든 필드를 노출하는 편이 나을 때도 있다.