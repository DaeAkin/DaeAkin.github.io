---
layout: post
title: 스트림에서는 부작용 없는 함수를 사용하자
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---



# 스트림 패러다임

스트림 패러다임의 핵심은 계산을 일련의 변환으로 재구성 하는 부분이다.

이때 각 변환 단계는 가능한 한 이전 단계의 결과를 받아 처리하는 **<u>순수 함수</u>**여야 한다.

순수 함수란 오직 입력만이 결과에 영향을 주는 함수를 말하며, 다른 가변 상태를 참조하지 않고, 함수 스스로도 다른 상태를 변경하지 않는다.

이렇게 하려면 스트림 연산에 건네는 함수 객체는 모두 사이드이펙트가 없어야 한다.



다음은 텍스트 파일에서 단어별 수를 세어 빈도표를 만드는 코드다. 

```java
//옳바르지 않는 방식
try (Stream<String> words = new Scanner(file).tokens()) {
  words.forEach( word-> {
    // 상태를 수정한다. 옳바르지 않음.
    freq.merge(word.toLowerCase(), 1L, Long::sum);
  });
}

//옳바른 방식
try (Stream<String> words = new Scanner(file).tokens()) {
  freq = words
    .collect(groupingBy(String::toLowerCase, counting()));
}
```

옳바르지 않는 방식의 문제점은

- 스트림 코드를 가장한 반복적 코드다.
- 스트림 API의 이점을 살리지 못하여, 같은 기능의 반복적 코드보다 길고 읽기 어렵고 유지보수가 좋지 않다.
- 외부 상태인 freq를 수정할 때 문제가 생긴다.
  - 람다에서 상태를 수정하게 된다.

- forEach 연산은 종단 연산 중 기능이 가장 적고 가장 '덜' 스트림 답다.
  - 병렬화가 불가능하다.
  - **그러므로 forEach 연산은 스트림 계산 결과를 보고할 때만 사용하고, 계산하는 데는 쓰지 말자.**



# 수집기(collector)

스트림을 사용하려면 꼭 배워야 하는 새로운 개념이다.

수집기를 사용하면 스트림의 원소를 손쉽게 컬렉션으로 모을 수 있다.

수집기는 총 세가지가 있다.

- toList()

- toSet()

- toCollection(collectionFactory)

  

## 빈도표에서 가장 흔한 단어 10개를 뽑아내기

```java
List<String> topTen = freq.keySet().stream()
  .sorted(comparing(freq::get).reversed())
  .limit(10)
  .collect(toList());
```



## toMap(keyMapper, valueMapper)

```java
public class Main {
     public static void main(String[] args) throws FileNotFoundException {
        System.out.println(Operation.stringToEnum.get("+"));
     }
}

 enum Operation {
    PLUS("+", (x, y) -> x + y),
    MINUS("-", (x, y) -> x - y),
    TIMES("*", (x, y) -> x * y),
    DIVIDE("/", (x, y) -> x / y);

   //추가
    public static final Map<String, Operation> stringToEnum =
            Stream.of(values()).collect(
                    toMap(Object::toString, o -> o)
            );

    private final String symbol;
    private final DoubleBinaryOperator op;

    Operation(String symbol, DoubleBinaryOperator op) {
        this.symbol = symbol;
        this.op = op;
    }

     @Override
     public String toString() {
         return symbol;
     }

     public double apply(double x, double y) {
        return op.applyAsDouble(x, y);
    }
}
```



가장 간단한 맵 수집기는 toMap(keyMapper, valueMapper)로 , 스트림 원소를 키에 매핑하는 함수와 값에 매핑하는 함수를 인수로 받는다.

**이 수집기는 열거 타입 상수의 문자열 표현을 열거 타입 자체에 매핑하는 fromString을 구현하는데 사용했다.**



## 인자를 3개 받는 toMap

인수 3개 받는 toMap은 **어떤 키와 그 키에 연관된 원소들 중 하나를 골라 연관 짓는 맵을 만들 때** 유용하다. 

