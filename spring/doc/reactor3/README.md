# Reactor 3

## 개요

- 비동기 프로그래밍 패러다임
- data streams, propagation of change에 초점을 맞추고 있음
- 선언형이므로 개발자는 흐름을 제어하는 것이 아니라 논리에 대한 연산을 표현해야 함
- non-blocking, backpressure를 이용하여 비동기 스트림 처리의 표준을 제공하는 것이 목적

## Spec

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
1. Publisher는 subscribe(Subscriber)를 통해 Subscriber를 등록
1. Subscriber.onSubscribe(Subscription)를 통해 Subscription (Publisher - Subscription - Subscriber)
1. Subscription.request()를 통해 data 구독 시작
1. 

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

## Request

### Backpressure(Volume Control)

![image](https://user-images.githubusercontent.com/39113923/185055859-9faf198a-bea0-4b08-a412-77c1c61d1f7f.png)

- Subscriber가 Publisher에게 받고자 하는 event의 양을 알리는 것
- Subscription level에서 처리됨(Subscription.request() 으로 알림)
- Subscription lifecycle
  - 1. Publisher.subscribe() -> Subscriber 생성
  - 2. Subscriber.onSubscribe() -> Subscription 생성
  - 3. 
