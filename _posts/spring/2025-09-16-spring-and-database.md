---
title: Spring boot와 데이터베이스 연결하기
date: 2025-09-16 13:25:00 +0900
categories: Spring
tags: ["Spring", "공부"]
---

해당 글은 2023년 7월 6일 노션에 쓴 글을 2025년 9월 16일에 정리하고 내용을 추가한 것이다.

---

[요 영상](https://www.youtube.com/watch?v=A8qZUF-GcKo&feature=youtu.be)을 참고해 진행한 프로젝트다.

+) 본 프로젝트에서는 PostgreSQL을 사용했지만 여기서는 MySQL을 사용했다. MySQL이 이미 깔려있고, 이 글에서는 단순 API 연동만 할 거기 때문이다.

## 사용한 종속성

```
dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-web'
	implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
	runtimeOnly 'com.mysql:mysql-connector-j'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
	testRuntimeOnly 'org.junit.platform:junit-platform-launcher'
}
```
- `spring-boot-starter-web`: Restful을 포함한 웹 애플리케이션 빌딩용 종속성
- `spring-boot-starter-data-jpa`: Hibernate를 포함한 spring-data-jpa 스타터(관련 종속성 모음집)

> 해당 종속성을 통해 데이터베이스에서 데이터를 가져온다.

> spring data jpa는 Java Persistence API 기반 레포지토리를 이용해, 데이터 접근 애플리케이션을 구축할 수 있도록 해주는 종속성이다.

- `com.mysql:mysql-connector-j`: mysql DB 연결

## application.properties

데이터베이스와 연결하기위해 다음과 같은 설정을 해야한다.

```
# 애플리케이션 명. 자동생성된다.
spring.application.name=example

# 데이터베이스 주소
# jdbc:데이터베이스://아이피:포트:DB명
spring.datasource.url=jdbc:mysql://localhost:3306/test
# DB 유저 명
spring.datasource.username=root
# DB 유저 비밀번호
spring.datasource.password=1234
```

위는 데이터베이스와 연결하는데 반드시 필요한 기본적인 설정이다! 아래로는 부가적인 설정이다.

```
# 데이터베이스 유형 지정
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQLDialect
# true이면, createClob() 메서드 생성과 관련된 오류출력을 숨긴다.
spring.jpa.properties.hibernate.jdbc.lob.non_contextual_creation=true

# DB 테이블을 다루는 방식
spring.jpa.hibernate.ddl-auto=update
# 실시간으로 실행되는 SQL 로그 출력
spring.jpa.show-sql=true
```

- ddl-auto: ddl(데이터 정의 언어)를 자동으로 실행해준다.
  - create: 기존테이블 삭제 후 다시 생성 (DROP + CREATE)
  - create-drop: create와 같으나 종료시점에 테이블 DROP
  - update: 모델 추가와 같은 변경점만 반영. 이미 존재하는 칼럼의 경우 삭제할 수 없으며, 다른 타입으로 신규생성을 할 수 없다.
  - validate: 엔티티와 테이블이 정상 매핑되었는지만 확인
  - none: 자동 실행 사용하지 않음

> 실제 서비스 배포 시 create, create-drop과 같은 옵션을 사용하면 데이터베이스가 다 날아갈 수가 있다! 

이외에도 다른 종속성(Hikari)를 사용해 connectTimeout을 설정하거나, 연결 풀 사이즈를 설정할 수 있다.

## Model - Product

```java
package com.nohchunsam.example;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;

// 해당 클래스를 테이블과 매핑한다고 jpa에 알려준다.
@Entity
public class Product {
    @Id // 식별자임을 지정
    @GeneratedValue(strategy = GenerationType.AUTO) // 자동 증가 id
    private Long id;

    private String name;
    private String description;

    public Product() {

    }

    public Product(Long id, String name, String description) {
        this.id = id;
        this.name = name;
        this.description = description;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }
}
```

## Repository - ProductRepository

```java
package com.nohchunsam.example;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface ProductRepository extends JpaRepository<Product, Long> {
}
```

실제 DB에 접근해 쿼리를 수행하는 역할을 한다.

> 폴더를 따로 구성하지 않았기에 바로 Model - Product를 가져올 수 있었다.

- JpaRepository
    - JPA 구현을 위해 인터페이스로 `JpaRepository`를 상속했다.
    - `JpaRepository<Entity 클래스, PK 타입>`를 상속하면 기본적인 메서드가 자동으로 생성된다. 이를 통해 DB Create/Read/Update/Delete 쿼리를 수행할 수 있다.
    - 정해진 규칙과 단어를 조합해 메서드명을 작성하는 것만으로 다양한 쿼리를 생성할 수 있다.

## Controller - ProductController 

```
package com.nohchunsam.example;

import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

// REST API를 제공하는 컨트롤러로 설정
// @Controller에 @responseBody 추
@RestController
// URL 주소를 맵핑
@RequestMapping("/product")
public class ProductController {
    private final ProductRepository productRepository;
    // 빈 생성. 아래의 함수에 삽입되어 사용될 수 있음
    public ProductController(ProductRepository productRepository) {
        this.productRepository = productRepository; // productRepository에 의존성을 주입한다.
    }

    @GetMapping // 링크로 Get 호출을 받을 때 실행
    public ResponseEntity getAllProducts(){
        return ResponseEntity.ok(productRepository.findAll());
    }
}
```

Repository를 실행하고 API 응답을 리턴하기 위한 컨트롤러. 

이제 localhost:8080/product로 들어가면, 데이터베이스 테이블에 있는 데이터를 볼 수 있다.

![](img/spring_db_connect_sample.png)