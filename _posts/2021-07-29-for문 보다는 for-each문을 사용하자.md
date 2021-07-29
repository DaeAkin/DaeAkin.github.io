---
layout: post
title: for문 보다는 for-each문을 사용하자
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---



# for문 

**컬렉션 순회하는 for문**

```java
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
  Element e = i.next();
}
```



**배열 순회하는 for문 **

```java
for (int i = 0; i < a.length; i++) {
  	//a[i]
}
```

이 관용구들은 while문 보다는 낫지만 가장 좋은 방법은 아니다.

**반복자와 인덱스 변수는 모두 코드를 지저분하게 할 뿐 우리에게 진짜 필요한 건 원소들 뿐이다.**

더군다나 이처럼 쓰이는 요소 종류가 늘어나면 오류가 생길 가능성이 높아진다.

1회 반복에서 반복자는 세 번 등장하며 (Iterator가 3번등장), 인덱스는 네번(i가 4번등장) 이나 등장하여 변수를 잘못 사용할 틈새가 넓어진다.

혹시라도 잘못된 변수를 사용했을 때 컴파일러가 잡아주리라는 보장도 없다. 

마지막으로 컬렉션이나 배열이냐에 따라 코드 형태가 상당히 달라지므로 주의해야 한다.



# for-each 문

위에 문제들은 for-each문을 사용하면 해결 된다.

반복자와 인덱스 변술르 사용하지 않으니 코드가 깔끔해지고 오류가 날 일도 없다.

하나의 관용구로 컬렉션과 배열을 모두 처리할 수 있어서 어떤 컨테이너를 다루는지는 신경 쓰지 않아도 된다.

```java
for (Element e : elements) {
  ... // e로 무언가를 한다.
}
```

반복 대상이 컬렉션이든 배열이든, for-each 문을 사용해도 속도는 그대로다.

컬렉션을 중첩해 순회해야 한다면 for-each 문의 장점은 더 커진다.

**6 X 6 의 조합을 갖는 코드 for-each** 

```java
enum Face { ONE, TWO, THREE, FOUR, FIVE, SIX }
Collection<Face> faces = EnumSet.allOf(Face.class);

for (Face face1 : faces) {
  for (Face face2 : faces) {
    System.out.println(face1 + "," + face2);
  }
}
```



위와 같은 기능을 for 문으로 구현 한다면 아래처럼 된다.

```java
enum Face { ONE, TWO, THREE, FOUR, FIVE, SIX }
Collection<Face> faces = EnumSet.allOf(Face.class);

for (Iterator<Face> i = faces.iterator();  i.hasNext();) {
  Face face1 = i.next();
  for (Iterator<Face> j = faces.iterator();  j.hasNext();) {
    System.out.println(face1 + "," + j.next());
  }
}
```



# for-each 단점

for-each문을 사용할 수 없는 상황이 세 가지 존재한다.

- **파괴적인 필터링**(destructive filtering) : 컬렉션을 순회하면서 선택된 원소를 제거해야 한다면 반복자의 remove 메서드를 호출해야 한다. 자바 8부터는 Collection의 removeIf 메서드를 사용해 컬렉션을 명시적으로 순회하는 일을 피할 수 있다. 
- **변형**(transforming) : 리스트나 배열을 순회하면서 그 원소의 값 일부 혹은 전체를 교체해야 한다면 리스트의 반복자나 배열의 인덱스를 사용해야 한다.
- **병렬 반복(parallel iteration)** : 여러 컬렉션을 병렬로 순회해야 한다면 각각의 반복자와 인덱스 변수를 사용해 엄격하고 명시적으로 제어해야 한다. 

세 가지 상황 중 하나에 속할 때 일반적인 for 문을 사용하면 된다.



# for-each를 주로 쓰자

for-each 문은 컬렉션과 배열은 물론 Iterable 인터페이스를 구현한 객체라면 무엇이든 순회할 수 있다. 

Iterable 인터페이스는 다음과 같이 메서드가 단 하나 뿐이다.

```java
public interface Iterable<E> {
  Iterator<E> iterator();
}
```

Iterable을 처음부터 직접 구현하기는 까다롭지만, 원소들의 묶음을 표현하는 타입을 작성해야 한다면 Iterable을 구현하는 쪽으로 고민해보자.

Iterable을 구현해두면 그 타입을 사용하는 프로그래머가 for-each 문을 사용할 때 마다 편해진다.
