# Flutter 아키텍처 개요 Flutter architectural overview

이 문서에서는 Flutter의 아키텍처에 대한 개요를 제공하며, Flutter 디자인의 핵심 원칙과 개념을 포함하고 있습니다.

Flutter는 iOS 및 Android와 같은 운영 체제 전반에서 코드 재사용을 허용하는 동시에 애플리케이션이 기본 플랫폼 서비스와 직접 인터페이스할 수 있도록 설계된 크로스 플랫폼 UI 툴킷입니다. 목표는 개발자가 다양한 플랫폼에서 자연스럽게 느껴지는 고성능 앱을 제공하고 가능한 한 많은 코드를 공유하면서 존재하는 차이를 수용할 수 있도록 하는 것입니다.

개발 중에 Flutter 앱은 전체 재컴파일 없이 변경 사항의 상태를 저장하는 핫 리로드를 제공하는 VM에서 실행됩니다. 출시를 위해 Flutter 앱은 Intel x64 또는 ARM 명령어 등 기계어 코드로 직접 컴파일되거나 웹을 대상으로 하는 경우 JavaScript로 컴파일됩니다. 프레임워크는 허용된 BSD 라이선스를 갖춘 오픈 소스이며, 핵심 라이브러리 기능을 보완하는 제3자 패키지의 번창하는 생태계를 갖추고 있습니다.

이 개요는 다음과 같은 여러 섹션으로 구분됩니다.

1. 레이어 모델: Flutter가 구성되는 부분입니다.
2. 반응형 사용자 인터페이스: Flutter 사용자 인터페이스 개발의 핵심 개념입니다.
3. 위젯 소개: Flutter 사용자 인터페이스의 기본 구성 요소입니다.
4. 렌더링 프로세스: Flutter가 UI 코드를 픽셀로 변환하는 방법
5. 플랫폼 임베더 개요: 모바일 및 데스크톱 OS에서 Flutter 앱을 실행할 수 있게 해주는 코드입니다.
6. Flutter를 다른 코드와 통합: Flutter 앱에서 사용할 수 있는 다양한 기술에 대한 정보입니다.
7. 웹 지원: 브라우저 환경에서 Flutter의 특징에 대한 결론입니다.

## 아키텍처 레이어 Architectural layers

Flutter는 확장 가능한 계층적 시스템으로 설계되었습니다. 각 계층은 독립적인 라이브러리로 구성되어 있으며, 각 라이브러리는 계층에 의존합니다. 어떤 계층도 다른 계층에 특별한 접근 권한을 가지지 않으며, 프레임워크의 모든 부분은 선택적이고 교체 가능하도록 설계되었습니다.

Flutter 애플리케이션은 각 운영 체제에서 다른 네이티브 애플리케이션과 동일한 방식으로 패키징됩니다. 플랫폼에 맞는 임베더(embedder)는 진입점을 제공하며, 렌더링 표면, 접근성, 입력 같은 서비스에 대한 접근을 위해 운영 체제와 조율하고, 메시지 이벤트 루프를 관리합니다. 임베더는 플랫폼에 적합한 언어로 작성되며, 현재 Android에서는 Java와 C++, iOS와 macOS에서는 Objective-C/Objective-C++, Windows와 Linux에서는 C++로 작성됩니다. 임베더를 사용하면 Flutter 코드를 기존 애플리케이션의 모듈로 통합할 수 있거나, 애플리케이션의 전체 콘텐츠로 사용할 수도 있습니다. Flutter는 일반적인 타겟 플랫폼에 대한 여러 임베더를 포함하고 있지만, 다른 임베더도 존재합니다.

Flutter의 핵심에는 Flutter 엔진이 있습니다. 이 엔진은 대부분 C++로 작성되었으며 모든 Flutter 애플리케이션을 지원하는 데 필요한 기본 요소를 지원합니다. 엔진은 새 프레임을 그려야 할 때마다 합성된 장면을 래스터화하는 역할을 담당합니다. 이것은 Flutter의 핵심 API에 대한 저수준 구현을 제공하며, 그래픽(Impeller는 iOS에서 사용되며 Android와 macOS로 확장 예정이고, 다른 플랫폼에서는 Skia를 사용), 텍스트 레이아웃, 파일 및 네트워크 I/O, 접근성 지원, 플러그인 아키텍처, 그리고 Dart 런타임 및 컴파일 도구 체인을 포함합니다.

엔진은 Dart 클래스의 기본 C++ 코드를 래핑하는 dart:ui를 통해 Flutter 프레임워크에 노출됩니다. 이 라이브러리는 입력, 그래픽 및 텍스트 렌더링 하위 시스템을 구동하기 위한 클래스와 같은 가장 낮은 수준의 기본 요소를 노출합니다.

일반적으로 개발자는 Dart 언어로 작성된 현대적인 반응형 프레임워크를 제공하는 Flutter 프레임워크를 통해 Flutter와 상호 작용합니다. 여기에는 일련의 레이어로 구성된 풍부한 플랫폼, 레이아웃 및 기본 라이브러리 세트가 포함되어 있습니다. 아래에서 위로 작업하면 다음과 같은 이점이 있습니다.

- 기본적인 클래스와 일반적으로 사용되는 추상화를 제공하는 애니메이션, 페인팅, 제스처 등의 블록을 구성하는 서비스를 가집니다.

- 렌더링 레이어는 레이아웃 처리를 위한 추상화를 제공합니다. 이 레이어를 사용하면 렌더링 가능한 객체의 트리를 구축할 수 있습니다. 변경 사항을 반영하기 위해 트리가 레이아웃을 자동으로 업데이트하여 이러한 객체를 동적으로 조작할 수 있습니다.

- 위젯 레이어는 구성 추상화를 제공합니다. 렌더링 계층의 각 렌더링 객체에는 위젯 레이어에 대응하는 클래스가 있습니다. 또한 위젯 레이어에서는 재사용할 수 있는 클래스 조합을 정의할 수 있습니다. 이는 반응형 프로그래밍 모델이 도입되는 계층입니다.

- Material 및 Cupertino 라이브러리는 위젯 레이어의 구성 원칙을 사용하여 Material 또는 iOS 디자인 언어를 구현하는 포괄적인 컨트롤 세트를 제공합니다.

Flutter 프레임워크는 상대적으로 작습니다. 개발자가 사용할 수 있는 많은 고수준 기능은 카메라 및 웹뷰와 같은 플랫폼 플러그인뿐만 아니라 핵심 Dart 및 Flutter 라이브러리를 기반으로 구축된 캐릭터, http 및 애니메이션과 같은 플랫폼에 구애받지 않는 기능을 포함하여 패키지로 구현됩니다. 이러한 패키지 중 일부는 인앱 결제, Apple 인증 및 애니메이션과 같은 서비스를 포함하는 더 넓은 생태계에서 제공됩니다.

