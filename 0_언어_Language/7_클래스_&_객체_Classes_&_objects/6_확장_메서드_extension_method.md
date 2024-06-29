# 확장 메서드 Extension methods

확장 메서드는 기존 라이브러리에 기능을 추가합니다. 확장 메서드를 알지 못한 채 사용할 수도 있습니다. 예를 들어, IDE에서 코드 완성을 사용하면 일반 메서드와 함께 확장 메서드를 제안합니다.

비디오를 시청하는 것이 학습에 도움이 된다면 확장 방법 개요를 확인해 보세요.

## 개요 Overview

다른 사람의 API를 사용하거나 널리 사용되는 라이브러리를 구현할 때 API를 변경하는 것은 종종 비실용적이거나 불가능합니다. 하지만 여전히 일부 기능을 추가하고 싶을 수 있습니다.

예를 들어, 문자열을 정수로 파싱하는 다음 코드를 생각해 보세요.

```dart
int.parse('42')
```

대신에 해당 기능을 String에 두는 것이 좋을 것입니다. 즉, 도구를 사용하여 더 짧고 쉽게 사용할 수 있습니다.

```dart
'42'.parseInt()
```

해당 코드를 활성화하려면 String 클래스의 확장이 포함된 라이브러리를 가져올 수 있습니다.

```dart
import 'string_apis.dart';
// ···
print('42'.parseInt()); // 확장 메서드 사용.
```

확장은 메서드뿐만 아니라 getter, setter, 연산자와 같은 다른 멤버도 정의할 수 있습니다. 또한 확장은 이름을 가질 수 있으며, 이는 API 충돌이 발생하는 경우 도움이 될 수 있습니다. 다음은 문자열에 대해 작동하는 확장(NumberParsing이라는)을 사용하여 확장 메서드 parseInt()를 구현하는 방법입니다.

```dart
extension NumberParsing on String {
  int parseInt() {
    return int.parse(this);
  }
  // ···
}
```

다음 섹션에서는 확장 메서드를 사용하는 방법을 설명합니다. 그 다음에는 확장 메서드 구현 에 대한 섹션이 있습니다.

## 확장 메서드 사용 Using extension methods

모든 Dart 코드와 마찬가지로 확장 메서드도 라이브러리에 있습니다. 확장 메서드를 사용하는 방법을 이미 살펴보았습니다. 확장 메서드가 포함된 라이브러리를 가져와서 일반 메서드처럼 사용하면 됩니다.

```dart
// String에 대한 확장을 포함하는 라이브러리를 가져오세요. Import a library that contains an extension on String.
import 'string_apis.dart';
// ···
print('42'.padLeft(5)); // String 메서드 사용.
print('42'.parseInt()); // 확장 메서드 사용.
```

이것이 확장 메서드를 사용하기 위해 일반적으로 알아야 할 전부입니다. 코드를 작성할 때 확장 메서드가 정적 타입(dynamic과 반대로)에 의존하는 방식과 API 충돌을 해결하는 방법을 알아야 할 수도 있습니다.

## 정적 타입과 dynamic Static types and dynamic

dynamic 타입의 변수에 확장 메서드를 호출할 수 없습니다. 예를 들어, 다음 코드는 런타임 예외를 발생시킵니다.

```dart
dynamic d = '2';
print(d.parseInt()); // Runtime exception: NoSuchMethodError
```

확장 메서드는 Dart의 타입 추론과 함께 작동합니다 . 다음 코드는 변수 v가 String 타입을 갖는 것으로 추론되기 때문에 괜찮습니다.

```dart
var v = '2';
print(v.parseInt()); // Output: 2
```

dynamic이 작동하지 않는 이유는 확장 메서드가 수신자의 정적 타입에 대해 해결되기 때문입니다. 확장 메서드는 정적으로 해결되므로 정적 함수를 호출하는 것만큼 빠릅니다.

정적 타입 및 dynamic에 대한 자세한 내용은 Dart 타입 시스템을 참조하세요.

## API 충돌 API conflicts

확장 멤버가 인터페이스나 다른 확장 멤버와 충돌하는 경우, 몇 가지 옵션이 있습니다.

한 가지 옵션은 충돌하는 확장을 가져오는 방법을 변경하여 show와 hide를 사용해 노출된 API를 제한하는 것입니다.