예컨대 다양한 가수와 앨범들을 담은 스트림을 가지고와, 가수와 가수의 베스트앨범을 연관 짓고 싶다고 해보자.

```java
public class Main {
    public static void main(String[] args) throws FileNotFoundException {
        List<Album> albums = List.of(
                new Album(new Artist("SG워너비"), "Timeless",5),
                new Album(new Artist("SG워너비"),"라라라", 10),
                new Album(new Artist("빅마마"),"체념", 15)
        );

        final Map<Artist, Album> collect = albums.stream().collect(
                toMap(Album::getArtist, album -> album, BinaryOperator.maxBy(comparing(Album::getSale)))
        );

        System.out.println(collect);
    }

}

class Artist {

    public String name;

    public Artist(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "가수 : " + name;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Artist artist = (Artist) o;
        return Objects.equals(name, artist.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name);
    }
}

class Album {
    private Artist artist;
    private String name;
    private int sale;

    public Artist getArtist() {
        return artist;
    }

    public int getSale() {
        return sale;
    }

    public Album(Artist artist, String name, int sale) {
        this.artist = artist;
        this.name = name;
        this.sale = sale;
    }

    @Override
    public String toString() {
        return "곡 제목 :" + name;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Album album = (Album) o;
        return artist.equals(album.artist);
    }

    @Override
    public int hashCode() {
        return Objects.hash(artist);
    }
}
```

결과

```
{가수 : SG워너비=곡 제목 :라라라, 가수 : 빅마마=곡 제목 :체념}
```

이 코드를 말로 풀어보자면 **"앨범 스트림을 맵으로 바꾸는데, 이 맵은 각 가수와 그 가수의 베스트 앨범을 짝지은 것이다"** 는 이야기로 풀이 된다.



## 마지막에 쓴 값을 취하는 수집기

인수가 3개인 toMap은 충돌이 나면 마지막 값을 취하는(last-write-wins) 수집기를 만들 때도 유용하다.

많은 스트림의 결과가 비결정적이다. 하지만 매핑 함수가 키 하나에 연결해준 값들이 모두 같을 때, 혹은 값이 다르더라도 모두 허용되는 값일 때 이렇게 동작하는 수집기가 필요하다.

```java
public class Main {
    public static void main(String[] args) throws FileNotFoundException {
        List<Book> books = List.of(
                new Book("자바의 정석", "한빛 소프트"),
                new Book("자바의 정석", "소프트")
        );

       Map<String, Book> resultBook = books.stream().collect(
                toMap(Book::getName, book -> book)
        );

       System.out.println(resultBook);
    }

}

class Book {
    private String name;
    private String publisher;

    public Book(String name, String publisher) {
        this.name = name;
        this.publisher = publisher;
    }

    public String getName() {
        return name;
    }
    
    @Override
    public String toString() {
        return "Book{" +
                "name='" + name + '\'' +
                ", publisher='" + publisher + '\'' +
                '}';
    }
}
```

위에 코드는 책의 이름을 Key로, 책을 Value로 변환하는 스트림이다.

코드를 돌려보면 아쉽게도 예외가 발생한다.

```
 java.lang.IllegalStateException: Duplicate key 자바의 정석 (attempted merging values Book{name='자바의 정석', publisher='한빛 소프트'} and Book{name='자바의 정석', publisher='소프트'})
```

이 오류는 중복된 키가 발생하기 때문에 오류가 난 것이다.

그럼 중복된 키가 생길 경우 어떻게 처리할지 3번째 인자로 전달해줄 수 있다.

```java
public class Main {
    public static void main(String[] args) throws FileNotFoundException {
        List<Book> books = List.of(
                new Book("자바의 정석", "한빛 소프트"),
                new Book("자바의 정석", "소프트")
        );

       Map<String, Book> resultBook = books.stream().collect(
                toMap(Book::getName, book -> book, (oldBook, newBook) -> newBook)
        );

       System.out.println(resultBook);
    }

}
```

위와 같이 작성하면, 새로운 값으로 세팅을 하게 된다.



