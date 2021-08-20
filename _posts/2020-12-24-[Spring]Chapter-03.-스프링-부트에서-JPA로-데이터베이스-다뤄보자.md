---
title:  "[Spring]Chapter 03. 스프링 부트에서 JPA로 데이터베이스 다뤄보자"
subtitle: "스프링 부트와 AWS로 혼자 구현하는 웹 서비스"

categories: spring
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

```java
package com.ldayeon.springboot.web;

import com.ldayeon.springboot.service.posts.PostsService;
import com.ldayeon.springboot.web.dto.PostsSaveRequestDto;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

@RequiredArgsConstructor
@RestController
public class PostsApiController {
    private final PostsService postsService;

    @PostMapping("/api/v1/posts")
    public Long save(@RequestBody PostsSaveRequestDto requestDto){
        return postsService.save(requestDto);
    }
}
```

+ `/api/v1/posts` 주소를 요청하였을 때 `postsService.save(requestDto)`를 반환
  + `@RestController`가 JSON을 반환할 수 있도록 설정해줌

<br>

## src/main/java에 com.ldayeon.springboot.service.posts package 생성

### PostsService.java 생성

```java
package com.ldayeon.springboot.service.posts;

import com.ldayeon.springboot.domain.posts.PostsRepository;
import com.ldayeon.springboot.web.dto.PostsSaveRequestDto;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import javax.transaction.Transactional;

@RequiredArgsConstructor
@Service
public class PostsService {
    private final PostsRepository postsRepository;

    @Transactional
    public Long save(PostsSaveRequestDto requestDto){
        return postsRepository.save(requestDto.toEntity()).getId();
    }
}
```

+ `@Transactional`
  + 데이터베이스의 상태를 변경시키는 작업 또는 한번엥 수행되어야 하는 연산
  + 클래스 또는 메소드에 선언할 수 있음
  + 실행 도중 문제가 생기면 Rollback

<br>

## src/main/java에 com.ldayeon.springboot.web.dto package

### PostsSaveRequestDto.java 생성

```java
package com.ldayeon.springboot.web.dto;

import com.ldayeon.springboot.domain.posts.Posts;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

@Getter
@NoArgsConstructor
public class PostsSaveRequestDto {
    private String title;
    private String content;
    private String author;

    @Builder
    public PostsSaveRequestDto(String title, String content, String author){
        this.title=title;
        this.content=content;
        this.author=author;
    }

    public Posts toEntity() {
        return Posts.builder().title(title)
                .content(content)
                .author(author)
                .build();
    }
}
```

+ Entity를 request/response 클래스로 사용하지 않는 이유
  + request/response 용 Dto는 View를 위한 클래스라 자주 변경하게 됨
  + Entity는 데이터베이스와 맞닿은 클래스이므로 변경에 예민하지 않게 설계하기 위해서임
  + View Layer와 DB Layer의 역할을 확실히 분리해야 함

<br>

## src/test/java에 com.ldayeon.springboot.web package

### PostsApiContollerTest.java 생성

```java
package com.ldayeon.springboot.web;

import com.ldayeon.springboot.domain.posts.Posts;
import com.ldayeon.springboot.domain.posts.PostsRepository;
import com.ldayeon.springboot.web.dto.PostsSaveRequestDto;
import org.junit.After;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.boot.web.server.LocalServerPort;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.List;

import static org.assertj.core.api.Assertions.assertThat;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class PostsApiControllerTest {

    @LocalServerPort
    private int port;

    @Autowired
    private TestRestTemplate restTemplate;

    @Autowired
    private PostsRepository postsRepository;

    @After
    public void tearDown() throws Exception{
        postsRepository.deleteAll();
    }

    @Test
    public void Posts_등록된다() throws Exception{
        //given
        String title = "title";
        String content = "content";
        PostsSaveRequestDto requestDto = PostsSaveRequestDto.builder()
                .title(title)
                .content(content)
                .author("author")
                .build();
        String url = "http://localhost:"+port+"/api/v1/posts";

        //when
        ResponseEntity<Long> responseEntity = restTemplate.postForEntity(url, requestDto, Long.class);

        //then
        assertThat(responseEntity.getStatuscoding_test()).isEqualTo(HttpStatus.OK);
        assertThat(responseEntity.getBody()).isGreaterThan(0L);
        List<Posts> all = postsRepository.findAll();
        assertThat(all.get(0).getTitle()).isEqualTo(title);
        assertThat(all.get(0).getContent()).isEqualTo(content);
    }
}
```

