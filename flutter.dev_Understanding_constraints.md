# 제약 조건 이해하기 Understanding constraints

Flutter를 학습하는 사람이 "폭이 100인 위젯이 왜 100픽셀이 되지 않나요?"라고 물으면, 기본적으로 그 위젯을 Center 안에 넣으라고 대답하죠, 맞나요?

```
Center 안에 넣는 이유

위젯을 Center 안에 넣으라고 하는 이유는 Center가 자식 위젯을 가운데에 위치시키고, 자식 위젯이 지정한 크기만큼의 공간을 사용할 수 있도록 하기 때문입니다.

Flutter에서 위젯의 크기는 부모 위젯의 제약 조건에 따라 결정됩니다. 만약 위젯이 특정한 크기를 갖도록 설정되었지만, 부모 위젯이 그것보다 작은 공간만 제공한다면, 해당 위젯은 원하는 크기를 가지지 못할 수 있습니다. 이때 Center 위젯은 자식 위젯이 가능한 한 지정된 크기를 유지할 수 있도록 부모로부터 전달받은 공간을 최대한 활용하면서 자식 위젯을 중앙에 배치합니다.

예를 들어, Center 위젯은 자식 위젯의 크기를 제한하지 않기 때문에, 자식 위젯이 width: 100으로 설정되어 있다면 실제로 100픽셀의 너비를 가질 수 있게 됩니다. 이로 인해 위젯이 기대한 크기를 갖지 않는 문제를 해결할 수 있습니다.
```

그러지 마세요.

그렇게 하면 그들은 계속해서 돌아와 일부 FittedBox가 작동하지 않는 이유, 해당 Column이 오버플로되는 이유 또는 IntrinsicWidth가 수행해야 하는 작업이 무엇인지 묻습니다.

대신, 먼저 Flutter 레이아웃이 HTML 레이아웃과 매우 다르다는 점을 알려주고(아마도 HTML 레이아웃에서 나온 것일 수 있음) 다음 규칙을 기억하도록 하세요.

제약 조건이 내려갑니다. 사이즈가 올라갑니다. 부모는 위치를 설정합니다.

Flutter 레이아웃은 이 규칙을 모르면 실제로 이해할 수 없으므로 Flutter 개발자는 조기에 이를 배워야 합니다.

더 자세히:
    - 위젯은 부모로부터 자체 제약 조건을 받습니다. 제약 조건은 최소 및 최대 너비, 최소 및 최대 높이 등 4개의 double 집합입니다.
    - 그런 다음 위젯은 자신의 자식들 목록을 살펴봅니다. 위젯은 하나씩 자식에게 제약 조건이 무엇인지 알려주고(자식마다 다를 수 있음) 각 자식에게 원하는 크기를 묻습니다.
    - 그런 다음 위젯은 해당 하위 요소(가로로 x축, 세로로 y축)를 하나씩 배치합니다.
    - 그리고 마지막으로 위젯은 자신의 부모에게 자신의 크기에 대해 알려줍니다(물론 원래 제약 조건 내에서).
    - 예를 들어 구성된 위젯에 패딩이 있는 열이 포함되어 있고 두 자식을 다음과 같이 배치하려는 경우:

협상은 다음과 같이 진행됩니다:

위젯: "안녕하세요 부모님, 제 제약사항은 무엇인가요?"

부모: "너비는 0~300픽셀, 높이는 0~85픽셀이어야 한다."

위젯: "흠, 나는 5픽셀의 패딩을 원하므로 내 아이들은 최대 너비가 290픽셀, 높이가 75픽셀을 가질 수 있겠군."

위젯: "안녕 첫째 아이야, 너는 너비가 0~290픽셀, 키가 0~75픽셀이어야 한다."

첫 번째 아이: "좋아요, 그럼 너비가 290픽셀, 높이가 20픽셀이 되었으면 좋겠어요."

위젯: "흠, 둘째 아이를 첫째 아이 아래에 배치하고 싶기 때문에 두 번째 아이의 높이는 55픽셀만 남는군."

위젯: "안녕 둘째 아이야, 너는 너비가 0에서 290 사이이고 키가 0에서 55 사이여야 해."

