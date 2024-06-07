# 이터러블 컬렉션 Iterable Collections

이 코드랩에서는 List나 Set과 같은 Iterbale 클래스를 구현하는 컬렉션을 사용하는 방법에 대해서 배웁니다.

Iterbale은 모든 종류의 Dart 애플리케이션을 위한 기본적인 구성 요소이며 아마 눈치채지 못했더라도 이미 사용하고 있을 겁니다.

이 코드랩은 이를 최대한 활용하는 것에 도움이 됩니다.

내장된 DartPad 편집기를 사용하면 예제 코드를 실행하고 연습을 완료하여 지식을 시험할 수 있습니다.

이 코드랩을 활용하려면 Dart 문법에 대한 기본적인 지식이 있어야 합니다.

이 코드랩에서는 다음과 같은 자료를 다룹니다:

- Iterable의 요소를 읽는 방법.

- Iterable의 요소가 조건을 충족하는지 확인하는 방법.

- Iterable의 내용을 필터링하는 방법.

- Iterable의 내용을 다른 값에 매핑하는 방법.

이 코드랩을 완료하는 데 예상되는 시간 : 60분.

```
주의

이 페이지는 내장된 DartPad를 사용하여 예제와 연습을 표시합니다.

DartPad 대신 빈 상자가 표시되면 DartPad 문제 해결 페이지로 이동하세요.
```

이 Codelab의 연습에는 부분적으로 완성된 코드 스니펫이 있습니다. DartPad를 사용하여 코드를 완성하고 실행 버튼을 클릭하여 지식을 시험할 수 있습니다. 메인 함수나 그 이하의 테스트 코드를 편집하지 마세요.

도움이 필요하면 힌트나 솔루션 드롭다운을 확장하세요.

## 컬렉션이란 무엇인가? What are collections?

컬렉션이란 요소라고 불리는 객체 그룹을 나타내는 객체입니다. Iterable은 일종의 컬렉션입니다.

컬렉션은 비어있을 수도 있고 많은 요소를 포함할 수도 있습니다. 목적에 따라 컬렉션의 구조와 구현은 다를 수 있습니다. 다음은 가장 일반적인 컬렉션 타입 중 일부입니다.

- List: 인덱스로 요소를 읽는 데 사용됩니다.

- Set: 한 번만 발생할 수 있는 요소를 포함하는 데 사용됩니다.

- Map: 키를 사용하여 요소를 읽는 데 사용됩니다.

## Iterable이란 무엇입니까? What is an Iterable?

Iterable은 순차적으로 접근할 수 있는 요소의 컬렉션입니다.

Dart에서 Iterable은 추상 클래스이고 그것은 직접 인스턴스화할 수 없다는 것을 의미합니다. 그러나, 새 List 또는 Set을 생성하여 새 Iterable을 생성할 수 있습니다.

List와 Set은 둘 다 Iterable이므로 Iterable 클래스와 같은 메서드와 속성을 갖고 있습니다.

Map은 구현에 따라 내부적으로 다른 데이터 구조를 사용합니다. 예를 들어, HashMap은 키를 사용하여 요소(값이라고도 하는)를 얻는 해시 테이블을 사용합니다.

Map의 요소는 Map의 항목 또는 값 속성을 사용하여 Iterable 객체로 읽을 수도 있습니다.

아래의 예제는 int의 Iterable이기도 한 int의 List를 보여줍니다.

```dart
Iterable<int> iterable = [1, 2, 3];
```

List와의 차이점은 Iterable을 사용하면, 인덱스로 요소를 읽는 것이 효율적이라고 보장할 수 없다는 것입니다. List와 달리 Iterable에는 [] 연산자가 없습니다.

예를 들어, 다음과 같은 유효하지 않은 코드를 생각해보세요.

```dart
Iterable<int> iterable = [1, 2, 3];
int value = iterable[1];
```

[]를 사용하여 요소를 읽으면 컴파일러는 '[]' 연산자가 Iterable 클래스에 대해 정의되지 않았음을 알려줍니다. 이것은 이런 경우에 [index]를 사용할 수 없음을 의미합니다.

대신 해당 위치에 도달할 때까지 반복 가능한 요소를 단계별로 실행하는 elementAt()을 사용하여 요소를 읽을 수 있습니다.

```dart
Iterable<int> iterable = [1, 2, 3];
int value = iterable.elementAt(1);
```

