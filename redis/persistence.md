# Redis Persistence

[원본 문서 링크](https://redis.io/topics/persistence)

RDB 와 AOF 간의 스위칭 및 세부 장애 대응 전략은 위 문서의 하단을 참고

## persistence options

### RDB

- redis database
- 지정된 간격으로 데이터셋의 특정 시점 스냅샷을 생성

#### Snapshotting

- SAVE or BGSAVE 참고
- 스냅샷 파일명: dump.rdb
- copy-on-write 의 이점을 얻을 수 있음
- fork() 호출 -> 자식 프로세스 생성 -> 자식이 임시 RDB 파일에 데이터셋을 쓰고 이전 파일을 대체

매 60초 또는 1000개의 키 변경이 있을 때 마다 스냅샷을 생성하는 명령어 예시
```
save 60 1000
```

#### RDB 장점

- ex) 24시간 내에는 매 시간별 스냅샷 생성하고 30일 내에는 일별 스냅샷 생성
- 원거리 데이터 센터나 aws s3 같은 저장소에 단일 파일로 보관하기 용이함
- redis 의 부모 프로세스가 시점 분기만 하면 나머지 작업은 자식이 처리하기 때문에 성능을 극대화할 수 있음
- AOF 에 비해 큰 데이터셋을 더 빠르게 시작할 수 있음
- replica 구성 시, 다시 시작 또는 장애 조치 후 부분 재동기화를 지원함

#### RDB 단점

- 장애가 발생하면 스냅샷 간격동안 유실된 데이터는 복구 불가능
- 자식 프로세스가 자주 fork 하여 디스크에 저장해야 하므로 데이터셋 크기에 영향을 많이 받음 CPU 성능이 부족할 경우, redis client 서비스가 초 단위로 멈출 수 있음
- AOF도 fork가 필요하지만 어느 정도 조절이 가능

### AOF

- Append Only File
- 모든 write operation 을 디스크에 저장하고 서버 재시작 시 원본 데이터셋을 구성

설정 방법
```
appendonly yes
```

fsync 옵션
```
appendfsync {parameter}
```
- always: 모든 command에 대해 기록하므로 매우 느림
- everysec(추천): 매 초마다 기록하므로 1초 내의 변경사항에 대해 유실 가능
- no: fsync가 아닌 os의 커널이 데이터를 flush 하는 시간 단위마다 수행(보통 30초)

#### AOF 장애 대응

- AOF 파일을 쓰는 동안 서버 다운, 디스크 풀과 같은 원인으로 장애 발생이 가능
- 최신 버전의 경우, 쓰는 도중에 멈춰버린 분량의 데이터를 제거하고 다시 데이터셋을 로드하여 복구됨
- 구 버전에서 자동으로 복구되지 않는 경우, 아래와 같이 대응 필요

```
1. AOF 파일 복구
2. 명령어 실행: redis-check-aof --fix (이후 diff -u 를 통해 두 파일의 내용을 비교할 수 있음)
3. 재시작하면 수정된 파일로 복구
```

AOF 파일의 끝이 잘린 것이 아니라 중간에 잘못된 바이트가 있는 경우 아래와 같은 에러가 출력됨

```
* Reading the remaining AOF tail...
# Bad file format reading the append only file: make a backup of your AOF file, then use ./redis-check-aof --fix <filename>
```

- 우선 별도의 옵션 없이 redis-check-aof 명령을 실행하여 문제를 눈으로 확인
- 문제가 있는 오프셋을 수동으로 제거하는것으로 문제를 해결할 수 있음
- 만일 redis-check-aof --fix 명령을 수행하면 문제가 해결될 수는 있지만 문제가 생긴 지점부터 끝까지의 모든 데이터가 유실될 수 있음

#### Log rewriting

- 같은 키에 대해서 여러 번 변경이 일어나면 마지막 로그 이외의 이전 로그는 불필요함(예: 0부터 100까지 카운팅)
- 이를 제거하는 최적화를 서비스 중단 없이 백그라운드에서 재빌드할 수 있음(명령어: BGREWRITEAOF)
- 2.4 버전 이후부터 자동으로 동작하도록 설정할 수 있음

#### AOF 장점

- 가장 안정성이 높음(fsync 옵션을 다양하게 선택 가능: 안함, 초당 수행, 쿼리당 수행)
- 최악의 이슈 상황(로그를 중간에 쓰다가 멈추거나..)에서도 redis-check-aof 도구로 해결 가능
- 용량이 너무 클 때에도 백그라운드 처리로 성능을 확보할 수 있음
  - 현재 사용중인 파일외에 별도의 파일을 생성하여 작업을 멈추지 않도록 함
  - 새 파일에 AOF 작업이 완료되면 두 파일을 교체하여 안정성을 유지함
- AOF 파일의 내용은 이해하기 쉬운 형태이며 쉽게 export 할 수 있음
- 실수로 FLUSHALL 명령어를 수행하더라도 즉시 서버를 중단하고 가장 최근 명령을 제거한 후 다시 시작하는 것 만으로 데이터셋을 복구할 수 있음

#### AOF 단점

- 동일한 데이터셋을 가진 RDB 파일보다 크기가 큼
- fsync 설정에 따라 RDB 보다 느릴 수 있음
- 아주 드물게 BRPOPLPUSH 와 같은 명령어를 처리할 때 버그가 발생할 가능성이 있으나 실제 사용자 보고가 있었던 적은 없음

### No persistence

- 서버가 켜져있는동안에만 데이터셋을 유지

### RDB + AOF

- 두 가지 persistence 를 동시에 활성화할 수 있음
- AOF 가 가장 완전한 데이터셋을 보장하므로 재구성하는데에 사용

## What should I use?

- RDBMS 수준의 안정성이 필요하다면 RDB + AOF 를 사용해야 함
- 장애 발생 후 몇 분의 데이터 손실을 허락한다면 RDB 가 더 좋은 선택이 될 수 있음
- 장기적으로 AOF 와 RDB 를 합친 단일 Persistence 모델로 통합 작업이 예정되어 있음
