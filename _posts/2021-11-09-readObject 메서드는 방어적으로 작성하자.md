---

layout: post
title: readObject 메서드는 방어적으로 작성하자
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---

[이 내용](https://donghyeon.dev/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C%EC%9E%90%EB%B0%94/2021/07/03/%EC%A0%81%EC%8B%9C%EC%97%90-%EB%B0%A9%EC%96%B4%EC%A0%81-%EB%B3%B5%EC%82%AC%EB%B3%B8%EC%9D%84-%EB%A7%8C%EB%93%A4%EC%9E%90/)에서는 불변인 날짜 범위 클래스를 만드는데 가변인 Date 필드를 이용했다.

그래서 불변식을 지키고 불변을 유지하기 위해 생성자와 접근자에서 Date 객체를 방어적으로 복사하느라 상당히 길어졌다.

**방어적 복사를 사용하는 불변클래스**

```java
public final class Period {
  private final Date start;
  private final Date end;
  
  public Period(Date start, Date end) {
    this.start = new Date(start.getTime());
    this.end = new Date(end.getTime());
    if (this.start.compareTo(this.end) > 0)
      throw new IllegalArgumentException(start + "after" + end); 
  }
  
  public Date start() {
    return new Date(start.getTime()); 
  }

  public Date end() {
    return new Date(end.getTime());
  }
}
```

이 클래스를 직렬화하기로 결정했다고 해보자.

Period 객체의 물리적 표현이 논리적 표현과 부합하므로 기본 직렬화 형태를 사용해도 나쁘지 않다.

그러니 이 클래스 선언에 `implements Serializable` 을 추가하는 것으로 모든 일을 끝낼 수 있을 것 같다.

하지만 이렇게 해서는 이 클래스의 주요한 불식을 더는 보장하지 못하게 된다.

문제는 readObject 메서드가 실질적으로 또 다른 public 생성자이기 때문이다.

따라서 다른 생성자와 똑같은 수준으로 주의를 기울여야 한다.

보통의 생성자처럼 readObject 메서드에서도 인수가 유효한지 검사해야 하고

필요하다면 매개변수를 방어적으로 박사해야 한다.

readObject가 이 작업을 제대로 수행하지 못하면 공격자는 아주 손쉽게 해당 클래스의 불변식을 깨뜨릴 수 있다.

쉽게 말해, readObject는 매개변수로 바이트 스트림을 받는 생성자라고 할 수 있다.

보통의 경우 바이트 스트림은 정상적으로 생성된 인스턴스를 직렬화해 만들어진다.

하지만 불변식을 깨뜨릴 의도로 임의 생성한 바이트 스트림을 건네면 문제가 생긴다.

정상적인 생성자로는 만들어낼 수 없는 객체를 생성해낼수 있기 때문이다.

# 허용되지 않은 Period 인스턴스를 생성할 수 있다

단순히 Period 클래스 선언에 `implments Serializable` 만 추가했다고 해보자.

그러면 다음의 코드가 수행해 **<u>종료 시각이 시작 시각보다 앞서는 Period 인스턴스를 만들 수 있다.</u>**

**허용되지 않는 Period 인스턴스를 생성할 수 있다.**

```java
public class BogusPeriod {
	//진짜 Period 인스턴스에서는 만들어질 수 없는 바이트 스트림
	private static final byte[] serializedForm = new byte[] { (byte) 0xac,
			(byte) 0xed, 0x00, 0x05, 0x73, 0x72, 0x00, 0x06, 0x50, 0x65, 0x72,
			0x69, 0x6f, 0x64, 0x40, 0x7e, (byte) 0xf8, 0x2b, 0x4f, 0x46,
			(byte) 0xc0, (byte) 0xf4, 0x02, 0x00, 0x02, 0x4c, 0x00, 0x03, 0x65,
			0x6e, 0x64, 0x74, 0x00, 0x10, 0x4c, 0x6a, 0x61, 0x76, 0x61, 0x2f,
			0x75, 0x74, 0x69, 0x6c, 0x2f, 0x44, 0x61, 0x74, 0x65, 0x3b, 0x4c,
			0x00, 0x05, 0x73, 0x74, 0x61, 0x72, 0x74, 0x71, 0x00, 0x7e, 0x00,
			0x01, 0x78, 0x70, 0x73, 0x72, 0x00, 0x0e, 0x6a, 0x61, 0x76, 0x61,
			0x2e, 0x75, 0x74, 0x69, 0x6c, 0x2e, 0x44, 0x61, 0x74, 0x65, 0x68,
			0x6a, (byte) 0x81, 0x01, 0x4b, 0x59, 0x74, 0x19, 0x03, 0x00, 0x00,
			0x78, 0x70, 0x77, 0x08, 0x00, 0x00, 0x00, 0x66, (byte) 0xdf, 0x6e,
			0x1e, 0x00, 0x78, 0x73, 0x71, 0x00, 0x7e, 0x00, 0x03, 0x77, 0x08,
			0x00, 0x00, 0x00, (byte) 0xd5, 0x17, 0x69, 0x22, 0x00, 0x78 };

	public static void main(String[] args) {
		Period p = (Period) deserialize(serializedForm);
		System.out.println(p);
	}

	// 주어진 직렬화 형태(바이트 스트림)로부터 객체를 만들어 반환한다.
	private static Object deserialize(byte[] sf) {
		try {
			InputStream is = new ByteArrayInputStream(sf);
			ObjectInputStream ois = new ObjectInputStream(is);
			return ois.readObject();
		} catch (Exception e) {
			throw new IllegalArgumentException(e);
		}
	}
}
```

이 코드를 실행하면 **Fri Jan 01 12:00:00 PST 1999 - Sun Jan 01 12:00:00 PST 1984** 를 출력한다.

보다시피 Period를 직렬화할 수 있도록 선언한 것만으 클래스의 불변식을 깨뜨리는 객체를 만들 수 있게 된 것이다.

이 문제를 고치려면 Period의 readObject 메서드가 defaultReadObject를 호출한 다음 역직렬화된 객체가 유효한지 검사해야 한다.

이 유효성 검사에 실패하면 InvalidObjectException을 던지게 하여 잘못된 역직렬화가 일어나는 것을 막을 수 있다.

**유효성 검사를 수행하는 readObject 메서드 - 아직 부족하다.**

```java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
  
  //불변식을 만족하는지 검사한다.
  if (start.compareTo(end) > 0) 
    throw new InvalidObjectException(start +" after " + end);
}
```

이상의 작업으로 공격자가 허용되지 않은 Period 인스턴스를 생성하는 일을 막을 수 있지만, 아직도 미묘한 문제 하나가 숨어 있다.

**정상 Period 인스턴스에서 시작된 바이트 스트림 끝에 private Date 필드로의 참조를 추가하면 가변 Period 인스턴스를 만들어 낼 수 있다.**

공격자는 ObjectInputStream에서 Period 인스턴스를 읽은 후 스트림 끝에 추가된 악의적인 객체 참조를 읽어 Period 객체의 내부 정보를 얻을 수 있다.

이제 이 참조로 얻은 Date 인스턴스들은 수정할 수 있으니, Period 인스턴스는 더는 불변이 아니게 되는 것이다.

다음은 공격이 어떻게 이뤄지는지 보여주는 예다.

**가변 공격 예시**

```java
public class MutablePeriod {
	// Period 인스턴스
	public final Period period;

	// 시작 시각 필드
	public final Date start;

	// period's end field, to which we shouldn't have access
	public final Date end;

	public MutablePeriod() {
		try {
			ByteArrayOutputStream bos = new ByteArrayOutputStream();
			ObjectOutputStream out = new ObjectOutputStream(bos);

			// Serialize a valid Period instance
			out.writeObject(new Period(new Date(), new Date()));

			/*
			 * Append rogue "previous object refs" for internal Date fields in
			 * Period. For details, see "Java Object Serialization
			 * Specification," Section 6.4.
			 */
			byte[] ref = { 0x71, 0, 0x7e, 0, 5 }; // Ref #5
			bos.write(ref); // The start field
			ref[4] = 4; // Ref # 4
			bos.write(ref); // The end field

			// Deserialize Period and "stolen" Date references
			ObjectInputStream in = new ObjectInputStream(
					new ByteArrayInputStream(bos.toByteArray()));
			period = (Period) in.readObject();
			start = (Date) in.readObject();
			end = (Date) in.readObject();
		} catch (Exception e) {
			throw new AssertionError(e);
		}
	}
}
```

다음 코드를 실행하면 이 공격이 실제로 이뤄지는 모습을 확인할 수 있다.

```javascript
	public static void main(String[] args) {
		MutablePeriod mp = new MutablePeriod();
		Period p = mp.period;
		Date pEnd = mp.end;

		// 시간을 되돌리자
		pEnd.setYear(78);
		System.out.println(p);

		// 60년대로 회귀
		pEnd.setYear(69);
		System.out.println(p);
	}
```

다음 결과가 출력 된다.

```
Wed Nov 22 00:21:29 PST 2017 - Wed Nov 22 00:21:29 PST 1978
Wed Nov 22 00:21:29 PST 2017 - Wed Nov 22 00:21:29 PST 1969
```

이 예에서 Period 인스턴스는 불변식으로 유지한 채 생성됐지만, 의도적으로 내부의 값을 수정할 수 있었다.

이처럼 변경할 수 있는 Period 인스턴스를 흭득한 공격자는 이 인스턴스가 불변이라고 가정하는 클래스에 넘겨 엄청만 보안 문제를 일으킬 수 있다.

이것이 너무 극단적인 예가 아닌 것이, 실제로도 보안 문제를 String이 불변이라는 사실에 기댄 클래스들이 존재하기 때문이다.

이 문제의 근원은 Period의 readObject 메서드가 방어적 복사를 충분히 하지 않은 데 있다.

# 역직렬화 할때도 방어적 복사를 해야한다

**객체를 역직렬화할 때는 클라이언트가 소유해서는 안되는 객체 참조를 갖는 필드를 모두 반드시 방어적으로 복사해야 한다.**

따라서 readObject에서는 불변 클래스 안의 모든 private 가변 요소를 방어적으로 복사해야 한다.

다음의 readObject 메서드라면 Period 불변식과 불변 성질을 지켜내기 충분하다.

**방어적 복사와 유효성 검사를 수행하는 readObject 메서드**

```java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
  s.defaultReadObject();
  
  //가변 요소들을 방어적으로 복사한다.
  start = new Date(start.getTime());
  end = new Date(end.getTime());
  
  // 불변식을 만족하는지 검사한다.
  if (start.compareTo(end) > 0)
    throw new InvalidObjectException(start +" after " + end);
}
```

방어적 복사를 유효성 검사보다 앞서 수행하며, Date의 clone 메서드는 사용하지 않았음에 주목하자.

두 조치 모두 Period를 공격으로부터 보호하는 데 필요하다.

**또한 final 필드는 방어적 복사가 불가능하니 주의하자.**

그래서 이 readObject 메서드를 사용하려면 start와 end 필드에서 final 한정자를 제거해야 한다.

아쉬운 일이지만 앞서 살펴본 공격 위험에 노출되는 것보다야 낫다.

start와 end에서 final 한정자를 제거하고 이 새로운 readObject를 적용하면 MutablePeriod 클래스는 정상작동한다.



# 기본 readObject 메서드를 써도 좋을지 판단하는 방법

transient 필드를 제외한 모든 필드의 값을 매개변수로 받아 유효성 검사 없이 필드에 대입하는 public 생성자를 추가해도 괜찮은가?

아니라면 커스텀 readObject 메서드를 만들어 생성자에서 수행했어야 할 모든 유효성 검사와 방어적 복사를 수행해야 한다.

또는 직렬화 프록시 패턴을 사용하는 방법도 있다.

final이 아닌 직렬화 가능 클래스라면 readObject와 생성자의 공통점이 하나 더 있다.

마치 생성자처럼 readObject 메서드도 재정의 가능 메서드를 직접적으로든 간접적으로든 호출해서는 안 된다.

해당 메서드가 재정의되면, 하위 클래스의 상태가 완전히 역직렬화되기 전에 하위 클래스에서 재정의된 메서드가 실행되서, 오작동으로 이어진다.



# 정리

- private이어야 하는 객체 참조 필드는 각 필드가 가리키는 객체를 방어적으로 복사하자. 불변 클래스 내의 가변 요소가 여기 속한다.
- 모든 불변식을 검사하여 어긋나는게 발견되면 `InvalidObjectException` 을 던진다. 방어적 복사 다음에는 반드시 불변식 검사가 뒤따라야 한다.
- 역직렬화 후 객체 그래프 전체의 유효성을 검사해야 한다면 `ObjectInputValidation` 인터페이스를 사용하자.
- 직접적이든 간접적이든, 재정의할 수 있는 메서드는 호출하지 말자.
