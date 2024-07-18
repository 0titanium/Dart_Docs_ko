# 비동기 지원 Asynchrony support

Dart 라이브러리는 Future 또는 Stream 객체를 반환하는 함수들로 가득 차 있습니다. 이러한 함수들은 비동기 함수입니다: 시간이 많이 소요될 수 있는 작업(I/O 같은)을 설정한 후 해당 작업이 완료될 때까지 기다리지 않고 반환합니다.

async 및 await 키워드는 비동기 프로그래밍을 지원하여, 동기 코드와 유사한 형태로 비동기 코드를 작성할 수 있게 합니다.

## Future 처리하기 Handling Futures

완료된 Future의 결과가 필요할 때 두 가지 옵션이 있습니다:

- 여기와 비동기 프로그래밍 튜토리얼에서 설명한 대로 async와 await를 사용합니다.

- dart:async 문서에서 설명한 대로 Future API를 사용합니다.

async와 await를 사용하는 코드는 비동기적이지만 동기 코드와 매우 유사해 보입니다. 예를 들어, 다음 코드는 비동기 함수의 결과를 기다리기 위해 await를 사용하는 코드입니다:

```dart
await lookUpVersion();
```

await를 사용하려면 코드가 async 함수 안에 있어야 합니다. async로 표시된 함수입니다:

```dart
Future<void> checkVersion() async {
  var version = await lookUpVersion();
  // 버전으로 무언가를 수행합니다
}
```

```
노트

비록 async 함수가 시간이 많이 소요되는 작업을 수행할 수 있지만, 해당 작업을 기다리지는 않습니다. 대신 async 함수는 첫 번째 await 표현식을 만날 때까지 실행됩니다. 그런 다음 Future 객체를 반환하고, await 표현식이 완료된 후에만 실행을 재개합니다.
```

await를 사용하는 코드에서 오류를 처리하고 정리하려면 try, catch 및 finally를 사용하세요:

```dart
try {
  version = await lookUpVersion();
} catch (e) {
  // 버전을 조회할 수 없는 경우에 반응합니다
}
```

async 함수에서 await를 여러 번 사용할 수 있습니다. 예를 들어, 다음 코드는 세 번 함수의 결과를 기다립니다:

```dart
var entrypoint = await findEntryPoint();
var exitCode = await runExecutable(entrypoint, args);
await flushThenExit(exitCode);
```

await 표현식에서 표현식의 값은 보통 Future입니다. 그렇지 않은 경우 해당 값은 자동으로 Future로 래핑됩니다. 이 Future 객체는 객체를 반환할 것을 약속하는 것입니다. await 표현식의 값은 반환된 객체입니다. await 표현식은 해당 객체가 사용 가능할 때까지 실행을 일시 중지시킵니다.

await를 사용할 때 컴파일 타임 오류가 발생하면 await가 async 함수 내에 있는지 확인하십시오. 예를 들어, 앱의 main() 함수에서 await를 사용하려면 main()의 본문이 async로 표시되어야 합니다:

```dart
void main() async {
  checkVersion();
  print('In main: version is ${await lookUpVersion()}');
}
```

```
노트

위 예제는 결과를 기다리지 않고 async 함수(checkVersion())를 사용하는데, 이는 함수가 실행을 마쳤다고 가정하는 코드가 있을 경우 문제를 일으킬 수 있습니다. 이 문제를 피하려면 unawaited_futures 린터 규칙을 사용하십시오.
```

Future, async 및 await를 사용하는 상호적인 소개는 비동기 프로그래밍 튜토리얼을 참조하십시오.

## async 함수 선언하기 Declaring async functions

async 함수는 본문이 async 수정자로 표시된 함수입니다.

함수에 async 키워드를 추가하면 해당 함수는 Future를 반환합니다. 예를 들어, 문자열을 반환하는 동기 함수를 고려해 보십시오:

```dart
String lookUpVersion() => '1.0.0';
```

이 함수를 async 함수로 변경하면(예를 들어 future 구현이 시간이 많이 소요되기 때문에) 반환 값은 Future가 됩니다:

```dart
Future<String> lookUpVersion() async => '1.0.0';
```

함수 본문이 Future API를 사용할 필요는 없습니다. 필요하다면 Dart가 Future 객체를 생성합니다. 함수가 유용한 값을 반환하지 않는 경우 반환 타입을 Future<void>로 만드십시오.

Future, async 및 await를 사용하는 상호적인 소개는 비동기 프로그래밍 튜토리얼을 참조하십시오.

## Stream 처리하기 Handling Streams

Stream에서 값을 가져와야 할 때 두 가지 옵션이 있습니다:

- async와 비동기 for 루프(await for)를 사용합니다.
- dart:async 문서에서 설명한 대로 Stream API를 사용합니다.

```
노트

await for을 사용하기 전에 코드가 더 명확해지고 스트림의 모든 결과를 정말로 기다리고 싶은지 확인하십시오. 예를 들어, UI 이벤트 리스너의 경우에는 일반적으로 await for를 사용하지 않아야 합니다. UI 프레임워크는 끝없는 이벤트의 스트림을 전송하기 때문입니다.
```

비동기 for 루프는 다음 형식을 가집니다:

```dart
await for (varOrType identifier in expression) {
  // 스트림이 값을 방출할 때마다 실행됩니다.
}
```

expression의 값은 Stream 유형이어야 합니다. 실행은 다음과 같이 진행됩니다:

1. 스트림이 값을 방출할 때까지 기다립니다.

2. for 루프의 본문을, 변수가 방출된 값으로 설정된 상태로 실행합니다.

3. 스트림이 닫힐 때까지 1과 2를 반복합니다.

스트림 수신을 중지하려면 break 또는 return 문을 사용할 수 있습니다. 이는 for 루프에서 벗어나 스트림 구독을 취소합니다.

비동기 for 루프를 구현할 때 컴파일 타임 오류가 발생하면 await for가 async 함수 내에 있는지 확인하십시오. 예를 들어, 앱의 main() 함수에서 비동기 for 루프를 사용하려면 main()의 본문이 async로 표시되어야 합니다:

```dart
void main() async {
  // ...
  await for (final request in requestServer) {
    handleRequest(request);
  }
  // ...
}
```
Dart의 비동기 프로그래밍 지원에 대한 자세한 내용은 dart:async 라이브러리 문서를 참조하십시오.