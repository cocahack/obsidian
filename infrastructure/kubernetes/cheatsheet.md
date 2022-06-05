# 쿠버네티스 커맨드 정리

## 들어가기 전에

쿠버네티스는 imperative, declarative 둘 다 지원한다. 쿠버네티스가 관리하는 자원에 대한 CRUD 명령어도 지원하지만, 이를 선언적으로 정의한 후 적용하는 커맨드도 있는 것이다.

선언적인 방법은 YAML 파일을 수정한 뒤 `kubectl apply -f some-resource.yml` 명령어를 사용하면 된다.

## 기본 명령어

| command              |        description        | further                                      |
|:-------------------- |:-------------------------:|:-------------------------------------------- |
| kubectl get          |     k8s의 리소스 조회     | `-o wide`로 더 자세한 정보                   |
| kubectl run POD_NAME |         POD 시작          | --image 옵션으로 이미지 지정                 |
| kubectl describe     | 리소스의 자세한 정보 출력 | kubectl describe RESOURCE_TYPE RESOURCE_NAME |
| kubectl edit         |    POD yaml 파일 수정     |                                              |


## 네임스페이스 관련

| command     |    description    | further                    |
|:----------- |:-----------------:|:-------------------------- |
| kubectl get | k8s의 리소스 조회 | `-o wide`로 더 자세한 정보 |



## 템플릿 

| command         |                       description                        | further                           |
|:--------------- |:--------------------------------------------------------:|:--------------------------------- |
| kubectl create  |        템플릿으로부터 오브젝트 또는 컨트롤러 생성        | `kubectl create -f template.yml`  |
| kubectl replace | 템플릿으로부터 생성한 오브젝트 또는 컨트롤러의 정의 수정 | `kubectl replace -f template.yml` |

## Deployment

| command                                       | description |
| --------------------------------------------- | ----------- |
| kubectl create deployment --image=nginx nginx |             |
| kubectl expose deployment nginx --port 80     |             |
|                                               |             |
