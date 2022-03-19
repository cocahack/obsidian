# 쿠버네티스 커맨드 정리

## 기본 명령어

| command              |        description        | further                                      |
|:-------------------- |:-------------------------:|:-------------------------------------------- |
| kubectl get          |     k8s의 리소스 조회     | `-o wide`로 더 자세한 정보                   |
| kubectl run POD_NAME |         POD 시작          | --image 옵션으로 이미지 지정                 |
| kubectl describe     | 리소스의 자세한 정보 출력 | kubectl describe RESOURCE_TYPE RESOURCE_NAME |
| kubectl edit         |    POD yaml 파일 수정     |                                              |
|                      |                           |                                              |

## 네임스페이스 관련

| command     |    description    | further                    |
|:----------- |:-----------------:|:-------------------------- |
| kubectl get | k8s의 리소스 조회 | `-o wide`로 더 자세한 정보 |



## 템플릿 

| command         |                       description                        | further                           |
|:--------------- |:--------------------------------------------------------:|:--------------------------------- |
| kubectl create  |        템플릿으로부터 오브젝트 또는 컨트롤러 생성        | `kubectl create -f template.yml`  |
| kubectl replace | 템플릿으로부터 생성한 오브젝트 또는 컨트롤러의 정의 수정 | `kubectl replace -f template.yml` |

