---
title:  "[Spring] Chapter 04. 머스테치로 화면 구성하기"
subtitle: "스프링 부트와 AWS로 혼자 구현하는 웹 서비스"

categories: Spring
tags:
- Spring
- Spring Boot
- AWS
- Web Programming

last_modified_at:   2021-01-01
---

<br>

# 1. 서버 템플릿 엔진과 머스테치 소개

## 서버 템플릿 엔진

### 템플릿 엔진

웹 개발에서 지정된 템플릿 양식과 데이터가 합쳐져 HTML문서를 출력하는 SW<br>

ex. JSP, React, Vue

+ JSP, Freemarker : 서버 템플릿 엔진
+ React, Vue : 클라이언트 템플릿 엔진

<br>

+ 서버 템플릿 엔진을 이용한 화면 생성
  + 서버에서 Java 코드 처리 → **HTML 코드로 변환** → 화면에 출력
  + 자바스크립트는 브라우저 위에서 작동
  + 그렇기 때문에 브라우저에서 자바스크립트가 작동할 때는 서버 템플릿 엔진이 제어할 수 없음
+ 클라이언트 템플릿 엔진
  + Vue.js, React.js를 이용한 pSPA(Single Page Application)은 브라우저에서 화면을 생성
    + 서버에서 코드가 벗어난 상태
  + **Server에는 XML, Json 형태의 데이터**만 전달하고 **클라이언트에서 조립**
  + React나 Vue와 같은 자바스크립트 프레임워크에서는 서버 사이드 렌더링을 지원하기도 함
    + Spring Boot에서 지원하는 기술 : Nashorn, J2V8

<br>

## 머스테치

### 머스테치(mustache)란?

JSP와 같이 HTML을 만들어주는 템플릿 언어<br>

+ 루비, 자바스크립트, 파이썬, PHP, 자바, 펄, ASP 등 다양한 언어 지원
  + 자바에 사용 - 서버 템플릿 엔진
  + 자바스크립트에 사용 - 클라이언트 템플릿 엔진
+ 





### <br><br>

> **[참고]**
> 참고 도서 : 스프링 부트와 AWS로 혼자 구현하는 웹 서비스 (이동욱 저)
>
> Github : <a href="https://github.com/ldayeon/aws-spring-webservice">https://github.com/ldayeon/aws-spring-webservice</a>
