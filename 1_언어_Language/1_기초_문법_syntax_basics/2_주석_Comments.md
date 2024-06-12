# 주석 Comments

Dart는 단일 주석, 여러 줄 주석, 문서 주석을 지원합니다.

## 단일 주석 Single-line comments

단일 주석은 //로 시작합니다. //와 줄 끝 사이의 모든 내용은 Dart 컴파일러에서 무시됩니다.

```dart
void main() {
  // 할 일: AbstractLlamaGreetingFactory로 리팩터링?
  print('Welcome to my Llama farm!');
}
```

## 여러 줄 주석 Multi-line comments

여러 줄 주석은 \/*로 시작해서 *\/로 끝납니다. /*와 /*사이의 모든 내용은 Dart 컴파일러에서 무시됩니다. (주석이 문서 주석이 아니라면; 다음 섹션을 보세요.) 여러 줄 주석은 중첩 가능합니다.

```dart
void main() {
  /*
   * 많은 작업 입니다. 닭을 키우는 것을 고려하세요.

  Llama larry = Llama();
  larry.feed();
  larry.exercise();
  larry.clean();
   */
}
```

## 문서 주석 Documentation comments

문서 주석은 ///나 /**로 시작하는 여러 줄 주석 또는 단일 주석입니다. 연속된 줄에 ///를 사용

