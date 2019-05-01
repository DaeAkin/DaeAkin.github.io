# private 생성자나 enum 자료형은 싱글턴 패턴을 따르도록 설계 하자.

싱글턴은 객체를 하나만 만들 수 있는 클래스이다.

싱글턴은 보통 유일할 수 밖에 없는 시스템 컴포넌트를 나타낸다.

창 관리자(window manager)나 파일 시스템 같은 것들이 싱글턴이다.

**그런데 클래스를 싱글턴으로 만들면 클라이언트를 테스트하기가 어려워질 수가 있다.**

싱글턴이 어떤 인터페이스를 구현하는 것이 아니면 가짜 구현으로 대체할 수 없기 때문이다.

보통 사람들이 쓰는 싱글턴은 대체로 이렇다

```java
public class Rule3 {
    private static final Rule3 INSTANCE = new Rule3();
    
    public static Rule3 getInstance() {
        return INSTANCE;
    }
}
```

Rule3의 객체를 쓰고자한다면 Rule3.getInstance()만 써주면 

Rule3은 객체를 생성하지 않고 INSTANCE변수에 저장되어있는 Rule3을 

반환 해주므로 메모리 낭비 없이 항상 같은객체만 생성되게 된다.

(public으로 선언된 getInstance()는 정적 팩터리 메서드를 이용한 방법이다.)

하지만 주의할 것이 있는데 AccessibleObject, setAccessible 메소드의 도움을 받아

권한을 흭득한 클라이언트는 리플렉션(reflection) 기능을 통해 

또 다른 객체를 만들 수 있다는 것이다.

이런 종류의 공격을 방어하고 싶다면, 두 번째 객체를 생성하라는 요청을 받으면 예외를 

던지도록 수정해야한다.

(생성자를 private로 해주어도 호출이 되어버린다.)



[리플렉션 더 알아보기](http://stackoverflow.com/questions/4081227/singletom-pattern)



앞서 설명한 방법들로 구현한 싱글턴 클래스를 직렬화 가능(Serializable) 클래스로 만들려면

클래스 선언에 implements Serializable을 추가하는 것으로는 부족하다

싱글턴 특성을 유지하려면 모든 필드를 transient로 선언하고 readResolve 메소드를 추가해야 한다.

그렇지 않으면 serialize된 객체가 역직렬화(deserialize)될 때마다 새로운 객체가 생기게 된다.

이 문제를 막으려면 Rule3 클래스에 아래의 readResolve 메소드를 추가 해야 한다.

```java
private Object readResolve() {
    //동일한 Rule3객체가 반환되도록 하는 동시에, 가짜 Rule3객체는
    //쓰레기 수집기(garbage collector)가 처리하도록 만든다. 
    return INSTANCE;
}
```

JDK 1.5 부터는 싱글턴을 구현할 때 새로운 방법을 사용할 수 있다.

원소가 하나뿐인 enum 자료형을 정의하는 것 이다.

enum : 열거형 (JDK 1.5 이상 )

\- 클래스처럼 보이게 하는 상수

\- 서로 관련 있는 상수들을 모아 심볼릭한 명칭의 집합으로 정의한 것

\- Enum 클래스형 기반으로 한 클래스형 선언



```java
//Enum 싱글턴 
public enum Rule3 {
    INSTANCE;
    public void leaveTheBuilding() {...}
}
```

이 접근법은 기능적으로 public 필드를 사용하는 구현법과 동등하다.

한가지 차이는 좀 더 간결한다는 것과, 직렬화가 자동으로 처리된다는 것이다.

직렬화가 아무리 복잡하게 이루어져도 여러 객체가 생길 일이 없으며

리플렉션을 통한 공격에도 안전하다. 

아직 널리 사용되는 접근법은 아니지만, 원소가 하나뿐인 enum 자료형이야 말로

**싱글턴을** **구현하는** **가장** **좋은** **방법이다**