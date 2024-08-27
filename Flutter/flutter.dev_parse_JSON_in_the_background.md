# 백그라운드에서 JSON 파싱 Parse JSON in the background

기본적으로 Dart 앱은 모든 작업을 싱글 스레드에서 수행합니다. 많은 경우 이 모델은 코딩을 간소화하고 충분히 빠르기 때문에 앱 성능이 저하되거나 종종 "쟁크"라고 불리는 끊기는 애니메이션이 발생하지 않습니다.

하지만 매우 큰 JSON 문서를 파싱하는 것과 같이 비용이 많이 드는 계산을 수행해야 할 수도 있습니다. 이 작업이 16밀리초 이상 걸리면 사용자는 쟁크를 경험하게 됩니다.

쟁크를 피하려면 이와 같은 비용이 많이 드는 계산을 백그라운드에서 수행해야 합니다. Android에서 이는 다른 스레드에 작업을 예약하는 것을 의미합니다. Flutter에서는 별도의 Isolate를 사용할 수 있습니다 . 이 레시피는 다음 단계를 사용합니다.

1. http 패키지를 추가합니다.
2. http 패키지를 사용하여 네트워크 요청을 만듭니다.
3. 응답을 사진 목록으로 변환합니다.
4. 이 작업을 별도의 아이솔레이트로 옮깁니다.

## 1. http 패키지 추가 Add the http package

먼저, 프로젝트에 http 패키지를 추가합니다. http 패키지는 JSON 엔드포인트에서 데이터를 가져오는 것과 같은 네트워크 요청을 수행하기 쉽게 해줍니다.

http 패키지를 dependency로 추가하려면 flutter pub add를 실행하세요:

```
flutter pub add http
```

## 2. 네트워크 요청하기 Make a network request

이 예제에서는 JSONPlaceholder REST API에서 http.get() 메서드를 사용하여 5000개의 사진 객체 목록이 포함된 대용량 JSON 문서를 가져오는 방법을 다룹니다.

```dart
Future<http.Response> fetchPhotos(http.Client client) async {
  return client.get(Uri.parse('https://jsonplaceholder.typicode.com/photos'));
}
```

```
노트

이 예에서 함수에 http.Client를 제공하고 있습니다. 이렇게 하면 함수를 다른 환경에서 테스트하고 사용하기가 더 쉬워집니다.
```

## 3. JSON을 파싱하고 사진 목록으로 변환합니다. Parse and convert the JSON into a list of photos

다음으로, 인터넷에서 데이터 가져오기 레시피의 안내에 따라, http.Response를 Dart 객체 리스트로 변환합니다. 이렇게 하면 데이터를 다루기가 더 쉬워집니다.

## Photo 클래스 만들기 Create a Photo class

먼저, Photo사진에 대한 데이터를 포함하는 클래스를 만듭니다. JSON 객체로 시작하는 Photo 클래스를 쉽게 만들 수 있도록 fromJson() 팩토리 메서드를 포함합니다.

```dart
class Photo {
  final int albumId;
  final int id;
  final String title;
  final String url;
  final String thumbnailUrl;

  const Photo({
    required this.albumId,
    required this.id,
    required this.title,
    required this.url,
    required this.thumbnailUrl,
  });

  factory Photo.fromJson(Map<String, dynamic> json) {
    return Photo(
      albumId: json['albumId'] as int,
      id: json['id'] as int,
      title: json['title'] as String,
      url: json['url'] as String,
      thumbnailUrl: json['thumbnailUrl'] as String,
    );
  }
}
```

# 응답을 사진 목록으로 변환하세요 Convert the response into a list of photos

이제 다음 지침을 사용하여 fetchPhotos() 함수를 업데이트하여 Future<List<Photo>>를 반환합니다.

1. 응답 본문을 List<Photo>으로 변환하는 parsePhotos() 함수를 만듭니다.
2. fetchPhotos() 함수 안에서 parsePhotos() 함수를 사용하세요.

```dart
// 응답 본문을 List<Photo>으로 변환하는 함수.
List<Photo> parsePhotos(String responseBody) {
  final parsed =
      (jsonDecode(responseBody) as List).cast<Map<String, dynamic>>();

  return parsed.map<Photo>((json) => Photo.fromJson(json)).toList();
}

Future<List<Photo>> fetchPhotos(http.Client client) async {
  final response = await client
      .get(Uri.parse('https://jsonplaceholder.typicode.com/photos'));

  // 메인 아이솔레이트에서 parsePhotos를 동기적으로 실행.
  return parsePhotos(response.body);
}
```

