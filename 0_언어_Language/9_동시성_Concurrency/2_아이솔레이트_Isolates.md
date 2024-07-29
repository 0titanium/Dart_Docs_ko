# 아이솔레이트 Isolates

이 페이지에서는 Isolate API를 사용하여 아이솔레이트를 구현하는 몇 가지 예를 설명합니다.

애플리케이션이 다른 계산을 일시적으로 차단할 만큼 큰 계산을 처리할 때마다 아이솔레이트를 사용해야 합니다. 가장 일반적인 예는 Flutter 애플리케이션에서 UI가 응답하지 않을 수 있는 대규모 계산을 수행해야 하는 경우입니다.

아이솔레이트를 사용해야 하는 경우에 대한 규칙은 없지만 다음은 아이솔레이트가 유용할 수 있는 몇 가지 상황입니다.

- 매우 큰 JSON blob(Binary Large Object)을 파싱하고 디코딩할 때.
- 사진, 오디오, 비디오를 처리하고 압축할 때.
- 오디오 및 비디오 파일 변환할 때.
- 대규모 리스트나 파일 시스템 내에서 복잡한 검색 및 필터링을 수행할 때.
- 데이터베이스와의 통신과 같은 I/O를 수행할 때.
- 대량의 네트워크 요청을 처리할 때.

## 간단한 작업자 아이솔레이트 구현 Implementing a simple worker isolate

이 예제에서는 간단한 작업자 아이솔레이트를 생성하는 메인 아이솔레이트를 구현합니다. Isolate.run()은 작업자 아이솔레이트 설정 및 관리 단계를 단순화합니다.

1. 아이솔레이트를 생성(시작 및 생성)합니다.
2. 생성된 아이솔레이트에 대해 함수를 실행합니다.
3. 결과를 캡처합니다.
4. 결과를 메인 아이솔레이트로 반환합니다.
5. 작업이 완료되면 아이솔레이트를 종료합니다.
6. 메인 아이솔레이트로 예외 및 오류를 확인, 캡처 및 발생시킵니다.

```
플러터 노트

Flutter를 사용하는 경우 Isolate.run() 대신 Flutter의 compute 함수를 사용할 수 있습니다.
```

## 새로운 아이솔레이트에서 기존 메서드 실행 Running an existing method in a new isolate


1. run()을 호출하여 main()이 결과를 기다리는 동안 메인 아이솔레이트에서 직접 새 아이솔레이트(백그라운드 작업자)를 생성합니다.

```dart
const String filename = 'with_keys.json';

void main() async {
  // 데이터 읽기.
  final jsonData = await Isolate.run(_readAndParseJson);

  // 해당 데이터 사용.
  print('Number of JSON keys: ${jsonData.length}');
}
```

2. 작업자 아이솔레이트에 실행하려는 함수를 첫 번째 인자로 전달합니다. 이 예에서는 기존 함수 _readAndParseJson()입니다.

```dart
Future<Map<String, dynamic>> _readAndParseJson() async {
  final fileData = await File(filename).readAsString();
  final jsonData = jsonDecode(fileData) as Map<String, dynamic>;
  return jsonData;
}
```

3. Isolate.run()은 _readAndParseJson()이 반환한 결과를 가져와 해당 값을 메인 아이솔레이트로 다시 보내 작업자 아이솔레이트를 종료합니다.

4. 작업자 아이솔레이트는 결과를 보관하는 메모리를 메인 아이솔레이트로 전송합니다. 이것은 데이터를 복사하지 않습니다. 작업자 아이솔레이트는 객체 전송이 허용되는지 보장하기 위해 확인 과정을 수행합니다.

_readAndParseJson()은 메인 아이솔레이트에서 직접 쉽게 실행할 수 있는 기존 비동기 함수입니다. 대신 Isolate.run()을 사용하여 실행하면 동시성이 활성화됩니다. 작업자 아이솔레이트는 _readAndParseJson()의 계산을 완전히 추상화합니다. 메인 아이솔레이트를 차단하지 않고 완료할 수 있습니다.

Isolate.run()의 결과는 항상 Future입니다. 메인 아이솔레이트의 코드가 계속 실행되기 때문입니다. 작업자 아이솔레이트가 실행하는 계산이 동기식인지 비동기식인지 여부는 메인 아이솔레이트에 영향을 주지 않습니다. 어느 쪽이든 동시에 실행되기 때문입니다.

전체 프로그램을 보려면 send_and_receive.dart 샘플을 확인하세요.

## 아이솔레이트를 사용해 클로저 보내기 Sending closures with isolates

또한 메인 아이솔레이트에서 직접 함수 리터럴 또는 클로저를 사용한 run()을 사용하여 간단한 작업자 아이솔레이트를 만들 수도 있습니다.

```dart
const String filename = 'with_keys.json';

void main() async {
  // 데이터 읽기.
  final jsonData = await Isolate.run(() async {
    final fileData = await File(filename).readAsString();
    final jsonData = jsonDecode(fileData) as Map<String, dynamic>;
    return jsonData;
  });

  // 해당 데이터 사용.
  print('Number of JSON keys: ${jsonData.length}');
}
```

이 예제는 이전 예제와 동일하게 수행됩니다. 새로운 아이솔레이트가 생성되고, 무언가를 계산하고, 결과를 다시 보냅니다.

