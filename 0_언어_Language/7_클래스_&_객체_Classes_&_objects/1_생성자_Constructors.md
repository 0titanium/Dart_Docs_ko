# 생성자 Constructors

생성자는 클래스의 인스턴스를 생성하는 특별한 함수입니다.

Dart는 다양한 타입의 생성자를 구현합니다. 기본 생성자를 제외하고 이 함수는 클래스와 같은 이름을 사용합니다.

- 생성형 생성자 : 새 인스턴스를 생성하고 인스턴스 변수를 초기화합니다.

- 기본 생성자 : 생성자가 지정되지 않은 경우 새 인스턴스를 만드는 데 사용됩니다. 인자를 사용하지 않으며 이름도 지정되지 않습니다.

- 명명된 생성자 : 생성자의 목적을 명확하게 하거나 동일한 클래스에 대해 여러 생성자를 생성할 수 있도록 허용합니다.

- 상수 생성자 : 인스턴스를 컴파일 타입 상수로 생성합니다.

- 팩토리 생성자 : 하위 타입의 새 인스턴스를 생성하거나 캐시에서 기존 인스턴스를 반환합니다.

- 생성자 리디렉션 : 동일한 클래스의 다른 생성자로 호출을 전달합니다.

## 생성자의 타입 Types of constructors

## 생성형 생성자 Generative constructors

클래스를 인스턴스화 하려면 생성형 생성자를 사용하세요.

```dart
class Point {
  // 변수와 값의 초기화 리스트
  double x = 2.0;
  double y = 2.0;

  // 형식 매개변수를 초기화하는 생성형 생성자:
  Point(this.x, this.y);
}
```

## 기본 생성자 Default constructors

생성자를 선언하지 않으면 Dart는 기본 생성자를 사용합니다. 기본 생성자는 인자나 이름이 없는 생성형 생성자입니다.

## 명명된 생성자 Named constructors

명명된 생성자를 사용하여 클래스에 대해 여러 생성자를 구현하거나 추가 명확성을 제공합니다:

```dart
const double xOrigin = 0;
const double yOrigin = 0;

class Point {
  final double x;
  final double y;

  // 생성자 내부가 실행되기 전에
  // 인스턴스 변수 x와 y를 설정합니다.
  Point(this.x, this.y);

  // Named constructor
  Point.origin()
      : x = xOrigin,
        y = yOrigin;
}
```

하위 클래스는 상위 클래스의 명명된 생성자를 상속하지 않습니다. 상위 클래스에 정의된 명명된 생성자를 사용하여 하위 클래스를 만들려면 하위 클래스에서 해당 생성자를 구현합니다.

## 상수 생성자 Constant constructors

클래스가 변경되지 않는 객체를 생성하는 경우 이러한 객체를 컴파일 타임 상수로 만드세요. 객체를 컴파일 타임 상수로 만들려면 const모든 인스턴스 변수가 final로 설정된 생성자를 정의하세요.

```dart
class ImmutablePoint {
  static const ImmutablePoint origin = ImmutablePoint(0, 0);

  final double x, y;

  const ImmutablePoint(this.x, this.y);
}
```

상수 생성자가 항상 상수를 생성하는 것은 아닙니다. const가 아닌 컨텍스트에서 호출될 수도 있습니다. 자세한 내용은 생성자 사용 섹션을 참조하세요.

## 생성자 리디렉션 Redirecting constructors

생성자는 동일한 클래스의 다른 생성자로 리디렉션될 수 있습니다. 리디렉션 생성자에는 본문이 비어 있습니다. 생성자는 콜론(:) 뒤에 클래스 이름 대신 this를 사용합니다.

```dart
class Point {
  double x, y;

  // 이 클래스의 메인 생성자.
  Point(this.x, this.y);

  // 메인 생성자에게 위임.
  Point.alongXAxis(double x) : this(x, 0);
}
```

## 팩토리 생성자 Factory constructors

생성자를 구현하는 경우 다음 두 가지 경우 중 하나가 발생하면 factory 키워드를 사용하세요.

- 생성자가 항상 해당 클래스의 새 인스턴스를 생성하는 것은 아닙니다. 팩토리 생성자는 null을 반환할 수 없지만 다음을 반환할 수 있습니다.
  - 새 인스턴스를 반환하는 대신에 캐시에서 존재하는 인스턴스.
  - 하위 타입의 새 인스턴스.

- 인스턴스를 생성하기 전에 중요한 작업을 수행해야 합니다. 여기에는 인자 확인이나 초기화 리스트에서 처리할 수 없는 다른 처리 수행이 포함될 수 있습니다.

```
팁

late final을 사용하여 final 변수의 지연 초기화를 처리할 수도 있습니다 (신중하게!).
```

