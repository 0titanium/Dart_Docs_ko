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