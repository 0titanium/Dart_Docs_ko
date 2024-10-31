# Improving rendering performance

앱에서 애니메이션을 렌더링하는 것은 성능 측정에서 가장 관심이 높은 주제 중 하나입니다. Flutter의 Skia 엔진과 위젯을 빠르게 생성하고 폐기하는 기능 덕분에 Flutter 애플리케이션은 기본적으로 성능이 뛰어나므로, 우수한 성능을 위해서는 일반적인 실수만 피하면 됩니다.

## 일반적인 조언 General advice

애니메이션이 끊겨 보인다면, 프로파일 모드로 빌드된 앱에서 성능을 프로파일링하고 있는지 확인하세요. 기본 Flutter 빌드는 디버그 모드에서 앱을 생성하므로 실제 출시 성능을 반영하지 않습니다. 자세한 내용은 Flutter의 빌드 모드를 참고하세요.

일반적인 몇 가지 실수:

- 매 프레임마다 예상보다 훨씬 많은 UI를 리빌드하는 경우. 위젯 리빌드를 추적하려면 성능 데이터 표시를 확인하세요.
- ListView를 사용하지 않고 많은 자식 목록을 직접 빌드하는 경우.

성능 평가 및 일반적인 실수에 대한 자세한 내용은 아래 문서를 참고하세요:

- [성능 모범 사례](Performance best practices)
- [Flutter 성능 프로파일링](Flutter performance profiling)

## 모바일 전용 조언

애니메이션의 첫 실행에서만 눈에 띄게 끊김이 발생하나요? 그렇다면 [모바일에서 셰이더 애니메이션 끊김 줄이기](Reduce shader animation jank on mobile)를 참고하세요.

## 웹 전용 조언

다음 연재 기사에서는 Flutter Gallery 앱의 웹 성능을 개선하면서 Flutter Material 팀이 얻은 인사이트를 다룹니다:

- [트리 쉐이킹 및 지연 로딩을 통한 Flutter 웹 앱 성능 최적화](Optimizing performance in Flutter web apps with tree shaking and deferred loading)
- [이미지 플레이스홀더, 프리캐싱, 네비게이션 전환 비활성화를 통한 체감 성능 향상](Improving perceived performance with image placeholders, precaching, and disabled navigation transitions)
- [퍼포먼스가 우수한 Flutter 위젯 빌딩](Building performant Flutter widgets)