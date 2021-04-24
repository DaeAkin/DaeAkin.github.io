---
layout: post
title: 타입 안전 이종 컨테이너
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---



우리가 개발할 때 쓰는  `Set<E>` 과 `Map<K,V>` 등 컬렉션과 `ThreadLocal<T>` , `AtomicReference<T>` 등의 단일원소 컨테이너에도 제너릭이 많이 쓰인다.

**하지만 하나의 컨테이너에서 매개변수화할 수 있는 타입의 수가 제한 된다.**

**Interger 타입으로 제한되는 Set**

```java
Set<Integer> set = new HashSet<>(); // Integer 타입 밖에 사용 못한다.
```

하지만, 이런 제약조건들은 일반적인 용도에 맞게 설계되었기 때문에 문제가 없다.

# 다음의 구현 조건 있다고 가정해보자

예를 들어 타입별로(String,Integer 등) 즐겨 찾는 인스턴스를 저장하고 검색할 수 있는 Favorite 구현을 생각해보자.

`map.put(String.class,"밥")`  이렇게, **key에는 타입 , value에는 타입의 값이 들어가야 한다.**

**Favroite 구현** 

```java
public static void main(String[] args) {

  Map<Class<?>,Object> favorites = new HashMap<>();

  favorites.put(String.class,"밥");
  favorites.put(Integer.class,"이것도 된다.");

  Integer o = (Integer) favorites.get(Integer.class); // ClassCastException
}
```

아무 생각 없이 위에 조건대로 만든 코드다.

이 코드의 문제점은 **타입 안전성**이 없다는 것이 문제다. 

Key가 String 클래스라면, 값도 String이어야 한다.

Integer로 key를 만들었으면 값도 Integer여야 한다. 

**<u>하지만 내가만든 코드는 Key가 Integer인데, String 값이 들어갈 수 있다.</u>**

이러면 get으로 조회해올 때 `ClassCastException`이 발생한다.



# 타입 안전 이종 컨테이너를 쓰자

위에 문제점인 타입 안전성을 해결하기 위해 타입 안전 이종 컨테이너를 사용하면 된다.

```java
public class Main {
    public static void main(String[] args) {

        Favorites favorites = new Favorites();

        favorites.put(String.class,"밥");
        //밑에 코드는 컴파일 에러가 난다. 타입안정성을 얻을 수 있음!
        favorites.put(Integer.class,"이것도 된다.");

        //타입 형변환도 자동으로 해줘서 이런 코드가 필요 없다.
        Integer o = (Integer) favorites.get(Integer.class);
        //이렇게만 작성하면 된다.
        Integer o2 = favorites.get(Integer.class);

    }
}
//타입 안전 이종 컨테이너
class Favorites {
    private Map<Class<?>, Object> favorites = new HashMap<>();

    public <T> void put(Class<T> type, T instance) {
        favorites.put(Objects.requireNonNull(type), instance);
    }

    public <T> T get(Class<T> type) {
        return type.cast(favorites.get(type));
    }
}
```

위와 같이 타입 안전 이종 컨테이너를 사용하면 다음의 이점이 있다.

- **컴파일타임에 타입안정성을 보장해준다.**
- **map에서 꺼내올 때, 타입캐스팅을 클라이언트쪽에서 해주지 않아도 되서 더욱 깔끔하다.**



# 위에 코드의 단점

위에 코드도 완벽한 것 같지만 타입 안전성에 완전히 자유롭지는 않다.

```java
public class Main {
    public static void main(String[] args) {
        Favorites favorites = new Favorites();

        // raw 타입으로 넘기면 타입 안정성이 깨진다.
        favorites.put((Class)Integer.class,"이것도 된다.");
    }
}
```

**위와 같이 악의적인 클라이언트가 Class 객체를 Raw 타입으로 넘겨버리면 타입 안전성이 깨진다.** 

하지만 컴파일할 때 비검사 경고가 뜨긴 한다. 

이 문제를 해결하기 위해서 Favorites.put() 메서드에서 형변환 검사를 한번더 해주자.

**동적 형변환으로 런타임 타입 안전성 확보**

```java
/*
public <T> void put(Class<T> type, T instance) {
  favorites.put(Objects.requireNonNull(type), instance);
}
*/

public <T> void put(Class<T> type, T instance) {
  favorites.put(Objects.requireNonNull(type), type.cast(instance));
}
```



# 단점 2

Favorites 클래스는 실체화 불가 타입에는 사용할 수 없다. Key로 String이나, String[]은 가능하지만, `List<String>` 같은 실체화 불가 타입은 하지 못한다. 

생각해보면 `List<String`> 과 `List<Integer>` 는 둘다 서로 List.class 라는 같은 Class 객체이기 때문에, 둘다 똑같은 타입을 반환한다면 Favorites 객체의 내부는 아수라장이 된다.

하지만 이 단점에 대해 완벽한 해법은 없다.

[http://gafter.blogspot.com/2006/12/super-type-tokens.html](http://gafter.blogspot.com/2006/12/super-type-tokens.html)

## 그래서 타입 안전 이종 컨테이너란?

컨테이너 대신 키를 매개변수화한 다음, 컨테이너에 값을 넣거나 뺄 때 매개변수화한 키를 함께 제공한다. **이렇게 하면 제너릭 타입 시스템이 값의 타입이 키와 같음을 보장해주는데, 이러한 설계 방식을 타입 안전 이종 컨테이너 패턴이라고 한다.**

각 타입의 Class 객체를 매개변수화한 키 역할로 사용하면 되는데, 이 방식이 동작하는 이유는 class의 클래스가 제너릭이기 때문이다. class 리터럴의 타입은 Class가 아닌 `Class<T>` 다. 예를 들어 String.class의 타입은 `Class<String>` 이고, Integer.class의 타입은 `Class<Integer>` 이다. 

> 한편 컴파일타임 타입 정보와 런타임 타입 정보를 알아내기 위해 메서드들이 주고받는 class 리터럴은 타입 토큰이라고 한다.



# 하나의 컨테이너에서 매개변수화할 수 있는 타입의 수가 제한 된다라는 뜻은?

Set의 경우 인스턴스 생성할 때 `Set<Integer>` 로 만들면 이 Set은 Integer 타입밖에 사용하지 못한다.

하지만 여러 개의 타입을 사용하고 싶을 경우, 예를 들어 `Set<Class<?>>` 로 사용해도 된다. 

하지만 이렇게 사용하면, 타입안정성을 해치게 되므로, 타입안전성을 얻기 위해 `컨테이너` 를 하나 만들어야 한다.

그 컨테이너가 바로, **타입 안전 이종 컨테이너라고 부르게 되는 것이다.**

# 같이 보기 좋은 자료

[참고자료](https://homoefficio.github.io/2016/11/30/%ED%81%B4%EB%9E%98%EC%8A%A4-%EB%A6%AC%ED%84%B0%EB%9F%B4-%ED%83%80%EC%9E%85-%ED%86%A0%ED%81%B0-%EC%88%98%ED%8D%BC-%ED%83%80%EC%9E%85-%ED%86%A0%ED%81%B0/)