둘째 아이: "좋아요. 저는 가로가 140픽셀, 세로가 30픽셀이었으면 좋겠어요."

위젯: "아주 좋아. 첫 번째 아이의 위치는 x: 5 및 y: 5이고, 두 번째 아이의 위치는 x: 80 및 y: 25야."

위젯: "안녕하세요 부모님, 제 크기는 가로 300픽셀, 세로 60픽셀로 결정했습니다."

## 제한 Limitations

Flutter의 레이아웃 엔진은 원패스 프로세스로 설계되었습니다. 이는 Flutter가 위젯을 매우 효율적으로 배치하지만 몇 가지 제한 사항이 있음을 의미합니다.

- 위젯은 부모가 제공한 제약 조건 내에서만 자체 크기를 결정할 수 있습니다. 이는 일반적으로 위젯이 원하는 크기를 가질 수 없음을 의미합니다.

- 위젯의 위치를 ​​결정하는 것은 위젯의 부모이기 때문에 위젯은 화면에서 자신의 위치를 ​​알 수 없고 결정하지도 않습니다.

- 부모의 크기와 위치도 자신의 부모에 따라 달라지므로 트리 전체를 고려하지 않고 위젯의 크기와 위치를 정확하게 정의하는 것은 불가능합니다.

- 자식이 부모와 다른 크기를 원하고 부모를 정렬할 정보가 충분하지 않은 경우 자식의 크기가 무시될 수 있습니다. 정렬을 정의할 때는 구체적으로 설명하세요.

Flutter에서 위젯은 기본 RenderBox 객체에 의해 렌더링됩니다. Flutter의 많은 상자, 특히 자식 하나만 가져오는 상자는 제약 조건을 자식에게 전달합니다.

일반적으로 제약 조건을 처리하는 방법에 따라 세 가지 종류의 상자가 있습니다.

- 가능한 한 크게 되려고 시도하는 상자. 예를 들어 Center 및 ListView에서 사용되는 상자입니다.
- 자식과 같은 크기가 되려고 시도하는 상자. 예를 들어 Transform 및 Opacity에서 사용되는 상자입니다.
- 특정 크기가 되려고 시도하는 상자. 예를 들어 Image 및 Text에 사용되는 상자입니다.

Container와 같은 일부 위젯은 생성자 인자를 기반으로 하는 타입마다 다릅니다. Container 생성자는 기본적으로 가능한 한 크게 만들려고 시도하지만,  예를 들어 너비를 지정하면 이를 존중하고 특정 크기로 설정하려고 합니다.

예를 들어 Row 및 Column(flex 상자)과 같은 기타 위젯은 Flex 섹션에 설명된 대로 제공된 제약 조건에 따라 달라집니다.

## 예시 1

```dart
Container(color: red)
```

화면은 Container의 부모이며 Container가 화면과 정확히 같은 크기가 되도록 강제합니다.

따라서 Container는 화면을 채우고 빨간색으로 칠합니다.

## 예시 2

```dart
Container(width: 100, height: 100, color: red)
```

빨간색 Container는 100 × 100이 되기를 원하지만 화면이 화면과 정확히 같은 크기로 강제하기 때문에 그럴 수 없습니다.

따라서 Container가 화면을 채웁니다.

## 예시 3

```dart
Center(
  child: Container(width: 100, height: 100, color: red),
)
```

화면은 Center가 화면과 정확히 같은 크기가 되도록 강제하므로 Center가 화면을 채웁니다.

Center는 Container에게 원하는 크기로 지정할 수 있지만 화면보다 크지는 않음을 알려줍니다. 이제 Container는 실제로 100 × 100이 될 수 있습니다.

## 예시 4

```dart
Align(
  alignment: Alignment.bottomRight,
  child: Container(width: 100, height: 100, color: red),
)
```

Center 대신 Align을 사용한다는 점에서 이전 예제와 다릅니다.

Align은 또한 컨테이너에 원하는 크기를 지정할 수 있음을 알려주지만, 빈 공간이 있으면 컨테이너를 중앙에 배치하지 않습니다. 대신 컨테이너를 사용 가능한 공간의 오른쪽 하단에 정렬합니다.

## 예시 5

