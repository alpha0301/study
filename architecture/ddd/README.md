# DDD 용어 정리

### Domain Logic

- 모델링의 목적
- 비즈니스 로직이라고 봐도 무방함
- 데이터 생성, 저장, 수정 방식에 대한 비즈니스 규칙을 정의

### Domain model

- 문제를 해결하는데에 필요한 idea, knowledge, data, metrics, and goals
- 복잡한 비즈니스 논리를 처리하는데에 필요한 모든 규칙과 패턴이 포함

### Subdomain

- 비즈니스 로직의 다른 부분을 참조하는 여러 하위 도메인
- 예) 온라인 소매점의 서브도메인: 제품 카탈로그, 재고 및 배송

### Design Pattern

- 코드 재사용을 위한 장치

### Bounded context

- DDD의 중심 패턴
- 특정 하위 도메인의 정의 및 적용하는 경계를 나타냄
- 이 컨텍스트 내에서 특정 하위 도메인 내에서만 사용하는 정의들이 존재함
- 다른 컨텍스트간의 소통은 별도의 어댑터를 정의해서 사용
- 의존성이 끊어지기 때문에 이 컨텍스트 내의 변경사항이 있더라도 전체 시스템을 변경할 필요가 없음

### The Ubiquitous Language

- 개발자와 타 영역의 전문가들이 소통할 때 동일한 언어로 사용하기 위한 방법론
- 일상생활 언어와 기술 언어는 차이가 있기 때문에 공통 언어를 정의해야 함

### Entities

- 사용자 또는 제품과 같은 데이터와 행동의 조합

### Value objects and aggregates

- VO는 attributes가 있지만, 자체적으로 존재할 수는 없음
- 도메인 모델에는 수 많은 엔티티들과 VO가 섞여있어 관리하기 쉬운 논리적 그룹으로 분류
- 이와 같은 그룹을 aggregates 라고 함
- 여러 연결된 개체를 하나의 단위로 취급하기 위해서 존재함
- aggregate root: 외부의 접근을 허용하기 위한 유일한 entity

### Domain Service

- 도메인 로직을 포함하는 Layer(additional)
- Entity, VO 와 마찬가지로 도메인 모델의 일부
- Application Service: 비즈니스 로직이 없지만, Domain Model 위에 배치된 Application의 활동을 coodinate 하기 위하여 존재

### Repository

- 데이터 인프라를 단순화하는 패턴
- 비즈니스 Entity 모음

