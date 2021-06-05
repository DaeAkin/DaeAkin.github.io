---
layout: post
title: 람다보다는 메서드 참조를 사용하자
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---



람다가 익명 클래스보다 나은 점 중에서 가장 큰 특징은 간결함이다.

그런데 메소드참조를 이용하면 람다보다 더 간결하게 만들 수 있다.

```java
Map<String,Integer> map = new HashMap<>();
map.put("a",10);
map.merge("a", 1 ,(count, incr) -> count + incr);
```

이 코드는 맵에 `"a"` 라는 키가 있으면 값을 가져와서 +1을 해주고, 없으면 1을 넣는다.

깔끔해 보이지만, 매개변수인 count와 incr은 크게 하는 일이 없어 공간을 꽤 차지한다. 

사실 이 람다는 두 인수의 합을 단순히 반환할 뿐이다.  

자바 8이 되면서 Integer 클래스와 모든 기본 타입의 박싱 타입은 이 람다와 기능이 같은 정적 메서드 sum을 제공하기 시작했다.

그래서 이 기능을 사용하면 가독성을 더 높일 수 있다.

```java
Map<String,Integer> map = new HashMap<>();
map.put("a",10);
map.merge("a", 1 ,Integer::sum);
```

매개변수 수가 늘어날수록 메서드 참조를 제거할 수 있는 코드양도 늘어난다.

하지만 어떤 람다에서는 매개변수의 이름 자체가 프로그래머에게 좋은 가이드가 되기도 한다. 이런 람다는 길이는 더 길지만 메서드 참조보다 읽이 쉽고 유지보수도 쉬울 수 있다.

람다로 할 수 없는 일이라면 메서드 참조로도 할 수 없다. 그렇더라도 메서드 참조를 사용하는 편이 보통이 더 짧고 간결하므로, 람다로 구현했을 때 너무 길거나 복잡하다면 메서드 참조가 좋은 대안이 되어준다.

즉, 람다로 작성할 코드를 새로운 메서드에 담은 다음, 람다 대신 그 메서드 참조를 사용하는 식이다. 메서드 참조에는 기능을 잘 드러내는 이름을 지어줄 수 있고 설명을 문서로 남길 수도 있다.



# 항상 메소드 참조가 좋은 것은 아니다.

Intellij 같은 IDE를 사용하다보면, 람다를 메서드참조로 바꾸라고 하지만, 보통은 따르는게 이득이지만, 항상 그런 것은 아니다.

```java
public class Main {
     public static void main(String[] args) {
         
        Service service = new Service();

        service.execute(GoshThisClassNameIsHumongous::action);

        service.execute(() -> action());

        service.execute(new ServiceExecutor() {
            @Override
            public void action() {
                System.out.println("annoy action!");
            }
        });
      }
}

class Service {
    public void execute(ServiceExecutor serviceExecutor) {
        serviceExecutor.action();
    }
}

interface ServiceExecutor {
    void action();
}

class GoshThisClassNameIsHumongous {
    public static void action() {
        System.out.println("action!");
    }
}
```

메서드 참조를 사용한  `service.execute(GoshThisClassNameIsHumongous::action)`  코드를 

람다로 변경하면 `service.execute(() -> action());` 로 된다.

메서드 참조가 더 짧지도, 명확하지도 않으니, 람다쪽이 더 낫다. 

이와 비슷하게도 `java.util.function` 패키지가 제공하는 제너릭 정적 팩터리 메서드인 `Function.identity()` 를 사용하기보다는 똑같은 기능을 제공하는 `Function.apply()` 을 이용하여 람다로 만드는 편이 코드도 짧고 명확하다.

```java
Function<Object, Object> identity1 = x -> x;
Function<Object, Object> identity2 = Function.identity();
```



# 메소드의 참조 유형

메서드 참조의 유형은 다섯 가지로, 가장 흔한 유형은 앞의 예에서 본 것처럼 정적 메서드를 가리키는 메서드 참조이다. 

먼저 인스턴스 메서드를 참조하는 유형이 두 가지 있다. 그 중 하나는 수신 객체를 특정하는 한정적 인스턴스 메서드 참조이고,

다른 하나는 수신 객체를 특정하지 않는 비한정적 인스턴스 메서드 참조다.

## 한정적 메서드 참조와 비한정적 메서드 참조

한정적 참조는 근본적으로 정적 참조와 비슷하다. **즉 함수 객체가 받는 인수와 참조되는 메서드가 받는 인수가 똑같다.**

**비한정적 참조에서는 함수 객체를 적용하는 시점에 수신 객체를 알려준다.**

이를 위해 수신 객체 전달용 매개변수가 매개변수 목록의 첫 번째로 추가되며, 그 뒤로는 참조되는 메서드 선언에 정의된 매개변수들이 뒤따른다.

비한정적 참조는 주로 스트림 파이프라인에서 매핑과 필터 함수에 쓰인다.

## 클래스 생성자와 배열 생성자

마지막으로 클래스 생성자를 가리키는 메서드 참조와, 배열 생성자를 가리키는 메서드 참조가 있다.

생성자 참조는 팩터리 객체로 사용 된다.

| 메서드 참조 유형   | 예                      | 같은 기능을 하는 람다                               |
| ------------------ | ----------------------- | --------------------------------------------------- |
| 정적               | Integer::parseInt       | str -> Integer.parseInt(str)                        |
| 한정적(인스턴스)   | Instance.now()::isAfter | Instance then = Instant.now(); t -> then.isAfter(t) |
| 비한정적(인스턴스) | String::toLowerCase     | str -> str.toLowerCase()                            |
| 클래스 생성자      | TreeMap<K, V>::new      | () -> `new TreeMap<K, V>()`                         |
| 배열 생성자        | int[]::new              | len -> new int[len]                                 |



# 정리

메서드 참조는 람다의 간단명료한 대한이 될 수 있다. 메서드 참조 쪽이 짧고 명확하다면 메서드 참조를 쓰고, 그렇지 않으면 람다를 쓰자.