```dart
Center(
  child: Container(
      width: double.infinity, height: double.infinity, color: red),
)
```

화면은 Center가 화면과 정확히 같은 크기가 되도록 강제하므로 Center가 화면을 채웁니다.

Center는 Container에게 원하는 크기로 지정할 수 있지만 화면보다 클 수는 없음을 알려줍니다. Container는 무한한 크기를 원하지만 화면보다 클 수 없기 때문에 화면을 채울 뿐입니다.

## 예시 6

```dart
Center(
  child: Container(color: red),
)
```

화면은 Center가 화면과 정확히 같은 크기가 되도록 강제하므로 Center가 화면을 채웁니다.

Center는 Container에게 원하는 크기로 지정할 수 있지만 화면보다 클 수는 없음을 알려줍니다. Container에는 자식이 없고 크기가 고정되어 있지 않기 때문에 가능한 한 크게 크기를 결정하여 전체 화면을 채웁니다.

그런데 왜 Container가 그런 결정을 내리는 걸까요? 이는 Container 위젯을 만든 사람들의 디자인 결정이기 때문입니다. 다르게 생성되었을 수도 있으며 상황에 따라 어떻게 작동하는지 이해하려면 Container API 문서를 읽어야 합니다.

## 예시 7

```dart
Center(
  child: Container(
    color: red,
    child: Container(color: green, width: 30, height: 30),
  ),
)
```

화면은 Center가 화면과 정확히 같은 크기가 되도록 강제하므로 Center가 화면을 채웁니다.

Center는 빨간색 Container에게 원하는 크기로 지정할 수 있지만 화면보다 클 수는 없음을 알려줍니다. 빨간색 Container에는 크기가 없지만 자식이 있으므로 자식과 동일한 크기를 갖기로 결정합니다.

빨간색 Container는 자식에게 원하는 크기로 지정할 수 있지만 화면보다 크지는 않음을 알려줍니다.

자식은 30 × 30이 되기를 원하는 녹색 Container입니다. 빨간색 Container의 크기가 자식의 크기에 맞춰진다는 점을 고려하면 역시 30 × 30입니다. 녹색 Container가 빨간색을 완전히 덮기 때문에 빨간색 Container가 보이지 않습니다.

## 예시 8

```dart
Center(
  child: Container(
    padding: const EdgeInsets.all(20),
    color: red,
    child: Container(color: green, width: 30, height: 30),
  ),
)
```

빨간색 Container는 자식 크기에 맞춰 크기가 조정되지만 자체 패딩도 고려됩니다. 따라서 30 × 30에 패딩을 더한 것이기도 합니다. 패딩으로 인해 빨간색이 보이고, 녹색 Container는 이전 예시와 크기가 동일합니다.


## 예시 9

```dart
ConstrainedBox(
  constraints: const BoxConstraints(
    minWidth: 70,
    minHeight: 70,
    maxWidth: 150,
    maxHeight: 150,
  ),
  child: Container(color: red, width: 10, height: 10),
)
```

Container가 70~150픽셀 사이여야 한다고 추측할 수 있지만 이는 틀렸습니다. ConstrainedBox는 부모로부터 받은 제약 조건에만 추가 제약 조건을 적용합니다.

여기서 화면은 ConstrainedBox가 화면과 정확히 같은 크기가 되도록 강제하므로 자식 Container에도 화면 크기를 가정하도록 지시하여 ConstrainedBox 매개변수를 무시합니다.

## 예시 10

```dart
Center(
  child: ConstrainedBox(
    constraints: const BoxConstraints(
      minWidth: 70,
      minHeight: 70,
      maxWidth: 150,
      maxHeight: 150,
    ),
    child: Container(color: red, width: 10, height: 10),
  ),
)
```

이제 Center에서는 ConstrainedBox를 화면 크기까지 원하는 크기로 설정할 수 있습니다. ConstrainedBox는 Constraints 매개변수의 추가 제약 조건을 자식에 적용합니다.

Container는 70~150픽셀 사이여야 합니다. 10개의 픽셀을 원하므로 결국 70(최소값)이 됩니다.

## 예시 11

