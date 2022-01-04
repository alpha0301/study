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

## 입력 검증 

- 도메인 로직만 남으면 좋지만 검증 코드에 대한 책임도 있음
- 코드를 분리하는게 아무래도 좋음
- 생성자 검증 vs builder 활용
- builder 패턴의 보일러 플레이트 생성 및 필드 누락 가능성 보다는 생성자를 쓰는게 좋음
- SelfValidating 상속해서 this.validateSelf()

## 비즈니스 규칙 검증

- 도메인 엔티티 내에서 동작을 수행할 때 검증 로직을 수행하는 것이 바람직
- UseCase 내에서 도메인 로직을 수행하기 직전에 수행해도 괜찮은 방법
- 비즈니스 규칙 검증은 사실상 도메인 로직에 포함한다고 봐도 될 것으로 보임
  
## 도메인 모델 내에 로직을 구현하는 것에 대하여
  
- 외부 의존성이 필요하지 않은 로직에 대해 격리할 수 있으므로 장점이 있음
- 팀의 개발 규칙에 따라야(UseCase 에 모든 규칙을 정의하기로 했는데 엔티티 내에 로직이 있다면...)
- 불필요하게 UseCase 의존성이 퍼지는 문제를 피할 수 있음
  
# 웹 어댑터
  
## 책임
  
- map request to pojo
- permission
- validation
- map input to model for UseCase
- call UseCase
- out from UseCase to response
- return response

## Controller에서 하면 안되나?

- 이미 많은 책임이 fw에서 대신 처리해주는 세상이지만 만능은 아님
  
## Controller 나누기
  
- Service 와 마찬가지로 횡으로 넓은 Controller 는 테스트를 어렵게 하고 merge conflict 를 많이 발생 시킴
- 작고 많은 클래스 코딩에 익숙해져야
- Controller 는 전체 시스템 테스트, 어댑터들은 다 단위테스트가 쉽게 가능

# 영속성 어댑터
  
## 책임

- map input to db format
- send input to db
- map output to app format
- return output

## Repository 가 이미 꽤 다양한 영속성 영역을 커버해주는데?

- redis, es, cassandra, mongodb etc...
- 영속상태 관리, 캐시 처리를 해놨는데 다른 저장소를 연동했더니 망했네
- 사실 실제 업무에서는 interface Repository 가 port out 역할

## Repository 나누기

- Controller 와 마찬가지로 횡으로 넓은 Repository 문제

## Transaction?

- 영속성과는 기술적 접함이 있지만 Service 에 위임해야 알맞음
- 어느 범위의 시나리오가 Atomic 인지, 오류 시 Rollback 범위는 어디까지인지

# 테스트

비용이 적고, 유지보수하기 쉽고, 빨리 실행되고, 안정적인 작은 테스트들에 대해 커버리지를 높게 유지하는 것이 중요

## 비용

시스템 테스트 >> 통합 테스트 >> 단위 테스트

## 단위테스트 주의사항

ex) SendMoneyServiceTest

- 상태 검증이 안되고 메서드 호출 등 상호작용 여부를 체크하는 경우, 행동 변경과 동시에 구조 변경에도 취약해짐
  - 리팩토링 -> 테스트 코드 변경 가능성 매우 높음
- 의미없는 모든 동작 검증 보다는 핵심 동작 검증에 집중해야 테스트 코드 변경을 줄일 수 있음

## 웹 어댑터는 통합 테스트가 나을까

- Controller 가 강하게 Spring 과 묶여 있음
- 웹 어댑터 단위 테스트로는 눈에 보이지 않는 웹, 네트워크와의 의존성 때문에 프로덕션 환경에서 정상적으로 동작할 지 확신할 수 없게 됨

## 영속성 어댑터는 통합 테스트가 나을까

- 웹 어댑터와 동일한 이유로 통합 테스트가 합리적
- 모킹 DB와 실제 DB간의 동작이 너무나도 달라서 현실적으로 통합 테스트를 하는게 나음
- ex) H2 에서는 인덱스 명칭이 전역으로 유니크하지만 Mysql 에서는 그렇지 않음
- Embedded Mysql같은 것도 있지만 TestContainer 이 상당히 유용해보임

## 테스트 코드는 얼마나 짜야 할까?

- 커버리지 기준이 되면 100% 가 아니면 의미가 없어짐(심지어 100% 라도 버그가 발생할 수 있음)
- 더 잦은 배포가 더 잦은 테스트를 수행하게 되고 신뢰도는 높아짐
- 프로덕션 버그에 대해 복기하고 메꿀 수 있도록 테스트를 발전시키는 것이 중요

### 육각형 아키텍쳐에서 사용하는 테스트 코드 작성 전략

- domain entity : 단위 테스트
- UseCase : 단위 테스트
- Adapter : 통합 테스트
- 프로덕션 사용 시나리오 : 시스템 테스트

