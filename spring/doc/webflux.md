# Reactive Core

## spring-web module

- HttpHandler: 다양한 http servers에 대한 handler 추상화
- WebHandler: 웹 어플리케이션에서 사용하는 광범위한 기능들을 제공

# DispatcherHandler

- front controller pattern
- Spring Configuration을 바탕으로 delegate components를 찾음
- context에 접근하기 위해 ApplicationContextAware를 구현하고 있음
- special beans
  - DispatcherHandler에 의해 탐지됨
  - request 처리 및 적절한 response를 render하기 위해 필요
  - HandlerMapping: map a request to handler
    - RequestMappingHandlerMappinng: for @RequestMapping
    - RouterFunctionMapping: for functional endpoint routers
    - SimpleUrlHandlerMapping: for 명시적 URI path 등록 및 WebHandler instances