그러나 이제 아이솔레이트는 클로저를 보냅니다. 클로저는 작동 방식과 코드에 작성되는 방식 모두에서 일반적인 명명된 함수보다 덜 제한적입니다. 이 예에서 Isolate.run()은 로컬 코드처럼 보이는 것을 동시에 실행합니다. 그런 의미에서 run()이 "병렬 실행"에 대한 제어 흐름 연산자처럼 작동한다고 상상할 수 있습니다.

## 포트가 있는 아이솔레이트 간에 여러 메시지 보내기 Sending multiple messages between isolates with ports

단기 아이솔레이트는 사용하기 편리하지만 새 아이솔레이트를 생성하고 한 아이솔레이트에서 다른 아이솔레이트로 객체를 복사하려면 성능 오버헤드가 필요합니다. 코드가 Isolate.run을 사용하여 동일한 계산을 반복적으로 실행하는 데 의존하는 경우 즉시 종료되지 않는 장기 아이솔레이트를 대신 생성하여 성능을 향상시킬 수 있습니다.

이를 위해 Isolate.run이 추상화하는 로우 레벨 아이솔레이트 API 중 일부를 사용할 수 있습니다.

- Isolate.spawn() 및 Isolate.exit()
- ReceiverPort 및 SendPort
- SendPort.send() 메서드

이 섹션에서는 새로 생성된 아이솔레이트와 메인 아이솔레이트 간의 양방향 통신을 설정하는 데 필요한 단계를 설명합니다. 첫 번째 예인 Basic ports에서는 프로세스를 하이 레벨에서 소개합니다. 두 번째 예인 Roubust ports는 첫 번째 예에 점점 더 실용적이고 실제적인 기능을 추가합니다.

## ReceivePort 와 SendPort ReceivePort and SendPort

아이솔레이트 간 장기 통신을 설정하려면 두 클래스(아이솔레이트 외에), ReceiverPort 및 SendPort가 필요합니다. 이러한 포트는 아이솔레이트가 서로 통신할 수 있는 유일한 방법입니다.

ReceivePort는 다른 아이솔레이트에서 전송된 메시지를 처리하는 객체입니다. 해당 메시지는 SendPort를 통해 전송됩니다.

```
노트

SendPort 객체는 정확히 하나의 ReceivePort와 연결되지만 단일 ReceivePort에는 여러 개의 SendPort가 있을 수 있습니다. ReceiverPort를 생성하면 자체적으로 SendPort가 생성됩니다. 기존 ReceivePort에 메시지를 보낼 수 있는 추가 SendPort를 만들 수 있습니다.
```

포트는 Stream 객체와 유사하게 동작합니다(실제로 수신 포트는 Stream을 구현합니다!) SendPort 및 ReceiverPort는 각각 Stream의 StreamController 및 리스너와 같다고 생각할 수 있습니다. SendPort는 SendPort.send() 메서드를 사용하여 메시지를 "추가"하고 해당 메시지는 리스너(이 경우에는 ReceiverPort)에 의해 처리되므로 StreamController와 같습니다. 그러면 ReceiverPort는 수신한 메시지를 제공한 콜백에 인자로 전달하여 처리합니다.

## 포트 설정 Setting up ports

새로 생성된 아이솔레이트에는 Isolate.spawn 호출을 통해 수신한 정보만 있습니다. 초기 생성 이후 생성된 아이솔레이트와 계속 통신하기 위해 메인 아이솔레이트가 필요한 경우 생성된 아이솔레이트가 메인 아이솔레이트에 메시지를 보낼 수 있는 통신 채널을 설정해야 합니다. 아이솔레이트는 메시지 전달을 통해서만 통신할 수 있습니다. 서로의 메모리 내부를 '볼' 수 없기 때문에 '아이솔레이트'라는 이름이 유래됐습니다.

이 양방향 통신을 설정하려면 먼저 메인 아이솔레이트에 ReceiverPort를 만든 다음 Isolate.spawn을 사용하여 생성할 때 해당 SendPort를 새 아이솔레이트에 인자로 전달합니다. 그런 다음 새 아이솔레이트는 자체 ReceiverPort를 생성하고 메인 아이솔레이트가 전달한 SendPort에서 SendPort를 다시 보냅니다. 메인 아이솔레이트는 이 SendPort를 수신하며 이제 양쪽 모두 메시지를 보내고 받을 수 있는 열린 채널을 갖게 됩니다.

```
노트

이 섹션의 다이어그램은 하이 레벨이며 아이솔레이트를 위해 포트를 사용하는 개념을 전달하기 위한 것입니다. 실제 구현에는 조금 더 많은 코드가 필요하며, 이에 대해서는 이 페이지의 뒷부분에서 확인할 수 있습니다.
```

1. 메인 아이솔레이트에 ReceiverPort를 만듭니다. SendPort는 ReceiverPort의 속성으로 자동 생성됩니다.
2. Isolate.spawn()을 사용하여 작업자 아이솔레이트를 생성합니다.
3. ReceiverPort.sendPort에 대한 참조를 작업자 아이솔레이트에 대한 첫 번째 메시지로 전달합니다.
4. 작업자 아이솔레이트에 또 다른 새 수신 포트를 만듭니다.
5. 작업자 아이솔레이트의 receivePort.sendPort에 대한 참조를 첫 번째 메시지로 메인 아이솔레이트에 다시 전달합니다.

포트를 생성하고 통신을 설정하는 것과 함께 메시지를 받을 때 수행할 작업을 포트에 알려주어야 합니다. 이는 각각의 ReceiverPort에서 listen 메서드를 사용하여 수행됩니다.

