# Reactor 3

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

```java

```

### flatMap

- 각 element들을 비동기적으로 Publisher로 변환
- 각 Publisher를 비동기적으로 subscribe()
- subscribe의 결과로 발생한 데이터들을 모아서 Flux화
- 순서 보장이 안됨

![image](https://user-images.githubusercontent.com/39113923/185051951-aff99eed-185e-463b-a08d-d45307fd054c.png)

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
