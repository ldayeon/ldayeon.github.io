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
+ h2
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
+ 
+ `@GeneratedValue`
+ `@Column`

<br>

## src/test/java에 com.ldayeon.springboot.web 생성

### HelloControllerTest.java 생성

```java
package com.ldayeon.springboot.web;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@RunWith(SpringRunner.class)
@WebMvcTest(controllers = HelloController.class)
public class HelloControllerTest {
    @Autowired
    private MockMvc mvc;

    @Test
    public void hello가_리턴된다() throws Exception{
        String hello="hello";

        mvc.perform(get("/hello"))
                .andExpect(status().isOk()) 
                .andExpect(content().string(hello));
    }
}
```

+ `@RunWith(SpringRunner.class)`
  + JUnit에 내장된 실행자 외에 다른 실행자를 실행시킴
  + 여기서는 스프링 실행자인 'SpringRunner' 사용
    + SpringBootTest와 JUnit 사이에 연결자 역할
+ `@WebMvcTest(controllers = HelloController.class)` 
  + 사용 가능 Annotation : @Controller, @ControllerAdvice
  + 사용 불가 Annotation : @Service, @Component, @Repository
+ `@Autowired`
  + Spring이 관리하는 Bean을 주입받기
+ `MockMvc mvc`
  + 웹API Test할 때 사용
  + GET, POST 등 API 테스트 가능
+ `mvc.perform(get("/hello"))`
  + `/hello` 주소로 Get 요청 받기
+ `.andExpect(status().isOk())`
  + mvc.perform의 결과 검증 & Header의 Status 검증
+ `.andExpect(content().string(hello))`
  + ResponseBody 검증 & Controller에서 반환한 값 확인

<br><br>

# 3. 롬복 설치

## 라이브러리 롬복(Lombok)

Java 개발할 때 자주 사용하는 코드 Getter, Setter, 기본 생성자, toString 등을 어노테이션으로 자동 생성

```groovy
compile('org.projectlombok:lombok')
```

`build.gradle`파일의 제일 아래 `dependencies` 블록에 위의 코드를 추가

*Lombok은 프로젝트마다 성정해주어야 함.*

<br><br>

# 4. HelloController 코드를 롬봄으로 전환하기

## com.ldayeon.springboot.web.dto package 생성

### Dto란

Data Transfer Object로 계층간 데이터 교환을 위한 객체

<br>

### HelloResponseDto.java 생성

```java
package com.ldayeon.springboot.web.dto;

import lombok.Getter;
import lombok.RequiredArgsConstructor;

@Getter
@RequiredArgsConstructor
public class HelloResponseDto {
    private final String name;
    private final int amount;
}
```

+ `@Getter`
  + 선언된 모든 필드의 get 메소드를 생성
+ `@RequiredArgsConstructor`
  + 선언된 모든 final 필드가 포함된 생성자를 생성
  + final이 없는 필드는 생성자에 포함 X
+ final : 초기값이 저장되면 프로그램 도중에 값을 수정할 수 없는 변수

<br>

## Dto에 적용된 롬복 Test 코드 작성

### HelloResponseDtoTest.java

```java
package com.ldayeon.springboot.web.dto;

import org.junit.Test;
import static org.assertj.core.api.Assertions.assertThat;

public class HelloResponseDtoTest {
    @Test
    public void 롬복_기능_테스트(){
        //given
        String name = "test";
        int amount = 1000;

        //when
        HelloResponseDto dto = new HelloResponseDto(name, amount);

        //then
        assertThat(dto.getName()).isEqualTo(name);
        assertThat(dto.getAmount()).isEqualTo(amount);
    }
}
```

+ `assertThat`
  + `assertj`라는 테스트 검증 라이브러리의 검증 메소드
  + 검증하고 싶은 메소드를 인자로 받음
+ `isEqualTo`
  + `assertj`의 동등 비교 메소드
  + `assertThat`에 있는 값과 `isEqualTo`의 값을 비교해서 같을 때만 성공

