# 간단한 앱 상태 관리 Simple app state management

이제 선언형 UI 프로그래밍과 임시 상태와 앱 상태의 차이점을 알았으므로 간단한 앱 상태 관리에 대해 배울 준비가 되었습니다.

이 페이지에서는 provider 패키지를 사용할 것입니다. Flutter를 처음 사용하고 다른 접근 방식(Redux, Rx, 후크 등)을 선택할 강력한 이유가 없다면 아마도 이 접근 방식부터 시작해야 할 것입니다. provider 패키지는 이해하기 쉽고 많은 코드를 사용하지 않습니다. 또한 다른 모든 접근 방식에도 적용할 수 있는 개념을 사용합니다.

즉, 다른 반응형 프레임워크의 상태 관리에 대한 강력한 배경 지식이 있다면 옵션 페이지에 나열된 패키지와 튜토리얼을 찾을 수 있습니다.

## 예시

설명을 위해 다음과 같은 간단한 앱을 고려해보세요.

앱에는 카탈로그와 카트(각각 MyCatalog 및 MyCart 위젯으로 표시됨)라는 두 개의 별도 화면이 있습니다. 쇼핑 앱일 수도 있지만 간단한 소셜 네트워킹 앱에서도 동일한 구조를 상상할 수 있습니다("벽"은 카탈로그로 바꾸고 "즐겨찾기"는 카트로 교체).

카탈로그 화면에는 커스텀한 앱 바(MyAppBar)와 여러 목록 항목의 스크롤 보기(MyListItems)가 포함되어 있습니다.

따라서 최소한 5개의 위젯 하위 클래스가 있습니다. 이들 중 다수는 다른 곳에 "속해 있는" 상태에 대한 액세스가 필요합니다. 예를 들어, 각 MyListItem은 장바구니에 추가될 수 있어야 합니다. 또한 현재 표시된 항목이 이미 장바구니에 있는지 확인하고 싶을 수도 있습니다.

이는 우리의 첫 번째 질문으로 이어집니다. 카트의 현재 상태를 어디에 두어야 할까요?

## 상태 끌어올리기 Lifting state up

Flutter에서는 상태를 사용하는 위젯 위에 상태를 유지하는 것이 합리적입니다.

왜일까요? Flutter와 같은 선언적 프레임워크에서는 UI를 변경하려면 UI를 다시 빌드해야 합니다. MyCart.updateWith(somethingNew)를 갖는 쉬운 방법은 없습니다. 즉, 위젯에 대한 메서드를 호출하여 외부에서 위젯을 강제로 변경하는 것은 어렵습니다. 그리고 이 작업을 수행할 수 있더라도 프레임워크가 도움이 되도록 두는 대신 프레임워크와 싸우게 될 것입니다.

```dart
// BAD: 이렇게 하지 마세요
void myTapHandler() {
  var cartWidget = somehowGetMyCartWidget();
  cartWidget.updateWith(item);
}
```

위 코드가 작동하더라도 MyCart 위젯에서 다음을 처리해야 합니다.

```dart
// BAD: 이렇게 하지 마세요
Widget build(BuildContext context) {
  return SomeWidget(
    // 카트의 초기 상태
  );
}

void updateWith(Item item) {
  // 어떻게든 여기에서 UI를 변경해야 합니다.
}
```

UI의 현재 상태를 고려하고 여기에 새 데이터를 적용해야 합니다. 이런 식으로 버그를 피하는 것은 어렵습니다.

Flutter에서는 내용이 변경될 때마다 새 위젯을 구성합니다. MyCart.updateWith(somethingNew)(메서드 호출) 대신 MyCart(contents)(생성자)를 사용합니다. 해당 상위의 빌드 메소드에서만 새 위젯을 구성할 수 있으므로 컨텐츠를 변경하려면 해당 위젯이 MyCart의 상위 또는 상위에 있어야 합니다.

```dart
// GOOD
void myTapHandler(BuildContext context) {
  var cartModel = somehowGetMyCartModel(context);
  cartModel.add(item);
}
```

이제 MyCart에는 모든 버전의 UI를 구축하기 위한 코드 경로가 하나만 있습니다.

