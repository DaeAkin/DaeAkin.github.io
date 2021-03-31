---
layout: post
title: 배열보다는 리스트를 사용하자
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---

## 단어 정리

| 한글용어                 | 영문용어                | 예                          |
| ------------------------ | ----------------------- | --------------------------- |
| 매개변수화 타입          | parameterized type      | List<String\>               |
| 실제 타입 매개변수       | actual type parameter   | String                      |
| 제너릭 타입              | generic type            | List<E\>                    |
| 정규 타입 매개변수       | formal type parameter   | E                           |
| 비한정적 와일드카드 타입 | unbounded wildcard type | List<?\>                    |
| 로 타입                  | raw type                | List                        |
| 한정적 타입 매개 변수    | bounded type parameter  | <E extends Number\>         |
| 재귀적 타입 한정         | recursive type bound    | <T extends Comparable<T\>\> |
| 한정적 와일드카드 타입   | bounded wildcard type   | List<? extends Number\>     |



배열과 제너릭 타입에는 중요한 차이가 두 가지 있다. 첫 번째는 배열은 공변성이다. 반면에 제너릭은 불공변이다. 

Sub가 Super의 하위 타입이라면 배열 Sub[]는 배열 Super[]의 하위 타입이 되는 것이 공변이다.

반면 제너릭은 서로 다른 타입 Type1과 Type2가 있을 때, List<Type1\> 은 List<Type2\>의 하위 타입도 아니고 상위 타입도 아니다. 

> S extends T 일 경우를 가정.
>
> 공변성(covariance) : S와 T 사이의 서브타입 관계가 그대로 유지된다. 이 경우 해당 위치에서 서브타입인 S가 슈퍼타입인 T 대신 사용될 수 있다. 우리가 흔히 이야기하는 리스코프 치환 원칙은 공변성과 관련된 원칙이라고 생각하면 된다.
>
> 비공변성(invariance) : S와 T 사이에는 아무런 관계가 존재하지 않는다. 따라서 S대신 T를 사용하거나 대신 S를 사용할 수 없다..



## 배열 vs 제너릭

**런타임시 실패하는 배열**

```java
Object[] objectArray = new Long[1];
objectArray[0] = "타입이 달라서 넣을 수 없다."; // ArrayStoreException을 던진다.
```

**컴파일시 실패하는 제너릭**

```java
List<Object> ol = new ArrayList<Long>(); // 호환되지 않는 타입이다.
ol.add("타입이 달라 넣을 수 없다.");
```

배열은 다른 타입을 넣게 되면 런타임 시 실패하고, 제너릭은 컴파일시 실패해서 둘다 String 객체를 넣을 수는 없다. 다만 배열에서는 그 사실을 런타임에 알고, 제너릭은 컴파일 때 바로 알수 있다.

두 번째로는 배열은 실체화가 된다. 배열은 런타임에도 자신이 담기로 한 원소의 타입을 알고 있다. 그러기 때문에 런타임시 ArrayStoreException이 나오게 되는 것이다.

반대로 제너릭은 런타임시 제너릭 소거가 되기 때문에, 런타임 시 내가 어떤 타입만 담는 List였는지 알 수가 없다.

만약 위에 코드가 정상적으로 실행 되었다면,

```java
List o1 = new ArrayList();
ol.add("타입이 달라 넣을 수 없다.");
```

이며, LIst의 타입 매개 변수인 E는 Object로 변경되었을 것이다.

이렇게 둘의 차이로 인해 배열과 제너릭은 잘 함께 쓰지 못한다. 배열은 제너릭 타입, 매개변수화 타입, 타입 매개변수로 사용할 수 없다. 즉 코드를 new List<E\>, new List<String\> , new E[] 식으로 작성하면 컴파일 할 때 제너릭 배열 생성 오류를 일으 킨다.

 

## 제너릭 배열 생성을 허용하지 않는 이유

제너릭 배열을 만들지 못하게 막은 이유는 타입이 안전하지 않기 때문이다. **이를 허용한다면 컴파일러가 자동 생성한 형변환 코드에서 런타임에 ClassCastException이 발생할 수 있다. 런타임에 ClassCastException이 발생하는 일을 막아주겠다는 제너릭 타입 시스템의 취지에 어긋나는 것이다.**

**제너릭 배열 생성을 허용하지 않는 이유의 코드 **  (컴파일은 되지 않음)

```java
List<String>[] stringLists = new List<String>[1]; // (1)
List<Integer> intList = List.of(42);							// (2)
Object[] objects = stringLists;										// (3)
objects[0] = intList; 														// (4)
String s = stringLists[0].get(0);									// (5)
```

**만약의 컴파일 되었다면 아래의 모습을 가진다.**

```java
List[] stringList = new List[1];
List[] intList = List.of(42);
Object[] objects = stringLists;										
objects[0] = intList; 														
String s = stringLists[0].get(0);	
```

이 코드를 천천히 읽어보자. 제너릭 소거 때문에 타입은 다 사라지게 되어, 결국 타입 안전성이 사라져서, 5번째 줄에서 ClassCastException이 발생하게 될 것이다.

이런 일을 방지하려면 제너릭 배열이 생성되지 않도록 컴파일 오류를 내야 한다.

E, List<E\>, List<String\> 같은 타입을 실체화 불가 타입(non-reifiable type)이라고 한다. **쉽게 말해, 실체화 되지 않아서 런타임에는 컴파일타임 보다 타입 정보를 가지는 타입이다.** 소거 메커니즘 때문에 매개변수화 타입 가운데 실체화될 수 있는 타입은 List<?\> 와 Map<?,?\> 같은 비한정적 와일드카드 타입 뿐이다. 배열을 비한정적 와일드카드 타입으로 만들 수는 있지만, 유용하게 쓰일 일은 거의 없다. 

## 정리

배열과 제너릭에는 매우 다른 타입 규칙이 적용된다. 배열은 공변이고 실체화되는 반면, 제너릭은 불공변이고 타입 정보가 소거 된다. 그 결과 배열은 런타임에는 타입이 안전하지만, 컴파일타임에는 그렇지 않다. 제너릭은 반대다. 그래서 둘을 섞어 쓰기란 쉽지 않다. 둘을 섞어 쓰다가 컴파일 오류나 경고를 만나면, 가장 먼저 배열을 리스트로 대체하는 방법을 적용해보자. 

