# Dart의 타입 시스템 The Dart type system

다트는 타입 안전한 언어입니다. 다트 언어는 변수의 값이 항상 변수의 정적 타입과 일치함을 보장하기 위해 정적 타입 검사와 런타임 검사의 조합을 사용합니다(때때로 건전한 타입이라고도 함). 타입을 표기하는 것은 의무적이지만 타입 추론때문에 타입 주석은 선택적입니다.

정적 타입 검사의 한 가지 이점은 다트의 정적 분석기를 사용하여 컴파일 타임에 버그를 찾는 능력입니다.

제네릭 클래스에 타입 주석을 추가하면 대부분의 정적 분석 오류를 고칠 수 있습니다. 가장 일반적인 제네릭 클래스는 컬렉션 타입 List<T>와 Map<K, V>입니다.

예를 들어, 아래와 같은 코드에서 printInts() 함수는 정수 리스트를 출력하고, main() 함수는 리스트를 만들어 printInts()에 전달합니다.

```dart
void printInts(List<int> a) => print(a);

void main() {
  final list = [];

  list.add(1);
  list.add('2');
  printInts(list);
}
```

위의 코드는 printInts(list)를 호출할 때 list에서 타입 오류가 발생합니다:

```
error - The argument type 'List<dynamic>' can't be assigned to the parameter type 'List<int>'. - argument_type_not_assignable
```

오류는 List<dynamic>에서 List<int>로 불건전한 암묵적 타입 캐스팅을 강조합니다. list 변수는 정적 타입 List<dynamic>입니다. 이것은 초기화 선언 var list = []이 dynamic보다 더 구체적인 타입 인자를 추론할만한 충분한 정보를 분석기에 제공하지 않았기 때문입니다. printInts() 함수는 타입 List<int>의 매개변수를 예상하므로 타입 불일치가 발생합니다.

타입 주석(<int>와 같은)을 리스트 생성에 추가할 때 분석기는 문자열 인자를 정수 매개변수에 할당할 수 없다고 알려줍니다. 코드의 list.add('2')에서 홑따옴표를 제거하면 정적 분석을 통과하고 오류나 경고 없이 실행합니다.

```dart
void printInts(List<int> a) => print(a);

void main() {
  final list = <int>[];
  list.add(1);
  list.add(2);
  printInts(list);
}
```

DartPad에서 확인해보세요.

## 건전성이란 무엇인가? What is soundness?

건전성은 프로그램이 유효하지 않은 특정 상태가 될 수 없도록 보장하는 것입니다. 건전한 타입 시스템은 표현식이 표현식의 정적 타입과 일치하지 않는 값을 평가하는 상태가 될 수 없음을 의미합니다. 예를 들어 표현식의 정적 타입이 String이라면 런타임에 표현식을 평가할 때 문자열만 얻는 것이 보장됩니다.

Dart의 타입 시스템은 Java, C#처럼 건전합니다. 정적 검사(컴파일 타임 오류)와 런타임 검사의 조합을 사용하여 건전성을 강제합니다. 예를 들어, String을 int에 할당하면 컴파일 타임 오류가 발생합니다. String이 아닌 객체에 as String을 사용하여 String으로 캐스팅하면 런타임 오류와 함께 실패합니다.

## 건전성의 이점 The benefits of soundness

건전한 타입 시스템은 몇 가지 이점이 있습니다:

- 컴파일 타임에 타입 관련 버그를 드러냅니다.
    건전한 타입 시스템은 코드가 타입이 명확하도록 강제하여 런타임에 발견하기 어려운 타입 관련 버그가 컴파일 타임에 드러나도록 합니다.

- 더 읽기 쉬운 코드.
    실제로 구체적인 타입을 갖는 값에 의존하므로 코드가 더 읽기 쉽습니다. 건전한 Dart에서 타입은 거짓말할 수 없습니다.

- 더 유지보수가 가능한 코드.
    건전한 타입 시스템을 사용하면 코드 하나를 변경할 때 타입 시스템이 손상된 코드에 대해 경고할 수 있습니다.

- 더 나은 AOT 컴파일.
    AOT 컴파일은 타입없이 가능하지만, 생성된 코드는 덜 효율적입니다.

## 정적 분석 통과에 대한 팁 Tips for passing static analysis

