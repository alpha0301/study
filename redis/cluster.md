# Redis Cluster

## goals

- 최대 1000개 노드의 성능 및 선형 확장성 확보
  - 프록시x
  - 비동기 복제
  - 병합 작업이 없음
- 모든 쓰기 작업에 대한 안정성 확보(일정한 크기의 window 만큼의 유실 가능성을 항상 갖고 있음)
- Availability: Redis Cluster is able to survive partitions where the majority of the master nodes are reachable and there is at least one reachable replica for every master node that is no longer reachable. Moreover using replicas migration, masters no longer replicated by any replica will receive one from a master which is covered by multiple replicas.

## 특징 및 주의사항

- 단일 Redis 에서 실행 가능한 모든 command 를 동일하게 실행 가능
- 같은 키가 같은 슬롯에 저장되도록 해시 태그라는 개념을 구현함
- 수동으로 resharding 중에는 multi-key 작업을 한동안 사용할 수 없음
- multiple databases 는 지원하지 않으므로 오직 database 0 만 존재하며, select 명령은 허용되지 않음

