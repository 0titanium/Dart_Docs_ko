# 클래스 Classes

Dart는 클래스와 믹스인 기반의 상속을 갖춘 객체 지향 언어입니다. 모든 객체는 클래스의 인스턴스이며 Null을 제외한 모든 클래스는 Object의 자손입니다. 믹스인 기반 상속은 모든 클래스(최상위 클래스인 Object? 제외)가 정확히 하나의 상위 클래스를 가지더라도 클래스 내부를 여러 클래스 계층에서 재사용할 수 있음을 의미합니다. Extension 메서드는 클래스를 변경하거나 하위 클래스를 생성하지 않고 클래스에 기능을 추가하는 방법입니다. 클래스 수정자를 사용하면 라이브러리가 클래스의 하위 타입을 지정하는 방법을 제어할 수 있습니다.

## 클래스 멤버 사용하기 Using class members

객체에는 함수와 데이터(각각 메서드와 인스턴스 변수)로 구성된 멤버가 있습니다. 메서드를 호출하면 객체에서 호출됩니다. 메서드는 해당 객체의 함수와 데이터에 액세스할 수 있습니다.

인스턴스 변수나 메서드를 참조하려면 점(.)을 사용하세요.

```dart
var p = Point(2, 2);

// y 값 얻기.
assert(p.y == 2);

// p의 distanceTo() 호출하기.
double distance = p.distanceTo(Point(4, 4));
```

가장 왼쪽 피연산자가 null일 때 예외를 방지하려면 . 대신 ?.를 사용하세요.

```dart
// p가 null이 아닌 경우 y 값과 동일한 변수를 설정합니다.
var a = p?.y;
```

## 생성자 사용하기 Using constructors

생성자를 사용하여 객체를 생성할 수 있습니다. 생성자 이름은 ClassName 또는 ClassName.identifier일 수 있습니다. 예를 들어 다음 코드는 Point() 및 Point.fromJson() 생성자를 사용하여 Point 객체를 만듭니다:

```dart
var p1 = Point(2, 2);

var p2 = Point.fromJson({'x': 1, 'y': 2});
```

다음 코드는 동일한 효과를 갖지만 생성자 이름 앞에 선택적 new 키워드를 사용합니다:

```dart
var p1 = new Point(2, 2);
var p2 = new Point.fromJson({'x': 1, 'y': 2});
```

일부 클래스는 상수 생성자를 제공합니다. 상수 생성자를 사용하여 컴파일 타임 상수를 만들려면 생성자 이름 앞에 const 키워드를 넣으세요:

```dart
var p = const ImmutablePoint(2, 2);
```

두 개의 동일한 컴파일 타임 상수를 생성하면 단일 정식 인스턴스가 생성됩니다:

```dart
var a = const ImmutablePoint(1, 1);
var b = const ImmutablePoint(1, 1);

assert(identical(a, b)); // a와 b는 같은 인스턴스입니다!
```

상수 컨텍스트 내에서는 생성자나 리터럴 앞에 const를 생략할 수 있습니다. 예를 들어 const 맵을 생성하는 다음 코드를 살펴보세요:

```dart
// 많은 const 키워드.
const pointAndLine = const {
  'point': const [const ImmutablePoint(0, 0)],
  'line': const [const ImmutablePoint(1, 10), const ImmutablePoint(-2, 11)],
};
```

const 키워드의 첫 번째 사용을 제외하고 모두 생략할 수 있습니다:

```dart
// 상수 컨텍스트를 설정하는 단 하나의 const.
const pointAndLine = {
  'point': [ImmutablePoint(0, 0)],
  'line': [ImmutablePoint(1, 10), ImmutablePoint(-2, 11)],
};
```

상수 생성자가 상수 컨텍스트 외부에 있고 const 없이 호출되면 상수가 아닌 객체가 생성됩니다:

```dart
var a = const ImmutablePoint(1, 1); // 상수 생성
var b = ImmutablePoint(1, 1); // 상수를 생성하지 않습니다.

assert(!identical(a, b)); // 같은 인스턴스가 아닙니다!
```

## 객체 타입 얻기 Getting object's type

런타임에 객체의 타입을 가져오려면 Type 객체를 반환하는 객체 속성 RuntimeType을 사용할 수 있습니다.

```dart
print('The type of a is ${a.runtimeType}');
```

```
경고
객체의 타입을 테스트하려면 RuntimeType 대신 타입 테스트 연산자를 사용하세요.

프로덕션 환경에서 테스트 Object is Type은 Object.RuntimeType == Type보다 더 안정적입니다.
```

지금까지 클래스를 사용하는 방법을 살펴보았습니다. 이 섹션의 나머지 부분에서는 클래스를 구현하는 방법을 보여줍니다.

## 인스턴스 변수 Instance variables

인스턴스 변수를 선언하는 방법은 다음과 같습니다:

```dart
class Point {
  double? x; // 초기값이 null인 인스턴스 변수 x를 선언합니다.
  double? y; // 초기값이 null인 y를 선언합니다.
  double z = 0; // 초기값이 0인 z를 선언합니다.
}
```

null 허용 타입으로 선언된 초기화되지 않은 인스턴스 변수의 값은 null입니다. null을 허용하지 않는 인스턴스 변수는 선언 시 초기화되어야 합니다.

모든 인스턴스 변수는 암시적 getter 메서드를 생성합니다. final이 아닌 인스턴스 변수와 이니셜라이저가 없는 late final 인스턴스 변수도 암시적 setter 메서드를 생성합니다. 자세한 내용은 Getter 및 Setter를 확인하세요.