> ❗ Test 코드 빌드 중 오류 발생
>
> ```
> Starting Gradle Daemon...
> Gradle Daemon started in 3 s 745 ms
> 
> > Task :compileJava
> C:\Users\user\IdeaProjects\aws-spring-webservice\src\main\java\com\ldayeon\springboot\web\dto\HelloResponseDto.java:9: error: variable name not initialized in the default constructor
>     private final String name;
>                          ^
> C:\Users\user\IdeaProjects\aws-spring-webservice\src\main\java\com\ldayeon\springboot\web\dto\HelloResponseDto.java:10: error: variable amount not initialized in the default constructor
>     private final int amount;
>                       ^
> 2 errors
> > Task :compileJava FAILED
> FAILURE: Build failed with an exception.
> * What went wrong:
> Execution failed for task ':compileJava'.
> > Compilation failed; see the compiler error output for details.
> * Try:
> Run with --stacktrace option to get the stack trace. Run with --info or --debug option to get more log output. Run with --scan to get full insights.
> * Get more help at https://help.gradle.org
> BUILD FAILED in 15s
> 1 actionable task: 1 executed
> 
> ```
>
> 위와 같이 오류가 발생했고 저자의 Github의 Issue를 참고하였다. Gradle의 버전 차이로 발생하는 오류였다.
>
> Intellij에서 Alt+F12로 Terminal창을 켠 뒤 아래의 명령어를 입력하여 Gradle을 다운그레이드한 뒤 해결할 수 있었다.
>
> ```
> gradlew wrapper --gradle-version 4.10.2
> ```
>
> > [참고] <a href="https://github.com/jojoldu/freelec-springboot2-webservice/issues/2">https://github.com/jojoldu/freelec-springboot2-webservice/issues/2</a>

<br>

### HelloController.java에 ResponseDto 코드 추가

```java
package com.ldayeon.springboot.web;

import com.ldayeon.springboot.web.dto.HelloResponseDto;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello(){
        return "hello";
    }
    
	/*추가된 부분(여기부터)*/
    @GetMapping("/hello/dto")
    public HelloResponseDto helloDto(@RequestParam("name") String name, @RequestParam("amount") int amount){
        return new HelloResponseDto(name, amount);
    }
    /*추가된 부분(여기까지)*/
}

```

+ `@RequestParam`
  +  외부에서 넘긴 파라미터를 가져오는 Annotation
  + 외부에서 `name`이라는 이름으로 넘긴 값을 메소드 파라미터 변수 `name`에 저장

<br>

### HelloControllerTest.java에 Test code 추가

```java
package com.ldayeon.springboot.web;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;

import static org.hamcrest.Matchers.is;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;


//JUnit에 내장된 실행자 외에 다른 실행자를 실행시킴
//여기서는 스프링 실행자인 'SpringRunner' 사용 - SpringBootTest와 JUnit 사이에 연결자 역할
@RunWith(SpringRunner.class)
//사용 가능 Annotation : @Controller, @ControllerAdvice
//사용 불가 Annotation : @Service, @Component, @Repository
@WebMvcTest(controllers = HelloController.class)
public class HelloControllerTest {
    @Autowired
    private MockMvc mvc;

    @Test
    public void hello가_리턴된다() throws Exception{
        String hello="hello";

        mvc.perform(get("/hello"))
                .andExpect(status().isOk())
                .andExpect(content().string(hello));
    }
    
    /*추가된 부분(여기부터)*/
    @Test
    public void helloDto가_리턴된다() throws Exception{
        String name="hello";
        int amount = 1000;

        mvc.perform(get("/hello/dto")
                .param("name", name)
                .param("amount",String.valueOf(amount)))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.name", is(name))) //jsonPath : json 응답ㄱ값을 필드별로 검증할 수 있는 메소드
                .andExpect(jsonPath("$.amount", is(amount)));
    }
    /*추가된 부분(여기까지)*/
}

```

+ `param`
  + API 테스트할 때 사용될 요청 파라미터 설정
  + String만 가능함
+ `jasonPath`
  + JSON 응답값을 필드별로 검증할 수 있는 메소드
  + `$`를 기준으로 필드명을 명시 ex. `$.name`

<br>

## JSON이란

### JavaScript Object Notation

데이터를 저장하거나 전송할 때 많이 사용되는 경량의 데이터 교환 형식

+ Server와 Client 간의 교류에서 일반적으로 사용

+ *Postman에서 사용했던 parameter post를 넘기는 방식과 같은 방식*

  ```json
  {
  	"name" : "ldayeon",
  	"amount" : "1,000,000,000"
  }
  ```

<br><br>

> **[참고]**
> 참고 도서 : 스프링 부트와 AWS로 혼자 구현하는 웹 서비스 (이동욱 저)
>
> Github : <a href="https://github.com/ldayeon/aws-spring-webservice">https://github.com/ldayeon/aws-spring-webservice</a>
