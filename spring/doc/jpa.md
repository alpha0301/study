
## Spring Data Repositories

- Spring Data repository abstraction
- data access layer 를 구현할 때 필요한 boilerplate 코드를 줄이는게 목적

### interface Repository

```java
public interface CrudRepository<T, ID> extends Repository<T, ID> {

  <S extends T> S save(S entity);      

  Optional<T> findById(ID primaryKey); 

  Iterable<T> findAll();               

  long count();                        

  void delete(T entity);               

  boolean existsById(ID primaryKey);   

  // … more functionality omitted.
}
```
#### 페이징
```java
public interface PagingAndSortingRepository<T, ID> extends CrudRepository<T, ID> {

  Iterable<T> findAll(Sort sort);

  Page<T> findAll(Pageable pageable);
}

/// 

PagingAndSortingRepository<User, Long> repository = // … get access to a bean
Page<User> users = repository.findAll(PageRequest.of(1, 20));

```

#### 파생 쿼리
```java
// Derived Query example
interface UserRepository extends CrudRepository<User, Long> {

  long countByLastname(String lastname);
  
  long deleteByLastname(String lastname);
  
  List<User> removeByLastname(String lastname);
}
```

#### 실사용 예제
```java
// JavaConfig
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;

@EnableJpaRepositories
class Config { … }

//////////////////

// how to use
class SomeClient {

  private final UserRepository repository;

  SomeClient(UserRepository repository) {
    this.repository = repository;
  }

  void doSomething() {
    List<User> persons = repository.findByLastname("Matthews");
  }
}
```

#### CRUD 중에 일부만 사용
```java
@NoRepositoryBean // 실제 쓰는게 아니니까
interface MyBaseRepository<T, ID> extends Repository<T, ID> {

  Optional<T> findById(ID id);

  <S extends T> S save(S entity);
}

interface UserRepository extends MyBaseRepository<User, Long> {
  User findByEmailAddress(EmailAddress emailAddress);
}
```

#### 여러 종류의 repository 환경에서 나쁜 예시
```java
interface JpaPersonRepository extends Repository<Person, Long> { … }

interface MongoDBPersonRepository extends Repository<Person, Long> { … }

@Entity
@Document
class Person { … }
```

#### 여러 종류의 repository 환경에서 관리 방법
```java
@EnableJpaRepositories(basePackages = "com.acme.repositories.jpa")
@EnableMongoRepositories(basePackages = "com.acme.repositories.mongo")
class Configuration { … }
```

### 쿼리 메소드 정의

#### Query Creation

- find…By, read…By, query…By, count…By, get…By 제거 후 나머지 구문 분석
- Distinct 사용 가능
- 첫번째 By 부터 실제 criteria 가 시작되는 지점
- 기초적인 수준에서 And, Or 지원

```java
interface PersonRepository extends Repository<Person, Long> {

  List<Person> findByEmailAddressAndLastname(EmailAddress emailAddress, String lastname);

  // Enables the distinct flag for the query
  List<Person> findDistinctPeopleByLastnameOrFirstname(String lastname, String firstname);
  List<Person> findPeopleDistinctByLastnameOrFirstname(String lastname, String firstname);

  // Enabling ignoring case for an individual property
  List<Person> findByLastnameIgnoreCase(String lastname);
  // Enabling ignoring case for all suitable properties
  List<Person> findByLastnameAndFirstnameAllIgnoreCase(String lastname, String firstname);

  // Enabling static ORDER BY for a query
  List<Person> findByLastnameOrderByFirstnameAsc(String lastname);
  List<Person> findByLastnameOrderByFirstnameDesc(String lastname);
}
```

#### PropertyExpression

- 참조 객체의 Field 에 접근할 수 있음

```java
// {person instance}.address.zipCode
List<Person> findByAddressZipCode(ZipCode zipCode);

// Person 에 addressZipCode 라는 Field 가 있을 때 모순 해결
List<Person> findByAddress_ZipCode(ZipCode zipCode);
```

#### Special Parameter

```java
Page<User> findByLastname(String lastname, Pageable pageable);

Slice<User> findByLastname(String lastname, Pageable pageable);

List<User> findByLastname(String lastname, Sort sort);

List<User> findByLastname(String lastname, Pageable pageable);
```

