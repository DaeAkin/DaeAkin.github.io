---
layout: post
title: 멤버 클래스는 되도록 static으로 만들자
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---

## 중첩 클래스

중첩 클래스란 다른 클래스 안에 정의된 클래스를 말한다. 중첩 클래스는 자신을 감싼 바깥 클래스에서만 쓰여야 하며, 그 외의 쓰임새가 있다면 톱레벨 클래스로 만들어야 한다. 

### 중첩 클래스의 종류

- **정적 멤버 클래스** -> 애만 nested 클래스다.
- **(비정적) 멤버 클래스**
- **익명 클래스**
- **지역 클래스**

#### 정적 멤버 클래스

정적 멤버 클래스는 다른 클래스 안에 선언되고, 바깥 클래스의 private 멤버에도 접근할 수 있다는 점만 제외하고는 일반 클래스와 똑같다.

```java
public class OuterClass {

  private int x = 10;

  private static class InnerClass {
    void test() {
      OuterClass outerClass = new OuterClass();
      //바깥 클래스에 private 멤버에 접근하는 중
      outerClass.x = 100;
    }
  }
}
```

정적 멤버 클래스는 다른 정적 멤버와 똑같은 규칙을 적용받는다.(위에 예제에서는 OuterClass의 x 변수가 되겠다.)  private으로 선언하면 바깥 클래스에서만 접근할 수 있고, 다른 클래스에서는 절대 접근할 수 없다.

#### 비정적 멤버 클래스

비정적 멤버 클래스의 인스턴스와 바깥 인스턴스의 사이의 관계는 멤버 클래스가 인스턴스화될 때 확립되며, 더 이상 변경할 수 없다.

```java
public class TestClass {

    void x() {
    }
    
     class NestedClass {
        
        void x() {
            TestClass.this.x();
        }
        
    }
}
```

이렇게 되면 비정적 멤버 클래스의 인스턴스는 바깥 클래스인 TestClass와 암묵적으로 연결되어, 비정적 멤버 클래스의 인스턴스 메서드에서 정규화된 this를 사용해 바깥 인스턴스의 메서드를 호출하거나 바깥 인스턴스의 참조를 가져올 수 있다.

```java
public class Main {

     public static void main(String[] args) {
         //관계가 확립되었음.
         new TestClass().new NestedClass();
     }
}
```

비정적 멤버클래스의 인스턴스와 바깥 인스턴스 사이의 관계는 위와 같이 비정적 멤버 클래스가 인스턴스화 될 때 확립되며 더 이상 변경할 수 없다. 