```dart
// GOOD
Widget build(BuildContext context) {
  var cartModel = somehowGetMyCartModel(context);
  return SomeWidget(
    // 카트의 현재 상태를 사용하여 UI를 한 번만 구성하면 됩니다.
    // ···
  );
}
```

이 예에서는 콘텐츠가 MyApp에 있어야 합니다. 변경될 때마다 위에서부터 MyCart를 다시 빌드합니다(자세한 내용은 나중에 설명). 이 때문에 MyCart는 수명 주기에 대해 걱정할 필요가 없습니다. 단지 특정 콘텐츠에 대해 표시할 내용을 선언하기만 하면 됩니다. 변경되면 이전 MyCart 위젯이 사라지고 새 위젯으로 완전히 대체됩니다.

위젯이 불변이라고 말할 때 이것이 의미하는 바입니다. 변경되지 않고 교체됩니다.

이제 카트의 상태를 어디에 두어야 하는지 알았으니, 카트에 액세스하는 방법을 살펴보겠습니다.

## 상태에 접근하기 Accessing the state

사용자가 카탈로그의 항목 중 하나를 클릭하면 해당 항목이 장바구니에 추가됩니다. 하지만 카트가 MyListItem 위에 있으므로 어떻게 해야 할까요?

간단한 옵션은 MyListItem을 클릭할 때 호출할 수 있는 콜백을 제공하는 것입니다. Dart의 함수는 일급 객체이므로 원하는 방식으로 전달할 수 있습니다. 따라서 MyCatalog 내에서 다음을 정의할 수 있습니다.

```dart
@override
Widget build(BuildContext context) {
  return SomeWidget(
    // 위젯을 구성하고 위 메드에 대한 참조를 전달합니다.
    MyListItem(myTapCallback),
  );
}

void myTapCallback(Item item) {
  print('user tapped on $item');
}
```

이 방법은 괜찮지만 여러 곳에서 수정해야 하는 앱 상태의 경우 많은 콜백을 전달해야 하므로 꽤 빨리 낡아집니다.

다행히 Flutter에는 위젯이 해당 하위 항목(즉, 해당 하위 항목뿐만 아니라 그 아래에 있는 모든 위젯)에 데이터와 서비스를 제공하는 메커니즘이 있습니다. 모든 것이 Widget인 Flutter에서 기대할 수 있듯이 이러한 메커니즘은 InheritedWidget, InheritedNotifier, InheritedModel 등과 같은 특별한 종류의 위젯일 뿐입니다. 우리가 하려는 일에 비해 약간 로우 레벨이기 때문에 여기서는 다루지 않겠습니다.

대신, 로우 레벨의 위젯과 함께 작동하지만 사용이 간편한 패키지를 사용할 것입니다. provider라고 합니다.

provider로 작업하기 전에 provider에 대한 종속성을 pubspec.yaml에 추가하는 것을 잊지 마십시오.

provider 패키지를 종속 항목으로 추가하려면 flutter pub add를 실행하세요.

```
flutter pub add provider
```

이제 'package:provider/provider.dart'를 가져올 수 있습니다. 그리고 빌드를 시작하세요.

provider를 사용하면 콜백이나 InheritedWidget에 대해 걱정할 필요가 없습니다. 하지만 세 가지 개념을 이해해야 합니다.

- ChangeNotifier
- ChangeNotifierProvider
- Consumer

## ChangeNotifier

ChangeNotifier는 리스너에게 변경 알림을 제공하는 Flutter SDK에 포함된 간단한 클래스입니다. 즉, ChangeNotifier인 경우 해당 변경 사항을 구독할 수 있습니다. (이 용어에 익숙한 사람들에게 말하자면, Observable의 한 형태입니다.)

provider에서 ChangeNotifier는 애플리케이션 상태를 캡슐화하는 한 가지 방법입니다. 매우 간단한 앱의 경우 단일 ChangeNotifier를 사용하면 됩니다. 복잡한 모델에는 여러 모델이 있으므로 ChangeNotifier도 여러 개 있습니다. (provider와 함께 ChangeNotifier를 전혀 사용할 필요는 없지만 작업하기 쉬운 클래스입니다.)

