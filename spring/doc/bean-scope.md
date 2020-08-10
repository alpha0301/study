# 개요

- 스프링 컨테이너에서 bean 라이프사이클을 관리하는 규칙
- singleton(default), prototype, request(web), session(web), global session(web)

## Singleton

- single bean cache 에 저장
- 요청 시 캐시된 객체를 즉시 반환
- scope 정보 미지정 시 기본값이지만 명시적으로 지정 가능
- @Scope("singleton")

## Prototype

- 모든 요청에 대해 새로운 객체를 생성함
- GC 대상
- bean lifecycle 그대로 따름
- @Scope("prototype")
