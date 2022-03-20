# Deployment

## 개요

쿠버네티스에서 stateless 앱을 배포할 때 샤용하는 가장 기본적인 컨트롤러.

ReplicaSet을 관리하면서 부가적인 기능이 더 들어가 있다. 

- 롤링 업데이트
- 앱 배포 도중 일시 중단
- 중단했던 배포 재개
- 이전 버전으로 롤백

## 템플릿 예시

```yaml 
apiVersion: apps/v1
kind: Deployment
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

ReplicaSet의 템플릿 예시에서 `.kind` 의 값을 `Deployment`으로 변경하기만 하면 된다.


---
#k8s 