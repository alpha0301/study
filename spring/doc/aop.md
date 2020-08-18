# 개념

- 전체 절차를 핵심 관점 / 부가 관점으로 분리하여 관리
- 핵심 관점: 비즈니스 로직
- 부가 관점: IO 요청(DB, 로깅, 파일 입출력 등등)
- 모듈화: 공통으로 쓰이는 관점을 분리하고 재사용

# 스프링 AOP 특징
- 프록시 패턴 기반의 AOP 구현체
- 스프링 빈에만 AOP를 적용 가능
- 모든 AOP 기능을 제공하는 것이 아닌 필요한 개념과 기능만 선별되어 있음
- 기존의 interception 기능만 제공하는 기술과의 차이점
```
The concept of join points matched by pointcuts is the key to AOP, 
 which distinguishes it from older technologies offering only interception. 
Pointcuts enable advice to be targeted independently of the object-oriented hierarchy. 
For example, you can apply an around advice providing declarative transaction management 
 to a set of methods that span multiple objects 
(such as all business operations in the service layer).
```

# 스프링 AOP 주요 기능

- Aspect : 흩어진 관심사를 모듈화 한 것.
- Target object : Aspect를 적용하는 곳. (클래스, 메서드 .. ) Adviced object 라고도 함. 항상 프록시 객체임.(Spring AOP는 런타임 프록시를 사용하여 구현되므로)
- Advice : 실질적으로 어떤 일을 해야할 지에 대한 것, 실질적인 부가기능을 담은 구현체
- JointPoint : Target에 전달될 매개변수, method 정보를 접근할 수 있는 api 제공. Advice method 매개변수로 선언. Around Advice는 ProceedingJoinPoint type 을 사용해야 함
- PointCut : JointPoint의 상세한 스펙을 정의한 것. 'A란 메서드의 진입 시점에 호출할 것'과 같이 더욱 구체적으로 Adviced object(Target object)가 실행될 지점을 정할 수 있음
- Intruduction: type 대신 메소드나 필드를 선언하는것. Advise 에 특정 interface를 구현하게 할 수 있음. 예를 들어, 캐싱을 위해 IsModified interface 를 구현하게 할 수 있음.
- AOP proxy: 프록시 객체. JDK dynamic proxy 또는 CGLIB proxy
- Weaving: Aspect 지정 객체에 대한 프록시 객체를 생성하는 과정. Spring AOP 에서는 Runtime weaving 방식을 제공함. AspectJ 를 사용하면 Load-time weaving, Compile-time weaving 사용이 가능함.
  - Load-time weaving: appContext 에 객체를 로드한 후 weaving. app init 속도가 좀 느림.
  - Compile-time weaving: 컴파일하면서 weaving 해서 class 를 미리 생성. 성능 빠름. Lombok처럼 다른 Compile-time plugin 과 같이 쓸 때 충돌할 수 있어서 주의해야 함.
- 대부분의 요구사항은 Spring AOP 기능으로 해결될 것으로 보이지만 더 강력한 기능이 필요하면 AspectJ 와 연동해서 사용하면 좋음.(아래 참고 글)
```
Spring AOP never strives to compete with AspectJ to provide a comprehensive AOP solution. 
We believe that both proxy-based frameworks such as Spring AOP and full-blown frameworks
 such as AspectJ are valuable and that they are complementary, rather than in competition. 
Spring seamlessly integrates Spring AOP and IoC with AspectJ, to enable all uses of AOP
 within a consistent Spring-based application architecture. 
This integration does not affect the Spring AOP API or the AOP Alliance API. 
Spring AOP remains backward-compatible. 
```

## AspectJ 사용 시

- AspectJ compiler 필요
- method level weaving 이외에도 field, constructor, static initializer, final class/method, etc... weaving 가능
- Spring IoC Bean 외의 모든 Domain 에 구현 가능
- 모든 pointcuts 지원
- 이해하고 쓰기 좀 어려움

## Spring AOP vs Full AspectJ?

- Spring bean 에만 적용할 것이냐 그 외의 object 에도 적용할 필요가 있느냐로 결정
- 물론 method 외에 적용하고 싶으면 써야 함

## @AspectJ vs XML 설정

