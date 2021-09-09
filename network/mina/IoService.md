
## 개요

 ![image](https://user-images.githubusercontent.com/39113923/132648386-b3b3a6f4-f263-4b0b-9877-2b45e7176289.png)

 - 기본 I/O 서비스를 제공하고 I/O 세션을 관리
 
## Responsibilities

- 세션 생명주기 관리 및 idleness 감시
- filter chain 관리
- 새로운 메시지에 대한 handler 호출
- 보낸 메시지에 대한 다양한 통계 제공
- listeners 관리
- 데이터 전송 처리

## interface

- getTransportMetadata()
  - 공급자 정보: nio, apr, rxtx
  - connection type: connectionless / connection oriented
- 서버 생명주기, 세션관리, filter chain 관리, handler 관리 등등..

## implements

- IoAcceptor : 서버
- IoConnector : 클라이언트

