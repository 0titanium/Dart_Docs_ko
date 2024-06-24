# 에러 핸들링 Error handling

## 예외 Exceptions

Dart는 예외를 발생시키고 잡아냅니다. 예외는 예기치않은 일이 발생했을 때 나타내는 오류입니다. 만약 예외가 잡히지 않으면, 예외를 발생시킨 아이솔레이트가 지연되고 일시적으로 아이솔레이트와 프로그램은 종료됩니다.

Java와 달리 Dart의 모든 예외는 확인되지않은 예외입니다. 메서드는 어떤 예외를 발생시킬지 선언하지 않으며 예외를 잡아낼 필요가 없습니다.

Dart는 Exception과 Error 타입뿐만 아니라 다른 수많은 미리 선언된 하위타입을 제공합니다. 물론 자신만의 예외를 만들 수도 있습니다. 그러나 Dart 프로그램은 Exception과 Error 객체뿐만 아니라 null이 아닌 어떤 객체도 예외로 발생시킬 수 있습니다.

## 던지기 Throw

다음은 예외를 던지거나 발생시키는 예시입니다:

```dart
throw FormatException('Expected at least 1 section');
```

임의의 객체를 던질 수도 있습니다:

```dart
throw 'Out of llamas!';
```

```
노트

프로덕션 퀄리티의 코드는 대개 Error나 Exception을 구현한 타입을 던집니다.
```

예외를 던지는 것은 표현식이기 때문에, => 문 뿐만아니라 표현식을 허용하는 어디에서든 예외를 던질 수 있습니다.

```dart
void distanceTo(Point other) => throw UnimplementedError();
```

## 잡기 Catch

예외를 잡거나 포착하면 예외 전파가 중지됩니다(예외를 다시 던지지 않는 한). 예외를 잡는 것은 이를 처리할 기회가 주어집니다.

```dart
try {
  breedMoreLlamas();
} on OutOfLlamasException {
  buyMoreLlamas();
}
```

두 가지 이상의 예외 타입을 발생시킬 수 있는 코드를 처리하려면 여러개의 catch 절을 지정할 수 있습니다. 던져진 객체의 타입과 일치하는 첫 번째 catch 절이 예외를 처리합니다. catch 절이 타입을 지정하지 않으면 해당 절은 던져진 객체의 모든 유형을 처리할 수 있습니다.

```dart
try {
  breedMoreLlamas();
} on OutOfLlamasException {
  // 특정 예외
  buyMoreLlamas();
} on Exception catch (e) {
  // 그 밖에 예외인 것
  print('Unknown exception: $e');
} catch (e) {
  // 지정된 타입 없음, 모두 처리
  print('Something really unknown: $e');
}
```

위의 코드에서 볼 수 있듯이 on이나 cath, 또는 둘 다 사용할 수 있습니다. 예외 타입을 지정해야하는 경우 on을 사용합니다. 예외 처리에 예외 객체가 필요한 경우 catch를 사용하세요.

catch()에 하나 또는 두 개의 매개변수를 지정할 수 있습니다. 첫 번째는 발생한 예외이고 두 번째는 스택 추적(StackTrace 객체)입니다.

```dart
try {
  // ···
} on Exception catch (e) {
  print('Exception details:\n $e');
} catch (e, s) {
  print('Exception details:\n $e');
  print('Stack trace:\n $s');
}
```

예외의 전파를 허용하면서 부분적으로 예외를 처리하려면 rethrow 키워드를 사용하세요.

```dart
void misbehave() {
  try {
    dynamic foo = true;
    print(foo++); // 런타임 오류
  } catch (e) {
    print('misbehave() partially handled ${e.runtimeType}.');
    rethrow; // 호출자가 예외를 볼 수 있도록 허용
  }
}

void main() {
  try {
    misbehave();
  } catch (e) {
    print('main() finished handling ${e.runtimeType}.');
  }
}
```

## Finally

예외 발생 여부에 관계없이 일부 코드가 실행되도록 하려면 finally 절을 사용하세요. 예외와 일치하는 catch 절이 없으면 finally 절이 실행된 후 예외가 전파됩니다.

```dart
try {
  breedMoreLlamas();
} finally {
  // 예외가 발생해도 항상 정리.
  cleanLlamaStalls();
}
```

finally 절은 일치하는 catch절 이후에 실행됩니다.

```dart
try {
  breedMoreLlamas();
} catch (e) {
  print('Error: $e'); // 예외 먼저 처리.
} finally {
  cleanLlamaStalls(); // 그런 다음 정리.
}
```

## Assert

개발 중에는 assert 문(assert(<condition>, <ionalMessage>))을 사용하십시오. 부울 조건이 거짓인 경우 정상적인 실행을 방해합니다.

```dart
// 변수에 null이 아닌 값이 있는지 확인.
assert(text != null);

// 값이 100보다 작은지 확인.
assert(number < 100);

// https URL인지 확인.
assert(urlString.startsWith('https'));
```

어설션에 메시지를 첨부하려면 어설션할 두 번째 인자로 문자열을 추가합니다(선택적으로 후행 쉼표 포함).

```dart
assert(urlString.startsWith('https'),
    'URL ($urlString) should start with "https".');
```

어설션할 첫 번째 인자는 부울 값으로 확인되는 표현식일 수 있습니다. 표현식의 값이 true이면 어설션이 성공하고 실행이 계속됩니다. false인 경우 어설션이 실패하고 예외(AssertionError)가 발생합니다.

어설션은 정확히 언제 작동할까요? 이는 사용 중인 도구와 프레임워크에 따라 다릅니다.

- Flutter는 디버그 모드에서 어설션을 활성화합니다.

- webdev 서비스와 같은 개발 전용 도구는 일반적으로 기본적으로 어설션을 활성화합니다.

- dart run 및 dart compile js와 같은 일부 도구는 --enable-asserts 명령줄 플래그를 통해 어설션을 지원합니다.

프로덕션 코드에서는 어설션이 무시되고 어설션할 인수가 평가되지 않습니다.