정적 타입에 대한 대부분의 규칙은 이해하기 쉽습니다. 여기 덜 명백한 규칙 몇 가지가 있습니다.

- 메서드를 재정의할 때 건전한 반환 타입을 사용하세요.
- 메서드를 재정의할 때 건전한 매개변수를 사용하세요.
- dynamic 리스트를 타입이 있는 리스트처럼 사용하지 마세요.

위의 규칙들을 다음의 타입 계층 구조를 사용한 예시와 함께 세부적으로 살펴봅시다.

## 메서드를 재정의할 때 건전한 반환 타입을 사용하세요. Use sound return types when overriding methods

하위 클래스에 있는 메서드의 반환 타입은 상위 클래스에 있는 메서드의 반환 타입과 같거나 하위 타입이어야 합니다. Animal 클래스에 있는 getter 메서드를 생각해보세요:

```dart
class Animal {
  void chase(Animal a) { ... }
  
  Animal get parent => ...
}
```

parent getter 메서드는 Animal을 반환합니다. 하위 클래스 HoneyBadger에서 HoneyBadger(또는 Animal의 다른 하위 타입)을 getter의 반환 타입으로 대체할 수 있지만 관계없는 타입은 허용되지 않습니다.

```dart
// 정적 분석: 성공
class HoneyBadger extends Animal {
  @override
  void chase(Animal a) { ... }

  @override
  HoneyBadger get parent => ...
}
```

```dart
// 정적 분석: 실패
class HoneyBadger extends Animal {
  @override
  void chase(Animal a) { ... }

  @override
  Root get parent => ...
}
```

## 메서드를 재정의할 때 건전한 매개변수 타입을 사용하세요. Use sound parameter types when overriding methods

재정의된 메서드의 매개변수는 상위 클래스의 해당 매개변수와 같은 타입이거나 상위 타입이어야 합니다. 원래 매개변수의 하위 타입을 대체하여 매개변수 타입을  구체화하지 마세요.

```
노트

하위타입을 사용하는 합리적인 이유가 있다면 convariant 키워드를 사용하세요.
```

Animal 클래스에 대한 chase(Animal) 메서드를 생각해보세요.

```dart
class Animal {
  void chase(Animal a) { ... }
  Animal get parent => ...
}
```

chase() 메서드는 Animal을 매개변수로 가집니다. HoneyBadger는 무엇이든 쫓을 수 있습니다. chase() 메서드가 어떤 것(Object)을 매개변수로 받든 괜찮습니다.

```dart
// 정적 분석: 성공
class HoneyBadger extends Animal {
  @override
  void chase(Object a) { ... }

  @override
  Animal get parent => ...
}
```

다음 코드는 chase() 메서드의 매개변수를 Animal에서 Animal의 하위 타입인 Mouse로 구체화했습니다.

```dart
// 정적 분석: 실패
class Mouse extends Animal { ... }

class Cat extends Animal {
  @override
  void chase(Mouse a) { ... }
}
```

이 코드는 고양이를 정의하고 악어 다음으로 보내는 것이 가능하기 때문에 타입이 안전하지 않습니다.

```dart
Animal a = Cat();
a.chase(Alligator()); // 타입이 안전하지 않거나 고양이가 안전하지 않습니다.
```

## dynamic 리스트를 타입이 있는 리스트처럼 사용하지 마세요. Don't use a dynamic list as a typed list

dynamic 리스트는 다른 타입이 존재하는 리스트를 원할 때 사용하면 좋습니다. 그러나 dynamic 리스트를 타입이 있는 리스트처럼 사용할 수는 없습니다.

이 규칙은 제네릭 타입의 인스턴스에도 적용됩니다.

다음 코드는 Dog의 dynamic 리스트를 만들고 Cat 타입의 리스트를 할당합니다. 이것은 정적 분석 동안 오류를 생성합니다.

```dart
// 정적 분석: 실패
void main() {
  List<Cat> foo = <dynamic>[Dog()]; // 오류
  List<dynamic> bar = <dynamic>[Dog(), Cat()]; // 괜찮음
}
```

## 런타임 검사 Runtime checks

런타임 검사는 컴파일 타입에 발견되지 못한 타입 안전성 이슈를 다룹니다.

예를 들어 아래의 코드는 런타임에 예외를 발생시킵니다. dogs 리스트를 cats 리스트로 캐스팅하는 것은 오류이기 때문입니다.