이 개요의 나머지 부분에서는 UI 개발의 반응형 패러다임부터 시작하여 계층 아래로 광범위하게 탐색합니다. 그런 다음 위젯이 어떻게 함께 구성되고 애플리케이션의 일부로 렌더링될 수 있는 객체로 변환되는지 설명합니다. Flutter의 웹 지원이 다른 대상과 어떻게 다른지에 대한 간략한 요약을 제공하기 전에 Flutter가 플랫폼 수준에서 다른 코드와 어떻게 상호 운용되는지 설명합니다.

## 앱 분석 Anatomy of an app

다음 다이어그램은 flutter create로 생성된 일반 Flutter 앱을 구성하는 부분의 개요를 제공합니다. Flutter Engine이 이 스택에 있는 위치를 보여주고, API 경계를 강조 표시하며, 개별 조각이 있는 저장소를 식별합니다. 아래 범례는 Flutter 앱의 일부를 설명하는 데 일반적으로 사용되는 일부 용어를 명확히 설명합니다.

##### 다트 앱 Dart App

- 위젯을 원하는 UI로 구성합니다.

- 비즈니스 로직을 구현합니다.

- 앱 개발자가 소유합니다.

##### 프레임워크 Framework

- 고품질 앱(예: 위젯, 적중 테스트, 동작 감지, 접근성, 텍스트 입력)을 구축하기 위한 더 높은 수준의 API를 제공합니다.

- 앱의 위젯 트리를 장면에 합성합니다.

##### 엔진 Engine

- 합성된 장면을 래스터화하는 일을 담당합니다.

- Flutter의 핵심 API(예: 그래픽, 텍스트 레이아웃, Dart 런타임)의 저수준 구현을 제공합니다.

- dart:ui API를 사용하여 해당 기능을 프레임워크에 노출합니다.

- 엔진의 Embedder API를 사용하여 특정 플랫폼과 통합됩니다.

##### 임베더 Embedder

- 렌더링 표면, 접근성 및 입력과 같은 서비스에 대한 액세스를 위해 기본 운영 체제와 조정합니다.

- 이벤트 루프를 관리합니다.

- Embedder를 앱에 통합하기 위해 플랫폼별 API를 노출합니다.

##### 러너 Runner

- 임베더의 플랫폼별 API에 의해 노출된 조각을 대상 플랫폼에서 실행 가능한 앱 패키지로 구성합니다.

- 앱 개발자가 소유한 flutter create로 생성된 앱 템플릿의 일부입니다.

## 반응형 사용자 인터페이스 Reactive user interfaces

표면적으로 볼 때, Flutter는 반응형, 선언형 UI 프레임워크입니다. 이 프레임워크에서 개발자는 애플리케이션 상태를 인터페이스 상태로 매핑하는 작업을 제공하며, 애플리케이션 상태가 변경될 때 런타임에서 인터페이스를 업데이트하는 작업은 프레임워크가 수행합니다. 이 모델은 Facebook이 자체 React 프레임워크를 위해 만든 작업에서 영감을 받았으며, 이는 많은 전통적인 설계 원칙들을 재고하게 만들었습니다.

대부분의 전통적인 UI 프레임워크에서는 사용자 인터페이스의 초기 상태를 한 번 설명한 후, 이벤트에 따라 런타임에서 사용자 코드에 의해 별도로 업데이트됩니다. 이 접근 방식의 한 가지 문제는 애플리케이션이 복잡해짐에 따라 상태 변화가 전체 UI에 걸쳐 어떻게 연쇄적으로 영향을 미치는지 개발자가 인지해야 한다는 점입니다. 예를 들어, 다음과 같은 UI를 생각해 보십시오:

상태가 변경될 수 있는 장소가 많이 있습니다: 색상 박스, 색조 슬라이더, 라디오 버튼 등이 그 예입니다. 사용자가 UI와 상호작용할 때, 이러한 변경 사항은 다른 모든 부분에 반영되어야 합니다. 더 나쁜 것은, 주의하지 않으면 사용자 인터페이스의 한 부분에 대한 작은 변경이 겉보기에는 관련이 없어 보이는 코드에까지 파급 효과를 일으킬 수 있다는 점입니다.

이 문제에 대한 한 가지 해결책은 MVC와 같은 접근 방식입니다. 이 방식에서는 데이터를 컨트롤러를 통해 모델에 푸시하고, 그런 다음 모델이 새로운 상태를 컨트롤러를 통해 뷰로 푸시합니다. 그러나 이 방법 또한 문제가 될 수 있습니다. UI 요소를 생성하고 업데이트하는 작업이 두 개의 별도 단계로 나뉘어 있어 쉽게 동기화되지 않을 수 있기 때문입니다.

Flutter와 다른 반응형 프레임워크들은 이 문제에 대해 대안적인 접근 방식을 취합니다. 즉, 사용자 인터페이스를 그 기저 상태와 명시적으로 분리하는 것입니다. React 스타일의 API를 사용하면 UI 설명만 생성하면 되고, 프레임워크가 그 하나의 설정을 사용하여 사용자 인터페이스를 적절하게 생성하거나 업데이트하는 작업을 처리합니다.

Flutter에서는 위젯(React의 컴포넌트와 유사)이 불변 클래스에 의해 표현되며, 이 클래스들은 객체의 트리를 구성하는 데 사용됩니다. 이러한 위젯들은 레이아웃을 위한 별도의 객체 트리를 관리하는 데 사용되며, 이후에는 합성을 위한 또 다른 객체 트리를 관리하는 데 사용됩니다. Flutter의 핵심은 수정된 트리의 부분을 효율적으로 탐색하고, 객체의 트리를 더 낮은 수준의 객체 트리로 변환하며, 이러한 트리 전반에 걸쳐 변경 사항을 전파하는 일련의 메커니즘입니다.

위젯은 build() 메서드를 재정의하여 사용자 인터페이스를 선언합니다. 이 메서드는 상태를 UI로 변환하는 함수입니다.

```
UI = f(state)
```

build() 메서드는 설계상 실행 속도가 빠르며 부작용이 없어야 하며, 이를 통해 프레임워크는 필요할 때마다(잠재적으로 렌더링된 프레임당 한 번씩) 이 메서드를 호출할 수 있습니다.

이 접근 방식은 언어 런타임의 특정 특성(특히, 빠른 객체 인스턴스화 및 삭제)에 의존합니다. 다행히도 Dart는 이러한 작업에 특히 적합합니다.

## 위젯 Widgets

앞서 언급했듯이, Flutter는 위젯을 구성의 단위로 강조합니다. 위젯은 Flutter 앱의 사용자 인터페이스를 구성하는 기본 요소이며, 각 위젯은 사용자 인터페이스의 일부를 불변적으로 선언한 것입니다.

