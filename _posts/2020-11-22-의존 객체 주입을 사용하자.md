---
layout: post
title: 의존 객체 주입을 사용하자
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---

이번 장의 내용은 스프링의 **DI(Dependency Injection)**과 관련있는 내용이다.

대부분의 클래스는 하나 이상의 클래스에 의존한다. 가령 맞춤법 검사기는 사전(dictionary)에 의존하는데, 이런 클래스를 정적 유틸리티 클래스로 구현한 모습을 볼 수 있다.

**정적 유틸리티를 잘못 사용한 예 - 유연하지 않고 테스트하기 어려움**

```java
public class SpellChecker {
  private static final Lexicon dictionary = ...;
}
```

이와 비슷하게도 싱글턴으로도 구현하는 경우도 있다.

**싱글턴을 잘못 사용한 예 - 유연하지 않고 테스트하기 어려움**

```java
public class SpellChecker {
  private final Lexicon dictionary = ...;
  
  private SpellChecker(...) {}
  public static SpellChecker INSTANCE = new SpellChecker(...);
}
```

두 방식 모두 사전을 단 하나만 사용한다고 가정을 했기때문에 유연성이 떨어진다. 실제로는 사전이 언어별로 따로 있고, 특수 어휘용 사전을 별도로 두기도 한다, 그런 점에서 위에 설계는 새로운 사전으로 바꿀 수 없기에 유연성이 매우 떨어진다.

**<u>그럼 사전을 바꿀 수 있도록 final을 제거하고, setter 메소드를 추가하면 되지 않을까?</u>**

이 방법은 아쉽게도 오류를 내기 쉽고, 멀티스레드 환경에서는 사용할 수 없다. **사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않다.**

SpellChcker 클래스가 여러 자원 인스턴스를 지원해야 하며, 클라이언트가 원하는 자원(dictionary)를 사용해야 한다. 이 조건을 만족하기 위해선, **인스턴스를 생성할 때 생성자에 필요한 자 원을 넘겨주는 방식** 을 사용해야 한다. 이는 의존 객체 주입의 한 형태로, 맞춤법 검사기인 SpellChecker를 생성할 때 원하는 사전의 객체를 주입해주면 된다.

**의존 객체 주입은 유연성과 테스트 용이성을 높여준다.**

```java
public class SpellChecker {
  private final Lexicon dictionary;
  
  public SpellChecker(Lexicon dictionary) { 
    this.dictionary = Objects.requireNonNull(dictionary);
  }
}
```

의존 객체 주입 패턴은 또한 불변을 보장하기 때문에, 여러 클라이언트가 의존 객체들을 안심하고 공유할 수 있기도 하다. 

이 패턴을 변형하여 생성자에 `Supplier<T>` 를 이용하여 팩터리를 넘겨줄 수 있다.

**Supplier 를 이용한 팩터리 패턴**

```java
public class SimpleShapeFactory {

    private static Map<String, Supplier<? extends Shape>> shapesByCriteria = new HashMap<>();

    static {
        shapesByCriteria.put(ShapeType.SQUARE, Square::new);
        shapesByCriteria.put(ShapeType.CIRCLE, Circle::new);
    }

    public static Shape createShape(ShapeType shapeType) {
        return shapesByCriteria.get(shapeType).get();
    }
  
    enum ShapeType{
      SQUARE,
      CIRCLE
    }

}
```

이 방식을 사용해 클라이언트는 자신이 명시한 타입의 하위 타입이라면 무엇이든 생성할 수 있는 팩터리를 넘길 수 있다.

이 방식을 사전 예제에 대입해 보면 다음과 같다.

**LexiconFactory.java**

```java
public class LexiconFactory {

    private static final Map<LexiconType, Supplier<? extends Lexicon>> DICTIONARY = new HashMap<>();

    static {
        DICTIONARY.put(LexiconType.KOREAN, KoreanDictionary::new);
        DICTIONARY.put(LexiconType.ENGLISH, EnglishDictionary::new);
        DICTIONARY.put(LexiconType.FRENCH, FrenchDictionary::new);
    }

    private LexiconFactory() {
    }

    public static Lexicon getDictionary(LexiconType lexiconType){
				return DICTIONARY.get(lexiconType).get();
    }

    enum LexiconType{
        KOREAN,
        ENGLISH,
        FRENCH
    }
}
```

**클라이언트 사이드 코드**

```java
SpellChecker spellChecker = 
  new SpellChecker(LexiconFactory.getDictionary(LexiconFactory.LexiconType.KOREAN));
```

얼핏 보면 LexicionFactory 클래스는 DICTIONARY 변수 안에 객체를 미리 생성해 놓고, 싱글턴처럼 동일 객체를 반환 하는 것만 같다.

하지만 Supplier 라는 [람다](https://donghyeon.dev/java/2019/07/07/Java-Lambda/)표현식을 적어놨기 때문에, Supplier.get() 하는 순간 매번 새로운 객체를 반환한다.

## 정리

의존 객체주입은 유연성과 테스트 용이성을 개선해준다. 그치만, 의존성이 수천 개가 되는 큰 프로젝트에선 코드를 어지럽게 만든다. 이를 간단하게 해주는 프레임워크 들이 몇개 있다

- 대거(Dagger)
- 주스(Guice)
- 스프링(Spring)

