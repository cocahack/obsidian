# Replication

카프카는 고가용성을 위해 리플리케이션을 사용한다. 토픽에 `replication-factor`를 설정하면 그 개수만큼 같은 토픽이 여러 카프카 클러스터에 분산되어 생성된다.

## 리더와 팔로워

카프카는 내부적으로 동일한 리플리케이션을 리더와 팔로워로 구분하며, 리플리케이션 중 하나는 리더가 된다. 모든 읽기와 쓰기 요청은 리더로만 처리하게 된다. 

리더에 문제가 발생하면 팔로워 중 하나가 리더가 되어 요청을 처리하게 된다. 따라서 팔로워는 리더가 새로운 메시지를 받았는지 확인하고, 있다면 메시지를 리더로부터 복제해오는 작업을 계속 수행한다.

### 복제 유지와 커밋

리더와 팔로워는 ISR(In Sync Replica)라는 논리적인 그룹에 묶여 있다. 리더는 이 그룹 내에서만 선정된다. 

ISR과 관련된 사실을 나열해보면 다음과 같다.

- ISR 내 모든 팔로워는 리더를 따라가며, 리더는 ISR 내 모든 팔로워가 자신과 일치하는지 지속적으로 확인한다.
- 팔로워가 ISR에 속하기 위해서는 리더와 데이터가 일치해야 한다. 즉, ISR 내 팔로워는 언제든지 리더로 승격될 수 있는 후보인 것이다.
- ISR 내 모든 팔로워가 복제를 완료하면 리더는 내부적으로 커밋되었다는 표시를 한다. 이 때, 마지막 커밋 위치를 **하이 워터 마크(high water mark)**라고 한다.
- '커밋되었다'의 의미는 리플리케이션 팩터로 지정한 수만큼의 모든 리플리케이션이 메시지를 저장했다는 것이다.

#### 커밋 위치의 중요성

커밋 위치는 일관성을 유지하는데 중요한 역할을 한다. 

만약 커밋되기 전 메시지를 컨슈머가 읽을 수 있다면 어떤 일이 발생하는가?

컨슈머 A가 리더로부터 메시지 1과 2를 소비했고, 그 이후 리더에 문제가 발생하여 팔로워 중 하나가 새로운 리더로 선출되었다고 가정하자. 이전 리더는 메시지 1과 2를 가지고 있지만 실제로 커밋된 위치는 1이었다면, 새로 선출된 리더는 메시지 1만 가지고 있는 상태이다. 여기서 컨슈머 B가 새로운 리더로부터 메시지를 읽는다면 메시지 1만 소비하게 되는 현상이 발생하는 것이다.

위와 같은 문제때문에, 메시지는 항상 커밋된 메시지만 읽을 수 있는 것이다.

그리고 커밋된 위치도 매우 중요하기 때문에, 각 브로커의 로컬 디스크에 `replication-offset-checkpoint`라는 파일에 마지막 커밋 오프셋 위치를 저장한다.

### 복제 과정

1. 리더에 메시지가 저장된다. (리더 오프셋 0)
2. 팔로워들은 리더에게 fetch 요청을 보낸다. 이 시점에 리더는 어떤 팔로워가 어떤 메시지를 요청했는지 알 수 있게 된다. (팔로워 오프셋 0 또는 실패)
3. 다음 메시지가 리더에 저장된다. (리더 오프셋 1, 커밋 0)
4. 팔로워들은 다시 리더에게 fetch 요청을 보낸다. 이 시점에 리더는 팔로워들이 오프셋 0 임을 알게 된다.
5. 리더는 팔로워에게 메시지와 함께 오프셋 0의 메시지가 커밋되었다는 사실을 알린다.
6. 메시지를 받은 팔로워는 메시지와 커밋된 사실을 처리한다. (팔로워 오프셋 1, 커밋 0)
7. 이 과정이 계속 반복된다.

이러한 복제 과정은 리더와 팔로워 사이에 메시지를 성공적으로 복제했다는 ACK 통신을 줄일 수 있는 장점이 있다. 그래서 pull 방식 복제가 채택된 것이다. 

그러나 ACK 를 받지 않기 때문에 팔로워의 복제 여부는 팔로워가 다음 메시지를 복제하려는 요청이 들어올 때 알 수 있게 된다. 이러한 시간 차이로 인해 불일치 문제가 발생할 수 있으므로, 하이 워터 마크를 사용하여 ISR 내 모든 브로커가 가진 메시지만 컨슈머에게 제공하여 불일치 문제를 차단한다.

### 리더 장애 복구 

카프카의 파티션들이 복구 동작을 할 때 메시지의 일관성을 유지하기 위한 용도로 리더에포크(LeaderEpoch)를 사용한다.

리더에포크는 컨트롤러가 관리하는 32 비트의 숫자이다. 이 리더에포크 정보는 리플리케이션 프로토콜에 의해 전파되고, 새로운 리더가 변경된 후 변경된 리더에 대한 정보는 팔로워에 전달된다.

#### 팔로워 장애 복구

팔로워에 문제가 생겨 종료되었다가 다시 시작되었을 때, 리더에포크를 사용하여 리더와 메시지를 일치시킨다. 

팔로워는 재시작될 때 리더에게 리더에포크를 요청한다. 리더가 보내온 리더에포크 응답으로 자신의 하이워터마크보다 높은 오프셋을 확인하고 리더와 동일한 메시지로 맞춘다.

#### 리더 장애 복구

두 개의 복제를 가진 파티션에 리더와 팔로워가 동시에 다운되었다고 하자. 리더의 오프셋은 1이고, 하이워터마크도 1이다. 팔로워의 오프셋은 0이며 하이워터마크도 0이다.

이렇게 오프셋 한 개가 차이나는 상황에서 팔로워가 먼저 복구되면, 팔로워는 새로운 리더가 된다. 리더로 승격된 후 메시지가 새로 도착하여 오프셋이 1이 된다. 그러나 이전 리더의 오프셋 1 메시지와는 다른 메시지이다.

이전 리더가 복구되면 새로운 리더로 리더에포크를 요청한다. 새로운 리더는 자신이 팔로워일 때 하이워터마크와 리더로 승격된 후 하이워터마크를 모두 알고 있기 때문에, 승격된 이후 메시지들을 알 수 있다. 이를 통해 이전 리더는 자신의 메시지를 검사한다. 

오프셋 1의 메시지가 새로운 리더가 가진 메시지와 다른 것임을 알게 되면, 이전에 가지고 있던 오프셋 1의 메시지를 지우고 새로운 리더의 메시지로 사용한다.

---

정리하면, 리더가 변경되면 이전 리더가 가지고 있던 불일치 메시지는 현재 리더의 메시지로 맞춰지며, 이는 리더에포크를 사용한다는 것이다.



