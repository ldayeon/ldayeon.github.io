---
title:  "[Spring] Chapter 02. 스프링 부트에서 테스트 코드를 작성하자"
subtitle: "스프링 부트와 AWS로 혼자 구현하는 웹 서비스"

categories: spring
tags:
- Spring
- Spring Boot
- AWS
- Web Programming

last_modified_at:   2020-12-20
---
<br>

# 1. 테스트 코드

## TDD와 단위 테스트

### (1) TDD (Test-Driven Development)

+ TDD : 테스트가 주도하는 개발
  + 테스트 코드를 먼저 작성하고 개발

### (2) 단위 테스트 (Unit Test)

+ 단위 테스트 : TDD의 첫 단계인 기능 단위의 테스트 코드를 작성하는 것
  + TDD와 다르게 테스트 코드를 꼭 먼저 작성해야하는 것이 아님

<br>

## 단위 테스트의 필요성

1. 개발 초기에 문제 발견 가능
2. 코드 리팩토링 & 라이브러리 업그레이드 등에서 기존 기능이 올바르게 작동하는지 확인 가능
3. 실제 문서를 제공. 즉, 단위 테스트 자체가 문서로 사용될 수 있음

<br>

## 테스트 프레임워크

### xUnit

개발환경(x)에 따라 Unit 테스트를 도와주는 도구

+ JUnit(Java), DBUnit(DB), CppUnit(C++) 등

책에서는 JUnit 사용

<br><br>

# 2. HelloController 테스트 코드 작성하기

## src/main/java에 com.ldayeon.springboot package 생성

### Application.java 파일 생성

```java
package com.ldayeon.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application { //main class
    public static void main(String[] args){
        SpringApplication.run(Application.class,args);
    }
}

```

+ `@SpringBootApplication`
  + Springboot의 자동설정
  + 스프링 Bean 읽기&생성 모두 자동으로 설정
  + 이 코드가 있는 곳부터 설정을 읽음 → 항상 프로젝트의 최상단에 있어야 함
+ `SpringApplication.run` : 내장 WAS(톰캣x → Jar 패키징 파일 사용)를 실행

<br>

## src/main/java에 com.ldayeon.springboot.web package 생성

### HelloController.java 생성

```java
package com.ldayeon.springboot.web;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello(){
        return "hello";
    }
}
```

+ `@RestController`
  + JSON를 반환하는 Controller로 만듦
  + `@ResponseBody`를 메소드마다 선언하지 않아도 됨
+ `@GetMapping`
  + Get 요청을 받을 수 있는 API 생성
  + `@RequestMapping(method=RequestMethod.GET)`를 대신하여 사용

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
>> Task :compileJava
> C:\Users\user\IdeaProjects\aws-spring-webservice\src\main\java\com\ldayeon\springboot\web\dto\HelloResponseDto.java:9: error: variable name not initialized in the default constructor
>     private final String name;
>                          ^
> C:\Users\user\IdeaProjects\aws-spring-webservice\src\main\java\com\ldayeon\springboot\web\dto\HelloResponseDto.java:10: error: variable amount not initialized in the default constructor
>     private final int amount;
>                       ^
> 2 errors
>> Task :compileJava FAILED
> FAILURE: Build failed with an exception.
> * What went wrong:
> Execution failed for task ':compileJava'.
>> Compilation failed; see the compiler error output for details.
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
>> [참고] <a href="https://github.com/jojoldu/freelec-springboot2-webservice/issues/2">https://github.com/jojoldu/freelec-springboot2-webservice/issues/2</a>
>>

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
  + 외부에서 넘긴 파라미터를 가져오는 Annotation
  + 외부에서 `name`이라는 이름으로 넘긴 값을 메소드 파라미터 변수 `name`에 저장

<br>

### HelloControllerTest.java에 Test coding_test 추가

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
