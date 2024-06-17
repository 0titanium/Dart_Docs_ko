## 패턴 타입 Pattern types

이 페이지는 다양한 종류의 패턴에 대한 참조입니다. 패턴 작동 방식, Dart에서 패턴을 사용할 수 있는 위치 및 일반적인 사용 사례에 대한 개요는 메인 패턴 페이지를 방문하세요.

## 패턴 우선순위 Pattern precedence

연산자 우선 순위와 유사하게 패턴 평가는 우선 순위 규칙을 따릅니다. 괄호로 묶은 패턴을 사용하여 우선순위가 낮은 패턴을 먼저 평가할 수 있습니다.

이 문서는 우선순위가 오름차순으로 패턴 유형이 나열합니다.

- 논리적 or 패턴은 논리적 and보다 낮은 우선 순위를 가지고 논리적 and 패턴은 관계적 패턴보다 낮은 우선 순위를 가지며 그렇게 나아갑니다.

- 후위 단항 패턴 (캐스팅, null 체크, null assert)는 같은 우선 순위를 공유합니다.

- 남은 기본 패턴은 가장 높은 우선 순위를 공유합니다. 컬렉션 타입 (record, list, map)과 Object 패턴은 다른 데이터를 포함하므로 우선 외부 패턴으로 평가됩니다.

## 논리적 or Logical-or

하위패턴1 || 하위패턴2

논리적 or 패턴은은 ||로 하위 패턴을 구분합니다. 분기 중 하나라도 일치하면 일치합니다. 분기는 왼쪽에서 오른쪽으로 평가됩니다. 분기가 일치하면 나머지는 평가되지 않습니다.

```dart
var isPrimary = switch (color) {
  Color.red || Color.yellow || Color.blue => true,
  _ => false
};
```

논리적 or 패턴의 하위 패턴은 변수를 바인딩할 수 있지만 패턴이 일치하면 하나의 분기만 평가되므로 분기는 동일한 변수 집합을 정의해야 합니다.

## 논리적 and Logical-and

하위패턴1 && 하위패턴2

&&로 구분된 패턴 쌍은 두 하위 패턴이 모두 일치하는 경우에만 일치합니다. 왼쪽 분기가 일치하지 않으면 오른쪽 분기가 평가되지 않습니다.

논리적 and 패턴의 하위 패턴은 변수를 바인딩할 수 있지만, 각 하위 패턴의 변수는 패턴이 일치하는 경우 둘 다 바인딩되므로 겹쳐서는 안 됩니다.

```dart
switch ((1, 2)) {
  // 오류, 두 서브패턴이 'b'를 바인딩하려고 시도함.
  case (var a, var b) && (var b, var c): // ...
}
```

## 관계적 패턴 Relational

== 표현식

< 표현식

관계적 패턴은 동등 또는 관계 연산자(==, !=, <, >, <= 및 >=)를 사용하여 일치하는 값을 지정된 상수와 비교합니다.

상수를 인자로 사용하여 일치하는 값에 대해 적절한 연산자를 호출하면 패턴이 일치하여 true를 반환합니다.

관계적 패턴은 특히 논리적 and 패턴과 결합된 경우 숫자 범위를 일치시키는 데 유용합니다.

```dart
String asciiCharType(int char) {
  const space = 32;
  const zero = 48;
  const nine = 57;

  return switch (char) {
    < space => 'control',
    == space => 'space',
    > space && < zero => 'punctuation',
    >= zero && <= nine => 'digit',
    _ => ''
  };
}
```

## 캐스팅 Cast

foo as String

캐스팅 패턴을 사용하면 값을 다른 하위 패턴에 전달하기 전에 구조 분해 할당중에 타입 캐스트를 삽입할 수 있습니다.

```dart
(num, Object) record = (1, 's');
var (i as int, s as String) = record;
```

값에 명시된 타입이 없으면 캐스트 패턴은 예외를 발생시킵니다. null-assert 패턴과 마찬가지로 이 패턴을 사용하면 일부 구조 분해 할당된 값의 예상 유형을 강제로 assert할 수 있습니다.

## Null 검사 Null-check

하위패턴?

Null 검사 패턴은 값이 null이 아닌 경우 먼저 매칭한 다음 동일한 값에 대해 내부 패턴을 매칭시킵니다. 일치하는 null 허용 값의 null 허용이 아닌 기본 타입인 변수를 바인딩할 수 있습니다.

예외 발생없이 null 값을 매칭 실패로 처리하려면 null 검사 패턴을 사용하세요.

```dart
String? maybeString = 'nullable with base type String';
switch (maybeString) {
  case var s?:
  // 's'는 null 비허용 문자열 타입을 가집니다.
}
```

값이 null일 때 매칭하려면 상수 패턴 null을 사용하세요.


## Null-assert

하위패턴!

null-assert 패턴은 객체가 null이 아닌 경우 먼저 매칭한 다음 값에 매칭합니다. null이 아닌 값의 흐름을 허용하지만 일치하는 값이 null이면 예외를 발생시킵니다.

