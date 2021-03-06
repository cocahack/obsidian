# 키-값 저장소 설계

#db #CAP_Threory #non-relational


키-값 저장소(key-value store)는 키-값 데이터베이스라고 불리는 비 관계형 데이터베이스로, 저장되는 값은 고유 식별자를 키로 가져야 한다. 키와 값 사이의 이런 연결 관계를 key-value pair라고 부른다.

## 분산 키-값 저장소

키-값 쌍을 여러 서버에 분산시키는 구조의 저장소라면 분산 키-값 저장소라고 할 수 있다. 이런 시스템을 설계할 때 따라다니는 이론이 CAP 이론이다.

### CAP 이론

CAP는 아래 세 가지 요소를 가리킨다.

- Consitency: 분산 시스템에 접속하는 모든 클라이언트는 어떤 노드에 접근하는 것과 관계없이 언제나 같은 데이터를 봐야 한다.
- Availability: 분산 시스템에 접근하는 클라이언트는 일부 노드에 장애가 발생해도 응답을 받을 수 있어야 한다.
- Partition Torelance: 두 노드 사이에 통신 장애가 발생하여 파티션이 생겨도 시스템은 계속 동작해야 한다.

그리고 CAP 이론은 뜻하는 바는 세 가지 요소를 모두 만족하는 시스템은 존재할 수 없다는 것이다.

> [!caution]
> CAP 이론은 세 가지 요소 중 두 가지 요소를 고르라는 것처럼 표현되기도 한다. 그러나 이는 잘못된 것이다. 네트워크 단절로 인해 발생하는 파티션은 일종의 결함이므로 선택할 수 있는 것이 아니기 때문이다. 
> CAP 이론은 **파티션이 생겼을 때 일관성과 가용성 중 하나를 선택하라**는 의미로 보는 것이 더 좋다.

### 분산 시스템을 설계하는 면접 질문에 대비

분산 시스템에서는 CP 시스템 또는 AP 시스템을 선택해야 한다. 따라서 면접 자리에서 이러한 질문을 받았다면 면접관와 상의하여 그 결론에 따라 시스템을 설계해야 한다.

## 분산 키-값 저장소의 컴포넌트들

DynamoDB, Cassandra, BigTable 의 사례를 참고하여 핵심 컴포넌트를 정리하였다.

### 데이터 파티션

대규모 애플리케이션이라면 전체 데이터를 한 서버에 몰아 넣을 수가 없다. 가장 단순한 해결책은 작은 파티션으로 분할한 다음 여러 대의 서버에 저장하는 것이다. 이 때, 데이터 파티션을 나눌 때 두 가지 항목을 잘 따져봐야 한다.

- 데이터를 여러 서버에 고르게 분산할 수 있는가?
- 노드가 추가되거나 삭제될 때 데이터의 이동을 최소화할 수 있는가?

이 문제는 [[consistent-hashing|consistent hashing]] 으로 풀 수 있다. 안정 해시를 사용하여 데이터를 파티션하면 얻는 장점은 다음과 같다.

- 규모 확장 자동화: 시스템 부하에 따라 서버가 자동으로 추가되거나 삭제되도록 만들 수 있다.
- 다양성: 각 서버의 용량에 맞게 가상 노드의 수를 조절할 수 있다.

### 데이터 다중화

안정 해시를 사용하는 시스템에서 데이터 다중화는 링을 순회하면서 만나는 첫 $N$개의 노드에 데이터를 보관하여 달성할 수 있다.

그런데 가상 노드를 사용한다면 선택한 $N$개의 노드에 대응될 실제 물리 서버의 개수가 $N$보다 작아질 수도 있다. 이 문제를 피하려면 노드를 선택할 때 같은 물리 서버를 중복하여 선택하지 않도록 해야 한다. 

### 데이터 일관성

여러 노드에 다중화된 데이터는 적절히 동기화가 되어야 한다. Quorum consensus 프로토콜을 사용하면 읽기/쓰기 연산에 일관성을 보장할 수 있다. 

이와 관련된 정의가 있다.

- $N$: 데이터 사본 개수
- $W$: 쓰기 연산에 대한 정족수. 쓰기 연산이 성공하려면 최소 $W$개의 서버로부터 쓰기 연산이 성공했다는 응답을 받아야 한다.
- $R$: 읽기 연산에 대한 정족수. 읽기 연산이 성공하려면 최소 $R$개의 서버로부터 읽기 연산이 성공했다는 응답을 받아야 한다.