1. 작업자 아이솔레이트의 SendPort에 대한 메인 아이솔레이트의 참조를 통해 메시지를 보냅니다.
2. 작업자 아이솔레이트의 ReceiverPort에 있는 리스너를 통해 메시지를 수신하고 처리합니다. 이곳은 메인 아이솔레이트에서 옮기려는 계산이 실행되는 곳입니다.
3. 작업자 아이솔레이트의 참조를 통해 메인 아이솔레이트의 SendPort에 대한 반환 메시지를 보냅니다.
4. 메인 아이솔레이트의 ReceiverPort에 있는 리스너 통해 메시지를 수신합니다.

## Basic port 예제 Basic ports example

이 예에서는 메인 아이솔레이트와 양방향 통신을 통해 수명이 긴 작업자 아이솔레이트를 설정하는 방법을 보여줍니다. 코드에서는 JSON 텍스트를 새 아이솔레이트로 보내는 예제를 사용합니다. 여기서 JSON은 메인 아이솔레이트로 다시 전송되기 전에 파싱 및 디코딩됩니다.

```
경고

이 예는 시간이 지남에 따라 여러 메시지를 보내고 받을 수 있는 새로운 아이솔레이트를 생성하는 데 필요한 최소한의 정보를 가르치기 위한 것입니다.

오류 처리, 포트 종료, 메시지 순서 지정 등 프로덕션 소프트웨어에서 예상되는 중요한 기능은 다루지 않습니다.

다음 섹션의 Robust port 예제에서는 이 기능을 다루고 이 기능 없이 발생할 수 있는 몇 가지 문제에 대해 설명합니다.
```

## 1단계: 작업자 클래스 정의하기 Step 1: Define the worker class

먼저 백그라운드 작업자 아이솔레이트를 위한 클래스를 만듭니다. 이 클래스에는 다음 작업에 필요한 모든 기능이 포함되어 있습니다.

- 아이솔레이트를 생성합니다.
- 해당 아이솔레이트에 메시지를 보냅니다.
- 일부 JSON을 디코딩하는 아이솔레이트를 가집니다.
- 디코딩된 JSON을 메인 아이솔레이트로 다시 보냅니다.


클래스는 두 가지 퍼블릭 메서드를 노출합니다. 하나는 작업자 아이솔레이트를 생성하는 메서드이고 다른 하나는 해당 작업자 아이솔레이트로 메시지 보내기를 처리하는 메서드입니다.

이 예제의 나머지 섹션에서는 클래스 메서드를 하나씩 채우는 방법을 보여줍니다.

```dart
class Worker {
  Future<void> spawn() async {
    // TODO: 작업자 아이솔레이트를 생성하는 기능을 추가합니다.
  }

  void _handleResponsesFromIsolate(dynamic message) {
    // TODO: 작업자 아이솔레이트에서 다시 전송된 메시지를 처리합니다.
  }

  static void _startRemoteIsolate(SendPort port) {
    // TODO: 작업자 아이솔레이트에서 실행되어야 하는 코드를 정의합니다.
  }

  Future<void> parseJson(String message) async {
    // TODO: 작업자 아이솔레이트에 메시지를 보내는 데 사용할 수 있는 
    // 공용 메서드를 정의합니다.
  }
}
```

## 2단계: 작업자 아이솔레이트 생성 Step 2: Spawn a worker isolate

Worker.spawn 메서드는 작업자 아이솔레이트를 생성하고 메시지를 주고받을 수 있는지 확인하기 위한 코드를 그룹화하는 곳입니다.

- 먼저, ReceiverPort를 만듭니다. 이렇게 하면 메인 아이솔레이트가 새로 생성된 작업자 아이솔레이트에서 전송된 메시지를 수신할 수 있습니다.

- 다음으로 작업자 아이솔레이트가 다시 보낼 메시지를 처리하기 위해 수신 포트에 리스너를 추가합니다. 리스너에 전달된 콜백인 _handleResponsesFromIsolate는 4단계에서 다룹니다.

- 마지막으로 Isolate.spawn을 사용하여 작업자 아이솔레이트를 생성합니다. 작업자 아이솔레이트에서 실행될 함수(3단계에서 설명)와 수신 포트의 sendPort 속성이라는 두 가지 인자가 필요합니다.

```dart
Future<void> spawn() async {
  final receivePort = ReceivePort();
  receivePort.listen(_handleResponsesFromIsolate);
  await Isolate.spawn(_startRemoteIsolate, receivePort.sendPort);
}
```

receivePort.sendPort 인자는 작업자 아이솔레이트에서 호출될 때 콜백(_startRemoteIsolate)에 인자로 전달됩니다. 이는 작업자 아이솔레이트가 메인 아이솔레이트로 다시 메시지를 보낼 수 있는 방법을 갖도록 하는 첫 번째 단계입니다.

## 3단계: 작업자 아이솔레이트에서 코드 실행 Step 3: Execute code on the worker isolate

이 단계에서는 작업자 아이솔레이트가 생성될 때 실행되도록 작업자 아이솔레이트로 전달되는 _startRemoteIsolate 메서드를 정의합니다. 이 방법은 작업자 아이솔레이트의 "메인" 방법과 같습니다.

- 먼저 또 다른 새 ReceiverPort를 만듭니다. 이 포트는 메인 아이솔레이트로부터 future 메시지를 수신합니다.

- 그런 다음 해당 포트의 SendPort를 메인 아이솔레이트로 다시 보냅니다.

