---
aliases: [consistent hashing]
---
# Consistent hashing

해시 테이블 크기가 조정될 때 평균적으로 $\frac{k}{n}$ 개의 키만 재배치하는 해시 기술이다. 여기서 k는 키의 개수이고, n은 슬롯의 개수이다.

대부분의 해시 테이블에서 키와 슬롯의 매핑을 모듈러 연산으로 정의하기 때문에 슬롯 배열의 크기가 바뀌면 전체 키를 재배치해야 하는 문제가 생긴다(rehash).

Consistent hashing은 이런 문제를 방지할 수 있다.

## Concepts

### 해시 공간과 해시 링

해시 공간(hash space)은 해시 함수 출력 값의 범위를 말한다. 예를 들어, SHA-1의 해시 공간은 0부터 $2^{160}-1$  이다. 

이 해시 공간의 양 끝을 구부려 접는다고 생각하면, 해시 링(hash ring)이 만들어진다.

### 서버와 캐시할 키의 배치

서버는 서버 IP나 이름으로 해시 값을 만들어는 것으로, 캐시할 키는 키 자체를 해시 함수를 이용하여 해시 값을 만들어 해시 공간에 배치할 수 있다. 

![Consistent hash](https://miro.medium.com/max/684/1*WaTV_sMa0q_ke-G1B6_1Wg.png)

### 서버 조회

어떤 키가 배치될 서버는 해시 링을 기준으로 봤을 때 해당 키의 위치로부터 시계 방향으로 전진했을 때 가장 처음 만나게 되는 서버이다.

### 서버 추가

서버가 추가될 경우, 일부 키는 새로 추가된 서버로 이동한다. 전체 키가 움직이는 것이 아님에 주목해야 한다. 이동하는 키들은 새로 추가된 서버가 현재 할당된 서버보다 가까이 있기 때문에 이동하게 되는 것이다.

### 서버 삭제

서버가 삭제되면 그 서버에 있던 키들은 다른 서버로 이동한다. 서버 조회 방법과 동일하다.

## 알고리즘

Consistent hash 알고리즘은 MIT에서 처음 제안했는데, 그 기본 절차는 다음과 같다.

- 서버와 키를 균등 분포(uniform distribution) 해시 함수를 사용하여 해시 링에 배치한다.
- 키의 위치에서 링을 시계 방향으로 탐색하다 만나는 최초의 서버가 키가 저장되는 서버이다.

이 방법에는 두 가지 문제가 있다. 

1. 서버가 추가되거나 삭제되는 상황을 감안할 때, 파티션의 크기가 균등하게 유지될 수 없다.
2. 키를 균등하게 배치하는 것이 어렵다.

이를 해결하기 위해 제안된 방법은 가상 노드(Virtual Node) 또는 복제(replica) 라는 방법이다.

### 가상 노드(Virtual Node)

가상 노드는 실제 노드 또는 서버를 가리키는 노드로, 하나의 서버는 링 위에 여러 개의 가상 노드를 가질 수 있다. 아래 예시는 서버 당 세 개의 가상 노드를 가질 때를 도식화한 것이다.

![virtual node](https://1865312850-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-M4Bkp-b8HYQgJF1rkOc%2F-M5FwA4YIBAqAjZvdVpU%2F-M5FyQai2CtC2j4GGr5K%2Fimage.png?alt=media&token=77d5d346-f37f-4f28-8f64-66a1627d2deb)

가상 노드의 개수가 많아질 수록 키의 분포는 점점 균등해진다. 표준 편차가 작아져서 데이터가 고르게 분포되기 때문이다. 

한 블로그 글[^1]에 따르면 가상 노드의 개수가 100~200개 사이일 때 표준 편차 값은 평균의 5%~10% 사이라고 한다. 가상 노드의 개수를 늘리면 표준 편차는 더 떨어지지만, 가상 노드 데이터를 저장할 공간이 더 많이 필요하므로 이는 trade-off 요소라고 할 수 있다.

## 장단점

- 장점
	- 서버가 추가되거나 삭제될 때 재배치되는 키의 수를 최소화할 수 있다.
	- 데이터가 보다 균등하게 배포되므로 수평적 확장이 용이하다.
	- 핫스팟 키 문제를 줄힌다. 데이터를 균등하게 분배할 수 있기 때문이다.
- 단점
	- 노드가 일시적으로 다운되었을 때의 키 관리 비용은 비쌀 수 있다. [^2]

## Success stories

### Discord[^3]

Discord는 consistent hashing을 사용하여 분산 시스템을 구축했다. 연결되는 세션을 노드에 고르게 분포시키기 위한 도구로 사용했을 것으로 추측된다. 처음에는 Ring 자료 구조를 사용하기 위해 C로 작성된 [hash-ring](https://github.com/chrismoos/hash-ring) 프로젝트를 Erlang C Port를 사용했었다. 그러나 세션을 재연결할 때 링을 검색하는 시간이 너무 길어(30초) 문제가 있었다고 한다. hash-ring 프로젝트를 Erlang으로 마이그레이션하고 서로 다른 프로세스가 ring에서 데이터를 넣고 가져오는 작업에서 발생하는 데이터 복사 과정을 제거하여 750ms로 최적화했다고 한다.

[https://github.com/discordapp/fastglobal](https://github.com/discordapp/fastglobal)


[^1]:https://tom-e-white.com/2007/11/consistent-hashing.html
[^2]: https://people.eecs.berkeley.edu/~kubitron/courses/cs262a-F12/lectures/lec22-Dynamo.pptx
[^3]: https://discord.com/blog/how-discord-scaled-elixir-to-5-000-000-concurrent-users











