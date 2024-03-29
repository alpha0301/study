# 리소스 관리

## 구성

- CPU, 메모리 등등
- 파드를 지정할 때 컨테이너에 필요한 각 리소스 양을 선택적으로 지정 가능
- 파드를 실행한 노드의 리소스가 충분하면 limit 설정 범위 내로 컨테인너는 더 많은 리소스를 사용 가능

```
ex) 컨테이너 메모리 설정: 256M, 노드 메모리: 8G, limit 4G 설정인 경우
-> 4G 초과 사용 시 OOM 오류와 함께 할당을 시도한 프로세스를 종료
-> limit 옵션으로 초과 시 반응하거나 초과하지 못하도록 강제화할 수 있음
```

## 리소스 타입

- CPU, 메모리는 각각 리소스 타입이다.
- CPU 기본 단위: k8s CPU 단위로 지정됨
- 메모리 기본 단위: 바이트
- 리눅스 워크로드는 huge page 리소스를 기정할 수 있다.
- API 리소스와 헷갈리지 않도록 주의해야 함
