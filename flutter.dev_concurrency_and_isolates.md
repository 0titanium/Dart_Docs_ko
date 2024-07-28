# 동시성 및 아이솔레이트 Concurrency and isolates

모든 Dart 코드는 스레드와 비슷한 아이솔레이트에서 실행되지만, 아이솔레이트는 자체 격리된 메모리를 갖는다는 점에서 다릅니다. 아이솔레이트는 어떤 식으로도 상태를 공유하지 않으며, 메시지로만 통신할 수 있습니다. 기본적으로 Flutter 앱은 모든 작업을 싱글 아이솔레이트 (메인 아이솔레이트)에서 수행합니다. 대부분의 경우 이 모델은 더 간단한 프로그래밍을 허용하고 애플리케이션의 UI가 무반응이 되지 않을 정도로 빠릅니다.

하지만 때때로 애플리케이션은 "UI 쟁크"(거친 동작)를 일으킬 수 있는 매우 큰 계산을 수행해야 합니다. 이런 이유로 앱에서 쟁크가 발생하는 경우 이러한 계산을 헬퍼 아이솔레이트로 옮길 수 있습니다. 이렇게 하면 기본 런타임 환경에서 메인 UI 아이솔레이트의 작업과 동시에 계산을 실행할 수 있으며 멀티코어 장치를 활용할 수 있습니다.

각 아이솔레이트된 개체에는 자체 메모리와 자체 이벤트 루프가 있습니다. 이벤트 루프는 이벤트 큐에 추가된 순서대로 이벤트를 처리합니다. 메인 아이솔레이트에서 이러한 이벤트는 UI에서 사용자 탭 처리, 함수 실행, 화면에 프레임 그리기 등 무엇이든 될 수 있습니다. 다음 그림은 처리를 기다리는 3개의 이벤트가 있는 예시 이벤트 큐를 보여줍니다.

매끄러운 렌더링을 위해 Flutter는 초당 60회(60Hz 기기의 경우) 이벤트 큐에 "페인트 프레임" 이벤트를 추가합니다. 이러한 이벤트가 제때 처리되지 않으면 애플리케이션에서 UI 쟁크가 발생하거나 더 나쁜 경우 전혀 응답하지 않게 됩니다.

프로세스가 프레임 갭(두 프레임 사이의 시간)에서 완료될 수 없는 경우, 메인 아이솔레이트가 초당 60프레임을 생성할 수 있도록 작업을 다른 아이솔레이트로 오프로드하는 것이 좋습니다. Dart에서 아이솔레이트를 스폰하면 메인 아이솔레이트를 차단하지 않고 동시에 작업을 처리할 수 있습니다.

Dart 설명서의 동시성 페이지 에서 아이솔레이트와 이벤트 루프가 Dart에서 작동하는 방식에 대해 자세히 알아볼 수 있습니다.

아이솔레이트 및 이벤트 루프 | Flutter in Focus

## 아이솔레이트의 일반적인 사용 사례 Common use cases for isolates

아이솔레이트를 사용해야 하는 경우에 대한 엄격한 규칙은 하나뿐이며, 그것은 대규모 계산으로 인해 Flutter 애플리케이션에서 UI 멈춤 현상이 발생하는 경우입니다. 이 멈춤 현상은 Flutter의 프레임 갭보다 오래 걸리는 계산이 있을 때 발생합니다.

구현과 입력 데이터에 따라 모든 프로세스를 완료하는 데 더 오랜 시간이 걸릴 수 있으므로 아이솔레이트 사용을 고려해야 하는 경우에 대한 전체 목록을 만드는 것은 불가능합니다.

즉, 아이솔레이트는 일반적으로 다음과 같은 경우에 사용됩니다.

- 로컬 데이터베이스에서 데이터 읽기
- 푸시 알림 보내기
- 대용량 데이터 파일 파싱 및 디코딩
- 사진, 오디오 파일, 비디오 파일 처리 또는 압축
- 오디오 및 비디오 파일 변환
- FFI(Foreign Function Interface, 한 프로그래밍 언어에서 다른 프로그래밍 언어의 코드를 호출하기 위한 인터페이스)를 사용하는 동안 비동기 지원이 필요한 경우
- 복잡한 목록이나 파일 시스템에 필터링 적용

