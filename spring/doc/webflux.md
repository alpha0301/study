# Reactive Core

## spring-web module

- HttpHandler: 다양한 http servers에 대한 handler 추상화
- WebHandler: 웹 어플리케이션에서 사용하는 광범위한 기능들을 제공

# Functional Endpoint

## HandlerFunction

- @RequestMapping에 대응되는 function
- ServerRequest -> Mono\<ServerResponse\>(delayed ServerResponse)
- request body는 Flux, Mono를 포함한 모든 Reactive Streams Publisher로 표현됨

## RouterFunction

- @RequestMapping + behavior
- Incoming request -> Mono\<HandlerFunction\>
- 매칭되는 RouterFunction이 있으면 HandlerFunction이 반환됨(else return empty Mono)
- 일반적으로는 따로 구현하지 않고 RouterFunction 내의 static 유틸 메소드를 사용해서 생성함

### Predicates

- 분기 처리는 router 내에 parameter 형식에 따라 동작이 바뀔 수 있도록 overload 로 구현되어 있음
- 분기 종류
  - request path
  - http method
  - content-type
  - etc

```java
RouterFunction<ServerResponse> route = RouterFunctions.route()
  .GET("/hello-world"), accept(MediaType.TEXT_PLAIN), // GET 메소드, hello-world path, MediaType 으로 각각 분기 처리되고 있음
    request -> ServerResponse.ok().bodyValue("Hello World")).build();
```

- 분기를 여러 개 조합하는 경우
  - RequestPredicate.and(RequestPredicate)
  - RequestPredicate.or(RequestPredicate)

```java
RouterFunction<ServerResponse> route = RouterFunctions.route(
            RequestPredicates.GET("/hello").or(RequestPredicates.GET("/world"))
                    .and(RequestPredicates.accept(MediaType.APPLICATION_JSON)),
            request -> ServerResponse.ok().contentType(MediaType.APPLICATION_JSON)
                    .body(BodyInserters.fromValue("hello world"))
    );
```

### Routes

- router function들은 등록된 순서대로 비교해서 선택되기 때문에 구체적으로 명세한 것 부터 등록해야 의도하지 않은 선택이 발생하는 것을 피할 수 있음
- router function들을 Spring bean으로 등록할 때에도 동일하게 중요함
- annotation-based(예전 방식)에서는 순서와 관계 없이 구체적으로 명세한 메소드가 자동으로 선택됨

### Nested Routes

- predicate의 파라미터로 predicate를 넣어서 중첩해서 사용할 수 있음
- 재활용 가능

```java

```

# DispatcherHandler

예전 방식

- front controller pattern
- Spring Configuration을 바탕으로 delegate components를 찾음
- context에 접근하기 위해 ApplicationContextAware를 구현하고 있음

## special beans

- DispatcherHandler에 의해 탐지됨
- request 처리 및 적절한 response를 render하기 위해 필요
- HandlerMapping: map a request to handler
  - RequestMappingHandlerMappinng: for @RequestMapping
  - RouterFunctionMapping: for functional endpoint routers
  - SimpleUrlHandlerMapping: for 명시적 URI path 등록 및 WebHandler instances

## WebFlux Config

## Processing

- DispatcherhHandler 가 요청을 처리하는 순서
  1. 각 HandlerMapping은 매칭되는 handler를 탐색해서 처음으로 발견한 것을 사용함(순서는 어떻게 결정되는거지?)
  2. 적합한 HandlerAdapter를 통해 실행되고 HandlerResult를 반환함. 
  3. 몇 가지 추가 context와 함께 HandlerResultHandler 중 첫번째 호환되는 구현체에 전달됨
  4. HandlerResultHandler들은 WebFLuxConfig에 선언되어있음

### Implementations of HandlerResultHandler

- ResponseEntityResultHandler: @Controller -> ResponseEntity
- ServerResponseResultHandler: functional endpoints -> ServerResponse
- ResponseBodyResultHandler: @ResponseBody or @RestController
- ViewResolutionResultHandler: HTML 형태로 만들기 위한 View instance(String, CharSequence, Model, Map...)

## Exceptions

- HandlerResult는 에러 처리 function을 노출할 수 있고 아래와 같은 경우에 호출됨
  - @Controller와 같은 handler를 호출하는데에 실패했을 때
  - HandlerResultHandler가 반환한 value를 처리하다가 실패했을 때
 - 데이터 item들을 생성하는 handler로부터 reactive type이 반환되기 전 까지 에러 처리 function은 response를 고칠 수 있음
   - error status와 같은 것들
 - 주의사항: WebFlux에서는 handler를 선택하기 이전에 발생한 exception에 대해서는 @ControllerAdvice를 사용할 수 없음

