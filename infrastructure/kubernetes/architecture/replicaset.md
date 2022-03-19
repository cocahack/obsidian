# ReplicaSet

## 개요

Replication controller의 발전된 형태. Replication controller가 항상 지정한 숫자만큼의 파드가 항상 클러스터 안에서 실행되도록 관리하는데, ReplicaSet도 동일한 기능을 가지고 있다. 이 기능외에 집합 기반의 셀렉터를 지원하는 것이 차이점이다.

## Label & Selector

- Label: 쿠버네티스를 사용하다보면 수많은 파드들이 클러스터 내부에서 실행될 것이다. 이렇게 되면 수많은 파드 중에서 원하는 파드를 찾는 것은 쉽지 않을 것이다. 그러나 레이블을 사용하면 특정 파드를 쉽게 찾아낼 수 있다. 레이블은 파드에 key-value 쌍으로 달아둘 수 있다.
- Selector: 파드에 달아둔 label을 식별할 때 사용한다. 

### 템플릿 예시

```yaml 
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
  	app: myapp
	type: front-end
spec:
  template:
  	metadata:
	  name: myapp-pod
	  labels:
	    app: myapp
		type: front-end
	  spec:
	    containers:
		- name: nginx-container
		  image: nginx
  replicas: 3
  selector:
    matchLabels:
	  type: front-end
```

- `.spec.selector` 에 어떤 레이블 (.label)의 파드를 선택해서 관리할지를 적어주어야 한다. 레이블을 기준으로 파드를 관리하므로 실행 중인 파드를 중단하거나 재시작하지 않고 RelicaSet이 관리하는 파드를 변경할 수 있다. 
	- 따라서 처음 ReplicaSet을 생성할 때 `.spec.template.metadata.labels`의 하위 필드 설정과 `.spec.selector.matchLabels`의 하위 필드 설정이 같아야 한다.
	- `.spec.selector` 가 없으면 `.spec.template.metadata.labels.app`에 있는 내용을 기본값으로 설정한다. 

## Scale

레플리카 개수는 변경할 수 있다.

1. 템플릿 수정
	- 템플릿의 `replicas` 값을 수정한다.
	- `kubectl replace -f template.yml` 커맨드를 사용한다.
2. 커맨드로 수정
	- `kubectl scale --replicas=6 -f template.yml` 또는 
	- `kubectl scale --replicas=6 replicaset my-replica-set`