- 마지막으로 새 ReceiverPort에 리스너를 추가합니다. 이 리스너는 메인 아이솔레이트가 작업자 아이솔레이트로 보내는 메시지를 처리합니다.

```dart
static void _startRemoteIsolate(SendPort port) {
  final receivePort = ReceivePort();
  port.send(receivePort.sendPort);

  receivePort.listen((dynamic message) async {
    if (message is String) {
      final transformed = jsonDecode(message);
      port.send(transformed);
    }
  });
}
```

작업자의 ReceiverPort에 있는 리스너는 메인 아이솔레이트에서 전달된 JSON을 디코딩한 다음 디코딩된 JSON을 메인 아이솔레이트로 다시 보냅니다.

이 리스너는 메인 아이솔레이트에서 작업자 아이솔레이트로 전송된 메시지의 진입점입니다. 작업자 아이솔레이트에 나중에 실행할 코드를 알려줄 수 있는 유일한 기회입니다.

## 4단계: 메인 아이솔레이트에서 메시지 처리 Step 4: Handle messages on the main isolate

마지막으로 작업자 아이솔레이트에서 다시 메인 아이솔레이트로 보낸 메시지를 처리하는 방법을 메인 아이솔레이트에 알려야 합니다. 이렇게 하려면 _handleResponsesFromIsolate 메서드를 채워야 합니다. 2단계에 설명된 대로 이 메서드는 receivePort.listen 메서드에 전달된다는 점을 기억하세요.

```dart
Future<void> spawn() async {
  final receivePort = ReceivePort();
  receivePort.listen(_handleResponsesFromIsolate);
  await Isolate.spawn(_startRemoteIsolate, receivePort.sendPort);
}
```

또한 3단계에서 메인 아이솔레이트로 SendPort를 다시 보냈다는 점을 기억하세요. 이 메서드는 해당 SendPort 수신을 처리할 뿐만 아니라 future 메시지(JSON으로 디코딩됨)도 처리합니다.

- 먼저 메시지가 SendPort인지 확인하세요. 그렇다면 해당 포트를 클래스의 _sendPort 속성에 할당하여 나중에 메시지를 보내는 데 사용할 수 있습니다.

- 다음으로 메시지가 디코딩된 JSON의 예상 타입인 Map<String, Dynamic> 타입인지 확인합니다. 그렇다면 애플리케이션별 로직을 사용하여 해당 메시지를 처리하세요. 이 예에서는 메시지가 출력됩니다.

```dart
void _handleResponsesFromIsolate(dynamic message) {
  if (message is SendPort) {
    _sendPort = message;
    _isolateReady.complete();
  } else if (message is Map<String, dynamic>) {
    print(message);
  }
}
```

## 5단계: 아이솔레이트가 설정되었는지 확인하기 위해 완성자를 추가합니다. Step 5: Add a completer to ensure your isolate is set-up

클래스를 완료하려면 작업자 아이솔레이트에 메시지를 보내는 일을 담당하는 parsJson이라는 퍼블릭 메서드를 정의하세요. 또한 아이솔레이트가 완전히 설정되기 전에 메시지를 보낼 수 있는지 확인해야 합니다. 이를 처리하려면 Completer를 사용하세요.

- 먼저 Completer라는 클래스 수준 속성을 추가하고 이름을 _isolateReady로 지정합니다.

- 다음으로, 메시지가 SendPort인 경우 _handleResponsesFromIsolate 메서드(4단계에서 생성됨)의 완성자에 대한 Complete() 호출을 추가합니다.

- 마지막으로, parseJson 메서드에서 _sendPort.send를 추가하기 전에 await _isolateReady.future를 추가합니다. 이렇게 하면 작업자 아이솔레이트가 생성되어 SendPort를 메인 아이솔레이트로 다시 보낼 때까지 작업자 아이솔레이트로 메시지를 보낼 수 없습니다.

```dart
Future<void> parseJson(String message) async {
  await _isolateReady.future;
  _sendPort.send(message);
}
```

## 완성된 예 Complete example

```dart
import 'dart:async';
import 'dart:convert';
import 'dart:isolate';

void main() async {
  final worker = Worker();
  await worker.spawn();
  await worker.parseJson('{"key":"value"}');
}

class Worker {
  late SendPort _sendPort;
  final Completer<void> _isolateReady = Completer.sync();

  Future<void> spawn() async {
    final receivePort = ReceivePort();
    receivePort.listen(_handleResponsesFromIsolate);
    await Isolate.spawn(_startRemoteIsolate, receivePort.sendPort);
  }

  void _handleResponsesFromIsolate(dynamic message) {
    if (message is SendPort) {
      _sendPort = message;
      _isolateReady.complete();
    } else if (message is Map<String, dynamic>) {
      print(message);
    }
  }

  static void _startRemoteIsolate(SendPort port) {
    final receivePort = ReceivePort();
    port.send(receivePort.sendPort);

    receivePort.listen((dynamic message) async {
      if (message is String) {
        final transformed = jsonDecode(message);
        port.send(transformed);
      }
    });
  }

  Future<void> parseJson(String message) async {
    await _isolateReady.future;
    _sendPort.send(message);
  }
}
```

## Robust port의 예 Robust ports example

이전 예에서는 양방향 통신을 통해 수명이 긴 아이솔레이트를 설정하는 데 필요한 기본 구성 요소를 설명했습니다. 언급한 바와 같이, 이 예에는 오류 처리, 더 이상 사용하지 않을 때 포트를 닫는 기능, 일부 상황에서 메시지 순서 불일치와 같은 몇 가지 중요한 기능이 부족합니다.