+ `assertThat(all.get(0).getTitle()).isEqualTo(title)`
  + 전체 포스트 중 첫 번째 포스트의 제목을 "title"과 확인

<br>

## src/main.java에 com.ldayeon.springboot.web package

### PostsApiControlelr.java 추가

```java
package com.ldayeon.springboot.web;

import com.ldayeon.springboot.service.posts.PostsService;
import com.ldayeon.springboot.web.dto.PostsSaveRequestDto;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;

@RequiredArgsConstructor
@RestController
public class PostsApiController {
    private final PostsService postsService;

    @PostMapping("/api/v1/posts")
    public Long save(@RequestBody PostsSaveRequestDto requestDto){
        return postsService.save(requestDto);
    }
/*추가된 부분(여기부터)*/
    @PutMapping("/api/v1/posts/{id}")
    public Long update(@PathVariable Long id, @RequestBody PostsUpdateRequestDto requestDto){
        return postsService.update(id, requestDto);
    }

    @GetMapping("/api/v1/posts/{id}")
    public PostsResponseDto findById(@PathVariable Long id){
        return postsService.findById(id);
    }
/*추가된 부분(여기까지)*/
}

```

+ `@PutMapping`
  + PUT HTTP request를 처리하기 위한 Annotation
+ `@GetMapping`
  + GET HTTP request를 처리하기 위한 Annotation

<br>

## src/main/java에 com.ldayeon.springboot.web.dto package

### PostsResponseDto.java  생성

```java
package com.ldayeon.springboot.web.dto;

import com.ldayeon.springboot.domain.posts.Posts;
import lombok.Getter;

@Getter
public class PostsResponseDto {
    private Long id;
    private String title;
    private String content;
    private String author;
  
    public PostsResponseDto(Posts entity){
        this.id = entity.getId();
        this.title = entity.getTitle();
        this.content = entity.getContent();
        this.author = entity.getAuthor();
    }
}
```

+ Entity의 필드 중 일부만 사용

<br>

## src/main/java에 com.ldayeon.springboot.web.dto package

### PostsUpadateRequestDto.java 생성

```java
package com.ldayeon.springboot.web.dto;

import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

@Getter
@NoArgsConstructor
public class PostsUpdateRequestDto {
    private String title;
    private String content;

    @Builder
    public PostsUpdateRequestDto(String title, String content){
        this.title = title;
        this.content = content;
    }
}
```

<br>

## src/main/java에 com.ldayeon.springboot.domain.posts package

### Posts.java 추가

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
    /*추가된 부분(여기부터)*/
    public void update(String title, String content){
        this.title = title;
        this.content = content;
    }
    /*추가된 부분(여기까지)*/
}
```

<br>

## src/main/java에 com.ldayeon.springboot.service.posts package

### PostsService.java 추가

```java
package com.ldayeon.springboot.service.posts;

import com.ldayeon.springboot.domain.posts.Posts;
import com.ldayeon.springboot.domain.posts.PostsRepository;
import com.ldayeon.springboot.web.dto.PostsResponseDto;
import com.ldayeon.springboot.web.dto.PostsSaveRequestDto;
import com.ldayeon.springboot.web.dto.PostsUpdateRequestDto;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import javax.transaction.Transactional;

@RequiredArgsConstructor
@Service
public class PostsService {
    private final PostsRepository postsRepository;

    @Transactional
    public Long save(PostsSaveRequestDto requestDto){
        return postsRepository.save(requestDto.toEntity()).getId();
    }
/*추가된 부분(여기부터)*/
    @Transactional
    public Long update(Long id, PostsUpdateRequestDto requestDto){
        Posts posts = postsRepository.findById(id)
                .orElseThrow(()->new IllegalArgumentException("해당 게시글이 없습니다. id ="+id));

        posts.update(requestDto.getTitle(), requestDto.getContent());

        return id;
    }

