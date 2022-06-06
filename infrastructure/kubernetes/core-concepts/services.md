# 서비스

동적으로 위치가 바뀌는 파드에 고정적으로 접근할 때 사용하는 컴포넌트이다.

## 인프라 관점에서 바라보기

쿠버네티스 노드 안에 존재하는 파드를 쿠버네티스 바깥에서 접근하기 위해서 필요한 것이 바로 서비스이다.

![[Pasted image 20220605225539.png]]

위의 그림처럼, 192.168.1.0 서브넷에 속한 클라이언트가 K8S Node 내의 파드에 직접 접근할 수가 없다. 접근하려면 서비스를 만들면 된다. 서비스를 만들게 되면 다음과 같이 접근할 수 있다.

![[Pasted image 20220606090642.png]]

## 서비스 타입

- NodePort: 노드 내부의 파드에 접근할 수 있는 포트를 제공하는 방식. 이전의 그림에서 사용했던 서비스의 타입이 바로 NodePort이다.
- Cluster IP: 클러스터 내부에 Virtual IP를 만들어 프론트 엔드 서버 등 다른 서비스와 통신할 수 있게 해준다.
- LoadBalancer: 클라우드 프로바이더가 제공하는 로드밸런서를 사용한다.
- ExternalName: 서비스를 `.spec.externalName` 필드에 설정한 값과 연결한다. 클러스터 안에서 외부에 접근할 때 사용한다. 이 서비스는 설정해둔 CNAME 값을 이용해 클러스터 외부에 접근하는 방식을 취한다. 클러스터 외부에 접근할 때 사용하므로 셀렉터(`.spec.selector` 필드)를 쓰지 않는다. 

### NodePort

![[Pasted image 20220606161748.png]]

NodePort에서 명시해야 하는 요소는 TargetPort, Port, NodePort 세 가지이다. 이를 정의한 yaml 파일의 예시는 다음과 같다.

```yaml 
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  ports:
    - targetPort: 80
	  port: 80
	  nodePort: 30000
  selector:
    app: myapp
    type: front-end
```

`.spec.ports[]` 에서 `port` 값은 반드시 있어야 한다. `targetPort`를 명시하지 않으면 `port` 와 같은 값이라고 간주한다. `nodePort`를 명시하지 않으면 30000~32767 사이의 사용하지 않는 포트 중 하나가 자동으로 할당된다.

서비스는 여러 노드에 흩어져 있는 포트에 할당되어야 하므로, `selector`를 사용하여 명시한다.

한 노드에 여러 파드가 있을 때 `NodePort` 서비스를 적용하고 싶다면, 추가적으로 해야할 일은 없다. 서비스를 정의하고 `selector`를 사용하여 파드를 선택하면 된다. 이 때 서비스로 들어온 요청은 자동으로 부하 분산된다.

파드가 여러 노드에 걸쳐 있을 때 서비스를 적용하면, 서비스도 여러 파드에 걸쳐 생성된다. 각 노드마다 서비스가 생성되는 것이다. 예를 들어, 각각 192.168.1.2, 192.168.1.3 IP를 가진 노드에 서비스를 생성하면 각각의 IP와 서비스에 지정된 nodePort 로 접근할 수 있다.

### Cluster IP

프론트엔드에서 scale out 되어 있는 백엔드 서버에 접근한다고 가정해보자. 각 백엔드 서버는 서로 다른 IP를 가지고 있을 것이며, 클라우드 환경이라면 그 IP는 고정된 IP가 아닐 것이다. 이런 상황에서 프론트엔드 애플리케이션은 백엔드 서버에 접근하기 위한 정보를 관리하기가 쉽지 않다. 

쿠버네티스에서는 서비스를 생성하면 파드를 그룹으로 묶어 단일 IP 주소로 접근하는 것이 가능하다. 단, 이 IP는 클러스터 안 노드나 파드에서만 접근할 수 있고, 클러스터 외부에서는 접근할 수 없다.

yaml 파일 예시는 다음과 같다.


```yaml 
apiVersion: v1
kind: Service
metadata:
  name: back-end
spec:
  type: ClusterIP
  clusterIP: 10.0.10.10
  ports:
    - targetPort: 80
	  port: 80
  selector:
    app: myapp
    type: front-end
```

`clusterIP` 로 직접 IP를 할당할 수 있다. 값을 설정하지 않으면 쿠버네티스에서 자동으로 할당한다.

### LoadBalancer

여러 노드에 걸쳐 NodePort를 생성했다면, 단일 진입점이 없기 때문에 접근하기 힘들 뿐만 아니라, 부하분산도 쉽지 않다. 이런 상황에서 LoadBalancer 타입의 서비스를 사용할 수 있다. 이 때, 클라우드 프로바이더가 제공하는 로드밸런서를 사용해야 한다.

yaml 파일 예시는 다음과 같다.


```yaml 
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: LoadBalancer
  ports:
    - targetPort: 80
	  port: 80
	  protocol: TCP
  selector:
    app: myapp
    type: front-end
```
