# Apache Kafka

## 소개

Apache Kafka는 **분산 스트리밍 플랫폼**입니다. 분산 스트리밍 플랫폼을 무엇을 의미하는 걸까요?

분산 스트리밍 플랫폼은 다음의 **세가지** 특징을 갖고 있습니다.

- 메세지 큐또는 엔터프라이즈 메세징 시스템과 비슷하게 데이터 Stream을 Publish(발행) 하거나 Subscribe(구독) 합니다.
- 시스템 장애가 일어나면 데이터들을 스트림에 저장하므로 장애에 대해 내성을 지니고 있습니다.
- 이벤트가 일어나면 즉시 처리합니다.

카프가는 두 개의 광범위한 클래스가 사용됩니다.

- 시스템 또는 애플리케이션간에 데이터를 안정적으로 가져 오는 실시간 스트리밍 데이터 파이프라인 구축
- 데이터를 변환하거나, 반응하는 실시간 스트리밍 애플리케이션 구축

이러한 것들을 카프카가 어떻게 하는지 알기 위해서는 카프카 바닥부터 탐험해 보겠습니다. 

첫번째로 카프카의 **개념**입니다.

- 카프카는 한개 또는 그 이상 클러스터를 구축해서 여러 개의 데이터센터를 가질 수 있습니다.
- 카프카 클러스터는 카테고리처럼 **토픽(topic)** 이라고 곳에 데이터들을 분류해서 저장할 수 있습니다.
- 각각의 데이터들은 **key**와 **value** 그리고 **timestamp로** 이루어져 있습니다.

**다음은 카프카의 5개의 주요 API들 입니다** 

