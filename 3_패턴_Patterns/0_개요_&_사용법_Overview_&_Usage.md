# 패턴 Patterns

```
버전 노트

패턴은 최소 3.0의 언어 버전을 요구합니다.
```

패턴은 문과 표현식 같은 Dart 언어의 문법적 범주입니다. 패턴은 실제 값과 일치할 수 있는 값 집합의 형태를 나타냅니다.

이 페이지는 설명합니다:

- 패턴이 하는 일.

- Dart 코드에서 패턴이 허용되는 위치.

- 패턴의 일반적인 사용 예시가 무엇인지.

패턴의 다양한 형태에 대해 알고 싶다면, 패턴 타입 페이지를 방문하세요.

## 패턴이 하는 일 What Patterns do

일반적으로 패턴은 패턴의 맥락과 모양에 따라 값을 매칭하거나 구조 분해 할당 혹은 둘 다 할 수 있습니다.

먼저 패턴 매칭은 주어진 값이 다음과 같은지 확인할 수 있습니다.

- 특정한 형태를 가졌는지.

- 특정 상수인지.

- 어떤 다른 것과 동일한지.

- 특정한 타입을 가졌는지.

그런 다음 패턴 구조 분해 할당은 해당 값을 구성 부분으로 나누는 편리한 선언적 구문을 제공합니다. 같은 패턴을 사용하면 프로세스의 일부 또는 전체에 변수를 바인딩할수도 있습니다.

## 매칭 Matching

패턴은 항상 값이 예상한 형식인지 테스트합니다. 즉, 값이 패턴과 일치하는지 확인합니다.

일치 항목을 구성하는 것은 패턴의 종류에 따라 다릅니다. 예를 들어, 값이 패턴의 상수와 일치하면 상수 패턴이 일치합니다.

```dart
switch (number) {
  // 1 == number 이면 상수 패턴이 일치합니다.
  case 1:
    print('one');
}
```

많은 패턴은 외부나 내부 패턴으로 불리는 하위 패턴을 이용합니다. 패턴은 재귀적으로 하위 패턴과 일치합니다. 예를 들어, 컬렉션 타입 패턴의 개별 필드 는 변수 패턴 또는 상수 패턴일 수 있습니다.

```dart
const a = 'a';
const b = 'b';

switch (obj) {
  // List 패턴 [a, b]는 obj가  두 필드가 있는 리스트인 경우 먼저 obj와 매칭합니다.
  // 그 다음 해당 필드가 상수 하위패턴 'a', 'b'와 일치하는 경우. 
  case [a, b]:
    print('$a, $b');
}
```

일치하는 값의 일부를 무시하려면 플레이스홀더같은 와일드카드 패턴을 사용할 수 있습니다. 리스트 패턴의 경우, 나머지 요소를 사용할 수 있습니다.

## 구조 분해 할당 Destructuring

객체와 패턴이 일치할 때 패턴은 객체의 데이터에 접근할 수 있고 부분적으로 추출할 수 있습니다. 즉 패턴은 객체를 구조 분해 할당합니다:

```dart
var numList = [1, 2, 3];
// 리스트 패턴 [a, b, c]는 numList로부터 세 요소로 구조 분해 할당합니다.
var [a, b, c,] = numList;
// ...그리고 새 변수에 할당합니다.
print(a+b+c);
```

구조 분해 할당 패턴에서 어떤 종류의 패턴은 중첩할 수 있습니다. 예를 들어, 아래의 case 패턴은 첫 요소가 'a'나 'b'인 두 요소 리스트를 매칭하거나 구조 분해 할당할 수 있습니다.

```dart
switch (list) {
  case ['a' || 'b', var c]:
    print(c);
}
```

## 장소 패턴이 나타날 수 있습니다. Places patterns can appear

Dart 언어에서 몇 장소에 패턴을 사용할 수 있습니다.

- 로컬 변수 선언과 할당

- for, for-in 반복문

- if-case와 switch-case

- 컬렉션 리터럴에서 제어문

이 섹션은 패턴을 사용한 매칭과 구조 분해 할당의 일반적인 사용 예시를 설명합니다.

## 변수 선언 Variable declaration

Dart가 로컬 변수 선언을 허용하는 어디에든 패턴 변수 선언을 사용할 수 있습니다. 패턴은 선언의 오른쪽에 있는 값과 매칭합니다. 한번 매칭되면 값을 구조 분해 할당하고 새 로컬 변수에 바인딩합니다.

```dart
// 새 변수 a, b, c를 선언합니다.
var (a, [b, c]) = ('str', [1, 2]);
```