Iterable의 요소에 접근하는 방법에 대해 자세히 알아보려면 다음 섹션을 계속 진행하세요.

## 요소 읽기 Reading elements

for-in 반복문을 사용하여 iterable의 요소를 순차적으로 읽을 수 있습니다.

## 예시: for-in 반복문 사용 Example: Using a for-in loop

다음 예제는 for-in 반복문을 사용하여 요소를 읽는 방법을 보여줍니다.

```dart
void main() {
  const iterable = ['Salad', 'Popcorn', 'Toast'];
  for (final element in iterable) {
    print(element);
  }
}
```

```
자세히

for-in 반복문은 뒤에서 iterator을 사용합니다. 그러나 Iterator API가 직접 사용되는 것은 거의 볼 수 없습니다. for-in은 읽고 이해하기가 쉬우며 오류가 발생할 가능성이 적기 때문입니다.
```

```
주요 용어
- Iterable: Dart의 Iterable 클래스
- Iterator: Iterable 객체에서 요소를 읽기 위해 for-in이 사용하는 객체
- for-in 반복문: Iterable에서 요소를 순차적으로 읽는 쉬운 방법입니다.
```

## 예시: 첫 요소와 마지막 요소 사용

어떤 경우에는 Iterable의 첫 번째 또는 마지막 요소에만 접근하는 것을 원합니다.

Iterable 클래스를 사용하면 요소에 직접 접근할 수 없으므로 iterable[0]를 호출하여 첫 번째 요소에 접근할 수 없습니다. 대신, 첫번째 요소를 가져오는 first를 사용할 수 있습니다.

또한 Iterable 클래스를 사용하면 [] 연산자로 마지막 요소를 사용할 수 없지만, last 속성을 사용할 수 있습니다.

```
경고

Iterable의 마지막 요소에 접근하려면 다른 모든 요소를 단계별로 거쳐야 하기 때문에 last는 느릴 수 있습니다. 빈 Iterable에 first 또는 last를 사용하면 StateError가 발생합니다.
```

```dart
void main() {
  Iterable<String> iterable = const ['Salad', 'Popcorn', 'Toast'];
  print('The first element is ${iterable.first}');
  print('The last element is ${iterable.last}');
}
```

이 예시에서는 first와 last를 사용하여 Iterable의 첫 번째 요소와 마지막 요소를 가져오는 방법을 살펴보았습니다. 또한 조건을 만족하는 첫 번째 요소를 찾는 것도 가능합니다. 다음 섹션에서는 firstWhere() 메서드를 사용하여 이를 수행하는 방법을 보여줍니다.

## 예시: firestWhere() 사용 Example: Using firstWhere()

Iterable의 요소에 순차적으로 접근이 가능한 것과 첫 번째 또는 마지막 요소를 쉽게 얻을 수 있다는 것을 이미 살펴봤습니다.

이제, 특정 조건을 만족하는 첫 번째 요소를 찾는 firestWhere()를 사용하는 방법에 대해 배웁니다. 이 메서드는 입력이 특정 조건을 만족하면 true를 반환하는 함수인 predicate를 전달해야합니다.

```dart
String element = iterable.firstWher((element) => element.length > 5);
```

예를 들어, 다섯 자를 초과하는 첫 String을 찾길 원한다면 요소의 크기가 5보다 클 때 true를 반환하는 predicate를 전달해야 합니다.

다음 예제를 실행하여 firstWhere()가 어떻게 작동하는지 확인하세요. 모든 함수가 동일한 결과를 제공할 것이라고 생각하시나요?

```dart
bool predicate(String item) {
  return item.length > 5;
}

void main() {
  const items = ['Salad', 'Popcorn', 'Toast', 'Lasagne'];

  // 간단한 표현으로 찾을 수 있습니다:
  var foundItem1 = items.firstWhere((item) => item.length > 5);
  print(foundItem1);

  // 또는 함수 블록을 사용해 보십시오:
  var foundItem2 = items.firstWhere((item) {
    return item.length > 5;
  });
  print(foundItem2);

  // 또는 함수 참조를 전달할 수도 있습니다:
  var foundItem3 = items.firstWhere(predicate);
  print(foundItem3);

  // 값을 찾을 수 없는 경우 `orElse` 함수를 사용할 수도 있습니다!
  var foundItem4 = items.firstWhere(
    (item) => item.length > 10,
    orElse: () => 'None!',
  );
  print(foundItem4);
}
```