    public PostsResponseDto findById (Long id){
        Posts entity = postsRepository.findById(id)
                .orElseThrow(() -> new IllegalArgumentException("해당 게시글이 없습니다. id="+id));

        return new PostsResponseDto(entity);
    }
/*추가된 부분(여기까지)*/
}

```

+ JPA의 영속성 컨텍스트
  + update 기능에서 쿼리를 날리는 부분이 없음
  + Entity를 영구 저장하는 환경
  + JPA의 엔티티 매니저가 활성화된 상태로 트랜잭션 안에서 데이터베이스에서 데이터를 가져오면 영속성 컨텍스트 유지된 상태
  + 이 상태에서 데이터의 값을 변경하면 트랜잭션이 끝나는 시점에 해당 테이블에 변경된 사항 적용
  + 때문에 Update 기능은 쿼리를 날리지 않아도 됨
    + 이러한 개념을 '더티 체킹'이라고 함

<br>

## src/test/java에 com.ldayeon.springboot.web package

### PostsApiControllerTest.java에 추가

```java
package com.ldayeon.springboot.web;

import com.ldayeon.springboot.domain.posts.Posts;
import com.ldayeon.springboot.domain.posts.PostsRepository;
import com.ldayeon.springboot.web.dto.PostsSaveRequestDto;
import com.ldayeon.springboot.web.dto.PostsUpdateRequestDto;
import org.junit.After;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.boot.web.server.LocalServerPort;
import org.springframework.http.HttpEntity;
import org.springframework.http.HttpMethod;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.List;

import static org.assertj.core.api.Assertions.assertThat;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class PostsApiControllerTest {

    @LocalServerPort
    private int port;

    @Autowired
    private TestRestTemplate restTemplate;

    @Autowired
    private PostsRepository postsRepository;

    @After
    public void tearDown() throws Exception{
        postsRepository.deleteAll();
    }

    @Test
    public void Posts_등록된다() throws Exception{
        //given
        String title = "title";
        String content = "content";
        PostsSaveRequestDto requestDto = PostsSaveRequestDto.builder()
                .title(title)
                .content(content)
                .author("author")
                .build();
        String url = "http://localhost:"+port+"/api/v1/posts";

        //when
        ResponseEntity<Long> responseEntity = restTemplate.postForEntity(url, requestDto, Long.class);

        //then
        assertThat(responseEntity.getStatuscoding_test()).isEqualTo(HttpStatus.OK);
        assertThat(responseEntity.getBody()).isGreaterThan(0L);
        List<Posts> all = postsRepository.findAll();
        assertThat(all.get(0).getTitle()).isEqualTo(title);
        assertThat(all.get(0).getContent()).isEqualTo(content);
    }
/*추가된 부분(여기부터)*/
    @Test
    public void Posts_수정된다() throws Exception{
        //given
        Posts savedPosts = postsRepository.save(Posts.builder()
        .title("title")
        .content("content")
        .author("author")
        .build());

        Long updateId = savedPosts.getId();
        String expectedTitle = "title2";
        String expectedContent="content2";

        PostsUpdateRequestDto requestDto = PostsUpdateRequestDto.builder()
                .title(expectedTitle)
                .content(expectedContent)
                .build();

        String url = "http://localhost:"+port+"/api/v1/posts/"+updateId;

        HttpEntity<PostsUpdateRequestDto> requestEntity = new HttpEntity<>(requestDto);

        //when
        ResponseEntity<Long> responseEntity = restTemplate.exchange(url, HttpMethod.PUT, requestEntity, Long.class);

        //then
        assertThat(responseEntity.getStatuscoding_test()).isEqualTo(HttpStatus.OK);
        assertThat(responseEntity.getBody()).isGreaterThan(0L);

        List<Posts> all = postsRepository.findAll();
        assertThat(all.get(0).getTitle()).isEqualTo(expectedTitle);
        assertThat(all.get(0).getContent()).isEqualTo(expectedContent);
    }
/*추가된 부분(여기까지)*/
}
```

<br>

## src/main/resources

#### application.properties 추가

```java
spring.jpa.show_sql = true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
/*추가된 부분(여기부터)*/
spring.h2.console.enabled=true
/*추가된 부분(여기까지)*/
```

+ 데이터베이스 H2는 메모리에서 실행하기 때문에 직접 접근하려면 웹 콘솔을 사용해야 함
+ 웹 콘솔 옵션 활성화

<br>

### 웹 콘솔

`http://localhost:8080/h2-console/`로 접속하면 아래와 같은 웹 콘솔에 접근할 수 있다.