이 예에서는 이러한 추가 기능 등을 포함하고 더 나은 디자인 패턴을 따르는 수명이 긴 작업자 아이솔레이트를 만들어 첫 번째 예의 정보를 확장합니다. 이 코드는 첫 번째 예제와 유사하지만 첫 번째 예제의 확장은 아닙니다.

```
노트

이 예에서는 이전 예에서 다룬 Isolate.spawn과 포트를 사용한 아이솔레이트 간의 통신 설정에 이미 익숙하다고 가정합니다.
```

## 1단계: 작업자 클래스 정의하기 Step 1: Define the worker class

먼저 백그라운드 작업자 아이솔레이트를 위한 클래스를 만듭니다. 이 클래스에는 다음 작업에 필요한 모든 기능이 포함되어 있습니다.

- 아이솔레이트를 생성합니다.
- 해당 아이솔레이트 대상에게 메시지를 보냅니다.
- 일부 JSON을 디코딩하는 아이솔레이트를 가집니다.
- 디코딩된 JSON을 메인 아이솔레이트로 다시 보냅니다.

클래스는 세 가지 퍼블릭 메서드를 노출합니다. 하나는 작업자 아이솔레이트를 생성하고, 다른 하나는 해당 작업자 아이솔레이트로 메시지 전송을 처리하며, 다른 하나는 더 이상 사용하지 않을 때 포트를 종료할 수 있습니다.

```dart
class Worker {
  final SendPort _commands;
  final ReceivePort _responses;

  Future<Object?> parseJson(String message) async {
    // TODO: 포트가 아직 열려 있는지 확인하세요.
    _commands.send(message);
  }

  static Future<Worker> spawn() async {
    // TODO: 생성된 아이솔레이트에 대한 연결을 사용하여 새 작업자 객체를 
    // 생성하는 기능을 추가합니다.
    throw UnimplementedError();
  }

  Worker._(this._responses, this._commands) {
    // TODO: 메인 아이솔레이트 수신 포트 리스너를 초기화합니다.
  }

  void _handleResponsesFromIsolate(dynamic message) {
    // TODO: 작업자 아이솔레이트에서 다시 보낸 메시지를 처리합니다.
  }

  static void _handleCommandsToIsolate(ReceivePort rp, SendPort sp) async {
    // TODO: 작업자 아이솔레이트에서 다시 보낸 메시지를 처리합니다.
  }

  static void _startRemoteIsolate(SendPort sp) {
    // TODO: 작업자 아이솔레이트의 포트를 초기화합니다.
  }
}
```

```
노트

이 예에서 SendPort 및 ReceiverPort 인스턴스는 메인 아이솔레이트와 관련하여 이름이 지정되는 네이밍 컨벤션의 모범 사례를 따릅니다. SendPort를 통해 메인 아이솔레이트에서 작업자 아이솔레이트로 전송되는 메시지를 명령이라고 하며, 메인 아이솔레이트로 다시 전송되는 메시지를 응답이라고 합니다.
```

## 2단계: Worker.spawn 메서드에서 RawReceivePort 생성 Step 2: Create a RawReceivePort in the Worker.spawn method

아이솔레이트를 생성하기 전에 더 로우 레벨의 ReceiverPort인 RawReceivePort를 생성해야 합니다. RawReceivePort를 사용하면 아이솔레이트 시작 논리와 아이솔레이트에 대한 메시지 전달을 처리하는 논리를 분리할 수 있으므로 선호되는 패턴입니다.

Worker.spawn 메서드에서:

- 먼저 RawReceivePort를 만듭니다. 이 ReceiverPort는 SendPort가 될 작업자 아이솔레이트로부터 초기 메시지를 수신하는 역할만 담당합니다.

- 다음으로, 아이솔레이트가 메시지를 수신할 준비가 되었음을 나타내는 Completer를 만듭니다. 이 작업이 완료되면 ReceiverPort 및 SendPort가 포함된 레코드가 반환됩니다.

- 다음으로 RawReceivePort.handler 속성을 정의합니다. 이 속성은 ReceiverPort.listener처럼 동작하는 Function?입니다. 이 포트에서 메시지가 수신되면 함수가 호출됩니다.

- 핸들러 함수 내에서 Connection.complete()를 호출합니다. 이 메서드는 인자로 ReceiverPort 및 SendPort가 있는 레코드를 예상합니다. SendPort는 작업자 아이솔레이트에서 보낸 초기 메시지이며, 다음 단계에서 _commands라는 클래스 수준 SendPort에 할당됩니다.

- 그런 다음, ReceiverPort.fromRawReceivePort 생성자를 사용하여 새 ReceiverPort를 생성하고 initPort를 전달합니다.

```dart
class Worker {
  final SendPort _commands;
  final ReceivePort _responses;

  static Future<Worker> spawn() async {
    // 수신 포트를 만들고 초기 메시지 핸들러를 추가합니다.
    final initPort = RawReceivePort();
    final connection = Completer<(ReceivePort, SendPort)>.sync();
    initPort.handler = (initialMessage) {
      final commandPort = initialMessage as SendPort;
      connection.complete((
        ReceivePort.fromRawReceivePort(initPort),
        commandPort,
      ));
    };
// ···
  }
```

RawReceivePort를 먼저 생성한 다음, receivePort를 생성하면 나중에 receiveport.listen에 새 콜백을 추가할 수 있습니다. 반대로, ReceiverPort를 바로 생성한다면 ReceiverPort는 BroadcastStream이 아닌 Stream을 구현하기 때문에 하나의 리스너만 추가할 수 있습니다.

