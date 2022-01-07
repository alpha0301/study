# 경로 표현 전략

[참고 링크](https://stackoverflow.com/questions/49366042/rest-api-architecture-how-to-represent-joined-tables)

## join 의 경우

```
GET /agents/1/policies
```

본질적으로 중복이 발생하는 형태. 결국 최종 목표인 정책을 찾는다면

```
GET /policy/{id}
```

이어야 함. 정책에 집중해서 agent 를 검색조건으로 생각한다면 다음과 같이 치환할 수 있다.

```
GET /policies?agent=1
```

response 는 아래와 같다.

```json
{"policies": ["/policy/234234", "/policy/383383"]}
```

정책에 다양한 도메인의 리소스들의 정보가 필요하다면 그것도 id 를 제공해서 해결할 수 있다.

```
GET /policy/234234?filter=agentName,policyStatus,vehicleYear
```

```json
{"policy": {"...."}, "customer": "/customer/2888", "vehicle": "/vehicle/9392", "agent": "/agent/32"}
```

request 횟수가 매우 늘어날 수는 있지만 기본적으로 각 리소스의 상세 내용을 가져오는 것은 클라이언트의 책임이다.
이렇게 함으로써 REST의 많은 목표들... 캐싱이나 Idempotency 등을 효과적으로 가져올 수 있다.
클라이언트는 로컬 캐시에서 각 리소스들을 처리할 수 있는지 서버로부터 받아야 하는지 효율적으로 결정할 수 있다.


