---
layout: post
title: 계승을 위한 설계와 문서를 갖추거나, 그럴 수 없다면 계승을 금지하라
categories: [Design Pattern]
comments: true
---
# 규칙 17 계승을 위한 설계와 문서를 갖추거나, 그럴 수 없다면 계승을 금지하라

이 포스팅의 관련된 코드는 [여기에](https://github.com/DaeAkin/effectivejava/tree/master/src/main/java/com/donghyeon/effectivejava/rule17) 있습니다.

계승을 위한 설계와 문서를 갖춘다는 것은 어떤 의미일까요?

우선, 메소드를 재정의하면 무슨 일이 생기는지 정확하게 문서로 남겨야 합니다. 다시 말해, 재정의 가능 메소드를 내부적으로 어떻게 사용하는지 반드시 문서에 남기라는 것입니다.

public 또는 protected로 선언된 모든 메소드와 생성자에 대해, 어떤 재정의 가능 메소드를 어떤 순서로 호출하는지, 그리고 호출 결과가 추후 어떤 영향을 미치는지 문서로 남겨야 합니다. 

> 재정의가 가능하다는 것은 public 또는 protected로 선언된 비-final 메소드라는 뜻입니다.

다시 말하면 상위 클래스에서 상속 가능한 메소드들의 스펙을 명시애햐 합니다.

관습적으로, 재정의 가능 메소드를 어떤 식으로 호출하는지 메소드 주석문 마지막에 명시합니다.주석은. "이 구현은"이라는 문구로 시작합니다. 릴리즈에 따라서 달라질 수 있다는 뜻으로 하는 말은 아니고, 메소드 내부 동작 원리에 관한 주석입니다.

## AbstractCollection

예를들어 AbstractCollection은 Collection 인터페이스의 뼈대를 구현한 추상클래스 인데,

AbstractCollection.remove() 함수를 보면 이런 주석이 달려 있습니다.

```java
 /**
     * {@inheritDoc}
     *
     * <p>This implementation iterates over the collection looking for the
     * specified element.  If it finds the element, it removes the element
     * from the collection using the iterator's remove method.
     *
     * <p>Note that this implementation throws an
     * <tt>UnsupportedOperationException</tt> if the iterator returned by this
     * collection's iterator method does not implement the <tt>remove</tt>
     * method and this collection contains the specified object.
     *
     * @throws UnsupportedOperationException {@inheritDoc}
     * @throws ClassCastException            {@inheritDoc}
     * @throws NullPointerException          {@inheritDoc}
     */
    public boolean remove(Object o) {
        Iterator<E> it = iterator();
        if (o==null) {
            while (it.hasNext()) {
                if (it.next()==null) {
                    it.remove();
                    return true;
                }
            }
        } else {
            while (it.hasNext()) {
                if (o.equals(it.next())) {
                    it.remove();
                    return true;
                }
            }
        }
        return false;
    }
```

짧은 영어지식으로 해석을 해보자면, 삭제할 요소를 찾으면 iterator의 remove 메소드를 이용해서 요소를 삭제합니다. 
이 컬렉션의 iterator 메서드가 반환하는 반복자가 remove 메서드를 구현하고 있지 않을 경우에는 UnsupportedOprationException 예외가 발행할 수 있습니다. 라고 적혀 있습니다.

이 문서를 보면 iterator 메서드를 재정의하면 remove가 영향을 받는다는 사실을 확실히 알 수 있습니다.

이전 규칙인 규칙 16번에서는 HashSet의 하위 클래스를 만드는 프로그래머는 add를 재정의하면 addAll에 무슨 일이 생길지 알 수 없었습니다.

그러나 이렇게 하면 "좋은 API 문서는 메서드가 하는 일이 무엇인지 명시해야지, 그 일을 어떻게 하는지 명시해서는 안된다"는 원칙을 깨는 것이 되는데, 이는 계승이 캡슐화 원칙을 침해하기 때문에 발생하는 결과입니다. 하위 클래스가 안전한 클래스가 되려면, 문서에 반드시 구현 세부사항을 기술해야 합니다. 

그러나 문서만 제대로 썼다고 계승에 적합한 설계가 되는건 아닙니다. 너무 애쓰지 않고도 효율적인 하위 클래스를 작성할 수 있도록 하려면, 클래스 내부 동작에 개입할 수 있는 훅(hooks)을 신중하게 골라 protected 메소드 형태로 제공해야 합니다.

## AbstractList 클래스

AbstractList 클래스 중에 removeRange() 라는 메소드가 있습니다.

```java
protected void removeRange(int fromIndex, int toIndex) {
    ListIterator<E> it = listIterator(fromIndex);
    for (int i=0, n=toIndex-fromIndex; i<n; i++) {
        it.next();
        it.remove();
    }
}
```

이 메소드는 AbstractList.clear()가 내부적으로 사용하는 메소드인데,  removeRange() 메소드를 재정의하여 리스트 구현체의 내부 구현을 활용하면 clear 연산 성능을 많이 올릴 수 있습니다.

```java
public void clear() {
    removeRange(0, size());
}
```

이 메소드는 List 구현체 사용자에게 흥미로울 것 없는 메소드 입니다. 하위 클래스에서 리스트에 대한 clear 메소드 실행 성능을 개선하기 쉽게 하려고 제공하는 메소드 입니다.

removeRange 메소드가 없다면, 하위 클래스에서는 clear 메소드 성능이 리스트 길이의 제곱에 비례하여 나빠진다는 점을 감안하고 사용하거나, subList 메커니즘을 처음부터 끝까지 다시 작성해야 할 것입니다. 절대로 쉬운 작업이 아닙니다.

그렇다면 계승을 고려하여 클래스를 설계할 때 protected로 선언할 멤버는 어떻게 정할까요? 불행히도, 이 문제를 푸는 마법은 없습니다. 그냥 열심히 생각하고 신중하게 고른 다음, 실제로 하위 클래스를 만들어보면서 테스트하는 것이 최선입니다. 

반대로 하위 클래스를 몇개 만들어 봐도 사용할 일이 없는 protected 멤버는 다시 private로 선언해야 합니다.

자바 이펙티브 저자에 의하면 계승에 맞게 클래스를 테스트하는 데는 하위 클래스 세 개면 대체로 충분 했다고 합니다.

## 계승의 제약사항

계승을 허용하려면 반드시 따라야 할 제약사항이 몇 가지 더 있습니다. 그 중 첫번째는, **생성자는 직접적이건 간접적이건 재정의 가 가능한 메소드를 호출해서는 안됩니다.** 이 규칙을 위반하면 프로그램에서는 오류가 발생합니다.

상위 클래스 생성자는 하위 클래스 생성자보다 먼저 실행되므로, 하위 클래스에서 재정의한 메서드는 하위 클래스생성자가 실행되기 전에 호출 됩니다. 재정의한 메소드가 하위 클래스 생성자가 초기화한 결과에 의존할 경우, 그 메소드는 원하는 대로 실행되지 않습니다.

### Super

```java
public class Super {
    public Super() {
        System.out.println("Super()");
        overrideMe();
    }
    public void overrideMe() {
        System.out.println("Super overrideMe()");
    }
}
```

### Sub

```java
public class Sub extends Super{
    
    private final LocalDateTime localDateTime;

    public Sub() {
        System.out.println("Sub()");
        localDateTime = LocalDateTime.now();
    }

    @Override
    public void overrideMe() {
        System.out.println("sub overrideMe()");
        System.out.println(localDateTime.toLocalDate());
    }
}
```

### 실행

```java
public class Rule17 {
    public static void main(String[] args) {
        Sub sub = new Sub();
        sub.overrideMe();
    }
}
```



실행을 하게되면 localDateTime이 초기화가 되지 않아 NullPointerException이 일어납니다.

```
> Task :Rule17.main() FAILED
Super()
sub overrideMe()
Exception in thread "main" java.lang.NullPointerException
	at com.donghyeon.effectivejava.rule17.Sub.overrideMe(Sub.java:17)
```



## 마무리 

이렇게 계승에 대한 제약조건과 고려해야할 사항들이 많습니다. 이런 문제를 풀는 가장 좋은 방법은 계승에 맞도록 설계하고 문서화하지 않은 클래스에 대한 하위 클래슨느 만들지 않는 것입니다. 하위 클래스 생성을 금지하는 방법에는 두가지가 있습니다.

가장 쉬운 방법은 클래스를 final로 선언하는 거나, 모든 생성자를 private나 package-private로 선언하고 생성자대신 public 정적 팩터리 메소드를 추가하는 것입니다.