- [Producer API](https://kafka.apache.org/documentation.html#producerapi) 는 한개 또는 여러 개의 토픽에 데이터를 publish(발행) 해줍니다.
- [Consumer API](https://kafka.apache.org/documentation.html#consumerapi) 는 한개 또는 여러 개의 토픽 을 subscribe(구독)하고 그 데이터를 가져옵니다.
- [Stream API](https://kafka.apache.org/documentation/streams) 는 stream 프로세서이며, 한개 또는 여러개의 토픽에서 데이터를 가져와 가공한 다음 다시 stream에 넣습니다. 인풋 스트림에서 아웃풋 스트림으로 어떤걸 변환해야 할 때 효과적 입니다.
- [Connector API](https://kafka.apache.org/documentation.html#connect) 는 카프카 토픽에 연결하여 이미 존재하는 애플리케이션이나, DB에 접근할 때 재사용 가능한 producer나 consumer를 제공해줍니다.
- [Admin API](https://kafka.apache.org/documentation.html#adminapi) 는 토픽과 브로커 다른 카프카 객체들을 관리하거나 검사합니다.

![kafka-apis](https://kafka.apache.org/24/images/kafka-apis.png)

카프카에서 클라이언트와 서버는 **간단하며**, **높은 성능**과 **언어에 구애받지 않는 TCP 프로토콜**로 서로 통신 합니다. 



## 토픽과 로그

첫번 째로 핵심적인 토픽을 살펴 보겠습니다.

토픽은 데이터가 발행되는 카테고리를 말합니다. 카프카에서 토픽은 항상 여러 명의 구독자(multi-subscribe)가 있습니다. 즉 토픽은 0개나 1개 또는 그 이상의 데이터들을 구독(subscribe)를 하는 소비자(consumer)를 가질 수 있습니다.

각각의 **토픽**은 **카프카 클러스터가** 다음과 같이 생긴 **파티션**된 **로그**로 유지합니다.

![log_anatomy](https://kafka.apache.org/24/images/log_anatomy.png)

각각의 파티션은 **정렬**되있으며, **불변적인 데이터**이며, 계속해서 데이터가 덧붙여집니다. 이 파티션에 있는 데이터는 파티션 내부에서 **offset**이라고 불리는 **유니크한 식별자**인 **연속적인 ID**를 할당 받습니다.

카프카에 들어간 데이터들이 소비자(Consumer)에 의해 소비(consumption) 됐던 말던 데이터들을 안전하게 보관합니다.(보관 기간은 설정 가능) 예를 들어 데이터 보관 기간을 2일로 정했다면, 2일동안 데이터가 살아 남아있고, 그 후에는 데이터가 사라집니다. 

![log_consumer](https://kafka.apache.org/24/images/log_consumer.png)



소비자(Consumer)마다  전달되는건 로그에 있는 소비자의 **offset**이나 **위치**의 **메타데이터**들 입니다. 이 offset은 소비자가 제어하는데, 소비자가 데이터를 읽을 때 offset을 선형으로 진행합니다. 그러나 위치는 소비자자가 제어하므로 소비자가 원하는 순서대로 데이터를 읽어들일 수 있습니다. 예를 들어 소비자가 예전 데이터를 재가공해서 오래된 offset을 리셋하거나, 최근의 데이터를 스킵하거나, 지금부터의 데이터를 읽어들일 수 있습니다. 



이런 장점들은 카프카 클러스터나 다른 소비자에 영향을 받지 않고 데이터를 왔다갔다 할 수 있습니다. 예를 들어 현재 컨슈머가 어떤 데이터를 갖고 오고 있는지 상관없이 아무 토픽에서 마지막의 데이터를 가져올 수 있습니다.

로그안에 있는 파티션은 몇가지 목적을 수행합니다. 첫번째로, 로그를 서버 1대에 맞게 스케일 할 수 있습니다. 개별의 파티션은 이를 호스트 하고 있는 서버와 알맞아야 하는데, 토픽이 파티션을 많이 갖고 있다면 임의적으로 데이터의 양을 조절할 수 있습니다. 두번째로, 병렬처리를 지원 합니다.



![](./images/kafka01.png)

## 분산

로그의 파티션들은 카프카 클러스터에게 전해지는데, 이 카프카 클러스터는 데이터를 핸들링하고 파티션을 공유를 합니다. 각각의 파티션은 장애내성을 위해 설정 값들을 복사합니다. 

각각의 파티션은 **"leader"** 처럼 행동하는 서버를 하나씩 갖고 있고, 0개 또는 여러 개의 **followers** 서버를 갖고 있습니다. **leader**는 파티션에 관한 모든 읽기와 쓰기 요청을 담당하고, 반면에 **follower** 는 수동적으로 leader를 복사합니다. 만약 leader가 어떤 이유로 죽으면, follwer중 하나가 자동적으로 **새로운** leader가 됩니다. 그래서 각각 서버는 클러스터안에 밸런스있게 리더와 팔로워가 공존하고 있습니다.

## Geo-Replication

카프카 MirrorMaker는 클러스터에 geo-replication을 제공합니다. MirrorMaker를 사용하면 메세지는 여러 개의 데이터센터 또는 클라우드를 넘어 메세지가 복제 됩니다. 백업 또는 리커버리지에 관한 시나리오로 활성화하거나 수동적으로 사용할 수 있으며, CDN처럼 사용자와 가까운 곳에 데이터를 위치시키기 위해 사용합니다.

## Producers (생산자)

Producers는 선택된 topic에게 데이터를 주는 역할을 합니다. 생산자는 어떤 데이터를 토픽안에 어떤 파티션 넣을지에 대한 책임이 있습니다. 로드밸런싱을 위해 라운드로빈을 사용하거나, 특정 파티션 함수를 이용합니다. 

> 주로 저는 Producer를 한글로 생산자라고 말합니다.

## Consumers(소비자)

Consumer는 그룹 이름으로 구별하고, 토픽으로 들어온 데이터를 토픽을 구독한 소비자 인스턴스에 전달 됩니다. Consumer 인스턴스는 프로세스에 분리되있거나, 머신에서 분리되어있습니다.(같은 카프카에 존재하지 않는다는 뜻.)

모든 consumer 인스턴스가 같은 consumer 그룹을 갖는다면, 데이터가 효과적으로 로드밸런싱 됩니다.

모든 consumer 인스턴스가 각각 다른 consumer 그룹을 갖는다면, 각각의 데이터는 consumer 프로세스에 브로드캐스트 됩니다. 

![consumer-groups](https://kafka.apache.org/24/images/consumer-groups.png)

2개의 카프카서버가 클러스터를 이루어서 파티션 P0과 P3까지 호스팅하고 두 개의 컨슈머 그룹이 있습니다.  컨슈머 그룹 A는 2개의 컨슈머 인스턴스(C1,C2)가 있고, 컨슈머 그룹 B는 4개의 인스턴스(C3,C4,C5,C6)가 있습니다.

그런데 토픽들은 consumer group들을 갖고 있는데, 그 이유는 확장성과, 장애 내성을 위한 것입니다. 

More commonly, however, we have found that topics have a small number of consumer groups, one for each "logical subscriber". Each group is composed of many consumer instances for scalability and fault tolerance. This is nothing more than publish-subscribe semantics where the subscriber is a cluster of consumers instead of a single process.

카프카에서 소비자들이 소비하는 방법은 로그안에 있는 파티션을 소비에 인스턴스들에게 분배를 해서, 인스턴스마다 언제든지 독점 소비자가 될 수 있습니다. 그룹에서 이런 유지 작업들은 카프카 프로토콜이 동적으로 처리합니다. 새로운 인스턴스가 그룹에 들어오게 되면 그룹 안에 있는 다른 소비자들의 일부부 파티션을 조금 때어 줍니다. 인스턴스가 죽으면 해당 인스턴스가 갖고 있는 파티션들은 남아 있는 인스턴스들에게 분배 됩니다.

Kafka only provides a total order over records *within* a partition, not between different partitions in a topic. Per-partition ordering combined with the ability to partition data by key is sufficient for most applications. However, if you require a total order over records this can be achieved with a topic that has only one partition, though this will mean only one consumer process per consumer group. 

## Multi-tenacy(멀티 테넌시 : 다중소유)

카프카를 멀티 테넌시 방법으로 배치할 수 있습니다. 멀티 테넌시는 어떤 토픽이 데이터를 생산하거나 소비할지 설정함으로써 활성화 시킬수 있습니다. 또한 할당량을 조절해줄 수 있습니다.  관리자는 요청에 대한 할당량을 정의하고 적용하여 클라이언트가 사용하는 브로커 리소스를 제어 할 수 있습니다.

> 멀티 테넌시(Multi-tenacy) : 다중 소유라는 뜻



## Gurantees

카프카는 다음과 같은 보장을 합니다.

- Producer가 특정한 토픽 파티션으로 보낸 메세지는 전송된 순서대로 추가됩니다. 같은 Producer가 M1을 보낸다음 M2를 보내면, M1 다음의 M2가 옵니다.
- Consumer 인스턴스는 로그에 담겨진 순서대로 데이터를 받습니다.
- Consumer들을 복제하기 때문에 서버가 죽어도, 데이터를 잃지 않습니다.



## 카프카의 메세징 시스템

전통적인 엔터프라이즈 메세징 시스템과 비교해서 카프카의 스트림의 개념은 어떻게 다를까요?

전통적인 메세징 시스템은 두개의 모델을 갖는데, queue와 publish-subscribe 입니다.

큐는 서버에서 데이터를 읽고 데이터들을 컨슈머에게 전해주는 컨슈머의 풀 입니다.  publish-subscribe 모델은 데이터를 모든 컨슈머에게 전달해주는 것입니다. 이 두개의 모델은 장점과 단점을 갖고 있습니다. 큐의 강점은 여러 개의 컨슈머 인스턴스에 대하여 데이터 작업을 분할시키게 해주고, 작업을 스케일업 해줄 수 있습니다. 불행하게도 큐는 여러개의 구독자(subscribe)를 둘 수 없습니다. 데이터가 읽혀지면 그 데이터는 사라지기 때문입니다. Publish-subscribe는 멀티 프로세스에게 데이터를 전달할 수 있지만, 모든 메세지가 모든 구독자에게 가므로 스케일업 할 방법이 없습니다.

카프카에서 컨슈머 그룹의 개념은 두개의 컨셉으로 일반화됩니다. 큐를 사용해서 컨슈머 그룹을 나누어 처리할 수 있고(컨슈머 그룹의 수로), publish-subscribe로 카프카는 여러 개의 컨슈머 그룹에게 메세지를 전달할 수 있습니다.

카프카 모델의 장점은 모든 토픽은 두개의 프로퍼티를 갖는데, 처리량을 늘릴 수 있고, 또한 멀티-구독자를 가질 수 있습니다. 둘중에 하나를 선택할 필요가 없습니다.

카프카는 또한 전통적인 메세징 시스템보다 강하게 데이터의 순서를 보장합니다. 

전통적인 큐는 서버에서 순서대로 데이터를 가져옵니다, 그리고 여러 명의 컨슈머가 큐에서 데이터를 가져오면 서버는 데이터가 저장된 순서대로 데이터를 줍니다. 그러나 서버는 데이터를 순서대로 줬지만 , 데이터는 비동기적으로 컨슈머에게 전달될 수 있어서, 컨슈머에게 다른 데이터의 순서로 전달 될 수 있습니다. 병렬적으로 컨슈머가 소비한다면 데이터의 순서를 잃어버릴 수 있습니다.전통적인 메세징 시스템은 주로 "독점적 소비자"의 개념을 갖고있으며, 오직 한개의 프로세스만 큐에서 컨슈머에게 전달하는 것입니다, 그러나 이 방법은 병렬처리를 지원하지 않습니다. 

이에 반해 카프카는 더 나은 방법을 제공합니다. 카프카는 파티션에 병렬처리 개념이 있기 때문에, 순서를 보장하고, 로드밸런싱이 가능합니다. 이런 병렬처리는 카프카가 파티션을 할당하기 때문에 그룹에 안에 있는 컨슈머는 정확히 해당 파티션을 소비합니다. 이렇게 함으로써 컨슈머가 오직 파티션의 구독자가 되고, 데이터를 순서대로 소비할수 있게 됩니다. 그리고 많은 파티션이 로드밸런싱 상태가 됩니다. 

## 스토리지 시스템

이렇게 메세지 발행과 소비가 분리된 다른 메세지 큐 시스템은 스토리지 시스템으로 효과적으로 작동합니다.

카프카에 쓰여진 데이터는 하드디스크에 저장되며, 장애내성을 위해 복제됩니다. 카프카는 확인할 때 까지 기다리므로, 전부 다 복제되거

(Data written to Kafka is written to disk and replicated for fault-tolerance. Kafka allows producers to wait on acknowledgement so that a write isn't considered complete until it is fully replicated and guaranteed to persist even if the server written to fails.)

카프카는 확장성이 우수하며, 데이터가 50KB가 있던지, 50TB가 있던지, 동일한 성능을 냅니다.

클라이언트가 읽는 위치를 제어하기 때문에, 카프카는 고성능,low 레이턴시,복제,전파 등 특수 목적 파일 시스템으로 생각할 수 있습니다.

#### 스트림 처리

읽기,쓰기 데이터스트림 저장도 가능한 반면 실시간으로 스트림 처리도 가능합니다.

카프카에서 스트림 프로세서는 토픽으로부터 끊임없는 데이터의 스트림과 이 데이터스트림에서 어떤 처리를 수행하거나, 다시 토픽으로 끊임없는 데이터를 보내줄 수 있습니다.

예를 들어, 매장 어플리케이션에서 할인,배송 데이터를 인풋으로 받고, 아웃풋으로 재정렬된 데이터나, 가격할인이 된 데이터를 받을 수 있습니다.

직접적으로 producer와 consumer API를 사용하면 가능합니다. 그러나 좀 더 복잡한 계산에서는 카프카는 통합된 Stream API를 제공해줍니다. 이 동작은 Stream을 모아서 계산한다거나, Stream을 join해서 같이 계산하는 작업을 하게 해줍니다.

#### 조각들 합치기 

이런 메세징,스토리지,stream 처리 기능들은 잘 사용하지 않을 것 같지만, 스트리밍 플랫폼에서 가장 중요한 역할을 합니다.

HDFS(하둡 분산 시스템)은 batch 처리를 위한 정적 파일을 저장합니다. 이와 같은 시스템은 가거의 데이터를 저장하고 처리할 수 있습니다(?)

전통적인 메세징 시스템은 메세지를 읽고 나서 들어오는 메세지를 처리해줍니다. 애

This combination of messaging, storage, and stream processing may seem unusual but it is essential to Kafka's role as a streaming platform.

A distributed file system like HDFS allows storing static files for batch processing. Effectively a system like this allows storing and processing *historical* data from the past.

A traditional enterprise messaging system allows processing future messages that will arrive after you subscribe. Applications built in this way process future data as it arrives.

Kafka combines both of these capabilities, and the combination is critical both for Kafka usage as a platform for streaming applications as well as for streaming data pipelines.

By combining storage and low-latency subscriptions, streaming applications can treat both past and future data the same way. That is a single application can process historical, stored data but rather than ending when it reaches the last record it can keep processing as future data arrives. This is a generalized notion of stream processing that subsumes batch processing as well as message-driven applications.

Likewise for streaming data pipelines the combination of subscription to real-time events make it possible to use Kafka for very low-latency pipelines; but the ability to store data reliably make it possible to use it for critical data where the delivery of data must be guaranteed or for integration with offline systems that load data only periodically or may go down for extended periods of time for maintenance. The stream processing facilities make it possible to transform data as it arrives.

For more information on the guarantees, APIs, and capabilities Kafka provides see the rest of the [documentation](https://kafka.apache.org/documentation.html).



# 사용 예

## Messaing

카프카는 전통적인 메세지 브로커를 대체에 쉽습니다. 메세지 브로커는 다양한 이유 때문에 사용됩니다(데이터 생산자로부터 프로세스를 분리, 처리되지 않는 메세지를 버퍼에 쌓거나). 대부분의 메세징 시스템과 비교하여 카프카는 처리량, 내장 파티셔닝, 복제 및 장애내성이 향상되고 대규모 메세지 처리에 적합한 솔루션 입니다.

## 웹사이트 활동 추적

본래의 카프카의 사용 사례는 실시간으로 publish-subscribe을 할 수 있는 사용자의 활동을 추적하는 것이였습니다. 이 뜻은 사이트 활동들(페이지,검색,유저들이 할 수 있는 다른 활동)이 활동마다 토픽이 있어 그 토픽으로 데이터가 발행되는 것이였습니다.  이런 피드들은 실시간 처리, 실시간 모니터링 그리고 하둡으로 로딩되어서 데이터들이 구독되었습니다. 

활동 추적은 주로 유저의 페이지 하나당 많은 활동 메세지들이 생성되었습니다.



## Metrics

카프카는 주로 운영 모니터링 데이터에 사용됩니다. 분산된 어플리케이션에서 모니터링 데이터들을 한곳으로 모으기위한 목적으로 사용됩니다.



## Log Aggregation

많은 사람들이 로그 수집의 대체 솔루션으로 카프카를 사용합니다. 로그 수집은 일반적으로 서버에서 실제 로그 파일을 수집하여 처리를 위해 중앙위치(파일 서버 또는 HDFS)에 저장합니다. 카프카는 파일의 세부 정보를 추상화하고 메세지 스트림으로 로그나 이벤트 데이터들을 보다 깔끔하게 추상화합니다. 이런 방법은 low-latency 처리를 하고, 여러 개의 데이터 소스 지원 및 데이터 분산 소비를 지원합니다. Scribe나 Flume 같은 로그 집중화 시스템과 비교하면 카프카는 좋은 성능과 , replication으로 인한 강한 내구성 그리고 레이턴시가 낮은 장점이 있습니다. 

## Stream Processing

Many users of Kafka process data in processing pipelines consisting of multiple stages, where raw input data is consumed from Kafka topics and then aggregated, enriched, or otherwise transformed into new topics for further consumption or follow-up processing. For example, a processing pipeline for recommending news articles might crawl article content from RSS feeds and publish it to an "articles" topic; further processing might normalize or deduplicate this content and publish the cleansed article content to a new topic; a final processing stage might attempt to recommend this content to users. Such processing pipelines create graphs of real-time data flows based on the individual topics. Starting in 0.10.0.0, a light-weight but powerful stream processing library called [Kafka Streams](https://kafka.apache.org/documentation/streams) is available in Apache Kafka to perform such data processing as described above. Apart from Kafka Streams, alternative open source stream processing tools include [Apache Storm](https://storm.apache.org/) and [Apache Samza](http://samza.apache.org/).



## Event Sourcing

[Event sourcing](http://martinfowler.com/eaaDev/EventSourcing.html) is a style of application design where state changes are logged as a time-ordered sequence of records. Kafka's support for very large stored log data makes it an excellent backend for an application built in this style.



## Commit Log

Kafka can serve as a kind of external commit-log for a distributed system. The log helps replicate data between nodes and acts as a re-syncing mechanism for failed nodes to restore their data. The [log compaction](https://kafka.apache.org/documentation.html#compaction) feature in Kafka helps support this usage. In this usage Kafka is similar to [Apache BookKeeper](https://bookkeeper.apache.org/) project.





## 시작하기

지금부터 카프카를 실습해보겠습니다. 전 맥을 쓰고 있기 때문에 맥 위주로 설명하겠습니다. 만약 윈도우를 사용하신다면 `bin/` 대신에 `bin\windows\ ` 과 .bat을 사용하시면 됩니다.



### 다운로드하기

[Download](https://www.apache.org/dyn/closer.cgi?path=/kafka/2.4.0/kafka_2.12-2.4.0.tgz) 여기에서 카프카를 다운 받으시면 됩니다.

```
$ tar -xzf kafka_2.12-2.4.0.tgz
$ cd kafka_2.12-2.4.0
```

### 카프카 서버 시작하기

카프카는 [ZooKeeper](https://zookeeper.apache.org/) 를 사용하기 때문에 먼저, ZooKeeper 서버를 시작해야합니다. 

**주키퍼 시작**

```
$ bin/zookeeper-server-start.sh config/zookeeper.properties 
```

**카프카 시작**

```
$ bin/kafka-server-start.sh config/server.properties 
```

### 토픽 만들기

test라는 토픽을 하나의 파티션으로 만들고 복제를 한번 해보겠습니다.

```
$ bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic test

```

list 토픽 명령을 실행하면 다음과 같은 토픽이 나옵니다

```
$ bin/kafka-topics.sh --list --bootstrap-server localhost:9092
test
```

대안으로는 이렇게 토픽을 수동으로 생성하는 대신에 브로커에게 토픽이 없을 때 자동으로 생성하는 설정을 할 수 있습니다.

### 메세지 보내기

파일 또는 표준입력에서 입력 받아 카프카 클러스터에 메세지로 전송하는 명령을 할 수 있습니다. 디폴트로 엔터 치자마자 메세지가 발행됩니다.

몇개의 메세지를 콘솔을 이용해서 서버에게 보내보겠습니다.

```
$ bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
This is a message
This is another message
```

### 소비자 시작하기

또한 표준 출력으로 메세지를 받는 명령을 할 수 있습니다.

```
$ bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
This is a message
This is another message
```

위에 있는 명령을 새로운 터미널에 입력하면 위에서 보낸 메세지를 받는걸 알 수 있습니다.

### 멀티 브로커 클러스터 설정하기

지금까지는 하나의 브로커만 사용했습니다. 카프카에서 싱글 브로커는 그냥 클러스터 사이즈가 1여서 몇개의 새로운 브로커 인스턴스를 실행해도 딱히 달라지는 건 없습니다. 그러나 느낌좀 내기위해서 세개를 더 추가해보겠습니다.

첫번 째로 각각의 브로커의 설정파일을 만들어줘야 합니다.(윈도우에서는 `copy` 명렁어를 사용하세요.)

```
$ cp config/server.properties config/server-1.properties
$ cp config/server.properties config/server-2.properties
```

그 다음 방금 복사한 파일의 설정을 조금 수정 하겠습니다.

```
config/server-1.properties:
    broker.id=1
    listeners=PLAINTEXT://:9093
    log.dirs=/tmp/kafka-logs-1
 
config/server-2.properties:
    broker.id=2
    listeners=PLAINTEXT://:9094
    log.dirs=/tmp/kafka-logs-2
```

`broker.id` 속성은 클러스트 안에서 유니크하고 영구적인 이름입니다. 여러 개의 카프카를 로컬에서 돌리기때문에port와 log를 오버라이드 해줘야 합니다. 

이미 ZooKeeper와 하나의 카프카서버가 이미 시작됐으므로 2개의 카프카를 시작해봅시다.

```
$  bin/kafka-server-start.sh config/server-1.properties 
```

```
$  bin/kafka-server-start.sh config/server-2.properties 
```

이제 Now create a new topic with a replication factor of three: 

```
$ bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 3 --partitions 1 --topic my-replicated-topic
```

이제 클러스터가 생겼으니 어떤 브로커가 무엇을 하고 있는지 알려면 "describe topics" 명령어를 실행하면 됩니다.

```
$ bin/kafka-topics.sh --describe --bootstrap-server localhost:9092 --topic my-replicated-topic
Topic:my-replicated-topic   PartitionCount:1    ReplicationFactor:3 Configs:
    Topic: my-replicated-topic  Partition: 0    Leader: 1   Replicas: 1,2,0 Isr: 1,2,0

```

첫번 째 줄은 모든 파티션의 요약을 나타냅니다. 해당 토픽에 관해서는 오직 한개의 파티션만 있기 때문에 1줄만 나옵니다. 

- "leader"는 주어진 파티션에 대해 읽기와 쓰기를 담당합니다. leader는 파티션에서 랜덤으로 선택됩니다. 
- "replicas"는 이 파티션을 복제한 로그의 리스트 수 입니다. (egardless of whether they are the leader or even if they are currently alive.)
- "isr" is the set of "in-sync" replicas. This is the subset of the replicas list that is currently alive and caught-up to the leader.

이 예제에서는 토픽의 파티션에 대해서는 node 1이 leader 역할을 합니다. 

우리가 만들었던 원래의 토픽이 어딨는지 확인하려면 다음과 같은 명령어를 입력하면 됩니다.

```
$ bin/kafka-topics.sh --describe --bootstrap-server localhost:9092 --topic test
Topic:test  PartitionCount:1    ReplicationFactor:1 Configs:
    Topic: test Partition: 0    Leader: 0   Replicas: 0 Isr: 0
```

역시나 원래 토픽은 replicas가 없으며, 서버를 만들 때 클러스터의 유일한 서버인 서버0에 있습니다.? (So there is no surprise there—the original topic has no replicas and is on server 0, the only server in our cluster when we created it.)

새로운 토픽에 몇 개의 메세지만 전송해보겠습니다.

```
$ bin/kafka-console-producer.sh --broker-list localhost:9092 --topic my-replicated-topic
...
my test message 1
my test message 2
^C
```

그 다음 이 메세지들을 받아 보겠습니다.

```
$ bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --from-beginning --topic my-replicated-topic
...
my test message 1
my test message 2
^C
```

그 다음 장애 내성을 테스트해보겠습니다. 브로커 1은 leader기 때문에 죽여보도록 하겠습니다.

```
$ ps aux | grep server-1.properties
7564 ttys002    0:15.91 /System/Library/Frameworks/JavaVM.framework/Versions/1.8/Home/bin/java...
$ kill -9 7564
```

> 윈도우에서는
>
> ```
> > wmic process where "caption = 'java.exe' and commandline like '%server-1.properties%'" get processid
> ProcessId
> 6016
> > taskkill /pid 6016 /f
> ```

팔로워중에 하나가 리더로 바뀌었으므로 노드 1은 더이상 replicat와 동기화 하지 않습니다.(Leadership has switched to one of the followers and node 1 is no longer in the in-sync replica set:)

```
$ bin/kafka-topics.sh --describe --bootstrap-server localhost:9092 --topic my-replicated-topic
Topic:my-replicated-topic   PartitionCount:1    ReplicationFactor:3 Configs:
    Topic: my-replicated-topic  Partition: 0    Leader: 2   Replicas: 1,2,0 Isr: 2,0
```

그러나 메세지를 갖고 있던 리더가 죽었음에도 불구하고 아까 보낸 메세지는 아직 유효합니다. 

```
$ bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --from-beginning --topic my-replicated-topic
...
my test message 1
my test message 2
^C
```

### 카프카 Connect를 이용해서 데이터를 import/export 하기

콘솔에서 데이터를 넣고 빼고 하는 작업은 처음에만 편합니다. 다른 소스의 데이터나 다른 시스템 카프카에 있는 데이터를 export하고 싶을 수도 있습니다. 커스텀 통합 코드를 작성하는 대신 카프카 Connect를 이용해서 데이터를 import 하거나 export 할 수 있습니다.

카프카 커넥트는 데이터를 카프카에게 import하거나 export를 할 수 있는 카프카가 포함된 툴입니다. 

테스트하기 위해 기본 데이터를 만들겠습니다.

```
$ echo -e "foo\nbar" > test.txt
```

windows에서

```
$ echo foo> test.txt
$ echo bar>> test.txt
```

이제 두개의 커텍터를 독립모드로 실행시킵니다. 실행시킬때 3개의 설정파일을 파라미터로 같이 줍니다. 첫번 째는 항상 카프카 커넥트 프로세스에 관한 설정입니다. 카프카 브로커를 연결시키고 데이타를 직렬화하는 포맷의 관한 공통 설정 파일 입니다. 나머지 설정 파일은 생성할 커넥터를 명시한 설정파일 입니다. 이 파일들은 유니크한 커넥트 이름과 시작할 커넥터 클래스와 커텍너가 필요한 설정들이 포함되어 있습니다.

```
$ bin/connect-standalone.sh config/connect-standalone.properties config/connect-file-source.properties config/connect-file-sink.properties
```

카프카에 포함 된 샘플 구성 파일은 앞의 예제에서 구성한 클러스터를 그대로 사용하고 두 개의 커텍트를 만듭니다. 첫번 재로 인풋 파일에서 한줄마다 읽어고 각각 카프카 토픽에 발행해주는 source 커넥터이고, 두번 째는 카프카 토픽에서 메세지를 읽어서 아웃풋 파일에 한줄바다 발행해주는 sink 커넥터 입니다. 

명렁어를 시작하면 많은 로그메세지들이 나타납니다. 커넥터가 시작되고있다는 메세지도 같이 나타납니다. 카프카 커넥터 프로세스가 한번 시작되면 source 커넥터는 test.txt의 파일을 읽고 connect-test 토픽에게 발행해줍니다. 그러면 sink 커넥터는 connect-test로부터 메세지를 읽고 test.sink.txt에 쓰기를 시작합니다. 

```
$ more test.sink.txt
foo
bar
```

데이터는 카프카 토픽인 `connect-test` 에 저장되고 있습니다. 이제 컨슈머를 콘솔을 열어 토픽안에 있는 데이터를 봐보겠습니다. 

```
$ bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic connect-test --from-beginning
{"schema":{"type":"string","optional":false},"payload":"foo"}
{"schema":{"type":"string","optional":false},"payload":"bar"}
```

커넥터가 데이터를 계속 처리하려면 파일에다가 데이터를 추가하게 되면 파이프라인에서 어떻게 움직이는지 볼 수 있습니다.

```
$ echo Another line>> test.txt
```

### 데이터 처리를 위한 카프카 스트림의 사용

카프카 스트림은 인풋 및 아웃풋 데이터가 카프카 클러스터에 저장되는미션 크리티컬한 실시간 애플리케이션 및 마이크로서비스를 구축하기 위한 클라이언트 라이브러리 입니다. 

카프카 스트림은 클라이언트 사이드에서 표준 자바 및 스칼라 애플리케이션을 작성 및 배포하는 단순함과 카프카의 서버사이드 클러스터 기술의 장점을 결합하여 애플리케이션의 확장성이 높아지고, 탄력성 및 장애내성 분산을 제공합니다. 



## 

## 참고자료

[카프카 공식문서](https://kafka.apache.org/documentation/#introduction)

[고승범님이 쓴 블로그](https://www.popit.kr/author/peter5236)







# Spring boot stream

스프링 부트 스트림에서의 용어 정리

![](./images/msa.png)

- SOURCE : 마이크로서비스가 메세지를 발행할 준비가 되면, source라는 놈을 이용해서 메세지를 발행한다. Source는 메세지 발행자와 메세지 목적지 사이의 인터페이스임. 
- CHANNEL : 채널은 메세지 생산자와 소비자가 메세지를 발행하거나 소비한 후 메세지를 보관할 큐를 추상화 한 것 채널이름은 설정에서 타겟이 된 큐의이름과 연관이 되어있음. 설정을 이용하면 큐이름을 변경하기가 쉽다. 메세지가 채널로 발행된다.
- BINDER : 바인더는 특정한 메세징 플랫폼(RabbitMq, Kafka 등)과 통신하기위한 것, 스프링 클라우드 스트림의 바인더를 사용하면 메세지를 발생하고 소비하기 위해 플랫폼마다 별도의 라이브러리와 API를 제공하지 않고도 메세징을 사용할 수 있다.
- SINK : 스프링 클라우드 스트림에서 서비스는 싱크를 사용해 큐에서 메세지를 받는다. 싱크는 들어오는 메세지를 위해 채널을 수신 대기하고, 메세지를 다시 POJO로 역직렬화한다. 이 과정에서 스프링 서비스의 비즈니스 로직이 메세지를 처리할 수 있다.

![](./images/binding.png)

@EnableBinding 어노테이ㅌ

spring-stream이  채널을 만든이유는 모지?

### Binder?

![img](https://saramin.github.io/img/binder.png)

- Spring Cloud Stream이 제공해 주는 미들웨어(kafka)와의 통신 컴포넌트.
- Spring이 @Configuration 정보를 읽어 미들웨어(kafka) 바인더를 구현체로 제공.
- 미들웨어(kafka)와 producer 및 consumer의 연결, 위임 및 라우팅 등을 담당.
- 따라서 Application에서는 미들웨어(kafka)와는 독립적으로 개발 진행 가능.
- 해당 properties는 spring.cloud.stream.binder에 해당.
  실제 kafka topic에 매핑하는 설정은 spring.cloud.stream.kafka.binder에 해당.

### Binding?

- 바인더의 입/출력을 미들웨어(kafka)에 연결하기 위한 Bridge(해당 바인더에서 만든 Pub/Sub역할).
- @EnableBinding을 사용하여 Application에서 사용할 바인더 인터페이스를 추가 바인딩이 적용됨.
- 해당 properties는 spring.cloud.stream.bindings.에 해당.
  실제 kafka topic에 매핑하는 설정은 spring.cloud.stream.kafka.bindings에 해당.