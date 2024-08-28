# 임시 상태와 앱 상태 구별 Differentiate between ephemeral state and app state

이 문서에서는 앱 상태, 임시 상태 및 Flutter 앱에서 각 상태를 관리하는 방법을 소개합니다.

가장 넓은 의미에서 앱 상태는 앱이 실행될 때 메모리에 존재하는 모든 것입니다. 여기에는 앱의 자산, Flutter 프레임워크가 UI에 대해 유지하는 모든 변수, 애니메이션 상태, 텍스처, 글꼴 등이 포함됩니다. 상태에 대한 가장 광범위한 정의는 유효하지만 앱을 설계하는 데는 별로 유용하지 않습니다.

첫째, 일부 상태(예: 텍스처)도 관리하지 않습니다. 프레임워크가 이를 처리합니다. 따라서 상태에 대한 보다 유용한 정의는 "언제든지 UI를 다시 작성하기 위해 필요한 모든 데이터"입니다. 둘째, 사용자가 직접 관리하는 상태는 임시 상태와 앱 상태라는 두 가지 개념적 유형으로 구분될 수 있습니다.

## 임시 상태 Ephemeral state

임시 상태(UI 상태 또는 로컬 상태라고도 함)는 단일 위젯에 깔끔하게 포함할 수 있는 상태입니다.

이는 의도적으로 모호한 정의이므로 여기에 몇 가지 예가 있습니다.

- PageView의 현재 페이지
- 복잡한 애니메이션의 현재 진행 상황
- BottomNavigationBar에서 현재 선택된 탭

위젯 트리의 다른 부분은 이런 종류의 상태에 액세스할 필요가 거의 없습니다. 직렬화할 필요도 없고 복잡한 방식으로 변경되지도 않습니다.

즉, 이러한 종류의 상태에는 상태 관리 기술(ScopedModel, Redux 등)을 사용할 필요가 없습니다. 필요한 것은 StatefulWidget뿐입니다.

아래에서는 BottomNavigationBar에서 현재 선택한 항목이 _MyHomepageState 클래스의 _index 필드에 어떻게 보관되어 있는지 확인할 수 있습니다. 이 예에서 _index는 임시 상태입니다.

```dart
class MyHomepage extends StatefulWidget {
  const MyHomepage({super.key});

  @override
  State<MyHomepage> createState() => _MyHomepageState();
}

class _MyHomepageState extends State<MyHomepage> {
  int _index = 0;

  @override
  Widget build(BuildContext context) {
    return BottomNavigationBar(
      currentIndex: _index,
      onTap: (newIndex) {
        setState(() {
          _index = newIndex;
        });
      },
      // ... items ...
    );
  }
}
```

여기서 setState()와 StatefulWidget의 State 클래스 내부 필드를 사용하는 것은 완전히 자연스럽습니다. 앱의 다른 부분은 _index에 액세스할 필요가 없습니다. 변수는 MyHomepage 위젯 내부에서만 변경됩니다. 그리고 사용자가 앱을 닫았다가 다시 시작하면 _index가 0으로 재설정되어도 괜찮습니다.

## 앱 상태 App state

일시적이지 않고 앱의 여러 부분에서 공유하고 싶고 사용자 세션 간에 유지하려는 상태를 애플리케이션 상태(때때로 공유 상태라고도 함)라고 합니다.

애플리케이션 상태의 예시:

- 사용자 환경설정
- 로그인 정보
- 소셜 네트워킹 앱의 알림
- 전자상거래 앱의 장바구니
- 뉴스 앱의 기사 읽음/읽지 않음 상태

앱 상태를 관리하려면 옵션을 조사해야 합니다. 선택은 앱의 복잡성과 성격, 팀의 이전 경험, 기타 여러 측면에 따라 달라집니다. 계속 읽어보세요.

## 명확한 규칙은 없습니다 There is no clear-cut rule

명확하게 말하면 State 및 setState()를 사용하여 앱의 모든 상태를 관리할 수 있습니다. 실제로 Flutter 팀은 많은 간단한 앱 샘플(모든 Flutter 생성과 함께 제공되는 시작 앱 포함)에서 이 작업을 수행합니다.

다른 방향으로도 진행됩니다. 예를 들어 특정 앱의 컨텍스트에서 BottomNavigationBar에서 선택한 탭이 임시 상태가 아니라고 결정할 수 있습니다. 클래스 외부에서 변경해야 하거나 세션 간에 유지해야 할 수도 있습니다. 이 경우 _index 변수는 앱 상태입니다.

특정 변수가 임시 변수인지 아니면 앱 상태인지 구별하는 명확하고 보편적인 규칙은 없습니다. 때로는 하나를 다른 것으로 리팩터링해야 할 수도 있습니다. 예를 들어 명확하게 임시 상태로 시작하지만 애플리케이션의 기능이 성장함에 따라 앱 상태로 이동해야 할 수도 있습니다.

React의 setState와 Redux의 스토어에 대해 물었을 때 Redux의 저자인 Dan Abramov는 다음과 같이 대답했습니다.

```
"경험의 법칙은 덜 어색한 일을 하라는 것입니다."
```

요약하자면 모든 Flutter 앱에는 두 가지 개념적 상태 유형이 있습니다. 임시 상태는 State 및 setState()를 사용하여 구현할 수 있으며 단일 위젯에 로컬인 경우가 많습니다. 나머지는 앱 상태입니다. 두 유형 모두 모든 Flutter 앱에서 고유한 위치를 가지며, 두 유형 사이의 구분은 사용자의 선호도와 앱의 복잡성에 따라 다릅니다.