null 값이 자동으로 일치 실패로 처리되지 않도록 하려면 매칭하는 동안 null-assert 패턴을 사용하세요.

```dart
List<String?> row = ['user', null];
switch (row) {
  case ['user', var name!]: // ...
  // `name'은 널 비허용 문자열입니다.
}
```

변수 선언 패턴에서 null 값을 제거하려면 null-assert 패턴을 사용하세요.

```dart
(int?, int?) position = (2, 3);

var (x!, y!) = position;
```

값이 null일 때 일치시키려면 상수 패턴 null을 사용하세요.

## 상수 Constant

123, null, 'string', math.pi, SomeClass.constant, const Thing(1, 2), const (1 + 2)

값이 상수와 같을 때 상수 패턴이 일치합니다.

```dart
switch (number) {
  // 1 == number 일때 매칭합니다.
  case 1: // ...
}
```

간단한 리터럴과 명명된 상수에 대한 참조를 직접 상수 패턴으로 사용할 수 있습니다.

- 숫자 리터럴 (123, 45.56)

- 부울 리터럴 (true)

- 문자열 리터럴 ('string')

- 명명된 상수 (someConstant, math.pi, double.infinity)

- 상수 생성자 (const Point(0, 0))

- 상수 컬렉션 리터럴 (const [], const {1, 2})

보다 복잡한 상수 표현식은 괄호로 묶고 const(const (1 + 2))에 접두사를 붙여야 합니다.

```dart
// 리스트 또는 맵 패턴:
case [a, b]: // ...

// 리스트 또는 맵 리터럴:
case const [a, b]: // ...
```

## 변수 Variable

var bar, String str, final int _

변수 패턴은 새 변수에 매칭되거나 구조 분해 할당된 값에 바인딩합니다. 일반적으로 구조 분해 할당된 값을 캡처하기 위한 구조 분해 할당 패턴의 일부로 발생합니다.

변수는 패턴이 일치하는 경우에만 도달할 수 있는 코드 영역의 스코프 내에 있습니다.

```dart
switch ((1, 2)) {
  // 'var a'와 'var b'는 1과 2에 바인딩하는 변수 패턴입니다.
  case (var a, var b): // ...
  // 'a'와 'b'는 case 내부 스코프에 있습니다.
}
```

타입이 있는 변수 패턴은 매칭된 값에 선언된 타입이 있는 경우에만 매칭되고 그렇지 않으면 실패합니다:

```dart
switch ((1, 2)) {
  // 매칭되지 않음.
  case (int a, String b): // ...
}
```

와일드카드 패턴을 변수 패턴으로 사용할 수 있습니다.

## 식별자 Identifier

foo, _

식별자 패턴은 나타나는 상황에 따라 상수 패턴이나 변수 패턴처럼 동작할 수 있습니다.

- 선언 컨텍스트: 식별자 이름을 사용하여 새 변수를 선언합니다: var (a, b) = (1, 2);

- 할당 컨텍스트: 식별자 이름이 있는 기존 변수에 할당: (a, b) = (3, 4);

- 매칭 컨텍스트: 명명된 상수 패턴으로 처리됩니다(이름이 _가 아닌 경우).

    ```dart
    const c = 1;
    switch (2) {
    case c:
        print('match $c');
    default:
        print('no match'); // "no match" 출력.
    }
    ```

- 모든 컨텍스트의 와일드카드 식별자: 모든 값과 매칭하고 이를 삭제합니다. case [_, var y, _]: print('The middle element is $y');


## 괄호로 묶기 Parenthesized

(하위패턴)

괄호로 묶인 표현식과 마찬가지로 패턴의 괄호를 사용하면 패턴 우선 순위를 제어하고 우선 순위가 더 높을 것으로 예상되는 곳에 우선 순위가 낮은 패턴을 삽입할 수 있습니다.

예를 들어 부울 상수 x, y, z가 각각 true, true, false와 같다고 가정해 보세요. 다음 예는 부울 표현식 평가와 유사하지만 패턴과 일치합니다.

```dart
// ...
x || y => 'true 일치',
x || y && z => 'true 일치',
x || (y && z) => 'true 일치',
// `x || y && z` 는 `x || (y && z)`과 같습니다.
(x || y) && z => '일치 없음',
// ...
```

다트는 왼쪽에서 오른쪽으로 패턴 매칭을 시작합니다.

1. 첫 번째 패턴은 x가 true와 일치하므로 true와 일치합니다.

2. 두 번째 패턴은 x가 true와 일치하므로 true와 일치합니다.

3. 세 번째 패턴은 x가 true와 일치하므로 true와 일치합니다.

4. 네 번째 패턴 (x || y) && z에는 아무것도 일치하지 않습니다.

## 리스트 List

[하위패턴1, 하위패턴2]

리스트 패턴은 리스트를 구현하는 값과 매칭한 다음 해당 하위 패턴을 리스트 요소와 재귀적으로 매칭해 위치별로 구조 분해 할당합니다.

```dart
const a = 'a';
const b = 'b';
switch (obj) {
  // 리스트 패턴 [a, b]는 obj가 두 개의 필드가 있는 리스트인 경우 먼저 obj와 매칭합니다.
  // 그런 다음 해당 필드가 상수 하위 패턴 'a' 및 'b'와 일치하면 매칭합니다.
  case [a, b]:
    print('$a, $b');
}
```

리스트 패턴에서는 패턴의 요소 수가 전체 리스트와 일치해야 합니다. 그러나 나머지 요소를 자리 표시자로 사용하여 리스트의 요소 수에 관계없이 설명할 수 있습니다.

## 나머지 요소 Rest element

리스트 패턴에는 임의 길이의 목록 일치를 허용하는 하나의 나머지 요소(...)가 포함될 수 있습니다.

```dart
var [a, b, ..., c, d] = [1, 2, 3, 4, 5, 6, 7];
// "1 2 6 7" 출력.
print('$a $b $c $d');
```

나머지 요소에는 리스트의 다른 하위 패턴과 일치하지 않는 요소를 새 리스트로 수집하는 하위 패턴이 있을 수도 있습니다.

```dart
var [a, b, ...rest, c, d] = [1, 2, 3, 4, 5, 6, 7];
// "1 2 [3, 4, 5] 6 7" 출력.
print('$a $b $rest $c $d');
```

## 맵 Map

{"key": 하위패턴1, 어떤상수: 하위패턴2}

맵 패턴은 Map을 구현하는 값과 매칭한 다음 해당 하위 패턴을 맵의 키와 반복적으로 매칭시켜 구조 분해 할당합니다.

맵 패턴에서는 패턴이 전체 맵과 일치할 필요가 없습니다. 맵 패턴은 패턴과 일치하지 않는 맵에 포함된 모든 키를 무시합니다.

## 레코드 Record

(하위패턴, 하위패턴2)

(x: 하위패턴1, y: 하위패턴2)

레코드 패턴은 레코드 객체와 매칭하고 해당 필드를 구조 분해 할당합니다. 값이 패턴과 동일한 모양의 레코드가 아닌 경우 매칭이 실패합니다. 그렇지 않으면 필드 하위 패턴이 레코드의 해당 필드와 매칭됩니다.

레코드 패턴에서는 패턴이 전체 레코드와 매칭되어야 합니다. 패턴을 사용하여 명명된 필드가 있는 레코드를 구조 분해 할당하려면 패턴에 필드 이름을 포함하십시오.

```dart
var (myString: foo, myNumber: bar) = (myString: 'string', myNumber: 1);
```

getter 이름은 생략할 수 있으며 필드 하위 패턴의 변수 패턴 또는 식별자 패턴에서 추론할 수 있습니다. 다음 패턴 쌍은 각각 동일합니다.

```dart
// 변수 하위 패턴을 사용한 레코드 패턴:
var (untyped: untyped, typed: int typed) = record;
var (:untyped, :int typed) = record;

