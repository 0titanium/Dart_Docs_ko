# 믹스인 Mixins

믹스인은 여러 클래스 계층에서 재사용할 수 있는 코드를 정의하는 방법입니다. 이는 멤버 구현을 한꺼번에 제공하기 위한 것입니다.

믹스인을 사용하려면 with 키워드 뒤에 하나 이상의 믹스인 이름을 사용하십시오. 다음 예제에서는 믹스인(또는 믹스인의 하위 클래스인)을 사용하는 두 클래스를 보여줍니다.

```dart
class Musician extends Performer with Musical {
  // ···
}

class Maestro extends Person with Musical, Aggressive, Demented {
  Maestro(String maestroName) {
    name = maestroName;
    canConduct = true;
  }
}
```

믹스인을 정의하려면 mixin선언을 사용하세요. 믹스인과 클래스를 모두 정의해야 하는 드문 경우에는 믹스인 클래스 선언을 사용할 수 있습니다 .

믹스인과 믹스인 클래스는 extends절을 가질 수 없으며 생성형 생성자를 선언해서는 안 됩니다.

예를 들어:

```dart
mixin Musical {
  bool canPlayPiano = false;
  bool canCompose = false;
  bool canConduct = false;

  void entertainMe() {
    if (canPlayPiano) {
      print('Playing piano');
    } else if (canConduct) {
      print('Waving hands');
    } else {
      print('Humming to self');
    }
  }
}
```

## 믹스인이 자체적으로 호출할 수 있는 멤버 지정 Specify members a mixin can call on itself

때로 믹스인은 메서드 호출이나 필드 접근 가능 여부에 따라 달라지지만, 해당 멤버 자체를 정의할 수는 없습니다(믹스인은 생성자 매개 변수를 사용하여 자신의 필드를 인스턴스화할 수 없기 때문입니다).

다음 섹션에서는 믹스인의 하위 클래스가 믹스인의 동작이 의존하는 멤버를 정의하는 것을 보장하기 위한 다른 전략들을 다룹니다.

## 믹스인에서 추상 멤버 정의 Define abstract members in the mixin

믹스인에서 추상 메서드를 선언하면 믹스인을 사용하는 모든 타입이 해당 동작에 의존하는 추상 메서드를 정의하도록 강제됩니다.

```dart
mixin Musician {
  void playInstrument(String instrumentName); // 추상 메서드.

  void playPiano() {
    playInstrument('Piano');
  }
  void playFlute() {
    playInstrument('Flute');
  }
}

class Virtuoso with Musician { 
  void playInstrument(String instrumentName) { // 하위 클래스가 반드시 정의.
    print('Plays the $instrumentName beautifully');
  }  
}
```

## 믹스인 하위 클래스의 접근 상태 Access state in the mixin's subclass

추상 멤버를 선언하면 믹스인에서 추상으로 정의된 getter를 호출하여 믹스인의 하위 클래스에 대한 상태에 접근할 수도 있습니다.

```dart
/// [name] 속성이 있는 모든 유형에 적용할 수 있으며 이에 따라
/// [hashCode] 및 `==` 연산자의 구현을 제공합니다.
mixin NameIdentity {
  String get name;

  int get hashCode => name.hashCode;
  bool operator ==(other) => other is NameIdentity && name == other.name;
}

class Person with NameIdentity {
  final String name;

  Person(this.name);
}
```

## 인터페이스 구현 Implement an interface

mixin abstract를 선언하는 것과 유사하게, 실제로 인터페이스를 구현하지 않고 mixin에 implements 절을 추가하면 mixin에 정의된 모든 멤버 종속성을 보장합니다.

```dart
abstract interface class Tuner {
  void tuneInstrument();
}

mixin Guitarist implements Tuner {
  void playSong() {
    tuneInstrument();

    print('Strums guitar majestically.');
  }
}

class PunkRocker with Guitarist {
  void tuneInstrument() {
    print("Don't bother, being out of tune is punk rock.");
  }
}
```

## on 절을 사용하여 상위 클래스 선언 Use the on clause to declare a superclass

on 절은 super 호출이 해결하는 타입을 정의하기 위해 존재합니다. 따라서 믹스인 내부에서 super 호출이 필요한 경우에만 사용해야 합니다.

on 절은 믹스인을 사용하는 모든 클래스가 on 절 안에 있는 타입의 하위 클래스가 되도록 강제합니다. 믹스인이 상위 클래스 안에 있는 멤버에 의존한다면 믹스인이 사용되는 곳에서 멤버들이 사용 가능하다는 것을 보장합니다.

```dart
class Musician {
  musicianMethod() {
    print('Playing music!');
  }
}

mixin MusicalPerformer on Musician {
  perfomerMethod() {
    print('Performing music!');
    super.musicianMethod();
  }
}

class SingerDancer extends Musician with MusicalPerformer { }

main() {
  SingerDance().performerMethod();
}
```

위의 예시에서 Musician 클래스를 확장 또는 구현한 클래스들만 믹스인 MusicalPerformer를 사용할 수 있습니다. SingerDnacer는 Musician을 확장하므로 SingerDancer는 MusicalPerformer를 믹스인할 수 있습니다.

## Class, mixin, 또는 mixin class? class, mixin, or mixin class?

```
버전 노트

mixin class 선언은 최소한 3.0 이상의 언어 버전을 요구합니다.
```

mixin선언은 mixin을 정의합니다. class 선언은 class를 정의합니다. mixin class 선언은 같은 이름과 같은 타입을 사용하는 일반 클래스와 믹스인 모두로 사용할 수 있는 클래스를 정의합니다.

```dart
mixin class Musician {
  // ...
}

class Novice with Musician { // Musician을 mixin으로 사용
  // ...
}

class Novice extends Musician { // Musician을 클래스로 사용
  // ...
}
```

클래스나 mixin에 적용되는 모든 제한은 mixin 클래스에 똑같이 적용됩니다.

- mixin은 extends나 with 절을 가질 수 없으며 mixin class 또한 마찬가지입니다.

- 클래스는 on 절을 가질 수 없으며 mixin class 또한 마찬가지입니다.