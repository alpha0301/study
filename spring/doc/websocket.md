## 웹소켓

- HTTP와는 다른 TCP 프로토콜이지만 포트 80 및 443을 사용하고 기존 방화벽 규칙을 다시 사용할 수 있도록 HTTP를 통해 작동하도록 설계
- WebSocket 프로토콜 인 RFC 6455 는 단일 TCP 연결을 통해 클라이언트와 서버간에 전이중 양방향 통신 채널을 설정하는 표준화 된 방법을 제공[링크](https://tools.ietf.org/html/rfc6455)

### Handshake

```
HTTP/1.1 101 Switching Protocols 
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: 1qVdfYHU9hPOl4JYYNXF623Gzn0=
Sec-WebSocket-Protocol: v10.stomp
```

- Connection: Upgrade : HTTP 사용 방식을 변경
- Upgrade : websocket : WebSocket을 사용
- Sec-WebSocket-Protocol: xxx, yyy, zzz : WebSocket을 쓰면서 이 중에서 protocol을 골라서 씀
- Sec-WebSocket-Key : 보안을 위한 요청 키

```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

- 101 Switching Protocols : Handshake 요청 내용을 기반으로 다음부터 WebSocket으로 통신
- Sec-WebSocket-Accept : 보안을 위한 응답 키 - base64.encode(Sec-WebSocket-Key.concat(GUID))


### 언제 사용?

- 자주 + 많은 양의 + 지연이 짧아야 하는 통신을 할 수록 WebSocket이 적합
- 채팅, 게임 등

### 기본 설정

```java
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {
    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(new MyHandler(), "/myHandler").setAllowedOrigins("*");
    }
}
```

#### SocketJS

- 브라우저가 지원안하면 중간에 polling 처리해주고 내부적으로 같은 코드를 사용할 수 있게 해줌

```java
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {
    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(new MyHandler(), "/myHandler").setAllowedOrigins("*").withSockJS();
    }
}
```

### STOMP

- Simple/Streaming Text Oriented Messaging Protocol
- 중간에 브로커를 두고 알맞는 Subscriber 에게 분배
- MessageHandler 구현하면 됨

![simple](https://docs.spring.io/spring-framework/docs/5.2.9.RELEASE/spring-framework-reference/images/message-flow-simple-broker.png)
![ext](https://docs.spring.io/spring-framework/docs/5.2.9.RELEASE/spring-framework-reference/images/message-flow-broker-relay.png)

#### 설정

```java
import org.springframework.web.socket.config.annotation.EnableWebSocketMessageBroker;
import org.springframework.web.socket.config.annotation.StompEndpointRegistry;

@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/portfolio").withSockJS();  // http://{address}/portfolio 접속 시 STOMP 시작
    }

    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        config.setApplicationDestinationPrefixes("/app"); // /app/{MAPPING}
        config.enableSimpleBroker("/topic", "/queue"); 
    }
}
```

#### 메시지 흐름

```java
@Controller
public class GreetingController {

    @MessageMapping("/greeting") {
    public String handle(String greeting) {
        return "[" + getTimestamp() + ": " + greeting;
    }
}
```

#### 메시지 구조

```
COMMAND
header1:value1
header2:value2

Body^@
```

#### Connect

```
<<< CONNECTED
version:1.1
heart-beat:0,0
user-name:admin1
```

- 버전정보, 현재의 세션정보

#### Subscribe

```
>>> SUBSCRIBE
id:sub-0
destination:/topic/roomId
```

```
>>> SUBSCRIBE
id:sub-1
destination:/topic/out
```

```
>>> SUBSCRIBE
id:sub-2
destination:/topic/in
```

#### Message

```
<<< MESSAGE
destination:/topic/roomId
content-type:application/json;charset=UTF-8
subscription:sub-0
message-id:asdfasdf-12
content-length:43

{"sender":"admin1","message":"안뇽"}
```

#### Disconnect

```
>>> DISCONNECT
```