귀찮은 작업이 아니려면 실제 기능이 구현된 후가 아니라 구현하는 도중에 작업이 이루어지는것이 좋음

# 경계 간 매핑 전략

## 매핑하지 않기

- 웹, 영속성 관련 의존성을 하나의 모델에서 모두 처리해야 함
- 아주 간단한 CRUD 작성 시에는 효과적일 수 있음
- 나중에 필요할 때 얼마든지 매핑 전략을 바꿀 수 있으므로 기계적으로 매핑하는 습관에 대해 고민해봐야

## 양방향 매핑

- 매핑하지 않는 전략 만큼 명확
- 웹 모델, UseCase 도메인, 영속성 모델
- 단일 책임 원칙을 완벽하게 만족
- silver bullet 이 아니라는 점을 명심해야
  - 수 많은 보일러 플레이트
  - 웹 모델이 그대로 request, response 에 전달되는 경우가 많은데 외부 계층의 변경 요구에 취약해짐

## 완전 매핑

- 양방향 매핑 + 입출력 시 별도의 모델(Command)
- ex) 유효성 검증
- 전역으로 적용하지 말고 필요에 따라 사용하는 것이 중요
- 매핑 전략은 한 가지가 전역 패턴으로 되기 보다는 요구사항에 따라 섞어 써야

## 단방향 매핑 전략

- 모델에 대한 interface(보통 State 로 많이 표현)
- 각 계층의 의존성이 필요할 때가 아니면 getter 만 노출(getter 안에서 코딩하는 사람도 있는데...)
- 계층간의 모델이 거의 비슷하면서도 웹, 영속성 관련 의존성이 필요할 때 효과적

## 언제 어떤 매핑 전략을 사용할 것인가?

- 어제의 최선이 오늘은 아닐수도
- 팀 내에서 합의할 수 있는 가이드라인을 정하는 것이 중요(명확한 근거를 바탕으로)

### 변경 UseCase 와 조회 UseCase 의 매핑 전략을 다르게 가져갈 경우

- 변경 UseCase : 확실한 유효성 검증과 UseCase 에서 필요한 필드만을 확실히 다루어야 하므로 완전 매핑 전략이 알맞음
  - 우선 빠른 코드 작업을 위해 매핑하지 않는 전략으로 작성
  - 영속성 문제를 다룰 필요가 생기면 완전 매핑으로 작성
- 쿼리 UseCase : 양방향 매핑 전략이 알맞으며 마찬가지로 시작은 매핑하지 않는 전략

# 애플리케이션 조립

## 왜 계층형에서 의존성 역전을 해야 할까

- 의존성은 반드시 외부에서 내부로 향하도록 해야 함
- 계층형은 사실상 영속성 계층에서 외부로 향하도록 되어있기 때문에 데이터베이스 기반 아키텍쳐라고 볼 수 있음
- 다양한 Repository 형태가 필요한 시점에서는 곤란한 구조
- 외부의 변화에 대한 대응 코드를 격리함으로써 수 많은 이득을 얻을 수 있음

## IoC 적절한 활용법

- 스프링 클래스패스 스캐닝은 다 좋지만 단위 테스트, 통합 테스트에서 비효율적
  - 단위테스트 하는데 @Component 로 전역 탐색을 하고 있으니...
  - 만약 테스트시에 context 에 적재하면 안되는 인스턴스가 있다면...?
- @Configuration 을 통해 필요한 부분만 조립
  - @Bean 으로 적재할 떄 new 를 써야 되는데 같은 패키지가 아니면 public 강요되네?

## 의존성 방향을 강제할 수 없을까

- 접근제한자로 하면 좋지만 전역 클래스 스캐닝이 필수적
- 작은 모듈에서 접근제한자가 효과적

### 컴파일 후 런타임에서 체크

- ArchUnit 과 같은 도구를 통해 외부에서 내부로의 의존성 규칙이 잘 지켜지도록 테스트 코드를 작성할 수 있음
- 현실적인가..? (DependencyRuleTests)

### 작게 쪼개서 빌드

- 의존 방향이 유지되기를 바라는 영역을 별도의 아티펙트(jar)로 쪼개서 빌드
- 자바 컴파일러는 순환 참조를 못잡아내지만 빌드 툴은 순환 참조를 효과적으로 예방
- 개발, 테스트, 빌드 및 배포를 완전히 분리해서 작업 가능
- 빌드 스크립트 관리, jar 의존성 버전 관리, 멍청한 IDE 인덱싱 등등 다양한 문제점이 있으니 적당히 해야...

# 지름길?

## UseCase 간 모델이 동일할 때

- 빠른 코딩, 중복 제거를 위해 같은 모델을 공유할 때가 있음(SendMoneyCommand)
- 각 UseCase 가 앞으로 독립적으로 진화할 것으로 보이면 과감하게 분리해야
- 설계 원칙의 중요성 >> 중복 제거
