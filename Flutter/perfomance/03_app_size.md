# 앱 크기 측정 Measuring your app's size

많은 개발자들이 컴파일된 앱의 크기에 대해 걱정합니다. Flutter 앱의 APK, 앱 번들 또는 IPA 버전은 자체 포함되어 있으며 앱을 실행하는 데 필요한 모든 코드와 자산을 포함하므로 크기가 문제가 될 수 있습니다. 앱의 크기가 클수록 장치에서 필요한 공간이 많아지고 다운로드 시간이 길어지며, Android 인스턴트 앱과 같은 유용한 기능의 한계를 초과할 수 있습니다.

## 디버그 빌드는 대표적이지 않음 Debug builds are not representative

기본적으로 `flutter run` 명령어로 앱을 실행하거나 IDE에서 재생 버튼을 클릭하면 (테스트 드라이브 및 첫 Flutter 앱 작성 시 사용됨) Flutter 앱의 디버그 빌드가 생성됩니다. 디버그 빌드는 핫 리로드 및 소스 수준 디버깅을 허용하는 디버깅 오버헤드로 인해 크기가 큽니다. 따라서 이는 최종 사용자가 다운로드하는 프로덕션 앱을 대표하지 않습니다.

## 총 크기 확인 Checking the total size

`flutter build apk` 또는 `flutter build ios`와 같이 기본 릴리스 빌드는 Play 스토어와 App 스토어에 업로드 패키지를 편리하게 구성하도록 빌드됩니다. 따라서 이 빌드 또한 최종 사용자의 다운로드 크기를 대표하지 않습니다. 스토어에서는 일반적으로 업로드 패키지를 재처리하고 분할하여 특정 다운로드 사용자와 해당 하드웨어를 대상으로 합니다. 예를 들어, 전화의 DPI에 맞는 자산을 필터링하거나 전화의 CPU 아키텍처에 맞는 네이티브 라이브러리를 필터링합니다.

## 총 크기 추정 Estimating total size

각 플랫폼에서 가장 근접한 크기를 추정하려면 다음 지침을 사용하세요.

### Android

Google Play Console의 지침을 따라 앱 다운로드 및 설치 크기를 확인하세요.

애플리케이션에 대한 업로드 패키지를 생성하세요:

```
flutter build appbundle
```

Google Play Console에 로그인합니다. .aab 파일을 드래그 앤 드롭하여 애플리케이션 바이너리를 업로드합니다.

Android Vitals의 "앱 크기" 탭에서 애플리케이션의 다운로드 및 설치 크기를 확인하세요.

다운로드 크기는 XXXHDPI (~640dpi) 장치에서 arm64-v8a 아키텍처를 기준으로 계산됩니다. 최종 사용자의 다운로드 크기는 하드웨어에 따라 달라질 수 있습니다.

상단 탭에는 다운로드 크기와 설치 크기를 전환할 수 있는 토글이 있습니다. 페이지 하단에는 최적화 팁도 포함되어 있습니다.

## iOS

Xcode 앱 크기 보고서를 생성합니다.

먼저, iOS 빌드 아카이브 지침에 따라 앱 버전 및 빌드를 구성합니다.

그런 다음:

1. `flutter build ipa --export-method development` 명령을 실행합니다.
2. `open build/ios/archive/*.xcarchive`를 실행하여 Xcode에서 아카이브를 엽니다.
3. **Distribute App**을 클릭합니다.
4. 배포 방법을 선택합니다. 애플리케이션을 배포할 계획이 없다면 **Development**가 가장 간단합니다.
5. **App Thinning**에서 '모든 호환 가능한 장치 변형(all compatible device variants)'을 선택합니다.
6. **Strip Swift symbols**를 선택합니다.
7. IPA에 서명하고 내보냅니다. 내보낸 디렉토리에는 다양한 장치 및 iOS 버전에서의 예상 애플리케이션 크기에 대한 세부정보가 포함된 **App Thinning Size Report.txt**가 포함되어 있습니다.

Flutter 1.17의 기본 데모 앱에 대한 앱 크기 보고서는 다음과 같습니다:

```
Variant: Runner-7433FC8E-1DF4-4299-A7E8-E00768671BEB.ipa
Supported variant descriptors: [device: iPhone12,1, os-version: 13.0] and [device: iPhone11,8, os-version: 13.0]
App + On Demand Resources size: 5.4 MB compressed, 13.7 MB uncompressed
App size: 5.4 MB compressed, 13.7 MB uncompressed
On Demand Resources size: Zero KB compressed, Zero KB uncompressed
```

