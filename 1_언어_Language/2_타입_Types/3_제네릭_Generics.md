# 제네릭 Generics

기본 배열 타입인 List에 대한 API 문서를 본 적이 있다면, 이것의 타입이 실제로는 List<E>라는 것을 알 수 있습니다. <...> 표기법은 List를 제네릭(혹은 매개변수화 된) 타입, 즉 형식 타입 매개변수를 가진 타입으로 표시합니다. 관례적으로, 대부분의 타입 변수들은 E, T, S, K, V와 같은 단일 문자 이름을 가집니다.

## 왜 제네릭을 사용하는가? Why use generics?

제네릭은 타입 안전성을 위해 요구되는 경우가 많지만, 제네릭은 단지 코드를 실행하는 것보다 더 많은 이점이 있습니다.

- 제네릭 타입을 적절하게 지정하면 더 나은 코드가 생성됩니다.
- 제네릭 타입을 사용하면 코드의 중복을 제거할 수 있습니다.

오직 문자열만 포함하도록 하는 리스트를 의도한다면 List<String>(문자열 리스트라고 읽으세요)로 선언할 수 있습니다. 이렇게 하면 동료 프로그래머와 도구가 리스트에 문자열이 아닌 것이 할당되었을 경우 이것이 실수임을 발견할 수 있습니다. 여기에 예시가 있습니다:

```dart
var names = <String>[];

names.addAll(['Seth', 'Kathy', 'Lars']);

names.add(42); // 오류
```

제네릭을 사용하는 도다른 이유는 코드의 중복을 제거하기 위해서 입니다. 제네릭을 사용하면 여러 타입 간에 단일 인터페이스와 구현을 공유하는 동시에, 정적 분석을 활용할 수 있습니다. 예를 들어, 객체를 캐싱하기 위한 인터페이스를 만든다고 가정해봅시다:

```dart
abstract class ObjectCache {
  Object getByKey(String key);
  
  void setByKey(String key, Object value);
}
```

이 인터페이스의 문자열 버전이 필요한 것을 발견했을 때, 또 다른 인터페이스를 만들 수 있습니다.

```dart
abstract class StringCache {
  String getByKey(String key);
  
  void setByKey(String key, String value);
}
```

나중에 당신은 이 인터페이스의 숫자 버전이 필요한 것을 발견했을 때... 아이디어를 얻었습니다.

제네릭 타입을 사용하면 이 모든 인터페이스를 만드는 대신에 타입 매개변수를 가지는 단일 인터페이스를 만들 수 있습니다:

```dart
abstract class Cache<T> {
  T getByKey(String key);
  
  void setByKey(String key, T value);
}
```

위의 코드에서 T는 대체가능한 타입입니다. 개발자가 나중에 정의할 타입으로 생각되는 임시 대체 타입입니다.

## 컬렉션 리터럴 사용 Using collection literals

List, Set, Map 리터럴은 매개변수화될 수 있습니다. 매개변수화된 리티럴은 여는 괄호 앞에 <type>(list나 set의 경우) 또는 <keyType, valueType>(map의 경우) 추가한다는 점을 제외하면 이미 본 리터럴과 같습니다. 여기에 타입 리터럴을 사용하는 예시가 있습니다.

```dart
var names = <String>['Seth', 'Kathy', 'Lars'];

var uniqueNames = <String>{'Seth', 'Kathy', 'Lars'};

var pages = <String, String>{
  'index.html': 'Homepage',
  'robots.txt': 'Hints for web robots',
  'humans.txt': 'We are people, not machines'
};
```

## 생성자와 함께 매개변수화된 타입 사용 Using parameterized types with constructors

생성자를 사용할 때 하나 이상의 타입을 지정하려면, 클래스 이름 바로 다음에 타입을 꺾쇠괄호(<...>)안에 넣으세요. 예를 들어:

```dart
var nameSet = Set<String>.from(names);
```

다음 코드는 정수 키와 View 타입의 값을 가지는 맵을 생성합니다.

```dart
var views = Map<int, View>();
```

## 제네릭 컬렉션과 여기에 포함된 타입 Generic collections and the types they contain

Dart의 제네릭 타입은 구체화 되었습니다. 즉 런타임에 타입 정보를 전달한다는 의미입니다. 예를 들어 컬렉션의 타입을 테스트할 수 있습니다:

```dart
var names = <String>[];

names.addAll(['Seth', 'Kathy', 'Lars']);

print(names is List<String>); // true
```

```
노트

대조적으로 자바의 제네릭은 erasure를 사용합니다. 즉 제네릭 타입 매개변수가 런타임에 제거된다는 것을 의미합니다. 자바에서 객체가 리스트인지 테스트할 수 있지만 객체가 List<String>인지 테스트할 수는 없습니다.
```

## 매개변수화된 타입 제한 Restricting the parameterized type

제네릭 타입을 구현할 때, 인자처럼 제공될 수 있는 타입을 제한하기를 원한다면 인자는 반드시 특정 타입의 하위 타입이어야 합니다. extends를 사용하여 이 작업을 수행할 수 있습니다.

일반적인 사용 예시는 타입을 Object(기본 값인 Object? 대신)의 하위 타입으로 만들어 타입이 null을 허용하지 않도록 보장하는 것입니다.

```dart
class Foo<T extends Object> {
  // Foo에 제공된 T 타입은 반드시 null을 허용하지 않아야 합니다.
}
```

Object 외에 다른 타입과 함께 extends를 사용할 수 있습니다. 여기에 SomeBaseClass를 확장하여 타입 T의 객체에서 SomeBaseClass의 멤버를 호출할 수 있는 예시가 있습니다.

```dart
class Foo<T extends SomeBaseClass> {
  // 여기에 구현...
  String toString() => "Instance of 'Foo<$T>'";
}

class Extender extends SomeBaseClass {...}
```

SomeBaseClass나 그 하위 타입을 제네릭 인자로 사용하는 것은 괜찮습니다.

```dart
var someBaseClassFoo = Foo<SomeBaseClass>();

var extenderFoo = Foo<Extender>();
```

제네릭 인자를 지정하지 않아도 물론 괜찮습니다.

```dart
var foo = Foo();

print(foo); // Instance of 'Foo<SomeBaseClass>'
```

SomeBaseClass 타입으로 지정하지 않으면 오류가 발생합니다.

```dart
var foo = Foo<Object>();
```

## 제네릭 메서드 사용 Using generic methods

메서드와 함수 또한 타입 인자를 허용합니다.

```dart
T first<T>(List<T> ts) {
  // 초기 작업와 오류 확인을 하세요, 그리고...
  T tmp = ts[0];
  
  // 추가적인 확인과 처리를 하세요...
  return tmp;
}
```

여기 first(<T>) 제네릭 타입 매개변수를 사용하면 여러 위치에서 타입 인자 T를 사용할 수 있습니다.

- 함수의 반환 타입 (T).
- 인자의 타입 (List<T>).
- 로컬 변수의 타입 (T tmp).