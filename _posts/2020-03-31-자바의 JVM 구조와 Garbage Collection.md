---
layout: post
title: 자바의 JVM 구조와 Garbage Collection
categories: [Java]
comments: true 
---

## JVM 메모리 모델 

![]({{"/img/gc/JVMMemory.png" | relative_url}})

위에 사진을 보면 JVM 메모리는 부분적으로 나누어져 있습니다. 큰 부분에서 보면 JVM Heap 메모리는 물리적으로 **Young 영역**과 **Old 영역** 두부분으로 나누어져 있습니다.

## Young 영역

Young 영역은 **새로운 객체가 생성되는 곳입니다.** Young 영역이 가득차게 되면 GC가 동작됩니다. 이 동작을 **Minor GC**라고 합니다. Young 영역은 3가지 부분으로 나눌 수 있는데, **Eden 공간과 두개의 Survivor 공간**입니다.

### Young 영역의 흐름 

- 최근에 만들어진 객체는 Eden 영역에 위치합니다.

  ![](https://github.com/DaeAkin/DaeAkin.github.io/blob/master/img/gc/Minor1.png?raw=true)

- Eden **영역이 가득차면, Minor GC**가 발생하는데, 여기서 살아남은 객체들은 **survivor 공간 중 하나**로 이동합니다.

  ![](https://github.com/DaeAkin/DaeAkin.github.io/blob/master/img/gc/Minor2.png?raw=true)

- Minor GC는 **survivor 공간**도 같이 검사를 해서 **다른 survivor 공간**으로 이동시킵니다. 그러므로 survivor 공간 두개 중 하나는 항상 비어있습니다.

  ![](https://github.com/DaeAkin/DaeAkin.github.io/blob/master/img/gc/Minor3.png?raw=true)

- 여러 번의 Minor GC를 많이 했음에도 불구하고 살아남은 객체들은 Old 영역으로 이동합니다. **Minor GC를 당했지만 살아남은 객체는 1살씩(?) 먹습니다**. 몇 살이 먹으면 Old 영역으로 가야하는지 설정 해 줄 수 있습니다. 

## Old 영역

Old 영역에는 여러 번의 **Minor GC를 겪고 살아남은 객체들**이 있습니다. Old 영역에서 메**모리가 가득차게되면 Major GC**가 발생합니다. 

Major GC는 Minor GC에 비해 많은 시간이 걸립니다.

## Stop the World(세상을 멈춰라 😅)

모든 GC는 **"Stop the World"** 라는 이벤트가 발생합니다. **GC가 동작하면 GC를 제외한 모든 스레드가 멈추기 때문입니다.**

Young 영역의 Minor GC는 객체는 수명이 짧고 많은 객체를 검사하지 않기 때문에 속도가 매우 빨라 거의 애플리케이션에 영향을 주지 않습니다.

그러나 Major GC는 살아 있는 **모든 객체를 검사해야 하기 때문에 오랜 시간이 걸립니다.** Major GC가 일어나면 애플리케이션이 아무런 동작을 하지 않기 때문에 **Major GC가 일어나는 횟수를 최소화 해야 합니다.**  만약 애플리케이션이 Major GC가 일어나는 횟수가 많다면 timeout 에러를 발생시킬 확률이 높습니다.

GC가 동작하는 시간은 GC의 전략에 따라 다릅니다. 그래서 Stop the World를 발생시키는 횟수를 줄이기 위해서는 GC를 모니터링 하여 적절하게 튜닝하는 습관이 필요합니다.

## Permanent 영역 

Permanent 영역은 JVM이 애플리케이션에 있는 **클래스들과 메소드들을 알기위해 메타데이터들**이 포함되어 있습니다. **Permanent 영역은 Heap 공간이 아닙니다.** 

Permanent 영역은 애플리케이션이 사용하는 클래스들을 기초로 JVM이 런타임 시 생성합니다. 또한 Java SE 라이브러리 클래스들과 메소드들을 포함하고 있으며, Full GC에 의해 GC가 됩니다.

> Minor GC Vs Major GC Vs Full GC
>
> - Minor GC : Young 부분
> - Major GC : Old 부분
> - Full GC : Minor GC + Major GC

## Method Area

Method Area는 Permanent 영역의 한부분이며 **클래스 구조(런타임 상수나, static 변수들)와 메소드와 생성자 코드를 저장**하기 위해 사용합니다. 

## Memory Pool

Memory Pool은 변경 불가능 객체의 pool을 만들기 위해 JVM 메모리 매니저가 만듭니다. String Pool이 Memory pool의 좋은 예제 중 하나 입니다. **Memory Pool은 JVM 메모리 관리 구현에 따라 Heap 또는 Perm 영역에 존재합니다.**

## 자바 힙 메모리 스위치

자바에서 메모리 사이즈와 비율을 설정할 수 있는 메모리 스위치를 제공합니다. 

| VM Switch         | 설명                                                         |
| ----------------- | ------------------------------------------------------------ |
| -Xms              | JVM이 시작할 때 초기 heap 사이즈를 설정                      |
| -Xmx              | heap의 최대 사이즈를 설정                                    |
| -Xmn              | Young 영역의 사이즈를 설정한다. 나머지는 Old 영역사이즈로 설정됨. |
| -XX:PermGen       | Permanent 영역의 초기 사이즈를 설정                          |
| -XX:MaxPermGen    | Permanent 영역의 최대 사이즈를 설정                          |
| -XX:SurvivorRatio | Eden 영역과 , Survivor 영역의 비율을 설정.<br /> 예를 들어 Young 영역의 사이즈가 10m 인데, -XX:SurvivorRatio=2 로 설정하면<br />Eden 영역에는 5m가 설정되고, Survivor 영역은 각각 2.5m씩 할당됨.<br />기본값은 8이다. |
|                   |                                                              |

[JVM 옵션들](https://www.oracle.com/technetwork/java/javase/tech/vmoptions-jsp-140102.html)

## Garbage Collection(GC)

자바의 Garbage Collection(GC)는 사용하지 않는 객체를 메모리에서 제거하는 것이 목적 입니다. 다른 프로그래밍 언어인 C언어 같은경우 메모리 할당과 제거를 수동으로 해줘야하지만, 자바는 자동으로 GC가 처리해줍니다.

GC는 프로그램 실행 중에 메모리안에 있는 모든 객체를 검사하여 참조하지 않는 부분을 찾아내는 백그라운드 프로그램입니다. 참조를 하지 않는 모든 객체는 모두 삭제되며 추후에 다른 객체를 위해 할당을 해줄 수 있습니다.

## GC의 종류

애플리케이션에서 사용할 수 있는 GC의 종류는 총 5가지가 있습니다.

1. **Serial GC(-XX:+UseSerialGC)** : Serial GC는 Young(Minor GC)과 Old(Major GC)에 mark-sweep-compact 라는 방법을 이용합니다.  클라이언트의 머신이 독립적인 애플리케이션이거나, CPU 성능이 낮을 때 유용하며 하나의 스레드만 사용합니다. 멀티 프로세스 하드웨어를 활용할 수 없고, 멀티 프로세서 환경에서도 소형 데이터셋(최대 100MB정도)을 다루는 애플리케이션이라면 쓸만합니다.

   - Young GC 알고리즘 : Serial
   - Old GC 알고리즘 : Serial Mark-Sweep-Compact

2. **Parallel GC(-XX:+UseParallelGC)** :  Parallel GC는 **시스템에 있는 CPU 코어의 수 만큼 스레드를 만들어 Young(Minor GC)에 이용하는 것**을 제외하고는 Serial GC와 같습니다. `-XX:ParallelGCThreads=n ` JVM 옵션을 사용해서 스레드의 수를 조절할 수 있습니다. Parallel GC는 GC 성능을 높이기 위해 여러 CPU를 사용하기 때문에 throughput collector라고도 불립니다. **Parallel GC는 Old(Major GC)에서는 한개의 스레드만 사용합니다.**

   - Young GC 알고리즘 : Parallel Scavenge
   - Old Gc 알고리즘 : Serial Mark-Sweep-Compact

3. **Parallel Old GC(-XX:+UseParallelOldGC)** : **Young(Minor GC)와 Old(Major GC)에서 모두 여러 개의 스레드를 사용**하는 것만 제외 하고는 Parallel GC와 동일합니다.

4. **Concurrent Mark Sweep(CMS) Collector(-XX:+UseConcMarkSweepGC)** : CMS Collector는 Concurrent low pause collector라고도 불립니다. Old(Major GC)에 관련된 GC입니다. **CMS Collector는 GC 작업과 애플리케이션이 사용하는 스레드를 동시에 수행하여 Stop-The-World 때문에 일어나는 애플리케이션 중지 상태를 최소화 합니다.** **Young 영역에서는 Parallel Collector와 똑같은 알고리즘**을 사용합니다. 이 GC는 GC때문에 애플리케이션이 정지되는 일이 없어야 할 때 사용합니다. `-XX:ParallelCMSThreads=n` JVM 옵션을 사용하면 CMS collector의 스레드 수를 조절할 수 있습니다.

   CMS GC는 Stop-The-World때문에 일어나는 일시 정지 시간을 줄이는 것이 목적이며, STW때문에 일어나는 **일시정지를 포함한 평균 응답 시간을 줄이고자** 할 때 사용합니다.

   - Young GC 알고리즘 : Parallel
   - Old GC 알고리즘 : Concurrent Mark-Sweep

5. **G1 GC(-XX:UseG1GC)** : G1 GC는 Java7 에서부터 사용할 수 있으며, CMS collector를 대체하는 것이 주된 목표입니다. G1 GC는 병렬성, 동시성 , 적은 Stop-the-world 를 가진 GC 입니다. G1 GC는 다른 GC와 다르게 **Young 영역과 Old 영역이 없습니다.** 힙 공간을 여러 개의 동일한 사이즈의 힙 공간으로 분리하는데 분리된 공간을 region이라고 부릅니다. GC가 호출되면 region 중에 liveness가 가장 적은 곳이 GC 됩니다. 

   - Young GC 알고리즘 : Snapshot-At-The-Beginning(SATB)
   - Old GC 알고리즘 : Snapshot-At-The-Beginning(SATB)

    [Garbage-First Collector Oracle Documentation](https://docs.oracle.com/javase/7/docs/technotes/guides/vm/G1.html).

## GC 모니터링 

애플리케이션의 GC 활동을 모니터링 하기 위해 GUI 또는 CUI를 사용할 수 있습니다. 

### jstat

JVM 메모리와 GC활동을 모니터링 하기 위해 CUI 에서 jstat을 사용할 수 있습니다. 기본 JDK를 이용하므로 추가작업은 없습니다.

아무 애플리케이션이나 실행시키고 다음과 같은 작업을 해주세요.

> 저는 현재 프로젝트 하고 있는 outstagram-discovery.jar 을 실행시켰습니다.

```
$ java -Xmx120m -Xms30m -Xmn10m -XX:PermSize=20m -XX:MaxPermSize=20m -XX:+UseSerialGC -jar outstagram-discovery.jar
```

`jstat` 을 실행하려면 애플리케이션의 **프로세스 id**를 알고 있어야 합니다. 터미널을 열고 `ps -eaf | grep application.jar` 을 입력하시면 됩니다.

```
Donghyeonui-iMac:~ donghyeonmin$ ps -eaf | grep outstagram-discovery.jar
  501 32235 32069   0 11:29PM ttys008    0:20.42 /usr/bin/java -Xmx120m -Xms30m -Xmn10m -XX:PermSize=20m -XX:MaxPermSize=20m -XX:+UseSerialGC -jar outstagram-discovery.jar
  501 32331 32088   0 11:31PM ttys009    0:00.00 grep outstagram-discovery.jar
```

여기서 프로세스 id는 32235입니다. jstat을 실행해보겠습니다.

```
Donghyeonui-iMac:~ donghyeonmin$ jstat -gc 32235 1000
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       PC     PU    YGC     YGCT    FGC    FGCT     GCT
1024.0 1024.0  0.0    0.0    8192.0   7933.3   42108.0    23401.3   20480.0 19990.9    157    0.274  40      1.381    1.654
1024.0 1024.0  0.0    0.0    8192.0   8026.5   42108.0    23401.3   20480.0 19990.9    157    0.274  40      1.381    1.654
1024.0 1024.0  0.0    0.0    8192.0   8030.0   42108.0    23401.3   20480.0 19990.9    157    0.274  40      1.381    1.654
1024.0 1024.0  0.0    0.0    8192.0   8122.2   42108.0    23401.3   20480.0 19990.9    157    0.274  40      1.381    1.654
1024.0 1024.0  0.0    0.0    8192.0   8171.2   42108.0    23401.3   20480.0 19990.9    157    0.274  40      1.381    1.654
1024.0 1024.0  48.7   0.0    8192.0   106.7    42108.0    23401.3   20480.0 19990.9    158    0.275  40      1.381    1.656
1024.0 1024.0  48.7   0.0    8192.0   145.8    42108.0    23401.3   20480.0 19990.9    158    0.275  40      1.381    1.656

```

명령어의 마지막 인자는 GC 데이터를 매번 1초마다 출력하라는 출력시간의 변수 입니다. 

컬럼하나씩 훑어보겠습니다.

> 사이즈는 KB(킬로바이트)가 기본입니다.

- **S0C 와 S1C** : Survivor0과 Survivor1 영역의 사이즈 입니다.
- **S0U 와 S1U** : Survivor0과 Survivor1 영역에서 사용하고 있는 사이즈 입니다. 둘중 하나는 항상 비어있습니다.
- **EC 와 EU** : EC는 Eden 영역의 사이즈를 말합니다.EU는 Eden의 사용량을 말합니다. 위에를 보면 EU 사이즈가 계속 증가하는 걸 볼 수 있는데, Minor GC가 동작하면 다시 사이즈가 줄어듭니다.
- **OC 와 OU** : OC는 Old 영역의 사이즈를 말합니다. OU는 Old 영역의 사용률 입니다.
- **PC 와 PU** : PC는 Permanent 영역의 사이즈를 말합니다. PU는 Permanent 영역의 사용률 입니다. 
- **YGC와 YGCT** : YGC는 young 영역 안에 GC가 얼마큼 일어났는지 보여줍니다. YGCT는 Young 영역안에서 GC가 걸리시간의 누적시간을 보여줍니다. EU의 값이 줄어들 때 바로 Minor GC가 발생했기 때문에, 그 부분을 보시면 EU 값은 줄고 YGC와 YGCT는 값이 늘어났습니다.
- **FGC and FGCT** : FGC는 Full GC가 몇 번 일어났는지 보여줍니다. FGCT는 Full GC가 걸린 시간의 누적을 보여줍니다. 
- **GCT** : 모든 GC가 걸린 누적 시간을 보여줍니다. YGCT와 FGCT의 합입니다.

jstat은 GUI를 가지고 있지 않은 원격 서버에서 실행하기에 좋습니다. 애플리케이션을 실행할 때 `-Xmn10m` 을 주어 Young영역의 사이즈를 지정해주었기 때문에 S0C , S1C , EC의 합은 10m 입니다.



### Java VisualVM with Visual GC

만약 GUI로 메모리를 보고 싶다면 jvisualvm 툴을 사용하면 됩니다.

[VisualVM](https://visualvm.github.io/)

VisualVM 설치후 Tool -> plugin에 들어가서 Visual GC를 설치해줍니다.

![](https://github.com/DaeAkin/DaeAkin.github.io/blob/master/img/gc/visualVM2.png?raw=true)



![](https://github.com/DaeAkin/DaeAkin.github.io/blob/master/img/gc/visualVM.png?raw=true)

이렇게 GUI로 볼 수도 있습니다.

## GC 튜닝 

GC튜닝은 애플리케이션의 처리량을 올리거나 GC의 시간이 길어 timeout이 발생할 때 사용하는 최후의 보루로 사용해야합니다.

만약 `java.lang.OutOfMemoryError: PermGen space` 에러를 본다면 Permanent 영역의 메모리를 -XX:PermGen 과 -XX:MAXPermGEN을 이용해서 늘리면 됩니다. 

만약 Full GC가 많아진다면 Old 영역의 메모리 크기를 늘리면 됩니다.

전반적인 GC 튜닝은 많은 노력과 시간이 필요하며, 그것에 대한 정확한 답은 없습니다. 애플리케이션에 가장 알맞은 옵션을 찾는 노력이 필요합니다.

- [JDK 12 Garbage Collection Tuning Guide](https://docs.oracle.com/en/java/javase/12/gctuning/introduction-garbage-collection-tuning.html)
- [JDK 11 Garbage Collection Tuning Guide](https://docs.oracle.com/en/java/javase/11/gctuning/introduction-garbage-collection-tuning.html)
- [JDK 10 Garbage Collection Tuning Guide](https://docs.oracle.com/javase/10/gctuning/introduction-garbage-collection-tuning.htm)
- [JDK 9 Garbage Collection Tuning Guide](https://docs.oracle.com/javase/9/gctuning/introduction-garbage-collection-tuning.htm)
- [JDK 8 Garbage Collection Tuning Guide](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/)

## 마치며

만약 응답시간이 전체 처리량보다 중요시 하는 애플리케이션이라면  CMS GC를 선택해볼만 합니다. 그러나 항상 어떤 GC를 선택해야 합니까? 에 답은 없습니다. 하드웨어에 따라 다르고 애플리케이션 마다 다른 GC가 최적의 방법일 수도 있습니다.

개발자가 컬렉터를 바꿔가면서 테스트를 해보고 그에 맞게 Heap 사이즈도 조절도 해봐야 합니다.