위젯은 구성 기반의 계층 구조를 형성합니다. 각 위젯은 부모 위젯 안에 중첩되어 있으며, 부모로부터 컨텍스트를 받을 수 있습니다. 이 구조는 루트 위젯(일반적으로 MaterialApp이나 CupertinoApp)까지 계속 이어집니다. 다음은 이 단순한 예제입니다:

```dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(
          title: const Text('My Home Page'),
        ),
        body: Center(
          child: Builder(
            builder: (context) {
              return Column(
                children: [
                  const Text('Hello World'),
                  const SizedBox(height: 20),
                  ElevatedButton(
                    onPressed: () {
                      print('Click!');
                    },
                    child: const Text('A button'),
                  ),
                ],
              );
            },
          ),
        ),
      ),
    );
  }
}
```

앞선 코드에서는 모든 인스턴스화된 클래스가 위젯입니다.

앱은 이벤트(예: 사용자 상호작용)에 응답하여 사용자 인터페이스를 업데이트합니다. 이를 위해 프레임워크에 위젯 계층에서 기존 위젯을 다른 위젯으로 교체하라고 지시합니다. 프레임워크는 새 위젯과 기존 위젯을 비교하고, 효율적으로 사용자 인터페이스를 업데이트합니다.

Flutter는 시스템에서 제공하는 UI 컨트롤을 사용하는 대신, 자체적으로 각 UI 컨트롤을 구현합니다. 예를 들어, iOS의 Toggle 컨트롤과 Android의 해당 컨트롤 모두 순수 Dart로 구현되어 있습니다.

이 접근 방식은 여러 가지 이점을 제공합니다:

- 무제한 확장성 제공: Switch 컨트롤의 변형을 원하는 개발자는 OS에서 제공하는 확장 지점에 제한받지 않고 임의의 방식으로 하나를 만들 수 있습니다.

- 성능 병목 현상 방지: Flutter가 전체 장면을 한 번에 합성할 수 있게 해줍니다. 이로 인해 Flutter 코드와 플랫폼 코드 간에 왔다 갔다 하는 전환 없이 작업할 수 있어 성능 병목 현상을 피할 수 있습니다.

- 운영 체제 의존성 분리: 애플리케이션의 동작이 운영 체제의 의존성에서 분리됩니다. 애플리케이션은 OS의 버전이 변경되더라도 모든 버전에서 동일한 모양과 느낌을 유지합니다.

## 구성 Composition

위젯은 일반적으로 많은 작은 단일 목적의 위젯들로 구성되어 강력한 효과를 만들어냅니다.

가능한 경우, 디자인 개념의 수는 최소화하면서 전체 어휘는 넓게 유지됩니다. 예를 들어, 위젯 레이어에서 Flutter는 화면 그리기, 레이아웃(위치 지정 및 크기 조정), 사용자 상호작용, 상태 관리, 테마 설정, 애니메이션, 네비게이션을 모두 같은 핵심 개념(위젯)을 사용하여 표현합니다. 애니메이션 레이어에서는 두 개의 개념(애니메이션과 트윈)이 대부분의 디자인 영역을 커버합니다. 렌더링 레이어에서는 RenderObject가 레이아웃, 페인팅, 히트 테스트, 접근성을 설명하는 데 사용됩니다. 이러한 각 경우에서, 해당 어휘는 결국 방대해지며, 수백 개의 위젯과 렌더 오브젝트, 수십 개의 애니메이션 및 트윈 유형이 존재합니다.

클래스 계층 구조는 의도적으로 얕고 넓게 설계되어 가능한 조합의 수를 극대화하며, 각 위젯이 하나의 일을 잘 수행하는 작은 조합 가능한 위젯에 집중합니다. 핵심 기능은 추상화되어 있으며, 패딩과 정렬과 같은 기본 기능조차도 핵심에 내장되지 않고 별도의 컴포넌트로 구현됩니다. (이는 패딩과 같은 기능이 모든 레이아웃 컴포넌트의 공통 핵심에 내장된 더 전통적인 API와 대조됩니다.) 예를 들어, 위젯을 중앙에 배치하려면 개념적인 Align 속성을 조정하는 대신, Center 위젯으로 감싸서 사용합니다.

패딩, 정렬, 행, 열, 그리드용 위젯이 있습니다. 이러한 레이아웃 위젯은 자체적으로 시각적 표현을 가지지 않습니다. 대신, 그들의 유일한 목적은 다른 위젯의 레이아웃의 일부 측면을 제어하는 것입니다. Flutter는 이러한 구성적 접근 방식을 활용하는 유틸리티 위젯도 포함하고 있습니다.

## 위젯 만들기 Building widgets

앞서 언급했듯이, 위젯의 시각적 표현은 build() 함수를 재정의하여 새로운 엘리먼트 트리를 반환함으로써 결정됩니다. 이 트리는 위젯의 사용자 인터페이스 일부를 더 구체적으로 나타냅니다. 예를 들어, 툴바 위젯은 build 함수가 텍스트와 여러 버튼을 포함한 수평 레이아웃을 반환할 수 있습니다. 필요에 따라, 프레임워크는 각 위젯에게 재귀적으로 빌드를 요청하여 트리가 완전히 구체적인 렌더링 가능한 객체들로 설명될 때까지 진행합니다. 그 후, 프레임워크는 렌더링 가능한 객체들을 모아 렌더링 가능한 객체 트리로 엮어냅니다.

위젯의 build 함수는 부작용이 없어야 합니다. 함수가 호출될 때마다 위젯은 이전에 반환했던 것과 관계없이 새로운 위젯 트리를 반환해야 합니다. 프레임워크는 렌더링 객체 트리를 기반으로 어떤 build 메서드가 호출되어야 하는지 결정하는 무거운 작업을 수행합니다(자세한 설명은 후에 다룰 예정입니다). 이 과정에 대한 더 많은 정보는 Inside Flutter 주제에서 확인할 수 있습니다.

각 렌더링된 프레임에서 Flutter는 상태가 변경된 UI의 부분만을 재생성할 수 있습니다. 이를 위해 해당 위젯의 build() 메서드를 호출합니다. 따라서 build 메서드는 신속하게 반환되어야 하며, 무거운 계산 작업은 비동기적으로 수행한 후 상태의 일부로 저장하여 build 메서드에서 사용해야 합니다.

비교적 단순한 접근 방식이지만, 이 자동화된 비교는 매우 효과적이며, 높은 성능의 인터랙티브한 앱을 가능하게 합니다. build 함수의 디자인은 위젯이 무엇으로 구성되어 있는지를 선언하는 데 집중함으로써, 상태를 변경하는 복잡성에서 벗어나 코드 작성을 단순화합니다.

## 위젯 상태 Widget state

