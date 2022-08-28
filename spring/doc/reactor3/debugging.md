# 디버깅

## 어려운 점

### 명령형, 절차형에서는?

- 작성한 로직 -> 작성한 로직 호출
- 한줄씩 이동하므로 추적이 용이

### 선언형

- 스케쥴러가 각 로직을 대신 호출해줌
- IoC 와 유사
- 에러 발생시 스택 트레이스는 결과적으로 로직 보다는 스케쥴러 내용으로 가득 차게 됨

```java
@Test
void debugTest() {
        int seconds = LocalTime.now().getSecond();
        System.out.println("### " + seconds);
        Mono<Integer> element;

    if (seconds % 2 == 0)
    element = Flux.range(1, 5).elementAt(5);
    else if (seconds % 3 == 0)
    element = Flux.range(0, 4).elementAt(5);
    else
    element = Flux.just(1, 2, 3, 4).elementAt(5);

    element.subscribeOn(Schedulers.parallel()).block();
}
```
```
source had 5 elements, expected at least 6
java.lang.IndexOutOfBoundsException: source had 5 elements, expected at least 6
	at reactor.core.publisher.MonoElementAt$ElementAtSubscriber.onComplete(MonoElementAt.java:165)
	at reactor.core.publisher.FluxRange$RangeSubscription.fastPath(FluxRange.java:138)
	at reactor.core.publisher.FluxRange$RangeSubscription.request(FluxRange.java:109)
	at reactor.core.publisher.MonoElementAt$ElementAtSubscriber.request(MonoElementAt.java:103)
	at reactor.core.publisher.MonoSubscribeOn$SubscribeOnSubscriber.trySchedule(MonoSubscribeOn.java:189)
	at reactor.core.publisher.MonoSubscribeOn$SubscribeOnSubscriber.onSubscribe(MonoSubscribeOn.java:134)
	at reactor.core.publisher.MonoElementAt$ElementAtSubscriber.onSubscribe(MonoElementAt.java:118)
	at reactor.core.publisher.FluxRange.subscribe(FluxRange.java:69)
	at reactor.core.publisher.Mono.subscribe(Mono.java:4397)
	at reactor.core.publisher.MonoSubscribeOn$SubscribeOnSubscriber.run(MonoSubscribeOn.java:126)
	at reactor.core.scheduler.WorkerTask.call(WorkerTask.java:84)
	at reactor.core.scheduler.WorkerTask.call(WorkerTask.java:37)
	at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:264)
	at java.base/java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:304)
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1136)
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:635)
	at java.base/java.lang.Thread.run(Thread.java:833)
	Suppressed: java.lang.Exception: #block terminated with an error
		at reactor.core.publisher.BlockingSingleSubscriber.blockingGet(BlockingSingleSubscriber.java:99)
		at reactor.core.publisher.Mono.block(Mono.java:1707)
		at com.example.reactordemo.ReactorDemoApplicationTests.debugTest(ReactorDemoApplicationTests.java:32)
		at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
		at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:77)
		at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
		at java.base/java.lang.reflect.Method.invoke(Method.java:568)
```

- 의미 있는 정보가 거의 나오지 않음
- IndexOutOfBoundsException, 5 elements, expected at least 6  
- 실제 exception이 발생한 위치를 알 수 없음
- Hooks.onOperatorDebug() 적용 시 좀 더 필요한 정보를 얻을 수 있음