- test를 위해 XML 설정이 좀 더 나을 때가 있음
- @AspectJ 가 좀 더 기능 지원 범위가 넓음(pointcut 합쳐서 쓰기 와 같이)
- 두 개의 설정방법을 동시에 사용 가능

## Advice types

- Before advice: join point 실행 전 수행. exception 이 없으면 무조건 join point 로 이행됨
- After returning advice: method 가 exception 없이 실행이 끝나면 수행
- After throwing advice: method 에서 exception 이 발생하면 수행
- After (finally) advice: method 에서 에러가 발생해도 반드시 수행
- Around advice: method 실행 전 후에 자유롭게 동작을 수행하도록 할 수 있음. 왠만하면 다른 type을 사용하는것이 오류 발생 가능성을 낮출 수 있음

## PointCut

- union, intersection 등의 연산을 지원함
- union: the methods that either pointcut matches(일치하는 하나의 메소드)
- intersection: the methods that both pointcuts match(일치하는 모든 메소드)
- 일반적으로 union 이 더 유용함
- 스프링 내의 static 메소드로 비교하거나 ComposablePointcut 을 활용할 수 있지만 AspectJ Pointcut expressions 을 사용하는것이 더 나음
- AspectJ PointCut: org.springframework.aop.aspectj.AspectJExpressionPointcut

```java
public interface Pointcut {

    ClassFilter getClassFilter();

    MethodMatcher getMethodMatcher();

}

public interface ClassFilter {

    boolean matches(Class clazz);
}

public interface MethodMatcher {

    boolean matches(Method m, Class targetClass);

    boolean isRuntime();

    // isRuntime() == true 일 때만 호출됨
    // 대부분의 MethodMatcher 구현체는 static이라 isRuntime() == false -> 잘 사용되지 않음
    boolean matches(Method m, Class targetClass, Object[] args);
}
```

### Control Flow Pointcut

- Dynamic Pointcut 종류 중 하나
- call stack 을 확인해서 특정
- 특정 패키지 또는 class 에서  method 에서 호출한 경우
- 비용이 비싸니까 사용할 때 주의

### Pointcut 사용법

- expression: AspectJ5 pointcut expression 을 사용함
- JDK proxy 를 사용하면 public method 만 intercept 를 허용
- desinator
  - bean : 특정 이름의 bean 또는 bean 집단으로 제한. Spring AOP 에서만 사용 가능
  - execution : Advice 를 적용할 method 를 명시
    - execution([접근제한자] 리턴타입 [class이름].method이름(파라미터)
    - 접근제한자는 생략 가능
    - execution(public Integer com.estsoft.asm4.*.*(*)) : com.estsoft.asm4 패키지의 public Integer 에 parameter 가 1개인 모든 method
    - execution(* com.estsoft.asm4..*.get*(..)) : com.estsoft.asm4 패키지 및 하위 패키지의 get 으로 시작하는 모든 method
  - within : 특정 type 내의 method 를 JoinPoint 로 설정되도록 명시
    - within(com.estsoft.asm4.Service) : Service interface 구현체
    - within(com.estsoft.asm4.common.*) : common 패키지의 모든 method
    - within(com.estsoft.asm4..*)) : asm4 패키지 및 하위 패키지의 모든 method
  - this : 매칭할 조인포인트를 프록시가 전달한 타입의 인스턴스인 경우로 제한
  - target : 매칭할 조인포인트를 원본이 전달한 타입의 인스턴스 인 경우로 제한
    - target(com.estsoft.asm4.Service) : Service interface 구현체
  - args : 매칭할 조인포인트를 매개변수가 전달한 타입의 인스턴스인 경우로 제한
    - args(java.io.Serializable) : 하나의 매개변수, 런타임에 전달된 매개변수 타입이 Serializable 인 method
  - @target : 매칭할 조인포인트를 실행하는 객체의 클래스가 전달한 타입의 어노테이션이 붙어있는 경우로 제한
    - @target(org.springframework.transaction.annotation.Transactional) : @Transactional 이 선언된 class의 모든 method
  - @args : 매칭할 조인포인트를 전달된 실제 매개변수의 런타임 타입이 전달한 타입의 어노테이션이 붙어있는 경우로 제한
    - @args(com.xyz.security.Classified) : @Classified 이 선언된 매개변수를 가진 method
  - @within : 매칭할 조인포인트를 전달한 어노테이션이 붙은 타입으로 제한
  - @annotation : 매칭할 조인포인트를 조인포인트의 주체가 전달한 어노테이션이 붙어있는 경우로 제한
    - @annotation(org.springframework.transaction.annotation.Transactional) : @Transactional 이 선언된 method

