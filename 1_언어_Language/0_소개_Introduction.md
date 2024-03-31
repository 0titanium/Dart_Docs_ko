# Dart 언어 소개

이 페이지는 주요 특징들의 예시들을 통해 Dart 언어에 대한 간략한 소개를 제공합니다.

Dart 언어에 대해 자세히 알아보려면 왼쪽 메뉴의 언어 아래에 나열된 심층적이고 개별적인 주제 페이지를 방문하세요.

Dart의 핵심 라이브러리들에 대한 내용은 핵심 라이브러리 문서를 확인하세요. 더 실제적인 소개에 대해서는 Dart에서 자주 사용되는 명령이나 기능들의 코드랩을 사용해 볼 수도 있습니다.

## Hello word

모든 앱에는 실행이 시작되는 top-level(어느 코드 블럭에도 속하지 않는 최상위 영역)의 main() 함수가 필요합니다. 명시적으로 값을 반환하지 않는 함수들은 반환 타입이 void입니다. 콘솔에 텍스트를 표시하기 위해서 최상위 영역의 print() 함수를 사용할 수 있습니다.

```dart

void main() {
  print('Hello, World!);
}

```

명령줄 전달 인자에 대한 선택적 매개변수를 포함한 Dart의 main() 함수에 대해 더 자세히 알아보세요.

## 변수 Variables

타입이 안전한 Dart 코드에서도 var를 사용하여 타입을 명시적으로 지정하지 않고도 대부분의 변수를 선언할 수 있습니다. 타입 추론 덕분에 이러한 변수의 타입은 초기 값에 따라 결정됩니다.

```dart
var name = 'Voyager I'; // String

var year = 1977; // int

var antennaDiameter = 3.7; // double

var flybyObjects = ['Jupiter', 'Saturn', 'Uranus', 'Neptune']; // List

var image = { // Map
  'tags': ['saturn'],
  'url': '//path/to/saturn.jpg'
};

```

기본 값, final 및 const 키워드와 static 타입을 포함한 Dart의 변수에 대해 자세히 알아보세요.

## 제어 흐름 구문 Control flow statements

Dart는 익숙한 흐름 제어 구문을 지원합니다.

```dart

if (year >= 2001) {
  print('21st century');
} else if (year >= 1901) {
  print('20th century');
}

for (final object in flybyObjects) {
  print(object);
}

for (int month = 1; month <= 12; month++) {
  print(month);
}

while (year < 2016) {
  year += 1;
}
```

break, continue, switch and case, assert를 포함한 Dart의 흐름 제어 구문에 대해 자세히 알아보세요.

## 함수 Functions

우리는 각 함수의 매개변수와 반환 값의 타입을 명시하기를 추천합니다.

```dart
int fibonacci(int n) {
  if (n == 0 || n == 1) return n;

  return fibonacci(n - 1) + fibonacci(n - 2);
}



var result = fibonacci(20);

```

단축형 => (화살표) 구문은 단일 명령문을 포함하는 함수에 유용합니다. 이 구문은 익명 함수를 인자로 전달할 때 특별히 유용합니다.

```dart
flybyObjects.where((name) => name.contains('turn')).forEach(print);
```

익명 함수(where()의 전달 인자)를 표시하는 것 외에도, 이 코드는 함수를 전달 인자처럼 사용할 수 있다는 것을 보여줍니다. 최상위 수준의 print() 함수는 forEach()의 전달 인자입니다.

선택적 매개변수, 기본 매개변수 값, 정적 범위를 포함  Dart의 함수에 대해서 자세히 알아보세요.

## 주석 Comments

Dart의 주석은 대개 //로 시작합니다.

```dart

// 이것은 일반적인 한 줄 주석입니다.

/// 이것은 라이브러리 문서, 클래스 및 해당  멤버를 문서화 하는데 사용되는

/// 문서 주석입니다. IDE 및 dartdoc과 같은 도구는 문서 주석을 특별히

/// 처리합니다.

/* 이러한 형태의 주석도 지원합니다. */

```

문서 도구의 작동 방식을 포함한 Dart의 주석에 대해 더 자세히 알아보세요.

## 가져오기 Imports

다른 라이브러리에 정의된 API에 접근하기 위해서 import를 사용하세요.