패턴 변수 선언은 반드시 var 또는 final로 시작해야하며 그 뒤에 패턴이 뒤따라야 합니다.


## 변수 할당 Variable assignment

변수 할당 패턴은 할당의 왼쪽에 있습니다. 먼저 일치하는 객체를 구조 분해 할당합니다. 그런 다음 새 변수에 바인딩하는 대신 존재하는 변수에 값을 할당합니다.

세 번째 임시 변수 선언 없이 두 변수의 값을 바꾸기 위해 변수 할당 패턴을 사용하세요.

```dart
var (a, b) = ('left', 'right');
(b, a) = (a, b); // 바꾸기.
print('$a $b'); // "right left" 출력.
```

## switch 문과 표현식 Switch statements and expressions

모든 case 절에는 패턴이 포함되어 있습니다. 이는 if-case문과 마찬가지로 switch 문과 표현식에도 적용됩니다. case에서 어떤 종류의 패턴도 사용할 수 있습니다.

case 패턴은 거부될 수 있고 다음 중 하나의 제어 흐름을 허용합니다.

- switch에 들어가는 객체를 매칭하거나 구조 분해 할당.

- 객체가 일치하지 않으면 continue 실행.

case의 패턴이 구조 분해 할당한 값은 로컬 변수가 됩니다. 스코프는 오직 그 case의 바디 내에서만 적용됩니다.

```dart
switch (obj) {
  1 == obj면 매칭
  case 1:
    print('one');

  // obj의 값이 상수 값 'first'와 'last' 사이이면 매칭
  case >= first && <= last:
    print('in range');

  // obj가 두 필드가 있는 레코드이면 매칭하고
  // 필드를 'a'와 'b'에 할당합니다. 
  case (var a, var b):
    print('a = $a, b = $b');

  default:
}
```

논리적 or 패턴은 switch 표현식이나 문에서 여러 case를 공유하도록 하는 데 유용합니다.

```dart
var isPrimary = switch (color) {
  Color.red || Color.yellow || Color.blue => true,
  _ => false
};
```

switch 문은 논리적 or 패턴 사용없이 여러 case를 공유할 수 있습니다. 그러나 여전히 여러 case가 guard를 공유하는 데 유용합니다:

```dart
switch (shape) {
  case Square(size: var s) || Circle(size: var s) when s > 0:
    print('Non-empty symmetric shape');
}
```

guard 절은 조건이 거짓인 경우 switch를 종료하지 않고 임의의 조건을 case의 일부로 평가합니다(case 안의 if문을 사용하는 것처럼).

```dart
switch (pair) {
  case (int a, int b):
    if (a > b) print('First element greater');
  // 거짓이면 아무것도 출력하지 않고 switch를 종료합니다.
  case (int a, int b) when a > b:
    // 거짓이면 아무것도 출력하지 않지만 다음 case를 진행합니다.
    print('First element greater');
  case (int a, int b):
    print('First element not greater');
}
```

## for와 for-in 반복문 For and for-in loops

컬렉션 내부의 값을 순회하거나 구조 분해 할당하기 위해서 for와 for-in 내부에서 패턴을 사용할 수 있습니다.

아래의 예시는 <Map>.entries 호출이 반환하는 MapEntry 객체를 구조 분해 할당하여 for-in 내부에서 객체 구조 분해 할당을 사용합니다.

```dart
Map<String, int> hist = {
  'a': 23,
  'b': 100,
};

for (var MapEntry(key: key, value: count) in hist.entries) {
  print('$key occurred $count times');
}
```

객체 패턴은 hist.entries가 명명된 타입 MapEntry를 가지는지 확인하고 명명된 필드 하위패턴 key와 value로 재귀됩니다.

각 순회 내 MapEntry의 key getter와 value getter를 호출하고 결과를 로컬 변수 key와 count에 바인딩합니다.

getter 호출의 결과를 같은 이름의 변수에 바인딩하는 것은 일반적인 사용 예시입니다. 그래서 객체 패턴은 변수 하위패턴으로부터 getter 이름을 추론할 수 있습니다. 이는 key: key와 같은 불필요한 것을 :key:와 같은 변수 패턴으로 단순화하는 것을 허용합니다.

```dart
for (var MapEntry(:key, value: count) in hist.entries) {
  print('$key occurred $count times');
}
```

## 패턴의 사용 예시 Use cases for patterns

이전 섹션에서 어떻게 패턴이 다른 Dart 코드의 구조에 적용되는지 설명했습니다. 두 변수의 값을 바꾸거나 map의 키-값 쌍을 구조 분해 할당하는 것처럼 흥미로운 사용 사례를 예시를 통해 봤습니다. 이 섹션은 더 많은 사용 사례를 설명합니다.

