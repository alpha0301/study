
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

## Spring Data JPA

### feature

- Querydsl 및 type-safe JPA 쿼리
- 페이징, 동적 쿼리
- 