```dart
// 핵심 라이브러리를 가져오기
import 'dart:math';

// 외부 패키지에서 라이브러리를 가져오기
import 'package:test/test.dart';

// 파일 가져오기
import 'path/to/my_other_file.dart';
```

라이브러리 접두사, show와 hide, deffered 키워드를 통한 lazy loading 포함한 Dart의 라이브러리와 가시성에 대해 더 자세히 알아보세요.

## 클래스 Classes

여기에 세 속성과 두 생성자, 한 메서드를 가진  클래스의 예시가 있습니다. 속성 중의 하나는 직접 설정할 수 없으므로 변수 대신 getter 메서드를 사용하여 정의됩니다. 이 메서드는 문자열 보간법을 사용하여 문자열 리터럴 내부의 변수의 문자열에 해당하는 항목을 출력합니다.

```dart
class Spacecraft {
  String name;
  DateTime? launchDate;

  // 읽기 전용 논 파이널 속성
  int? get launchYear => launchDate?.year;

  // 멤버에게 할당하기 위한 문법 설탕이 포함된 생성자
  Spacecraft(this.name, this.launchDate) {
    // 여기에 초기화 코드를 사용합니다.
  }

  // 기본 생성자로 전달되는 명명된 생성자
  Spacecraft.unlaunched(String name) : this(name, null);

  // 메서드
  void describe() {
    print('Spacecraft: $name');

    // 타입 승격은 getter에서 작동하지 않습니다.
    var launchDate = this.launchDate;

    if (launchDate != null) {
      int years = DateTime.now().difference(launchDate).inDays ~/ 365;

      print('Launched: $launchYear ($years years ago)');
    } else {
      print('Unlaunched');
    }
  }
}
```

문자열 보간법, 리터럴, 표현식, toString() 메서드를 포함한 문자열에 대해서 자세히 알아보세요.

Spacecraft 클래스를 이런 식으로 사용할 수 있습니다.

```dart
var voyager = Spacecraft('Voyager I', DateTime(1977, 9, 5));

voyager.describe();

var voyager3 = Spacecraft.unlaunched('Voyager III');

voyager3.describe();
```

이니셜라이저 리스트, 선택적 new, const, 리디렉션 생성자, factory 생성자, getter, setter를 비롯한 Dart의 클래스의 더 많은 내용에 대해 자세히 알아보세요.

## 열거형 Enums

열거형은 해당 타입의 다른 인스턴스가 있을 수 없도록 미리 정의된 값 또는 인스턴스 집합을 열거하는 방법입니다.

다음은 미리 정의된 행성 타입의 간단한 목록을 정의하는 간단한 열거형의 예입니다.

```dart
enum PlanetType { terrestrial, gas, ice }
```

다음은 정의된 상수 인스턴스의 집합, 즉 우리 태양계의 행성을 사용하여  행성을 설명하는 클래스의 향상된 열거형 선언의 예시가 있습니다.

```dart
/// 태양계의 다양한 행성과 그 속성 중 일부를 열거하는 열거형
enum Planet {
  mercury(planetType: PlanetType.terrestrial, moons: 0, hasRings: false),

  venus(planetType: PlanetType.terrestrial, moons: 0, hasRings: false),
  // ···

  uranus(planetType: PlanetType.ice, moons: 27, hasRings: true),

  neptune(planetType: PlanetType.ice, moons: 14, hasRings: true);

  /// 상수 생성 생성자
  const Planet(
      {required this.planetType, required this.moons, required this.hasRings});

  /// 모든 인스턴스 변수는 final입니다.
  final PlanetType planetType;
  final int moons;
  final bool hasRings;

  /// 향상된 열거형은 getter 및 기타 메서드를 지원합니다.
  bool get isGiant =>
      planetType == PlanetType.gas || planetType == PlanetType.ice;
}
```

다음과 같이 Planet 열거형을 사용할 수 있습니다.

```dart

final yourPlanet = Planet.earth;

if (!yourPlanet.isGiant) {
  print('Your planet is not a "giant planet".');
}
```

향상된 열거형 요구 사항, 자동으로 도입된 속성, 열거된 값의 이름에 접근, switch 문 지원 등을 포함하여 Dart의 열거형에 대해 자세히 알아보세요.

## 상속 Inheritance