```dart
// String의 확장 메서드인 parseInt() 정의합니다.
import 'string_apis.dart';

// 또한 parseInt()를 정의하지만 NumberParsing2를 숨기면
// 해당 확장 메서드가 숨겨집니다.
import 'string_apis_2.dart' hide NumberParsing2;

// ···
// 'string_apis.dart'에 정의된 parseInt()를 사용합니다.
print('42'.parseInt());
```

또 다른 옵션은 확장을 명시적으로 적용하는 것인데, 그러면 확장이 래퍼 클래스인 것처럼 보이는 코드가 생성됩니다.

```dart
// 두 라이브러리 모두 parsInt()를 포함하는 String에 대한 확장을 정의하며
// 확장은 서로 다른 이름을 갖습니다.

import 'string_apis.dart'; // NumberParsing 확장을 포함합니다.
import 'string_apis_2.dart'; // NumberParsing2 확장을 포함합니다.

// ···
// print('42'.parseInt()); // 작동하지 않습니다.
print(NumberParsing('42').parseInt());
print(NumberParsing2('42').parseInt());
```

두 확장의 이름이 동일한 경우 접두사를 사용하여 가져와야 할 수도 있습니다.

```dart
// 두 라이브러리 모두 확장 메서드인 parseInt()를 포함하는 NumberParsing이라는 확장을 정의합니다.
// 하나의 NumberParsing 확장('string_apis_3.dart'에 있는)도 parseNum()을 정의합니다.
import 'string_apis.dart';
import 'string_apis_3.dart' as rad;

// ···
// print('42'.parseInt()); // 작동하지 않습니다.

// string_apis.dart의 ParseNumbers 확장을 사용하세요.
print(NumberParsing('42').parseInt());

// string_apis_3.dart의 ParseNumbers 확장을 사용하세요.
print(rad.NumberParsing('42').parseInt());

// string_apis_3.dart에만 parseNum()이 있습니다..
print('42'.parseNum());
```

예에서 보듯이 접두사를 사용하여 가져오더라도 확장 메서드를 암묵적으로 호출할 수 있습니다. 접두사를 사용해야 하는 유일한 경우는 확장을 명시적으로 호출할 때 이름 충돌을 피하기 위한 것입니다.

## 확장 메서드 구현 Implementing extension methods

다음 문법을 사용하여 확장 메서드를 만드세요.

```dart
extension <extension name>? on <type> { // <extension-name>은 선택접입니다.
  (<member definition>)* // 하나 이상의 <member definition>를 제공할 수 있습니다.
}
```

예를 들어, String 클래스에 확장을 구현하는 방법은 다음과 같습니다.

```dart
extension NumberParsing on String {
  int parseInt() {
    return int.parse(this);
  }

  double parseDouble() {
    return double.parse(this);
  }
}
```

확장의 멤버는 메서드, getter, setter 또는 연산자일 수 있습니다. 확장에는 정적 필드와 정적 헬퍼 메서드가 있을 수도 있습니다. 확장 선언 외부의 정적 멤버에 접근하려면 클래스 변수 및 메서드와 같은 선언 이름을 통해 호출합니다.

## 이름이 없는 확장 Unnamed extensions

확장자를 선언할 때 이름을 생략할 수 있습니다. 이름없는 확장은 선언된 라이브러리에서만 볼 수 있습니다. 이름이 없으므로 API 충돌을 해결하기 위해 명시적으로 적용할 수 없습니다 .

```dart
extension on String {
  bool get isBlank => trim().isEmpty;
}
```

```
노트

확장 선언 내에서만 이름없는 확장의 정적 멤버를 호출할 수 있습니다.
```

## 제네릭 확장 구현 Implementing generic extensions

확장에는 제네릭 타입 매개변수가 있을 수 있습니다. 예를 들어, 다음은 getter, 연산자 및 메서드를 사용하여 내장된 List<T> 타입을 확장하는 일부 코드입니다.

```dart
extension MyFancyList<T> on List<T> {
  int get doubleLength => length * 2;
  List<T> operator -() => reversed.toList();
  List<List<T>> split(int at) => [sublist(0, at), sublist(at)];
}
```

T 타입은 메서드가 호출되는 리스트의 정적 타입을 기반으로 바인딩됩니다.

## 자원 Resources

확장 메서드에 대한 자세한 내용은 다음을 참조하세요.

- 기사: 다트 확장 메서드 기초
- 기능 사양
- 확장 메서드 샘플