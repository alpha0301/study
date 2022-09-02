## 개요

spring에서 트랜잭션을 처리하기 위한 방식과 주의사항

## 어노테이션

- @Transactional

### readonly

- @Transactional(readonly = true)
- read에 대한 default값이 true라서 특별한 상황이 아니면 설정할 필요가 없음
  - save, update, delete는 자동으로 false
- db 구축 형태에 따라 master가 아닌 read 전용에서 읽을 수 있음
- hibernate session flush mode = MANUAL
  - flush 호출해야 적용됨
  - CUD 작업이 동작하지 않음(스냅샷 저장, 변경 감지 등의 작업을 수행하지 않음)

#### MySQL

- select에서만 지원
- txid가 부여되지 않아서 오버헤드를 줄일 수 있음
- 별도의 스냅샷(Undo)을 통해 데이터를 조회하기 때문에 데이터 일관성을 보장할 수 있다.

#### Oracle

- @Transaction(readonly = true) 가 시작되기 전에 커밋된 데이터만 접근 가능
  - Repeatable Read
  - 성능뿐만이 아니라 안전한 격리수준도 보장하기 위해 사용
- select만 적용됨

#### PostgreSQL

- select를 제외한 DDL, DML, DCL 동작하지 않음
- 동시성 제어를 하기 위함
- txid를 가상으로 발급하기 때문에 일부 성능의 이점이 있을 수 있음
