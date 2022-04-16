# NoSQL

일부 특수한 사례에서는 RDBMS를 선택하기 어려울 수 있는데, 그럴 때 NoSQL을 선택하게 된다.  NoSQL을 선정하게 되는 대표적인 이유는 다음과 같다.

- Large-scale dataset이나 매우 높은 쓰기 처리량을 관계형 데이터베이스보다 쉽게 할 수 있는 뛰어난 확장성
- RDBMS에서 지원하지 않는 특수한 쿼리 동작
- RDBMS의 스키마 제약을 뛰어 넘어 동적이고 표현력이 풍부한 데이터 모델

NoSQL은 다시 네 가지 정도로 분류해볼 수 있다.

- Key-value store
- Graph store
- Column store
- Document store

## Key-value store

가장 많이 사용되는 자료 구조 중 하나인 해시 테이블 형태로 데이터를 저장할 수 있다. 데이터 조회는 키로 할 수 있으며, 키에 대응되는 값은 어떤 형태여도 상관없다.

가장 널리 쓰이는 key-value store에는 Redis가 있다. Redis는 모든 데이터를 메모리에 저장하므로 캐시로 많이 쓰이며, 그 외에 유저 세션 저장소 등으로 활용된다. 

또, Riak이라는 key-value store가 있다. Riak은 쉬운 확장성, 데이터 안정성, 샤딩과 일관성 모델을 감추는 등의 기능에 초점을 맞추고 있다. 이러한 기능을 제공하기 위해 디스크에 데이터를 쓰며, 이 때문에 Redis보다는 높은 레이턴시가 발생한다. 

## Graph store

관계를 저장하고 탐색하는데 유용한 저장소이다. 대표적인 적용 사례가 SNS 서비스이다. Meta, Line 등 SNS 서비스 회사들의 기술 블로그를 찾아보면 그들의 고민을 엿볼 수 있다.

- [TAO: The power of graph - META](https://m.facebook.com/nt/screen/?params=%7B%22note_id%22%3A10158791581867200%7D&path=%2Fnotes%2Fnote%2F&refsrc=deprecated&_rdr)
- [# LINE Timeline의 새로운 도전 1편 – 추천 컨텐츠 탐색을 위한 Discover와 새로운 구독 모델 Follow](https://engineering.linecorp.com/ko/blog/a-new-challenge-for-line-timeline-1/)


## Column store

RDBMS는 로우 기반의 데이터베이스이다. 어떤 대상을 테이블로 만들고, 그 대상이 가지는 속성을 컬럼으로 정의하며, 하나의 로우는 그 대상의 특정 인스턴스가 되는 것이다. 

그에 반해 column store는 데이터를 컬럼 단위로 저장한다. RDBMS를 예로 들면, 유저 테이블의 속성인 이름, 전화번호, 주소 등 각각의 속성이 별도의 테이블로 빠져나왔다고 할 수 있다.

이커머스 서비스를 기준으로 생각해보면, 주문, 결제 등 여러 시스템은 그 사용자 개인의 데이터이며, 다른 사용자의 데이터는 크게 중요하지 않다. 그러나 주문 내역을 바탕으로 통계를 내고, 그 결과를 바탕으로 의사 결정을 하거나 추천 시스템에 활용하는 등 유저에게 겉으로 드러나지 않고 회사 내부의 비즈니스에 활용되는 데이터는 사용자 한 명에 초점을 맞추는 것이 아니라 사용자 전체로부터 파생된 데이터를 써야하는 것이다. 

이럴 때 컬럼 기반의 데이터베이스가 유용하다. 통계 데이터를 내고자 할 때는 테이블 전체가 아닌 일부 컬럼만을 필요로 하는 경우가 대부분이기 때문이다. 

대표적인 column store는 다음과 같다.

- Apache Cassandra
- Apache HBase
- Bigtable

## Document store

이 유형의 저장소는 데이터를 문서(document) 단위로 저장한다. 이력서를 데이터로 나타내 그것을 저장하고, 이력서를 조회할 때는 그 사람의 이름으로 조회하는 등의 활용이 가능할 것이다.

이력서를 예로 들었는데, 이런 문서는 일종의 형식이 존재한다. 그러나 그 형식이 항상 같지는 않을 것이다. 자유 형식의 이력서로 예를 들면 이력서의 내용 자체는 비슷하겠지만, 그 형식은 서로 제각각일 것이다. 이런 성격 때문에, document store는 semi-structed data를 저장한다고 한다.

대표적인 document store로는 MongoDB가 있다.
