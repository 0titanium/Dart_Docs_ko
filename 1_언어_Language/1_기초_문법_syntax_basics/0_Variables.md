# 변수 Variables

다음은 변수를 생성하고 초기화하는 예입니다.

```dart
var name = 'Bob';
```

변수는 참조를 저장합니다. name이라는 변수에는 값이 "Bob"인 String 객체에 대한 참조가 포함되어 있습니다.

name 변수의 타입은 문자열로 추론되지만 String 타입으로 선언하여 해당 유형을 변경할 수 있습니다. 객체가 단일 타입으로 제한되지 않는 경우, Object 타입(또는 필요한 경우 dynamic)으로 선언합니다.

```dart
Object name = 'Bob';
```

또 다른 옵션은 추론할 타입을 명시적으로 선언하는 것입니다.

```dart
String name = 'Bob';
```

```
노트

이 페이지는 지역 변수에 유형 주석 대신 var를 사용하라는 스타일 가이드 권장 사항을 따릅니다.
```

## null 안정성 Null safety

Dart 언어는 견고한 null 안전성을 강제합니다.

null 안전성은 null로 설정된 변수에 대한 의도하지 않은 접근으로 인해 발생하는 오류를 방지합니다. 이 오류를 null 역참조 오류라고 합니다. null 역참조 오류는 속성에 접근하거나 null로 평가되는 식에서 메서드를 호출할 때 발생합니다. 이 규칙의 예외는 null이 toString() 또는 hashCode와 같은 속성이나 메서드를 지원하는 경우입니다. null 안전성을 통해 Dart 컴파일러는 컴파일 타임에 이러한 잠재적인 오류를 감지합니다.

예를 들어, int 변수 i의 절대값을 찾고 싶다고 가정해 보겠습니다. i가 null인 경우 i.abs()를 호출하면 null 역참조 오류가 발생합니다. 다른 언어에서는 이를 시도하면 런타임 오류가 발생할 수 있지만 Dart의 컴파일러는 이러한 작업을 금지합니다. 따라서 Dart 앱은 런타임 오류를 일으킬 수 없습니다.

null 안전성에는 세 가지 주요 변경 사항이 도입되었습니다.

1. 변수, 매개변수 또는 기타 관련 구성 요소에 대한 타입을 선언할 때, 해당 유형이 null을 허용하는지 여부를 제어할 수 있습니다. null 허용 여부를 활성화하려면 타입 선언의 끝에 물음표(?)를 추가하면 됩니다.

```dart
String? name  // nullable 타입. null 또는 문자열이 될 수 있습니다.

String name   // Non-nullable 타입. null은 될 수 없지만 문자열은 될 수 잇습니다.
```

2. 변수를 사용하기 전에 초기화해야 합니다. null 가능 변수는 기본적으로 null이므로 기본적으로 초기화됩니다. Dart는 null을 허용하지 않는 타입에 초기 값을 설정하지 않습니다. 이것은 초기값을 설정하도록 강제합니다. Dart에서는 초기화되지 않은 변수를 작성하는 것을 허용하지 않습니다. 이렇게 하면 리시버의 타입이 null일 수 있지만 null이 사용된 메서드나 속성을 지원하지 않는 속성에 접근하거나 메서드를 호출하는 것을 방지할 수 있습니다.

3. nullable 타입이 있는 표현식에서는 속성에 접근하거나 메서드를 호출할 수 없습니다. hashCode 또는 toString()과 같이 null이 지원하는 속성이나 메서드인 경우에도 동일한 예외가 적용됩니다.

견고한 null 안정성은 잠재적인 런타임 오류를 편집 시 분석 오류로 변경합니다. null 안전성은 다음 중 하나에 해당하는 경우 null이 아닌 변수를 표시합니다.

- null이 아닌 값이 초기화 되지 않았을 때

- null 값이 할당 되었을 때

이 검사를 통해 앱을 배포하기 전에 이러한 오류를 수정할 수 있습니다.

## 기본값 Default value

null 허용 유형이 있는 초기화되지 않은 변수의 초기 값은 null입니다. 숫자 유형이 있는 변수도 초기에는 null입니다. 숫자는 Dart의 다른 모든 것과 마찬가지로 객체이기 때문입니다.

```dart
int? lineCount;
assert(lineCount == null);
```

```
노트

프로덕션 코드는 Assert() 호출을 무시합니다. 반면 개발 중에는 조건이 거짓인 경우 assert(조건)이 예외를 발생시킵니다. 자세한 내용은 Assert를 확인하세요.
```

null 안전성을 사용하면 null을 허용하지 않는 변수를 사용하기 전에 해당 값을 초기화해야 합니다.

```dart
int lineCount = 0;
```

선언된 지역 변수를 초기화할 필요는 없지만 사용하기 전에 값을 할당해야 합니다. 예를 들어, 다음 코드는 print()에 전달될 때 lineCount가 null이 아님을 Dart가 감지할 수 있기 때문에 유효합니다.

```dart
int lineCount;

if (weLikeToCount) {
  lineCount = countLines();
} else {
  lineCount = 0;
}

print(lineCount);
```