```java
@Test
void debugTest() {
    int seconds = LocalTime.now().getSecond();
    System.out.println("### " + seconds);
    Hooks.onOperatorDebug();

    Mono<Integer> element;

    if (seconds % 2 == 0)
        element = Flux.range(1, 5).elementAt(5);
    else if (seconds % 3 == 0)
        element = Flux.range(0, 4).elementAt(5);
    else
        element = Flux.just(1, 2, 3, 4).elementAt(5);

    element.subscribeOn(Schedulers.parallel()).block();
}
```
```
source had 4 elements, expected at least 6
java.lang.IndexOutOfBoundsException: source had 4 elements, expected at least 6
	at reactor.core.publisher.MonoElementAt$ElementAtSubscriber.onComplete(MonoElementAt.java:165)
	Suppressed: The stacktrace has been enhanced by Reactor, refer to additional information below: 
Assembly trace from producer [reactor.core.publisher.MonoElementAt] :
	reactor.core.publisher.Flux.elementAt(Flux.java:4865)
	com.example.reactordemo.ReactorDemoApplicationTests.debugTest(ReactorDemoApplicationTests.java:29)
Error has been observed at the following site(s):
	*____Flux.elementAt ⇢ at com.example.reactordemo.ReactorDemoApplicationTests.debugTest(ReactorDemoApplicationTests.java:29)
	|_ Mono.subscribeOn ⇢ at com.example.reactordemo.ReactorDemoApplicationTests.debugTest(ReactorDemoApplicationTests.java:32)
Original Stack Trace:
		at app//reactor.core.publisher.MonoElementAt$ElementAtSubscriber.onComplete(MonoElementAt.java:165)
		at app//reactor.core.publisher.FluxArray$ArraySubscription.fastPath(FluxArray.java:177)
		at app//reactor.core.publisher.FluxArray$ArraySubscription.request(FluxArray.java:97)
		at app//reactor.core.publisher.MonoElementAt$ElementAtSubscriber.request(MonoElementAt.java:103)
		at app//reactor.core.publisher.MonoSubscribeOn$SubscribeOnSubscriber.trySchedule(MonoSubscribeOn.java:189)
		at app//reactor.core.publisher.MonoSubscribeOn$SubscribeOnSubscriber.onSubscribe(MonoSubscribeOn.java:134)
		at app//reactor.core.publisher.MonoElementAt$ElementAtSubscriber.onSubscribe(MonoElementAt.java:118)
		at app//reactor.core.publisher.FluxArray.subscribe(FluxArray.java:53)
		at app//reactor.core.publisher.FluxArray.subscribe(FluxArray.java:59)
		at app//reactor.core.publisher.Mono.subscribe(Mono.java:4397)
		at app//reactor.core.publisher.MonoSubscribeOn$SubscribeOnSubscriber.run(MonoSubscribeOn.java:126)
		at app//reactor.core.scheduler.WorkerTask.call(WorkerTask.java:84)
		at app//reactor.core.scheduler.WorkerTask.call(WorkerTask.java:37)
		at java.base@17.0.4.1/java.util.concurrent.FutureTask.run(FutureTask.java:264)
		at java.base@17.0.4.1/java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:304)
		at java.base@17.0.4.1/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1136)
		at java.base@17.0.4.1/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:635)
		at java.base@17.0.4.1/java.lang.Thread.run(Thread.java:833)
	Suppressed: java.lang.Exception: #block terminated with an error
		at reactor.core.publisher.BlockingSingleSubscriber.blockingGet(BlockingSingleSubscriber.java:99)
		at reactor.core.publisher.Mono.block(Mono.java:1707)
		at com.example.reactordemo.ReactorDemoApplicationTests.debugTest(ReactorDemoApplicationTests.java:32)
		at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
		at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:77)
		at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
		at java.base/java.lang.reflect.Method.invoke(Method.java:568)
```

- Hooks.onOperatorDebug() 적용에 의해 Exception이 발생한 위치를 알 수 있음
  - com.example.reactordemo.ReactorDemoApplicationTests.debugTest(ReactorDemoApplicationTests.java:29)
  - 에러가 나기 전에 수집 준비를 해야 하므로 전역적으로 크게 성능이 저하됨

