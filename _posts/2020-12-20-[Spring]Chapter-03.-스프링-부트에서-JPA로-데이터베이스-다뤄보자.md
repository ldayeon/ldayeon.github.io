---
title:  "[Spring]Chapter03. 스프링 부트에서 JPA로 데이터베이스 다뤄보자"
subtitle: "스프링 부트와 AWS로 혼자 구현하는 웹 서비스"

categories: Spring
tags:
- Spring
- Spring Boot
- AWS
- Web Programming

last_modified_at:   2020-12-24
---

<br>

+ SQL Mapper

  + 객체와 관계형 데이터베이스의 데이터를 자동으로 연결해주는 것

  + ex. MyBatis

  + SI환경에서 주로 사용

    <br>

+ ORM(Object Relational Mapping)

  + 객체 관계 매핑

  + 데이터베이스와 객체 지향 프로그래밍 언어 간의 호환되지 않는 데이터를 변환하는 프로그래밍 기법

  + 기존의 Mapper보다 객체 지향적인 프로그래밍을 가능하게 함

    <br>

+ JPA

  + Java 표준 ORM

  + 쿠팡, 우아한 형제들, NHN 등 자사 서비스 개발 기업에서 주로 사용

    <br><br>

# 1. JPA 소개

## 관계형 데이터베이스와 객체지향 프로그래밍

### 문제 배경

관계형 데이터베이스가 SQL만 인식할 수 있기 때문에 기본적인 CRUD(Create, Read, UPdate, Delete) SQL을 매번 생성해야 함.<br>
	→ 애플리케이션 코드 < SQL 코드

<br>

### 문제점

+ 단순 반복 작업
  + 테이블 수보다 몇 배의 SQL 코드가 필요
  + 유지 보수 문제와도 연결
+ 패러다임 불일치
  + 관계형 데이터베이스의 초점 : 어떻게 데이터를 저장할지
  + 객체지향 프로그래밍의 초점 : 기능과 속성을 한 곳에서 관리
    + 따라서, 객체에 데이터베이스를 저장하려고 하면 문제 발생
    + 1:N, 상속 등의 다양한 객체 모델링을 데이터베이스로 구현할 수 X

<br>

### ✔ JPA가 이러한 문제점을 해결

+ 중간에서 두 패러다임을 일치시켜주기 위한 기술(인터페이스)
  + 개발자는 객체지향적으로 프로그래밍 → JPA가 이를 관계형 데이터베이스에 맞게 SQL 생성

<br>

### Spring Data JPA

JPA를 사용하기 위해서는 구현체 필요 ex. Hibernate, Eclipse Link 등 <br>

  → 하지만 Spring에서는 구현체를 직접 다루지 않고 **Spring Data JPA**라는 모듈 이용<br><br>

`JPA ← Hibernate ← Spring Data JPA` 의 관계성을 가짐<br>

+ 구현체 교체의 용이성
  + Hibernate 외의 다른 구현체로 십게 교체 가능
+ 저장소 교체의 용이성
  + 관계형 데이터베이스 외에 다른 저장소로 쉽게 교체 가능 ex. MongoDB
  + Spring Data의 하위 프로젝트는 CRUD의 인터페이스가 같기 때문 (Spring Data JPA, Spring Data Redis, Spring Data MongoDB 등)

<br><br>

## 실무에서 JPA

### 실무에서 사용하지 못하는 이유

1. 높은 러닝 커브를 야기
2. 객체지향 프로그래밍과 관계형 데이터베이스를 둘 다 이해

<br>

### JPA가 주는 보상

1. CRUD 쿼리를 직접 작성할 필요 X
2. 객체지향 프로그래밍 가능

<br>



> #### 👩‍💻 여기서부터 본격적이 웹 사이트 구축

<br><br>

# 2. 프로젝트에 Spring Data JPA 적용하기

## build.gradle에 의존성 등록

### build.gradle에 코드 추가