프레임워크는 두 가지 주요 위젯 클래스를 도입합니다: 상태가 있는 위젯(StatefulWidget)과 상태가 없는 위젯(StatelessWidget)입니다.

많은 위젯은 변경 가능한 상태가 없습니다. 즉, 시간이 지남에 따라 변하는 속성이 없으며(예를 들어, 아이콘이나 레이블), 이러한 위젯은 StatelessWidget의 하위 클래스가 됩니다.

그러나 위젯의 고유한 특성이 사용자 상호작용이나 다른 요소에 따라 변경되어야 한다면, 그 위젯은 상태를 갖는 위젯(StatefulWidget)입니다. 예를 들어, 사용자가 버튼을 탭할 때마다 카운터가 증가하는 위젯이 있다면, 이 카운터의 값이 해당 위젯의 상태가 됩니다. 이 값이 변경되면, 위젯은 UI의 해당 부분을 업데이트하기 위해 다시 빌드되어야 합니다. 이러한 위젯은 StatefulWidget의 하위 클래스가 되며, 위젯 자체는 불변이므로 변경 가능한 상태는 State의 하위 클래스인 별도의 클래스에 저장됩니다. StatefulWidget 자체는 build 메서드를 가지지 않고, 대신 State 객체를 통해 사용자 인터페이스가 구축됩니다.

State 객체를 변경할 때(예를 들어, 카운터를 증가시키는 경우), setState()를 호출하여 프레임워크에 사용자 인터페이스를 업데이트하도록 신호를 보내야 합니다. 이렇게 하면 프레임워크는 State의 build 메서드를 다시 호출하여 UI를 업데이트합니다.

별도의 상태와 위젯 객체를 갖는 것은 다른 위젯들이 상태가 없는 위젯과 상태가 있는 위젯을 동일하게 다룰 수 있게 해줍니다. 상태를 잃어버릴 걱정 없이 말이죠. 부모 위젯은 자식 위젯의 상태를 유지하기 위해 자식 위젯을 계속 보유할 필요 없이, 언제든지 자식의 새로운 인스턴스를 생성할 수 있습니다. 프레임워크는 적절할 때 기존 상태 객체를 찾아 재사용하는 모든 작업을 처리합니다.

## 상태 관리 State management

그래서 많은 위젯이 상태를 포함할 수 있다면, 상태는 어떻게 관리되고 시스템을 통해 전달될까요?

다른 클래스와 마찬가지로, 위젯의 생성자를 사용하여 데이터를 초기화할 수 있습니다. 이를 통해 build() 메서드는 자식 위젯이 필요한 데이터를 가지고 인스턴스화되도록 보장할 수 있습니다.

```dart
@override
Widget build(BuildContext context) {
   return ContentWidget(importantState);
}
```

importantState는 위젯에 중요한 상태를 포함하는 클래스를 나타내는 자리 표시자입니다.

하지만 위젯 트리가 깊어지면, 상태 정보를 트리의 상위 및 하위로 전달하는 것이 번거로울 수 있습니다. 이 문제를 해결하기 위해, 세 번째 위젯 유형인 InheritedWidget이 도입되었습니다. InheritedWidget을 사용하면 위젯 트리의 공통 조상을 감싸는 상태 위젯을 쉽게 생성할 수 있습니다. 다음 예제에서 이를 확인할 수 있습니다:

ExamWidget 또는 GradeWidget 객체가 StudentState로부터 데이터를 필요로 할 때, 이제 다음과 같은 명령어를 사용하여 접근할 수 있습니다:

```dart
final studentState = StudentState.of(context);
```

of(context) 호출은 빌드 컨텍스트(현재 위젯 위치에 대한 핸들)를 받아들여, 트리에서 StudentState 타입과 일치하는 가장 가까운 조상을 반환합니다. InheritedWidget은 또한 updateShouldNotify() 메서드를 제공하며, 이 메서드는 Flutter가 상태 변경이 자식 위젯의 재빌드를 트리거해야 하는지를 결정하는 데 사용됩니다.

Flutter 자체는 프레임워크의 일부로서 InheritedWidget을 광범위하게 사용하여 공유 상태를 관리합니다. 예를 들어, 애플리케이션의 시각적 테마는 색상 및 타입 스타일과 같은 속성을 포함하며, 이는 애플리케이션 전반에 걸쳐 퍼져 있습니다. MaterialApp의 build() 메서드는 빌드할 때 트리에 테마를 삽입하고, 그 후 더 깊은 계층의 위젯은 .of() 메서드를 사용하여 관련 테마 데이터를 조회할 수 있습니다.

예를 들어:

```dart
Container(
  color: Theme.of(context).secondaryHeaderColor,
  child: Text(
    'Text with a background color',
    style: Theme.of(context).textTheme.titleLarge,
  ),
);
```

애플리케이션이 커짐에 따라, 상태가 있는 위젯을 생성하고 사용하는 복잡성을 줄이는 더 발전된 상태 관리 접근 방식이 더 매력적으로 다가옵니다. 많은 Flutter 앱은 provider와 같은 유틸리티 패키지를 사용하며, 이는 InheritedWidget을 감싸는 래퍼를 제공합니다. 또한, Flutter의 계층화된 아키텍처는 flutter_hooks 패키지와 같은 대체 접근 방식을 통해 상태를 UI로 변환하는 작업을 구현할 수 있게 해줍니다.

## 렌더링과 레이아웃 Rendering and layout

이 섹션은 Flutter가 위젯 계층 구조를 실제 화면에 그려지는 픽셀로 변환하는 일련의 단계를 설명하는 렌더링 파이프라인에 대해 다룹니다.

## Flutter의 렌더링 모델 Flutter's rendering model

Flutter가 크로스 플랫폼 프레임워크라면 어떻게 단일 플랫폼 프레임워크와 유사한 성능을 제공할 수 있을까요?

전통적인 Android 앱이 어떻게 작동하는지 생각해 보는 것이 도움이 될 수 있습니다. 그릴 때, 먼저 Android 프레임워크의 Java 코드를 호출합니다. Android 시스템 라이브러리는 Canvas 객체에 그려지는 컴포넌트를 제공하고, Android는 이를 Skia라는 C/C++로 작성된 그래픽 엔진을 통해 렌더링하여 CPU 또는 GPU를 사용해 기기에서 그림을 그립니다.

크로스 플랫폼 프레임워크는 일반적으로 기본 Android 및 iOS UI 라이브러리에 대한 추상화 계층을 만들어 각 플랫폼의 불일치를 해결하려고 시도합니다. 앱 코드는 종종 JavaScript와 같은 인터프리트 언어로 작성되며, 이는 Java 기반의 Android 또는 Objective-C 기반의 iOS 시스템 라이브러리와 상호작용해야 하므로 UI와 앱 로직 간의 많은 상호작용이 있을 때 오버헤드가 상당할 수 있습니다.

