# 템플릿

## 기본 틀

```yaml 
apiVersion:
kind:
metadata:

spec:
```

- `apiVersion` : 쿠버네티스 API의 버전
- `kind`: 오브젝트 또는 컨트롤러를 명시
- `metadata`: 이름, 라벨 등을 설정
- `spec`: 파드가 어떤 컨테이너를 가지고 있으며, 실행할 때 어떻게 동작해야 할지 명시

### 예시 - ReplicationController

```yaml 
apiVersion: v1
kind: ReplicationController
metadata:
  name: myapp-rc
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
```

`ReplicationController` 컨트롤러를 명시한 템플릿의 예시이다. 

nginx 컨테이너를 가지는 파드를 세 개 생성하게 될 것이다. 템플릿으로 컨트롤러를 생성하는 방법은 다음과 같다.

```shell
$ kubectl create -f rep-ctrl-def.yml
```

### 예시 - [[replicaset]]

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

## 팁

- 이미지 이름을 변경하고 싶은 경우
  - edit 명령어로 변경 후 모든 파드를 제거하면 새로운 파드는 새롭게 지정한 이미지로부터 생성됨
  - 또는 레플리카세트를 제거한 뒤 템플릿 파일을 수정하고 다시 레플리카세트를 만든다.


---
#k8s 

