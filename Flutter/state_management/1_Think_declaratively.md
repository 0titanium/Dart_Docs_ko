# 선언적으로 생각하기 시작하기 Start thinking declaratively

명령형 프레임워크(예: Android SDK 또는 iOS UIKit)에서 Flutter를 사용하는 경우 새로운 관점에서 앱 개발에 대해 생각해야 합니다.

여러분이 생각하는 많은 가정이 Flutter에는 적용되지 않습니다. 예를 들어 Flutter에서는 UI를 수정하는 대신 UI의 일부를 처음부터 다시 빌드해도 괜찮습니다. Flutter는 필요한 경우 모든 프레임에서라도 이를 수행할 수 있을 만큼 빠릅니다.

Flutter는 선언적입니다. 이는 Flutter가 앱의 현재 상태를 반영하기 위해 사용자 인터페이스를 구축한다는 것을 의미합니다.

앱 상태가 변경되면(예: 사용자가 설정 화면에서 스위치를 전환하는 경우) 상태가 변경되고 이로 인해 사용자 인터페이스가 다시 그려집니다. UI 자체(예: widget.setText)의 명령적 변경은 없습니다. 상태를 변경하면 UI가 처음부터 다시 빌드됩니다.

시작 가이드에서 UI 프로그래밍에 대한 선언적 접근 방식에 대해 자세히 알아보세요.

UI 프로그래밍의 선언적 스타일에는 많은 이점이 있습니다. 놀랍게도 UI의 모든 상태에 대한 코드 경로는 단 하나뿐입니다. 주어진 상태에 대해 UI가 어떤 모습이어야 하는지를 한 번만 설명하면 그게 전부입니다.

처음에는 이러한 프로그래밍 스타일이 명령형 스타일만큼 직관적이지 않은 것처럼 보일 수 있습니다. 이것이 바로 이 섹션이 여기에 있는 이유입니다. 계속 읽어보세요.