반면, Flutter는 이러한 추상화를 최소화하여 시스템 UI 위젯 라이브러리를 우회하고 자체 위젯 세트를 사용합니다. Flutter의 시각적 요소를 그리는 Dart 코드는 네이티브 코드로 컴파일되며, Skia(또는 미래에는 Impeller)를 사용해 렌더링합니다. 또한, Flutter는 Skia의 사본을 엔진 일부로 포함하고 있어 개발자는 휴대폰이 새 Android 버전으로 업데이트되지 않더라도 최신 성능 향상을 적용할 수 있습니다. 이는 Windows나 macOS와 같은 다른 네이티브 플랫폼에서도 마찬가지입니다.

```
노트

Flutter 3.10부터 iOS에서는 Impeller가 기본 렌더링 엔진으로 설정되었습니다. Android에서는 enable-impeller 플래그 뒤에서 Impeller를 미리 볼 수 있습니다. 자세한 내용은 Impeller 렌더링 엔진을 참조하세요.
```

## 사용자 입력에서 GPU까지 From user input to the GPU

Flutter는 "단순함은 빠르다"는 원칙을 렌더링 파이프라인에 적용합니다. Flutter는 데이터가 시스템으로 흘러가는 간단한 파이프라인을 가지고 있으며, 다음 순서도에 그 흐름이 나와 있습니다.

이제 이러한 단계 중 몇 가지를 자세히 살펴보겠습니다.

## Build: 위젯에서 엘리먼트까지

다음 코드는 위젯 계층 구조를 보여줍니다:

```dart
Container(
  color: Colors.blue,
  child: Row(
    children: [
      Image.network('https://www.example.com/1.png'),
      const Text('A'),
    ],
  ),
);
```

Flutter가 이 코드를 렌더링할 때, build() 메서드를 호출해 현재 앱 상태를 기반으로 UI를 렌더링하는 위젯 하위 트리를 반환합니다. 이 과정에서 build() 메서드는 상태에 따라 필요한 경우 새로운 위젯을 도입할 수 있습니다. 예를 들어, 위의 코드에서 Container는 color와 child 속성을 가지고 있습니다. Container의 소스 코드를 보면 color가 null이 아니면 색을 나타내는 ColoredBox를 삽입하는 것을 알 수 있습니다:

```dart
if (color != null)
  current = ColoredBox(color: color!, child: current);
```

마찬가지로 Image와 Text 위젯은 RawImage 및 RichText와 같은 자식 위젯을 빌드 과정에서 삽입할 수 있습니다. 따라서 최종 위젯 계층 구조는 코드에서 나타난 것보다 더 깊어질 수 있습니다.

이는 Flutter Inspector와 같은 디버그 도구를 통해 트리를 검사할 때 코드에서 작성한 것보다 훨씬 깊은 구조를 볼 수 있는 이유를 설명합니다.

빌드 단계에서 Flutter는 코드에서 표현된 위젯을 대응하는 엘리먼트 트리로 변환하며, 트리의 각 위치마다 하나의 엘리먼트가 존재합니다. 각 엘리먼트는 트리 구조의 특정 위치에서 위젯의 특정 인스턴스를 나타냅니다. 엘리먼트는 기본적으로 두 가지 유형이 있습니다:

- ComponentElement: 다른 엘리먼트를 호스팅하는 역할.
- RenderObjectElement: 레이아웃이나 페인팅 단계에 참여하는 엘리먼트.

RenderObjectElement는 위젯의 아날로그와 기저 RenderObject 간의 중간 역할을 합니다.

각 위젯의 엘리먼트는 BuildContext를 통해 참조할 수 있으며, 이는 위젯의 트리에서의 위치에 대한 핸들입니다. Theme.of(context)와 같은 함수 호출에서 context는 이 빌드 컨텍스트이며, build() 메서드에 매개변수로 전달됩니다.

위젯은 불변이므로 부모-자식 관계도 변경할 수 없습니다. 위젯 트리에 대한 변경이 있으면(예: Text('A')를 Text('B')로 변경) 새 위젯 객체 세트가 반환됩니다. 그러나 기저 표현이 반드시 다시 구축되는 것은 아닙니다. 엘리먼트 트리는 프레임에서 프레임까지 지속되며, 성능 상 중요한 역할을 합니다. Flutter는 전체 위젯 계층 구조를 일회용으로 취급하면서도 기저 표현을 캐시하여, 변경된 위젯만 다시 빌드함으로써 엘리먼트 트리의 필요한 부분만 재구성할 수 있습니다.

## 레이아웃과 렌더링 Layout and rendering

하나의 위젯만 그리는 애플리케이션은 매우 드뭅니다. 따라서 모든 UI 프레임워크에서 중요한 부분은 위젯 계층 구조를 효율적으로 배치하고, 각 요소의 크기와 위치를 화면에 렌더링하기 전에 결정하는 능력입니다.

렌더 트리의 모든 노드의 기본 클래스는 RenderObject이며, 이는 레이아웃 및 페인팅을 위한 추상 모델을 정의합니다. 이는 매우 일반적이어서 고정된 차원 수나 심지어 데카르트 좌표계에도 고정되지 않습니다. 각 RenderObject는 부모를 알고 있지만, 자식에 대해서는 방문 방법과 제약 조건 외에는 거의 알지 못합니다. 이는 다양한 사용 사례를 처리할 수 있도록 충분한 추상화를 제공합니다.

빌드 단계에서 Flutter는 엘리먼트 트리의 RenderObjectElement마다 RenderObject에서 상속된 객체를 생성하거나 업데이트합니다. RenderObject는 기본 요소로, RenderParagraph는 텍스트를, RenderImage는 이미지를, RenderTransform은 자식을 페인팅하기 전에 변환을 적용합니다.

대부분의 Flutter 위젯은 2D 데카르트 공간에서 고정 크기의 RenderObject를 나타내는 RenderBox 하위 클래스를 상속받는 객체에 의해 렌더링됩니다. RenderBox는 박스 제약 모델의 기초를 제공하며, 각 위젯이 렌더링될 최소 및 최대 너비와 높이를 설정합니다.

레이아웃을 수행하기 위해 Flutter는 렌더 트리를 깊이 우선 탐색 방식으로 순회하며 부모에서 자식에게 크기 제약을 전달합니다. 자식은 자신의 크기를 결정할 때 부모가 설정한 제약을 준수해야 하며, 자식은 부모가 설정한 제약 내에서 자신의 크기를 부모 객체에게 전달하여 응답합니다.

이 트리를 한 번 순회한 후, 모든 객체는 부모의 제약 내에서 정의된 크기를 가지게 되며, paint() 메서드를 호출하여 그릴 준비가 됩니다.