```java
buildscript{
    ext{ //전역변수 선언
        springBootVersion = '2.1.9.RELEASE'
    }
    repositories{
        mavenCentral()
        jcenter()
    }
    dependencies{
        classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
    }
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'

group 'com.aws.spring'
version '1.0-SNAPSHOT'

sourceCompatibility = 1.8

repositories { //라이브러리를 다운받을 원격저장소 선택 - mavenCentral, jcenter
    mavenCentral()
    jcenter()
}

dependencies { //프로젝트에 필요한 의존성 선언(버전 명시하지 않아야 위의 전역변수 사용)
    compile('org.springframework.boot:spring-boot-starter-web')
    compile('org.projectlombok:lombok') 
    compile('org.springframework.boot:spring-boot-starter-data-jpa') /*추가된 부분*/
    compile('com.h2database:h2') /*추가된 부분*/
    testCompile('org.springframework.boot:spring-boot-starter-test')
}

```

+ `Spring-boot-starter-data-jpa` 
  + 스프링 부트용 Spring Data JPA 추상화 라이브러리
  + 스프링 부트 버전에 맞춰 자동으로 JPA관련 라이브러리들의 버전을 관리
+ `h2`
  + 인메모리 관계형 데이터베이스
  + 별도의 설치가 필요 없이 프로젝트 의존성만으로 관리할 수 있음
  + 메모리에서 실행
    + 애플리케이션을 재시작할 때마다 초기화된다는 점을 이용하여 테스트 용도로 많이 사용
  + JPA의 테스트, 로컬 환경에서의 구동에서 활용

<br>

## src/main/java에 com.ldayeon.springboot.domain package 생성

+ domain package
  + 게시글, 댓글, 회원, 정산, 결제 등 SW에 대한 요구사항 또는 문제 영역
  + 기존에 MyBatis와 같은 쿼리 매퍼를 사용하 때 `dao` 패키지와 비슷
    + 도메인 = xml에 쿼리를 담는 일 + 클래스에 쿼리의 결과를 담는 일

<br>

### src/java/com.ldayeon.springboot.domain에 posts package생성 후 그 안에 Posts.java 추가

```java
package com.ldayeon.springboot.domain.posts;

import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

import javax.persistence.*;

@Getter
@NoArgsConstructor
@Entity
public class Posts {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY) 
    private Long id;

    @Column(length=500, nullable = false)
    private String title;

    @Column(columnDefinition = "TEXT", nullable = false)
    private String content;

    private String author;
    
    @Builder
    public Posts(String title, String content, String author){
        this.title=title;
        this.content=content;
        this.author=author;
    }
}
```

+ `Posts class` 
  + 실제 DB의 테이블과 매칭될 클래스
  + Entity 클래스
  + DB에 쿼리를 날리는 것 X → Entity 클래스의 수정을 통해 작업 O

**[JPA에서 제공하는 Annotation]**

+ `@Entity`
  + Get 요청을 받을 수 있는 API 생성
  + `@RequestMapping(method=RequestMethod.GET)`를 대신하여 사용
+ `@Id`
  + 해당 테이블의 PK
+ `@GeneratedValue`
  + PK의 생성규칙
  + 스프링 부트 2.0에서는 GenerationType.IDENTITY 옵션을 추가해야만 auto_increment 가능
+ `@Column`
  + 테이블의 칼럼
  + 굳이 선언하지 않더라도 해당 클래스의 필드는 모두 칼럼이 됨 (위의 author처럼)
  + 기본값 외에 추가로 변경하고 싶은 옵션이 있으면 사용

**[Lombok 라이브러리의 Annotation]**

+ `@NoArgsConstructor`
  + 기본 생성자 자동 추가
  + public Posts(){}와 같은 효과
+ `@Getter`
  + 클래스 내 모든 필드의 Getter 메소드 자동 생성
+ `@Builder`
  + 해당 클래스의 빌더 패턴 클래스 생성
  + 생성자 상단에 선언 시 생성자에 포함된 필드만 빌더에 포함

<br>

### Entity 클래스에서 DB 삽입

Posts 클래스에는 `Setter 메소드`가 없다.<br>`getter/setter`을 무작정 생성하는 경우 인스턴스값들이 언제 어디서 변해야하는지 코드상으로 구분하기 어려워지기 때문.

**→ 따라서, Entity 클래스에는 Setter 메소드를 만들지 않는다.**

대신, 목적과 결과가 분명한 방식으로 값 변경 함수를 구현한다.

<br>

**생성자를 통해 최종값**을 채운 후 DB에 삽입하거나 **`@Builder`를 통해 제공되는 빌더 클래스**를 사용해서 DB에 데이터를 삽입한다.

```java
Example.builder()
    .a(a)
    .b(b)
    build();
```

