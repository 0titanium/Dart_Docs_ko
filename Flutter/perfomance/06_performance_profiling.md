# 플러터 성능 프로파일링 Flutter perfomance profiling

```
노트

성능 문제를 디버깅하기 위해 Performance View(Flutter DevTools의 일부)를 사용하는 방법을 배우려면 "Using the Performance view"를 참조하세요.
```

```
학습 내용

- Flutter는 초당 60프레임(fps)의 성능을 제공하는 것을 목표로 하며, 120Hz 업데이트가 가능한 기기에서는 초당 120프레임의 성능을 제공합니다.  

- 60fps를 달성하려면 프레임이 약 16ms마다 렌더링되어야 합니다.  

- UI가 매끄럽게 렌더링되지 않을 때 버벅거림이 발생합니다. 예를 들어, 때때로 프레임 렌더링에 10배 더 긴 시간이 걸려 프레임이 누락되고 애니메이션이 눈에 띄게 끊기는 현상이 나타날 수 있습니다.
```

"빠른 앱도 좋지만, 매끄러운 앱이 더 좋다"는 말이 있습니다. 앱이 매끄럽게 렌더링되지 않을 경우, 이를 어떻게 해결해야 할까요? 어디서부터 시작해야 할까요? 이 가이드는 시작 지점, 수행할 단계, 그리고 도움이 되는 도구를 알려줍니다.

```
노트

- 앱의 성능은 여러 요소에 의해 결정됩니다. 성능은 때로는 순수 속도를 의미하기도 하지만, UI의 매끄러움과 끊김 없는 동작을 포함하기도 합니다. 성능의 다른 예로는 I/O 또는 네트워크 속도가 있습니다. 이 페이지는 주로 두 번째 유형의 성능(UI 매끄러움)에 초점을 맞추지만, 대부분의 도구를 사용하여 다른 성능 문제도 진단할 수 있습니다.

- Dart 코드 내에서 추적을 수행하려면 디버깅 페이지의 "Tracing Dart code"를 참조하세요.
```

## 성능 문제 진단 Diagnosing performance problems

성능 문제를 가진 앱을 진단하려면, 성능 오버레이를 활성화하여 UI 스레드와 래스터 스레드를 확인해야 합니다. 시작하기 전에 프로파일 모드에서 실행 중인지 확인하고, 에뮬레이터를 사용하지 않아야 합니다. 최상의 결과를 얻으려면 사용자가 사용할 가능성이 있는 가장 느린 기기를 선택하는 것이 좋습니다

### 물리적 기기에 연결 Connect to a physical device

Flutter 애플리케이션의 성능 디버깅은 거의 모든 경우 물리적 Android 또는 iOS 기기에서, Flutter 애플리케이션이 프로파일 모드로 실행되는 상태에서 수행해야 합니다. 디버그 모드를 사용하거나 시뮬레이터 또는 에뮬레이터에서 앱을 실행하는 것은 일반적으로 릴리스 모드 빌드의 최종 동작을 반영하지 않습니다. 사용자가 실제로 사용할 가능성이 있는 가장 느린 기기에서 성능을 확인하는 것도 고려해야 합니다.

```
왜 실제 기기에서 실행해야 하는가

- 시뮬레이터와 에뮬레이터는 동일한 하드웨어를 사용하지 않으므로 성능 특성이 다릅니다. 일부 작업은 시뮬레이터에서 실제 기기보다 더 빠르게 실행되지만, 일부는 더 느릴 수 있습니다.  

- 디버그 모드는 프로파일 또는 릴리스 빌드에서 실행되지 않는 추가 검사(예: assert)를 활성화하며, 이러한 검사는 비용이 많이 들 수 있습니다.  

- 또한 디버그 모드는 릴리스 모드와 코드 실행 방식이 다릅니다. 디버그 빌드는 앱 실행 중에 Dart 코드를 "즉시 컴파일"(JIT)하지만, 프로파일 및 릴리스 빌드는 앱이 기기에 로드되기 전에 네이티브 명령어로 미리 컴파일(AOT)됩니다. JIT는 JIT 컴파일로 인해 앱이 일시적으로 멈추게 할 수 있으며, 이는 버벅거림을 유발할 수 있습니다.
```

### 프로파일 모드에서 실행

Flutter의 프로파일 모드는 릴리스 모드와 거의 동일하게 애플리케이션을 컴파일하고 실행하지만, 성능 문제를 디버깅할 수 있도록 약간의 추가 기능을 제공합니다. 예를 들어, 프로파일 모드는 프로파일링 도구에 추적 정보를 제공합니다.

```
노트

Dart/Flutter DevTools는 프로파일 모드에서 실행 중인 Flutter 웹 애플리케이션에 연결할 수 없습니다. 웹 애플리케이션의 타임라인 이벤트를 생성하려면 Chrome DevTools를 사용하세요.
```

