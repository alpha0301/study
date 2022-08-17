# isolation level

ACID 중 격리성 수준을 설정하여 성능과 격리성 사이의 트레이드 오프를 결정하기 위한 설정입니다.
아래 순서는 격리 수준을 오름차순으로 정렬한 결과입니다.

## READ UNCOMMITTED

DIRTY READ가 가능합니다.(commit이나 rollback 여부와 관계없이 트랜잭션의 변경내용을 다른 트랜잭션이 볼 수 있습니다.)
일반적인 DB에서는 잘 사용하지 않습니다.

## READ COMMITTED

commit이 완료된 데이터만 다른 트랜잭션에서 조회할 수 있습니다.(커밋 전 이전 데이터를 저장하는 UNDO 영역에서 가져옴)


## REAPEATABLE READ

## SERIALIZABLE