다음 예시에는 두 팩토리 생성자가 포함되어 있습니다.

- Logger 팩토리 생성자는 캐시에서 객체를 반환합니다

- Logger.fromJson팩토리 생성자는 JSON 객체에서 final 변수를 초기화합니다.

```dart
class Logger {
  final String name;
  bool mute = false;

  // _cache는 이름 앞의 _ 덕분에
  // 라이브러리 private입니다.
  static final Map<String, Logger> _cache = <String, Logger>{};

  factory Logger(String name) {
    return _cache.putIfAbsent(name, () => Logger._internal(name));
  }

  factory Logger.fromJson(Map<String, Object> json) {
    return Logger(json['name'].toString());
  }

  Logger._internal(this.name);

  void log(String msg) {
    if (!mute) print(msg);
  }
}
```

```
경고

팩토리 생성자는 this에 접근할 수 없습니다.
```

다른 생성자와 마찬가지로 팩토리 생성자를 사용합니다.

```dart
var logger = Logger('UI');
logger.log('Button clicked');

var logMap = {'name': 'UI'};
var loggerJson = Logger.fromJson(logMap);
```

## 팩토리 생성자 리디렉션 Redirecting factory constructors

팩토리 생성자 리디렉션은 생성자 리디렉션을 호출할 때마다 사용할 다른 클래스의 생성자에 대한 호출을 지정합니다.

```dart
factory Listenable.merge(List<Listenable> listenables) = _MergingListenable
```

일반 팩토리 생성자는 다른 클래스의 인스턴스를 생성하고 반환할 수 있는 것처럼 보일 수 있습니다. 이렇게 하면 팩토리 리디렉션이 불필요해집니다. 팩토리 리디렉션에는 다음과 같은 몇 가지 장점이 있습니다.

- 추상 클래스는 다른 클래스의 상수 생성자를 사용하는 상수 생성자를 제공할 수 있습니다.

- 리디렉션 팩토리 생성자를 사용하면 전달자가 매개 변수와 해당 기본값을 반복할 필요가 없습니다.

## 생성자 분리 Constructor tear-offs

Dart를 사용하면 생성자를 호출하지 않고도 생성자를 매개변수로 제공할 수 있습니다. 분리(괄호를 분리하는 것과 같은)는 동일한 매개변수를 사용하여 생성자를 호출하는 클로저 역할을 합니다.

분리가 메서드가 허용하는 것과 동일한 시그니처 및 반환 타입을 가진 생성자인 경우 분리를 매개변수 또는 변수로 사용할 수 있습니다.

분리는 람다나 익명 함수와 다릅니다. 람다는 생성자의 래퍼 역할을 하는 반면, 분리는 생성자입니다.

## 분리 사용 Use Tear-Offs

```dart
// good
// 명명된 생성자에 대한 분리: 
var strings = charCodes.map(String.fromCharCode);  

// 명명되지 않은 생성자에 대한 분리: 
var buffers = charCodes.map(StringBuffer.new);
```

## 람다 아님 Not Lambda

```dart
// bad
// 명명된 생성자에 대한 람다 대신:
var strings = charCodes.map((code) => String.fromCharCode(code));

// 명명되지 않은 생성자에 대한 람다 대신:
var buffers = charCodes.map((code) => StringBuffer(code));
```

시각적 학습자의 경우 분리에 대한 Decoding Flutter 비디오를 시청하세요.

## 인스턴스 변수 초기화 Instance Variable Initialization

Dart는 세 가지 방법으로 변수를 초기화할 수 있습니다.

## 선언에서 인스턴스 변수 초기화 Initialize instance variables in the declaration

변수를 선언할 때 인스턴스 변수를 초기화합니다.

```dart
class PointA {
  double x = 1.0;
  double y = 2.0;

  // 암시적 기본 생성자는 이러한 변수들을 (1.0,2.0)으로 설정합니다.
  // PointA();

  @override
  String toString() {
    return 'PointA($x,$y)';
  }
}
```

## 형식 매개변수 초기화 사용 Use initializing formal parameters

생성자 인자를 인스턴스 변수에 할당하는 일반적인 패턴을 단순화하기 위해 Dart에는 형식 매개변수 초기화가 있습니다.

생성자 선언에서 this.<propertyName>를 포함하거나 바디를 생략합니다. this 키워드는 현재 인스턴스를 참조합니다.

When the name conflict exists, use this. Otherwise, Dart style omits the this. An exception exists for the generative constructor where you must prefix the initializing formal parameter name with this.

이름 충돌이 있는 경우 this를 사용하세요. 그렇지 않으면, Dart 스타일에서는 this를 생략합니다. 초기화 형식 매개변수 이름 앞에 this를 붙여야 하는 생성형 생성자에는 예외가 있습니다.