![캡처](https://user-images.githubusercontent.com/37764581/103437948-b9ab6e00-4c70-11eb-96ca-dc02ef986b18.GIF)

Connect를 클릭하면 쿼리를 입력할 수 있는 화면이 나온다.

![화면 캡처 2021-01-01 203519](https://user-images.githubusercontent.com/37764581/103437956-e3fd2b80-4c70-11eb-8bbf-64a5d2a9f3fd.png)

<br><br>

# 5. JPA Auditing으로 생성시간/수정시간 자동화하기

보통 Entity는 해당 데이의 생성/수정 시간을 포함한다.<br>

→ JPA Auditing을 사용<br>

Java의 기본 날짜 타입인 `Date`의 문제점을 개선한 `LocalDate`와 `LocalDateTime`을 사용할 것이다.

## src/main/java에 com.ldayeon.springboot.domain package

### BaseTimeEntity.java 생성

```java
package com.ldayeon.springboot.domain;

import lombok.Getter;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

import javax.persistence.EntityListeners;
import javax.persistence.MappedSuperclass;
import java.time.LocalDateTime;

@Getter
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public class BaseTimeEntity {
    @CreatedDate
    private LocalDateTime createdDate;
  
    @LastModifiedDate
    private LocalDateTime modifiedDate;
}
```

+ `MappedSupperclass`
  + JPA Entity 클래스들이 BaseTimeEntity을 상속할 경우 필드들을 칼럼으로 인식하도록 함
+ `EntityListeners`
  + BaseTimeEntity 클래스에 Auditing 기능을 포함시킴
+ `@CreatedDate`, `@ModifiedDate`
  + Entity가 생성되고 수정될 때 시간이 자동 저장

<br>

## src/main/java에 com.ldayeon.springboot.domain.posts package

### Posts.java 추가

```java
public class Posts extends BaseTimeEntity { 
```

+ BaseTimeEntity를 상속하도록 설정

<br>

## src/main/java에 com.ldayeon.springboot package

### Application.java

```java
package com.ldayeon.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.data.jpa.repository.config.EnableJpaAuditing;

@EnableJpaAuditing /*추가된 부분*/
@SpringBootApplication
public class  Application {
    public static void main(String[] args){
        SpringApplication.run(Application.class,args);
    }
}

```

+ JPA Auditing 어노테이션들을 활성화

<br>

## src/test/java에 com.ldayeon.springboot.domain.posts package

### PostsRepositoryTest.java에 추가

```java
@Test
public void BaseTimeEntity_등록(){
    //given
    LocalDateTime now = LocalDateTime.of(2019, 6,4,0,0,0);
    postsRepository.save(Posts.builder()
                         .title("title")
                         .content("content")
                         .author("author")
                         .build());

    //when
    List<Posts> postsList = postsRepository.findAll();

    //then
    Posts posts = postsList.get(0);

    System.out.println(">>>>createDate="+posts.getCreatedDate()+", modifiedDate="+posts.getModifiedDate());
    assertThat(posts.getCreatedDate()).isAfter(now);
    assertThat(posts.getModifiedDate()).isAfter(now);
}

```

+ `Posts`가 `BaseTimeEntity`를 상속받았기 때문에 생성/수정 시간 필드를 가지고 있음

<br><br>

> **[참고]**
> 참고 도서 : 스프링 부트와 AWS로 혼자 구현하는 웹 서비스 (이동욱 저)
>
> Github : <a href="https://github.com/ldayeon/aws-spring-webservice">https://github.com/ldayeon/aws-spring-webservice</a>
