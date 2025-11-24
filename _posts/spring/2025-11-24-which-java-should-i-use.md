---
title: 어느 자바를 사용할까요
date: 2025-11-24 19:46:00 +0900
categories: Spring
tags: ["Spring", "Java", "공부"]
---


오랜만에 spring initializr를 키니까 못보던 java 버전이 훅 늘었다.

17이 맨 앞에 있던게 엊그제 같은데 17이 맨 뒤에 있고 25 21이 자리를 차지하고 있다.

당시 프로젝트를 할 때는 가장 최신 버전인 17을 사용했는데, 지금 프로젝트에서는 어느 걸 사용하는 게 제일 좋을지 좀 찾아보기로 했다.

## LTS가 뭐야

근데 찾아보니 다들 LTS LTS한다. 

이것도 찾아보니 Long Term Support = 장기 지원이라는 뜻이다. 그러니까 18 19 보다는 LTS인 21을 더 오래 지원한다는 이야기다.

더 오래 지원하면 뭐가 좋나? 일단 오래 배포할거라 회사에서도 일반 버전보다 더 안정적이게 내려고 하고 기간동안, 버그 수정도 해준다. 게다가 버전 수정도 크게 안해도 된다!

이런 지원을 받는게 17, 21, 25다.

## Java

버전이 이렇게 3개가 있으니, 최소한 이 세 개가 어떻게 다른지 알아보자.

## 17

https://www.youtube.com/watch?v=m2ak1zI-M8g

- Switch: switch문이 java에 들어왔다! 
- Sealed Class: 상속하거나 확장될 클래스를 제어한다.
- Vector API: 벡터 계산 API를 추가했다.
- Foreign Function and Memory API: 자바 런타임 외부에서 데이터와 코드와 상호작용할 수 있다. 이를 통해 메모리 관리를 더 쉽게 할 수 있다.

## 21

https://youtu.be/GYKgVwc_MvQ?si=e0w9cKibLHUdVwY3

- Sequenced Collections: collection에서 첫번째랑 마지막 항목을 찾거나 제거할 수 있다.
- Z Garbage Collector: Z Garbage Collector에 세대별(객체 생성 시간에 따른) 컬렉터를 추가했다.
- Record Pattern: Record 클래스에서 데이터를 더 쉽게 가지고 올 수 있도록 했다.
- Virtual Thread: 기존보다 더 가벼운 스레드! 동시성과 확장성을 높이기 위해 설계됐다.


## 25

https://youtu.be/lCNNA1erCfk?si=5nTgnX-zDsphl6Tx

- Sealed Type: Sealed type을 통해 switch 구문으로 견고함과 확장성을 유지하고, 코드이 가독성을 향상시킨다.
- Scoped Values: 특정 범위에서만 접근 가능한 변경 불가 값이다. context 정보 제공에 유용하다.
- Module import: 한 줄로 모듈의 모든 API를 가져오는 것이 가능해졌다
- Flexible constructor: super를 호출하기 전에 로직을 추가하는 것이 가능하다

...

## Spring Boot

하지만 프레임워크도 생각해야한다.

Spring boot는 3.2.x 부터 21을 지원하고, 2025년 11월 20일에 출시한 4.0.0부터 (Spring framework 7 기반) 25를 지원한다. 물론 두 버전 다 17이랑 호완도 가능하다.

다만... 4.0.0이 나온지 정말 얼마 안되고, 자료도 좀처럼 찾을 수 없어 나는 그전의 안정적인 버전인 3.5.8을 사용하려고 한다.

## 결론

그러면 17 vs 21 이다. 큰 API를 사용할 것 같진 않지만... 가상 스레드를 통해 좀 더 안정성을 확보하고 싶어 21을 사용하기로 했다. Spring boot는 5.7 버전을 사용하기로 했다.


## 출처
- https://blog.igooo.org/185
- https://velog.io/@kkd0059/Java-8-vs-Java-11-vs-Java-17-vs-Java-21
- https://kghworks.tistory.com/197