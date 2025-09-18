---
title: Spring boot의 내장 서버
date: 2025-09-18 08:28:00 +0900
categories: Spring
tags: ["Spring", "공부"]
---

해당 글은 2023년 7월 6일 노션에 쓴 글을 2025년 9월 18일에 정리하고 내용을 추가한 것이다....만, 그냥 [이 글](https://akadar899.medium.com/difference-between-jar-and-war-f39b4a430a25)을 번역한 것에 가까워졌다.

---

스프링 부트의 build.gradle에서 `.jar` 파일을 생성해준다고 한다. 그런데 스프링 부트를 조금만 검색해보면 `.war` 파일도 있다. 둘의 차이점은 무엇이고 장단점은 무엇일까?


우선 둘 다 **Java 기반 애프리케이션을 배포/동작시킬 수 있도록 관련 파일을 패키징**해주는 게 주 역할이다.

## JAR (Java Archive) 파일

JAR 파일은 자바 클래스, 메타데이터, 자료(이미지나 텍스트)를 한 파일로 압축시킨 패키지 파일이다. 주로 스탠드 얼론 애플리케이션이나 배포용 라이브러리를 배포하는데 사용된다. 자바 애플리케이션의 "설치자"로 생각하면 이해가 쉽다. JAR에는 컴파일된 자바 코드와 압축된 자료가 있어 배포와 실행이 쉽다.

> 스탠드 얼론 Stand alone: 파일 자체로 완전하게 구동한다

### JAR 사용 목적

 - 자바 프로그램이나 라이브러리 배포
 - 재사용 코드를 한 파일로 패키징
 - 복잡한 프로그램을 간단하게 실행

### JAR 동작 방식

 JAR 파일을 실행하면 자바 런타임 환경 JRE가 안에 있는 `manifest`를 읽고, 어느 클래스부터 실행하지 결정한다. 복잡한 실행을 단순화하기 때문에 데스크탑 애플리케이션이나 라이브러리에 자주 사용된다.

## WAR (Web Application Archive) 파일

 웹기반 자바 애플리케이션을 패키징하는데 특화된 파일이다. WAR 파일은 서블릿 컨테이너나 톰캣, 제리 같은 웹 서버 안에서 동작하도록 만들어진다.


WAR 파일은 웹 서버에 배포되도록 Java 서블릿, JSP(Java 서버 페이지) 및 기타 웹 관련 리소스(HTML, CSS, JS 등)를 단일 압축 파일로 묶는다.

### WAR 사용 목적

- 웹서버에 웹 애플리케이션 배포
- 웹 애플리케이션의 구성요소(서블릿, HTML, CSS)를 하나로 묶기

### WAR 동작 방식

WAR 파일은 `web.xml`을 포함하는데, 이 파일이 여러 요청들을 어떻게 처리해야하는지 알려준다.

## JAR 파일 구조

예시 JAR 파일이다.

```
- META-INF/
   - MANIFEST.MF (버전, 메인 클래스, 등 같은 메타데이터를 포함한다.)
- com/
   - example/
     - MyApp.class (컴파일된 자바 코드)
- resources/
   - image.png (앱에 필요한 다른 자료들)
```

## WAR 파일 구조

WAR 파일은 웹 구성요소들을 포함해 조금 복잡하다.

```
- WEB-INF/
   - web.xml (서블릿과 필터 구성)
   - classes/ (컴파일된 자바 클래스)
   - lib/ (앱에 필요한 외부 라이브러리)
- index.jsp (JSP 파일)
- css/ (CSS 파일)
- js/ (자바스크립트 파일)
```

## 주요 차이점

### 1. 사용 범위

- JAR: 스탠드얼론 자바 애플리케이션 패키징과 배포에 사용
- WAR: 웹 기반 자바 애플리케이션 패키징에 사용

### 2. 구조와 내용

- JAR: 압축된 클래스, 자료를 포함
- WAR: JSP, 서블릿, 구성 파일과 같은 웹 구성요소를 포함

### 3. 패키징과 배포

- JAR: 데스크톱이나 스탠드얼론 환경에서 배포
- WAR: HTTP 요청을 다루기 위해 웹서버에 배포

## 개발 과정에서의 JAR과 WAR

- `Maven`과 `Gradle`이 JAR과 WAR 파일을 만들고 관리한다. JAR 파일은 주로 컴파일 단계에서 빌드되고, WAR 파일은 웹 애플리케이션 배포 단계에서 빌드된다.

WAR은 이런식으로 빌드하면 된다.

```
plugins {
	id 'java'
	id 'org.springframework.boot' version '3.5.5'
	id 'io.spring.dependency-management' version '1.1.7'
}

group = 'com.nohchunsam'
version = '0.0.1-SNAPSHOT'
description = 'Demo project for Spring Boot'
apply plugin: 'war'
```

## JAR ↔ WAR
- JAR → WAR: 폴더에 `WEB-INF` 파일을 넣고 패키징
- WAR → JAR: 웹 구성요소를 제거하고 패키징


## 출처
- https://ifuwanna.tistory.com/224#google_vignette
- https://akadar899.medium.com/difference-between-jar-and-war-f39b4a430a25