Dart는 단일 상속을 가집니다.

```dart
class Orbiter extends Spacecraft {
  double altitude;

  Orbiter(super.name, DateTime super.launchDate, this.altitude);
}
```

클래스 확장, 선택적 @override 주석 등에 대해 자세히 알아보세요.

## 믹스인 Mixins

믹스인은 여러 클래스 계층에서 코드를 재사용하는 방법입니다. 다음은 믹스인 선언의 예시입니다.

```dart
mixin Piloted {
  int astronauts = 1;

  void describeCrew() {
    print('Number of astronauts: $astronauts');
  }
}
```

믹스인의 기능을 클래스에 추가하려면 믹스인을 사용하여 해당 클래스를 확장하면 됩니다.

```dart

class PilotedCraft extends Spacecraft with Piloted {
  // ···
}
```

PilotedCraft는 이제 astronauts 필드와 describeCrew() 메서드를 가집니다.

믹스인에 대해 자세히 알아보세요.

## 인터페이스와 추상 클래스 Interface and abstract classes

모든 클래스는 암시적으로 인터페이스를 정의합니다. 따라서 어떤 클래스든 implements 할 수 있습니다.

```dart
class MockSpaceship implements Spacecraft {
  // ···
}
```

암시적 인터페이스 또는 명시적 인터페이스 키워드에 대해 자세히 알아보세요.

구체적인 클래스에 의해 확장(또는 구현)될 추상 클래스를 만들 수 있습니다. 추상 클래스에는 추상 메서드(빈 본문 포함)가 포함될 수 있습니다.

```dart
abstract class Describable {
  void describe();

  void describeWithEmphasis() {
    print('=========');
    describe();
    print('=========');
  }
}
```

Describable을 확장하는 모든 클래스는 확장자의 describe() 구현을 호출하는 describeWithEmphasis()() 메서드를 가집니다.

추상 클래스 및 메서드에 대해 자세히 알아보세요.

## 비동기 Async

콜백 지옥을 피하고 async 및 await를 사용하여 코드를 훨씬 더 읽기 쉽게 만드세요.

```dart
const oneSecond = Duration(seconds: 1);

// ···

Future<void> printWithDelay(String message) async {
  await Future.delayed(oneSecond);
  print(message);
}
```

위의 방법은 다음과 동일합니다.

```dart
Future<void> printWithDelay(String message) {
  return Future.delayed(oneSecond).then((_) {
    print(message);
  });
}
```

다음 예제에서 볼 수 있듯이, async 및 await는 비동기 코드를 쉽게 읽을 수 있도록 도와줍니다.

```dart
Future<void> createDescriptions(Iterable<String> objects) async {
  for (final object in objects) {
    try {
      var file = File('$object.txt');

      if (await file.exists()) {
        var modified = await file.lastModified();
        print('File for $object already exists. It was modified on $modified.');

        continue;
      }

      await file.create();

      await file.writeAsString('Start describing $object in this file.');
    } on IOException catch (e) {
      print('Cannot create description for $object: $e');
    }
  }
}
```

스트림을 구축하는 훌륭하고 읽기 쉬운 방법을 제공하는 async\*를 사용할 수도 있습니다.

```dart
Stream<String> report(Spacecraft craft, Iterable<String> objects) async* {

  for (final object in objects) {
    await Future.delayed(oneSecond);
    yield '${craft.name} flies by $object';
  }
}
```

비동기 함수, Future, Stream 및 비동기 루프(대기)를 포함한 비동기 지원에 대해 자세히 알아보세요.

## 예외 Exceptions

예외를 발생시키려면 throw를 사용하세요.

```dart
if (astronauts == 0) {
  throw StateError('No astronauts.');
}
```

예외를 포착하려면 on 또는 try 문과 함께 catch(또는 둘 다)를 사용합니다.

```dart
Future<void> describeFlybyObjects(List<String> flybyObjects) async {
  try {
    for (final object in flybyObjects) {
      var description = await File('$object.txt').readAsString();

      print(description);
    }
  } on IOException catch (e) {
    print('Could not describe object: $e');
  } finally {
    flybyObjects.clear();
  }
}
```

위의 코드는 비동기식입니다. try는 동기 코드와 비동기 함수의 코드 모두에서 작동합니다.

