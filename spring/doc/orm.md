# hibernate

## 주의사항

### @Temporal
- java.util.Date -> java.sql.Date 변환
- milliseconds 제대로 안나오는거 해결 가능

### @Version

### Cascade.Remove vs with OrphanRemoval

### n+1
- 관계 객체 조회 시 미리 fetch 안하면 join 해도 proxy 가 넘어와서 접근할 때마다 포풍 select

### annotation field vs getter
- 캡슐화 허용 여부(캡슐화 위험성이 부각됨. mapping 이 필요한 경우 활용 가능)
- @Transient
- [참고링크](https://stackoverflow.com/questions/594597/hibernate-annotations-which-is-better-field-or-property-access)

### save vs persist
- save
  - tx 무관하게 즉각 insert 발생
  - 곧바로 영속성 인스턴스에 등록됨
  - id 중복 문제를 회피할 수 있음
  - 여러 object 를 동시에 처리할 때 유용함
- persist
  - flush 또는 commit 시 insert 발생
  - 많은 object 를 동시에 처리할 때 id 중복을 주의해야 함
```java

Department dep1;
Agent agent1;
Agent agent2;

@Before
public void before() {
  dep1 = new Department("dep1");
  agent1 = new Agent("a1");
  agent2 = new Agent("a2");
}

// @Transactional 생략
@Test
public void test() {
  agent1.setDepartment(dep1);
  agent2.setDepartment(dep1);
  
  // 1. persist
  session.persist(agent1);
  session.persist(agent2); // duplicated error!

  // 2. save
  session.save(agent1);
  session.save(agent2); // work!
}

// @Transactional 생략
@Test
public void test2() {
  session.persist(dep1);
  dep1 = session.find(Department.class, dep1.getId()); // dep1 <- get from persist instance!

  agent1.setDepartment(dep1);
  agent2.setDepartment(dep1);
  
  session.persist(agent1);
  session.persist(agent2); // work!
}
```