박스 제약 모델은 객체를 O(n) 시간 내에 배치하는 매우 강력한 방법입니다.

- 부모는 최대 및 최소 제약을 동일한 값으로 설정하여 자식 객체의 크기를 지정할 수 있습니다. 예를 들어, 휴대폰 앱의 최상위 렌더 객체는 자식을 화면 크기로 제한합니다. (자식은 그 공간을 어떻게 사용할지 선택할 수 있습니다. 예를 들어, 주어진 제약 내에서 렌더링할 항목을 중앙에 배치할 수 있습니다.)

- 부모는 자식의 너비를 지정하면서 높이에 대해서는 유연성을 줄 수 있고(또는 높이를 지정하고 너비에 유연성을 줄 수 있습니다). 실제 예로는 흐르는 텍스트가 있는데, 이는 가로 제약에 맞춰야 하지만 텍스트의 양에 따라 세로로는 변동될 수 있습니다.

이 모델은 자식 객체가 사용할 수 있는 공간의 크기를 알아야 콘텐츠를 렌더링할 방식을 결정할 때도 작동합니다. LayoutBuilder 위젯을 사용하면, 자식 객체가 전달된 제약을 확인하고 이를 사용하여 렌더링 방식을 결정할 수 있습니다. 예를 들어:

```dart
Widget build(BuildContext context) {
  return LayoutBuilder(
    builder: (context, constraints) {
      if (constraints.maxWidth < 600) {
        return const OneColumnLayout();
      } else {
        return const TwoColumnLayout();
      }
    },
  );
}
```

제약 및 레이아웃 시스템에 대한 자세한 정보와 예제는 [제약 이해하기] 주제에서 확인할 수 있습니다.

모든 RenderObject의 루트는 RenderView로, 이는 렌더 트리의 전체 출력을 나타냅니다. 플랫폼이 새 프레임을 렌더링해야 할 때(예: vsync 또는 텍스처 압축/업로드가 완료되었을 때), 렌더 트리의 루트에 있는 RenderView 객체의 일부인 compositeFrame() 메서드가 호출됩니다. 이는 장면 업데이트를 트리거하는 SceneBuilder를 생성합니다. 장면이 완료되면 RenderView 객체는 합성된 장면을 dart:ui의 Window.render() 메서드에 전달하고, 이 메서드는 GPU에 제어를 넘겨 렌더링을 수행합니다.

파이프라인의 구성 및 래스터화 단계에 대한 자세한 내용은 이 고수준 문서의 범위를 벗어나지만, Flutter 렌더링 파이프라인에 대한 자세한 정보는 관련 강연에서 확인할 수 있습니다.

## 플랫폼 임베딩 Platform embedding

봤듯이, Flutter 사용자 인터페이스는 해당 OS의 위젯으로 변환되는 대신 Flutter 자체에 의해 구축되고, 레이아웃이 설정되며, 합성되고, 그려집니다. 각 운영 체제의 텍스처를 얻고 앱 생명 주기에 참여하는 메커니즘은 해당 플랫폼의 고유한 요구 사항에 따라 다를 수밖에 없습니다. Flutter 엔진은 플랫폼에 종속되지 않으며, 안정적인 ABI(응용 프로그램 이진 인터페이스)를 제공하여 플랫폼 임베더가 Flutter를 설정하고 사용할 수 있게 합니다.

플랫폼 임베더는 모든 Flutter 콘텐츠를 호스팅하는 네이티브 OS 애플리케이션으로, 호스트 운영 체제와 Flutter 사이에서 접착제 역할을 합니다. Flutter 앱을 시작할 때 임베더는 진입점을 제공하고, Flutter 엔진을 초기화하며, UI와 래스터 작업을 위한 스레드를 확보하고, Flutter가 쓸 수 있는 텍스처를 만듭니다. 또한 임베더는 마우스, 키보드, 터치 같은 입력 제스처, 창 크기 조정, 스레드 관리, 플랫폼 메시지 등의 앱 생명 주기를 관리합니다. Flutter는 Android, iOS, Windows, macOS, Linux용 플랫폼 임베더를 포함하며, VNC 스타일의 프레임버퍼를 통해 Flutter 세션을 원격으로 지원하는 예시나 Raspberry Pi용 예시처럼 사용자 정의 플랫폼 임베더를 만들 수도 있습니다.

각 플랫폼은 고유한 API와 제약 조건을 가지고 있습니다. 다음은 간단한 플랫폼별 참고 사항입니다:

- iOS와 macOS에서 Flutter는 각각 UIViewController 또는 NSViewController로 임베더에 로드됩니다. 플랫폼 임베더는 FlutterEngine을 생성하며, 이는 Dart VM과 Flutter 런타임을 호스팅하는 역할을 합니다. 또한 FlutterViewController는 FlutterEngine에 연결되어 UIKit이나 Cocoa의 입력 이벤트를 Flutter로 전달하고, FlutterEngine이 Metal 또는 OpenGL을 사용해 렌더링한 프레임을 표시합니다.

- Android에서는 Flutter가 기본적으로 Activity로 임베더에 로드됩니다. FlutterView가 뷰를 제어하며, Flutter 콘텐츠는 구성 및 Z-순서 요구 사항에 따라 뷰 또는 텍스처로 렌더링됩니다. FlutterView는 Flutter 콘텐츠를 표시하는 역할을 하며, 이를 통해 앱 화면에 Flutter UI가 표시됩니다.

- Windows에서는 Flutter가 전통적인 Win32 애플리케이션으로 호스팅되며, 콘텐츠는 ANGLE(Almost Native Graphics Layer Engine)을 사용하여 렌더링됩니다. ANGLE은 OpenGL API 호출을 DirectX 11에 해당하는 호출로 변환하는 라이브러리로, Windows 환경에서 Flutter가 원활하게 그래픽을 처리할 수 있게 해줍니다.

## 다른 코드와 통합 Integrating with other code

Flutter는 다양한 상호 운용성 메커니즘을 제공합니다. Kotlin이나 Swift 같은 언어로 작성된 코드나 API에 접근하거나, 네이티브 C 기반 API를 호출하거나, Flutter 앱에 네이티브 컨트롤을 임베드하거나, 기존 애플리케이션에 Flutter를 임베드하는 경우에도 이러한 메커니즘을 사용할 수 있습니다.

## 플랫폼 채널 Platform channel

모바일 및 데스크톱 앱에서 Flutter는 플랫폼 채널을 통해 사용자 정의 코드를 호출할 수 있도록 합니다. 이는 Dart 코드와 호스트 앱의 플랫폼별 코드 간에 통신할 수 있는 메커니즘입니다. 공통 채널(이름과 코덱을 캡슐화)을 생성하면, Dart와 Kotlin이나 Swift 같은 언어로 작성된 플랫폼 구성 요소 간에 메시지를 주고받을 수 있습니다. 데이터는 Dart의 Map 같은 타입에서 표준 형식으로 직렬화된 후, Kotlin의 HashMap 또는 Swift의 Dictionary와 같은 동등한 표현으로 역직렬화됩니다.