# 아이솔레이트 간의 메시지 전달 Message passing between isolates

Dart의 아이솔레이트는 Actor 모델의 구현입니다. 이들은 Port 객체로 수행되는 메시지 전달을 통해서만 서로 통신할 수 있습니다. 메시지가 서로 "전달"될 때, 일반적으로 전송 아이솔레이트에서 수신 아이솔레이트로 복사됩니다. 즉, 아이솔레이트에 전달된 모든 값은 해당 아이솔레이트에서 변형되더라도 원래 아이솔레이트의 값을 변경하지 않습니다.

아이솔레이트에 전달될 때 복사되지 않는 유일한 객체는 변경할 수 없는 불변 객체, 예를 들어 String이나 수정 불가능한 바이트입니다. 아이솔레이트 간에 불변 객체를 전달하면 더 나은 성능을 위해 객체가 복사되는 대신 해당 객체에 대한 참조가 포트를 통해 전송됩니다. 불변 객체는 업데이트할 수 없으므로, 이는 액터 모델 동작을 효과적으로 유지합니다.

이 규칙의 예외는 아이솔레이트가 Isolate.exit 메서드를 사용하여 메시지를 보낼 때 종료되는 경우입니다. 메시지를 보낸 후에는 보내는 아이솔레이트가 존재하지 않으므로 메시지의 소유권을 한 아이솔레이트에서 다른 아이솔레이트로 넘겨서 한 아이솔레이트만 메시지에 액세스할 수 있도록 할 수 있습니다.

메시지를 보내는 두 개의 최하위 레벨 primitive는 SendPort.send, 이는 보낼 때 변경 가능한 메시지의 사본을 만들고, Isolate.exit는 메시지에 대한 참조를 보냅니다. Isolate.run과 compute 둘 다 내부적으로 Isolate.exit를 사용합니다.

# 단기 아이솔레이트 Short-lived isolates

Flutter에서 프로세스를 아이솔레이트로 옮기는 가장 쉬운 방법은 Isolate.run 메서드를 사용하는 것입니다. 이 메서드는 아이솔레이트를 생성하고, 생성된 아이솔레이트에 콜백을 전달하여 일부 계산을 시작하고, 계산에서 값을 반환한 다음, 계산이 완료되면 아이솔레이트를 종료합니다. 이 모든 작업은 주 아이솔레이트와 동시에 수행되며, 아이솔레이트를 차단하지 않습니다.


이 Isolate.run메서드는 새로운 아이솔레이트에서 실행되는 단일 인자인 콜백 함수를 필요로 합니다. 이 콜백의 함수 시그니처에는 정확히 하나의 필수적이고 명명되지 않은 인자가 있어야 합니다. 계산이 완료되면 콜백의 값을 메인 아이솔레이트로 반환하고 생성된 아이솔레이트를 종료합니다.

예를 들어, 파일에서 큰 JSON 블롭(Binary Large Object)을 로드하고 해당 JSON을 커스텀 Dart 객체로 변환하는 이 코드를 고려해 보세요. JSON 디코딩 프로세스가 새 아이솔레이트로 오프로드되지 않았다면 이 메서드로 인해 UI가 몇 초 동안 응답하지 않게 됩니다.

```dart
// 211,640개의 사진 객체의 리스트를 생성합니다.
// (The JSON file은 ~20MB.)
Future<List<Photo>> getPhotos() async {
  final String jsonString = await rootBundle.loadString('assets/photos.json');
  final List<Photo> photos = await Isolate.run<List<Photo>>(() {
    final List<Object?> photoData = jsonDecode(jsonString) as List<Object?>;
    return photoData.cast<Map<String, Object?>>().map(Photo.fromJson).toList();
  });
  return photos;
}
```

Isolates를 사용하여 백그라운드에서 JSON을 파싱하는 전체 연습 과정은 이 요리책 레시피를 참조하세요 .

# 상태가 있는 장기 아이솔레이트 Stateful, longer-lived isolates

단기 아이솔레이트는 사용하기 편리하지만, 새로운 아이솔레이트를 생성하고 한 아이솔레이트에서 다른 아이솔레이트로 객체를 복사하는 데 필요한 성능 오버헤드가 있습니다. 반복적으로 Isolate.run를 사용하여 동일한 계산을 수행하는 경우 즉시 종료되지 않는 아이솔레이트를 생성하면 성능이 더 좋아질 수 있습니다.

