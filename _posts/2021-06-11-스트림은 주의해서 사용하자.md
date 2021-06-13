---
layout: post
title: 스트림은 주의해서 사용하자
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---



# 스트림 API

스트림 API는 다량의 데이터 처리 작업을 돕고자 자바 8에서 추가 되었다. 

스트림 API가 제공하는 추상 개념 중 핵심은 두 가지다.

- 스트림은 데이터 원소의 유한 혹은 무한 시퀀스를 뜻한다.
- 스트림 파이프라인은 이 원소들로 수행하는 연산 단계를 표현하는 개념이다.

스트림 안의 데이터 원소들은 객체 참조나 기본 타입 값이다. 

기본 타임 값은 int, long, double 세가지를 지원 한다.



# 스트림 파이프라인

스트림 파이프라인은 소스 스트림에서 시작핵 종단 연산으로 끝나며, 그 사이의 하나 이상의 중간연산이 있을 수 있다.

각 중간 연산은 스트림 어떠한 방식으로 변환하며, 각 원소에 함수를 적용하거나 필터를 걸 수 있다.

중간 연산이 이루어지면, 다른 스트림으로 변환하는데, 변환된 스트림의 원소타입은 변환 전 스트림의 원소 타입과 같을 수 있고 다를 수 있다.

종단 연산은 마지막 연산이 내놓은 스트림에 최후의 연산을 가 한다. 

원소를 정렬해 컬렉션에 담거나, 특정 원소 하나를 선택하거나, 모든 원소를 출력하는 식이다.



스트림 파이프 라인은 지연 평가된다. 즉, 종단 연산이 호출될 때 이뤄지며, 종단 연산에 쓰이지 않는 데이터 원소는 계산에 쓰이지 않는다.

이러한 지연 평가가 무한 스트림을 다룰 수 있게 해준다.

종단 연산을 빼 먹으면, 아무 일도 하지 않으니 주의 하자.



# 스트림은 언제 써야 할까? 

## **아나그램 예제**

```java
// 사전 하나를 훑어 원소 수가 많은 아나그램 그룹들을 출력한다.
// 아나그램(anagram): 철자를 구성하는 알파벳이 같고 순서만 다른 단어
// - 즉, "staple"의 키는 "aelpst"가 되고 "petals"의 키도 "aelpst"가 되면서 두 단어는 아나그램이다.
public class Anagrams {
    public static void main(String[] args) throws IOException {
        File dictionary = new File(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        Map<String, Set<String>> groups = new HashMap<>();
        try (Scanner s = new Scanner(dictionary)) {
            while (s.hasNext()) {
                String word = s.next();
                //이 부분
                groups.computeIfAbsent(alphabetize(word), (unused) -> new TreeSet<>()).add(word);
            }
        }
        for (Set<String> group : groups.values()) {
            if (group.size() >= minGroupSize) {
                System.out.println(group.size() + ": " + group);
            }
        }
    }

    private static String alphabetize(String s) {
        char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
```



## 과도한 스트림 사용

```java
public class Anagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);
        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(Collectors.groupingBy(
                    word -> word.chars().
                            sorted().
                            collect(StringBuffer::new,
                                    (sb, c) -> sb.append((char) c),
                                    StringBuilder::append).
                            toString())).
                    values().stream().
                    filter(group -> group.size() >= minGroupSize).
                    map(group -> group.size() + ": " + group).
                    forEach(System.out::println);
        }


        private static String alphabetize (String s){
            char[] a = s.toCharArray();
            Arrays.sort(a);
            return new String(a);
        }
    }
}
```

- 앞의 코드와 같은 일을 하지만 스트림을 사용하여 사전 파일을 여는 부분만 제외하면 프로그램 전체가 단 하나의 표현식으로 표현된다.
  - 사전을 여는 작업을 분리한 이유는 그저 `try-with-resources`문을 사용해 사전 파일을 닫기 위함

- 이 코드는 확실히 짧지만 읽기는 어렵다.
  - **스트림을 과용하면 프로그램이 읽거나 유지보수하기 어려워진다.**



## 적절한 스트림 사용

```java
public class Anagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);
        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(Collectors.groupingBy(word -> alphabetize(word)))
                    .values().stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .forEach(g -> System.out.println(g.size() + ": " + g));
        }
    }

    private static String alphabetize(String s) {
        char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }

}
```

- 스트림을 전에 본 적 없더라도 이 코드를 이해하기 쉬울 것이다.
  - 스트림의 변수의 이름을 `words`로 지어 스트림 안의 각 원소가 단어(word)임을 명확히 밝힘
  - 스트림 파이프라인에는 중간 연산 없이 종단 연산에서 모든 단어를 수집하여 맵으로 모음 (아나그램 끼리 묶임)
  - 맵의 `values()`가 반환한 값으로부터 새로운 `Stream<List<String>>` 스트림을 열어서 필터링 후 출력
- 람다 매개변수의 이름을 주의해서 정해야 한다.
  - **람다에서는 타입 이름을 자주 생략하므로 매개변수 이름을 잘 지어야 스트림 파이프라인의 가독성이 유지됨**
