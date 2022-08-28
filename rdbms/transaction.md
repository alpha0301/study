# isolation level

ACID 중 격리성 수준을 설정하여 성능과 격리성 사이의 트레이드 오프를 결정하기 위한 설정입니다.
아래 순서는 격리 수준을 오름차순으로 정렬한 결과입니다.

## READ UNCOMMITTED

DIRTY READ가 가능합니다.(commit이나 rollback 여부와 관계없이 트랜잭션의 변경내용을 다른 트랜잭션이 볼 수 있습니다.)
일반적인 DB에서는 잘 사용하지 않습니다.

## READ COMMITTED

commit이 완료된 데이터만 다른 트랜잭션에서 조회할 수 있습니다.(커밋 전 이전 데이터를 저장하는 UNDO 영역에서 가져옴)
오라클의 기본 격리 수준 입니다. 온라인 서비스에서 가장 많이 사용되고 있습니다.
Dirty Read를 방지할 수 있습니다.
트랜잭션 커밋 이전 값을 다른 트랜잭션에게 제공하는 순서는 다음과 같습니다.

1. tx-1 시작 후 값을 A에서 B로 변경 시도 (A)
2. 이전 값을 Undo 영역으로 백업 (A)
3. 변경된 값을 테이블에 기록 (A)
4. tx-1 commit (B)

Undo 영역은 tx 격리 수준 뿐만 아니라 Rollback에 대한 복구에도 사용됩니다.
해당 격리수준에서는 NON-REPEATABLE READ 문제가 발생할 수 있습니다.

### NON-REPEATABLE READ

REPEATABLE READ란 같은 tx 내에서 동일한 select 쿼리는 항상 같은 결과를 보장해야 한다는 규칙입니다.
다음은 문제 상황입니다.

1. tx-1 start
2. tx-1 toto 조회 -> 없음
3. tx-2 start
4. tx-2 toto 추가
5. tx-2 commit
6. tx-1 toto 조회 -> 결과 1건 (문제 발생)

위 절차에서 2번의 결과와 6번의 결과가 같은 tx임에도 달라지는 문제가 발생했습니다.
일반적으로는 문제가 되지 않지만, 입금과 출금 처리가 진행되고 있을 때
다른 트랜잭션에서 오늘 입금된 금액의 총합을 여러번 조회한다고 하면 문제가 생길 수 있습니다.


## REAPEATABLE READ



## SERIALIZABLE