```dart
Center(
  child: ConstrainedBox(
    constraints: const BoxConstraints(
      minWidth: 70,
      minHeight: 70,
      maxWidth: 150,
      maxHeight: 150,
    ),
    child: Container(color: red, width: 1000, height: 1000),
  ),
)
```

Center를 사용하면 ConstrainedBox를 화면 크기까지 원하는 크기로 설정할 수 있습니다. ConstrainedBox는 Constraints 매개변수의 추가 제약 조건을 자식에 적용합니다.

Container는 70~150픽셀 사이여야 합니다. 1000픽셀을 원하므로 결국 150(최대값)이 됩니다.

## 예시 12

```dart
Center(
  child: ConstrainedBox(
    constraints: const BoxConstraints(
      minWidth: 70,
      minHeight: 70,
      maxWidth: 150,
      maxHeight: 150,
    ),
    child: Container(color: red, width: 100, height: 100),
  ),
)
```

Center를 사용하면 ConstrainedBox를 화면 크기까지 원하는 크기로 설정할 수 있습니다. ConstrainedBox는 Constraints 매개변수의 추가 제약 조건을 자식에 적용합니다.

Container는 70~150픽셀 사이여야 합니다. Container는 100픽셀을 원하고, 그 크기는 70에서 150사이이기 때문에 그 크기를 갖습니다.

## 예시 13

```dart
UnconstrainedBox(
  child: Container(color: red, width: 20, height: 50),
)
```

화면은 UnconstrainedBox의 크기를 화면과 정확히 동일하게 만듭니다. 그러나 UnconstrainedBox는 자식 컨테이너를 원하는 크기로 설정할 수 있습니다.

## 예시 14

```dart
UnconstrainedBox(
  child: Container(color: red, width: 4000, height: 50),
)
```

화면은 UnconstrainedBox가 화면과 정확히 같은 크기가 되도록 강제하고 UnconstrainedBox는 해당 하위 컨테이너를 원하는 크기로 만듭니다.

불행하게도 이 경우 Container의 너비는 4000픽셀이고 너무 커서 UnconstrainedBox에 맞지 않으므로 UnconstrainedBox는 매우 무서운 "오버플로 경고"를 표시합니다.

## 예시 15

```dart
OverflowBox(
  minWidth: 0,
  minHeight: 0,
  maxWidth: double.infinity,
  maxHeight: double.infinity,
  child: Container(color: red, width: 4000, height: 50),
)
```

화면은 OverflowBox를 화면과 정확히 같은 크기로 강제하고 OverflowBox는 자식 컨테이너를 원하는 크기로 만듭니다.

OverflowBox는 UnconstrainedBox와 유사합니다. 차이점은 아이가 공간에 맞지 않으면 경고가 표시되지 않는다는 것입니다.

이 경우 Container의 너비는 4000픽셀이고 너무 커서 OverflowBox에 맞지 않지만 OverflowBox는 경고 없이 가능한 만큼만 표시합니다.

## 예시 16

```dart
UnconstrainedBox(
  child: Container(color: Colors.red, width: double.infinity, height: 100),
)
```

이렇게 하면 아무것도 렌더링되지 않으며 콘솔에 오류가 표시됩니다.

UnconstrainedBox는 자식을 원하는 크기로 만들 수 있지만 자식은 크기가 무한한 컨테이너입니다.

Flutter는 무한한 크기를 렌더링할 수 없으므로 다음 메시지와 함께 오류가 발생합니다. BoxConstraints는 무한한 너비를 강제합니다.

## 예시 17

```dart
UnconstrainedBox(
  child: LimitedBox(
    maxWidth: 100,
    child: Container(
      color: Colors.red,
      width: double.infinity,
      height: 100,
    ),
  ),
)
```

여기서는 더 이상 오류가 발생하지 않습니다. 왜냐하면 UnconstrainedBox에 의해 LimitedBox에 무한한 크기가 주어질 때 최대 너비 100을 자식에게 전달하기 때문입니다.

UnconstrainedBox를 Center 위젯으로 교체하면 LimitedBox는 더 이상 제한을 적용하지 않으며(제한은 무한 제약 조건을 받을 때만 적용되므로) 컨테이너의 너비는 100을 초과할 수 있습니다.

이는 LimitedBox와 ConstrainedBox의 차이점을 설명합니다.
