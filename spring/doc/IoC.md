# IoC

## 역사
[참고링크](http://picocontainer.com/inversion-of-control-history.html)

### Avalon
- IoC type 1
- 98년 Stefano Mazzocchi 가 Java Apache Server Framework 설계에 사용하면서 대중화.
- 2004 년도에 Avalon 은 종료.

#### DependencyPool Lookup
```java
    BeanFactory factory = getBeanFactory();
    MessageRenderer mr = (MessageRenderer) factory.getBean("render");
    // do something with instance mr
```
- 중앙 의존성 관리 레지스트리에 필요할 때 요청해서 꺼내 씀.
- 레지스트리는 보통 xml 을 통해 정의함.(예: META-INF/spring/app-context.xml)

#### Contextualized Dependency Lookup
```java
public class ContextualizedDependencyLookup implements ManagedComponent {
    private UserService userService; // 의존성 주입 target
 
    @Override 
    public void performLookup (Container container) {    // ManagedComponent interface에 정의돈 메소드 구현
        this.userService = container.getDependency("userService"); // was나 fwk 에서 제공하는 container 대상으로 Lookup 
        // do something with userService 
    }
}
```
- Framework 에서 제공하는 Container 에서 요청해서 꺼내 씀.
- 요청하기 위해서는 반드시 fw 에서 요구하는 인터페이스를 구현해야 함. 이를 통해 도메인 격리 가능.

### Setter Injection
- IoC type 2
- Martin Fowler 는 setter 보다는 init 이라는 용어로 의존성 주입을 표현하는것이 좋다고 제안.
- 일정한 접두어(init)를 통해 컨테이너가 별도의 manifest 없이 자동으로 구성요소들을 어셈블 가능.

### Constructor Injection
- IoC type 3

### Field Injection or Getter Injection when you need
- IoC type 4

### Timeline

![timeline](http://picocontainer.com/images/ioc-timeline.png)

## 스프링에서의 IoC
```java
public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory, MessageSource, ApplicationEventPublisher, ResourcePatternResolver {
	@Nullable
	String getId();

	String getApplicationName();

	String getDisplayName();

	long getStartupDate();

	@Nullable
	ApplicationContext getParent();

	AutowireCapableBeanFactory getAutowireCapableBeanFactory() throws IllegalStateException;
}
```
- 위 인터페이스의 구현체가 스프링의 IoC
- 계층 구조를 지니며 자식 -> 부모 순으로 의존 객체를 탐색(양쪽 모두에 존재하면 자식 우선)
- 자식 컨테이너의 객체가 부모 컨테이너의 객체를 의존 가능(루트는 당연히 하나로 온전해야 함)

### BeanDefinition
- Xml, Annotation, Pojo 로 정의 가능
