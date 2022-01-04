# 계층형 아키텍처의 문제점

![image](https://user-images.githubusercontent.com/39113923/148009149-7545868d-5ede-4740-a31d-0eea240f59ad.png)

## 구체적인 것을 의존함으로 인해 발생하는 문제점

- 서비스 -> 영속성 레이어 의존
  - 영속성 레이어에서 발생하는 다양한 문제들의 여파를 격리할 수 없음
  - 비즈니스 코드 작성 시 entity loading, tx, cache flush 등에 대한 작업을 계속 해야 함

## 기준이 레이어이기 때문에 발생하는 문제점

- 클래스 넓이에 대한 제약이 없음
- 수 많은 기능을 가진 서비스는 동시 작업이 매우 어려움
- 특정 기능만 테스트를 하기 위해 초기화해야 할 서비스 기능이 방대

```java
// 공통 서비스에는 crud 메소드가 정의되어 있음
interface UserService extends CommonService {
  void reportBadUser(int id);
  void punish(Punishing punishing, int id);
  void disqualify(int id);
  void reSignIn(int id);  
  // 등등 온갖 기능
}

```

## 클린 아키텍처에서 말하는 의존성 방향

![image](https://user-images.githubusercontent.com/39113923/148010285-3c05c7de-2d69-4d6b-bb05-ec5e9be3d61d.png)

![image](https://user-images.githubusercontent.com/39113923/148011232-189d4203-92f8-476a-9cdd-325c9dc82f8f.png)

- 영속성을 포함하여 변경 가능성이 높은 영역의 구현은 adapter 로 처리하고 접근은 유연하게 port interface
- i/o 구간을 추상화하여 구조적으로 비동기 처리에도 유리함

### 패키지 구조

📦orderapp
 ┗ 📂adapter
   ┗ 📂out
     ┗ 📂infrastructure
       ┗ 📂persistence
         ┣ 📜OrderRecordEntity.java
         ┣ 📜OrderRecordRepository.java
         ┗ 📜RecordOrderAdapter.java
   ┗ 📂in
     ┗ 📂presentation
       ┣ 📜AdminReceiptController.java
       ┣ 📜PhoneOrderController.java
       ┣ 📜PhoneOrderResult.java
       ┗ 📜WebOrderController.java
 ┗ 📂application
   ┗ 📂order
     ┗ 📂port
       ┗ 📂in
         ┣ 📜GetReceiptUseCase.java
         ┣ 📜OrderRequest.java
         ┣ 📜OrderResult.java
         ┣ 📜PlaceOrderUseCase.java
         ┗ 📜ReceiptResult.java
       ┗ 📂out
         ┣ 📜GetOrderRecordPort.java
         ┣ 📜OrderRecord.java
         ┗ 📜RecordOrderPort.java
     ┗ 📂service
       ┣ 📜GetReceiptService.java
       ┗ 📜PlaceOrderService.java
 ┗ 📂domain
   ┗ 📂order
     ┣ 📜Order.java
     ┗ 📜Receipt.java

- 웹, 영속성 의존성 묶음 쉽게 격리 가능
- 