다음은 Dart에서 Kotlin(안드로이드) 또는 Swift(iOS)에서 이벤트 핸들러를 호출하는 플랫폼 채널의 짧은 예시입니다:

```dart
// Dart side
const channel = MethodChannel('foo');
final greeting = await channel.invokeMethod('bar', 'world') as String;
print(greeting);
```

```java
// Android (Kotlin)
val channel = MethodChannel(flutterView, "foo")
channel.setMethodCallHandler { call, result ->
  when (call.method) {
    "bar" -> result.success("Hello, ${call.arguments}")
    else -> result.notImplemented()
  }
}
```

```swift
// iOS (Swift)
let channel = FlutterMethodChannel(name: "foo", binaryMessenger: flutterView)
channel.setMethodCallHandler {
  (call: FlutterMethodCall, result: FlutterResult) -> Void in
  switch (call.method) {
    case "bar": result("Hello, \(call.arguments as! String)")
    default: result(FlutterMethodNotImplemented)
  }
}
```

플랫폼 채널을 사용하는 추가 예제는 flutter/packages 리포지토리에서 찾을 수 있습니다. 또한 Firebase, 광고, 카메라 및 블루투스와 같은 장치 하드웨어와 같은 다양한 일반 시나리오를 다루는 수천 개의 Flutter 플러그인이 이미 제공되고 있습니다. 이러한 플러그인을 사용하면 다양한 기능을 손쉽게 구현할 수 있으며, 많은 커뮤니티 지원과 문서도 함께 제공됩니다.

## 외부 함수 인터페이스 Foreign Function Interface

C 기반 API, 현대 언어로 작성된 코드(예: Rust나 Go)에서 생성된 API를 포함하여, Dart는 dart:ffi 라이브러리를 사용하여 네이티브 코드에 바인딩하는 직접적인 메커니즘을 제공합니다. 외부 함수 인터페이스(FFI) 모델은 데이터 전달을 위해 직렬화가 필요하지 않기 때문에 플랫폼 채널보다 상당히 빠를 수 있습니다. 대신 Dart 런타임은 Dart 객체에 의해 지원되는 힙 메모리를 할당하고 정적으로 또는 동적으로 링크된 라이브러리에 대한 호출을 수행할 수 있는 기능을 제공합니다. FFI는 웹을 제외한 모든 플랫폼에서 사용할 수 있으며, 웹에서는 JS 상호 운용성 라이브러리와 package:web가 유사한 목적을 수행합니다.

FFI를 사용하려면 Dart와 비관리 메서드 서명 각각에 대한 typedef를 생성하고 Dart VM에 이들 간의 매핑을 지시해야 합니다. 다음은 전통적인 Win32 MessageBox() API를 호출하는 코드 조각의 예입니다:

```dart
import 'dart:ffi';
import 'package:ffi/ffi.dart'; // .toNativeUtf16() 확장 메서드를 포함합니다.

typedef MessageBoxNative = Int32 Function(
  IntPtr hWnd,
  Pointer<Utf16> lpText,
  Pointer<Utf16> lpCaption,
  Int32 uType,
);

typedef MessageBoxDart = int Function(
  int hWnd,
  Pointer<Utf16> lpText,
  Pointer<Utf16> lpCaption,
  int uType,
);

void exampleFfi() {
  final user32 = DynamicLibrary.open('user32.dll');
  final messageBox =
      user32.lookupFunction<MessageBoxNative, MessageBoxDart>('MessageBoxW');

  final result = messageBox(
    0, // 소유자 창 없음
    'Test message'.toNativeUtf16(), // 메시지
    'Window caption'.toNativeUtf16(), // 윈도우 타이틀
    0, // 확인 버튼만
  );
}
```

## 플러터 앱에서 네이티브 컨트롤 렌더링 Rendering native controls in a Flutter app

Flutter 콘텐츠는 텍스처에 그려지고 위젯 트리가 완전히 내부적으로 관리되기 때문에, Flutter의 내부 모델이나 Flutter 위젯 내에서 렌더링된 Android 뷰와 같은 것이 존재할 자리가 없습니다. 이는 기존 플랫폼 구성 요소(예: 브라우저 컨트롤)를 Flutter 앱에 포함하고자 하는 개발자에게는 문제가 될 수 있습니다.

Flutter는 이러한 문제를 해결하기 위해 플랫폼 뷰 위젯(AndroidView 및 UiKitView)을 도입하여 각 플랫폼에서 이러한 종류의 콘텐츠를 포함할 수 있게 합니다. 플랫폼 뷰는 다른 Flutter 콘텐츠와 통합될 수 있습니다. 이러한 각 위젯은 기본 운영 체제와의 중개 역할을 합니다. 예를 들어, 안드로이드에서 AndroidView는 세 가지 주요 기능을 수행합니다:

- 네이티브 뷰에서 렌더링된 그래픽 텍스처의 복사본을 만들고, 매 프레임이 그려질 때 Flutter 렌더링된 서피스의 일부로 구성하기 위해 Flutter에 제시합니다.

- 히트 테스트 및 입력 제스처에 응답하고, 이를 동등한 네이티브 입력으로 변환합니다.

- 접근성 트리의 아날로그를 생성하고, 네이티브 레이어와 Flutter 레이어 간에 명령과 응답을 전달합니다.

이러한 동기화와 관련하여 불가피하게 일정한 오버헤드가 발생합니다. 따라서 일반적으로 이 접근 방식은 Google Maps와 같이 Flutter에서 재구현하는 것이 실용적이지 않은 복잡한 컨트롤에 가장 적합합니다.

일반적으로 Flutter 앱은 플랫폼 테스트를 기반으로 build() 메서드에서 이러한 위젯을 인스턴스화합니다. 예를 들어, google_maps_flutter 플러그인의 경우:

```dart
if (defaultTargetPlatform == TargetPlatform.android) {
  return AndroidView(
    viewType: 'plugins.flutter.io/google_maps',
    onPlatformViewCreated: onPlatformViewCreated,
    gestureRecognizers: gestureRecognizers,
    creationParams: creationParams,
    creationParamsCodec: const StandardMessageCodec(),
  );
} else if (defaultTargetPlatform == TargetPlatform.iOS) {
  return UiKitView(
    viewType: 'plugins.flutter.io/google_maps',
    onPlatformViewCreated: onPlatformViewCreated,
    gestureRecognizers: gestureRecognizers,
    creationParams: creationParams,
    creationParamsCodec: const StandardMessageCodec(),
  );
}
return Text(
    '$defaultTargetPlatform is not yet supported by the maps plugin');
```

