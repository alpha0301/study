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

## Redis Cluster 프로토콜

- cluster node 간 다른 nodes 를 자동 검색하고 작동하지 않는 node 를 감지하면 서비스를 지속적으로 유지하기 위해 replica node 를 master 로 승격할 수 있음
- cluster node 는 프록시가 불가능하기 때문에 다른 node 로 전환이 필요한 경우, 리다이렉션으로 처리함

## write safety

- 비동기로 replication 이 진행되므로 마지막으로 선택된 마스터의 데이터셋이 다른 모든 replica 를 대체하게 됨
- 쓰기 데이터 손실이 발생하는 시나리오들
  - 마스터와 클라이언트간 소통은 원활하여 쓰기에 성공했지만 마스터와 replica 간의 소통이 안되고 그 와중에 마스터가 장애가 발생하여 replica 가 마스터를 대체하게 될 때(실제 사례)
  - 파티션 문제로 마스터와 연결할 수 없을 때 failover 에 의해 replica 로 대체되고 잠시 후 정상으로 돌아왔을 때, 클라이언트가 오래된 라우팅 테이블의 정보를 보고 대체된 replica 로 연결하지 않고 이전 마스터에 쓰기를 시도
    - 파티션 문제: 마스터간의 통신이 원활하지 않을 때 접근 가능한 파티션이 고정될 수 있음(이 때, 쓰기가 거부됨)