## 인자를 4개받는 toMap

인자 4개를 받는 toMap의 4번째 인자는 맵 팩터리를 받는다.

이 인자로는 EnumMap이나 TreeMap처럼 원하는 특정 맵 구현체를 직접 지정할 수 있다.

```java
       Map<String, Book> resultBook = books.stream().collect(
                toMap(Book::getName, book -> book, (oldBook, newBook) -> newBook, HashMap::new)
        );
```



## Collectors.groupingBy()

이 메서드는 입력으로 분류 함수를 받고 출력으로는 원소들을 카테고리별로 모아 놓은 맵을 담은 수집기를 반환한다.

### groupingBy(classifier)

```java
public class Main {
    public static void main(String[] args){
        String[] word = new String[]{"abc", "cba", "bac", "bcd", "dcb", "cbd", "def", "edf", "fed"};

        Map<String, List<String>> result = Arrays.stream(word)
                .collect(groupingBy(s -> alphabetize(s)));

        System.out.println(result);
    }

    private static String alphabetize(String s) {
        char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
```

```
{bcd=[bcd, dcb, cbd], abc=[abc, cba, bac], def=[def, edf, fed]}
```



### groupingBy(classifier,downstream)

위 예제에서는 groupingBy의 결과값으로 `List` 를 반환한다. 하지만 다운스트림을 명시하면`List` 말고 다른 타입으로도 반환이 가 능하다.

```java
    public static void main(String[] args){
        String[] word = new String[]{"abc", "abc", "abc", "bcd", "dcb", "cbd", "def", "edf", "fed"};

         Map<String, Long> collect = Arrays.stream(word)
                .collect(groupingBy(String::toLowerCase, counting()));

        System.out.println(collect);
    }
```

```
{dcb=1, bcd=1, abc=3, fed=1, def=1, cbd=1, edf=1}
```



### groupingBy(classifier, mapFactory, downstream)

```java
public static void main(String[] args) {

        String[] word = new String[]{"abc","cba","bac","bcd","dcb","cbd","def","edf","fed"};

        Arrays.stream(word)
                .collect(Collectors.groupingBy(s -> alphabetize(s), TreeMap::new, Collectors.toCollection(LinkedList::new)));

    }

    private static String alphabetize(String s) {
        char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
```

이 메서드는 다운스트림 수집기에 더해 맵 팩터리를 지정하여 원하는 Map을 만들 수 있다.





## 다운스트림 수집기 전용 메서드

counting 메서드가 반환하는 수집기는 다운스트림 수집기 전용이다. Stream의 count 메서드를 직접 사용하여 같은 기능을 수행할 수 있으니 **collect(counting()) 형태로 사용할 일은 전혀 없다. **

Collections에는 이런 속성의 메서드가 16개나 더 있다. 그 중 9개는 이름이 

- summing
- averaging
- summarizing

으로 시작하며, 각각 int, long double 스트림용으로 하나씩 존재한다. 

그리고 다중 정의된 reducing 메서드들인

- filtering
- mapping
- flatMapping
- collectingAndThen

메서드가 있는데, 대부분 개발자는 이들의 존재를 모르고 있어도 상관 없다.

설계 관점에서 보면, 이 수집기들은 스트림 기능의 일부를 복제하여 다운스트림 수집기를 작은 스트림처럼 동작하게 한 것이다.



## joining(delimiter)

원소들 연결시 구분문자를 삽입한다. 

```java
public static void main(String[] args) {

  String[] word = new String[]{"abc", "cba", "bac", "bcd"};

  String result = Arrays.stream(word)
    .collect(joining("."));

  System.out.println(result);
}
```



```
abc.cba.bac.bcd
```



## joining(delimiter, prefix, suffix)

```java
public static void main(String[] args) {

    String[] word = new String[]{"abc", "cba", "bac", "bcd"};


    String result = Arrays.stream(word)
            .collect(Collectors.joining(",","[","]"));

    System.out.println(result);
}
```



```
[abc,cba,bac,bcd]
```

