# 클래스 확장 Extends a class

extneds는 하위 클래스를 만드는 데, super는 상위 클래스를 참조하는 데 사용하세요.

```dart
class Television {
  void turnOn() {
    _illuminateDisplay();
    _activateIrSensor();
  }
  // ···
}

class SmartTelevision extends Television {
  void turnOn() {
    super.turnOn();
    _bootNetworkInterface();
    _initializeMemory();
    _upgradeApps();
  }
  // ···
}
```

extends의 다른 사용법에 대해서는 제네릭 페이지의 매개변수화된 타입에 대한 토론을 참조하세요.

## 멤버 재정의 Overriding members

하위 클래스는 인스턴스 메서드(연산자 포함), getter 및 setter를 재정의할 수 있습니다. @override 주석을 사용하여 의도적으로 멤버를 재정의하고 있음을 나타낼 수 있습니다.

```dart
class Television {
  // ···
  set contrast(int value) {...}
}

class SmartTelevision extends Television {
  @override
  set contrast(num value) {...}
  // ···
}
```

메서드 재정의 선언은 여러 가지 방법으로 재정의하는 메서드와 일치해야 합니다.

- 반환 타입은 재정의된 메서드의 반환 타입과 동일한 타입(또는 하위 타입)이어야 합니다.

- 매개변수 타입은 재정의된 메서드의 매개변수 타입과 동일한 타입(또는 상위 타입)이어야 합니다. 앞의 예시에서 SmartTelevision의 contrast setter는 매개변수 타입을 int 에서 상위 타입인 num으로 변경합니다.

- 재정의된 메서드가 n개의 위치 매개변수를 허용하는 경우 재정의하는 메서드는 n개의 위치 매개변수도 허용해야 합니다.

- 제네릭 메서드는 제네릭이 아닌 메서드를 재정의할 수 없으며 제네릭이 아닌 메서드는 제네릭 메서드를 재정의할 수 없습니다.

때로는 메서드 매개변수나 인스턴스 변수의 타입을 좁히고 싶을 수도 있습니다. 이는 일반적인 규칙을 위반하며 런타임에 타입 오류를 일으킬 수 있다는 점에서 다운캐스팅과 유사합니다. 그러나 코드에서 타입 오류가 발생하지 않는다는 것을 보장할 수 있다면 타입을 좁힐 수 있습니다. 이 경우 매개변수 선언에 covariant 키워드를 사용할 수 있습니다. 자세한 내용은 Dart 언어 사양을 참조하세요.

```
경고

== 을 재정의하는 경우 객체의 hashCode getter도 재정의해야 합니다. == 및 hashCoded 재정의에 대한 예시 맵 키 구현에서 확인하세요.
```

## noSuchMethod()

코드가 존재하지 않는 메서드나 인스턴스 변수를 사용하려고 시도할 때마다 감지하거나 반응하려면 noSuchMethod()를 재정의하면 됩니다.

```dart
class A {
  // noSuchMethod를 재정의하지 않는 한,
  // 존재하지 않는 멤버를 사용하면 NoSuchMethodError가 발생합니다.
  @override
  void noSuchMethod(Invocation invocation) {
    print('You tried to use a non-existent member: '
        '${invocation.memberName}');
  }
}
```

다음 중 하나에 해당하지 않는 한 구현되지 않은 메서드를 호출할 수 없습니다.

- 수신자는 정적 타입 dynamic을 가집니다.

- 수신자는 구현되지 않은 메서드(추상은 괜찮음)를 정의하는 정적 타입을 가지고, 수신자의 동적 타입은 Object 클래스와의 것과는 다른 noSuchMethod()의 구현을 가집니다.

자세한 내용은 비공식 noSuchMethod 전달 사양을 참조하세요.