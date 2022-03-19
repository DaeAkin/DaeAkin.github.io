# 캐시 전략

캐싱을 하는 것은 시스템 성능을 올릴 수 있는 방법 중 하나다.

캐시를 제대로 구현하면, 응답 시간을 줄이거나, 데이터베이스 로드율을 줄여서 비용을 아낄 수 있다.

캐시를 사용하는 전략은 여러가지 있는데, 데이터 또는 데이터 접근 패턴에 따라 다르게 사용된다.

- 시스템이 쓰기 연산을 많이하지만 읽기 연산은 상대적으로 낮나? (시간 별로 로그를 찍는 작업 등)
- 데이터가 한번 쓰여지면 여러 번 읽을 수 있나?(유저 프로필 조회 등)
- 리턴하는 데이터가 항상 유니크한 데이터인가? (검색 등)



# Cache-Aside 전략

이 전략은 가장 많이 사용되는 전략이다.



1. 애플리케이션이 먼저 캐시를 확인한다.
2. 캐시에 데이터가 있다면, 그 데이터를 클라이언트에게 리턴한다.
3. 캐시에 없다면, 데이터베이스에 쿼리를 날려 데이터를 읽고, 클라이언트에게 리턴한다. 그리고 데이터를 캐시에 저장하여, 다음에 동일한 요청이 왔을 때, 캐시를 사용하도록 만들어준다.



## 장단점

Cache-aside 캐시는 일반적인 상황이나, **읽는 연산이 많이 할 때** 사용한다.

Memcached나 Redis가 주로 많이 사용 된다.

Cache-aisde를 사용하는 시스템은 **캐시 오류에 대해 회복력**을 갖고 있다. 

만약 캐시 클러스터가 죽게되면, 애플리케이션은 데이터베이스와 직접적으로 동작하기 때문이다.

다른 장점으로는 캐시의 데이터모델과, 데이터베이스의 데이터 모델이 다르다는 것이다. 

예를 들어, 생성된 여러 개의 쿼리의 대한 결과 응답을 특정 request id 캐시에 저장할 수 있다.

Cache-aside 전략의 쓰기 전략은 데이터베이스에 직접 데이터를 쓰는 것이다.

그렇게 되면 캐시하고 데이터베이스 간의 데이터 불일치가 일어날 수 있다.

이 문제를 해결하기 위해서는 개발자가 대게 TTL(time to live)를 설정하고 TTL이 만료 될 때 데이터를 삭제하는 방법을 사용 한다.

만약 최신의 데이터를 보장받아야 한다면, 다른 전략을 사용해야 한다.



# Read-Through Cache

Read-through 캐시는 캐시 미스가 발생하면 데이터베이스에서 조회하고, 캐시에 저장하고, 애플리케이션으로 리턴된다.

Cache-Aside와 Read-Through 두 전략 모두 데이터가 처음 읽혀야만 캐시에 적재되는 lazy 방법을 사용하고 있다.





## 장단점

read-through 전략과 cache-aside 전략은 비슷하지만, 몇 가지 차이점이 있다.

- Cache-Aside는 데이터를 데이터베이스에서 조회하고 캐시에 넣는 작업은 애플리케이션에게 책임이 있지만, read-through는 주로 지원되는 라이브러리나, 캐시 프로바이더를 이용한다.
- Cache-Aside와 다르게 Read-through는 캐시 데이터모델과 데이터베이스 모델이 같다.

Read-through는 같은 시간동안의 같은 데이터의 읽기 요청이 많을 때 적절하다.

단점으로는 데이터를 처음 요청했을 때 나타난다.

처음 요청은 항상 데이터가 캐시 미스가 나고, 데이터를 로딩하고, 캐시에 적재를 하기 때문에 추가적인 패털티가 존재한다.

그러므로 개발자는 수동으로 쿼리를 실행해서 캐시를 예열해서 이 문제를 처리하기도 한다.

cache-aside와 마찬가지로 데이터베이스와 캐시 사이의 데이터 불일치가 일어날 수 있다.



# Write-Through Cache

wrtie through은 데이터가 캐시에 쓰여진 다음 데이터베이스로 전달하는 방법이다.



## 장단점

데이터를 처음에 캐시에 먼저쓰고, 그 후 데이터베이스에 쓰기 때문에 추가적인 읽기 지연이 일어난다. 

하지만 read-through cache와 같이 사용한다면, read-through의 장점을 얻으면서, 데이터 불일치 문제가 해소 된다.

[DynamoDB Accelerator (DAX)](https://aws.amazon.com/ko/dynamodb/dax/) 이 read-through / write-through 캐시의 좋은 예다. 



# Write-Around

쓰기 연산은 데이터베이스로 직접가고, 읽은 데이터만 캐시에 적재한다.



## 장단점

Write-around can be combine with read-through and provides good performance in situations where data is written once and read less frequently or never. For example, real-time logs or chatroom messages. Likewise, this pattern can be combined with cache-aside as well.



# Write-Back