이를 위해 Isolate.run을 추상화하는 더 로우 레벨의 아이솔레이트 관련 API를 몇 가지 쉽게 사용할 수 있습니다.

- Isolate.spawn()와 Isolate.exit()
- ReceivePort와 SendPort
- send()메서드

Isolate.run 메서드를 사용하면 새 아이솔레이트는 단일 메시지를 메인 아이솔레이트에 반환한 후 즉시 종료됩니다. 때로는 오래 지속되고 시간이 지남에 따라 여러 메시지를 서로 전달할 수 있는 아이솔레이트가 필요합니다. Dart에서는 아이솔레이트 API와 Ports를 사용하여 이를 달성할 수 있습니다. 이러한 오래 지속되는 아이솔레이트는 일반적으로 백그라운드 작업자 라고 합니다 .

장기 아이솔레이트는 애플리케이션의 수명 동안 반복적으로 실행해야 하는 특정 프로세스가 있는 경우나, 일정 기간 동안 실행되고 메인 아이솔레이트에 여러 개의 반환 값을 생성해야 하는 프로세스가 있는 경우에 유용합니다.

또는 worker_manager를 사용하여 장기 아이솔레이트를 관리할 수도 있습니다.

# ReceivePorts 및 SendPorts ReceivePorts and SendPorts

두 개의 클래스(아이솔레이트 외) ReceivePort및 SendPort를 사용하여 아이솔레이트 간에 장기 통신을 설정합니다. 이러한 포트는 아이솔레이트된 개체가 서로 통신할 수 있는 유일한 방법입니다.

Ports는  Streams와 유사하게 동작하는데, 여기서는 StreamControlleror 또는 Sink가 한 아이솔레이트에서 생성되고 리스너는 다른 아이솔레이트에서 설정됩니다. 이 비유에서 StreamConroller는 SendPort라고 하며 send() 메서드로 메시지를 "추가"할 수 있습니다. ReceivePorts는 리스너이고 이러한 리스너가 새 메시지를 받으면 메시지를 인자로 사용하여 제공된 콜백을 호출합니다.

메인 아이솔레이트와 작업자 아이솔레이트 사이에 양방향 통신을 설정하는 방법에 대한 자세한 설명은 Dart 설명서의 예를 따르세요 .

# 아이솔레이트에서 플랫폼 플러그인 사용 Using platform plugins in isolates

Flutter 3.7부터 백그라운드 아이솔레이트에서 플랫폼 플러그인을 사용할 수 있습니다. 이를 통해 UI를 차단하지 않는 아이솔레이트에 무거운 플랫폼 종속 계산을 오프로드할 수 있는 많은 가능성이 열립니다. 예를 들어, 네이티브 호스트 API(예: Android의 Android API, iOS의 iOS API 등)를 사용하여 데이터를 암호화한다고 가정해 보겠습니다. 이전에는 호스트 플랫폼에 데이터를 마셜링(객체의 메모리 표현을 특히 서로 다른 런타임 간에 저장 또는 전송에 적합한 데이터 형식으로 변환하는 프로세스) 하면 UI 스레드 시간이 낭비될 수 있었지만, 이제는 백그라운드 아이솔레이트에서 수행할 수 있습니다.

플랫폼 채널 아이솔레이트는 BackgroundIsolateBinaryMessenger API를 사용합니다. 다음 스니펫은 백그라운드 아이솔레이트에서 shared_preferences 패키지를 사용하는 예를 보여줍니다.

```dart
import 'dart:Isolate';

import 'package:flutter/services.dart';
import 'package:shared_preferences/shared_preferences.dart';

void main() {
  // 백그라운드 아이솔레이트에 전달할 루트 아이솔레이트를 식별합니다.
  RootIsolateToken rootIsolateToken = RootIsolateToken.instance!;
  Isolate.spawn(_isolateMain, rootIsolateToken);
}

Future<void> _isolateMain(RootIsolateToken rootIsolateToken) async {
  // 루트 아이솔레이트로 백그라운드 아이솔레이트를 등록하세요.
  BackgroundIsolateBinaryMessenger.ensureInitialized(rootIsolateToken);

  // 이제 shared_preferences 플러그인을 사용할 수 있습니다.
  SharedPreferences sharedPreferences = await SharedPreferences.getInstance();

  print(sharedPreferences.getBool('isDebug'));
}
```

