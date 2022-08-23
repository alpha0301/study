## 개요

@Transactional과 TransactionTemplate 모두 ThreadLocal을 활용함

tx 관리를 하기 위해서는 tx 상태와 실행간의 연결이 필요함
명령형에서는 이것이 ThreadLocal
반응형에서는 이런 전제가 불가능

Spring Security의 SecurityContext에도 동일한 이슈가 발생하고 있음

## Reactor Context

Reactor Context는 ThreadLocal의 역할을 reactive 환경에서 동일하게 수행함
Spring이 tx를 특정 Subscription에 바인딩할 수 있도록 해줌
기존 코드를 전환하는 경우 다시 작성하는 방법 외에는 없음

### Reactive Transaction Management

#### reactive @Transactional method

return Publisher

```java
class TransactionalService {

  final DatabaseClient db

  TransactionalService(DatabaseClient db) {
    this.db = db;
  }

  @Transactional
  Mono<Void> insertRows() {

    return db.execute()
      .sql("INSERT INTO person (name, age) VALUES('Joe', 34)")
      .fetch().rowsUpdated()
      .then(db.execute().sql("INSERT INTO contacts (name) VALUES('Joe')")
      .then();
  }
}
```

기존 명령형에서 사용하던 @Transactional 방식과 유사함
DatabaseClient라는 reactive client를 사용한다는 점이 다름
백그라운드에서 Spring의 tx 인터셉터와 ReactiveTransactionManager를 활용해서 tx를 관리함
(반드시 return을 Publisher로 해야 함 안그러면 명령형에서 사용하던 @Transactional 적용됨)

#### TransactionalOperator

```java
ConnectionFactory factory = …
ReactiveTransactionManager tm = new R2dbcTransactionManager(factory);
DatabaseClient db = DatabaseClient.create(factory);

TransactionalOperator rxtx = TransactionalOperator.create(tm);

Mono<Void> atomicOperation = db.execute()
  .sql("INSERT INTO person (name, age) VALUES('joe', 'Joe')")
  .fetch().rowsUpdated()
  .then(db.execute()
    .sql("INSERT INTO contacts (name) VALUES('Joe')")
    .then())
  .as(rxtx::transactional);
```