앱을 프로파일 모드에서 실행하려면 다음과 같이 진행하세요:

- VS Code에서 `launch.json` 파일을 열고, `flutterMode` 속성을 `profile`로 설정합니다. (프로파일링이 끝나면 다시 `release` 또는 `debug`로 변경하세요.)

    ```json
    "configurations": [
    {
        "name": "Flutter",
        "request": "launch",
        "type": "dart",
        "flutterMode": "profile"
    }
    ]
    ```

- Android Studio와 IntelliJ에서는 Run > Flutter Run main.dart in Profile Mode 메뉴 항목을 사용하세요.

- 명령줄에서는 `--profile` 플래그를 사용하세요.
    ```
    flutter run --profile
    ```

다양한 모드에 대한 자세한 정보는 Flutter의 빌드 모드를 참조하세요.

다음 섹션에서 설명할 성능 오버레이를 보기 위해 DevTools를 열고 시작할 것입니다.

## DevTools 실행 Launch DevTools

DevTools는 프로파일링, 힙 검사, 코드 커버리지 표시, 성능 오버레이 활성화, 단계별 디버거와 같은 기능을 제공합니다. DevTools의 타임라인 뷰를 사용하면 애플리케이션의 UI 성능을 프레임별로 조사할 수 있습니다.

앱이 프로파일 모드에서 실행 중인 상태에서 DevTools를 실행하세요.

## 성능 오버레이 The performance overlay

성능 오버레이는 앱에서 시간이 어디에 소비되고 있는지 보여주는 두 개의 그래프에 통계를 표시합니다. UI가 버벅거리거나 프레임이 건너뛰어지는 경우, 이 그래프들을 통해 그 원인을 파악할 수 있습니다. 그래프는 실행 중인 앱 위에 표시되며, 일반적인 위젯처럼 그려지지 않습니다. Flutter 엔진 자체가 오버레이를 그리며 성능에 미치는 영향은 최소화됩니다. 각 그래프는 해당 스레드의 마지막 300프레임을 나타냅니다.

이 섹션에서는 성능 오버레이를 활성화하고 이를 사용하여 애플리케이션의 버벅거림 원인을 진단하는 방법을 설명합니다. 아래 스크린샷은 Flutter Gallery 예제에서 실행 중인 성능 오버레이를 보여줍니다.

성능 오버레이는 래스터 스레드(위)와 UI 스레드(아래)를 보여줍니다.

수직 녹색 막대는 현재 프레임을 나타냅니다.

## 그래프 해석 Interpreting the graphs

위쪽 그래프("GPU"로 표시)는 래스터 스레드에서 소비된 시간을, 아래쪽 그래프는 UI 스레드에서 소비된 시간을 보여줍니다. 그래프를 가로지르는 흰 선은 수직 축을 따라 16ms 단위를 나타냅니다. 그래프가 이 선을 넘으면 60Hz보다 낮은 속도로 실행되고 있음을 의미합니다. 수평 축은 프레임을 나타냅니다. 애플리케이션이 화면을 그릴 때만 그래프가 업데이트되므로, 애플리케이션이 유휴 상태일 경우 그래프는 멈춥니다.

오버레이는 항상 프로파일 모드에서 확인해야 합니다. 디버그 모드에서는 개발을 돕기 위해 비용이 많이 드는 assert를 활성화하여 성능이 의도적으로 희생되므로, 결과가 오해를 불러일으킬 수 있습니다.

각 프레임은 1/60초(약 16ms) 이내에 생성되고 표시되어야 합니다. 이 한계를 초과한 프레임(어느 그래프에서든)은 표시되지 못하고 버벅거림이 발생하며, 그래프의 하나 또는 양쪽에 수직 빨간 막대가 나타납니다. 빨간 막대가 UI 그래프에 나타나면 Dart 코드가 과도한 비용을 소모하는 것입니다. 빨간 막대가 GPU 그래프에 나타나면 장면을 신속히 렌더링하기에 너무 복잡한 것입니다.

수직 빨간 막대는 현재 프레임을 렌더링하고 그리는 데 비용이 많이 든다는 것을 나타냅니다.

두 그래프 모두 빨간 막대를 표시할 경우, UI 스레드부터 진단을 시작하세요.

## 플러터의 스레드 Flutter's threads

Flutter는 작업을 수행하기 위해 여러 스레드를 사용하지만, 오버레이에는 두 개의 스레드만 표시됩니다. 모든 Dart 코드는 UI 스레드에서 실행됩니다. 다른 스레드에 직접 액세스할 수는 없지만, UI 스레드에서의 작업은 다른 스레드의 성능에 영향을 미칩니다.

