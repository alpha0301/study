# Initialize Method

- Object 및 DI 완료 후 호출


## @PostConstructor

- JSR-250 스펙이므로 같은 스펙을 구현한 다른 프레임워크에서도 동작
- 스프링에 비교적 덜 의존적임

```java
@Slf4j
@Component
public class SimpleBean {

    @PostConstruct
    public void postConstruct() {
        log.info("postConstruct");
    }
}
```

## interface InitializeBean

- afterPropertiesSet 메소드를 호출함

```java
@Slf4j
@Component
public class SimpleBean implements InitializingBean {

    @Override
    public void afterPropertiesSet() throws Exception {
        log.info("afterPropertiesSet");
    }
}
```

## @Bean(initMethod)

- init method 를 String 으로 지정

```java
@Configuration
public class TestConfiguration {

    @Bean(initMethod = "init")
    public SimpleBean simpleBean() {
        return new SimpleBean();
    }

    @Slf4j
    public static class SimpleBean {

        public void init() throws Exception {
            log.info("init");
        }
    }
}
```

# Destroy Method

- 스프링 컨테이너 종료 시 호출함

## @PreDestroy

- @PostConstruct 처럼 JSR-250 스펙

```java
@Slf4j
@Component
public class SimpleBean {

    @PreDestroy
    public void preDestroy() {
        log.info("preDestroy");
    }
}
```

## interface DisposableBean

- destory Method 호출

```java
@Slf4j
@Component
public class SimpleBean implements DisposableBean {

    @Override
    public void destroy() throws Exception {
        log.info("destroy");
    }
}
```

## @Bean(destroyMethod)

- destroy method  String 으로 지정

```java
@Configuration
public class TestConfiguration {

    @Bean(destroyMethod = "destroy")
    public SimpleBean simpleBean() {
        return new SimpleBean();
    }

    @Slf4j
    public static class SimpleBean {

        public void destroy() throws Exception {
            log.info("destroy");
        }
    }
}
```