```dart
class Point {
  double? x; // 초기값이 null인 인스턴스 변수 x를 선언합니다.
  double? y; // 초기값이 null인 y를 선언합니다.
}

void main() {
  var point = Point();
  point.x = 4; // x에 setter 메서드 사용.
  assert(point.x == 4); // x에 getter 메서드 사용.
  assert(point.y == null); // 기본값은 null.
}
```

선언된 위치에서 late가 아닌 인스턴스 변수를 초기화하면, 생성자와 해당 초기화 리스트가 실행되기 전, 인스턴스가 생성될 때 값이 설정됩니다. 결과적으로, late가 아닌 인스턴스 변수의 초기화 표현식(= 뒤)은 this에 액세스할 수 없습니다.

```dart
double initialX = 1.5;

class Point {
  // 좋습니다, `this`에 의존하지 않는 선언에 접근할 수 있습니다:
  double? x = initialX;

  // 오류, 'late' 초기화가 아닌 경우 'this'에 액세스할 수 없습니다:
  double? y = this.x;

  // 좋습니다, `late` 초기화에서 `this`에 접근할 수 있습니다:
  late double? z = this.x;

  // 좋습니다, `this.x`와 `this.y`는 표현식이 아니라 매개변수 선언입니다:
  Point(this.x, this.y);
}
```
인스턴스 변수는 final일 수 있으며, 이 경우 정확히 한 번만 설정해야 합니다. 생성자 매개변수를 사용하거나 생성자의 초기화 리스트를 사용하여 final과 late가 아닌 인스턴스 변수를 선언 시에 초기화하세요.

```dart
class ProfileMark {
  final String name;
  final DateTime start = DateTime.now();

  ProfileMark(this.name);
  ProfileMark.unnamed() : name = '';
}
```

생성자 바디 안에 final 인스턴스 변수의 값을 할당해야 하는 경우 다음 중 하나를 사용할 수 있습니다.

- 팩토리 생성자를 사용하세요.
- late final을 사용하세요. 그러나 주의하세요: 초기화가 없는 late final은 API에 setter를 추가합니다.

## 암시적 인터페이스 Implicit interfaces

모든 클래스는 클래스와 클래스가 구현하는 모든 인터페이스의 모든 인스턴스 멤버를 포함하는 인터페이스를 암시적으로 정의합니다. B의 구현을 상속하지 않고 B 클래스의 API를 지원하는 A 클래스를 생성하려면 A 클래스가 B 인터페이스를 구현해야 합니다.

클래스는 implements 절에 인터페이스를 선언한 다음, 인터페이스가 요구하는 API를 제공함으로써 하나 이상의 인터페이스를 구현합니다. 예를 들어:

```dart
// 사람. 암시적 인터페이스에는 greet()이 포함되어 있습니다.
class Person {
  // 인터페이스에 있지만 이 라이브러리에만 표시됩니다.
  final String _name;

  // 생성자이므로 인터페이스에는 없습니다.
  Person(this._name);

  // 인터페이스에서.
  String greet(String who) => 'Hello, $who. I am $_name.';
}

// Person 인터페이스의 구현입니다.
class Impostor implements Person {
  String get _name => '';

  String greet(String who) => 'Hi $who. Do you know who I am?';
}

String greetBob(Person person) => person.greet('Bob');

void main() {
  print(greetBob(Person('Kathy')));
  print(greetBob(Impostor()));
}
```

다음은 클래스가 여러 인터페이스를 구현하도록 지정하는 예입니다.

```dart
class Point implements Comparable, Location {...}
```

## 클래스 변수와 메서드 Class variables and methods

Use the static keyword to implement class-wide variables and methods.
클래스 전체의 변수와 메소드를 구현하려면 static 키워드를 사용하십시오.

## static 변수 Static variables

클래스 정적 변수(클래스 변수)를 구현하려면 static 키워드를 사용하십시오. 클래스 전체 상태 및 상수에 유용합니다.

```dart
class Queue {
  static const initialCapacity = 16;
  // ···
}

void main() {
  assert(Queue.initialCapacity == 16);
}
```

static 변수는 사용될 때까지 초기화되지 않습니다.

```
노트

이 페이지는 상수 이름에 lowerCamelCase를 선호하는 스타일 가이드 권장 사항을 따릅니다.
```

## static 메서드 Static methods

static 메서드(클래스 메서드)는 인스턴스에서 작동하지 않으므로 this에 액세스할 수 없습니다. 그러나 static 변수에는 접근할 수 있습니다. 다음 예제에서 볼 수 있듯이 클래스에서 직접 static 메서드를 호출합니다.

```dart
import 'dart:math';

class Point {
  double x, y;
  Point(this.x, this.y);

  static double distanceBetween(Point a, Point b) {
    var dx = a.x - b.x;
    var dy = a.y - b.y;
    return sqrt(dx * dx + dy * dy);
  }
}

void main() {
  var a = Point(2, 2);
  var b = Point(4, 4);
  var distance = Point.distanceBetween(a, b);
  assert(2.8 < distance && distance < 2.9);
  print(distance);
}
```

```
노트

일반적이거나 널리 사용되는 유틸리티 및 기능에는 static 메서드 대신 최상위 함수를 사용하는 것이 좋습니다.
```

static 메서드를 컴파일 타임 상수로 사용할 수 있습니다. 예를 들어 static 메서드를 상수 생성자에 매개 변수로 전달할 수 있습니다.