$W + R > N$ 인 경우, 강한 일관성이 보장된다고 한다. 일관성을 보증할 최신 데이터를 가진 노드가 최소 한 개는 있을 것이기 때문이다. 

이처럼 분산 시스템에서는 N과 W, 그리고 R을 적절히 선택하는 트레이드 오프를 결정해야 한다. 몇가지 예시를 들면 다음과 같다.

- $R = 1, W = N$: 빠른 읽기 연산에 최적화된 시스템
- $W = 1, R = N$: 빠른 쓰기 연산에 최적화된 시스템
- $W + R > N$: 강한 일관성이 보장됨(보통 $N = 3, W = R = 2$)
- $W + R \leq N$: 강한 일관성이 보장되지 않음

### 일관성 모델

- 강한 일관성: 모든 읽기 연산은 가장 최근에 갱신될 결과를 반환하지 못한다. 즉, 클라이언트는 항상 최신 데이터만 본다.
- 약한 일관성: 읽기 연산은 가장 최근에 갱신된 결과를 반환하지 못할 수 있다.
- 최종 일관성: 약한 일관성의 한 형태로, 갱신 결과가 언젠가는 모든 사본에 반영된다.

### 일관성 불일치 해소

데이터를 다중화하면 가용성은 높아지나 사본 간 일관성이 깨질 가능성도 높아진다. **벡터 시계**는 이를 해결하기 위한 방법이다. 

#### 벡터 시계

벡터 시계는 [서버, 버전]의 순서쌍을 데이터에 달아 놓은 것이다. 표현법을 일반화하면 다음과 같다.

$$D([S_1, v_1], [S_2, v_2], \dots, [S_n, v_n])$$

데이터 $D$를 서버 $S_i$에 기록하면, 시스템은 아래 작업 중 하나를 수행해야 한다.

- $[S_i, V_i]$가 있으면 $v_i$를 증가시킨다.
- 없다면 새 항목 $[S_i, 1]$을 만든다.

벡터 시계를 사용했을 때 버전 판별 방법과 충돌 여부 판단 방법은 각각 다음과 같다.

- 버전 X와 버전 Y 중 어떤 버전이 이전 버전인지 확인하려면, 버전 Y에 포함된 모든 구성요소의 값보다 같거나 큰지 보면 된다.
- 버전 X와 Y 사이에 충돌이 있는지 확인하려면, Y의 백터 시계 구성요소 가운데 X의 벡터 시계 동일 서버 구성요소보다 작은 값을 갖는 것이 있는지 보면 된다. 단, 모든 서버 구성요소가 X보다 작다면 이는 충돌이 아니라 낮은 버전이다.

벡터 시계를 사용했을 때의 단점은 다음과 같다.

- 충돌 감지 및 해소 로직이 클라이언트에 들어가야 하므로, 클라이언트 구현이 복잡하다.
- [서버, 버전] 순서쌍 개수가 굉장히 빨리 늘어난다. 이를 해결하려면 순서쌍 개수의 임계치를 설정하고, 그 이상 길어지면 가장 오래된 순서쌍을 벡터 시계에서 제거하는 방법을 써야 한다. 이 떄 버전 간 선후 관계를 정확히 알 수 없는 문제가 발생할 수 있다.
	- 그러나 DynamoDB에 관계된 문헌에 의하면 아마존에서 위와 같은 상황에 직면한 적은 없다고 한다

### 장애 처리

#### 장애 감지

분산 시스템에서는 적어도 두 대 이상의 서버가 똑같이 어떤 서버 A의 장애를 보고해야 실제로 장애가 발생했다고 간주한다.

위와 같은 장애 감지 시스템을 구현하는 가장 단순한 방법은 모든 노드 사이에 멀티 캐스팅 채널을 구축하는 것이지만, 서버가 많다면 매우 비효율적이다. 그래서 **가십 프로토콜(Gossip protocol)**과 같은 분산형 장애 감지 솔루션을 채택하는 것이 좋다.

가십 프로토콜의 동작 원리는 다음과 같다.