이 가이드의 앞부분에서 설명한 것처럼 특정 생성자와 생성자의 특정 부분은 this에 액세스할 수 없습니다. 여기에는 다음이 포함됩니다.

- 팩토리 생성자

- 초기화 리스트의 오른쪽

- 상위 클래스 생성자에 대한 인자

형식 매개변수 초기화는 null 비허용 인스턴스나 final 인스턴스 변수를 초기화할 수 있습니다. 이러한 타입의 변수에는 모두 초기화 또는 기본값이 필요합니다.

```dart
class PointB {
  final double x;
  final double y;

  // x와 y 인스턴스 변수 설정
  // 생성자 바디가 실행되기 전에.
  PointB(this.x, this.y);

  // 형식 매개변수 초기화는 선택적 매개변수가 될 수 있습니다.
  PointB.optional([this.x = 0.0, this.y = 0.0]);
}
```

private 필드는 명명된 형식 초기화로 사용할 수 없습니다.

```dart
class PointB {
// ...

  PointB.namedPrivate({required double x, required double y})
      : _x = x,
        _y = y;

// ...
}
```

이는 명명된 변수에도 적용됩니다.

```dart
class PointC {
  double x; // 반드시 생성자 안에서 설정
  double y; // 반드시 생성자 안에서 설정

  // 형식 매개변수 초기화를 사용한 생성형 생성자
  // with default values
  PointC.named({this.x = 1.0, this.y = 1.0});

  @override
  String toString() {
    return 'PointC.named($x,$y)';
  }
}

// 명명된 변수를 사용한 생성자.
final pointC = PointC.named(x: 2.0, y: 2.0);
```

형식 매개변수 초기화에서 도입된 모든 변수는 final 변수이며 초기화된 변수의 범위에만 있습니다.

초기화 리스트에서 표현할 수 없는 논리를 수행하려면 해당 논리를 사용하여 팩토리 생성자 또는 static 메서드를 만듭니다. 그런 다음 계산된 값을 일반 생성자에 전달할 수 있습니다.

생성자 매개변수는 null 가능으로 설정되어 초기화되지 않을 수 있습니다.

```dart
class PointD {
  double? x; // 생성자에 설정되지 않으면 null
  double? y; // 생성자에 설정되지 않으면 null

  // 형식 매개변수를 초기화하는 생성형 생성자
  PointD(this.x, this.y);

  @override
  String toString() {
    return 'PointD($x,$y)';
  }
}
```

## 초기화 리스트 사용 Use an initializer list

생성자 본문이 실행되기 전에 인스턴스 변수를 초기화할 수 있습니다. 초기화를 쉼표로 구분하세요.

```dart
// 초기화 리스트는 생성자 내부가 실행되기 전에
// 인스턴스 변수를 설정합니다.
Point.fromJson(Map<String, double> json)
    : x = json['x']!,
      y = json['y']! {
  print('In Point.fromJson(): ($x, $y)');
}
```

```
경고

초기화 목록의 오른쪽은 this에 접근할 수 없습니다.
```

개발 중에 입력을 검증하려면 초기화 리스트에서 assert를 사용하세요.

```dart
Point.withAssert(this.x, this.y) : assert(x >= 0) {
  print('In Point.withAssert(): ($x, $y)');
}
```

초기화 리스트는 final 필드 설정에 도움이 됩니다.

다음 예시에서는 초기화 리스트의 세 final 필드를 초기화합니다. 코드를 실행하려면 Run을 클릭하세요.

```dart
import 'dart:math';

class Point {
  final double x;
  final double y;
  final double distanceFromOrigin;

  Point(double x, double y)
      : x = x,
        y = y,
        distanceFromOrigin = sqrt(x * x + y * y);
}

void main() {
  var p = Point(2, 3);
  print(p.distanceFromOrigin);
}
```

## 생성자 상속 Constructor inheritance

하위 클래스 또는 자식 클래스는 상위 클래스 또는 직계 부모 클래스에서 생성자를 상속하지 않습니다. 클래스가 생성자를 선언하지 않으면 기본 생성자 만 사용할 수 있습니다.

클래스는 상위 클래스의 매개변수를 상속받을 수 있습니다. 이것을 상위 매개변수라고 합니다.

Constructors work in a somewhat similar way to how you call a chain of static methods. Each subclass can call its superclass's constructor to initialize an instance, like a subclass can call a superclass's static method. This process doesn't "inherit" constructor bodies or signatures.

생성자는 dusthrehls static 메서드 호출하는 방법과 다소 유사한 방식으로 작동합니다. 하위 클래스가 상위 클래스의 static 메서드를 호출할 수 있는 것처럼 각 하위 클래스는 상위 클래스의 생성자를 호출하여 인스턴스를 초기화할 수 있습니다. 이 과정은 생성자 내부나 시그니처를 "상속"하지 않습니다.

