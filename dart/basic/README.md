# 온라인 dart 편집기

https://dartpad.dartlang.org/

# 자료형

- int, double, String, bool, num
- num 은 int 와 double 을 포함
- var 로 타입추론 가능
- final, const 로 상수 정의
- final, const 와 var 는 같이 사용 불가능

# 문법

- is : 타입 체크, 부정은 !is
  - {} is var 는 불가능
- as : 변환

# 접근 제한자

- 언더바(_)를 변수 앞에 붙이면 private
- getter, setter 구현 필요

```dart
class Person {
  String name;
  int _age;
  
  int get age => _age;
  set age(int value) => _age = value;
}

void main() {
  var person = Person();
  print(person.age); // age = null
  person.age = 1;
  print(person.age); // age = 1
}
```

# Named Parameter

- 파라미터에 기본값이 필요할 때 쓸 수 있음

```dart
oid something(String name, {int age = 10}) {}

void main() {
  something('안홍범', age: 26); // age = 26
  something('안홍범'); // age = 10
  something(age: 26); // error 필수 매개변수를 입력하지 않았다.
  something(); // error 마찬가지
}
```