```dart
// 정적 분석: 실패
void main() {
  List<Animal> animals = <Dog>[Dog()];
  List<Cat> cats = animals as List<Cat>;
}
```

## 타입 추론 Type inference

분석기는 필드, 메서드, 로컬 변수, 제네릭 타입 인자의 타입을 추론할 수 있습니다. 분석기가 구체적인 타입을 추론하기에 충분한 정보를 갖지 않으면 dynamic 타입을 사용합니다.

여기에 타입 추론이 어떻게 제네릭과 함께 작동하는지에 대한 예시가 있습니다. 이 예시에서는 arguments라고 명명된 변수가 문자열 키와 다양한 타입의 값이 짝을 이루는 맵을 담고 있습니다.

변수를 명시적으로 입력하는 경우, 다음과 같이 작성할 수 있습니다.

```dart
Map<String, dynamic> arguments = {'argA': 'hello', 'argB': 42};
```

위의 코드 대신 var 또는 final을 사용하여 Dart가 타입을 추론하게 할 수 있습니다.

```dart
var arguments = {'argA': 'hello', 'argB': 42}; // Map<String, Object>
```

맵 리터럴은 엔트리로부터 해당 타입을 추론하고 변수는 맵 리터럴의 타입으로부터 해당 타입을 추론합니다. 맵에서 키는 문자열이지만 값은 다른 타입을 갖습니다(상한이 Object인 String과 int). 그래서 맵 리터럴은 타입 Map<String, Object>를 가지며, arguments 변수도 마찬가지 입니다.

## 필드와 메서드 추론 Field and method inference

구체적입 타입이 없고 상위 클래스의 필드나 메서드를 재정의한 필드 또는 메서드는 상위 클래스 메서드나 필드의 타입을 상속합니다.

선언하지 않았거나 타입을 상속했지만 초기값을 선언하지 않은 필드는 초기값을 근거로 추론한 타입을 가집니다.

## 정적 필드 추론 Static field inference

정적 필드와 변수는 초기화 구문으로부터 타입을 얻습니다. 순환이 발생하면 추론이 실패하는 것에 주의하세요(즉 변수 타입을 추론하는 것은 그 변수의 타입을 아는 것에 달려있습니다).

## 로컬 변수 추론 Local variable inference

로컬 변수 타입은 초기화 구문(있는 경우)으로부터 추론됩니다. 나중에 할당하는 것은 고려되지 않습니다. 이것은 너무 정확한 타입이 추론될 수 있음을 의미합니다. 그렇다면 타입 주석을 추가할 수 있습니다.

```dart
// 정적 분석: 실패
var x = 3; // x는 int로 추론됩니다.
x = 4.0;
```

```dart
num y = 3; // num은 double 또는 int가 될 수 있습니다.
y = 4.0;
```

## 타입 인자 추론 Type argument inference

생성자 호출과 제네릭 메서드 호출에 대한 타입 인자는 발생 컨텍스트로부터 발생한 하향식 정보와 생성자나 제네릭 메서드의 인자로부터 발생한 상향식 정보의 조합으로 추론됩니다. 추론이 원하거나 기대한 바와 다르면 타입 인자를 명시적으로 규정할 수 있습니다.

```dart
// <int>[]로 추론됩니다.
List<int> listOfInt = [];

// <double>[3.0]으로 추론됩니다.
var listOfDouble = [3.0];

// Iterable<int>로 추론됩니다.
var ints = listOfDouble.map((x) => x.toInt());
```

마지막 예시에서 x는 하향식 정보를 사용하여 double로 추론됩니다. 클로저의 반환 타입은 상향식 정보를 사용하여 int로 추론됩니다. Dart는 map() 메서의 타입 인자를 <int>로 추론할 때 이 반환 타입을 상향식 정보로 사용합니다.

## 타입 대체하기 Substituting types

메서드를 재정의하면 이전 메서드의 타입을 새 메서드의 타입으로 대체하게 됩니다. 이와 비슷하게, 인자를 함수에 전달하는 경우 선언된 타입을 가지는 매개변수의 타입은 실제 인자의 다른 타입으로 대체하게 됩니다. 언제 한 타입을 가진 것을 하위타입이나 상위타입을 가진 것으로 대체할 수 있을 까요?

