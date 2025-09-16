---
title: Spring boot 시작시 기본 구성 요소
date: 2025-09-16 13:10:00 +0900
categories: Spring
tags: ["Spring", "공부"]
---

해당 글은 2023년 7월 12일 벨로그에 [쓴 글](https://velog.io/@jungenoh3/Spring-boot)을 2025년 9월 16일에 정리하고 내용을 추가한 것이다.

----

이 글에서는 스프링 부트를 구성하는 것들을 짧게 훑고 간다. 대부분 스프링 공식문서를 참고한다.

## Maven & Gradle

둘 다 빌드/라이브러리 관리 도구다. 라이브러리를 다운로드해 컴파일, 테스트, 패키징해 배포까지 자동으로 처리해주는 도구다.

- Maven: 자바용 프로젝트 관리도구다. 프로젝트, 빌드 순서, 라이브러리를 pom.xml 파일에 명시한다.
  - 소스 파일의 위치나 컴파일 결과를 저장할 위치를 미리 정해두고, 그 위치에 소스파일이 있으면 그것을 근거로 빌드를 진행한다.
  - 프로젝트에 필요한 설정 정보를 '프로젝트 객체'라는 개념으로 모델링한다.
- Gradle: 안드로이드에서 개발한 관리도구다. 각종 빌드를 build.gradle 파일에 명시한다.
  - 빌드 과정에서 병렬 처리와 캐싱을 활용해 Maven보다 더 나은 성을 구현할 수 있다.

여기서는 Gradle을 위주로 설명한다.

## build.gradle

주 설정을 진행하는 파일이다. 각 항목이 무엇을 하는지는 주석으로 적어놨다!

```
// 종속성 관리, 실행 가능한 jar 파일 생성/실행 등.
// 빌딩 시 플러그인을 기반해 .jar 파일을 구성한다.
plugins {
	id 'java'
	id 'org.springframework.boot' version '3.1.1'
	id 'io.spring.dependency-management' version '1.1.0'
}

group = 'com.spring-1' // 본 프로젝트 그룹명
version = '0.0.1-SNAPSHOT' // 본 프로젝트 출시 버전

// 컴파일할 때 사용하는 자바 버전
java {
	sourceCompatibility = '17'
}

// 종속성을 어떤 원격 저장소에서 받을지 정한다.
// jcenter()도 있다.
repositories {
	mavenCentral()
}

// 종속성 = 라이브러리
dependencies {
	implementation 'org.springframework.boot:spring-boot-starter'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

// Junit: 자바 프로그램용 단위 테스트 프레임워크
// Junit 기반 플랫폼으로 테스트를 실행한다
tasks.named('test') {
	useJUnitPlatform()
}
```

## main.java

```java
package com.spring1.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class MyApplication {

    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }

}
```
- `@SpringBootApplication`: 다양한 기능을 하는 어노테이션을 하나로 합친 것이다. MyApplication을 빈으로 생성해 컨테이너에 등록하고, `@Component` 어노테이션을 명시한 클래스를 스캔해 자동으로 빈으로 만들고 등록한다. (수동으로도 가능한 일!)

[더 자세한 설명은 여기](https://any-ting.tistory.com/143)


## +) 서블릿

동적 페이지를 만드는 데 사용되는 웹서버 프로그램

> 동적 페이지: 사용자가 보내는 요청, 시간, 상황에 따라 달라지는 결과, 웹페이지

**http 형태의 요청을 수신하고, 그 응답을 파싱해준다. 개발자는 중요한 로직에만 집중할 수 있다.**

```java
public class MyServlet extends HttpServlet {

    // Get 요청이 들어오면, Response에 Hello World를 담아 반환한다.
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        PrintWriter writer = resp.getWriter();
        writer.println("Hello World");
    }

}
```

설정 파일을 참고해 요청에 어떤 서블릿이 필요한지 확인하고, 컨테이너를 확인한다. 있다면 그걸 사용하고, 없다면 새로 생성해 사용한다. 컨테이너는 개발자대신 서블릿의 생명주기를 관리해준다!

하지만 모든 요청마다 서블릿을 만들면 용량을 어마무시하게 차지하고, (기본적으로 한 서블릿 당 한 스레드를 사용한다) 공통 로직이 중복된다는 단점들이 있다.

이런 단점을 해결하기 위한 게 `Dispatcher Servlet`이다. 이 서블릿은 하나만으로 여러 요청을 처리할 수 게 해준다! 이 서블릿은 Model - View - Controller 구조에서 주로 사용되니 알아두는 게 좋다.

## 출처
- https://www.elancer.co.kr/blog/detail/270
- https://youtu.be/calGCwG_B4Y?si=Mj3PGMzBhXxmSZew
- https://www.youtube.com/watch?v=calGCwG_B4Y&t=98s
- https://www.geeksforgeeks.org/java/introduction-java-servlets/
- https://daydayplus.tistory.com/31
- https://mangkyu.tistory.com/18