# 아이솔레이트의 한계 Limitations of Isolates

멀티스레딩이 있는 언어에서 Dart로 넘어왔다면 아이솔레이트가 스레드처럼 동작할 것이라고 기대하는 것이 합리적이지만, 그렇지 않습니다. 아이솔레이트된 객체는 자체 전역 필드를 가지고 있으며, 메시지 전달로만 통신할 수 있고, 아이솔레이트의 변경 가능한 객체는 단일 아이솔레이트에서만 액세스할 수 있습니다. 따라서 아이솔레이트는 자체 메모리에 대한 액세스만 가능합니다. 예를 들어, configuration이라는 전역 변경 가능한 변수가 있는 애플리케이션이 있는 경우, 생성된 아이솔레이트된 객체의 새 전역 필드로 복사됩니다. 생성된 아이솔레이트에서 해당 변수를 변경해도 메인 아이솔레이트에서는 그대로 유지됩니다. configuration 객체를 새 아이솔레이트에 메시지로 전달하더라도 마찬가지입니다. 이것이 아이솔레이트가 작동하는 방식이며 아이솔레이트를 사용할 때 명심해야 할 중요한 사항입니다.

# 웹 플랫폼 및 compute 메서드 Web platforms and compute

Flutter 웹을 포함한 Dart 웹 플랫폼은 아이솔레이트를 지원하지 않습니다. Flutter 앱으로 웹을 타겟팅하는 경우 compute 메서드를 사용하여 코드가 컴파일되도록 할 수 있습니다. compute() 메서드는 웹의 메인 스레드에서 계산을 실행하지만 모바일 기기에서는 새 스레드를 생성합니다. 모바일 및 데스크톱 플랫폼에서 await compute(fun, message)는 await Isolate.run(() => fun(message))과 동일합니다.

웹에서의 동시성에 대한 자세한 내용을 보려면 dart.dev의 동시성 문서를 확인하세요.

# rootBundle 접근 권한이나 dart:ui 메서드가 없습니다 No rootBundle access or dart:ui methods

모든 UI 작업과 Flutter 자체는 메인 아이솔레이트에 결합되어 있습니다. 따라서 생성된 아이솔레이트에서 rootBundle 사용하여 에셋에 액세스할 수 없으며, 생성된 아이솔레이트에서 위젯이나 UI 작업을 수행할 수 없습니다.

# 호스트 플랫폼에서 Flutter로의 제한된 플러그인 메시지 Limited plugin messages from host platform to Flutter

백그라운드 아이솔레이트 플랫폼 채널을 사용하면 아이솔레이트에서 플랫폼 채널을 사용하여 호스트 플랫폼(예: Android 또는 iOS)에 메시지를 보내고 해당 메시지에 대한 응답을 받을 수 있습니다. 그러나 호스트 플랫폼에서 요청하지 않은 메시지를 받을 수는 없습니다.

예를 들어, Firestore는 플랫폼 채널을 사용하여 Flutter에 요청하지 않은 업데이트를 푸시하기 때문에 백그라운드 아이솔레이트에서 장기 Firestore 리스너를 설정할 수 없습니다. 그러나 백그라운드에서 Firestore에 응답을 쿼리할 수 있습니다.

# 추가 정보 More information

아이솔레이트에 대한 자세한 내용은 다음 자료를 확인하세요.

- 여러 개의 아이솔레이트를 사용하는 경우 Flutter의 IsolateNameServer 클래스나 Flutter를 사용하지 않는 Dart 애플리케이션의 기능을 복제하는 pub 패키지를 고려하세요.

- Dart의 아이솔레이트는 Actor 모델을 구현한 것입니다.

- isolate_agents 는 포트를 추상화하고 장기 아이솔레이트를 쉽게 만들 수 있는 패키지입니다.

- BackgroundIsolateBinaryMessengerAPI 발표에 대해 자세히 알아보세요 .
