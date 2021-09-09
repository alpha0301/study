

# 멀티쓰레드 + 동기 vs 싱글쓰레드 + 비동기

## 멀티쓰레드 + 동기 단점

- 프레임워크 입장에서 비즈니스 로직의 블로킹을 예상할 수 없음
- CPU를 최대한 효율적으로 사용하기위해 멀티쓰레드는 쓰레드 수가 급격하게 늘어날 수 있음
- CS(Context Switching) 많이 발생하면 매우 비효율적
- 주 메모리만 사용하던 옛날 CPU와 달리 캐시 메모리를 많이 사용하므로 CS 발생 시 효율이 더 떨어짐
- 단일 클라이언트에 대한 반응은 빠르지만 확장성이 떨어짐(쓰레드 유저 1:1 구조로는 서버를 아무리 이어붙혀도...)

## 싱글쓰레드 + 비동기 단점

- 프레임워크 입장에서 비즈니스 로직의 논블로킹을 강요함
- 콜백 구조가 필연적인 문제가 있음(java)
- 예상치 못한 블로킹 발생 시 시스템 전체가 마비될 수 있음


# Architecture

![image](https://user-images.githubusercontent.com/39113923/132642969-bff2942a-e5e5-4e9b-96b5-90b2b188f6c6.png)

- IoService : I/O 동작
- IoFilterChain : raw bytes 와 개발 시 요구하는 자료 구조 간의 상호 변환
- IoHandler : 비즈니스 로직 구현

## Server

![image](https://user-images.githubusercontent.com/39113923/132643892-ae3747a4-878c-47f5-9835-ae3581d27a9a.png)

- 개별 연결 마다 세션 생성됨(id: ip 와 port 조합)

## Client

![image](https://user-images.githubusercontent.com/39113923/132644967-ce1a176d-39ce-43e5-8873-5c858f16263d.png)

- 세션에 write 하면 filter chain 을 거친 후 server 로 전달







