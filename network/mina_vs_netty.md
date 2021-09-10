
## mina vs netty

- 공식 문서: netty
- 성능: netty
- 코드 량: netty 이 적음
- 상위 버전 호환: netty

## netty 상대적 장점

- java nio ByteBuffer 개선한 ChannelBuffer 를 사용하여 사용성 및 성능 개선
  - 제로카피 일부 지원
  - 동적 버퍼 제공
  - flip() 호출 안해도 됨