비정적 멤버 클래스는 [어댑터](https://donghyeon.dev/design%20pattern/2020/02/11/%EC%96%B4%EB%8C%91%ED%84%B0-%ED%8C%A8%ED%84%B4/) 를 정의할 때 자주 쓰인다. 즉 다른 타입의 인스턴스를 리턴할 때를 말한다.

```java
public class MySet<E> extends AbstractSet<E> {
  
  @Override public Iterator<E> iterator() {
    return new MyIterator();
  }
  // 어댑터
  private class MyIterator implements Iterator<E> {
    
  }
}
```

이렇게 사용하면 바깥 인스턴스로의 숨은 외부 참조를 갖게 된다. 이 참조를 저장하려면 시간과 공간이 소비된다. 더 심각한 문제는 가비지 컬렉션이 바깥 클래스의 인스턴스를 수거하지 못는 **<u>메모리 누수</u>**가 생길 수 있다는 점이다. 

**그러므로 멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 static을 붙여서 정적 멤버 클래스로 만들자.**

```java
public class MySet<E> extends AbstractSet<E> {
  
  @Override public Iterator<E> iterator() {
    return new MyIterator();
  }
  // 어댑터
  private static class MyIterator implements Iterator<E> {
    
  }
}
```

비슷하게, Set과 List 같은 다른 컬렉션 인터페이스 구현들도 자신의 반복자를 구현할 때 비정적 멤버 클래스를 주로 사용한다.

**Collection.unmodifiableCollection()** 에서는 정적 멤버 클래스를 이용하고 있다.

```java
		public static <T> Collection<T> unmodifiableCollection(Collection<? extends T> c) {
        return new UnmodifiableCollection<>(c);
    }

    static class UnmodifiableCollection<E> implements Collection<E>, Serializable {

    }
```

또한 멤버 클래스 역시 공개 API가 되니, 혹시라도 향후 릴리스에서  static을 붙이면 하위 호환성이 깨진다.

#### 익명 클래스

- 선언한 지점에서만 인스턴스를 만들 수 있다.

- 비정적 문맥에서 사용될 때만 바깥 클래스의 인스턴스를 참조할 수 있다.

  ```java
  public class TestClass {
      Integer intInstance = 10;
  
      void doX() {
          new SInterface() {
              @Override
              public void doSometing() {
                  //바깥 인스턴스 참조
                  System.out.println(intInstance);
              }
          };
      }
  }
  
  
  interface SInterface {
      void doSometing();
  }
  ```

  - 정적 문맥에서라도 상수 변수 이외에 정적 멤버는 가질수 없다.

    ```java
    public class TestClass {
        // 익명클래스에서 사용 불가
        Integer intInstance = 10;
    
        static void doX() {
            new SInterface() {
                static final int x = 0;
    //            static int y = 0; // 컴파일에러
                @Override
                public void doSometing() {
                }
            };
        }
    }
    
    interface SInterface {
        void doSometing();
    }
    ```

  - 상수표현을 위해 초기화된 final 기본 타입과 문자열 필드만 가질 수 있다.

- instanceof 검사나 클래스의 이름이 필요한 작업은 수행할 수 없다.

- 여러 인터페이스를 구현할 수 없고 인터페이스를 구현하는 동시에 다른 클래스를 상속할 수도 없다.

- 익명 클래스를 사용하는 클라이언트는 그 익명 클래스가 상위 타입에서 상속한 멤버 외에는 호출할 수 없다.

- 익명 클래스는 표현식 중간에 등장하므로 짧지 않으면 가독성이 떨어진다.

- 자바가 람다를 지원하기 전에는 즉석에서 작은 함수 객체나 처리 객체를 만드는 데 익명 클래스를 주로 사용 했다.

  - 이제는 람다를 많이 사용한다.

- 익명클래스는 정적 팩터리 메서드를 구현할 때 자주 쓰인다.

  ```java
  public interface IntListHelper {
      
      static List<Integer> intArrayAsList(int[] a) {
          return new AbstractList<Integer>() {
              @Override
              public Integer get(int index) {
                  return a[index];
              }
  
              @Override
              public int size() {
                  return a.length;
              }
          };
      }
  }
  ```

#### 지역 클래스

- 가장 드물게 사용된다. 
- 지역변수를 선언할 수 있는 곳이면 어디서든 선언 가능하고, 유효 범위도 지역변수와 같다.
- 익명클래스처럼 비정적 문맥에서 사용될 때만 바깥 인스턴스를 참조할 수 있다.
- 정적 멤버는 가질 수 없으며 가독성을 위해 짧게 작성해야 한다.

```java
public class TestClass {

    void x() {
        class LocalClass {
            void doPrint() {
                System.out.println("LocalClass");
            }
        }
    }
}
```

## 정리

- 메서드 밖에서도 사용해야 하거나 메서드 안에 정의하기에 너무 길다면 -> **멤버 클래스**
  - 멤버 클래스의 인스턴스가 바깥 인스턴스를 참조하면 -> **비정적**
  - 멤버 클래스의 인스턴스가 바깥 인스턴스를 참조안하면 -> **정적**
- 중첩 클래스가 한 메서드 안에서만 쓰이면서 그 인스턴스를 생성하는 지점이 단 한곳이고 해당 타입으로 쓰기에 적잡한 클래스나 인터페이스가 있다면 -> **익명 클래스**
  - 그렇지 않다면? -> **지역 클래스**

## 참고할 만한 자료

https://stackoverflow.com/questions/1953530/why-does-java-prohibit-static-fields-in-inner-classes/1954119#1954119%EB%A5%BC