# 메서드 Methods

메소드는 객체에 대한 동작을 제공하는 함수입니다.

## 인스턴스 메서드 Instance methods

객체의 인스턴스 메소드는 인스턴스 변수 및 this에 접근할 수 있습니다. 다음 샘플의 메서드 distanceTo()는 인스턴스 메서드의 예시입니다.

```dart
import 'dart:math';

class Point {
  final double x;
  final double y;

  // 생성자 내부가 실행되기 전에
  // 인스턴스 변수 x와 y를 설정합니다.
  Point(this.x, this.y);

  double distanceTo(Point other) {
    var dx = x - other.x;
    var dy = y - other.y;
    return sqrt(dx * dx + dy * dy);
  }
}
```

## 연산자 Operators

대부분의 연산자는 특별한 이름을 가진 인스턴스 메서드입니다. Dart를 사용하면 다음 이름으로 연산자를 정의할 수 있습니다.

| | | | | | |
|---|---|---|---|---|---|
| < | > | <= | >= | == | ~ |
| - | + | / | ~/ | * | % |
| \| | ^ | & | << | >>> | >> |
| []= | [] | | | | |

```
노트

!=와 같은 일부 연산자가 목록에 없다는 것을 눈치채셨을 겁니다. 이러한 연산자는 인스턴스 메서드가 아닙니다.

이들의 행동은 Dart에 내장되어 있습니다.
```

연산자를 선언하려면 내장된 식별자 operator를 사용한 다음 정의하는 연산자를 사용하십시오. 다음 예시에서는 벡터 더하기(+), 빼기(-) 및 동등(==)을 정의합니다.

```dart
class Vector {
  final int x, y;

  Vector(this.x, this.y);

  Vector operator +(Vector v) => Vector(x + v.x, y + v.y);
  Vector operator -(Vector v) => Vector(x - v.x, y - v.y);

  @override
  bool operator ==(Object other) =>
      other is Vector && x == other.x && y == other.y;

  @override
  int get hashCode => Object.hash(x, y);
}

void main() {
  final v = Vector(2, 3);
  final w = Vector(2, 2);

  assert(v + w == Vector(4, 5));
  assert(v - w == Vector(0, 1));
}
```

## 게터와 세터 Getters and Setters

Getter 및 Setter는 객체의 속성에 대한 읽기 및 쓰기 접근을 제공하는 특수 메서드입니다. 각 인스턴스 변수에는 암시적 getter가 있고 적절한 경우 setter가 있다는 점을 기억하세요. get과 set 키워드를 사용하여 getter 및 setter를 구현하여 추가 속성을 생성할 수 있습니다.

```dart
class Rectangle {
  double left, top, width, height;

  Rectangle(this.left, this.top, this.width, this.height);

  // 두 계산된 속성 right과 bottom을 정의하세요:
  double get right => left + width;
  set right(double value) => left = value - width;
  double get bottom => top + height;
  set bottom(double value) => top = value - height;
}

void main() {
  var rect = Rectangle(3, 4, 20, 15);
  assert(rect.left == 3);
  rect.right = 12;
  assert(rect.left == -8);
}
```

getter 및 setter를 사용하면 클라이언트 코드를 변경하지 않고도 인스턴스 변수로 시작하여 나중에 메서드로 래핑할 수 있습니다.

```
노트

증가(++)와 같은 연산자는 getter가 명시적으로 정의되었는지 여부에 관계없이 예상대로 작동합니다.

예상치 못한 부작용을 방지하기 위해 연산자는 getter를 정확히 한 번 호출하여 해당 값을 임시 변수에 저장합니다.
```

## 추상 메서드 Absctract method

인스턴스, getter 및 setter 메서드는 추상화되어 인터페이스를 정의하지만 구현은 다른 클래스에 맡길 수 있습니다. 추상 메서드는 추상 클래스나 믹스인에만 존재할 수 있습니다.

메서드를 추상화하려면 메서드 내부 대신 세미콜론(;)을 사용하십시오.

```dart
abstract class Doer {
  // 인스턴스 변수와 메서드를 정의하세요...

  void doSomething(); // 추상 메서드를 정의하세요.
}

class EffectiveDoer extends Doer {
  void doSomething() {
    // 구현을 제공하므로 여기서 메서드는 추상화되지 않습니다...
  }
}
```

