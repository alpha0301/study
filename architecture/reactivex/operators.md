# Chaning Operators

- 대부분의 operator는 Observable에 대해 동작하고 Observable을 return함
  - chain 내에서 순서를 보장되게 해줌
  - 각 operator는 이전 operator의 결과인 Observable을 수정하게 됨
  - build pattern은 순서가 중요하지 않지만 Observable은 순서가 중요함
