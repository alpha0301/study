# k8s 오브젝트


## 개요

- k8s 시스템에서 persistence를 가진다.
- 클러스터의 상태를 나타내기 위해 사용한다.
  - 어떤 컨테이너화된 애플리케이션이 어느 노드에서 동작 중인지
  - 애플리케이션이 사용할 수 있는 리소스
  - 애플리케이션 재구동 정책, 업그레이드, 내고장성에 대한 정책 등
  - 내고장성: 일부가 고장나도 전체 시스템은 정상 작동을 유지하는 능력

## 라이프사이클

- k8s는 오브젝트에 대한 생성 보장을 위해 지속적으로 작동하게 됨
- 관리자는 오브젝트 정의를 통해 클러스터의 워크로드 형태를 k8s에 전달하게 됨
- 오브젝트를 동작하게 하려면 쿠버네티스 API를 사용해야 함(ex: kubectl)

## 명세

### spec

- 오브젝트가 사용할 리소스에 대한 명세

### status

- 오브젝트의 현재 상태
- k8s는 status, spec 간의 차이를 없에고 일치하도록 관리

## k8s deploy

- 클러스터에서 동작하는 애플리케이션을 표현해줄 수 있는 오브젝트
- spec 정의를 통해 레플리카 숫자를 결정할 수 있다.

## 오브젝트 기술하기

- 오브젝트 기본 정보, 의도한 상태를 기술한 spec 제시
- 필드
  - apiVersion: 쿠버네티스 API 버전
  - kind: 오브젝트 종류
  - metadata: 이름, UID, namespace(option) 등의 식별자 데이터
  - spec: 의도하는 상태 정보
    - spec 구성은 오브젝트마다 모두 다름

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # tells deployment to run 2 pods matching the template
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

- kubectl 명령어로 deployment 생성할 수 있다.

```shell
kubectl apply -f sample.yaml --record
>>> deployment.apps/nginx-deployment created
```

