---
layout: post
title: 문자열 연결은 느리니 주의하자
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---



# 문자열의 시간 복잡도

문자열 연결 연산자(+)는 여러 문자열을 하나로 합쳐주는 편리한 수단이다.

그런데 한 줄짜리 출력값 혹은 작고 크기가 고정된 객체의 문자열 표현을 만들 때라면 괜찮지만,

본격적으로 사용하기 시작하면 성능 저하가 일어 난다.

 **문자열 연결 연산자로 문자열 n개를 잇는 시간은 n^2에 비례한다** 

문자열은 불변이라서 두 문자열을 연결할 경우 양쪽의 내용을 모두 복사해야하므로 성능 저하는 피할 수 없다.

```java
public String statement() {
  String result = "";
  for (int i = 0; i < numItems(); i++) 
    result += lineForItem(i); // 문자열 연결
  return result;
}
```

만약 품목이 많을 경우 이 메서드는 느려 질 수 있다.

**성능을 포기하고 싶지 않다면 String 대신 StringBuilder를 사용하자.**

```java
public String statement2() {
  StringBuilder b = new StringBuilder(numItems() * LINE_WIDTH);
  for (int i = 0; i < numItems(); i++ )
    b.append(lineForItem(i));
  return b.toString();
}
```

자바6 이후 문자열 연결 성능을 다방면으로 개선했지만, 이 두 메서드의 성능 차이는 여전히 크다.

품목을 100개로 하고 lineForItem이 길이 80 문자열을 반환하게 하여 실행하면 후자가 6.5배 빠르다.

statement 메서드의 수행 시간은 품목 수의 제곱이 비례해 늘어나고 statement2는 선형으로 늘어나므로, 품목 수가 늘어날 수록 성능 격차도 점점 벌어진다.

# StringBuffer

StringBuilder는 스레드 안전성을 갖지 않는다.

만약 문자열 연결을 할 때 여러 스레드 간의 동기화가 필요하다면 StringBuffer를 사용해야 한다.

성능은 StringBuilder 보다는 떨어진다.
