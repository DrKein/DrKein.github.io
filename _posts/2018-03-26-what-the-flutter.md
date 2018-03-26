---
layout: post
title: What the F**tter!? Understanding Flutter as an Android(Java) Developer(번역)
description: >
  Flutter에 대해서 알아봅시다.
categories: [Kotlin]
tags: [Kotlin, Android, Flutter]
comments: true
---

원문 : <https://proandroiddev.com/what-the-f-tter-understanding-flutter-as-an-android-java-developer-2158086a2bd9>

![](https://cdn-images-1.medium.com/max/1600/1*2yFbiGdcACiuLGo4dMKmJw.jpeg)

# What the F**tter!? Understanding Flutter as an Android(Java) Developer

당신이 바위 아래 살고 있는게 아니라면, 아마도 당신은 Flutter와 관련 이야기들을 들었을 것입니다.

일반적인 개발자들의 반응은 혼합된 것이고 그들을 비난하는 것은 아니지만, 우리는 이제 막 사랑에 빠지지 않을 수 없는 언어(❥otlin)를 찾은것은 꽤나 분명해 보이고, 뜻밖에도 큰 "G"는 안드로이드(그리고 iOS)앱 개발을 위한 또다른 프레임워크(와 언어)를 제공합니다.

예, 제말을 제대로 들으셨습니다. Flutter를 쓰려면 앱을 개발하는데 Java나 Kotlin을 사용할 수 없고, 대신에 구글의 다른 언어인 Dart(예전에 저는 이것이 뭔지 몰랐습니다)를 배워야만 합니다.

그래서 호기심과 JS를 작성해 보지 않은 사람 (진정한 개발자는 어셈블리를 사용하죠!) 으로서, 저는 Flutter와 Dart를 사용하여 이 새로운 도구를 실제로 사용해보고 얼마나 좋은지 나쁜지를 살펴보기로 마음 먹었습니다.

여기 저의 생상한 여정과 그 과정에서 해결했던 미스테리들이 있습니다. 꼭 잡으세요!

> ***주의*** : *어느 언어가 더 좋다는 논쟁을 하려는게 아닙니다. 이 블로그는 단지 안드로이드 개발자 입장에서 Flutter를 이용하여 앱을 개발하는 구조에 대해 이해한 내용을 살펴보는 것 뿐입니다.*


## Episode 1: 시작
이것은 꽤나 수월했습니다. 저는 그냥 공식 flutter 웹사이트에 가서 여기(<https://flutter.io/get-started/install/>) 나열된 설명서를 따라 했습니다. 

**주의**: 만약 iOS앱을 만드는게 아니라면 **libimobiledevice, ideviceinstaller, ios-deploy** 그리고 **CocoaPods** 는 사실 필요 없습니다. 그러니 **flutter 박사** 의 충고를 무시하시고, 이것들을 설치하는데 필요한 엄청나게 많은 시간을 아끼세요.

다음으로 안드로이드 스튜디오에 Flutter플러그인을 설치하고 새로운 Flutter 프로젝트를 생성합니다.

## Episode 2: 코드로 손 더럽히기

이제 설치가 끝나면, Dart의 신비로운 세상에 노출됩니다.

이것은 처음 실행시키면 나타나는 앱의 모습니다 :

![Pretty simple, right?](https://cdn-images-1.medium.com/max/1600/1*C2q075E8hAAkCujGYswGDw.png)

플로팅 액션 버튼을 누르면 0이라고 적힌 숫자를 증가시킵니다.


재미있는 부분으로 가봅시다.  
코드를 살펴 보면, 메인 파일 이름이 **Main.dart**가 아니라 **main.dart** 인 것을 봤을 때 나의 강박증이 날뛰기 시작 했지만, 무시하고 넘어가기로 합니다.

```dart
import 'package:flutter/material.dart';

void main() => runApp(new MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new MaterialApp(
      title: 'Flutter Demo',
      theme: new ThemeData(  
        primarySwatch: Colors.blue,
      ),
      home: new MyHomePage(title: 'Flutter Demo Home Page'),
    );
  }
}

......
```

Java와 Kotlin에서 봤던 몇 몇 친숙한 자취들을(**classes, Objects**, 다양한 루트 클래스들 등.) Dart에서 볼 수 있었습니다. 하지만 제 눈을 사로 잡은것은 
```
void main() => runApp(new MyApp());
```
아주 오래된 C++ 시절이 떠오르게 하는 맨 위의 메서드 였습니다.

이것은 우리 어플리케이션의 시작점으로 밝혀졌습니다.

이 메서드가 하는 일은 setContentView()와 특정 위젯을 화면에 표시하는 역할을 하는 것으로 보이는 runApp() 메서드를 호출 하는 것이 전부 입니다.

아래에 우리의 실제 MyApp 클래스가 있습니다.  
MyApp은 여기에선 다양한 위젯을 가진 어플리케이션 클래스라고 생각 됩니다.   
MyApp은 **StatelessWidget**(Flutter에서는 거의 모든것이 위젯 입니다.)를 확장 합니다. 즉, 방금 더 많은 헛소리가 당신 얼굴에 뿌려졌습니다.

저의 첫 본능은 스택 오버플로우를 열고 이것들이 무엇을 의미하는지 찾는 거였고, 저를 문서들로 인도 하였고 그 것을 아래 인용 하였습니다 :
> ***State*****less** 위젯은 ***불변입니다*** 속성이 변경될 수 없고 값들은 모두 final 입니다.
> 
> ***State*****ful** 위젯은 위젯이 살아있는 동안 ***변경**가능한 상태를 유지합니다.

음, 이해 되네요, Java의 final, 또는 Kotlin에서의 val 과 var와 비슷 합니다.

MyApp내부를 보면, 특정 위젯을 어떻게 표시할 것인지를 책임지는 **build()** 메서드를 오버라이드 하고 있습니다.

우리의 어플리케이션 클래스와 비슷하게, 여기에서 우리의 어플리케이션이 Material 스타일로 만들어 지도록 MaterialApp 클래스를 생성하고 리턴 합니다.

**우리 앱의 라벨**에 영향을 주는 **타이틀**이나 **액션바 모양**에 적용되는 **테마**처럼 낯익은 이름들을 보실 수 있습니다.

조금 더 보면, MyApp위젯에 보여지는 **MyHomePage**를 설정하는 새로운 **home 변수** 가 있습니다.  
이 MyHomePage 위젯은 여기서 **액티비티**라고 생각 됩니다.

**MyHomePage**는 이 액티비티의 툴바 타이틀에 적용 되는 걸로 보이는 문자열을 받습니다.

**여기에 액티비티들이 없다는 점을 주목하세요. 오직 위젯뿐입니다!**

MyApp 클래스 아래에 MyHomePage 클래스가 있습니다 :

```dart
class MyHomePage extends StatefulWidget {
  MyHomePage({Key key, this.title}) : super(key: key);
  final String title;

  @override
  _MyHomePageState createState() => new _MyHomePageState();
}
....
```
**MyHomePage**가 **StatefulWidget**이라는 것은, 그것은 어떻게 보이는지를 저장하는 **State** 오브젝트를 가진다는 것을 의미 합니다.   
특히 이 클래스의 역할은 부모(이 경우 App 위젯)이 제공한 값 (이 경우 타이틀)을 유지해서, 우리가 나중에 구현하는 **State** 위젯에서 사용할 수 있도록 하는 것입니다.
(정말 쉬운 일을하고 있습니다. 그렇죠? :풍자:)

어쨋든 계속 진행하면, **createState()** 메서드에서 이 위젯의 UI클래스를 생성하고 정의하는 것처럼 보입니다.

> P.S. Dart가 Kotlin의 표현 함수와 유사한 한 줄 함수 (=>) 를 가지고 있는것을 주의 하세요.

우리 앱에서 실제 UI를 구현하는 클래스를 살펴 봅시다 :

```dart
class _MyHomePageState extends State<MyHomePage> {
  int _counter = 0;

  void _incrementCounter() {
    setState(() {
      _counter++;
    });
  }

  @override
  Widget build(BuildContext context) {
    ....
  }
}
```

**_MyHomePageState** 클래스가 **State**라 불리는 제너릭 클래스를 상속합니다. 

이 State 클래스는 로직과 StatefulWidget(여기서는 **MyHomePage**)을 위한 내부 상태를 포함합니다.

우리의 앱은 카운터이니, FAB를 클릭하면 화면의 카운트를 증가시킵니다.  
우리는 이 카운트를 저장할 변수와 이 변수값을 증가시킬 메서드가 필요합니다.

우리는 이 두가지를 클래스 앞부분 두개의 멤버로 구현된 것을 알 수 있습니다.

주의깊게 살펴보면 **_incrementCount()** 메서드 안에 **setState()** 메서드를 보실 수 있습니다.  
**setState()** 의 기능은 프레임워크에게 이 오브젝트의 내부 상태가 변경 되었다고 알리고, 위젯을 갱신시키는 일입니다. 

**build()** 메서드를 살펴보면, 내 위젯이 어떻게 보여질 것인지를 결정하는 메서드 입니다.

```dart
class _MyHomePageState extends State<MyHomePage> {
  
  ....
  
  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      appBar: new AppBar(
        title: new Text(widget.title),
      ),
      body: new Center(
        child: new Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            new Text(
              'You have pushed the button this many times:',
            ),
            new Text(
              '$_counter',
              style: Theme.of(context).textTheme.display1,
            ),
          ],
        ),
      ),
      floatingActionButton: new FloatingActionButton(
        onPressed: _incrementCounter,
        tooltip: 'Increment',
        child: new Icon(Icons.add),
      ), 
    );
  }
}
```

겁먹지 말고, 커피 한잔 하세요.  
친구, 이것은 긴 여정이 될거야!

이 메서드는 한가지 일을 합니다. 즉, 새로운 **Scaffold** 객체를 리턴 합니다.  
(또다른 특수 용어!)

이전 프로세스로 돌아가서, Scaffold에 대해 아래처럼 설명된 flutter웹 사이트에 도착 합니다 : 
> Material 라이브러리의 Scaffold 위젯은 기본 앱바, 타이틀 그리고 홈 화면 위젯 트리를 포함하는 본문 속성을 제공 합니다. 위젯 하위 트리는 상당히 복잡할 수 있습니다.

좋아, 알겠다!  
따라서 새로운 앱바를 추가하고 **MyHomePage** 안에 저장된 텍스트를 제목으로 설정 합니다.  
부모로 부터 전달받은 값들을 위한 자리라고 했던게 기억 나나요? 네, 그게 우리가 여기서 한 일입니다. 이 State의 부모에 대해 접근할 수 있고 제목을 거기서 얻었습니다.

다음, 우리는 **body**를 **Center**로 설정합니다.  
여기에서 **Center**는 기본적으로 우리가 안드로이드 개발에서 xml 레이아웃 작업 부분 입니다.  
가운데 정렬 레이아웃과 유사하게, 하나의 하위 요소를 가져와서 부모의 중간에 위치 시킵니다.

 하위 요소를 **세로로** (가로 폭은 match_parent) 정렬하는 **Column**이라는 하위 요소가 있습니다. 그리고, 당신이 추측한 것처럼 하위 요소들을 **가로로** 정렬하는 **Row**라는 이름의 다른 클래스가 있습니다.

**LinearLayout**의 세로와 가로와 다소 유사합니다.

**Column**클래스에게 가중치(gravity)를 알려주는 일을 하는 **mainAxisAlignment**키가 있습니다.  
시작 코드에 Center로 설정되어 있고 내용들이 중간에서 부터 시작 되었습니다.

마지막으로 이 컬럼 블럭안에 들어갈 내용을 정의하는 **위젯**의 **배열**이 있습니다. 

그리고 당신의 추측대로, 2개의 Text (우리가 사랑하는 TextViews)가 FAB 를 누른 횟수를 보여 줍니다.

FAB에 대해 말하자면, **Center**와 **Appbar** 블럭 외에 세번째 블럭을 가지고 있습니다.

그것은 **onPressed**메서드(**onClick**이 떠오르지 않나요?)를 가지고 있고, **툴팁**속성(**contentDescription**을 참조하는) 그리고 FAB의 **src/srcCompat** 을 모사하는 **Icon** 을 가지고 있습니다.  

![](https://cdn-images-1.medium.com/max/1600/1*5m7k64x1DJ2n0VeCVQAndw.jpeg)

전체 UI 구조는 위 모양과 같습니다.

XML보다 적은 코드 라인을 사용하지만, 사실 이 플랫폼을 처음 접하는 사람들에게는 어려울 수 있습니다.

## Episode 3: 결론 
Flutter가 멋져 보이고 핫 리로드가 실제로 빠르기는 하지만, 바꿀만한 가치가 있을까요?

제 의견은, 지금은 flutter로 완전히 갈아타서 앱을 만들기 시작할 시점은 아닙니다.  
Flutter는 여전히 베타 수준이고, StackOverflow나 다른 곳에서도 참조할 만한것들이 별로 없습니다.

가까운 미래에 유망한 플랫폼이 될 수는 있지만, 유일한 단점은 프레임워크에 Dart대신 Kotlin이 사용되는 기회를 놓친것 입니다.

![Match made in heaven?](https://cdn-images-1.medium.com/max/1600/1*2XTH6sgV_yClUCGNxGQO7w.jpeg)

여러분이 이제 우리가 알고 있고 사랑해 마지않는 전통적인 안드로이드 프레임워크와 Flutter 프레임워크가 서로 연동되는 것에 대해 높은 수준으로 이해하기를 바랍니다. 

저는 앞으로 몇주동안 Flutter에 대해서 탐험할 것이고, 좀 더 많은 블로그 글이 나오기를 기대하세요.

읽어주셔서 감사합니다. 이 글이 재미가 있었다면 👏 버튼을 클릭하고 다른 사람들이 볼 수 있도록 공유해주세요. 아래에 자유롭게 코멘트 💬 를 남겨주세요.  
피드백이 있나요? [트위터](https://twitter.com/daggerdwivedi) 친구가 되어 주세요.