- **도우미 메서드를 적절히 확용하는 일의 중요성은 일반 반복코드에서보다는 스트림 파이르라인에서 훨씬 크다.**
  - 세부 구현을 도우미 메서드인 `alphabetize()`로 분리하여 가독성을 높임
  - 만약, 스트림 내부에서 구현을 했다면 명확성이 떨어지고 잘못 구현할 가능성이 커짐
    - 심지어는, 자바는 기본 타입인 `char`용 스트림을 지원하지 않기 때문에 성능이 느려질 수도 있음 (물론 그렇게 했어야 했다는 건 아님)

------

### char값 스트림 처리

```
"Hello world!".chars().forEach(System.out::println);
```

- 결과는 `721011081081113211911111410810`을 출력한다.

  - 반환하는 스트림의 원소는 `char`가 아닌 `int`이기 때문임

- 올바른 `print`메서드를 호출하게 하려면 아래처럼 형변환을 명시적으로 해줘야 한다.

  ```
  "Hello world!".chars().forEach(System.out.println((char) x));
  ```

- **하지만 `char` 값들을 처리할 때는 스트림을 삼가는 편이 낫다.**

------

# 데카르트 곱

데카르트 곱이단 두 집합의 원소들로 만들 수 있는 가능한 모든 조합이다.

## 반복문으로 구현시

```java
public class Deck {
    public static List<Card> newDeck() {
        List<Card> result = new ArrayList<>();
        for (Suit suit : Suit.values())
            for (Rank rank : Rank.values())
                result.add(new Card(suit, rank));

        return result;
    }
}

class Card {
    Suit suit;
    Rank rank;

    public Card(Suit suit, Rank rank) {
        this.suit = suit;
        this.rank = rank;
    }
}

enum Suit {
    A, B
}

enum Rank {
    A, B
}
```



## 스트림으로 구현시

```java
public class Deck {
    public static List<Card> newDeck() {
        return Stream.of(Suit.values())
                .flatMap(suit ->
                        Stream.of(Rank.values())
                                .map(rank -> new Card(suit, rank)))
                .collect(Collectors.toList());
    }
}
class Card {
    Suit suit;
    Rank rank;

    public Card(Suit suit, Rank rank) {
        this.suit = suit;
        this.rank = rank;
    }
}
enum Suit {
    A, B
}

enum Rank {
    A, B
}
```

- 반복문과, 스트림 둘다 좋은 방식이다.
  - 동료들의 스트림 코드 이해도에 따라서 사용하면 된다.

# 그래서 스트림은?

- 모든 반복문을 스트림으로 바꾸고 싶은 유혹이 들때가 있지만, 중간 정도 복잡한 작업에도(앞선 프로그램 처럼) 스트림과 반복문을 적절히 조합하는 게 최선이다.
  - 그러니 **기존 코드는 스트림을 사용하도록 리팩터링하되, 새 코드가 더 나아 보일 때만 반영할 것**



## 스트림, 반복문

### 코드 블록 (반복문)을 써야만 할 때

- 코드 블록에서는 범위 안의 지역변수를 읽고 수정할 수 있지만 람다에서는 `final` 이거나 사실상 `final`인 변수만 읽을 수 있고, 지역변수를 수정하는 건 불가능하다. (람다 캡쳐링)
- 코드 블록에서는 `return` 문을 사용해 메서드를 빠져나가거나, `break`나 `continue`문으로 블록 바깥의 반복문을 종료하거나 반복을 건너뛸수 있다.
  - 또한 메서드 선언에 명시된 검사 예외를 던질 수 있음
- **스트림 파이프라인은 한 값을 다른 값에 매핑하고 나면 원래의 값은 잃는 구조이다.**
  - 그러므로 중간 결과 값이 필요하면 반복문이 낫다.

### 스트림을 써야할 때

- 계산 로직 이상의 일들을 수행해야 한다면 스트림과는 맞지 않는 것이다. 
- 스트림이 안성 맞춤인 일들
  - 원소들의 시퀀스를 일관되게 변환함
  - 원소들의 시퀀스를 필터링함
  - 원소들의 시퀀스를 하나의 연산을 사용해 결합함 (더하기, 연결하기, 최솟값 구하기 등)
  - 원소들의 시퀀스를 컬렉션에 모음 (공통된 속성을 기준으로)
  - 원소들의 시퀀스에서 특정 조건을 만족하는 원소를 찾음

## 결론

- 스트림을 사용해야 멋지게 처리할 수 있는 일이 있고, 반복 방식이 더 알맞는 일도 있다.
  - 수많은 작업은 이 둘을 적절하게 조합했을 때 가장 멋지게 해결됨
- 어느쪽은 선택하는 확고부동한 규칙은 없지만 참고할 만한 지침 정도는 있다.
- **스트림과 반복 중 어느 쪽이 나은지 확신하기 어렵다면 둘 다 해보고 더 나은 쪽을 택하라**

