---
layout: post
title: ordinal 인덱싱 대신 EnumMap을 사용하자
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---

# ordinal 메서드

배열이나 리스트에서 원소를 꺼낼 때 Enum의 `ordinal` 메서드로 인덱스를 얻는 코드가 있다.

**식물을 간단히 나타내는 클래스**

```java
class Plant {
    enum LifeCycle {ANNUAL, PERENNIAL, BIENNIAL}

    final String name;
    final LifeCycle lifeCycle;

    public Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }

    @Override
    public String toString() {
        return name;
    }
}
```

```java
 //비검사 형변환 경고
Set<Plant>[] plantsByLifeCycle =
        (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];

for(int i = 0; i < plantsByLifeCycle.length; i++)
    plantsByLifeCycle[i] = new HashSet<>();

 Plant[] garden = new Plant[6];
 garden[0] = new Plant("가", Plant.LifeCycle.ANNUAL);
 garden[1] = new Plant("나", Plant.LifeCycle.BIENNIAL);
 garden[2] = new Plant("다", Plant.LifeCycle.PERENNIAL);
 garden[3] = new Plant("마", Plant.LifeCycle.ANNUAL);
 garden[4] = new Plant("바", Plant.LifeCycle.BIENNIAL);
 garden[5] = new Plant("사", Plant.LifeCycle.PERENNIAL);

for (Plant p : garden) {
    plantsByLifeCycle[p.lifeCycle.ordinal()].add(p);
}

for(int i =0; i < plantsByLifeCycle.length; i++) {
    System.out.printf("%s : %s%n",
            Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
}
```

문제점은 다음과 같다.

- 배열과 제너릭은 호환되지 않으니, 비검사 형변환 경고가 나온다
- 배열은 각 인덱스의 의미를 모르니 마지막 출력문 코드 처럼 출력결과에 직접 레이블을 달아야함
- 내 코드가 정확한 정수값을 사용한다고 보증해야 한다. 실수가 생기면, 잘못된 동작을 계속 수행하거나, Exception이 던져진다.



# EnumMap으로 해결하기

```java
Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle =
  new EnumMap<>(Plant.LifeCycle.class);

for (Plant.LifeCycle lc : Plant.LifeCycle.values())
  plantsByLifeCycle.put(lc,new HashSet<>());

Plant[] garden = new Plant[6];
garden[0] = new Plant("가", Plant.LifeCycle.ANNUAL);
garden[1] = new Plant("나", Plant.LifeCycle.BIENNIAL);
garden[2] = new Plant("다", Plant.LifeCycle.PERENNIAL);
garden[3] = new Plant("마", Plant.LifeCycle.ANNUAL);
garden[4] = new Plant("바", Plant.LifeCycle.BIENNIAL);
garden[5] = new Plant("사", Plant.LifeCycle.PERENNIAL);

for(Plant p : garden) {
  plantsByLifeCycle.get(p.lifeCycle).add(p);
}

System.out.println(plantsByLifeCycle);
```

ordinal 메서드를 사용한거와 다르게 다음과 같은 장점이 있다. 

- 형변환이 필요 없다.
- 맵의 키로 출력용 문자열을 제공하기 때문에, 출력 결과에 직접 레이블을 달지 않아도 된다.
- 배열의 인덱스를 계산하는 코드를 직접 짤 필요가 없음.
- EnumMap은 내부적으로 배열을 사용하기 때문에 ordinal 메서드를 쓰는것과 비슷한 성능을 보여줌

EnumMap의 생성자가 받는 키 타입의 Class 객체는 한정적 타입 토큰으로, 런타임시 제너릭 타입 정보를 제공한다. 



# Stream을 이용해서 더 최적화 해보기

```java
Plant[] garden = new Plant[6];
garden[0] = new Plant("가", Plant.LifeCycle.ANNUAL);
garden[1] = new Plant("나", Plant.LifeCycle.BIENNIAL);
garden[2] = new Plant("다", Plant.LifeCycle.PERENNIAL);
garden[3] = new Plant("마", Plant.LifeCycle.ANNUAL);
garden[4] = new Plant("바", Plant.LifeCycle.BIENNIAL);
garden[5] = new Plant("사", Plant.LifeCycle.PERENNIAL);

System.out.println(Arrays.stream(garden)
        .collect(groupingBy(p -> p.lifeCycle)));
```

다음과 같이 Stream을 이용하면 코드가 단순해지지만, EnumMap이 아닌 고유한 맵 구현체를 사용했기 때문에, EnumMap을 써서 얻는 공간과 성능 이점이 사라진다는 문제점이 있다. 



**EnumMap으로 사용하기**

