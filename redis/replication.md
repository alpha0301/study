# Replication

- leader follower(master-replica)

## 3 mechanisms

- master 의 모든 변경사항을 replica 로 전달
- 연결 끊어지면 master 또는 replica 에서 timeout 감지하고 재 연결을 시도하고 끊어진 동안 발생한 변경사항에 대한 부분 재동기화를 진행
- 재 연결 후 부분 동기화가 불가능한 경우, master 에서 모든 데이터에 대한 스냅샷을 생성하여 replica 로 전송

## 주의사항

- master 는 여러 개의 replica 를 가질 수 있음
- replica 간의 연결을 수락할 수 있음(계단식 구조가 가능)
- replica 초기 동기화 설정을 진행하는 동안 master 에서 이전 상태의 데이터셋을 계속 사용할 수 있음
- replication 을 활용하여 master 의 모든 데이터셋을 상시에 디스크에 쓰지 않고 정기적으로 쓰도록 설정할 수 있음(재시작 시 빈 데이터셋으로 시작해야 하는 위험이 있음)

### master -> replica 과정이 비동기

- wait 을 통해 동기적인 복제를 요청할 수 있지만 강력한 일관성을 보장하지 않음
- 장애 조치 도중에 승인된 write 요청 에 대해 손실 위험이 있음
- wait 을 사용하면 실패 이벤트 후 write 요청이 손실할 위험이 크게 줄어듦
- HA, failover 는 Sentinel, Redis Cluster 를 참고
