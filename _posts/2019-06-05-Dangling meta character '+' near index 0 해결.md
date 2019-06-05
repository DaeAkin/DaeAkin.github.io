---
layout: post
title: Dangling meta character '+' near index 0 해결
categories: [Java]
comments: true
---
# java.util.regex.PatternSyntaxException: Dangling meta character '+' near index 0

다음과 같이 String 문자열을 `+` 기호로 나누고 싶을 때,

```java
String str = "1+4+2";
String[] splits = str.split("+");
```

이렇게 작성하게 되면 

`java.util.regex.PatternSyntaxException: Dangling meta character '+' near index 0 +`

오류가 발생하게 됩니다. 

이 오류는 `+` 가 특별한 의미로 쓰이기 때문인데요, `+` 기호 말고도 `*` 과 `^` 으로 나눌 때도 마찬가지 입니다.

정상적으로 동작하기 위해서는

```java
String str = "1+4+2";
String[] splits = str.split("\\+");
```

다음과 같이 `\\` 를 붙여주면 됩니다.

추가적으로 `split` 함수는 인자로 regex 표현식을 사용합니다. 

[regex 문서로 이동하기](https://docs.oracle.com/javase/6/docs/api/java/util/regex/Pattern.html)