```java
@Pointcut("execution(* transfer(..))") // the pointcut expression

private void anyOldTransfer() {} // the pointcut signature

```

#### Pointcut 결합

```java
@Pointcut("execution(public * *(..))")
private void anyPublicOperation() {}
    
@Pointcut("within(com.xyz.someapp.trading..*)")
private void inTrading() {}
    
@Pointcut("anyPublicOperation() && inTrading()")
private void tradingOperation() {}
```

# @AspectJ support

- AspectJ5 부터 도입된 어노테이션
- AspectJ 종속성 없이 순수하게 Spring AOP 만 사용해도 적용 가능

## 적용방법

- @EnableAspectAutoProxy 추가 필요
- xml: <aop:aspectj-autoproxy/>

```java
@Configuration
@EnableAspectJAutoProxy
public class AppConfig {

}

//

import org.aspectj.lang.annotation.Aspect;

@Component
@Aspect
public class TestAspect {
 // pointcut, advice, introduction(inner-type) 선언 가능
}
```

## @Aspect

- aspect 객체는 다른 aspect의 타겟이 될 수 없음
- @Aspect 를 체크하여 auto proxying 에서 제외됨

```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

@Aspect
public class BeforeExample {

  @Before("com.xyz.myapp.SystemArchitecture.dataAccessOperation()")
  public void doAccessCheck() {
    // ...
  }
}
```

```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

@Aspect
public class BeforeExample {

  @Before("execution(* com.xyz.myapp.dao.*.*(..))")
  public void doAccessCheck() {
    // ...
  }
}
```

```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.AfterReturning;

@Aspect
public class AfterReturningExample {

  @AfterReturning(
    pointcut="com.xyz.myapp.SystemArchitecture.dataAccessOperation()",
    returning="retVal")
  public void doAccessCheck(Object retVal) {
    // ...
  } 
}
```

```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.AfterThrowing;

@Aspect
public class AfterThrowingExample {

  @AfterThrowing(
    pointcut="com.xyz.myapp.SystemArchitecture.dataAccessOperation()",
    throwing="ex")
  public void doRecoveryActions(DataAccessException ex) {
    // ...
  }
}
```

```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.ProceedingJoinPoint;

@Aspect
public class AroundExample {

  @Around("com.xyz.myapp.SystemArchitecture.businessService()")
  public Object doBasicProfiling(ProceedingJoinPoint pjp) throws Throwable {
    long start = System.currentTimeMillis();
    Object retVal = pjp.proceed();
    System.out.println(System.currentTimeMillis() - start);
    return retVal;
  }
}
```

- Spring 은 AOP Alliance interface 를 준수함 

```java
public class DebugInterceptor implements MethodInterceptor {

    public Object invoke(MethodInvocation invocation) throws Throwable {
        System.out.println("Before: invocation=[" + invocation + "]");
        Object rval = invocation.proceed();
        System.out.println("Invocation returned");
        return rval;
    }
}
```

### Intruductions

```java
@Aspect
public class UsageTracking {

    @DeclareParents(value="com.xzy.myapp.service.*+", defaultImpl=DefaultUsageTracked.class)
    public static UsageTracked mixin;

    @Before("com.xyz.myapp.CommonPointcuts.businessService() && this(usageTracked)")
    public void recordUsage(UsageTracked usageTracked) {
        usageTracked.incrementUseCount();
    }

}
```

## Proxying Mechanisms

- 대상 object가 하나의 interface 라도 구현했다면 JDK dynamic proxy 를 사용함(모든 interface 가 프록시화)
- 아무런 interface 를 구현하지 않으면 CGLIB 프록시를 생성함

[참고 문서 링크](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop)
