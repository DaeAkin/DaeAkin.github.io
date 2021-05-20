---
layout: post
title: 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하자
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---



열거 타입은 거의 모든 상황에서, 앞에서 설명한 [타입 안전 열거 패턴](https://donghyeon.dev/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C%EC%9E%90%EB%B0%94/2021/05/08/int-%EC%83%81%EC%88%98-%EB%8C%80%EC%8B%A0-%EC%97%B4%EA%B1%B0-%ED%83%80%EC%9E%85%EC%9D%84-%EC%82%AC%EC%9A%A9%ED%95%98%EC%9E%90/) 보다 우수하다. 

**하지만 하나 예외가 있는데, 타입 안전 열거 패턴은 확장할 수 있으나, 열거 타입은 그럴 수가 없다.**

```java
// extends가 허용 되지 않음.
public enum TestEnum extends TestClass {
}

class TestClass {}
```

열거 타입을 확장할 수 없는 이유는 열거타입들은 이미 열거타입의 기능을 제공하는 `Enum<T>` 을 상속받았기 때문이다.



# 열거 타입 확장의 필요성

대부분 상황에서 열거 타입을 확장하는 건 좋지 않은 생각이다.

- **확장한 타입의 원소는 기반 타입의 원소로 취급하지만, 그 반대는 성립하지 않음**
- **기반 타입과 확장된 타입들의 원소를 모두 순회할 방법도 없다.**
- **확장성을 높이려면 고려할 요소가 늘어나 설계와 구현이 복잡해진다.**

# 열거 타입 확장

확장할 수 있는 열거 타입이 쓰는 경우는 연산 코드다.

연산코드의 각 원소는 특정 기계가 수행하는 연산을 뜻하는데, API가 제공하는 기본 연산 외에 사용자 확장 연산을 추가할 수 있도록 열어줘야 할 때가 있다.

그럼 열거 타입을 확장하고 싶으면 어떻게 해야 할까?

기본 아이디어는 열거 타입이 임의의 인터페이스를 구현할 수 있다는 특징을 이용하는 것이다.

다음은 Operation 타입을 확장할 수 있게 만든 코드다.

**인터페이스를 이용해 확장 가능 열거 타입을 흉내 냈다.**

```java
public interface Operation {
    double apply(double x, double y);
}
```

이 인터페이스를 열거타입이 구현하게 만들면 된다.

```java
public enum BasicOperation implements Operation{

    PLUS("+") {
        @Override
        public double apply(double x, double y) {
            return x + y;
        }
    },

    MINUS("-") {
        @Override
        public double apply(double x, double y) {
            return x - y;
        }
    };


    private final String symbol;

    BasicOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override
    public String toString() {
        return symbol;
    }
}

```

열거 타입인 `BasicOperation` 은 확장할 수 없지만, **<u>그 대신</u>** 인터페이스인 `Operation`은 확장할 수 있으니, 이 인터페이스를 연산의 타입으로 사용하면 된다.

이렇게 하면 Operation을 구현한 또 다른 열거 타입을 정의해 `BasicOperation` 을 대체할 수 있다. 

**지수 연산(EXP)과 나머지 연산(REMAINDER)을 추가해 보자.**

```java
public enum ExtendedOperation implements Operation{

    EXP("^") {
        @Override
        public double apply(double x, double y) {
            return Math.pow(x, y);
        }
    },
    REMAINDER("%") {
        @Override
        public double apply(double x, double y) {
            return x % y;
        }
    };


    private final String symbol;

    ExtendedOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override
    public String toString() {
        return symbol;
    }
}
```

- 새로 작성한 연산은 기존 연산을 쓰던 곳이면 어디든 쓸 수 있다. 
  - Operation 인터페이스를 구현만 하면 됨.
- apply가 Operation 인터페이스에 선언되어 있으니, 열거 타입안에 추상 메서드로 선언하지 않아도 됨.

클라이언트 코드에서도 기본 열거 타입 대신 확장된 열거 타입을 넘겨 확장된 열거 타입의 원소 모두를 사용하게 할 수도 있다.

```java
public class Main {

     public static void main(String[] args) {
         test(BasicOperation.class,4,2);
         test2(Arrays.asList(ExtendedOperation.values()),4,2);
      }

  		//방법 1
      private static <T extends Enum<T> & Operation> void test(
              Class<T> opEnumType, double x, double y) {
          for (Operation op : opEnumType.getEnumConstants())
              System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));

      }
			//방법 2
      private static void test2(Collection<? extends Operation> opSet,
                                double x, double y) {
         for(Operation op : opSet) {
             System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
         }
      }
}
```

방법1 보다는 방법 2가 덜 복잡하고 더 유연해진다.

- 방법2은 방법1보다 여러 구현 타입의 연산을 조합해 호출할 수 있다. 

  ```java
  List<Operation> extendedOperations = Arrays.asList(ExtendedOperation.values());
  List<Operation> opSet = Arrays.asList(BasicOperation.values());
  
  // ExtendedOperation과 BasicOperation을 합침
  List<Operation> list = new ArrayList<>();
  list.addAll(extendedOperations);
  list.addAll(opSet);
  
  test2(list,4,2);
  ```

  - 하지만, 특정 연산에서는 EnumSet과 EnumMap을 사용하지 못한다.



# 열거 타입 확장의 단점

인터페이스를 이용해 확장 가능한 열거 타입을 흉내 내는 방식에도 한 가지 사소한 문제가 있는데, **열거 타입끼리 구현을 상속할 수 없다는 점이다.**

**하지만, 아무 상태에도 의존하지 않는 경우에는 인터페이스의 디폴트 메서드를 이용해 추가하는 방법도 있다.** 



# JDK 사용 사례 

`java.nio.file.LinkOption` 열거 타입은 `CopyOption` 과 `OpenOption` 인터페이스를 구현 했다.

```java
public enum LinkOption implements OpenOption, CopyOption {

    NOFOLLOW_LINKS;
}
```



# 정리

열거 타입 자체는 확장할 수 없지만, 인터페이스와 그 인터페이스를 구현하는 기본 열거 타입을 함께 사용해 같은 효과를 낼 수 있다. 이렇게 하면 클라이언트는 이 인터페이스를 구현해 자신만의 열거 타입(혹은 다른 타입)을 만들 수 있다. 

그리고 API가 기본 열거 타입을 직접 명시하지 않고 인터페이스 기반으로 작성되었다면 기본 열거 타입의 인스턴스가 쓰이는 모든 곳을 새로 확장한 열거 타입의 인스턴스로 대체해 사용할 수 있다.