타입을 대체할 때 소비자와 생산자의 관점에서 생각하는 것은 도움이 됩니다. 소비자는 타입을 받아들이고 생산자는 타입을 만듭니다.

소비자의 타입을 상위 타입으로 대체하고, 생산자의 타입을 하위 타입으로 대체할 수 있습니다.

간단한 타입 할당과 제네릭 타입 할당의 예시를 살펴봅시다.

## 간단한 타입 할당 Simple type assignment

객체를 객체에 할당할 때, 언제 한 타입을 다른 타입으로 대체할 수 있을까요? 그 답은 객체가 소비자인지 생산자인지에 달려있습니다.

다음의 타입 계층 구조를 생각해보세요.

Cat c가 소비자이고 Cat()이 생산자인 다음의 간단한 할당을 고려해보세요.

```dart
Cat c = Cat();
```

소비하는 입장에서 구체적인 타입(Cat)을 소비하는 무언가를 아무것(Anything)이나 소비하는 무언가로 대체하는 것은 안전합니다. 그래서 Cat c를 Animal c로 대체하는 것이 허용됩니다. Animal은 Cat의 상위 타입이기 때문입니다.

```dart
정적 분석: 성공
Animal c = Cat();
```

그러나 Cat c를 MaineCoon c로 대체하면 타입 안전성이 손상됩니다. 상위 클래스 가 Lion과 같이 다른 행동을 가진 Cat 타입을 제공할 수 있기 때문입니다.

```dart
// 정적 분석: 실패
MainCoon c = Cat();
```

## 제네릭 타입 할당 Generic type assignment

제네릭 타입에 대한 규칙도 동일할까요? 그렇습니다. 동물 리스트의 계층 구조를 생각해보세요. Cat 리스트는 Animal 리스트의 하위 타입이고 MainCoon 리스트의 상위 타입입니다.

다음 예시에서 MaineCoon 리스트를 myCats 리스트에 할당할 수 있습니다. List<MaineCoon>은 List<Cat>의 하위타입이기 때문입니다.

```dart
// 정적 분석: 성공
List<MaineCoon> myMaineCoons = ...
List<Cat> myCats = myMaineCoons;
```

다른 방향은 어떨까요? Animal 리스트를 List<Cat>에 할당할 수 있을까요?

```dart
// 정적 분석: 실패
List<Animal> myAnimals = ...
List<Cat> myCats = myAnimals;
```

이 할당은 정적 분석을 통과할 수 없습니다. 이것은 암시적 다운 캐스팅을 생성하기 때문입니다. 이것은 Animal과 같은 비동적 타입에서 허용되지 않습니다.

위와 같은 코드가 정적 분석을 통과하도록 하려면 명시적 캐스팅을 사용할 수 있습니다.

```dart
List<Animal> myAnimals = ...
List<Cat> myCats = myAnimals as List<Cat>;
```

그러나 myAnimals같이 캐스팅되는 리스트의 실제 타입에 따라 명시적 캐스팅은 여전히 런타임에 실패할 수 있습니다.

## 메서드 Methods

메서드를 재정의할 때도 여전히 생산자와 소비자의 규칮기 적용됩니다. 예를 들어:

chase(Animal)같은 소비자의 경우 매개변수 타입을 상위 타입으로 대체할 수 있습니다. parent getter 메서드같은 생산자의 경우 반환 타입을 하위타입으로 대체할 수 있습니다.

더 많은 정보에 대한 것은 메서드를 재정의할 때 건전한 반환 타입을 사용하세요와 메서드를 재정의할 때 건전한 매개변수 타입을 사용하세요를 참고하세요.

## 다른 리소스 Other resources

다음 리소스에는 건전한 Dart에 대한 더 많은 정보가 있습니다.

- 일반적인 타입 문제 해결하기 - 건전한 Dart 코드를 작성할 때 마주치는 에러와 그것을 고치는 방법.

- 타입 승격 실패 해결하기 - 타입 승격 실패 오류를 고치는 방법을 이해하고 배우세요.

- 건전한 null 안전성 - 건전한 null 안전성을 가진 코드를 작성하는 법에 대해 배우세요.

- 정적 분석 커스터마이징 - 분석 옵션 파일을 사용한 분석기와 린터를 세팅하고 커스터마이징 하는 방법.