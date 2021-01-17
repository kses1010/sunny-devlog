---
title: '프로그래머스 SQL문제 정리'
date: 2021-01-17
category: 'SQL'
draft: false
---

가끔 코딩테스트에서 SQL문제가 나오기 때문에 그걸 위한 프로그래머스 문제 정리입니다.

## SUM, MAX, MIN

- 중복과 NULL을 제외한 COUNT

```sql
SELECT COUNT(DISTINCT column_name) FROM table_name;
```

## Group By

- 고양이와 개는 몇마리 일까?

```sql
SELECT ANIMAL_TYPE, COUNT(ANIMAL_TYPE) AS COUNT
FROM ANIMAL_INS
GROUP BY ANIMAL_TYPE ORDER BY ANIMAL_TYPE;
```

- 동명 동물 수 찾기

```sql
SELECT NAME, COUNT(NAME) AS COUNT
FROM ANIMAL_INS
GROUP BY NAME HAVING COUNT > 1 ORDER BY NAME;
```

- 입양 시각 구하기

```sql
SELECT HOUR(DATETIME) AS HOUR, COUNT(DATETIME) AS COUNT
FROM ANIMAL_OUTS
WHERE HOUR(DATETIME) >= 9
  AND HOUR(DATETIME) <= 19
GROUP BY HOUR(DATETIME)
ORDER BY HOUR(DATETIME)
```

- 입양 시각 구하기 2

```sql
SET @hour := -1; -- 변수 선언

SELECT (@hour := @hour + 1) as HOUR,
(SELECT COUNT(*) FROM ANIMAL_OUTS WHERE HOUR(DATETIME) = @hour) as COUNT
FROM ANIMAL_OUTS
WHERE @hour < 23
```

- **SET 옆에 변수명과 초기값을 설정**할 수 있습니다.
  - @가 붙은 변수는 프로시저가 종료되어도 **유지**된다고 생각하면 됩니다.
  - 이를 통해 값을 누적하여 0부터 23까지 표현할 수 있습니다.
- @hour은 초기값을 -1로 설정합니다. PL/-SQL 문법에서 **:=**은 비교 연산자 =과 혼동을 피하기 위한의 **대입 연산**입니다.
- SELECT (@hour := @hour +1) 은 @hour의 값에 1씩 증가시키면서 SELECT 문 전체를 실행하게 됩니다.
- 이 때 **처음에 @hour 값이 -1 인데, 이 식에 의해 +1 이 되어 0**이 저장됩니다.
  - HOUR 값이 0부터 시작할 수 있습니다.
  - WHERE @hour < 23일 때까지, @hour 값이 **계속 + 1씩 증가**합니다.

## IS NULL

- NULL 처리하기

```sql
SELECT ANIMAL_TYPE, IFNULL(NAME, 'No name') AS NAME, SEX_UPON_INTAKE
 FROM ANIMAL_INS
 ORDER BY ANIMAL_ID;
```

## LEFT JOIN

- 없어진 기록찾기

```sql
SELECT AO.ANIMAL_ID, AO.NAME FROM ANIMAL_OUTS AS AO
LEFT JOIN ANIMAL_INS AS AI
ON AO.ANIMAL_ID = AI.ANIMAL_ID
WHERE AI.ANIMAL_ID IS NULL
ORDER BY AO.ANIMAL_ID;
```

- 있었는데요 없었습니다.

```sql
SELECT AI.ANIMAL_ID, AI.NAME FROM ANIMAL_INS AS AI
LEFT JOIN ANIMAL_OUTS AS AO
ON AI.ANIMAL_ID = AO.ANIMAL_ID
WHERE AI.DATETIME > AO.DATETIME
ORDER BY AI.DATETIME;
```

- 오랜 기간 보호한 동물(1)

```sql
SELECT AI.NAME, AI.DATETIME FROM ANIMAL_INS AS AI
LEFT JOIN ANIMAL_OUTS AS AO
ON AI.ANIMAL_ID = AO.ANIMAL_ID
WHERE AO.ANIMAL_ID IS NULL
ORDER BY DATETIME LIMIT 3;
```

- 보호소에서 중성화된 동물

```sql
SELECT AO.ANIMAL_ID, AO.ANIMAL_TYPE, AO.NAME FROM ANIMAL_OUTS AS AO
LEFT JOIN ANIMAL_INS AS AI
ON AO.ANIMAL_ID = AI.ANIMAL_ID
WHERE AI.SEX_UPON_INTAKE LIKE 'Intact%'
AND AO.SEX_UPON_OUTCOME REGEXP 'Spayed|Neutered'
```

REGEXP를 입력하면 LIKE문 여러개를 쓰는 효과를 볼 수 있다.

## String, Date

- 루시와 엘라 찾기

```sql
SELECT ANIMAL_ID, NAME, SEX_UPON_INTAKE FROM ANIMAL_INS
WHERE NAME IN ('Lucy', 'Ella', 'Pickle', 'Rogan', 'Sabrina', 'Mitty')
ORDER BY ANIMAL_ID;
```

- 이름에 el

```sql
SELECT ANIMAL_ID, NAME FROM ANIMAL_INS
WHERE NAME LIKE '%EL%'
 AND ANIMAL_TYPE = 'DOG'
ORDER BY NAME;
```

- 중성화 여부 파악하기

```sql
SELECT ANIMAL_ID, NAME, CASE WHEN SEX_UPON_INTAKE REGEXP 'Neutered|Spayed'
THEN "O" ELSE "X" END AS '중성화'
FROM ANIMAL_INS
```

→ Case, Then, Else 문을 사용하여 조건문에 만족하는 컬럼에 표시.

- 오랜 기간 보호한 동물(2)

```sql
SELECT AI.ANIMAL_ID, AI.NAME FROM ANIMAL_INS AS AI
LEFT JOIN ANIMAL_OUTS AS AO ON AI.ANIMAL_ID = AO.ANIMAL_ID
ORDER BY AO.DATETIME - AI.DATETIME DESC LIMIT 2;
```

- DATETIME에서 DATE로 형 변환

```sql
SELECT ANIMAL_ID, NAME, DATE_FORMAT(DATETIME, '%Y-%m-%d') AS 날짜
FROM ANIMAL_INS;
```

## Summer/Winter Coding

- 우유와 요거트가 담긴 장바구니

```sql
SELECT CART_ID FROM CART_PRODUCTS
WHERE CART_ID IN
(SELECT CART_ID FROM CART_PRODUCTS WHERE NAME = 'Milk') ## 우유를 먼저 찾고
AND NAME = 'Yogurt' ORDER BY CART_ID; ## 요거트를 찾으면 된다.
```

→ 서브쿼리(이중쿼리)를 이용하여 쓰는 문제입니다. 다른 방법도 다양하지만 저는 이 방법이 쉽고 빠르게 찾을 수 있었습니다.

SQL 문제는 정말 안쓰면 잊어버리기 때문에 일주일에 2번 정도 푸는 게 좋겠네요. 다음에는 hackerrank 문제를 풀면서 어려웠던 문제를 하나씩 포스팅 할 생각입니다.
