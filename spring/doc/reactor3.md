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
