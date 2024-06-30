# 확장 타입 Extension types

확장 타입은 기존 타입을 다른 정적 전용 인터페이스로 "래핑"하는 컴파일 타임 추상화입니다. 실제 래퍼 비용을 들이지 않고도 기존 타입의 인터페이스(모든 종류의 상호 운용성에 중요한)를 쉽게 수정할 수 있기 때문에 이는 정적 JS 상호 운용성의 주요 구성 요소입니다.

확장 타입은 표현 타입 이라고 하는 기본 타입의 객체에 사용 가능한 작업(또는 인터페이스)집합에 규칙을 적용합니다. 확장 타입의 인터페이스를 정의할 때 표현 타입의 일부 멤버를 재사용하고, 다른 멤버를 생략하고, 다른 멤버를 대체하고, 새로운 기능을 추가하도록 선택할 수 있습니다.

다음 예에서는 int 타입을 래핑하여 ID 번호에 적합한 작업만 허용하는 확장 타입을 만듭니다.

```dart
extension type IdNumber(int id) {
  // int 타입의 < 연산자를 래핑:
  operator <(IdNumber other) => id < other.id;
  // + 연산자를 선언하지 마세요, 예를 들어,
  // 더하기는 ID numbers에 의미가 없기 때문입니다.
}

void main() {
  // 확장 타입의 규칙 없이,
  // int는 안전하지 않은 작업에 ID numbers를 노출합니다:
  int myUnsafeId = 42424242;
  myUnsafeId = myUnsafeId + 10; // 작동하지만, ID에 허용되어서는 안됩니다.

  var safeId = IdNumber(42424242);
  safeId + 10; // 컴파일 타임 오류: + 연산자가 없습니다.
  myUnsafeId = safeId; // 컴파일 타임 오류: 잘못된 타입. Compile-time error: Wrong type.
  myUnsafeId = safeId as int; // 괜찮습니다: 표현 타입으로 런타임 캐스팅 OK: Run-time cast to representation type.
  safeId < IdNumber(42424241); // 괜찮습니다: 래핑된 < 연산자를 사용합니다.
}
```

```
노트

확장 타입은 래퍼 클래스 와 동일한 목적을 제공 하지만, 많은 객체를 래핑해야 할 때 
비용이 많이 들 수 있는 추가 런타임 객체를 만들 필요가 없습니다. 
확장 타입은 정적 전용이고 런타임에 컴파일되므로 본질적으로 비용이 없습니다.

확장 메서드(또는 "확장"이라고도 함)는 확장 타입과 유사한 정적 추상화입니다. 
그러나 확장 메서드는 기본 타입의 모든 인스턴스에 직접 기능을 추가합니다. 확장 타입은 다릅니다. 
확장 타입의 인터페이스는 정적 타입이 해당 확장 타입인 표현식에만 적용됩니다. 
기본적으로 기본 타입의 인터페이스와 다릅니다.
```

## 문법 Syntax

## 선언 Declaration

extension type선언과 이름을 사용하여 새 확장 타입을 정의하고, 그 뒤에 괄호로 표현 타입 선언을 추가합니다.

```dart
extension type E(int i) {
  // 작업의 집합을 정의합니다.
}
```

표현 타입 선언(int i)은 확장 타입 E의 기본 타입이 int이고 표현 객체에 대한 참조가 i임을 지정합니다. 선언은 또한 다음을 소개합니다:

- 표현 타입을 반환 타입으로 사용하는 표현 객체에 대한 암시적 getter : int get i.

- 암시적 생성자 : E(int i) : i = i.

표현 객체는 확장 타입이 기본 타입의 객체에 접근할 수 있도록 합니다. 객체는 확장 타입 본문의 범위에 있으며, 이름을 getter로 사용하여 접근할 수 있습니다:

- i(또는 생성자 내의 this.i)를 사용한 확장 타입 본문 내부에서.

- e.i를 사용한 속성 추출이 있는 외부(e는 확장 타입을 정적 타입으로 가짐).

확장 타입 선언은 클래스나 확장과 마찬가지로 타입 매개변수도 포함할 수 있습니다.

```dart
extension type E<T>(List<T> elements) {
  // ...
}
```

## 생성자 Constructor

선택적으로 확장 타입의 본문에서 생성자를 선언할 수 있습니다. 표현 선언 자체는 암시적 생성자이므로 기본적으로 확장 타입에 대해 명명되지 않은 생성자를 대신합니다. 리디렉션되지 않는 추가 생성형 생성자는 초기화 리스트의 this.i나 형식 매개변수를 사용하여 표현 객체의 인스턴스 변수를 초기화해야 합니다.