```java
Plant[] garden = new Plant[6];
garden[0] = new Plant("가", Plant.LifeCycle.ANNUAL);
garden[1] = new Plant("나", Plant.LifeCycle.BIENNIAL);
garden[2] = new Plant("다", Plant.LifeCycle.PERENNIAL);
garden[3] = new Plant("마", Plant.LifeCycle.ANNUAL);
garden[4] = new Plant("바", Plant.LifeCycle.BIENNIAL);
garden[5] = new Plant("사", Plant.LifeCycle.PERENNIAL);

System.out.println(Arrays.stream(garden)
                   .collect(groupingBy(p -> p.lifeCycle,
                                       () -> new EnumMap<>(Plant.LifeCycle.class), toSet())));
```

위와 같이 사용하면 Map 구현체를 EnumMap으로 사용할 수 있다.

EnumMap 구현체를 직접 사용하는 방법과, Stream에서 EnumMap을 사용하는 방법의 차이점이 있다.

**`Plant.ANNUAL` 이 하나도 존재하지 않으면, EnumMap 버전에서는 맵을 3개 만들지만, Stream을 사용하면 2개만 만든다.**



# 또 다른 ordinal 사용 사례

두 열거 타입 값들을 매핑하기 위해서 ordinal을 두 번이나 쓴 배열들의 배열들이 있다.

다음의 코드는 상태(Phase)를 전이(Transition)와 매핑한 코드다.

**예를 들어 액체(LIQUID)에서 고체(SOLID)로 변화가 일어나면, 전이는 응고(FREEZE)이다.**

```java
enum Phase {

    SOLID, LIQUID, GAS; 

    enum Transition {
        MELT, FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;

        // 행은 from의 ordinal을, 열은 to의 ordinal을 인덱스로 쓴다.
        private static final Transition[][] TRANSITIONS = {
                {null, MELT, SUBLIME},
                {FREEZE, null, BOIL},
                {DEPOSIT, CONDENSE, null}
        };
        
        // 한 상태에서 다른 상태로의 전이를 반환한다.
        public static Transition from(Phase from, Phase to) {
            return TRANSITIONS[from.ordinal()][to.ordinal()];
        }
    }
}
```

앞에서 살펴봤던 oridinal 메서드의 단점을 그대로 갖고 있다.

- 컴파일러는 ordinal과 배열 인덱스의 관계를 알 수 없다.
  - Phase 클래스나, Transition 클래스를 수정하고, **Transition.TRANSITIONS** 변수를 수정하지 않으면 오류가 난다.
- Phase가 늘어나면, 제곱해서 커지기 때문에 null로 채워지는 칸도 늘어난다.



# EnumMap으로 해결하기



```java
enum Phase {

    SOLID, LIQUID, GAS;

    enum Transition {

        MELT(SOLID, LIQUID), FREEZE(LIQUID,SOLID),
        BOIL(LIQUID,GAS), CONDENSE(GAS,LIQUID),
        SUBLIME(SOLID,GAS), DEPOSIT(GAS,SOLID);

        private final Phase from;
        private final Phase to;

        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }
        
        //TRANSITIONS 맵을 초기화
        private static final Map<Phase, Map<Phase, Transition>> m =
                Stream.of(values())
                .collect(groupingBy(t -> t.from ,
                        () -> new EnumMap<>(Phase.class),
                        toMap(t -> t.to, t-> t,
                                (x,y) -> y, () -> new EnumMap<>(Phase.class))
                ));
        
        public static Transition from(Phase from, Phase to) {
            return m.get(from).get(to);
        }
    }
}
```

**Map<Phase, Map<Phase, Transition>>** 객체를 만들어, value에 사용된 맵에는 이후 상태와 전이를 연결하고, 바깥 맵은 이전 상태와 안쪽 맵을 연결한다.

코드가 복잡하긴 한데, 한줄 씩 살펴보자.

- `groupingBy` 에서는 전이를 이전 상태를 기준으로 묶고
- toMap에서는 이후 상태를 전이에 대응시키는 EnumMap을 생성한다.
  - `(x,y) -> y` 는 선언만하고 실제로는 쓰이지 않는다. 단지 EnumMap을 얻으려면 맵 팩터리가 필요하고 수집기들은 점층적 팩터리가 필요하기 때문



**여기서 새로운 상태인 플라스마(PLASMA)를 추가해보자.**

- 기체에서 플라스마로 변화하는 이온화(IONIZE)
- 플라스마에서 기체로 변하는 탈이온화(DEIONIZE)

```java
enum Phase {

    SOLID, LIQUID, GAS, PLASMA;

    enum Transition {

        MELT(SOLID, LIQUID), FREEZE(LIQUID,SOLID),
        BOIL(LIQUID,GAS), CONDENSE(GAS,LIQUID),
        SUBLIME(SOLID,GAS), DEPOSIT(GAS,SOLID),
      IONIZE(GAS, PLASMA), DEIONIZE(PLASMA, GAS);
    }
  ...
}   
```

다른 로직 수정 없이, enum만 추가하면 깔끔하게 끝난다.

## 정리

**배열의 인덱스를 얻기 위해 ordinal을 쓰는 것은 일반적으로 좋지 앟으니, 대신 EnumMap을 사용하자.**