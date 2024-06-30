## 호출 가능한 객체 Callable objects

Dart 클래스의 인스턴스가 함수처럼 호출되도록 하려면 call() 메서드를 구현하세요.

call() 메서드는 call() 메서드를 정의하는 모든 클래스의 인스턴스가 함수를 모방할 수 있습니다. 이 메서드는 매개변수, 반환 타입 등 일반 함수와 같은 기능을 지원합니다.

다음 예시에서 WannabeFunction 클래스는 세 개의 문자열을 받아서 연결하고 각각을 공백으로 구분한 후 느낌표를 추가하는 call() 함수를 정의합니다. 실행을 클릭하여 코드를 실행합니다.

```dart
class WannabeFunction {
  String call(String a, String b, String c) => '$a $b $c!';
}

var wf = WannabeFunction();
var out = wf('Hi', 'there,', 'gang');

void main() => print(out);
```