쇼핑 앱 예에서는 ChangeNotifier에서 장바구니 상태를 관리하려고 합니다. 다음과 같이 이를 확장하는 새 클래스를 만듭니다.

```dart
class CartModel extends ChangeNotifier {
  /// 카트의 내부 private 상태입니다.
  final List<Item> _items = [];

  /// 장바구니에 있는 항목을 수정할 수 없는 view 입니다.
  UnmodifiableListView<Item> get items => UnmodifiableListView(_items);

  /// 모든 항목의 현재 총 가격입니다(모든 항목의 가격이 $42라고 가정).
  int get totalPrice => _items.length * 42;

  /// 장바구니에 [itme]을 추가합니다. 이것과 [removeAll]은 외부에서 카트를 수정하는 
  /// 유일한 방법입니다.
  void add(Item item) {
    _items.add(item);
    // 이 호출은 이 모델을 듣고 있는 위젯에게 다시 빌드하라고 지시합니다.
    notifyListeners();
  }

  /// 장바구니에서 모든 항목을 제거합니다.
  void removeAll() {
    _items.clear();
    // 이 호출은 이 모델을 듣고 있는 위젯에게 다시 빌드하라고 지시합니다.
    notifyListeners();
  }
}
```

ChangeNotifier와 관련된 유일한 코드는 notifyListeners()에 대한 호출입니다. 앱의 UI를 변경할 수 있는 방식으로 모델이 변경될 때마다 이 메서드를 호출하세요. CartModel의 다른 모든 것은 모델 자체와 비즈니스 로직입니다.

ChangeNotifier는 flutter:foundation의 일부이며 Flutter의 상위 수준 클래스에 의존하지 않습니다. 쉽게 테스트할 수 있습니다(위젯 테스트를 사용할 필요도 없습니다). 예를 들어 다음은 CartModel의 간단한 단위 테스트입니다.

```dart
test('adding item increases total cost', () {
  final cart = CartModel();
  final startingPrice = cart.totalPrice;
  var i = 0;
  cart.addListener(() {
    expect(cart.totalPrice, greaterThan(startingPrice));
    i++;
  });
  cart.add(Item('Dash'));
  expect(i, 1);
});
```

## ChangeNotifierProvider

ChangeNotifierProvider는 ChangeNotifier의 인스턴스를 하위 항목에 제공하는 위젯입니다. provider 패키지에서 제공됩니다.

우리는 ChangeNotifierProvider를 어디에 두어야 하는지 이미 알고 있습니다: 액세스해야 하는 위젯 위입니다. CartModel의 경우 이는 MyCart와 MyCatalog 모두 위의 어딘가를 의미합니다.

범위를 오염시키고 싶지 않기 때문에 ChangeNotifierProvider를 필요 이상으로 높게 배치하고 싶지 않습니다. 하지만 우리의 경우 MyCart와 MyCatalog 모두 위에 있는 유일한 위젯은 MyApp입니다.

```dart
void main() {
  runApp(
    ChangeNotifierProvider(
      create: (context) => CartModel(),
      child: const MyApp(),
    ),
  );
}
```

CartModel의 새 인스턴스를 생성하는 빌더를 정의하고 있다는 점에 유의하세요. ChangeNotifierProvider는 꼭 필요한 경우가 아니면 CartModel을 다시 빌드하지 않을 만큼 똑똑합니다. 또한 인스턴스가 더 이상 필요하지 않은 경우 CartModel에서 dispose()를 자동으로 호출합니다.

둘 이상의 클래스를 제공하려면 MultiProvider를 사용할 수 있습니다.

```dart
void main() {
  runApp(
    MultiProvider(
      providers: [
        ChangeNotifierProvider(create: (context) => CartModel()),
        Provider(create: (context) => SomeOtherClass()),
      ],
      child: const MyApp(),
    ),
  );
}
```

## Consumer

이제 상단의 ChangeNotifierProvider 선언을 통해 CartModel이 앱의 위젯에 제공되었으므로 이를 사용할 수 있습니다.

이는 Consumer 위젯을 통해 수행됩니다.

```dart
return Consumer<CartModel>(
  builder: (context, cart, child) {
    return Text('Total price: ${cart.totalPrice}');
  },
);
```

