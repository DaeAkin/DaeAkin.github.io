---
layout: post
title: 네이티브 메서드는 신중히 사용하자
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---



# 자바 네이티브 인터페이스 (JNI)

자바 네이티브 인터페이스는 자바 프로그램이 네이티브 메서드를 호출하는 기술이다.

네이티브 메서드란 C나 C++ 같은 네이티브 프로그래밍 언어로 작성한 메서드를 말한다.

```java
public class Sample1 {

    // --- Native methods
    public native int intMethod(int n);
    public native boolean booleanMethod(boolean bool);
    public native String stringMethod(String text);
    public native int intArrayMethod(int[] intArray);


    // --- Main method to test our native library
    public static void main(String[] args) {
        System.loadLibrary("Sample1");
        Sample1 sample = new Sample1();
        int square = sample.intMethod(5);
        boolean bool = sample.booleanMethod(true);
        String text = sample.stringMethod("java");
        int sum = sample.intArrayMethod(new int[] {1, 1, 2, 3, 5, 8, 13});

        System.out.println("intMethod: " + square);
        System.out.println("booleanMethod: " + bool);
        System.out.println("stringMethod: " + text);
        System.out.println("intArrayMethod: " + sum);
    }
}
```

 `native` 라는 예약어를 이용해 사용한다. 



# 네이티브 메서드 쓰임새

- 레지스트리 같은 플랫폼 특화 기능을 사용할 때
- 네이티브 코드로 작성된 기존 라이브러리를 사용할 때
- 성능 개선을 목적으로 성능에 결정적인 영향을 주는 영역만 따로 네이티브 언어로 작성할 때

플랫폼 특화 기능을 활용하려면 네이티브 메서드를 사용해야 한다.

하지만 자바가 성숙해가면서 OS 같은 하부 플랫폼의 기능들을 점차 흡수하고 있다.

그래서 네이티브 메서드를 사용할 필요가 계속 줄고 있다.

예를 들어 자바 9에서는 process API를 추가해 OS 프로세스에 접근할 수 있게 되었다.

**대체할 만한 자바 라이브러리가 없는 네이티브 라이브러리를 사용해야 할 때 네이티브 메서드를 써야 한다.**



# 네이티브 메서드는 안쓰는게 좋다

**성능을 개선할 목적으로 네이티브 메서드를 사용하는 것은 권장하지 않는다.**

지금 현재 자바는 초기 시절보다 JVM이 그동안 많이 발전했다.

대부분 작업에서 지금의 자바는 다른 플랫폼에 견줄만한 성능을 보인다.

예를 들어 jdk1.1의 java.math의 BigInteger는 C로 작성한 고성능 라이브러리에 의지했지만, 순수 자바로 다시 구현되면서 네이티브 구현 보다 더 빨라졌다.

그러나 BigInteger는 자바 8에서 큰수의 곱셈 성능을 개선한 것을 제외하고 더 이상의 커다란 변화는 없었다.

**하지만, 네이티브 라이브러리는 GNU 다중 정밀 연산 라이브러리(GMP)를 필두로 개선 작업이 계속 되어왔다.**

그러므로 정말로 고성능 다중 정밀 연산이 필요하다면 네이티브 메서드를 통해 GMP를 사용하는걸 고려해볼 순 있다.

하지만 네이티브 메서드 심각한 단점이 있다.

- 네이티브 언어가 안전하지 않으므로 네이티브 메서드를 사용하는 애플리케이션도 메모리 훼손 오류부터 더 이상 안전하지 않음
- 네이티브 언어는 자바보다 플랫폼을 많이 타서 이식성도 낮고 디버깅도 어렵다.
- 가비지 컬렉터가 네이티브 메모리는 자동 회수하지 못하고, 심지어 추적조차 할 수 없다.
- 자바 코드와 네이티브 코드의 경계를 넘나들 때마다 비용이 추가 된다.
- 네이티브 메서드와 자바 코드 사이의 **접착 코드(glue code)** 를 작성해야 하는데, 귀찮은 작업이기도 하고, 가독성도 떨어진다.

# 정리

네이티브 메서드가 성능을 개선해주는 일은 많지 않으므로 사용하려거든 한번 더 생각하자.

저수준 자원이나 네이티브 라이브러리를 사용해야 어쩔 수 없이 사용해야 한다면, 최소한만 사용하고 철저히 테스트하자.