```dart
extension type E(int i) {
  E.n(this.i);
  E.m(int j, String foo) : i = j + foo.length;
}

void main() {
  E(4); // 암시적 이름없는 생성자.
  E.n(3); // 이름있는 생성자.
  E.m(5, "Hello!"); // 추가 매개변수가 있는 이름있는 생성자.
}
```

또는 표현 선언 생성자의 이름을 지정할 수 있습니다. 이 경우 본문에 이름없는 생성자를 위한 공간이 있습니다.

```dart
extension type const E._(int it) {
  E(): this._(42);
  E.otherName(this.it);
}

void main2() {
  E();
  const E._(2);
  E.otherName(3);
}
```

클래스에 대해 동일한 private 생성자 문법 _을 사용하여 새 생성자를 정의하는 대신 생성자를 완전히 숨길 수도 있습니다. 예를 들어, 기본 타입이 int임에도 클라이언트를 String을 사용한 E로 구성하려는 경우:

```dart
extension type E._(int i) {
  E.fromString(String foo) : i = int.parse(foo);
}
```

생성형 생성자 또는 팩토리 생성자(하위 확장 타입의 생성자로 전달할 수도 있음) 전달을 선언할 수도 있습니다.

## 멤버 Members

클래스 멤버와 동일한 방식으로 인터페이스를 정의하려면 확장 타입의 본문에서 멤버를 선언하세요. 확장 타입 멤버는 메서드, getter, setter 또는 연산자일 수 있습니다(external이 아닌 인스턴스 변수 및 추상 멤버는 허용되지 않음):

```dart
extension type NumberE(int value) {
  // 연산자:
  NumberE operator +(NumberE other) =>
      NumberE(value + other.value);
  // getter:
  NumberE get myNum => this;
  // 메서드:
  bool isValid() => !value.isNegative;
}
```

표현 타입의 인터페이스 멤버는 기본적으로 확장 타입의 인터페이스 멤버가 아닙니다. 확장 타입에서 사용 가능한 표현 타입의 단일 멤버를 만들려면 NumberE의 + 연산자와 같이 확장 타입 정의에 이에 대한 선언을 작성해야 합니다. i getter 및 isValid 메서드와 같이 표현 타입과 관련되지 않은 새 멤버를 정의할 수도 있습니다.

## 구현 Implements

선택적으로 implements 절을 사용하여 다음을 수행할 수 있습니다.

- 확장 타입에 하위 타입 관계를 도입하고

- 표현 객체의 멤버를 확장 타입 인터페이스에 추가합니다.

implements 절은 확장 메서드와 해당 on 타입 간의 관계와 같은 적용 가능성 관계를 소개합니다. 상위 타입에 적용 가능한 멤버는 하위 타입에 동일한 멤버 이름을 가진 선언이 없는 한 하위 타입에도 적용 가능합니다.

확장 타입은 다음만 구현할 수 있습니다.

- 표현 타입. 이렇게 하면 표현 타입의 모든 멤버를 확장 타입에서 암시적으로 사용할 수 있습니다.
  ```dart
  extension type NumberI(int i) 
  implements int{
  // 'NumberI'는 'int'의 모든 멤버를 호출할 수 있고,
  // 추가로 여기에 어떤 것이든 선언할 수 있습니다.
  }
  ```

- 표현 타입의 상위 타입. 이것은 상위 타입의 멤버를 사용 가능하게 하지만 표현 타입의 모든 멤버를 반드시 사용할 수 있는 것은 아닙니다.
  ```dart
  extension type Sequence<T>(List<T> _) implements Iterable<T> {
    // 리스트 보다 나은 작업.
  }

  extension type Id(int _id) implements Object {
    // 확장 타입을 널이 아니게 합니다.
    static Id? tryParse(String source) => int.tryParse(source) as Id?;
  }
  ```

- 동일한 표현 타입에 유효한 또 다른 확장 타입입니다. 이를 통해 여러 확장 타입에 작업을 재사용할 수 있습니다(다중 상속과 유사).
  ```dart
  extension type const Opt<T>._(({T value})? _) { 
  const factory Opt(T value) = Val<T>;
  const factory Opt.none() = Non<T>;
  }
  
  extension type const Val<T>._(({T value}) _) implements Opt<T> { 
    const Val(T value) : this._((value: value));
    T get value => _.value;
  }
  
  extension type const Non<T>._(Null _) implements Opt<Never> {
    const Non() : this._(null);
  }
  ```

다양한 시나리오에서 implents의 효과에 대해 자세히 알아보려면 사용법 섹션을 읽어보세요.