효과적으로 이를 통해 통신 설정이 완료된 후 메시지 수신을 처리하는 논리에서 아이솔레이트 시작 논리를 분리할 수 있습니다. 이 이점은 다른 메서드의 논리가 커짐에 따라 더욱 분명해집니다.

## 3단계: Isolate.spawn을 사용하여 작업자 아이솔레이트 생성 Step 3: Spawn a worker isolate with Isolate.spawn

이 단계에서는 계속해서 Worker.spawn 메서드를 채웁니다. 아이솔레이트를 생성하고 이 클래스에서 Worker 인스턴스를 반환하는 데 필요한 코드를 추가합니다. 이 예에서 Isolate.spawn에 대한 호출은 try/catch 블록에 래핑되어 아이솔레이트가 시작되지 않으면 initPort가 닫히고 Worker 개체가 생성되지 않도록 합니다.

- 먼저 try/catch 블록에서 작업자 아이솔레이트를 생성하려고 시도합니다. 작업자 아이솔레이트 생성이 실패하면 이전 단계에서 생성된 수신 포트를 닫습니다. Isolate.spawn에 전달된 메서드는 이후 단계에서 다루겠습니다.

- 다음으로, Connection.future를 기다리고, 반환되는 레코드에서 송신 포트와 수신 포트를 구조 분해 할당합니다.

- 마지막으로 프라이빗 생성자를 호출하고 해당 완성자의 포트를 전달하여 Worker 인스턴스를 반환합니다.

```dart
class Worker {
  final SendPort _commands;
  final ReceivePort _responses;

  static Future<Worker> spawn() async {
    // 수신 포트를 만들고 초기 메시지 핸들러를 추가합니다.
    final initPort = RawReceivePort();
    final connection = Completer<(ReceivePort, SendPort)>.sync();
    initPort.handler = (initialMessage) {
      final commandPort = initialMessage as SendPort;
      connection.complete((
        ReceivePort.fromRawReceivePort(initPort),
        commandPort,
      ));
    };
    // 아이솔레이트를 생성합니다.
    try {
      await Isolate.spawn(_startRemoteIsolate, (initPort.sendPort));
    } on Object {
      initPort.close();
      rethrow;
    }

    final (ReceivePort receivePort, SendPort sendPort) =
        await connection.future;

    return Worker._(receivePort, sendPort);
  }
```

이전 예제와 비교하여 이 예제에서 Worker.spawn은 이 클래스에 대한 비동기 정적 생성자 역할을 하며 Worker 인스턴스를 생성하는 유일한 방법입니다. 이는 API를 단순화하여 Worker 인스턴스를 생성하는 코드를 더 깔끔하게 만듭니다.

## 4단계: 아이솔레이트 설정 프로세스 완료 Step 4: Complete the isolate setup process

이 단계에서는 기본 아이솔레이트 설정 프로세스를 완료합니다. 이는 이전 예와 거의 전적으로 연관되어 있으며 새로운 개념은 없습니다. 코드가 더 많은 메서드로 분할된다는 점에서 약간의 변경이 있습니다. 이는 이 예제의 나머지 부분을 통해 더 많은 기능을 추가할 수 있도록 설정하는 디자인 방식입니다. 아이솔레이트 설정의 기본 프로세스에 대한 자세한 내용은 기본 포트 예를 참조하세요.

먼저 Worker.spawn 메서드에서 반환되는 프라이빗 생성자를 만듭니다. 생성자 본문에서 메인 아이솔레이트에서 사용하는 수신 포트에 리스너를 추가하고 _handleResponsesFromIsolate라는 해당 리스너에 아직 정의되지 않은 메서드를 전달합니다.


```dart
class Worker {
  final SendPort _commands;
  final ReceivePort _responses;
// ···
  Worker._(this._responses, this._commands) {
    _responses.listen(_handleResponsesFromIsolate);
  }
```

다음으로 작업자 아이솔레이트의 포트 초기화를 담당하는 _startRemoteIsolate에 코드를 추가합니다. 이 메서드는 Worker.spawn 메서드의 Isolate.spawn에 전달되었으며 메인 아이솔레이트의 SendPort가 인자로 전달된다는 점을 기억하세요.

- 새 수신 포트를 만듭니다.

- 해당 포트의 SendPort를 메인 아이솔레이트로 다시 보냅니다.

- _handleCommandsToIsolate라는 새 메서드를 호출하고 메인 아이솔레이트의 새 ReceiverPort 및 SendPort를 인자로 전달합니다.

```dart
static void _startRemoteIsolate(SendPort sendPort) {
  final receivePort = ReceivePort();
  sendPort.send(receivePort.sendPort);
  _handleCommandsToIsolate(receivePort, sendPort);
}
```

다음으로, 메인 아이솔레이트에서 메시지 수신, 작업자 아이솔레이트에서 json 디코딩, 디코딩된 json을 응답으로 다시 보내는 작업을 담당하는 _handleCommandsToIsolate 메서드를 추가합니다.

- 먼저 작업자 아이솔레이트의 ReceiverPort에 대한 리스너를 선언합니다.

- 리스너에 추가된 콜백 내에서 try/catch 블록 내의 메인 아이솔레이트에서 전달된 JSON을 디코딩하려고 시도합니다. 디코딩이 성공하면 디코딩된 JSON을 메인 아이솔레이트로 다시 보냅니다.

