---
layout: post
title: 태그 달린 클래스보다는 클래스 계층구조를 활용하자
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---

이번 아이템은 객체지향의 `다형성` 에 가까운 내용인 것 같다.

## 태그

**태그란 해당 클래스가 어떠한 타입인지에 대한 정보를 담고있는 멤버 변수를 의미한다.**

다음 코드는 원과 사각형을 표현할 수 있는 클래스다.

```java
public class Figure {
    enum Shape {RECTANGLE, CIRCLE}

    //태그 필드 - 현재 모양을 나타낸다.
    private Shape shape;

    // 다음 필드들은 모양이 사각형일 때만 사용.
    private double length;
    private double width;

    // 다음 필드들은 모양이 원일 때만 싸용.
    private double radius;

    //원용 생성자
    public Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

    //사각형용 생성자
    public Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    private double area() {
        switch (shape) {
            case RECTANGLE:
                return length + width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}
```

이 Figure 클래스는 두 가지의 책임이 있다. 한 가지는 Circle의 관련된 책임이고, 다른 한 가지는 Rectangle에 관련된 책임이다.  

로버트 마틴 만든 [SOLID 규칙](https://ko.wikipedia.org/wiki/SOLID_(%EA%B0%9D%EC%B2%B4_%EC%A7%80%E D%96%A5_%EC%84%A4%EA%B3%84)) 을 생각해보자.

### 단일 책임 원칙 (Single responsibility principle)

SRP는 클래스의 책임이 한가지만 있어야 한다는 것이다. 여기서 FIgure 클래스는 두 가지의 책임이 있다. 한 가지는 Circle의 관련된 책임이고, 다른 한 가지는 Rectangle에 관련된 책임이다.  

### 개방-폐쇄 원칙 (Open/closed principle)

OCP는 확장에는 열려있으나 변경에는 닫혀 있어야한다. **확장에 대해 열려 있다는 말은**,애플리케이션의 요구사항이 변경될 때 이 변경에 맞게 새로운 동작을 추가해서 애플린케이션의 기능을 확장할 수 있다는 말이다. **수정에 대해 닫혀 있다는 말은,** 기존의 코드를 수정하지 않고도 애플리케이션의 동작을 추가하거나 변경할 수 있다는 말이다.

Figure 클래스에서 원과 사각형이 아닌 정사각형을 추가한다고 가정해보자. 

```java
package test;

public class Figure {
    enum Shape {RECTANGLE, CIRCLE , SQUARE }

    //태그 필드 - 현재 모양을 나타낸다.
    private Shape shape;

    // 다음 필드들은 모양이 사각형일 때만 사용.
    private double length;
    private double width;

    // 다음 필드들은 모양이 원일 때만 싸용.
    private double radius;
    
    //정사각형 일때 사용 -- 추가
    private int side;
    
    //원용 생성자
    public Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

    //사각형용 생성자
    public Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }
    
    // 정사각형용 생성자
    public Figure(int side) {
        shape = Shape.SQUARE;
        this.side = side;
    }
    
    private double area() {
        switch (shape) {
            case RECTANGLE:
                return length + width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            case SQUARE:
                return side * side; // -- 추가
            default:
                throw new AssertionError(shape);
        }
    }
}

```

정사각형을 추가한다면 위와 같이 기존의 코드를 수정을 해야한다. 이런 태그 달린 클래스에 단점은, 우선 열거 타입 선언, 태그 필드, switch등 쓸데없는 코드가 많다. 여러 구현이 한 클래스에 혼합돼 있어서 가독성이 나쁘기 때문에 추후에 수정을해야 할 상황이 생기면 어디를 수정해야할지 감이 안온다. 또한 새로운 타입을 추가할 때마다 모든 switch 문을 찾아 새 의미를 처리하는 코드를 추가해야 하는데, 하나라도 빠지면 역시 런타임에 문제가 일어난다. 

마지막으로 클래스의 타입만으로는 현재 나타내는 의미를 알 길이 전혀 없다.

```java
Figure figure = new Figure(0.5,0.3); //어떤 타입인지 정확히 알 수 있을까?
```

따라서 태그 달린 클래스는 장황하고, 오류를 내기 쉽고, 비효율적이다.



## 해결법

이 문제를 해결하기 위해서는 클래스를 계층구조로 바꿔야 한다.

공통의 것을 추출하여 상위클래스 올리고, 변경되는 부분만 하위클래스에 재정의 해주면 된다.

```java
abstract class Figure {
  abstract dobule area(); 
}

class Circle extends Figure {
  final double radius;
  
  Circle(double radius) {this.radius = radius;}
  
  @Override double area() {return Math.PI * (radius * radius);}
}

class Rectangle extends Figure {
  final double length;
  final double width;
  
  Rectangle(double length, double width) {
    this.length = length;
    this.width = width;
  }
  
  @Override double area() {return length * width;}
}
```

이렇게 리팩토링을 하면 위에서 작성했던 코드의 단점을 모두 없애준다.

만약 새로운 타입을 추가하고 싶다면 기존의 코드를 변경할 필요 없이 다음의 클래스를 추가하면 된다.

```java
class Square extends Rectangle { 
	Square(double side) {
    super(side,side);
  }
}
```

또한 타입 사이의 자연스러운 계층 관계를 반영할 수 있어서 유연성은 물론 컴파일타임 타입 검사 능력을 높여준다는 장점도 있다.

## 정리

태그 달린 클래스를 써야 하는 상황은 거의 없다. 새로운 클래스를 작성하는 데 태그 필드가 등장한다면 태그를 없애고 계층구조로 대체하는 방법을 생각해보자. 기존 클래스가 태그 필드를 사용하고 있다면 계층구조로 리팩토링 하는걸 고민해보자.