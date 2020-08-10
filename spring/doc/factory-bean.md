# 용도

- 단순히 bean 의 scope 만이 아니라 복잡한 생성 로직이 필요할 때
- spring context 에 포함되어 bean lifecycle 기능을 활용 가능함(@PostConstruct 등)
- xml 을 통해 resource 를 관리할 때는 꽤 쓸만할듯
- AbstractFactoryBean 를 통해 좀 더 쉽게 만들 수 있음

```java
public interface FactoryBean {
    T getObject() throws Exception;
    Class<?> getObjectType();
    boolean isSingleton();
}
```

## 예시

- class Item 객체 생성 로직
- pojo 를 통한 설정

```java
public class ItemFactory implements FactoryBean<Item> {
 
    private int factoryId;
    private int itemId;
 
    @Override
    public Tool getObject() throws Exception {
        return new Item(itemId);
    }
 
    @Override
    public Class<?> getObjectType() {
        return Item.class;
    }
 
    @Override
    public boolean isSingleton() {
        return false;
    }
}
```

```java
@Configuration
public class FactoryBeanAppConfig {
 
    @Bean(name = "item")
    public ItemFactory getItemFactory() {
        ItemFactory factory = new ItemFactory();
        factory.setFactoryId(7070);
        factory.setItemId(2);
        return factory;
    }
 
    @Bean
    public Item getItem() throws Exception {
        return getItemFactory().getObject();
    }
}
```

## AbstractFactoryBean 

- 기본 설정이 Singleton 이라 new 로 생성해도 이후에는 Singleton
- scope 를 prototype 으로 하고 싶으면 setSingleton(false); 필요

```java
public class SingletonFactory extends AbstractFactoryBean {
    // itemId, factoryId...
    
    @Override
    protected Item createInstance() throws Exception {
        // singleton item
        return new Item(itemId);
    }
}
```

```java
public class PrototypeFactory extends AbstractFactoryBean {
    // itemId, factoryId...
    
    public PrototypeFactory() {
        setSingleton(false);
    }
    
    @Override
    protected Item createInstance() throws Exception {
        // new item
        return new Item(itemId);
    } 
}
```