AndroidView 또는 UiKitView의 기본 네이티브 코드와의 통신은 일반적으로 이전에 설명한 플랫폼 채널 메커니즘을 사용하여 이루어집니다.

현재 플랫폼 뷰는 데스크톱 플랫폼에서 사용할 수 없지만, 이는 아키텍처적인 제한이 아니며, 향후 지원이 추가될 가능성이 있습니다.

## 상위 앱에서 플러터 콘텐츠 호스팅 Hosting Flutter content in a parent app

앞서 설명한 시나리오의 반대는 기존의 안드로이드 또는 iOS 앱에 Flutter 위젯을 임베드하는 것입니다. 이전 섹션에서 설명한 바와 같이, 모바일 장치에서 실행되는 새로 생성된 Flutter 앱은 안드로이드 Activity 또는 iOS UIViewController에서 호스팅됩니다. Flutter 콘텐츠는 동일한 임베딩 API를 사용하여 기존의 안드로이드 또는 iOS 앱에 임베드할 수 있습니다.

Flutter 모듈 템플릿은 쉽게 임베딩할 수 있도록 설계되었습니다. 기존의 Gradle 또는 Xcode 빌드 정의에 소스 의존성으로 임베드하거나, 모든 개발자가 Flutter를 설치할 필요 없이 Android Archive 또는 iOS Framework 이진 파일로 컴파일하여 사용할 수 있습니다.

Flutter 엔진은 Flutter 공유 라이브러리를 로드하고, Dart 런타임을 초기화하며, Dart isolate를 생성 및 실행하고, UI에 렌더링 서피스를 연결해야 하므로 초기화하는 데 짧은 시간이 소요됩니다. Flutter 콘텐츠를 표시할 때 UI 지연을 최소화하려면, 전체 앱 초기화 시퀀스 중에 Flutter 엔진을 초기화하거나 최소한 첫 번째 Flutter 화면 이전에 초기화하는 것이 좋습니다. 이렇게 하면 사용자가 첫 번째 Flutter 코드가 로드되는 동안 갑작스러운 중단을 경험하지 않게 됩니다. 또한, Flutter 엔진을 분리하면 여러 Flutter 화면 간에 재사용할 수 있으며, 필요한 라이브러리를 로드하는 데 드는 메모리 오버헤드를 공유할 수 있습니다.

기존 안드로이드 또는 iOS 앱에 Flutter가 어떻게 로드되는지에 대한 자세한 정보는 "로드 순서, 성능 및 메모리" 주제에서 확인할 수 있습니다.

## 플러터 웹 지원 Flutter web support

Flutter가 지원하는 모든 플랫폼에 일반적인 아키텍처 개념이 적용되지만, Flutter의 웹 지원에는 주목할 만한 몇 가지 고유한 특성이 있습니다.

Dart는 언어가 존재한 이래로 JavaScript로 컴파일되어 왔으며, 개발 및 프로덕션 용도로 최적화된 도구 체인을 갖추고 있습니다. 오늘날에도 많은 중요한 앱이 Dart에서 JavaScript로 컴파일되어 프로덕션 환경에서 실행되고 있으며, 그 중에는 Google Ads의 광고 도구도 포함됩니다. Flutter 프레임워크가 Dart로 작성되었기 때문에, 이를 JavaScript로 컴파일하는 것은 상대적으로 간단한 작업이었습니다.

그러나 C++로 작성된 Flutter 엔진은 웹 브라우저가 아닌 기본 운영 체제와 인터페이스하도록 설계되었습니다. 따라서 다른 접근 방식이 필요합니다. 웹에서 Flutter는 표준 브라우저 API 위에 엔진을 재구현하여 제공합니다. 현재 웹에서 Flutter 콘텐츠를 렌더링하기 위한 두 가지 옵션이 있습니다: HTML과 WebGL입니다. HTML 모드에서는 Flutter가 HTML, CSS, 캔버스 및 SVG를 사용합니다. WebGL로 렌더링할 때 Flutter는 WebAssembly로 컴파일된 Skia 버전인 CanvasKit을 사용합니다. HTML 모드는 최고의 코드 크기 특성을 제공하지만, CanvasKit은 브라우저의 그래픽 스택으로 가는 가장 빠른 경로를 제공하며, 네이티브 모바일 타겟에 비해 다소 높은 그래픽 충실도를 제공합니다.

웹 버전의 아키텍처 레이어 다이어그램은 다음과 같습니다:

Flutter가 실행되는 다른 플랫폼과 비교했을 때 가장 주목할 만한 차이점은 Flutter가 Dart 런타임을 제공할 필요가 없다는 것입니다. 대신, Flutter 프레임워크(및 작성한 코드)는 JavaScript로 컴파일됩니다. 또한 Dart는 모든 모드(JIT 대 AOT, 네이티브 대 웹 컴파일)에서 언어 의미상의 차이가 매우 적다는 점도 주목할 만합니다. 대부분의 개발자는 이러한 차이를 겪는 코드를 한 줄도 작성하지 않을 것입니다.

개발 중에 Flutter 웹은 증가형 컴파일을 지원하는 컴파일러인 dartdevc를 사용하여 핫 리스타트를 가능하게 합니다(현재 핫 리로드는 지원되지 않음). 반대로 웹용 프로덕션 앱을 만들 준비가 되었을 때는 Dart의 고도로 최적화된 프로덕션 JavaScript 컴파일러인 dart2js가 사용되어, Flutter 코어와 프레임워크를 애플리케이션과 함께 패키징하여 배포 가능한 최소화된 소스 파일로 만듭니다. 코드는 단일 파일로 제공되거나 지연 가져오기(deferred imports)를 통해 여러 파일로 분할될 수 있습니다.

## 추가 정보 Further information

Flutter 내부에 대한 더 많은 정보에 관심이 있는 분들을 위해, Inside Flutter 백서가 프레임워크의 설계 철학에 대한 유용한 안내서를 제공합니다.

1. 빌드 함수는 새로 고침된 트리를 반환하지만, 새로운 구성을 통합해야 할 경우에만 다른 것을 반환하면 됩니다. 만약 구성이 실제로 동일하다면, 동일한 위젯을 반환하면 됩니다.

2. 이는 읽기 쉽게 하기 위한 약간의 단순화입니다. 실제로는 트리가 더 복잡할 수 있습니다.

3. 이 접근 방식에는 몇 가지 제한 사항이 있습니다. 예를 들어, 투명도는 다른 Flutter 위젯과 같은 방식으로 플랫폼 뷰에서 합성되지 않습니다.

4. 하나의 예는 그림자입니다. 그림자는 DOM 동등 원시 객체로 근사화해야 하며, 이로 인해 다소 충실도가 저하됩니다.