- 플랫폼 스레드*
  플랫폼의 메인 스레드로, 플러그인 코드가 실행됩니다. 자세한 내용은 iOS의 UIKit 문서 또는 Android의 MainThread 문서를 참조하세요. 이 스레드는 성능 오버레이에 표시되지 않습니다.  

- UI 스레드  
  UI 스레드는 Dart VM에서 Dart 코드를 실행합니다. 이 스레드에는 작성한 코드와 Flutter 프레임워크가 앱을 대신해 실행하는 코드가 포함됩니다. 앱이 장면(scene)을 생성하고 표시할 때 UI 스레드는 레이어 트리(layer tree)를 생성합니다. 레이어 트리는 장치에 독립적인 페인팅 명령을 포함하는 경량 객체로, 이를 래스터 스레드로 보내 장치에서 렌더링합니다. 이 스레드를 차단하지 마세요! 성능 오버레이의 하단 행에 표시됩니다.  

- 래스터 스레드  
  래스터 스레드는 레이어 트리를 가져와 GPU(그래픽 처리 장치)와 통신하여 표시합니다. 이 스레드나 데이터를 직접적으로 액세스할 수는 없지만, 이 스레드가 느리다면 Dart 코드에서 한 작업의 결과입니다. 그래픽 라이브러리인 Skia와 Impeller가 이 스레드에서 실행됩니다. 성능 오버레이의 상단 행에 표시됩니다. 참고로 래스터 스레드가 GPU를 위해 래스터화하는 동안, 스레드 자체는 CPU에서 실행됩니다.  

- I/O 스레드  
  UI 또는 래스터 스레드를 차단할 수 있는 작업(주로 I/O)을 수행합니다. 이 스레드는 성능 오버레이에 표시되지 않습니다.  

추가 정보와 동영상 링크는 Flutter 위키의 The Framework architecture 및 커뮤니티 기사를 참조하세요.

### 성능 오버레이 표시 Displaying the performance overlay

성능 오버레이 표시를 전환하는 방법은 다음과 같습니다:

- Flutter 인스펙터 사용
- 명령줄 사용
- 프로그래밍 방식으로 설정

#### Flutter 인스펙터 사용

PerformanceOverlay 위젯을 활성화하는 가장 쉬운 방법은 DevTools의 인스펙터 뷰에 있는 Flutter 인스펙터를 사용하는 것입니다. 실행 중인 앱에서 Performance Overlay버튼을 클릭하여 오버레이를 전환하세요.  

#### 명령줄 사용

명령줄에서 P 키를 눌러 성능 오버레이를 전환할 수 있습니다.  

#### 프로그래밍 방식

오버레이를 프로그래밍 방식으로 활성화하려면 Performance overlay 섹션을 참고하세요. 해당 섹션은 Debugging Flutter apps programmatically 페이지에 설명되어 있습니다.

## UI 그래프에서 문제 식별

성능 오버레이의 UI 그래프에 빨간색이 표시되면, GPU 그래프에도 빨간색이 나타나더라도 우선 Dart VM을 프로파일링하는 것부터 시작하세요.

## GPU 그래프에서 문제 식별

어떤 장면(scene)이 쉽게 레이어 트리를 생성할 수 있지만, 래스터 스레드에서 렌더링하기에 비용이 많이 드는 경우가 있습니다. 이런 상황에서는 UI 그래프에 빨간색이 나타나지 않지만, GPU 그래프에 빨간색이 표시됩니다. 이 경우, 렌더링 코드를 느리게 만드는 원인을 코드에서 찾아야 합니다. 특정 작업은 GPU에서 더 어려울 수 있습니다. 예를 들어, 불필요한 saveLayer 호출, 여러 객체와 교차하는 불투명도, 특정 상황에서의 클립 및 그림자가 포함될 수 있습니다.

애니메이션 중에 느려지는 원인이 의심된다면, Flutter 인스펙터의 Slow Animations 버튼을 클릭하여 애니메이션 속도를 5배 느리게 만들어 보세요. 속도에 대한 더 많은 제어가 필요하다면 프로그래밍 방식으로 설정할 수도 있습니다.  

- 느려짐이 첫 프레임에서 발생하나요, 아니면 애니메이션 전체에서 발생하나요?  

- 애니메이션 전체에서 발생한다면, 클리핑이 원인일 수 있습니다. 클리핑을 사용하지 않는 대체 방법으로 장면을 그릴 수 있는지 확인하세요. 예를 들어, 둥근 사각형을 클리핑하는 대신 불투명한 모서리를 사각형 위에 오버레이하는 방법이 있습니다.  

- 만약 페이드, 회전 또는 다른 방식으로 조작되는 정적인 장면이라면, RepaintBoundary를 사용하는 것이 도움이 될 수 있습니다.