## @redeclare

상위 타입의 멤버와 이름을 공유하는 확장 타입 멤버를 선언하는 것은 클래스 간의 재정의 관계가 아니라 재선언 입니다. 확장 타입 멤버 선언은 동일한 이름을 가진 상위 타입 멤버를 완전히 대체합니다. 동일한 기능에 대해 대체 구현을 제공하는 것은 불가능합니다.

@redeclare 주석을 사용하여 컴파일러에게 상위 타입의 멤버와 동일한 이름을 사용하도록 의도적으로 선택했음을 알릴 수 있습니다. 그러면 분석기가 실제로 그렇지 않은 경우, 예를 들어 이름 중 하나가 잘못 입력된 경우 경고합니다.

```dart
extension type MyString(String _) implements String {
  // 'String.operator[]'를 대체합니다.
  @redeclare
  int operator [](int index) => codeUnitAt(index);
}
```

상위 인터페이스 멤버를 숨기고 @redeclare로 주석을 달지 않는 확장 타입 메서드를 선언하는 경우, 경고를 표시하도록 린트 annotate_redeclares를 활성화할 수도 있습니다.

## 사용법 Usage

확장 타입을 사용하려면 클래스에서와 동일한 방식으로 생성자를 호출하여 인스턴스를 만듭니다.

```dart
extension type NumberE(int value) {
  NumberE operator +(NumberE other) =>
      NumberE(value + other.value);

  NumberE get next => NumberE(value + 1);
  bool isValid() => !value.isNegative;
}

void testE() { 
  var num = NumberE(1);
}
```

그런 다음 클래스 객체와 마찬가지로 객체의 멤버를 호출할 수 있습니다.

확장 타입에는 동일하게 유효하지만 실질적으로 다른 두 가지 핵심 사용 사례가 있습니다.

1. 기존 타입에 확장된 인터페이스를 제공합니다.
2. 기존 타입에 다른 인터페이스를 제공합니다.

```
노트

어떤 경우에도 확장 타입의 표현 타입은 결코 하위 타입이 아니므로, 
확장 타입이 필요한 곳에서 표현 타입을 서로 바꿔 사용할 수 없습니다.
```

1. 기존 타입에 확장된 인터페이스 제공 Provide an extended interface to an existing type

확장 타입이 해당 표현 타입을 구현할 때 확장 타입이 기본 타입을 "볼" 수 있도록 허용하므로 이를 "투명"하다고 간주할 수 있습니다.

투명한 확장 타입은 표현 타입(재선언되지 않은)의 모든 멤버와 정의된 보조 멤버를 호출할 수 있습니다. 그러면 기존 타입에 대한 새로운 확장 인터페이스가 생성됩니다. 새 인터페이스는 정적 타입이 확장 타입인 표현식에 사용할 수 있습니다.

이는 다음과 같이 (비투명 확장 타입과 달리) 표현 타입의 멤버를 호출할 수 있음을 의미합니다.

```dart
extension type NumberT(int value) 
  implements int {
  // 'int'의 멤버를 명시적으로 선언하지 않습니다.
  NumberT get i => this;
}

void main () {
  // 전부 괜찮습니다: 투명성은 확장 타입에 'int' 멤버를 호출할 수 있게 합니다:
  var v1 = NumberT(1); // v1 타입: NumberT
  int v2 = NumberT(2); // v2 타입: int
  var v3 = v1.i - v1;  // v3 타입: int
  var v4 = v2 + v1; // v4 타입: int
  var v5 = 2 + v1; // v5 타입: int
  // 오류: 확장 타입 인터페이스는 표현 타입에 사용할 수 없습니다
  v2.i;
}
```

또한 상위 타입에서 주어진 멤버 이름을 재선언하여 새 멤버를 추가하고 다른 멤버를 조정하는 "대부분 투명한" 확장 타입을 가질 수도 있습니다. 예를 들어, 이를 통해 메서드의 일부 매개변수에 대해 더 엄격한 타입을 사용하거나 다른 기본값을 사용할 수 있습니다.

또 다른 투명한 확장 타입 접근 방식은 표현 타입의 상위 타입인 타입을 구현하는 것입니다. 예를 들어 표현 타입이 private이지만 해당 상위 타입이 클라이언트에 중요한 인터페이스 부분을 정의하는 경우입니다.

2. 기존 타입과 다른 인터페이스 제공 Provide a different interface to an existing type

투명하지 않은(해당 표현 유형을 구현하지 않는) 확장 타입은 해당 표현 타입과 구별되는 완전히 새로운 타입으로 정적으로 처리됩니다. 표현 타입에 할당할 수 없으며 표현 타입의 멤버를 노출하지 않습니다.