- 각 노드는 멤버십 목록(membership list)을 유지한다. 멤버십 목록은 멤버 ID와 그 멤버의 하트비트 카운터 쌍의 목록이다.
- 각 노드는 주기적으로 하트비트 카운터를 증가시킨다.
- 각 노드는 무작위로 선정된 노드들에게 주기적으로 자신의 하트비트 카운터 목록을 보낸다.
- 하트비트 카운터 목록을 받은 노드는 멤버십 목록을 최신 값으로 갱신한다.
- 어떤 멤버의 하트비트 카운터 값이 지정된 시간 동안 갱신되지 않으면 해당 멤버는 offline 상태로 간주한다.


#### 일시적 장애 처리

가십 프로토콜로 장애를 감지한 시스템은 가용성을 보장하기 위해 필요한 조치를 해야 한다. 엄격한 정족수(strict quorum) 접근법을 쓴다면 읽기와 쓰기 연산을 금지해야 하고, 느슨한 정족수(sloppy quorum) 접근법은 이 조건을 완화하여 가용성을 높힌다. 

느슨한 정족수 접근법은 정족수 요구사항을 강제하는 대신, 쓰기 연산을 수행할 $W$개의 정상 서버와 읽기 연산을 수행할 $R$개의 정상 서버를 해시 링에서 고른다. 이 때 장애 상태의 서버는 무시한다.
네트워크나 서버 문제로 장애 상태인 서버로 가는 요청은 잠시동안 다른 서버가 맡아 처리한다. 그동안 발생한 변경사항은 해당 서버가 복구되었을 때 일괄 반영하여 데이터 일관성을 보존한다. 이를 위해 임시로 쓰기 연산을 처리한 서버에는 그에 관한 힌트를 남겨두는데, 이런 장애 처리 방안을 **hinted handoff**라고 한다. 

#### 영구 장애 처리

영구적인 노드의 장애 상태는 **anti-entropy** 프로토콜을 구현하여 사본들을 동기화한다. 이 프로토콜은 사본들을 비교하여 최신 버전으로 갱신하는 과정도 포함되어 있다. 사본 간 일관성이 망가진 상태를 탐지하고 전송 데이터의 양을 줄이기 위해 **머클(merkel) 트리**를 사용할 수 있다.


## 분산 키-값 저장소 시스템 아키텍처 다이어그램 

- 클라이언트는 키-값 저장소가 제공하는 `get(key)`와 `put(key, value)` 두 가지 API를 사용한다.
- 코디네이터는 클라이언트에게 키-값 저장소에 대한 프록시 역할을 하는 노드이다.
- 노드는 안정 해시의 해시 링 위에 분포한다.
- 노드를 자동으로 추가 또는 삭제할 수 있도록, 시스템은 완전히 분산된다.
- 데이터는 여러 노드에 다중화된다.
- 모든 노드가 같은 책임을 지므로, SPOF는 존재하지 않는다.

완전히 분산된 아키텍처이므로, 노드는 다음의 기능을 모두 지원해야 한다.

- 클라이언트 API
- 장애 감지
- 데이터 충돌 해소
- 장애 복구 메커니즘
- 다중화
- 저장소 엔진

### 쓰기 경로 (by Cassandra)

1. 쓰기 요청이 커밋 로그 파일에 기록된다.
2. 데이터가 메모리 캐시에 기록된다.
3. 메모리 캐시가 가득차거나 사전에 정의된 어떤 임계치에 도달하면 데이터는 디스크에 있는 SSTable에 기록된다.

### 읽기 경로

데이터가 메모리 캐시에 있는 경우에는 캐시에서 즉시 데이터를 가져와 반환한다. 그렇지 않은 경우는 디스크에서 가져와야 하는데, 어느 SSTable에 찾는 키가 있는지 알아낼 효율적인 방법이 필요하다. 이 때 **Bloom Filter**가 흔히 사용된다. 블룸 필터가 사용된 읽기 경로는 다음과 같다.

1. 데이터가 메모리에 있는지 검사하고, 있으면 즉시 반환한다.
2. 데이터가 메모리에 없으므로, 블룸 필터를 검사한다.
3. 블룸 필터를 통해 어떤 SSTable에 키가 보관되어 있는지 알아낸다.
4. SSTable에서 데이터를 가져온다.
5. 해당 데이터를 반환한다.


