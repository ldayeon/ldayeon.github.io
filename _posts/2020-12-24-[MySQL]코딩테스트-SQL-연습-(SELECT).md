---
title:  "[MySQL] 코딩테스트 SQL 연습 (SELECT)"
subtitle: "SELECT"

categories: coding_test
tags:
- MySQL
- SQL
- 프로그래머스 고득점 Kit
- 코딩테스트

last_modified_at:   2020-12-24
---


<br>
# 프로그래머스 > 고득점 Kit > SELECT

## 여러 기준으로 정렬하기 

>  동물 보호소에 들어온 모든 동물의 아이디와 이름, 보호 시작일을 이름 순으로 조회하는 SQL문을 작성해주세요. 단, 이름이 같은 동물 중에서는 보호를 나중에 시작한 동물을 먼저 보여줘야 합니다.

```sql
SELECT ANIMAL_ID, NAME, DATETIME FROM ANIMAL_INS ORDER BY NAME, DATETIME DESC
```



`ORDER BY [column1] [asc/desc],[column2] [asc/desc], ... ` 와 같이 기준으로 둘 column과 정렬방식을 함께 써준다. 앞에 쓰인 column 기준으로 먼저 정렬한 뒤 같은 것에 대해서는 그 다음 column 기준으로 정렬한다.  

<br>



## 상위 n개 레코드

> 동물 보호소에 가장 먼저 들어온 동물의 이름을 조회하는 SQL 문을 작성해주세요.

```sql
SELECT NAME FROM ANIMAL_INS ORDER BY DATETIME LIMIT 1
```

`LIMIT n` 과 같이 몇 개를 출력할 것인지 미리 정할 수 있다. SQL문의 마지막에 기재하면 된다.

<br>



추가 예정