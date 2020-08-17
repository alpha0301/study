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
- JointPoint : Advice가 적용될 위치, 끼어들 수 있는 지점. 메서드 진입 지점, 생성자 호출 시점, 필드에서 값을 꺼내올 때 등 다양한 시점에 적용가능
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