# 4. 이 작업을 별도의 아이솔레이트로 옮기세요. Move this work to a separate isolate

느린 기기에서 fetchPhotos() 함수를 실행하면 JSON을 파싱하고 변환하는 동안 앱이 잠시 정지되는 것을 알 수 있습니다. 이것은 쟁크이며, 이를 제거해야 합니다.

Flutter에서 제공하는 compute() 함수를 사용하여 파싱 및 변환을 백그라운드 아이솔레이트로 이동하여 jank를 제거할 수 있습니다. compute() 함수는 백그라운드 아이솔레이트에서 비용이 많이 드는 함수를 실행하고 결과를 반환합니다. 이 경우 백그라운드에서 parsePhotos() 함수를 실행합니다.

```dart
Future<List<Photo>> fetchPhotos(http.Client client) async {
  final response = await client
      .get(Uri.parse('https://jsonplaceholder.typicode.com/photos'));

  // compute 함수를 사용해 별도의 아이솔레이트에서 parsePhoto를 실행하세요.
  return compute(parsePhotos, response.body);
}
```

# 아이솔레이트를 사용한 작업에 대한 참고 사항 Notes on working with isolates


아이솔레이트는 메시지를 주고받으며 통신합니다. 이러한 메시지는 null, num, bool, double 같은 기본 값이거나 String 또는 이 예에서 List<Photo>같은 간단한 객체 일 수 있습니다.

아이솔레이트 간에 Future 또는 http.Response같은 더 복잡한 객체를 전달하려고 하면 오류가 발생할 수 있습니다.

또 다른 해결책으로, 백그라운드 처리를 위한 worker_manager또는 workmanager 패키지를 확인해보세요.

# 완전한 예

```dart
import 'dart:async';
import 'dart:convert';

import 'package:flutter/foundation.dart';
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

Future<List<Photo>> fetchPhotos(http.Client client) async {
  final response = await client
      .get(Uri.parse('https://jsonplaceholder.typicode.com/photos'));

  // compute 함수를 사용해 별도의 아이솔레이트에서 parsePhoto를 실행하세요.
  return compute(parsePhotos, response.body);
}

// 응답 본문을 List<Photo>으로 변환하는 함수.
List<Photo> parsePhotos(String responseBody) {
  final parsed =
      (jsonDecode(responseBody) as List).cast<Map<String, dynamic>>();

  return parsed.map<Photo>((json) => Photo.fromJson(json)).toList();
}

class Photo {
  final int albumId;
  final int id;
  final String title;
  final String url;
  final String thumbnailUrl;

  const Photo({
    required this.albumId,
    required this.id,
    required this.title,
    required this.url,
    required this.thumbnailUrl,
  });

  factory Photo.fromJson(Map<String, dynamic> json) {
    return Photo(
      albumId: json['albumId'] as int,
      id: json['id'] as int,
      title: json['title'] as String,
      url: json['url'] as String,
      thumbnailUrl: json['thumbnailUrl'] as String,
    );
  }
}

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    const appTitle = 'Isolate Demo';

    return const MaterialApp(
      title: appTitle,
      home: MyHomePage(title: appTitle),
    );
  }
}

class MyHomePage extends StatefulWidget {
  const MyHomePage({super.key, required this.title});

  final String title;

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  late Future<List<Photo>> futurePhotos;

  @override
  void initState() {
    super.initState();
    futurePhotos = fetchPhotos(http.Client());
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      body: FutureBuilder<List<Photo>>(
        future: futurePhotos,
        builder: (context, snapshot) {
          if (snapshot.hasError) {
            return const Center(
              child: Text('An error has occurred!'),
            );
          } else if (snapshot.hasData) {
            return PhotosList(photos: snapshot.data!);
          } else {
            return const Center(
              child: CircularProgressIndicator(),
            );
          }
        },
      ),
    );
  }
}

class PhotosList extends StatelessWidget {
  const PhotosList({super.key, required this.photos});

  final List<Photo> photos;

  @override
  Widget build(BuildContext context) {
    return GridView.builder(
      gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
        crossAxisCount: 2,
      ),
      itemCount: photos.length,
      itemBuilder: (context, index) {
        return Image.network(photos[index].thumbnailUrl);
      },
    );
  }
}
```