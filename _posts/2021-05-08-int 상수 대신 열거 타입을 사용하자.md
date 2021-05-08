---
layout: post
title: int 상수 대신 열거 타입을 사용하자
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---



# 정수 열거 타입 패턴

ENUM이 지원하기 전에는 다음 코드처럼 정수 상수를 한 묶음 선언해서 사용 했다.

```java
public static final int APPLE_FUJI			= 0;
public static final int APPLE_PIPPIN		= 1;

public static final int ORANGE_NAVEL		= 0;
public static final int ORANGE_BLOOD		= 1;
```

- 상수 값이 바뀌면 클라이언트도 반드시 다시 컴파일 해야함.
- 정수 상수는 문자열로 출력하기 까다롭다. 
  - 값을 출력하거나, 디버거로 살펴보면 숫자만 보임
- 상수가 몇개 있는지를 몰라서 순회할 수도 없음.



# 열거타입

정수 상수를 열거 타입으로 바꾸면 다음의 코드가 된다.

```java
public enum Apple {FUJI, PIPPIN}
public enum Orange {NAVEL,BLOOD}
```

열거 타입 자체는 클래스이며, 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final 필드로 공개한다.

- 밖에서 접근할 수 있는 생성자를 제공하지 않아서 사실상 fianl이다.
- 클라이언트가 인스턴스를 직접 생성하거나 확장할 수 없음
- 열거 타입 선언으로 만들어진 인스턴스들은 딱 하나씩만 존재함

- 열거 타입에는 각자의 이름공간이 있어서 이름이 같은 상수도 공존할 수 있다.

- 새로운 상수를 추가하거나 순서를 바꿔도 다시 컴파일 하지 않아도 된다.
- 열거 타입의 의  toString 메서드는 출력하기에 적합한 문자열을 내어준다.



# 데이터와 메서드를 갖는 열거 타입

열거 타입에는 타입에 관련된 메서드나 필드를 추가할 수 있어서 각 상수와 연관된 데이터를 해당 상수 자체 내재시키거나, 메서들르 추가할 수 있다.

그저 단순한 상수 모음인 열거 타입이지만, 실제로는 클래스이므로 고차원의 추상 개념을 표현해낼 수 있다.

```java
public enum Planet {
    MERCURY(3.302e+23, 2.439e6),
    MARS(6.4193 + 23, 3.393e6);


    private final double mass; // 질량
    private final double radius; // 반지름
    private final double surfaceGravity;

    // 중력 상수
    private static final double G = 6.67300E-11;

    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        surfaceGravity = G * mass / (radius * radius);
    }

    public double mass() {
        return mass;
    }

    public double radius() {
        return radius;
    }

    public double surfaceGravity() {
        return surfaceGravity;
    }
    
    public double surfaceWeight(double mass) {
        return mass * surfaceGravity; // F = ma
    }
    
}

```

**열거 타입 상수 각각을 특정 데이터와 연결지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장하면 된다.**

- 열거 타입은 근본적으로 불변이라 모든 필드는 fianl 이어야 한다.

- 열거 타입을 선언한 클래스 혹은 패키지에서만 유용한 기능은  private이나, package-private 메서드로 구현한다.

  

  

# enum 활용

- 널리쓰이는 열거 타입은 톱레벨 클래스로 만들고, 특정 톱레벨 클래스에서만 쓰인다면 해당 클래스의 멤버 클래스로 만든다.

  ```java
  public class TestClass { 
  	...
    enum TestEnum {
      ...
    }
  }
  ```

- 그게 아니라면, 톱레벨로 올려서 사용하자.



# 값에 따라 분기하는 열거타입 활용

위에서 봤던 Planet Enum 클래스의 상수들은 서로 다른 데이터와 연결되는 데 그쳤지만, 상수마다 동작이 달라져야 하는 상황이 있을 수 있다.

```java
public enum Operation {
    PLUS, MINUS, TIMES, DIVIDE;

    public double apply(double x, double y) {
        switch (this) {
            case PLUS: return x + y;
            case MINUS: return x - y;
            case TIMES: return x * y;
            case DIVIDE: return x / y;
        }
        throw new AssertionError("알 수 없는 연산 :" + this);
    }
}
```

이 코드는, 사칙연산에 관련된 ENUM 클래스이다.

간단히 설명하자면, ENUM 타입이  PLUS이면 더하기에 관련된 메소드가 실행이되고, MINUS이면, 뺄셈에 관련된 메소드가 실행된다.

하지만 이 코드는 **깨지기 쉬운 코드**이다. 

4가지 연산말고, 새로운 상수 연산을 추가한다고 가정해보자.

그럼 apply() 메서드에 case 문을 하나 더 넣어줘야 하는데, 개발자가 실수로 안넣을 수가 있는데, 컴파일러는 이를 잡아주지 못한다.

이를 해결하기 위해 **상수별 메서드 구현** 이라는 방식을 이용하면 된다.

**상수별 메서드 구현을 활용한 열거 타입**