## 기본이 아닌 슈퍼클래스 생성자

Dart는 다음 순서로 생성자를 실행합니다.

- 초기화 리스트

- 상위 클래스의 이름 없고 인자 없는 생성자.

- 메인 클래스의 인자 없는 생성자.

상위 클래스에 이름 없고 인자 없는 생성자가 없는 경우 상위 클래스의 생성자 중 하나를 호출합니다. 생성자 내부(있는 경우) 앞, 콜론( :) 뒤에 상위 클래스 생성자를 지정합니다.

다음 예제에서 Employee클래스 생성자는 해당 상위 클래스 Person에 대해 명명된 생성자를 호출합니다. 다음 코드를 실행하려면 Run을 클릭하세요.

```dart
class Person {
  String? firstName;

  Person.fromJson(Map data) {
    print('in Person');
  }
}

class Employee extends Person {
  // Person은 기본 생성자가 없습니다;
  // super.fromJson()을 반드시 호출해야합니다.
  Employee.fromJson(Map data) : super.fromJson(data) {
    print('in Employee');
  }
}

void main() {
  var employee = Employee.fromJson({});
  print(employee);
  // 출력:
  // Person 안의
  // Employee 안의
  // Instance of 'Employee'
}
```

Dart는 생성자를 호출하기 전에 상위 클래스 생성자에 대한 인자를 평가하므로 인자는 함수 호출과 같은 표현식이 될 수 있습니다.

```dart
class Employee extends Person {
  Employee() : super.fromJson(fetchDefaultData());
  // ···
}
```

```
경고

상위 클래스 생성자에 대한 인자는 this에 액세스할 수 없습니다. 예를 들어 인자는 static 메서드를 호출할 수 있지만 인스턴스 메서드는 호출할 수 없습니다.
```

## 상위 매개변수 Super parameters

To avoid passing each parameter into the super invocation of a constructor, use super-initializer parameters to forward parameters to the specified or default superclass constructor. You can't use this feature with redirecting constructors. Super-initializer parameters have syntax and semantics like initializing formal parameters.

각 매개변수를 생성자의 상위 호출에 전달하지 않으려면, 상위 초기화 매개변수를 사용하여 매개변수를 지정된 상위 클래스 생성자 또는 기본 상위 클래스 생성자에 전달하세요. 생성자 리디렉션과 함께 이 기능을 사용할 수 없습니다 . 상위 초기화 매개변수는 형식 매개변수 초기화와 같은 구문과 의미를 가집니다.

```
버전 노트

상위 초기화 매개변수를 사용하려면 언어 버전 2.17 이상이 필요합니다. 이전 언어 버전을 사용하는 경우 모든 상위 생성자 매개변수를 수동으로 전달해야 합니다.
```

상위 생성자 호출에 위치 인자가 포함된 경우 상위 초기화 매개변수는 위치 매개변수일 수 없습니다.

```dart
class Vector2d {
  final double x;
  final double y;

  Vector2d(this.x, this.y);
}

class Vector3d extends Vector2d {
  final double z;

  // x와 y 매개변수를 다음과 같이 기본 상위 생성자로 전달하세요:
  // Vector3d(final double x, final double y, this.z) : super(x, y);
  Vector3d(super.x, super.y, this.z);
}
```

더 자세히 설명하려면 다음 예시를 고려하세요.

```dart
  // 위치 인자와 함께 상위 생성자(`super(0)`)를 호출하는 경우
  // 상위 매개변수(`super.x`)를 사용하면 오류가 발생합니다.
  Vector3d.xAxisError(super.x): z = 0, super(0); // 나쁨
```

이 명명된 생성자는 x값을 두 번 설정하려고 시도합니다. 한 번은 상위 생성자에서, 한 번은 위치 상위 매개변수로 설정합니다. 둘 다 위치 매개변수 x를 다루므로 오류가 발생합니다.

When the super constructor has named arguments, you can split them between named super parameters (super.y in the next example) and named arguments to the super constructor invocation (super.named(x: 0)).

상위 생성자에 명명된 인자가 있는 경우, 명명된 상위 매개변수(다음 예시의 super.y)와 상위 생성자 호출(super.named(x: 0))에 대한 명명된 인자 간에 이를 분할할 수 있습니다.

```dart
class Vector2d {
  // ...
  Vector2d.named({required this.x, required this.y});
}

class Vector3d extends Vector2d {
  final double z;

  // y 매개변수를 명명된 상위 생성자에 다음과 같이 전달하세요:
  // Vector3d.yzPlane({required double y, required this.z})
  //       : super.named(x: 0, y: y);
  Vector3d.yzPlane({required super.y, required this.z}) : super.named(x: 0);
}
```