- Pageable, Sort 는 non-null value 이어야 함
- 만약 페이징이나 정렬이 필요없으면 Pageable.unpaged(), Sort.unsorted() 사용해야 함

#### Limits Query Result

```java
User findFirstByOrderByLastnameAsc();

User findTopByOrderByAgeDesc();

Page<User> queryFirst10ByLastname(String lastname, Pageable pageable);

Slice<User> findTop3ByLastname(String lastname, Pageable pageable);

List<User> findFirst10ByLastname(String lastname, Sort sort);

List<User> findTop10ByLastname(String lastname, Pageable pageable);
```

- Distinct 지원
- 예상하는 return 개수가 1개일 때 Optional keyword 지원

#### return Collections or Iterables

- Iterable, List, Set 지원
- Spring Data 의 Streamable 지원 (custom extension of Iterable)
- Streamable : filter, map, 다른 Stream 에 연결 제공

```java
interface PersonRepository extends Repository<Person, Long> {
  Streamable<Person> findByFirstnameContaining(String firstname);
  Streamable<Person> findByLastnameContaining(String lastname);
}

Streamable<Person> result = repository.findByFirstnameContaining("av")
  .and(repository.findByLastnameContaining("ea"));
```

#### Wrapper 를 통한 Streamable 기능 확장

```java
class Product { 
  MonetaryAmount getPrice() { … }
}

@RequiredArgConstructor(staticName = "of")
class Products implements Streamable<Product> { 

  private Streamable<Product> streamable;

  public MonetaryAmount getTotal() { 
    return streamable.stream() //
      .map(Priced::getPrice)
      .reduce(Money.of(0), MonetaryAmount::add);
  }
}

interface ProductRepository implements Repository<Product, Long> {
  Products findAllByDescriptionContaining(String text); 
}
```

#### java 8 Stream 사용

```java
@Query("select u from User u")
Stream<User> findAllByCustomQueryAndStream();

Stream<User> readAllByFirstnameNotNull();

@Query("select u from User u")
Stream<User> streamAllPaged(Pageable pageable);
```

```java
try (Stream<User> stream = repository.findAllByCustomQueryAndStream()) {
  stream.forEach(…);
}
```

- 주의사항1: 사용한 Stream 을 명시적으로 닫아야 함
- 주의사항2: 모든 Spring Data 모듈이 Stream 을 지원하는것은 아님

#### 비동기 지원

```java
@Async
Future<User> findByFirstname(String firstname);               

@Async
CompletableFuture<User> findOneByFirstname(String firstname); 

// org.springframework.util.concurrent.ListenableFuture
@Async
ListenableFuture<User> findOneByLastname(String lastname);    
```

#### Publishing Event

- aggregate root 란?
https://medium.com/@SlackBeck/%EC%95%A0%EA%B7%B8%EB%A6%AC%EA%B2%8C%EC%9E%87-%ED%95%98%EB%82%98%EC%97%90-%EB%A6%AC%ED%8C%8C%EC%A7%80%ED%86%A0%EB%A6%AC-%ED%95%98%EB%82%98-f97a69662f63

```java
@Entity
public class Aggregate {
 
    @Transient
    private final Collection<DomainEvent> domainEvents;
    // ...
    public void domainOperation() {
        // some domain operation
        domainEvents.add(new DomainEvent());
    }
 
    // call when saved
    @DomainEvents
    public Collection<DomainEvent> events() {
        return domainEvents;
    }
}
```

```java
@DisplayName("given aggregate with @DomainEvents,"
    + " when do domain operation and save,"
    + " then event is published")
@Test
void domainEvents() {
 
    Aggregate aggregate = new Aggregate();
 
    aggregate.domainOperation();
    repository.save(aggregate);
 
    verify(eventHandler, times(1)).handleEvent(any(DomainEvent.class));
}
```
```java
@AfterDomainEventPublication
public void clearEvents() {
    domainEvents.clear();
}
```

#### Extensions

- Querydsl 연동 지원
- WebSupport

```java
@Configuration
@EnableWebMvc
@EnableSpringDataWebSupport
class WebConfiguration {}
```
```java
@Controller
@RequestMapping("/users")
class UserController {

  @RequestMapping("/{id}")
  String showUserForm(@PathVariable("id") User user, Model model) {
    // 따로 UserRepository 접근 필요 없음
    model.addAttribute("user", user);
    return "userForm";
  }
}
```
