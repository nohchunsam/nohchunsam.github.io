---
title: Spring 기본 개념
date: 2023-07-12 00:00:00 +0900
categories: Spring
tags: ["Spring", "공부"]
---

해당 글은 2023년 7월 12일 벨로그에 쓴 글(현재 비공개)을 2025년 9월 15일에 정리하고 내용을 추가한 것이다.

----

현재 스프링을 통해 서버를 개발해야한다. (열심히) 클론 코딩하던게 버전이 터져서 막힌 김에 개념부터 다시 짚고 넘어가려 한다... 이 글은 스프링 프레임워크 첫걸음(주. 후루네스 키노시타 마사아키 저), 스프링 공식문서를 참고해서 작성했다.

## 스프링 프레임워크

자바 플랫폼용 애플리케이션 프레임워크다.

> 자바 플랫폼: 자바 애플릿(작은 애플리케이션), 애플리케이션을 개발하고 관리하기 위한 환경. 자바, 자바 패키지, 자바 가상 머신으로 구성된다.

> 프레임워크: 새로운 애플리케이션을 보다 효율적으로 개발할 수 있도록 하는 재사용 가능한 소프트웨어 구성 요소의 모음이다.

> 프레임워크를 흔히 **뼈대**로 비유한다. 프레임워크는 재사용 가능한 기초적인 규격을 재공한다. 이를 기반으로 개발자들은 원하는 서비스를 빠르게 개발할 수 있다.

요약하자면 자바 애플리케이션을 만드는 데 사용되는 개발 구성 요소다.

## 스프링 부트

스프링 프레임워크를 기반으로 구축된 프레임워크다. 스프링 프레임워크 애플리케이션를 복잡한 설정, 버전 제어 없이 **빠르게** 작성하는 것이 주 목적이다.

공식 문서에 적힌 특징은 다음과 같다.

- 독립 실행형 스프링 애플리케이션 생성
- Tomcat, Jetty, Undertow가 내장 (WAR 파일 배포 불필요)
- 빌드 구성을 간소화하는 'starter' dependency 제공
- 가능할 때마다 Spring 및 서드 파티 라이브러리를 자동으로 구성
- 메트릭, 헬스 체크, 외부 구성과 같은 실제 배포 환경에 바로 적용 가능한 기능을 제공
- 코드 생성 불필요, XML 구성 불필요

### 스프링 부트 - Starters

스프링 프레임워크는 dependency... 라이브러리 버전을 관리한다고 많은 시간을 들였는데, starters를 통해 주로 사용하는 라이브러리를 묶어 버전 관리를 단순화시켰다. 

예를 들어 `Spring Data JPA`를 사용하고 싶으면 `spring-boot-starter-data-jpa`를 추가하면 된다. 이러면 jpa에 자주 사용되는 모든 라이브러리가 한번에 추가된다.

## 스프링 핵심 특징

### 의존성 주입 Dependency Injection

**하나의 코드가 다른 코드에 의존하는 상태**

```
class A {
    new b = new B();
    /* 생략 */
}

class B {
    /* 생략 */
}
```

클래스 A가 클래스 B를 사용하고 있다. 클래스 A는 클래스 B에 의존적이다. 

의존성 주입의 경우, 클래스 A가 클래스 B의 인스턴스를 직접 사용하는 것이 아니라, 외부에서 B 인스턴스를 생성해 A로 주입해준다.

**스프링 프레임워크**에서는 이 상태가 `Bean`을 통해 이루어진다.

> Bean: 스프링 컨테이너에 의해 관리되는 모듈, 컴포넌트, 객체

```
class A {
    @Autowired
    private B b; // B 타입의 Bean을 주입
}

@Component // Bean으로 등록
class B {
    /* 생략 */
}
```

`@Component` 데커레이터를 통해 클래스 B를 스프링 컨테이너에 등록하고, 스프링 컨테이너에서 클래스 A의 멤버 변수에 등록된 B를 주입해준다.

이러면 의존성이 감소해서 유연성이 상승하고, 모듈을 직접할 필요가 없기 때문에 코드양이 감소한다.

### 제어 역전 Inversion of Control

**의존성을 직접 제어하던 것이 역전돼, 제어를 하지 않게 되는 것**

위의 예를 들고 이야기하면, 클래스 A가 매개체를 두고 클래스 B를 사용하게 돼, 제어 방향이 역전된다.

**스프링 프레임워크**에서는 xml 파일이나 Java Configuration 파일에서 의존 관계, `Bean`을 설정하면, 스프링 컨테이너가 등록된 `Bean`들의 생명주기를 전부 관리해준다.

### 관점 지향 프로그래밍 Aspect Oriented Programming

- 중심적 관심사: 본질적 기능을 나타내는 프로그램
- 횡단적 관심사: 본질적 기능 X. 품질 및 유지보수에 필요 O

⇒ 중심적 관심사와 횡단적 관심사를 분리시키고 각각을 모듈화한 프로그래밍

핵심 로직을 깔끔하게 유지하고, 가독성 유지보수성을 높이는 방법 중 하나다.

**스프링 프레임워크**에서 AOP는 별도의 라이브러리를 추가해 사용할 수 있다. `@Aspect` 데커레이터를 통해 횡단적 관심사 모듈임을 나타내고, 중심적 관심사에 적용한다.

### Portable Service Abstraction

환경의 변화와 관계없이 일관된 방식의 기술로 접근 환경을 제공하는 추상화 구조를 의미한다. = **내 코드를 바꾸지 않고 다른 기술로 바꿀 수 있는 추상화 구조를 의미한다.**

예를 들어 스프링 웹 MVC는 `Servlet`을 통해 코딩할 수 있고, `Reactive`를 통해 코딩할 수도 있다. 또한 서버를 Tomcat, Jetty, Undertow로 바꿔 사용할 수 있다! 


## 출처
- https://www.ibm.com/docs/en/i/7.4.0?topic=java-platform
- https://aws.amazon.com/what-is/framework/#:~:text=A%20framework%20provides%20a%20flexible,libraries%2C%20debuggers%2C%20and%20compilers
- https://velog.io/@courage331/Spring-%EA%B3%BC-Spring-Boot-%EC%B0%A8%EC%9D%B4
- https://spring.io/projects/spring-framework
- https://spring.io/projects/spring-boot
- https://www.geeksforgeeks.org/java/difference-between-spring-and-spring-boot/
- https://www.geeksforgeeks.org/springboot/spring-boot-starters/
- https://youtu.be/1vdeIL2iCcM?si=xK84B6J_YwunYt_R
- https://velog.io/@damiano1027/Spring-%EC%9D%98%EC%A1%B4%EC%84%B1-%EC%A3%BC%EC%9E%85-%EC%A0%9C%EC%96%B4%EC%9D%98-%EC%97%AD%EC%A0%84
- https://mangkyu.tistory.com/121
- https://engkimbs.tistory.com/entry/%EC%8A%A4%ED%94%84%EB%A7%81AOP
- https://ittrue.tistory.com/214