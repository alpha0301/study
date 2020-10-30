# Collection

```dart
List<String> list = ['고양이', '강아지', '호랑이'];
var items = ['고양이', '강아지', '호랑이']; // 위와 동일함.

Map<String, String> personMap = { 
  'hongbeom' : 'hi',
  'gildong' : 'hello'
};
var personMap = { 
  'hongbeom' : 'hi',
  'gildong' : 'hello'
}; // 위와 동일함.
// usage
print(personMap['hongbeom']); // hi

Set<String> personSet = {'hongbeom', 'gildong'};
var personSet = {'hongbeom', 'gildong'};
print(personSet.contains('hongbeom'); // true

var mySet = <String>{}; // Set<String>
var mySet2 = {}; // Map<dynamic, dynamic>으로 취급
```

# 스프레드 연산자

```dart

var items = [1, 2, 3, 4, 5];
var items2 = [0, ...items, 6]; // [0, 1, 2, 3, 4, 5, 6]
```

# where

```dart
final items = [1, 2, 3, 4, 5];
items.where((e) => e % 2 == 0).forEach(print); // 2, 4
```

# if

```dart
bool isInt = false;
print([1, 2, 3, 4, if (isInt) 5]); // [1, 2, 3, 4]
```
