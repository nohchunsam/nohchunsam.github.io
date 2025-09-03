---
title: 위젯을 메서드로 나누지 마세요.
date: 2025-07-24 12:10:00 +0900
categories: Flutter
tags: ["Flutter", "공부"]
---

[이 글](https://blog.codemagic.io/how-to-improve-the-performance-of-your-flutter-app./)을 참고하면 다음과 같은 글이 있다.

## 위젯을 메서드로 나누지 마세요.

레이아웃이 클 때, 보통 메서드로 위젯을 나눈다. 아래는 예시다.

```dart
class MyHomePage extends StatelessWidget {
  Widget _buildHeaderWidget() {
    final size = 40.0;
    return Padding(
      padding: const EdgeInsets.all(8.0),
      child: CircleAvatar(
        backgroundColor: Colors.grey[700],
        child: FlutterLogo(
          size: size,
        ),
        radius: size,
      ),
    );
  }

  Widget _buildMainWidget(BuildContext context) {
    return Expanded(
      child: Container(
        color: Colors.grey[700],
        child: Center(
          child: Text(
            'Hello Flutter',
            style: Theme.of(context).textTheme.display1,
          ),
        ),
      ),
    );
  }

  Widget _buildFooterWidget() {
    return Padding(
      padding: const EdgeInsets.all(8.0),
      child: Text('This is the footer '),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Padding(
        padding: const EdgeInsets.all(15.0),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            _buildHeaderWidget(),
            _buildMainWidget(context),
            _buildFooterWidget(),
          ],
        ),
      ),
    );
  }
}
```

하지만 이건 권장하지 않는 패턴이다. 이 경우 위젯 전체를 변경하고 새로 고칠 때, 메서드 내에 있는 위젯들도 함꼐 고쳐 CPU 사이클을 낭비하게 된다.

그래서 권장하는 방법은 이렇다.

```dart
class MyHomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Padding(
        padding: const EdgeInsets.all(15.0),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            HeaderWidget(),
            MainWidget(),
            FooterWidget(),
          ],
        ),
      ),
    );
  }
}

class HeaderWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final size = 40.0;
    return Padding(
      padding: const EdgeInsets.all(8.0),
      child: CircleAvatar(
        backgroundColor: Colors.grey[700],
        child: FlutterLogo(
          size: size,
        ),
        radius: size,
      ),
    );
  }
}

class MainWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Expanded(
      child: Container(
        color: Colors.grey[700],
        child: Center(
          child: Text(
            'Hello Flutter',
            style: Theme.of(context).textTheme.display1,
          ),
        ),
      ),
    );
  }
}

class FooterWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.all(8.0),
      child: Text('This is the footer '),
    );
  }
}
```

위젯들을 StatelessWdiget으로 감쌌다. setState가 필요하면 StatefulWidget으로 감싸도 무관하다.

아무튼 핵심은 이렇다.

> Stateful/Stateless Widget은 (키, 위젯 종류, 속성에 기반한) 특별한 캐시 메커니즘이 있습니다. 이를 통해 필요할 때만 재빌딩을 할 수 있습니다. 더불어 위젯을 캡슐화하고 리팩토링하는데 도움이 됩니다.

## 공식 문서 디비보기 - Stateless

이게 무슨 소리지? [공식 문서의 StatelessWidget 글](https://api.flutter.dev/flutter/widgets/StatelessWidget-class.html)에는 이런 내용이 있다.

> StatelessWidget을 StatefulWidget으로 리팩토링하는 것을 고려하십시오. 이를 통해 StatefulWidget에서 설명된 기법들(예: 하위 트리의 공통 부분 캐싱, 트리 구조 변경 시 GlobalKeys 사용)을 활용할 수 있습니다.

> 여기서 **트리**는 **위젯들의 계층적 구조**다. 트리를 통해 사요자들이 보는 UI를 그린다. 키는 Flutter가 각 위젯을 식별하는데 사용되는 식별자다. 동적 위젯(리스트, 그리드) 등에서 중요한 역할을 한다. 자세한 건 [이 글](https://changjoo-park.github.io/learn-flutter/part3/widget-tree/)을 참고해보자.

## 공식 문서 디비보기 - Stateful

우선 StatefulWidget에서 할 수 있는 게 있는 것 같다. [공식 문서의 StatefulWidget 글](https://api.flutter.dev/flutter/widgets/StatefulWidget-class.html)을 봤다.

> 서브트리가 변경되지 않으면 해당 서브트리를 나타내는 위젯을 캐싱하고, 가능한 경우 매번 재사용하세요. 이를 위해 위젯에 final 상태 변수를 할당한 후 build 메서드에서 재사용합니다. 새 위젯(동일한 구성)을 생성하는 것보다 위젯을 재사용하는 것이 더 효율적입니다. 또 다른 캐싱 전략은 위젯의 변경 가능한 부분을 자식 파라미터를 받는 StatefulWidget으로 추출하는 것입니다.
가능한 경우 const 위젯을 사용하십시오. (위젯을 캐싱하고 재사용하는 것과 동일합니다.)