최상위 및 클래스 변수는 느리게 초기화됩니다. 초기화 코드는 변수가 처음 사용될 때 실행됩니다.

## Late 변수 Late variables

late 제어자는 두 가지 사용례를 갖고 있습니다.

- 선언 후 초기화되는 null을 허용하지 않는 변수를 선언합니다.

- 변수를 느리게 초기화합니다.

종종 Dart의 제어 흐름 분석은 null이 불가능한 변수가 사용되기 전에 null이 아닌 값으로 설정되는 경우를 감지할 수 있지만 때로는 분석이 실패합니다. 두 가지 일반적인 경우는 최상위 변수와 인스턴스 변수입니다. Dart는 종종 설정 여부를 확인할 수 없으므로 시도하지 않습니다.

변수가 사용되기 전에 설정되었다고 확신하지만 Dart가 이에 동의하지 않는 경우 변수를 late로 표시하여 오류를 수정할 수 있습니다.

```dart
late String description;

void main() {
  description = 'Feijoada!';
  print(description);
}
```

```
주의

늦은 변수를 초기화하지 못하면 해당 변수를 사용할 때 런타임 오류가 발생합니다.
```

변수를 late로 표시했지만 선언 시 초기화하면 변수가 처음 사용될 때 초기화 함수가 실행됩니다. 이 지연 초기화는 다음과 같은 몇 가지 경우에 유용합니다.

- 변수가 필요하지 않으며 초기화에 많은 비용이 들 때

- 인스턴스 변수를 초기화하고 이에 대한 초기화 함수가 인스턴스 변수에 접근해야 할 때

다음 예에서 temperature 변수가 전혀 사용되지 않으면 비용이 많이 드는 readThermometer() 함수가 호출되지 않습니다.

```dart
// 이것은 readThermometer()에 대한 프로그램의 유일한 호출입니다.
late String temperature = readThermometer(); // 지연된 초기화.
```

## final과 const

변수를 변경할 생각이 없다면 var 또는 타입을 선언하는 것 대신 final 또는 const를 사용하세요. final 변수는 한 번만 설정할 수 있습니다. const 변수는 컴파일 타임 상수입니다. (Const 변수는 암시적으로 final 변수입니다.)


```
노트

인스턴스 변수는 final은 될 수 있지만 const는 될 수 없습니다.
```

여기에 final 변수를 생성하고 설정하는 예가 있습니다.

```dart
final name = 'Bob'; // 타입 선언 없이
final String nickname = 'Bobby';
```

final 변수의 값은 변경할 수 없습니다.

```dart
// 정적 분석: 실패
name = 'Alice'; // 오류: final 변수는 한번만 설정될 수 있습니다.
```

컴파일 타임 상수로 사용하려는 변수에는 const를 사용하세요. const 변수가 클래스 수준에 있는 경우 이를 static const로 표시합니다. 변수를 선언할 때 숫자나 문자열 리터럴, const 변수 또는 const 숫자에 대한 산술 연산 결과와 같은 값은 컴파일 타임 상수로 설정하세요.

```dart
const bar = 1000000; // 압력 단위 (dynes/cm2)
const double atm = 1.01325 * bar; // 국제 표준 단위
```

const 키워드는 단지 상수 변수를 선언하기 위한 것이 아닙니다. 또한 이를 사용하여 상수 값을 생성할 수 있을 뿐만 아니라 상수 값을 생성하는 생성자를 선언할 수도 있습니다. 모든 변수는 상수 값을 가질 수 있습니다.

```dart
var foo = const [];
final bar = const [];
const baz = []; // Equivalent to `const []`
```

위의 baz와 같이 const 선언의 초기화 표현식에서 const를 생략할 수 있습니다. 자세한 내용은 const를 중복하여 사용하지 마세요를 참조하세요.

const 값을 가졌던 경우에도 non-final non-const가 아닌 변수의 값을 변경할 수 있습니다.

```dart
foo = [1, 2, 3]; // Was const []
```

const 변수의 값은 변경할 수 없습니다.

```dart
baz = [42]; // 오류: const 변수는 값을 할당할 수 없습니다.
```

타입 체크와 타입 캐스트(is와 as), collection if, 스프레드 연산자(...와 ...?)를 사용하는 const 정의가 가능합니다.

```dart
const Object i = 3; // i는 정수 값을 가진 const Object입니다.
const list = [i as int]; // 타입 캐스트를 사용하세요.
const map = {if (i is int) i: 'int'}; // is와 collection if를 사용하세요.
const set = {if (list is List<int>) ...list}; // ...를 사용하여 전개하세요.
```

```
노트

final 객체는 수정될 수 없지만, final 객체의 필드는 바뀔 수 있습니다. 비교하자면 const 객체와 그 필드는 바뀔 수 없습니다.
const 객체와 그 필드는 불변입니다. 
```

const를 사용하여 상수 값을 만드는 방법에 대한 자세한 내용은 Lists, Maps 및 클래스를 참조하세요.