스택 추적, rethrow, Error와 Exception의 차이점을 포함한 예외에 대해 자세히 알아보세요.

## 주요 컨셉들 Important concepts

Dart 언어에 대해 학습하면서 다음과 같은 사실과 개념들을 염두에 두세요.

- 변수에 넣을 수 있는 모든 것은 객체이며, 모든 객체는 클래스의 인스턴스입니다. 심지어 숫자, 함수, null 역시 객체입니다. null을 제외하고(이상없는 null 안전성을 활성화한 경우), 모든 객체는 Object 클래스에서 상속됩니다.

버전 노트:

null 안전성은 Dart 2.12에서 도입되었습니다. null 안전성을 사용하려면 언어 버전이 2.12 이상이어야 합니다.

- Dart는 타입에 엄격하지만 Dart는 타입을 추론할 수 있으므로 타입 선언은 선택 사항입니다. var number = 101에서 number는 int 타입으로 추론됩니다.

- null 안전성을 활성화하면 변수에 null을 포함할 수 있다고 명시하지 않는 한 변수에 null이 포함될 수 없습니다. 해당 타입 끝에 물음표(?)를 추가하여 변수를 null 허용으로 만들 수 있습니다. 예를 들어 int? 타입의 변수는 정수일 수도 있고 null일 수도 있습니다. 표현식이 결코 null로 평가되지 않는다는 것을 알고 있지만 Dart가 이에 동의하지 않는 경우, !을 추가하여 null이 아니라고 주장할 수 있습니다(표현식이 null인 경우 예외를 발생시킵니다).  예: int x= nullableButNotNullInt!

- 명시적으로 어떤 타입이든 허용되게 하고 싶다면 Object?(null 안전성을 활성화한 경우), Object 또는 dynamic(런타임까지 타입 검사를 연기해야 ​​하는 경우)라는 특수한 타입을 사용하세요.

- Dart는 List<int>(정수 리스트) 또는 List<Object>(모든 타입의 리스트)와 같은 제네릭 타입을 지원합니다.

- Dart는 최상위 함수(예: main())뿐만 아니라 클래스나 객체에 연결된 함수(각각 정적 메서드와 인스턴스 메서드)도 지원합니다. 함수 내에서 함수(중첩 또는 로컬 함수)를 생성할 수도 있습니다.

- 마찬가지로 Dart는 클래스나 객체에　연결된 변수(정적 및 인스턴스 변수)뿐만 아니라 최상위 변수도 지원합니다.　인스턴스 변수는 필드 또는 속성이라고도 합니다.

- Java와 달리 Dart에는 public, protected 및 private 키워드가 없습니다. 식별자가 밑줄(\_)로 시작하면 해당 라이브러리의 전용 식별자입니다. 자세한 내용은 라이브러리 및 가져오기를 참조하세요.

- 식별자(변수명, 함수명, 클래스명)는 문자나 밑줄(\_)로 시작할 수 있으며 그 뒤에 문자, 밑줄과 ​​숫자의 조합이 올 수 있습니다.

- Dart에는 표현식(런타임 값을 가지는)과 명령문(런타임 값을 가지지 않는)이 모두 있습니다. 예를 들어 조건식(도 표현식) Condition ? expr1 : expr2의 값은 expr1 또는 expr2입니다. 이를 값이 없는 if-else 문과 비교해 보세요. 명령문에는 하나 이상의 표현식이 포함되는 경우가 많지만 표현식에 명령문이 직접 포함될 수는 없습니다.

- Dart 도구는 경고와 오류라는 두 가지 종류의 문제를 보고할 수 있습니다. 경고는 코드가 작동하지 않을 수 있음을 나타내는 것일 뿐 프로그램 실행을 방해하지는 않습니다. 오류는 컴파일 타임 또는 런타임일 수 있습니다. 컴파일 타임 오류로 인해 코드가 전혀 실행되지 않습니다. 런타임 오류로 인해 코드가 실행되는 동안 예외가 발생합니다.

## 추가 리소스 Additional resources

핵심 라이브러리 문서와 Dart API 참조에서 더 많은 문서와 코드 샘플을 찾을 수 있습니다. 이 사이트의 코드는 Dart 스타일 가이드의 규칙을 따릅니다.
