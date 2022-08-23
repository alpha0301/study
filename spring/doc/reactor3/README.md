# Reactor 3

## 개요

- 비동기 프로그래밍 패러다임
- data streams, propagation of change에 초점을 맞추고 있음
- 선언형이므로 개발자는 흐름을 제어하는 것이 아니라 논리에 대한 연산을 표현해야 함
- non-blocking, backpressure를 이용하여 비동기 스트림 처리의 표준을 제공하는 것이 목적

## 용어

- reactor operator
  - 메소드로 제공됨
  - just, fromArray, range, empty, error
  - never: 아무것도 안하는 경우
  - defer: subscription 시 결정
  - generator: 동기적으로 하나씩
  - create: 비동기적으로 한번에 여러개 생성
- concurrent agnostic: 동시성 불가지론적
## Spec

### java 9 flow api

```java
// 생산자는 구독자를 받아들인다
public interface Publisher<T> {
    public void subscribe(Subscriber<? super T> s);
}

// 구독자는 구독 정보를 등록하고 구독 정보를 통해 얻는 신호(onNext, onError, onComplete)에 따라 동작
public interface Subscriber<T> {
    public void onSubscribe(Subscription s);
    public void onNext(T t);
    public void onError(Throwable t);
    public void onComplete();
}

// 생산자와 구독자간의 데이터 요청 횟수를 관리
public interface Subscription {
    public void request(long n);
    public void cancel();
}
```

