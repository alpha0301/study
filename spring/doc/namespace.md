## 용도
xml에 element를 정의할 때 중복 발생을 예방할 수 있다.

## 주의사항
- 기본 ns는 접두어 불필요
- 기본 ns 지정 시 하위 element는 모두 ns에 속하게 됨

## 사용법
```xml
<element_name xmlns:sub_ns="URL_REFERENCE">
  /* 기본 ns */
  <sub1>
    <prop1>data</prop1>
    <prop2>data</prop2>
    <prop3>data</prop3>
  </sub_name>
  /* ns 지정 */
  <sub2>
    <sub_ns:prop1></sub_ns:prop1>
    <sub_ns:prop2></sub_ns:prop2>
    <sub_ns:prop3></sub_ns:prop3>
  </sub2>
</{element name}>
```

### 예제
```xml
<?xml version="1.0" encoding="euc-kr" standalone="yes"?>
<school xmlns="http://www.hankook.ac.kr/student" 
  xmlns:prof="http://www.hankook.ac.kr/professor">
  <student>
    <name>kyu</name>
    <email>xml@test.com</email>
    <address>방이동</address>
  </student>
  <professor>
    <prof:name>tom</prof:name>
    <prof:email>xml1@test.com</prof:email>
    <prof:address>성내동</prof:address>
  </professor>
</school>
```

## annotation scan 방식
context ns를 사용해야한다.

### 예제
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:c="http://www.springframework.org/schema/c"
    xmlns:context="http://www.springframework.org/schema/context" /* context ns 사용 */
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
        http://www.springframework.org/schema/beans/spring-beans-4.3.xsd
        http://www.springframework.org/schema/context 
        http://www.springframework.org/schema/context/spring-context-4.3.xsd">
 
    <context:component-scan base-package="di.annotation" />  /* di.annotation 패키지 하위에서 component scan */
</beans>
```
