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

# 클린 아키텍처에서 말하는 의존성 방향

![image](https://user-images.githubusercontent.com/39113923/148010285-3c05c7de-2d69-4d6b-bb05-ec5e9be3d61d.png)

![image](https://user-images.githubusercontent.com/39113923/148011232-189d4203-92f8-476a-9cdd-325c9dc82f8f.png)

- 영속성을 포함하여 변경 가능성이 높은 영역의 구현은 adapter 로 처리하고 접근은 유연하게 port interface
- i/o 구간을 추상화하여 구조적으로 비동기 처리에도 유리함

## 패키지 구조

```bash
buckpal
└── account
    ├── adapter
    │   ├── in
    │   │   └── web
    │   │       └── AccountController
    │   ├── out
    │   │   └── persistence
    │   │       ├── AccountPersistenceAdapter
    │   │       └── SpringDataAccountRepository
    ├── domain
    │   ├── Account
    │   └── Activity
    └── application
        ├── SendMoneyService
        └── port
            ├── in
            │   └── SendMoneyUseCase
            └── out
                ├── LoadAccountPort
                └── UpdateAccountStatePort
``` 

- 웹, 영속성 의존성 묶음 쉽게 격리 가능(adapter)
- 서비스는 in 포트에 숨겨지므로 public 일 필요가 없음
- DDD 개념에 직접적으로 대응 가능

# UseCase 구현

아래과 같은 일반적인 단계를 따름

1. 입력을 받는다.
2. 비즈니스 규칙을 검증한다.
3. 모델 상태를 조작한다.
4. 출력을 반환한다.

### 입력 검증 

- 도메인 로직만 남으면 좋지만 검증 코드에 대한 책임도 있음
- 코드를 분리하는게 아무래도 좋음
- 생성자 검증 vs builder 활용
  - builder 패턴의 보일러 플레이트 생성 및 필드 누락 가능성 보다는 생성자를 쓰는게 좋음
  - SelfValidating<T> 상속해서 this.validateSelf(); 활용

### 비즈니스 규칙 검증

- 도메인 엔티티 내에서 동작을 수행할 때 검증 로직을 수행하는 것이 바람직
- UseCase 내에서 도메인 로직을 수행하기 직전에 수행해도 괜찮은 방법
- 비즈니스 규칙 검증은 사실상 도메인 로직에 포함한다고 봐도 될 것으로 보임
  
### 도메인 모델 내에 로직을 구현하는 것에 대하여
  
- 외부 의존성이 필요하지 않은 로직에 대해 격리할 수 있으므로 장점이 있음
- 팀의 개발 규칙에 따라야(UseCase 에 모든 규칙을 정의하기로 했는데 엔티티 내에 로직이 있다면...)
- 불필요하게 UseCase 의존성이 퍼지는 문제를 피할 수 있음