- 오류가 있으면 RemoteError를 다시 보냅니다.

```dart
static void _handleCommandsToIsolate(
    ReceivePort receivePort, SendPort sendPort) {
  receivePort.listen((message) {
    try {
      final jsonData = jsonDecode(message as String);
      sendPort.send(jsonData);
    } catch (e) {
      sendPort.send(RemoteError(e.toString(), ''));
    }
  });
}
```

다음으로 _handleResponsesFromIsolate 메서드에 대한 코드를 추가합니다.

- 먼저 메시지가 RemoteError인지 확인하세요. 이 경우 해당 오류를 throw해야 합니다.

- 그렇지 않으면 메시지를 출력하십시오. 이후 단계에서는 메시지를 출력하는 대신 메시지를 반환하도록 이 코드를 업데이트합니다.

```dart
void _handleResponsesFromIsolate(dynamic message) {
  if (message is RemoteError) {
    throw message;
  } else {
    print(message);
  }
}
```

마지막으로, 외부 코드가 디코딩할 작업자 아이솔레이트에 JSON을 보낼 수 있도록 하는 퍼블릭 메서드인 parsJson 메서드를 추가합니다.

```dart
Future<Object?> parseJson(String message) async {
  _commands.send(message);
}
```

다음 단계에서 이 메서드를 업데이트합니다.

## 5단계: 동시에 여러 메시지 처리 Step 5: Handle multiple messages at the same time

현재 작업자 아이솔레이트에 메시지를 빠르게 보내는 경우 아이솔레이트는 전송된 순서가 아닌 완료된 순서대로 디코딩된 json 응답을 보냅니다. 어떤 응답이 어떤 메시지에 해당하는지 확인할 방법이 없습니다.

이 단계에서는 각 메시지에 ID를 제공하고 Completer 객체를 사용하여 외부 코드가 parseJson을 호출할 때 해당 호출자에게 반환되는 응답이 올바른 응답인지 확인함으로써 이 문제를 해결합니다.

먼저 Worker에 두 가지 클래스 수준 속성을 추가합니다.

- Map<int, Completer<Object?>> _activeRequests
- int _idCounter

```dart
class Worker {
  final SendPort _commands;
  final ReceivePort _responses;
  final Map<int, Completer<Object?>> _activeRequests = {};
  int _idCounter = 0;
```

_activeRequests 맵은 작업자 아이솔레이트로 전송된 메시지를 Completer와 연결합니다. _activeRequests에 사용되는 키는 _idCounter에서 가져오며 더 많은 메시지가 전송될수록 증가합니다.

다음으로, 작업자 아이솔레이트에 메시지를 보내기 전에 완성자를 생성하도록 parseJson 메서드를 업데이트합니다.

- 먼저 Completer를 만듭니다.

- 다음으로, 각 Completer가 고유 번호와 연결되도록 _idCounter를 증가시킵니다.

- 키가 _idCounter의 현재 번호이고 완성자가 값인 _activeRequests 맵에 항목을 추가합니다.

- 작업자 아이솔레이트에 id와 함께 메시지를 보냅니다. SendPort를 통해 하나의 값만 보낼 수 있으므로 id와 메시지를 레코드로 래핑합니다.

- 마지막으로, 완료자의 future를 반환합니다. 이는 결국 작업자 아이솔레이트의 응답을 포함하게 됩니다.

```dart
Future<Object?> parseJson(String message) async {
  final completer = Completer<Object?>.sync();
  final id = _idCounter++;
  _activeRequests[id] = completer;
  _commands.send((id, message));
  return await completer.future;
}
```

이 시스템을 처리하려면 _handleResponsesFromIsolate 및 _handleCommandsToIsolate도 업데이트해야 합니다.

_handleCommandsToIsolate에서는 메시지가 단지 json 텍스트가 아닌 두 개의 값이 있는 레코드라는 점을 고려해야 합니다. 메시지의 값을 구조 분해 할당하여 그렇게 하세요.

그런 다음 json을 디코딩한 후 sendPort.send에 대한 호출을 업데이트하여 다시 레코드를 사용하여 id와 디코딩된 json을 메인 아이솔레이트에 다시 전달합니다.

```dart
static void _handleCommandsToIsolate(
    ReceivePort receivePort, SendPort sendPort) {
  receivePort.listen((message) {
    final (int id, String jsonText) = message as (int, String); // New
    try {
      final jsonData = jsonDecode(jsonText);
      sendPort.send((id, jsonData)); // Updated
    } catch (e) {
      sendPort.send((id, RemoteError(e.toString(), '')));
    }
  });
}
```

마지막으로 _handleResponsesFromIsolate를 업데이트합니다.

- 먼저 메시지 인자의 id와 응답을 다시 구조 분해 할당합니다.

- 그런 다음 _activeRequests 맵에서 이 요청에 해당하는 완성자를 제거합니다.

- 마지막으로 오류를 발생시키거나 디코딩된 json을 출력하는 대신 응답을 전달하여 완성자를 완성하세요. 이 작업이 완료되면 메인 아이솔레이트에서 parseJson을 호출한 코드로 응답이 반환됩니다.

```dart
void _handleResponsesFromIsolate(dynamic message) {
  final (int id, Object? response) = message as (int, Object?); // New
  final completer = _activeRequests.remove(id)!; // New

  if (response is RemoteError) {
    completer.completeError(response); // Updated
  } else {
    completer.complete(response); // Updated
  }
}
```

## 6단계: 포트를 닫는 기능 추가 Step 6: Add functionality to close the ports