- 언제, 왜 패턴을 사용하는 것을 원할지.

- 패턴이 해결하는 문제의 종류가 무엇인지.

- 어떤 관용구가 가장 어울리는지.

## 여러 반환을 구조 분해 할당하기

레코드를 사용하면 단일 함수 호출에서 여러 값을 집계하고 반환할 수 있습니다. 패턴은 함수 호출과 함께 레코드의 필드를 로컬 변수로 직접 분해하는 기능을 추가합니다.

다음과 같이 각 레코드 필드에 대해 새 로컬 변수를 개별적으로 선언하는 대신:

```dart
var info = userInfo(json);
var name = info.$1;
var age = info.$2;
```

변수 선언 또는 할당 패턴과 하위 패턴인 레코드 패턴을 사용하여 함수가 지역 변수로 반환하는 레코드 필드를 구조 분해 할당할 수 있습니다.

```dart
var (name, age) = userInfo(json);
```

패턴을 사용하여 명명된 필드가 있는 레코드를 구조 분해 할당하려면:

```dart
final (:name, :age) =
    getData(); // 예를 들어, return (name: 'doug', age: 25);
```

## 클래스 인스턴스 구조 분해 할당 Destructuring class instances

객체 패턴은 명명된 객체 타입을 매칭하므로 객체 클래스가 이미 노출한 getter를 사용하여 데이터를 구조 분해 할당할 수 있습니다.

클래스의 인스턴스를 구조 분해 할당하려면 명명된 타입을 사용하고 그 뒤에 괄호로 묶어 구조 해제할 속성을 사용합니다.

```dart
final Foo myFoo = Foo(one: 'one', two: 2);
var Foo(:one, :two) = myFoo;
print('one $one, two $two');
```

## 대수 타입 Algebraic data types

객체 구조 분해 및 switch case는 대수 타입 스타일로 코드를 작성하는 데 도움이 됩니다. 다음과 같은 경우에 이 방법을 사용하세요.

- 관련 타입의 유형이 있을 때.

- 각 타입에 대한 특정한 행동이 필요한 연산이 있을 때.

- 다양한 타입 정의 전체에 분산시키는 대신 한 곳에 그룹화하려고 할 때.

모든 타입에 대한 인스턴스 메서드로 연산을 구현하는 대신 하위 타입을 전환하는 단일 함수에서 연산의 변형을 유지하세요.

```dart
sealed class Shape {}

class Square implements Shape {
  final double length;
  Square(this.length);
}

class Circle implements Shape {
  final double radius;
  Circle(this.radius);
}

double calculateArea(Shape shape) => switch (shape) {
      Square(length: var l) => l * l,
      Circle(radius: var r) => math.pi * r * r
    };
```

## JSON 검증 Validating incoming JSON

맵과 리스트 패턴은 JSON 데이터의 키-값 쌍을 구조 분해 할당하는 데 적합합니다.

```dart
var json = {
  'user': ['Lily', 13]
};
var {'user': [name, age]} = json;
```

JSON 데이터가 예상한 구조를 가지고 있다는 것을 알고 있다면 이전 예시가 현실적입니다. 그러나 데이터는 일반적으로 네트워크 등 외부 소스에서 제공됩니다. 구조를 확인하려면 먼저 유효성을 검사해야 합니다.

패턴이 없으면 유효성 검사가 길어집니다.

```dart
if (json is Map<String, Object?> &&
    json.length == 1 &&
    json.containsKey('user')) {
  var user = json['user'];
  if (user is List<Object> &&
      user.length == 2 &&
      user[0] is String &&
      user[1] is int) {
    var name = user[0] as String;
    var age = user[1] as int;
    print('User $name is $age years old.');
  }
}
```

단일 case 패턴으로 동일한 유효성 검사를 수행할 수 있습니다. 단일 case는 if-case 문으로 가장 잘 작동합니다. 패턴은 보다 선언적이고 덜 장황한 JSON을 검증하는 방법을 제공합니다.

```dart
if (json case {'user': [String name, int age]}) {
  print('User $name is $age years old.');
}
```

이 case 패턴은 다음을 동시에 검증합니다:

- json이 map인지 확인합니다. 계속 진행하려면 먼저 외부 map 패턴과 일치해야 하기 때문입니다.
    - 그리고 맵이므로 json이 null이 아닌지도 확인합니다.

- json은 키 user를 포함하는지 확인합니다.

- 키 user가 두 값의 리스트와 쌍을 이루는지 확인합니다.

- 리스트 갑의 타입이 String과 int인지 확인합니다.

- 값을 보유하는 새 로컬 변수가 name과 age인지 확인합니다.