```java
@Test
void debugTest() {
    int seconds = LocalTime.now().getSecond();
    System.out.println("### " + seconds);

    Mono<Integer> element;

    if (seconds % 2 == 0)
        element = Flux.range(1, 5).elementAt(5)
                .checkpoint();
    else if (seconds % 3 == 0)
        element = Flux.range(0, 4).elementAt(5)
                .checkpoint();
    else
        element = Flux.just(1, 2, 3, 4).elementAt(5)
                .checkpoint();

    element.subscribeOn(Schedulers.parallel()).block();
}
```
```
source had 4 elements, expected at least 6
java.lang.IndexOutOfBoundsException: source had 4 elements, expected at least 6
	at reactor.core.publisher.MonoElementAt$ElementAtSubscriber.onComplete(MonoElementAt.java:165)
	Suppressed: The stacktrace has been enhanced by Reactor, refer to additional information below: 
Assembly trace from producer [reactor.core.publisher.MonoElementAt] :
	reactor.core.publisher.Mono.checkpoint(Mono.java:2177)
	com.example.reactordemo.ReactorDemoApplicationTests.debugTest(ReactorDemoApplicationTests.java:29)
Error has been observed at the following site(s):
	*__checkpoint() ⇢ at com.example.reactordemo.ReactorDemoApplicationTests.debugTest(ReactorDemoApplicationTests.java:29)
Original Stack Trace:
		at app//reactor.core.publisher.MonoElementAt$ElementAtSubscriber.onComplete(MonoElementAt.java:165)
		at app//reactor.core.publisher.FluxArray$ArraySubscription.fastPath(FluxArray.java:177)
		at app//reactor.core.publisher.FluxArray$ArraySubscription.request(FluxArray.java:97)
		at app//reactor.core.publisher.MonoElementAt$ElementAtSubscriber.request(MonoElementAt.java:103)
		at app//reactor.core.publisher.MonoSubscribeOn$SubscribeOnSubscriber.trySchedule(MonoSubscribeOn.java:189)
		at app//reactor.core.publisher.MonoSubscribeOn$SubscribeOnSubscriber.onSubscribe(MonoSubscribeOn.java:134)
		at app//reactor.core.publisher.MonoElementAt$ElementAtSubscriber.onSubscribe(MonoElementAt.java:118)
		at app//reactor.core.publisher.FluxArray.subscribe(FluxArray.java:53)
		at app//reactor.core.publisher.FluxArray.subscribe(FluxArray.java:59)
		at app//reactor.core.publisher.Mono.subscribe(Mono.java:4397)
		at app//reactor.core.publisher.MonoSubscribeOn$SubscribeOnSubscriber.run(MonoSubscribeOn.java:126)
		at app//reactor.core.scheduler.WorkerTask.call(WorkerTask.java:84)
		at app//reactor.core.scheduler.WorkerTask.call(WorkerTask.java:37)
		at java.base@17.0.4.1/java.util.concurrent.FutureTask.run(FutureTask.java:264)
		at java.base@17.0.4.1/java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:304)
		at java.base@17.0.4.1/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1136)
		at java.base@17.0.4.1/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:635)
		at java.base@17.0.4.1/java.lang.Thread.run(Thread.java:833)
	Suppressed: java.lang.Exception: #block terminated with an error
		at reactor.core.publisher.BlockingSingleSubscriber.blockingGet(BlockingSingleSubscriber.java:99)
		at reactor.core.publisher.Mono.block(Mono.java:1707)
		at com.example.reactordemo.ReactorDemoApplicationTests.debugTest(ReactorDemoApplicationTests.java:31)
		at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
		at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:77)
		at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
		at java.base/java.lang.reflect.Method.invoke(Method.java:568)
```
- 각 오퍼레이트 메소드 체인에 checkpoint를 연결하면 성능 문제를 해결하면서 디버깅이 가능
- checkpoint(String)은 stacktrace를 생략하고 파라미터 문자열을 같이 표기해서 스택트레이스 생성 부담을 줄이면서도 오류 위치를 찾기 쉬움
- 문제가 생겨서 checkpoint 코드를 입력하고 다시 release 해서 오류를 잡는건 좀...