예를 들어 사용법 아래에 선언한 NumberE 확장 타입을 사용합니다.

```dart
void testE() { 
  var num1 = NumberE(1);
  int num2 = NumberE(2); // 오류: 'NumberE'를 'int'에 할당할 수 없습니다.
  
  num1.isValid(); // 괜찮습니다: 확장 멤버 호출.
  num1.isNegative(); // 오류: 'NumberE'는'int' 멤버 'isNegative'를 정의하지 않습니다.
  
  var sum1 = num1 + num1; // 괜찮습니다: 'NumberE'는 '+'를 정의합니다.
  var diff1 = num1 - num1; // 오류: 'NumberE'는'int' 멤버 '-'를 정의하지 않습니다.
  var diff2 = num1.value - 2; // 괜찮습니다: 참조로 표현 객체에 접근할 수 있습니다.
  var sum2 = num1 + 2; // 오류: 'NumberE' 매개변수 타입에 'int'를 할당할 수 없습니다. 
  
  List<NumberE> numbers = [
    NumberE(1), 
    num1.next, // 괜찮습니다: 'next' getter는 'NumberE' 타입을 반환합니다.
    1, // 오류: 'NumberE' 리스트 타입에 'int' 요소를 할당할 수 없습니다.
  ];
}
```

이 방법으로 확장 타입을 사용하여 기존 타입의 인터페이스를 바꿀 수 있습니다. 이를 통해 새로운 타입의 제약 조건(소개의 IdNumber 예제와 같은)에 적합한 인터페이스를 모델링하는 동시에 int와 같이 간단한 미리 정의된 타입의 성능과 편리함의 이점을 누릴 수 있습니다.

## 타입 고려사항 Type considerations

확장 타입은 컴파일 타임 래핑 구문입니다. 런타임에는 확장 타입에 대한 추적이 전혀 없습니다. 모든 타입 쿼리 또는 유사한 런타임 작업은 표현 타입에 대해 작동합니다.

이는 확장 타입을 안전하지 않은 추상화로 만듭니다. 왜냐하면 런타임에 항상 표현 타입을 찾아 기본 객체에 접근할 수 있기 때문입니다.

dynamic 타입 테스트(e is T), 캐스팅(e as T) 및 기타 런타임 타입 쿼리(예: switch(e) ... 또는 if(e case ...))는 모두 기본 표현 객체로 평가되고, 해당 객체의 런타임 타입에 대해 타입 검사를 합니다. 이는 e의 정적 타입이 확장 타입인 경우와 확장 타입(case MyExtensionType(): ...)에 대해 검사할 때에도 마찬가지입니다.

```dart
void main() {
  var n = NumberE(1);

  // 'n'의 런타임 유형은 표현 타입 'int'입니다.
  if (n is int) print(n.value); // 1 출력.

  // 런타임에 'n'에 대해 'int' 메서드를 사용할 수 있습니다.
  if (n case int x) print(x.toRadixString(10)); // 1 출력.
  switch (n) {
    case int(:var isEven): print("$n (${isEven ? "even" : "odd"})"); // 1(odd) 출력.
  }
}
```

마찬가지로 일치하는 값의 정적 타입은 아래 예시에서 확장 타입의 타입입니다.

```dart
void main() {
  int i = 2;
  if (i is NumberE) print("It is"); // 'It is' 출력.
  if (i case NumberE v) print("value: ${v.value}"); // 'value: 2' 출력.
  switch (i) {
    case NumberE(:var value): print("value: $value"); // 'value: 2' 출력.
  }
}
```

확장 타입을 사용할 때 이러한 퀄리티를 인식하는 것이 중요합니다. 확장 타입이 컴파일 타임에 존재하고 중요하지만 컴파일 중에 지워진다는 점을 항상 염두에 두십시오.

예를 들어 정적 타입이 확장 타입 E이고 E의 표현 타입이 R인 표현식 e를 생각해 보세요. 그러면 e 값의 런타임 타입은 R의 하위 타입입니다. 타입 자체도 지워집니다. List<E>는 런타임에 List<R>과 완전히 동일합니다.

즉, 실제 래퍼 클래스는 래핑된 객체를 캡슐화할 수 있는 반면 확장 타입은 래핑된 객체에 대한 컴파일 타임 뷰일 뿐입니다. 실제 래퍼가 더 안전하지만 확장 타입이 래퍼 객체를 피할 수 있는 옵션을 제공하므로 일부 시나리오에서는 성능이 크게 향상될 수 있다는 트레이드 오프가 있습니다.
