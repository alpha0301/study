## 개요

논블로킹 디코딩을 구현한 ByteToMessageDecoder 확장 클래스

[api 링크](https://netty.io/4.1/api/io/netty/handler/codec/ReplayingDecoder.html)


## ByteToMessageDecoder 와 차이점

필요한 바이트의 가용성을 확인하는 대신 모든 필요한 바이트가 이미 수신된 것 처럼 decode(), decodeLast() 를 구현할 수 있습니다.

### ByteToMessageDecoder 인 경우

```java
public class IntegerHeaderFrameDecoder extends ByteToMessageDecoder {

    @Override
   protected void decode(ChannelHandlerContext ctx,
                           ByteBuf buf, List<Object> out) throws Exception {

     if (buf.readableBytes() < 4) {
        return;
     }

     buf.markReaderIndex();
     int length = buf.readInt();

     if (buf.readableBytes() < length) {
        buf.resetReaderIndex();
        return;
     }

     out.add(buf.readBytes(length));
   }
 }
```

### ReplayingDecoder 인 경우

```java
public class IntegerHeaderFrameDecoder
      extends ReplayingDecoder<Void> {

   protected void decode(ChannelHandlerContext ctx,
                           ByteBuf buf, List<Object> out) throws Exception {

     out.add(buf.readBytes(buf.readInt()));
   }
 }
```

## 동작 방식

- ReplayingDecoder 는 버퍼에 충분히 데이터가 쌓이지 않았을 때 정의된 에러를 던지도록 구현된 특수한 ByteBuf 를 사용함
- 위 예제(IntegerHeaderFrameDecoder)에서 만약 buf에 4 bytes 가 없다면 에러를 생성하고 ReplayingDecoder 에서 buf 의 readerIndex 를 'initial' 위치로 되감음
- 캐싱된 에러를 던지기 때문에 매번 에러를 생성하는 비용이 발생하지 않음

## 제약

- 일부 버퍼 작업은 수행할 수 없음
- 네트워크가 느리고 메시지가 복잡한 경우, 성능이 저하될 수 있음(운영체제가 충분한 bytes 를 보내지 않을 경우에 같은 메시지를 읽었다가 버리는 것을 반복할 수 있음)

### 복잡한 메시지를 처리할 때 성능을 향상시키는 법

- checkpoint() 를 사용하면 'initial' 위치를 별도로 업데이트하면서 유지할 수 있음
- checkpoint() 를 호출한 마지막 위치로 readerIndex 를 되감을 수 있음
- 메시지가 여러 구간으로 나눠진 경우에 각 구간 시작 시 checkpoint() 를 enum 과 함께 호출하여 구간 표시를 할 수 있음

```java
public class Asm4TcpMessageDecoder extends ReplayingDecoder<Asm4TcpMessageDecodeState> {
    private int length;
    private Asm4TcpMessage packet;

    public Asm4TcpMessageDecoder() {
        super(Asm4TcpMessageDecodeState.READ_BODY_LENGTH);
    }

    @Override
    protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) throws Exception {
        switch (state()) {
            case READ_BODY_LENGTH:
                length = in.readInt();
                checkpoint(Asm4TcpMessageDecodeState.READ_HEADER);
                break;
            case READ_HEADER:
                packet = new Asm4TcpMessage();
                packet.setLength(length);
                packet.setRequest(in.readShort());
                packet.setPriority(in.readShort());
                packet.setStatusCode(Asm4TcpMessageStatus.valueOf(in.readInt()));
                packet.setRttl(in.readShort());
                packet.setProcessorId(in.readShort());
                packet.setTransactionId(in.readInt());
                checkpoint(Asm4TcpMessageDecodeState.READ_BODY);
                break;
            case READ_BODY:
                ByteBuf frame = in.readBytes(length);
                checkpoint(Asm4TcpMessageDecodeState.READ_BODY_LENGTH);
                packet.setBody(frame.array());
                out.add(packet);
                break;
            default:
                throw new Error();
        }
    }
}
```
