# 쿠버네티스 아키텍처

## 클러스터를 구성하는 컴포넌트

![k8s components](https://d33wubrfki0l68.cloudfront.net/2475489eaf20163ec0f54ddc1d92aa8d4c87c96b/e7c81/images/docs/components-of-kubernetes.svg)

쿠버네티스 클러스터는 크게 컨트롤 플레인과 노드로 나뉜다. 

컨트롤 플레인은 스케줄링과 같이 클러스터 전반에 걸친 사항을 결정하고, 클러스터 이벤트(예를 들어, `deployment` 의 `replicas` 가 충족되지 않아 새로운 파드를 시작해야 할 때) 를 감지하고 응답한다.

노드는 실행 중인 파드를 유지하고 쿠버네티스 런타임 환경을 제공한다.

### 컨트롤 플레인의 컴포넌트

- kube-apiserver: 쿠버네티스 API를 노출하는 API 서버이다.
- etcd: 클러스터의 모든 데이터를 key-value 형식으로 저장하는 백엔드 저장소.
- kube-scheduler: 파드를 실행할 노드를 선택해준다.
- kube-controller-manager: 컨트롤러 프로세스를 실행하는 컴포넌트. 논리적으로 각각의 컨트롤러는 별개의 프로세스이며, 복잡도를 줄이기 위해 단일 바이너리로 컴파일되어 단일 프로세스로 실행된다.
	- Node controller: 노드의 헬스체크를 담당
	- Job controller: 일회성 작업을 나타내는 작업(job) 객체를 감시한다. 그런 다음, 작업을 완료하기 위해 파드를 생성한다.
	- Endpoints controller: 엔드포인트 객체(즉, 서비스와 파드를 연결하는)가 위치한다.
	- Service account & Token controller:  새로운 네임스페이스를 위한 기본 계정과 API 엑세스 토큰을 생성한다.
- cloud-controller-manager: 클러스터를 클라우드 프로바이더 API와 연결해주고, 클라우드 플랫폼과 상호작용하는 컴포넌트를 클러스터에서만 상호작용하는 컴포넌트와 분리해준다.
	- Node controller: 노드가 응답을 멈춘 후 클라우드에서 제거되었는지 확인하기 위해 클라우드 프로바이더를 확인하는데 사용한다.
	- Route controller: 클라우드 인프라의 라우트 설정에 사용한다.
	- Service controller: 클라우드 프로바이더의 로드밸런서를 제어하는데 사용한다.

### 노드의 컴포넌트

- kubelet: 클러스터 내 노드마다 실행되는 에이전트. 파드 내 컨테이너가 실행될 수 있게 보장해준다.
- kube-proxy: 노드마다 실행되는 네트워크 프록시로 쿠버네티스 서비스 컨셉의 일부를 구현한다.
- container runtime: 컨테이너를 실행하는 책임이 있는 소프트웨어. k8s 1.20 버전 이후  [containerd](https://containerd.io/docs/) 가 기본 런타임이다.

### 애드온

애드온은 쿠버네티스 리소스(`DaemonSet`, `Deployment` 등)를 사용하여 클러스터 기능을 구현하는 파드이다. 애드온은 클러스터 수준의 기능을 제공하기 떄문에 `kube-system` 네임스페이스에 속한다.

대표적으로 쓰이는 애드온은 다음과 같다.

- Networking: 클러스터 안에 가상 네트워크를 구성해 사용할 때 사용하며, Calico, Celium 등 그 종류가 다양하다.
- DNS: 클러스터 안에서 동작하는 DNS 서버. 쿠버네티스 서비스에 DNS 레코드를 제공한다. 주로 사용하는 DNS 애드온은 `kube-dns`와 `CoreDNS`가 있다.
- 대시보드 애드온
- 컨테이너 자원 모니터링
- 로깅 

## 오브젝트와 컨트롤러

유저는 템플릿 등으로 쿠버네티스에 자원이 어떻게 되었으면 좋겠다라는 'desired state'를 정의하고 컨트롤러는 desired state와 현재 상태가 일치하도록 오브젝트들을 생성/삭제한다. 

- 오브젝트
	- [[pod]]
	- service
	- volume
	- [[namespace]]
- 컨트롤러
	- [[ReplicaSet]]
	- Deployment
	- StatefulSet
	- DaemonSet
	- Job

## References

[k8s components overview](https://kubernetes.io/docs/concepts/overview/components/)

---

#k8s



