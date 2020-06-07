---
layout: post
title: Interpreter 패턴
categories: [Design Pattern]
comments: true 
---

# Interpreter 디자인 패턴

Interpreter 디자인 패턴은 **행동 디자인 패턴** 중 하나 입니다. 

Interpreter 디자인 패턴의 **가장 좋은 예시**는 자바소스 코드를 JVM이 이해할 수 있도록 바이트 코드로 바꿔주는 자바 컴파일러 입니다. 

구글 번역기 또한 어떠한 언어를 입력해도 사용자가 원하는 언어로 해석해주기 때문에 좋은 예시로 들 수 있습니다.

![](https://github.com/DaeAkin/java-design-pattern/blob/master/docs/Interpreter.png?raw=true)

## 예시

interpreter 패턴을 구현하기 위해서는 해석의 작업을 위한 <u>Interpreter 컨텍스트</u>들이 필요합니다.

그 다음 interpreter 컨텍스트가 제공하는 기능들을 사용하기 위해 각각 **다른 Expression을** 구현을 해야 합니다.

> 한글로 번역이면 KoreanExpression , 영어로 번역이면 EnglishExpression 등등..

**마지막으로** 유저로부터 입력을 받아 어떤 Expression을 쓸지 결정하고, 그 결과를 출력합니다.

이번 예제로는 어떤 숫자를 입력으로 받아 그 결과로 2진수 숫자와 16진수 출력하는 예제를 만들어 보겠습니다.

가장 먼저 할 일은 Interpreter Context 클래스를 만드는 일입니다. 이 클래스는 **입력을 해석해주는** 역할을 합니다.

#### InterpreterContext.java

```java
public class InterpreterContext {
    public String getBinaryFormat(int i) {
        return Integer.toBinaryString(i);
    }
    public String getHexadecimalFormat(int i) {
        return Integer.toHexString(i);
    }
}
```



그 다음 각각 다른 타입의 Expression 클래스를 만들기 위해 Expression 인터페이스를 만듭니다.

#### Expression.java

```java
public interface Expression {
    String interpret(InterpreterContext ic);
}
```



이제 이 Expression 인터페이스의 구현하여 2진수와 16진수로 변경해주는 2개의 클래스를 만들어보겠습니다.

#### IntToBinaryExpression.java

```java
public class IntToBinaryExpression implements Expression {

    private int i;

    public IntToBinaryExpression(int i) {
        this.i = i;
    }

    @Override
    public String interpret(InterpreterContext ic) {
        return ic.getBinaryFormat(i);
    }
}
```



#### IntToHexExpression.java

```java
public class IntToHexExpression implements Expression {
    private int i;

    public IntToHexExpression(int i) {
        this.i = i;
    }

    @Override
    public String interpret(InterpreterContext ic) {
        return ic.getHexadecimalFormat(i);
    }
}
```



이제 입력을 받아, 적절한 Expression 클래스로 넘기고, 출력을 만드는 클라이언트를 만들어 보겠습니다.

#### InterpreterClient.java

```java
public class InterpreterClient {

    public InterpreterContext ic;

    public InterpreterClient(InterpreterContext ic) {
        this.ic = ic;
    }

    public String interpret(String str) {
        Expression exp = null;
        if(str.contains("16진수")) {
            exp = new IntToHexExpression(Integer.parseInt(str.substring(0,str.indexOf(" "))));
        } else if(str.contains("2진수")) {
            exp = new IntToBinaryExpression(Integer.parseInt(str.substring(0,str.indexOf(" "))));
        } else return str;

        return exp.interpret(ic);
    }

     public static void main(String[] args) {
         String str1 = "28 의 2진수 ";
         String str2 = "28 의 16진수 ";

         InterpreterClient ec = new InterpreterClient(new InterpreterContext());
         System.out.println(str1+"= "+ec.interpret(str1));
         System.out.println(str2+"= "+ec.interpret(str2));
      }
}
```



#### 출력

```java
28 의 2진수 = 11100
28 의 16진수 = 1c
```



## UML

![](https://github.com/DaeAkin/java-design-pattern/blob/master/docs/InterpreterPattenUML.png?raw=true)

## Interpreter 패턴의 중요한 포인트

- Interpreter 패턴은 Expression 클래스들이 많아지게 되면 유지보수가 어려워 지게 되므로, 적절히 추가하는 것이 좋습니다.