switch (record) {
  case (untyped: var untyped, typed: int typed): // ...
  case (:var untyped, :int typed): // ...
}

// null 검사와 null-assert 하위패턴을 사용한 레코드 패턴:
switch (record) {
  case (checked: var checked?, asserted: var asserted!): // ...
  case (:var checked?, :var asserted!): // ...
}

// Record pattern with cast subpattern 캐스팅 하위패턴을 사용한 레코드 패턴:
var (untyped: untyped as int, typed: typed as String) = record;
var (:untyped as int, :typed as String) = record;
```

## 객체 Object

SomeClass(x: 하위패턴1, y: 하위패턴2)

객체 패턴은 지정된 명명된 타입과 일치하는 값을 확인하여 객체 속성에 대한 getter를 사용하여 데이터를 구조 분해 할당합니다. 값의 타입이 동일하지 않으면 거부됩니다.

```dart
switch (shape) {
  // 타입이 Rect 타입인 경우 매칭하고 Rect의 속성과 매칭합니다.
  case Rect(width: var w, height: var h): // ...
}
```

getter 이름은 생략할 수 있으며 필드 하위 패턴의 변수 패턴 또는 식별자 패턴에서 추론할 수 있습니다.

```dart
// 새 변수 x 및 y를 Point의 x 및 y 속성 값에 바인딩합니다.
var Point(:x, :y) = Point(1, 2);
```

객체 패턴에서는 패턴이 전체 객체와 일치할 필요가 없습니다. 객체에 패턴이 구조 분해 할당되지 않는 추가 필드가 있는 경우에도 여전히 일치할 수 있습니다.

## 와일드카드 Wildcard

_

_라는 패턴은 어떤 변수에도 바인딩되거나 할당되지 않는 와일드카드(변수 패턴 또는 식별자 패턴)입니다.

나중에 위치 값을 구조 분해 할당하기 위해 하위 패턴이 필요한 위치에서 자리 표시자로 유용합니다.

```dart
var list = [1, 2, 3];
var [_, two, _] = list;
```

타입 주석이 있는 와일드카드 이름은 값의 타입을 테스트하려고 하지만 값을 이름에 바인딩하지 않으려는 경우에 유용합니다.

```dart
switch (record) {
  case (int _, String _):
    print('First field is int and second is String.');
}
```