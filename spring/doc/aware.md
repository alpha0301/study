# 용도

- ApplicationContext, BeanFactory 및 ResourceLoader 와 같은 프레임워크 객체에 접근해야 할 때
- IoC 스타일을 깨는 것이므로 좋지 않음
- 리소스 접근 시에는 의미있는 사용이 될 수 있음

# lifecycle
- bean lifecycle 중에서 의존성 주입 완료와 pre initialization 단계 사이에서 호출

![lifecycle](https://springframework.guru/wp-content/uploads/2019/05/aware_interfaces_callbacks_in_bean_lifecycle.png)

# 사용법

- interface Aware 를 구현하면 접근 가능
```java
public interface ApplicationContextAware {
    void setApplicationContext(ApplicationContext applicationContext) throw BeansException;
}
```

```java
public class ApplicationContextAwareImpl implements ApplicationContextAware {
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        User user = (User) applicationContext.getBean("user");
        System.out.println("User Id: " + user.getUserId() + " User Name :" + user.getName());
    }
}
```