액세스하려는 모델의 타입을 지정해야 합니다. 이 경우 CartModel이 필요하므로 Consumer<CartModel>을 작성합니다. 제네릭(<CartModel>)을 지정하지 않으면 provider 패키지가 도움을 줄 수 없습니다. provider는 타입을 기반으로 하며 타입이 없으면 사용자가 원하는 것이 무엇인지 알 수 없습니다.

Consumer 위젯의 유일한 필수 인자는 builder입니다. builder는 ChangeNotifier가 변경될 때마다 호출되는 함수입니다. (즉, 모델에서 notifyListeners()를 호출하면 해당하는 모든 Consumer 위젯의 모든 builder 메서드가 호출됩니다.)

builder는 세 가지 인자로 호출됩니다. 첫 번째는 모든 build 메서드에서도 얻을 수 있는 context입니다.

builder 함수의 두 번째 인자는 ChangeNotifier의 인스턴스입니다. 우리가 애초에 요구한 것이 바로 그것입니다. 모델의 데이터를 사용하여 특정 지점에서 UI가 어떻게 보일지 정의할 수 있습니다.

세 번째 인자는 최적화를 위해 존재하는 child입니다. 모델이 변경될 때 변경되지 않는 Consumer 아래에 큰 위젯 하위 트리가 있는 경우 이를 한 번 구성한 후 builder를 통해 가져올 수 있습니다.

```dart
return Consumer<CartModel>(
  builder: (context, cart, child) => Stack(
    children: [
      // 매번 다시 빌드하지 않고 여기서 SomeExpensiveWidget을 사용하세요.
      if (child != null) child,
      Text('Total price: ${cart.totalPrice}'),
    ],
  ),
  // 여기에서 expensive 위젯을 만드세요.
  child: const SomeExpensiveWidget(),
);
```

Consumer 위젯을 가능한 한 트리 깊은 곳에 배치하는 것이 가장 좋습니다. 어딘가에서 일부 세부 사항이 변경되었다는 이유로 UI의 많은 부분을 다시 빌드하고 싶지는 않습니다.

```dart
// 이렇게 하지 마세요.
return Consumer<CartModel>(
  builder: (context, cart, child) {
    return HumongousWidget(
      // ...
      child: AnotherMonstrousWidget(
        // ...
        child: Text('Total price: ${cart.totalPrice}'),
      ),
    );
  },
);
```

대신에:

```dart
// 이렇게 하세요.
return HumongousWidget(
  // ...
  child: AnotherMonstrousWidget(
    // ...
    child: Consumer<CartModel>(
      builder: (context, cart, child) {
        return Text('Total price: ${cart.totalPrice}');
      },
    ),
  ),
);
```

## Provider.of

때로는 UI를 변경하기 위해 모델의 데이터가 실제로 필요하지 않지만 여전히 액세스해야 하는 경우가 있습니다. 예를 들어, ClearCart 버튼은 사용자가 카트에서 모든 것을 제거할 수 있도록 하려고 합니다. 장바구니의 내용을 표시할 필요는 없으며 단지 clear() 메서드를 호출하기만 하면 됩니다.

이를 위해 Consumer<CartModel>을 사용할 수 있지만 이는 낭비입니다. 우리는 다시 빌드할 필요가 없는 위젯을 다시 빌드하도록 프레임워크에 요청합니다.

이 사례에서는 listen 매개변수를 false로 설정한 상태에서 Provider.of를 사용할 수 있습니다.

```dart
Provider.of<CartModel>(context, listen: false).removeAll();
```

build 메서드에서 위 코드를 사용하면 notifyListeners가 호출될 때 이 위젯이 다시 빌드되지 않습니다.

## 모든 것을 종합하면 Putting it all together

이 글에서 다루는 예시를 확인해 보세요. 더 간단한 것을 원한다면 provider를 사용하여 구축했을 때 간단한 Counter 앱이 어떻게 보이는지 확인하세요.

이 문서를 따라감으로써 상태 기반 애플리케이션을 만드는 능력이 크게 향상되었습니다. 이러한 기술을 익히려면 provider를 통해 직접 애플리케이션을 구축해 보세요.
