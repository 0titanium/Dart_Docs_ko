# 열거된 타입 Enumerated types

열거된 타입(종종 enumerations나 enums로 불리는)은 상수 변수의 고정된 숫자를 표현하기 위해 사용되는 특별한 종류의 클래스입니다.

```
노트

모든 열거형은 자동으로 Enum 클래스를 확장합니다. 또한 봉인되어 있으므로 하위 클래스화, 구현, 혼합 또는 명시적으로 인스턴스화할 수 없습니다.

추상 클래스와 믹스인은 명시적으로 Enum을 구현하거나 확장할 수 있지만, 열거형 선언에 의해 구현되거나 혼합되지 않는 한 어떤 객체도 실제로 해당 클래스나 믹스인의 유형을 구현할 수 없습니다.
```

## 간단한 열거형 선언 Declaring simple enums

단순힌 열거형 타입을 선언하려면 enum 키워드를 사용하여 열거되기 원하는 값을 나열하세요.

```dart
enum Color { red, green, blue }
```

```
팁

복사-붙여넣기 오류를 방지하기 위해 열거형 타입을 선언할 때 후행 쉼표를 사용할 수도 있습니다.
```

## 향상된 열거형 선언 Declaring enhanced enums

Dart에서는 또한 enum 선언을 통해 필드, 메서드 및 고정된 수의 알려진 상수 인스턴스로 제한되는 const 생성자를 사용하여 클래스를 선언할 수 있습니다.

향상된 enum을 선언하려면 클래스와 비슷한 문법을 따르지만 추가적인 요구사항이 있습니다.

- mixin에 의해 추가된 변수를 포함한 인스턴스 변수는 반드시 final이어야 합니다.
- 모든 생성형 생성자는 반드시 상수여야 합니다.
- 팩토리 생성자는 고정되고 알려진 열거형 인스턴스 중 하나만 반환할 수 있습니다.
- Enum은 자동으로 확장되므로 다른 클래스는 Enum으로 확장될 수 없습니다.
- index, hashCode, 동등 연산자 ==은 재정의될 수 없습니다.
- values라고 명명된 멤버는 열거형에서 선언될 수 없습니다. 자동으로 생성된 정적 values getter와 충돌할 수 있기 때문입니다.
- 열거형의 모든 인스턴스는 반드시 선언 초기에 선언되어야합니다. 그리고 최소한 하나의 인스턴스가 선언되어야 합니다.

향상된 열거형의 인스턴스 메서드는 현저 열거형 값을 참조하기 위해 this를 사용할 수 있습니다.

아래에 여러 인스턴스, 인스턴수 변수, getter, 구현된 인터페이스가 포함된 향상된 열거형을 선언한 예시가 있습니다.

```dart
enum Vehicle implements Comparable<Vehicle> {
  car(tires: 4, passengers: 5, carbonPerKilometer: 400),
  bus(tires: 6, passengers: 50, carbonPerKilometer: 800),
  bicycle(tires: 2, passengers: 1, carbonPerKilometer: 0);

  const Vehicle({
    required this.tires,
    required this.passengers,
    required this.carbonPerKilometer,
  });

  final int tires;
  final int passengers;
  final int carbonPerKilometer;

  int get carbonFootprint => (carbonPerKilometer / passengers).round();

  bool get isTwoWheeled => this == Vehicle.bicycle;

  @override
  int compareTo(Vehicle other) => carbonFootprint - other.carbonFootprint;
}
```

```
버전 노트

향상된 열거형은 최소한 2.17 이상의 언어 버전을 요구합니다.
```

## 열거형 사용 Using enums

다른 정적 변수와 같이 열거된 값에 접근하세요:

```dart
final favoriteColor = Color.blue;

if (favoriteColor == Color.blue) {
  print('Your favorite color is blue!');
}
```

열거형의 각 값에는 열거형 선언 내부에서 값의 0부터 시작하는 위치를 반환하는 index getter가 있습니다. 예를 들어 첫 번째 값의 인덱스는 0이고 두 번째 값의 인덱스는 1입니다.

```dart
assert(Color.red.index == 0);
assert(Color.green.index == 1);
assert(Color.blue.index == 2);
```

모든 열거된 값의 목록을 얻으려면 enum의 values 상수를 사용하세요.

```dart
List<Color> colors = Color.values;
assert(colors[2] == Color.blue);
```

switch 문 안에서 열거형을 사용할 수 있습니다. 모든 열거형의 값을 다루지 않으면 경고를 받습니다.

```dart
var aColor = Color.blue;

switch (aColor) {
  case Color.red:
    print('Red as roses!');
  case Color.green:
    print('Green as grass!');
  default: // 이 부분이 없으면, 경고를 보게 됩니다.
    print(aColor); // 'Color.blue'
}
```

Color.blue에서 'blue'와 같이 열거된 값의 이름에 접근해야한다면 .name 속성을 사용하세요.

```dart
print(Color.blue.name); // 'blue'
```

일반 객체에서와 마찬가지로 열거형 값의 멤버에 접근할 수 있습니다.

```dart
print(Vehicle.car.carbonFootprint);
```