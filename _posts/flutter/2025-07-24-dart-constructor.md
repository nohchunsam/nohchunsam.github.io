---
title: 플러터 생성자
date: 2025-07-24 12:10:00 +0900
categories: Flutter
tags: ["Flutter", "공부"]
---


클래스 작성 중 아이 클래스의 생성자 작성에 막혔다.
이 참에 생성자에 대해서 알아놔야겠다.. 싶어서 쓱쓱 문서 번역. 일부만 번역했다.

https://dart.dev/language/constructors

# 생성자 종류

## 생성형 생성자 Generative constructors
- 객체를 만드는데 주로 사용
```
class Point {
  // Instance variables to hold the coordinates of the point.
  double x;
  double y;

  // Generative constructor with initializing formal parameters:
  Point(this.x, this.y);
}
```

## 기본 생성자 Default constructors
생성자를 선언하지 않을 경우 생성된다. 기본 생성자는 생성형 생성자에 이름이나 매개변수를 뺀 것과 같다.

## 명명된 생성자 Named constructors
클래스에 여러 생성자를 만드는 데 사용된다.
서브 클래스는 슈퍼 클래스의 명명된 생성자를 상속하지 않는다. 슈퍼 클래스에서 생성된 명명된 생성자를 서브 클래스에 만드려면, 서브클래스에 그 생성자를 구현해야 한다.
```
const double xOrigin = 0;
const double yOrigin = 0;

class Point {
  final double x;
  final double y;

  // x와 y 변수를 생성자가
  // 실행되기 전에 설정한다.
  Point(this.x, this.y);

  // 명명된 생성자
  Point.origin() : x = xOrigin, y = yOrigin;
}
```

## 상수 생성자 Constant constructors
변하지 않는 객체를 만들 경우에 사용된다. 컴파일할 때 생성된다. 상수 생성자의 모든 변수는 `final`이어야 한다.
```
class ImmutablePoint {
  static const ImmutablePoint origin = ImmutablePoint(0, 0);

  final double x, y;

  const ImmutablePoint(this.x, this.y);
}
```

## 재지시 생성자 Redirect constructors
같은 클래스에 있는 다른 생성자를 호출하는 생성자. 이 생성자는 `:` 뒤에 `this`를 사용해 호출한다.
```
class Point {
  double x, y;

  // The main constructor for this class.
  Point(this.x, this.y);

  // Delegates to the main constructor.
  Point.alongXAxis(double x) : this(x, 0);
}
```

## 팩토리 생성자 Factory constructors
다음과 같은 경우에 `factory` 키워드를 사용한다.
- 항상 그 클래스의 객체를 생성하는 것은 아니다.
- 객체를 생성하기 전에 간단한 작업을 수행해야 한다.

```
class Logger {
  final String name;
  bool mute = false;

  // _cache는 이름 앞에 _를 붙였기
  // 때문에 private이다.
  static final Map<String, Logger> _cache = <String, Logger>{};

  factory Logger(String name) {
    return _cache.putIfAbsent(name, () => Logger._internal(name));
  }

  factory Logger.fromJson(Map<String, Object> json) {
    return Logger(json['name'].toString());
  }

  Logger._internal(this.name);

  void log(String msg) {
    if (!mute) print(msg);
  }
}
```

# 형식 파라미터로 초기화하기
형식 파라미터 초기화 기능은 생성자 인수를 객체 변수에 할당을 단순화해준다. 대표적인 것이 `this`.

- 생성자 선언에 `this.<propertyname>`을 포함하고 생성자 바디를 생략한다. `this` 키워드는 현재 객체를 참조한다.
- 같은 이름으로 충돌이 생긴다면 `this`를 선언한다. 그렇지 않으면 Dart는 `this`를 생략한다. 생성형 생성자의 경우 예외가 있는데, 초기화 형식 파라미터 앞에 `this`를 붙여야 한다.
- 특정 생성자와 생성자의 특정 부분에는 `this`를 접근할 수 없다.
  - 팩토리 생성자
  - 초기화 목록의 오론쪽 
  - 슈퍼클래스 생성자에 대한 인수


- 형식 파라미터 초기화는 null이 아니거나 final 변수도 초기화하게 해준다.
- private 변수는 사용할 수 없다.
- 명명된 변수에서도 가능하다.