```java
public enum Operation {
    PLUS {
        @Override
        public double apply(double x, double y) {
            return x + y;
        }
    },
    MINUS {
        @Override
        public double apply(double x, double y) {
            return x - y;
        }
    }, 
    TIMES {
        @Override
        public double apply(double x, double y) {
            return x * y;
        }
    },
    DIVIDE {
        @Override
        public double apply(double x, double y) {
            return x / y;
        }
    };

    public abstract double apply(double x, double y);
}
```

이렇게 코드를 작성하면, 새로운 상수를 추가하더라도, apply() 메서드도 재정의 해야한다고 컴파일러가 오류를 내준다.



# 전략 열거 타입

상수별 메서드 구현에는 열거 타입 상수끼리 코드를 공유하기 어렵다는 단점이 있다.

급여명세서에서 쓸 요일을 표현하는 열거타입을 만들고, 직원의 시간당 기본임금과 그날 일한 시간이 주어지만 일당을 계산한다는 메서드가 있다고 가정하자.

주중에는 초과근무를 하면 잔업수당이 주어지고, 주말 근무면 무조건 잔업수당이 주어진다. 

```java
public enum PayrollDay {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FIRADY,
    SATURDAY, SUNDAY;
    
    private static final int MINS_PER_SHIFT = 8 * 60;
    
    int pay(int minutesWorked, int payRate) { 
        int basePay = minutesWorked * payRate;
        
        int overtimePay;
        switch (this) {
            case SATURDAY: case SUNDAY: // 주말이면
                overtimePay = basePay / 2;
                break;
            default:
                overtimePay = minutesWorked <= MINS_PER_SHIFT ?
                        0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
        }
        
        return basePay + overtimePay;
    }
}
```

간결하지만, 관리 관점에서 위험하다,

휴가와 같은 새로운 타입을 추가하게 되면, case 문을 잊지말고 넣어줘야 한다.

이 요구사항을 구현할수 있는 방법은 여러가지 있다.

- 잔업수당을 계산하는 코드를 모든 상수에 중복해서 넣기
- 계산코드를 평일용과 주말용으로 나눠 각각 도우미 메서드로 작성한 다음 각 상수가 자신에게 필요한 메서드를 호출

두 방식 모드 위에 있는 코드와 비슷한 모양을 갖게 된다.

**가장 깔끔한 방법은 새로운 상수를 추가할 때 잔업수당 전략을 선택하도록 하는 방법이다.**

**전략 열거 타입 패턴**

```java
public enum PayrollDay {
    MONDAY(WEEKEND), TUESDAY(WEEKEND), WEDNESDAY(WEEKEND), THURSDAY(WEEKEND), FRIDAY(WEEKEND),
    SATURDAY(WEEKEND), SUNDAY(WEEKEND);


    private final PayType payType;

    PayrollDay(PayType payType) {
        this.payType = payType;
    }

    // 전략 열거 타입
     enum PayType {
        WEEKDAY {
            int overtimePay(int minutesWorked, int payRate) {
                return minutesWorked <= MINS_PER_SHIFT ?
                        0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int minutesWorked, int payRate) {
                return minutesWorked * payRate / 2;
            }
        };

        abstract int overtimePay(int minutesWorked, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;

        int pay(int minutesWorked, int payRate) {
            int basePay = minutesWorked * payRate;
            return basePay + overtimePay(minutesWorked,payRate);
        }
    }
}
```

PayrollDay 열거타입에 새로운 상수를 추가한 다음, 내가 원하는 PayType을 선택할 수 있어서 전략 열거 타입 패턴이라고 부른다. 

그런데 전략 열거 타입 패턴안에 포함할 만큼 유용하지 않거나, 의미상 열거타입이 속하지 않는다면 정적 메서드를 사용해볼 수 있다.

**처음에 봤던 Operation 열거타입에서, 자신과 반대되는 연산이 필요하다고 가정해보자.** 

```java
public static Operation inverse(Operation op) {
  switch(op) {
    case PLUS : return Operation.MINUS;
    case MINUS : return Operation.MINUS;
    case TIMES: : return Operation.DIVIDE;
    case DIVIDE: : return Operation.TIMES;
    default : throw new AssertionError("알 수 없는 연산 : " + op);
  }
}
```

**이렇게 기존 열거 타입에 상수별 동작을 혼합해 넣을 때는 switch 문이 좋은 선택이 될 수 있다.**

추가하려는 메서드가 의미상 열거 타입에 속하지 않고 직접 만든 열거 타입이라도 이 방식을 적용하는게 좋다.

종종 쓰이지만 열거 타입 안에 포함될 만큼 유용하지 않는 경우도 마찬가지다. 



# 결론

그래서 열거 타입은 언제 쓸까?

- 필요한 원소를 컴파일타임에 다 알 수 있는 상수 집합이라면 항상 열거 타입을 사용하자.
- 열거 타입의 성능은 정수 상수와 별반 다르지 않다.
  - 열거 타입을 메모리에 올리는 공간과 초기화는 시간이 들긴하지만 체감될 정도는 아님
- 각 상수를 특정 데이터와 연결짓거나 상수마다 다르게 동작할 때 유용
- 하나의 메서드가 상수별로 다르게 동작해야 할 때는 switch문 대신 상수별 메서드 구현을 사용하자.
- 열거 타입 상수 일부가 같은 동작을 공유한다면 전략 열거 타입 패턴을 사용하자.