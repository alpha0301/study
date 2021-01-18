

# 정의 방식

```java
@Bean
public RouterFunction<ServerResponse> personRoute(PersonRepo repo) {
    return route(GET("/persons/{id}"), req -> Mono.justOrEmpty(req.pathVariable("id"))                                             
                                                 .flatMap(repo::getById)
                                                 .flatMap(p -> ok().syncBody(p))
                                                 .switchIfEmpty(notFound().build()));
}
```

# controller 대신 써야 하는 이유??

```java
@RestController
@RequestMapping("persons")  // annotation 순회 파싱때문에 발생하는 초기화 비용
public class PersonController { 

    @Autowired
    private PersonRepo repo;

    @GetMapping("/{id}") 
    public Mono<Person> personById(@PathVariable String id){
        retrun repo.findById(id);
    }
}
```

- 대용량 처리를 위한 reactive 구성에 적합하지 않음
- 위 코드 작업을 위한 기본 구성(Servlet Stack: ServletContext, DispatcherServlet) 설계가 동기(synchronous)
- reactive 하게 만들려고 Mono 를 반환하더라도 Servlet 3.0 스펙에 따라 내부의 동기 구조를 그대로 통과하게 됨
  - 예) filter 로직은 동기처리
- 마이크로서비스, 아마존 람다 및 기타 클라우드 서비스와 잘 협업할 수 있는 경량 애플리케이션을 만들 수 있어야 해서 webflux 모듈을 통합하기로 결정

```java
// @SpringBootApplication 없이 필요한 최소한의 기능만 사용하여 개발된 예제

class StandaloneApplication { 
    public static void main(String[] args) { 
        HttpHandler httpHandler = RouterFunctions.toHttpHandler(
           routes(new BCryptPasswordEncoder(18))
        ); 

        ReactorHttpHandlerAdapter reactorHttpHandler = new ReactorHttpHandlerAdapter(httpHandler); 

        HttpServer.create() 
            .port(8080) 
            .handle(reactorHttpHandler) 
            .bind() 
            .flatMap(DisposableChannel::onDispose) 
            .block(); 
    }

    static RouterFunction<ServerResponse> routes(PasswordEncoder passwordEncoder ) { 
        return
            route(
                POST("/check"), 
                request -> request 
                          .bodyToMono(PasswordDTO.class)
                          .map(p -> passwordEncoder 
                              .matches(p.getRaw(), p.getSecured())) 
                          .flatMap(isMatched -> isMatched 
                              ? ServerResponse 
                                  .ok() 
                                  .build() 
                              : ServerResponse 
                                  .status(HttpStatus.EXPECTATION_FAILED) 
                                  .build() 
                           ) 
                ); 
    }
}
```

[참고링크](https://stackoverflow.com/questions/47092029/difference-between-controller-and-routerfunction-in-spring-5-webflux)
