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

## Schedule

- 기본적으로 동시성을 강요하지 않으며 이전 operator를 실행한 Thread가 그대로 operator를 수행

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

- 신호 처리 스케쥴링

### SubscribeOn

- 시퀀스를 실행할 스레드를 결정

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