```java
@SpringBootApplication
public class ReactorDemoApplication {

    public static void main(String[] args) {
        ReactorDebugAgent.init();
        SpringApplication.run(ReactorDemoApplication.class, args);
    }

}

public Mono<Integer> debug() {
    int seconds = LocalTime.now().getSecond();
    Mono<Integer> element;

    if (seconds % 2 == 0)
//      element = Flux.range(1, 10)
//          .elementAt(5);
        Flux flux = Flux.range(1, 10);
        flux = Hooks.addCallSiteInfo(flux, "Flux.range\n com.example.reactordemo.Example.debug(Example.java:16)");
        flux = flux.elementAt(5);
        flux = Hooks.addCallSiteInfo(flux, "Flux.elementAt\n com.example.reactordemo.Example.debug(Example.java:17)");
    else if (seconds % 3 == 0)
        element = Flux.range(0, 4).elementAt(5);
    else
        element = Flux.just(1, 2, 3, 4).elementAt(5);

    return element;
}
```
```
java.lang.IndexOutOfBoundsException: source had 4 elements, expected at least 6
	at reactor.core.publisher.MonoElementAt$ElementAtSubscriber.onComplete(MonoElementAt.java:165) ~[reactor-core-3.4.22.jar:3.4.22]
	Suppressed: reactor.core.publisher.FluxOnAssembly$OnAssemblyException: 
Assembly trace from producer [reactor.core.publisher.MonoElementAt] :
	reactor.core.publisher.Flux.elementAt
	com.example.reactordemo.handler.ExampleHandler.debug(ExampleHandler.java:18)
Error has been observed at the following site(s):
	*______Flux.elementAt ⇢ at com.example.reactordemo.handler.ExampleHandler.debug(ExampleHandler.java:18)
	|_          Mono.from ⇢ at org.springframework.http.codec.json.AbstractJackson2Encoder.encode(AbstractJackson2Encoder.java:149)
	|_           Mono.map ⇢ at org.springframework.http.codec.json.AbstractJackson2Encoder.encode(AbstractJackson2Encoder.java:150)
	|_          Mono.flux ⇢ at org.springframework.http.codec.json.AbstractJackson2Encoder.encode(AbstractJackson2Encoder.java:151)
	|_ Flux.singleOrEmpty ⇢ at org.springframework.http.codec.EncoderHttpMessageWriter.write(EncoderHttpMessageWriter.java:129)
	|_ Mono.switchIfEmpty ⇢ at org.springframework.http.codec.EncoderHttpMessageWriter.write(EncoderHttpMessageWriter.java:130)
	|_       Mono.flatMap ⇢ at org.springframework.http.codec.EncoderHttpMessageWriter.write(EncoderHttpMessageWriter.java:134)
	|_   Mono.doOnDiscard ⇢ at org.springframework.http.codec.EncoderHttpMessageWriter.write(EncoderHttpMessageWriter.java:140)
	|_                    ⇢ at org.springframework.http.codec.EncoderHttpMessageWriter.write(EncoderHttpMessageWriter.java:217)
	|_                    ⇢ at org.springframework.web.reactive.function.BodyInserters.lambda$write$13(BodyInserters.java:401)
	|_                    ⇢ at org.springframework.web.reactive.function.BodyInserters.write(BodyInserters.java:398)
	|_                    ⇢ at org.springframework.web.reactive.function.BodyInserters.lambda$writeWithMessageWriters$10(BodyInserters.java:380)
	|_                    ⇢ at org.springframework.web.reactive.function.BodyInserters.lambda$fromPublisher$4(BodyInserters.java:185)
	|_                    ⇢ at org.springframework.web.reactive.function.server.DefaultServerResponseBuilder$BodyInserterResponse.writeToInternal(DefaultServerResponseBuilder.java:407)
	|_                    ⇢ at org.springframework.web.reactive.function.server.DefaultServerResponseBuilder$AbstractServerResponse.writeTo(DefaultServerResponseBuilder.java:351)
	|_                    ⇢ at org.springframework.web.reactive.function.server.support.ServerResponseResultHandler.handleResult(ServerResponseResultHandler.java:94)
	|_         checkpoint ⇢ Handler com.example.reactordemo.config.RouterConfig$$Lambda$833/0x0000000800f9c000@4f984e84 [DispatcherHandler]
	*____________________ ⇢ at org.springframework.web.server.handler.DefaultWebFilterChain.lambda$filter$0(DefaultWebFilterChain.java:120)
	*__________Mono.defer ⇢ at org.springframework.web.server.handler.DefaultWebFilterChain.filter(DefaultWebFilterChain.java:119)
	|_                    ⇢ at org.springframework.web.server.handler.FilteringWebHandler.handle(FilteringWebHandler.java:59)
	|_                    ⇢ at org.springframework.web.server.handler.WebHandlerDecorator.handle(WebHandlerDecorator.java:56)
	|_ Mono.onErrorResume ⇢ at org.springframework.web.server.handler.ExceptionHandlingWebHandler.handle(ExceptionHandlingWebHandler.java:77)
	*__________Mono.error ⇢ at org.springframework.web.server.handler.ExceptionHandlingWebHandler$CheckpointInsertingHandler.handle(ExceptionHandlingWebHandler.java:98)
	|_         checkpoint ⇢ HTTP GET "/debug" [ExceptionHandlingWebHandler]
	|_                    ⇢ at org.springframework.web.server.handler.ExceptionHandlingWebHandler.lambda$handle$0(ExceptionHandlingWebHandler.java:77)
```

- ReactorDebugAgent 라이브러리를 통해 별도의 코드 수정 없이 로그를 확인할 수 있음
- StackTrace도 마찬가지로 출력됨