<br>

### PostsRepository.java 생성

```java
package com.ldayeon.springboot.domain.posts;

import org.springframework.data.jpa.repository.JpaRepository;

public interface PostsRepository extends JpaRepository<Posts, Long> {
}
```

+ `JpaRepository`
  + MyBatis나 ibatis에서 `Dao`로 불리는 DB Layer 접근자
  + JPA에서는 `Repository`라고 부름
  + `JpaRepository<Entity class, PK타입>`을 상송하면 기본적인 CRUD 메소드가 자동 생성
  + Entity 클래스와 Entity Repository는 함께 위치해야 함

## src/test/java에 com.ldayeon.springboot.domain.posts package 생성 

### PostsRepositoryTest.java 생성

```java
package com.ldayeon.springboot.domain.posts;

import org.junit.After;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.List;

import static org.assertj.core.api.Assertions.assertThat;

@RunWith(SpringRunner.class)
@SpringBootTest
public class PostsRepositoryTest {
    @Autowired
    PostsRepository postsRepository;

    @After
    public void cleanup(){
        postsRepository.deleteAll();
    }

    @Test
    public void 게시글저장_불러오기() {
        //given
        String title = "테스트 게시글";
        String content = "테스트 본문";

        postsRepository.save(Posts.builder()
                .title(title)
                .content(content)
                .author("dayeon964@naver.com")
                .build()
        );

        //when
        List<Posts> postsList = postsRepository.findAll();

        //then
        Posts posts = postsList.get(0);
        assertThat(posts.getTitle()).isEqualTo(title);
        assertThat(posts.getContent()).isEqualTo(content);
    }
}
```

+ `@After`
  + Junit에서 단위 테스트(`@Test`)가 끝날 때마다 수행되는 메소드를 지정
  + 테스트간 데이터 침범을 막기위해 사용
+ `postsRepository.save`
  + 테이블 `posts`에 insert/update 쿼리를 실행
  + id값이 있다면 update, 없다면 insert 실행
+ `postsRepository.findAll()`
  + 테이블 posts에 있는 모든 데이터를 조회

### src/main/resources에 application.properies 생성

```properties
spring.jpa.show_sql = true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
```

+ `spring.jpa.show_sql`
  + 콘솔에 SQL 쿼리 로그를 보이게 할 것인지 설정
+ `spring.jpa.properties.hibernate.dialect`
  + H2의 쿼리 문법을 MySQL 쿼리 문법으로 바꾸도록 설정

<br><br>

# 4. 등록/수정/조회 API 만들기

API를 만들기 위해서는 *3개의 Class* 가 필요하다.<br>

1. Request 데이터를 받을 **Dto**
2. API 요청을 받을 **Controller**
3. 트랜잭션, 도메인 기능 간의 순서를 보장하는 **Service**

<br><br>

## Spring 웹 계층

+ Web Layer
  + Controller(`@Controller`)와 JSP/Freemarker 등의 뷰 템플릿
  + 필터(`@Filter`), 인터셉터, 컨트롤러 어드바이스(`@ControllerAdvice`) 등 외부 요청과 응답에 대한 전반적인 영역
+ Service Layer
  + `@Service`에 사용되는 서비스 영역
  + 일반적으로 Controller와 Dao 중간 영역
  + `@Transactional`이 사용되는 영역
+ Repository Layer
  + Database와 같이 데이터 저장소에 접근하는 영역
  + Dao 영역과 비슷
+ Dtos(Data Transfer Object)
  + 계층 간에 데이터 교환을 위한 객체들의 영역
  + 뷰 템플릿 엔진에서 사용될 객체나 Repository Layer에서 결과로 넘겨준 객체 등
+ Domain Model
  + 도메인이라 불리는 개발 대상을 모든 사람이 동일한 관점에서 이해할 수 있고 공유할 수 있도록 단순화시킨 것

✅ 비즈니스 로직을 처리하는 곳은 ''**도메인**''

<br>

### src/main/javaPostsApiController.java 생성

<br><br>

> **[참고]**
> 참고 도서 : 스프링 부트와 AWS로 혼자 구현하는 웹 서비스 (이동욱 저)
>
> Github : <a href="https://github.com/ldayeon/aws-spring-webservice">https://github.com/ldayeon/aws-spring-webservice</a>