코드에서 아이솔레이트가 더 이상 사용되지 않으면 메인 아이솔레이트와 작업자 아이솔레이트의 포트를 닫아야 합니다.

- 먼저 포트가 닫혀 있는지 추적하는 클래스 수준 부울을 추가합니다.

- 그런 다음 Worker.close 메서드를 추가합니다. 이 메서드 내에서:
  - _closed를 true로 업데이트하세요.
  - 작업자 아이솔레이트에 최종 메시지를 보냅니다. 이 메시지는 "shutdown"이라는 String이지만 원하는 모든 객체가 될 수 있습니다. 다음 코드 스니펫에서 이를 사용합니다.

- 마지막으로 _activeRequests가 비어 있는지 확인하세요. 그렇다면 _responses라는 메인 아이솔레이트의 ReceiverPort를 닫습니다.

```dart
class Worker {
  bool _closed = false;
// ···
  void close() {
    if (!_closed) {
      _closed = true;
      _commands.send('shutdown');
      if (_activeRequests.isEmpty) _responses.close();
      print('--- port closed --- ');
    }
  }
```
- 다음으로 작업자 아이솔레이트에서 "shutdown" 메시지를 처리해야 합니다. _handleCommandsToIsolate 메서드에 다음 코드를 추가합니다. 이 코드는 메시지가 "shutdown"이라는 String인지 확인합니다. 그렇다면 작업자 아이솔레이트의 ReceiverPort를 닫고 반환합니다.

```dart
static void _handleCommandsToIsolate(
  ReceivePort receivePort,
  SendPort sendPort,
) {
  receivePort.listen((message) {
    // New if-block.
    if (message == 'shutdown') {
      receivePort.close();
      return;
    }
    final (int id, String jsonText) = message as (int, String);
    try {
      final jsonData = jsonDecode(jsonText);
      sendPort.send((id, jsonData));
    } catch (e) {
      sendPort.send((id, RemoteError(e.toString(), '')));
    }
  });
}
```

- 마지막으로 메시지 전송을 시도하기 전에 포트가 닫혀 있는지 확인하는 코드를 추가해야 합니다. Worker.parseJson 메서드에 한 줄을 추가합니다.

```dart
Future<Object?> parseJson(String message) async {
  if (_closed) throw StateError('Closed'); // New
  final completer = Completer<Object?>.sync();
  final id = _idCounter++;
  _activeRequests[id] = completer;
  _commands.send((id, message));
  return await completer.future;
}
```

## 완전한 예

```dart
import 'dart:async';
import 'dart:convert';
import 'dart:isolate';

void main() async {
  final worker = await Worker.spawn();
  print(await worker.parseJson('{"key":"value"}'));
  print(await worker.parseJson('"banana"'));
  print(await worker.parseJson('[true, false, null, 1, "string"]'));
  print(
      await Future.wait([worker.parseJson('"yes"'), worker.parseJson('"no"')]));
  worker.close();
}

class Worker {
  final SendPort _commands;
  final ReceivePort _responses;
  final Map<int, Completer<Object?>> _activeRequests = {};
  int _idCounter = 0;
  bool _closed = false;

  Future<Object?> parseJson(String message) async {
    if (_closed) throw StateError('Closed');
    final completer = Completer<Object?>.sync();
    final id = _idCounter++;
    _activeRequests[id] = completer;
    _commands.send((id, message));
    return await completer.future;
  }

  static Future<Worker> spawn() async {
    // Create a receive port and add its initial message handler
    final initPort = RawReceivePort();
    final connection = Completer<(ReceivePort, SendPort)>.sync();
    initPort.handler = (initialMessage) {
      final commandPort = initialMessage as SendPort;
      connection.complete((
        ReceivePort.fromRawReceivePort(initPort),
        commandPort,
      ));
    };

    // Spawn the isolate.
    try {
      await Isolate.spawn(_startRemoteIsolate, (initPort.sendPort));
    } on Object {
      initPort.close();
      rethrow;
    }

    final (ReceivePort receivePort, SendPort sendPort) =
        await connection.future;

    return Worker._(receivePort, sendPort);
  }

  Worker._(this._responses, this._commands) {
    _responses.listen(_handleResponsesFromIsolate);
  }

  void _handleResponsesFromIsolate(dynamic message) {
    final (int id, Object? response) = message as (int, Object?);
    final completer = _activeRequests.remove(id)!;

    if (response is RemoteError) {
      completer.completeError(response);
    } else {
      completer.complete(response);
    }

    if (_closed && _activeRequests.isEmpty) _responses.close();
  }

  static void _handleCommandsToIsolate(
    ReceivePort receivePort,
    SendPort sendPort,
  ) {
    receivePort.listen((message) {
      if (message == 'shutdown') {
        receivePort.close();
        return;
      }
      final (int id, String jsonText) = message as (int, String);
      try {
        final jsonData = jsonDecode(jsonText);
        sendPort.send((id, jsonData));
      } catch (e) {
        sendPort.send((id, RemoteError(e.toString(), '')));
      }
    });
  }

  static void _startRemoteIsolate(SendPort sendPort) {
    final receivePort = ReceivePort();
    sendPort.send(receivePort.sendPort);
    _handleCommandsToIsolate(receivePort, sendPort);
  }

  void close() {
    if (!_closed) {
      _closed = true;
      _commands.send('shutdown');
      if (_activeRequests.isEmpty) _responses.close();
      print('--- port closed --- ');
    }
  }
}
```