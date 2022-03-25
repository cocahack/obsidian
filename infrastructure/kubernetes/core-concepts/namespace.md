# Namespace

네임스페이스를 사용해 쿠버네티스 클러스터를 논리적인 단위로 나눠서 사용할 수 있다.

## 기능
- 격리: 한 네임스페이스의 자원은 다른 네임스페이스의 자원과 완전히 격리된다.
- 사용량 제한: 네임스페이스마다 quota를 정할 수 있다(아래 [Quota 지정하기](#Quota%20지정하기) 참조). 
- DNS: DNS 규칙에 의해 특정 네임스페이스에 속한 서비스에 접근할 수 있다.
	- default 네임스페이스의 경우, 서비스 이름으로 호스트 이름이 할당된다.
	- 그 외 다른 네임스페이스의 경우, 다음과 같이 호스트 이름이 할당된다.
		- dev 네임스페이스의 db-service 서비스에 접근하고 싶다면, `db-service.dev.svc.<cluster_domain>` 으로 접근할 수 있다.

## Default namespaces

쿠버네티스 시스템이 사용하는 네임스페이스와 기본 네임스페이스가 최초 쿠버네티스 구성 시 만들어진다.

- default: 기본 네임스페이스로, 별다른 설정을 하지 않는다면 항상 기본 네임스페이스를 사용한다.
- kube-system: 쿠버네티스 관리용 파드나 설정이 생성되는 곳
- kube-public: 클러스터 안 모든 유저가 읽을 수 있는 네임스페이스. 보통 클러스터 사용량 같은 정보를 여기서 관리한다.
- kube-node-lease: 각 노드의 lease object들을 관리하는 네임스페이스

## 네임스페이스 사용하기

### 명령어

각 명령어의 옵션으로 네임스페이스를 지정하거나, 전체 네임스페이스를 대상으로 명령어를 실행할 수 있다.

```shell
$ kubectl get pods --namespace=kube-system
$ kubectl get pods -n kube-system
$ kubectl get pods --all-namespaces
```

### 템플릿

`.metadata.namespace` 가 키 이름이다. 

#### quota 지정하기

```yaml
apiVersion: v1
kind: resourceQuota
metadata:
    name: compute-quota
	namespace: dev
spec:
    hard:
	    pods: "10"
		requests.cpu: "4"
		requests.memory: 5Gi
		limits.cpu: "10"
		limits.memory: 10Gi

```

특정 네임스페이스의 샤용량을 제한할 수 있다. 위의 템플릿은 `dev` 네임스페이스가 최소 4 CPU, 5GB 메모리를 가져야하며, 최대 10개의 pod를 생성할 수 있고, 최대 10 CPU, 10GB 메모리를 가질 수 있다는 뜻이다.

### 네임스페이스 변경하기

특정 네임스페이스에서 계속 작업해야 한다면 `--namespace` 플래그를 계속 넣어주는 것이 상당히 불편할 것이다. 컨텍스트를 변경하여 기본 네임스페이스를 바꿀 수 있다.

```shell
$ kubectl config set-context $(kubectl config current-context) --namespace=dev
```