### 오프스크린 레이어 확인  

saveLayer 메서드는 Flutter 프레임워크에서 가장 비용이 많이 드는 메서드 중 하나입니다. 장면에 후처리를 적용할 때 유용하지만, 앱을 느리게 만들 수 있으므로 필요하지 않을 경우 피해야 합니다. 명시적으로 saveLayer를 호출하지 않더라도, Clip.antiAliasWithSaveLayer 같은 클립 동작(clipBehavior)을 지정할 때 암묵적으로 호출될 수 있습니다.  

예를 들어, 불투명도(opacity)가 있는 객체 그룹이 saveLayer를 사용하여 렌더링되는 경우가 있을 수 있습니다. 이 경우, 위젯 트리에서 더 높은 부모 위젯에 불투명도를 적용하는 대신, 각 개별 위젯에 불투명도를 적용하는 것이 더 성능 면에서 효율적일 가능성이 높습니다.  

클리핑이나 그림자 같은 다른 비용이 많이 드는 작업도 동일한 방식으로 고려해야 합니다.

```
노트

불투명도(opacity), 클리핑(clipping), 그림자(shadow)는 그 자체로 나쁜 것은 아닙니다. 그러나 위젯 트리 상단에 이를 적용하면 추가적인 `saveLayer` 호출과 불필요한 처리를 초래할 수 있습니다.
```

saveLayer 호출을 확인했을 때, 다음 질문을 스스로에게 던져 보세요:

1. 앱에서 이 효과가 정말로 필요한가?  

2. 이러한 호출 중 일부를 제거할 수 있는가?  

3. 그룹 대신 개별 요소에 동일한 효과를 적용할 수 있는가?

### 캐시되지 않은 이미지 확인

RepaintBoundary를 사용해 이미지를 캐시하는 것은 상황에 따라 유용할 수 있습니다.  

리소스 관점에서 가장 비용이 많이 드는 작업 중 하나는 이미지 파일을 사용해 텍스처를 렌더링하는 것입니다. 이 과정은 다음 단계를 포함합니다:  

1. 압축된 이미지를 영구 저장소에서 가져옵니다.  

2. 이미지를 호스트 메모리(GPU 메모리)로 디컴프레션합니다.  

3. 디바이스 메모리(RAM)로 전송합니다.  

즉, 이미지 I/O는 비용이 많이 들 수 있습니다. 캐시는 복잡한 계층 구조의 스냅샷을 제공하여 이후 프레임에서 렌더링이 더 쉬워지도록 도와줍니다. 하지만 래스터 캐시 항목은 생성 비용이 높고 많은 GPU 메모리를 사용하므로, 이미지를 캐시하는 것은 꼭 필요한 경우에만 사용해야 합니다.

### 위젯 리빌드 프로파일러 보기  

Flutter 프레임워크는 60fps와 부드러운 애플리케이션을 유지하도록 설계되었습니다. 그러나 UI가 필요한 것보다 더 많이 매 프레임마다 다시 빌드되는 간단한 버그로 인해 잔상이 발생할 수 있습니다. 위젯 리빌드 프로파일러는 이러한 버그로 인한 성능 문제를 디버깅하고 수정하는 데 도움을 줍니다.  

Android Studio와 IntelliJ의 Flutter 플러그인을 사용하면 현재 화면과 프레임에 대한 위젯 리빌드 횟수를 확인할 수 있습니다. 자세한 방법은 Show performance data 섹션을 참조하세요.  

## 벤치마킹  

벤치마크 테스트를 작성하여 앱의 성능을 측정하고 추적할 수 있습니다. Flutter Driver 라이브러리는 벤치마킹을 지원합니다. 이 통합 테스트 프레임워크를 사용하여 다음과 같은 메트릭을 생성할 수 있습니다:  

- 버벅거림  
- 다운로드 크기  
- 배터리 효율  
- 시작 시간  

이러한 벤치마크를 추적하면 성능에 부정적인 영향을 미치는 회귀(regression)가 도입되었을 때 이를 인지할 수 있습니다.  

더 많은 정보를 보려면 Integration testing을 확인하세요.

## 기타 자료

다음 자료들은 Flutter 도구 사용 및 디버깅에 대한 추가 정보를 제공합니다:  

- Debugging  
- Flutter inspector  
- Flutter inspector talk, DartConf 2018에서 발표  
- Why Flutter Uses Dart, Hackernoon 기사  
- Why Flutter uses Dart, Flutter 채널의 영상  
- DevTools: Dart와 Flutter 앱을 위한 성능 도구  
- Flutter API 문서: 특히 PerformanceOverlay 클래스와 dart:developer 패키지  

이 자료들을 통해 Flutter의 도구와 디버깅 방법에 대해 더 깊이 이해할 수 있습니다.  