![image](https://user-images.githubusercontent.com/39113923/185822071-766ad9bb-c3bc-4235-85e9-cf4353924d8a.png)

1. Publisher는 Subscription을 구현하고 data 생성
2. Publisher는 subscribe(Subscriber)를 통해 Subscriber를 등록
3. Subscriber.onSubscribe(Subscription)를 통해 Subscription (Publisher - Subscription - Subscriber)
4. Subscription.request()를 통해 data 구독 시작
5. Subscripiton.request()는 조건에 따라 onNext(), onError(), onComplete() 호출

## Threading Model

- 기본적으로 동시성을 강요하지 않으며 이전 operator를 실행한 Thread가 그대로 operator를 수행
- Scheduler를 통해 Threading에 대한 제어를 할 수 있음

### Scheduler

- Thread를 통해 Worker를 생성할 수 있지만 반드시 Thread의 지원을 받아야 하는 것은 아님
- Worker, Time에 대한 책임을 갖고 있음
- 일반적인 사용예
  - Schedulers.immediate(): Scheduler가 필요하지만 Thread를 변경하지 않으려는 경우(현재 Thread가 즉시 수행)
  - Schedulers.single(): 짧은 일회성 작업에 최적화되어 있으며 Runnable을 사용
  - Schedulers.parallel(): CPU를 많이 사용하지만 lifecycle이 짧은 작업에 적합
  - Schedulers.boundedElastic(): lifecycle이 긴 작업에 적합하므로 일반적으로 블로킹 작업에 할당
  - Schedulers.elastic(): (Deprecated) boundedElastic()과 같은 용도지만 on-demand 형태로, Thread의 수가 무한정 증가할 수 있음(TTL 존재)
- 기본 제공되는 Scheduler들은 전역 싱글턴 형태지만, Schedulers.newParallel("parallel-1", 10)과 같이 새로운 인스턴스를 생성할 수도 있음
- ※ 작업을 수행할 Thread를 변경할 때 main thread -> scheduler 는 가능하지만, 임의의 thread -> main thread 는 불가능함(MainThreadExecutorService가 없기 때문)

#### VirtualTimeScheduler

- 긴 지연시간을 통해 반복하는 코드를 테스트한다고 가정할 떄 유용

```javascript
getData(timeSec) {
  return this.http.get('some_url')
    .pipe(
      repeatWhen((n) => n.pipe(
            delay(timeSec * 1000),
            take(2)
        ))
    );
}
```

http.get 호출을 성공하면 동일한 작업을 2회에 걸쳐서 반복하는 코드입니다. 반복 호출 사이에는 지정된 시간의 간격이 정해져 있습니다.

```javascript
it('should emit 3 specific values', (done) => {
  const range$ = service.getData(0.01);
  const result = [];
  mockHttp = {get: () => of(42, asyncScheduler)};

  range$.subscribe({
    next: (value) => {
      result.push(value);
    },
    complete: () => {
      expect(result).toEqual([42, 42, 42]);
      done();
    }
  });
});
```

jasmin done callback library를 사용해서 3번의 response가 complete 단계에서 모두 도착했는지 테스트를 하고 있습니다.
이런 count down latch 방식은 지연시간이 매우 길 경우에 테스트가 너무 오래 걸릴 수 있거나 예상하기 어렵습니다.

VirtualTimeScheduler는 가상 시간 매커니즘을 통해 스케쥴러 내에서 경과된 시간을 에뮬레이션하여 작업이 즉시 실행되도록 할 수 있습니다.
(real interval 세팅을 막고 각 작업을 queue에 넣고 delay를 기준으로 정렬 후 flush할 때 순서대로 실행)

java에서는 StepVerifier라는 VirtualTimeScheduler 제공 라이브러리가 있습니다.(reactor-test에 포함)

```java
void expect3600Elements(Supplier<Flux<Long>> supplier) {
  StepVerifier.withVirtualTime(supplier)
          .thenAwait(Duration.ofSeconds(3600)) // 3,600 초가 지났다고 간주함
          .expectNextCount(3600)
          .verifyComplete();
}
```

```java
StepVerifier.withVirtualTime(() -> Mono.delay(Duration.ofHours(3)))
        .expectSubscription()
        .expectNoEvent(Duration.ofHours(2)) // 2 시간동안 이벤트가 없고,
        .thenAwait(Duration.ofHours(1))     // 1 시간동안
        .expectNextCount(1)                 // 1 번의 이벤트가 발생 한 뒤
        .expectComplete()                   // 종료됨
        .verify();
```

### Non concurrent/parallel execution

```java
Logger logger = LoggerFactory.getLogger(MyApplication.class);

Mono.defer(() -> Mono.just("Hello!"))
        .doOnNext(logger::info)
        .subscribe();
```

```
22:06:15.235 [main] DEBUG reactor.util.Loggers$LoggerFactory - Using Slf4j logging framework
22:06:19.841 [main] INFO MyApplication - Hello!
```

### Parallel execution

- WebFlux 사용 시 reactor operator 실행 기본 스케쥴 설정과 동일

```java
Logger logger = LoggerFactory.getLogger(MyApplication.class);

Mono.defer(() -> Mono.just("Hello!"))
        .doOnNext(logger::info)
        .doOnTerminate(latch::countDown)
        .subscribeOn(Schedulers.elastic())
        .subscribe();

latch.await();
```

```
22:11:26.704 [main] DEBUG reactor.util.Loggers$LoggerFactory - Using Slf4j logging framework
22:11:26.733 [elastic-2] INFO MyApplication - Hello!
```

### reactor operator 동작 순서 이해

```java
Flux.range(1, 4)
        .log()
        .map(i -> i * 10)
        .log()
        .map(i -> "num: " + i)
        .log()
        .subscribe();
```

```
INFO 66019 --- [main] reactor.Flux.MapFuseable.35              : | request(unbounded)
INFO 66019 --- [main] reactor.Flux.MapFuseable.34              : | request(unbounded)
INFO 66019 --- [main] reactor.Flux.Range.33                    : | request(unbounded)
INFO 66019 --- [main] reactor.Flux.Range.33                    : | onNext(1)
INFO 66019 --- [main] reactor.Flux.MapFuseable.34              : | onNext(10)
INFO 66019 --- [main] reactor.Flux.MapFuseable.35              : | onNext(num: 10)
INFO 66019 --- [main] reactor.Flux.Range.33                    : | onNext(2)
INFO 66019 --- [main] reactor.Flux.MapFuseable.34              : | onNext(20)
INFO 66019 --- [main] reactor.Flux.MapFuseable.35              : | onNext(num: 20)
INFO 66019 --- [main] reactor.Flux.Range.33                    : | onNext(3)
INFO 66019 --- [main] reactor.Flux.MapFuseable.34              : | onNext(30)
INFO 66019 --- [main] reactor.Flux.MapFuseable.35              : | onNext(num: 30)
INFO 66019 --- [main] reactor.Flux.Range.33                    : | onNext(4)
INFO 66019 --- [main] reactor.Flux.MapFuseable.34              : | onNext(40)
INFO 66019 --- [main] reactor.Flux.MapFuseable.35              : | onNext(num: 40)
INFO 66019 --- [main] reactor.Flux.Range.33                    : | onComplete()
INFO 66019 --- [main] reactor.Flux.MapFuseable.34              : | onComplete()
INFO 66019 --- [main] reactor.Flux.MapFuseable.35              : | onComplete()
```

- map 단위로 동작하지 않고 개별 데이터 단위로 동작

```java
Flux.range(1, 4)
        .publishOn(Schedulers.newSingle("pub1"))
        .log()
        .map(i -> i * 10)
        .log()
        .map(i -> "num: " + i)
        .log()
        .subscribe();
```

```
INFO 66019 --- [restartedMain] reactor.Flux.MapFuseable.38              : | request(unbounded)
INFO 66019 --- [restartedMain] reactor.Flux.MapFuseable.37              : | request(unbounded)
INFO 66019 --- [restartedMain] reactor.Flux.PublishOn.36                : | request(unbounded)
INFO 66019 --- [pub1-45] reactor.Flux.PublishOn.36                : | onNext(1)
INFO 66019 --- [pub1-45] reactor.Flux.MapFuseable.37              : | onNext(10)
INFO 66019 --- [pub1-45] reactor.Flux.MapFuseable.38              : | onNext(num: 10)
INFO 66019 --- [pub1-45] reactor.Flux.PublishOn.36                : | onNext(2)
INFO 66019 --- [pub1-45] reactor.Flux.MapFuseable.37              : | onNext(20)
INFO 66019 --- [pub1-45] reactor.Flux.MapFuseable.38              : | onNext(num: 20)
INFO 66019 --- [pub1-45] reactor.Flux.PublishOn.36                : | onNext(3)
INFO 66019 --- [pub1-45] reactor.Flux.MapFuseable.37              : | onNext(30)
INFO 66019 --- [pub1-45] reactor.Flux.MapFuseable.38              : | onNext(num: 30)
INFO 66019 --- [pub1-45] reactor.Flux.PublishOn.36                : | onNext(4)
INFO 66019 --- [pub1-45] reactor.Flux.MapFuseable.37              : | onNext(40)
INFO 66019 --- [pub1-45] reactor.Flux.MapFuseable.38              : | onNext(num: 40)
INFO 66019 --- [pub1-45] reactor.Flux.PublishOn.36                : | onComplete()
INFO 66019 --- [pub1-45] reactor.Flux.MapFuseable.37              : | onComplete()
INFO 66019 --- [pub1-45] reactor.Flux.MapFuseable.38              : | onComplete()
```

```java
Flux.range(1, 4)
        .publishOn(Schedulers.newSingle("pub1"))
        .publishOn(Schedulers.newSingle("pub2"))
        .log()
        .map(i -> i * 10)
        .log()
        .map(i -> "num: " + i)
        .log()
        .subscribe();
```

```
INFO 68914 --- [restartedMain] reactor.Flux.MapFuseable.18              : | request(unbounded)
INFO 68914 --- [restartedMain] reactor.Flux.MapFuseable.17              : | request(unbounded)
INFO 68914 --- [restartedMain] reactor.Flux.PublishOn.16                : | request(unbounded)
INFO 68914 --- [pub2-17] reactor.Flux.PublishOn.16                : | onNext(1)
INFO 68914 --- [pub2-17] reactor.Flux.MapFuseable.17              : | onNext(10)
INFO 68914 --- [pub2-17] reactor.Flux.MapFuseable.18              : | onNext(num: 10)
INFO 68914 --- [pub2-17] reactor.Flux.PublishOn.16                : | onNext(2)
INFO 68914 --- [pub2-17] reactor.Flux.MapFuseable.17              : | onNext(20)
INFO 68914 --- [pub2-17] reactor.Flux.MapFuseable.18              : | onNext(num: 20)
INFO 68914 --- [pub2-17] reactor.Flux.PublishOn.16                : | onNext(3)
INFO 68914 --- [pub2-17] reactor.Flux.MapFuseable.17              : | onNext(30)
INFO 68914 --- [pub2-17] reactor.Flux.MapFuseable.18              : | onNext(num: 30)
INFO 68914 --- [pub2-17] reactor.Flux.PublishOn.16                : | onNext(4)
INFO 68914 --- [pub2-17] reactor.Flux.MapFuseable.17              : | onNext(40)
INFO 68914 --- [pub2-17] reactor.Flux.MapFuseable.18              : | onNext(num: 40)
INFO 68914 --- [pub2-17] reactor.Flux.PublishOn.16                : | onComplete()
INFO 68914 --- [pub2-17] reactor.Flux.MapFuseable.17              : | onComplete()
INFO 68914 --- [pub2-17] reactor.Flux.MapFuseable.18              : | onComplete()
```

- publishOn이 겹쳐지면 마지막으로 호출된 스케쥴로 설정됨

```java
Flux.range(1, 4)
        .subscribeOn(Schedulers.newSingle("sub1"))
        .subscribeOn(Schedulers.newSingle("sub2"))
        .log()
        .map(i -> i * 10)
        .log()
        .map(i -> "num: " + i)
        .log()
        .subscribe();
```

```
INFO 68914 --- [sub1-20] reactor.Flux.SubscribeOn.19              : onNext(1)
INFO 68914 --- [sub1-20] reactor.Flux.Map.20                      : onNext(10)
INFO 68914 --- [sub1-20] reactor.Flux.Map.21                      : onNext(num: 10)
INFO 68914 --- [sub1-20] reactor.Flux.SubscribeOn.19              : onNext(2)
INFO 68914 --- [sub1-20] reactor.Flux.Map.20                      : onNext(20)
INFO 68914 --- [sub1-20] reactor.Flux.Map.21                      : onNext(num: 20)
INFO 68914 --- [sub1-20] reactor.Flux.SubscribeOn.19              : onNext(3)
INFO 68914 --- [sub1-20] reactor.Flux.Map.20                      : onNext(30)
INFO 68914 --- [sub1-20] reactor.Flux.Map.21                      : onNext(num: 30)
INFO 68914 --- [sub1-20] reactor.Flux.SubscribeOn.19              : onNext(4)
INFO 68914 --- [sub1-20] reactor.Flux.Map.20                      : onNext(40)
INFO 68914 --- [sub1-20] reactor.Flux.Map.21                      : onNext(num: 40)
INFO 68914 --- [sub1-20] reactor.Flux.SubscribeOn.19              : onComplete()
INFO 68914 --- [sub1-20] reactor.Flux.Map.20                      : onComplete()
INFO 68914 --- [sub1-20] reactor.Flux.Map.21                      : onComplete()
```

- subscribeOn은 중복을 무시하고 가장 먼저 등록한 스케쥴로 설정됨

### PublishOn

- upstream -> downstream
- 데이터 경로를 지나는 도중에 적용
- Thread를 도약할 때(hop) 필요한 기본 연산자
- 다른 publishOn이 나타나기 전 까지 지정한 Scheduler에서 실행함
- onNext, onComplete, onError

### SubscribeOn

- downstream -> upstream
- subscribe() 호출 직후에 초기화하면서 적용
- 신호가 위쪽으로 흐르기 때문에 Publisher가 데이터를 생성하기 시작하는 위치에 직접적인 영향을 줌

```java
Flux.just("hello")                                                                           // 3.
    .doOnNext(v -> System.out.println("just " + Thread.currentThread().getName()))           // 4.
    .publishOn(Scheduler.boundedElastic())                                                   // 5.
    .doOnNext(v -> System.out.println("publish " + Thread.currentThread().getName()))        // 6.
    .delayElements(Duration.ofMillis(500))                                                   // 7.
    .subscribeOn(Schedulers.elastic())                                                       // 2. 8.
    .subscribe(v -> System.out.println(v + " delayed " + Thread.currentThread().getName())); // 1. 9.
```

```
just elastic-1
publish boundedElastic-1
hello delayed parallel-1
```

위 예제의 동작은 다음과 같습니다.

1. main thread가 subscribe 호출
2. subscribeOn에 의해 subscription 및 위 방향으로 곧바로 Schedulers.elastic() 으로 전환
3. elastic을 통해 emit hello
4. 첫번째 doOnNext에서 elastic이 직접 'just' 출력
5. publishOn에 의해 아래 방향으로 Schedulers.boundedElastic() 으로 전환
6. 두번째 doOnNext에서 boundedElastic이이 데이터를 수신해서 'publish' 출력
7. delayElements는 시간 연산자이므로 기본적으로 Schedulers.parallel()을 사용(publishOn)해서 아래 방향으로 전환
8. 데이터 경로상에서(on the data path) subscribeOn는 아무것도 하지 않음 (동일한 스레드에서 신호를 전)
9. 데이터 경로상에서 subscribe 내의 람다식은 데이터를 수신한 Thread에서 처리하므로 parallel이 'hello delayed' 출력

## Flux

- Reactive Streams Publisher
- Flux sequences를 생성, 변형 등을 할 수 있는 operator 제공
- 생성: onNext
- 완료: onComplete, onError
- 종료 이벤트 없이는 무한하게 수행
- instance의 operator는 비동기적으로 사용됨

```java
Flux.interval(Duration.ofMillis(100))
  .take(10)
  .subscribe(System.out::println);

Thread.sleep(500);
System.out.println("end!!");
```
```
0
1
2
3
end!!
4
```
(비동기로 동작함을 알 수 있는 예제)

## Mono

- 최대 1개의 element를 생성하는것에 특화된 Flux
- 무조건 0 or 1 이므로 element로 완료 또는 element 없이 에러로 완료만 있음

## StepVerifier

- 어떤 publisher든지 subscibe하는 것이 가능
- 검증 도중 하나라도 실패하면 AssertionError

```java
static void expectSkylerJesseComplete() {
  User user1 = new User("swhite");
  User user2 = new User("jpinkman");
  Flux<User> flux = Flux.fromIterable(Arrays.asList(user1, user2));

  StepVerifier.create(flux)
    .assertNext(user -> assertThat(user.getUserName()).isEqualTo("swhite"))
    .assertNext(user -> assertThat(user.getUserName()).isEqualTo("jpinkman"))
    .verifyComplete();
}
```

## Operators with async

- 웹서비스에서 호출되는 API는 동기적인 operator를 사용할 수 없음(ex: map)
- flaMap과 같은 비동기 operator를 사용해야 함

### flatMap

- 각 element들을 비동기적으로 Publisher로 변환
- 각 Publisher를 비동기적으로 subscribe()
- subscribe의 결과로 발생한 데이터들을 모아서 Flux화
- 순서 보장이 안됨

![image](https://user-images.githubusercontent.com/39113923/185051951-aff99eed-185e-463b-a08d-d45307fd054c.png)

![image](https://user-images.githubusercontent.com/39113923/185052154-eda14d38-5c37-4486-ba3f-a86f92e4db0e.png)

```java
Mono<String> toUpperCaseSync(Mono<String> str) {
    return str.map(String::toUpperCase);
}

Mono<String> toUpperCaseAsync(Mono<String> str) {
    return str.flatMap(s -> Mono.just(s.toUpperCase()));
}

Flux<String> toUpperCasesSync(Flux<String> strs) {
    return strs.map(String::toUpperCase);
}

Flux<String> toUpperCasesAsync(Flux<String> strs) {
    return strs.flatMap(s -> Mono.just(s.toUpperCase()));
}
```

## Merge

- 여러 개의 Publisher를 모아서 Flux 하나를 만드는 것

```java
Flux<String> merge(Flux<String> f1, Flux<String> f2) {
    return f1.mergeWith(f2);
}

Flux<String> merge(Mono<String> m1, Mono<String> m2) {
    return m1.mergeWith(m2);
}
```

위와 같이 하면 순서 보장이 되지 않음 순서를 보장하려면,

```java
Flux<String> keepTheOrder(Flux<String> f1, Flux<String> f2) {
    return f1.concatWith(f2);
}

Flux<String> keepTheOrder(Mono<String> m1, Mono<String> m2) {
    return m1.concatWith(m2);
}
```

concatWith를 사용하면 됨