이 예제에서 앱은 iOS 13.0을 실행하는 iPhone12,1 (iPhone 11의 모델 ID / 하드웨어 번호) 및 iPhone11,8 (iPhone XR)에서 대략 5.4 MB의 다운로드 크기와 약 13.7 MB의 설치 크기를 가집니다.

iOS 앱의 정확한 크기를 측정하려면, 릴리스 IPA를 Apple의 App Store Connect에 업로드하고(지침 참조) 그곳에서 크기 보고서를 얻어야 합니다. IPA는 일반적으로 APK보다 크기가 더 크며, 이는 Flutter FAQ의 "Flutter 엔진의 크기는 얼마나 됩니까?" 섹션에서 설명되어 있습니다.

## 크기 분석 Breaking down the size

Flutter 1.22 버전 및 DevTools 0.9.1 버전부터, 개발자가 애플리케이션의 릴리스 빌드 크기를 이해하는 데 도움이 되는 크기 분석 도구가 포함되었습니다.

```
경고

위의 총 크기 확인 섹션에서 언급한 바와 같이, 업로드 패키지는 최종 사용자의 다운로드 크기를 대표하지 않습니다. 크기 분석 도구에서 볼 수 있는 중복된 네이티브 라이브러리 아키텍처와 자산 밀도는 Play 스토어와 App Store에서 필터링될 수 있습니다.
```

크기 분석 도구는 빌드할 때 `--analyze-size` 플래그를 전달하여 호출됩니다:

- flutter build apk --analyze-size
- flutter build appbundle --analyze-size
- flutter build ios --analyze-size
- flutter build linux --analyze-size
- flutter build macos --analyze-size
- flutter build windows --analyze-size

이 빌드는 표준 릴리스 빌드와 두 가지 점에서 다릅니다.

1. 도구는 Dart 패키지의 코드 크기 사용량을 기록하는 방식으로 Dart를 컴파일합니다.
2. 도구는 터미널에 크기 분석의 높은 수준 요약을 표시하고, DevTools에서 더 자세한 분석을 위한 *-code-size-analysis_*.json 파일을 남깁니다.

단일 빌드를 분석하는 것 외에도, 두 개의 *-code-size-analysis_*.json 파일을 DevTools에 로드하여 두 빌드를 비교할 수도 있습니다. 자세한 내용은 DevTools 문서를 참조하세요.

요약을 통해 자산, 네이티브 코드, Flutter 라이브러리 등과 같은 카테고리별 크기 사용량에 대한 빠른 개요를 얻을 수 있습니다. 컴파일된 Dart 네이티브 라이브러리는 패키지별로 추가로 분류되어 빠른 분석이 가능합니다.

```
경고

iOS에서 이 도구는 IPA 대신 .app 파일을 생성합니다. 이 도구를 사용하여 .app의 콘텐츠 상대 크기를 평가하세요. 다운로드 크기에 대한 더 정확한 추정을 얻으려면 위의 총 크기 추정 섹션을 참조하세요.
```

## DevTools에서의 심층 분석 Deeper analysis in DevTools

위에서 생성된 *-code-size-analysis_*.json 파일은 DevTools에서 더 자세히 분석할 수 있으며, 여기서 트리 또는 트리맵 뷰를 통해 애플리케이션의 콘텐츠를 개별 파일 수준과 Dart AOT 아티팩트의 함수 수준까지 세분화할 수 있습니다.

이 작업은 `dart devtools`를 실행하고 "Open app size tool"을 선택한 후 JSON 파일을 업로드하여 수행할 수 있습니다.

DevTools 앱 크기 도구 사용에 대한 추가 정보는 DevTools 문서를 참조하세요.

## 앱 크기 줄이기 Reducing app size

앱의 릴리스 버전을 빌드할 때 `--split-debug-info` 태그를 사용하는 것을 고려하세요. 이 태그는 코드 크기를 크게 줄일 수 있습니다. 이 태그 사용에 대한 예는 Dart 코드 난독화에서 확인할 수 있습니다.

앱을 더 작게 만드는 다른 방법은 다음과 같습니다:

- 사용하지 않는 리소스 제거
- 라이브러리에서 가져온 리소스 최소화
- PNG 및 JPEG 파일 압축