---
title:  "[JAVA]String 처리 관련 함수들 정리"
subtitle: "JAVA 코딩 테스트"

categories: Code
tags:
- JAVA
- 코딩테스트
author_profile: true
last_modified_at:   2020-11-12
---

#### 📝 String 비교 함수

1. 두 String이 같은지 비교하는 함수로는 `equals()` 가 있다.  `Objects.equals(str1,str2)`를 사용한다면 문자열이 null인 상황을 따로 처리해주지 않아도 되므로 편리하다.

   ```java
   String str1="abcde";
   String str2="aaaaa";

   if(Objects.equals(str1,str2))
   	System.out.println("같음");

   if(str1.equals(str2))
   	System.out.println("같음");
   ```
2. 두 String의 대소관계를 비교할 수 있는 함수로는 `compareTo()`가 있다.
   반환값에 따라 method를 호출한 String이 사전적으로 얼마나 더 앞에 있는지 알 수 있다. 대소문자 구분하지 않고 비교하려면 `compareToIgnoreCase()`를 사용하면 된다.

   ```java
   String str1="abcde";
   String str2="aaaaa";
   System.out.println(str1.compareTo(str2));

   // 출력 : 1 (str1[1]의 b가 str2[1]의 a보다 1만큼 뒤에 있으므로)

   String str3="Abcde";
   String str4="aaaaa";

   System.out.println(str3.compareTo(str4));

   // 출력 : 1 (A와 a는 같은 것으로 처리됨)
   ```

  

#### 📝 특정 문자열을 포함하는지 확인하는 함수

1. 단순히 특정 문자열을 포함하는지 여부만 확인하려면 `contains()`를 사용하면 된다.

   ```java
   String str="I wanna go home.";

   if(str.contains("home"))
   	System.out.println("집 가고 싶다..");

   //출력 : 집 가고 싶다..
   ```
2. 특정 문자열이 어느 부분에 포함되었는지 제한을 두려면 `matches()`를 사용해야한다.

   ```java
   String str="I wanna go home.";

   if(str.matches("(.*)home(.*)")) //위와 동일
   	System.out.print("집 가고 싶다..");

   if(str.matches("home(.*)")) //home 앞에 어떤 문자열도 없어야 함
   	System.out.print("집?");

   if(str.matches("I wanna go .....")) //'.' 하나가 하나의 char을 대체
   	System.out.print("어디?");

   if(str.matches("I wanna go home\\.")) // '\\.'가 '.'을 찾는다는 의미
   	System.out.print("...");

   //출력 : 집 가고 싶다..어디?...
   ```
