---
layout: post
title: try-with-resource를 사용하자
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---

자바 라이브러리에는 close 메서드를 호출해서 자원을 직접 닫아줘야 하는 자원이 많다. 대표적으로 InputStream, OutputStream, java.sql.Connection 등이 좋은 예다.
그러나 자원 닫기는 개발자가 놓치기 쉬워서 예측할 수 없는 성능 문제로 이어진다. 그래서 이런 라이브러리 들은 안전망으로 [finalizer](https://donghyeon.dev/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C%EC%9E%90%EB%B0%94/2020/12/14/finalizer%EC%99%80-cleaner-%EC%82%AC%EC%9A%A9%EC%9D%84-%ED%94%BC%ED%95%98%EC%9E%90/)를 활용해 닫히지 않는 자원을 닫을 수 있게 설계를 해놨지만, finalizer는 그리 믿을만하지 못하다.

## 자원닫기 try-finally 무슨 문제인가?

예전부터 개발자들이 많이 쓰는 자원 닫기 중 하나는 try-finally 가 주로 쓰였다.

**예제 1**

```java
String firstLineOfFile(String path) throw IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine();
    } finally {
        br.close();
    }
}
```

이렇게 써도 나쁘지 않아 보인다, 하지만 여기서 자원을 하나 더 사용한다고 가정 해보자.

**예제 2**

```java
void copy(String src, String dst) throw IOException {
    InputStream in = new FileInputStream(src);
    try {
        OutputStream out = new FileOutputStream(dst);
        try {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
        } finally {
            out.close();
        }
    } finally {
        in.close();
    }
}
```

**문제점**

**예제 1**에서 어떠한 이유로 기기에 물리적인 문제가 생기면, firstLineOfFile의 readLine() 은 예외를 던지고, br.close() 도 실패하게 된다. 이런 상황이면 <u>두번 째 예외가 첫번 째 예외를 완전히 집어 삼켜 버리기 때문에</u>, 오류 스택 추적 내역이 첫번 째 예외가 남지 않게 되어 디버깅하기가 어렵다.

**예제 2**는 자원의 개수 만큼, 자원을 닫는 코드가 늘어나는 단점이 생긴다.



## 더 나은 방법 try-with-resource

**예제 3**  - 예제 1를 변경한 코드

```java
String firstLineOfFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(
    		new FileReader(path))) {
        return br.readLine();
    }
}
```



**예제 4** - 예제 2를 변경한 코드

```java
void copy(String src, String dst) throw IOException {
    try(InputStream in = new FileInputStream(src);
    	OutputStream out = new FileOutputStream(dst)) {

        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while ((n = in.read(buf)) >= 0)
             out.write(buf, 0, n);
    }
}
```

try-with-resources 는 짧고 읽기 수월할 뿐 아니라 문제를 진단하기도 훨씬 좋다. 



**예제 5** - catch 절도 사용 할 수 있다.

```java
String firstLineOfFile(String path, String defaultVal) throws IOException {
    try (BufferedReader br = new BufferedReader(
    		new FileReader(path))) {
        return br.readLine();
    } catch (IOException e) {
        return defaultVal;
    }
}
```

위에 예제는 IOException이 나면, defaultVal을 반환한다.



## 정리

한 때 회사에서 개발하다가, OutputStream 으로 사진을 업로드 하는 메소드에서, 실수로 자원을 닫는 코드를 빼먹었다. 그 결과로 윈도우에서 해당 사진 자원을 계속 붙잡고 있어서 사진이 삭제가 안되는 버그가 있었다. 이처럼 실무에서도 많이 빼먹는 것이 자원 닫기이다.  그러므로 꼭 회수해야 하는 자원을 다룰 때는 try-finally 말고, try-with-resource를 무조건 사용하자. 코드는 더 짧고 분명해지고, 만들어지는 예외 정보도 훨씬 유용하다. try-finally로 작성하면 실용적이지 못할 만큼 코드가 지저분해지지만, try-with-resource로는 정확하고 쉽게 자원을 회수 할 수 있다.