---
title: '관계형 데이터베이스 실전 입문 - 05. 릴레이션의 직교성'
date: 2021-03-16
category: 'MySQL'
draft: false
---

# 릴레이션의 직교성

> 직교성은 여러 개의 릴레이션 사이의 중복에 관한 개념이다.

즉, DB 전체에서 중복을 제거하는 작업이다. 각 릴레이션에서 중복을 제거해도 DB 전체를 볼 때 중복이 남아 있다면 결국 불일치의 원인이 된다.

# 1. 릴레이션의 직교성과 중복

릴레이션의 직교성은 한마디로 말하면 같은 값을 갖지 않는 것이다.

## 레플리카

가장 단순하고 알기 쉬운 직교화되지 않은 릴레이션의 예는 완전히 같은 구조와 같은 데이터를 가진 릴레이션이 여러 개 있는 경우다. 같은 데이터를 일부러 여러 개의 릴레이션에 작성하는 것은 굉장한 낭비이며 모순이 생기는 원인이 된다.

→ 실제로는 이러한 DB를 만날 일은 없다.

## 같은 형태의 릴레이션

직교란 두 개 이상의 릴레이션이 같은 값을 갖지 않는 상태를 말한다.

제목이 완전히 같은 릴레이션끼리라면 같은 값을 갖는지 아닌지는 튜플의 값을 비교한다. 만약 같은 값을 가지고 있다면 중복되므로 모순이 발생하는 원인이 된다.

같은 제목을 가진 두 개의 릴레이션이 같은 값을 가졌는지 확인하려면 두 개의 릴레이션을 결합해 보면 된다. 같은 값을 가지지 않으면 결합한 결과가 공집합이 된다.
→ 결합(JOIN)은 SQL에서도 쉽게 테스트가 가능.

두 개의 테이블이 같은 데이터를 가지지 않는 것을 보장할 수 있는 제약을 SQL로 쉽게 표현할 수 있다면 좋겠지만, 그런 기능은 없다.(외부키는 두 개의 테이블에 같은 데이터가 보장하는 제약)

현실적인 해결책은 트리거를 사용해야 한다.

## 제목 일부만 같은 릴레이션

부분적으로만 값이 일치할 때도 직교하지 않는다고 판단해야 하는 경우가 있다.

해결책은 있다. 바로 6NF이다.

6NF까지 분해한 릴레이션에는 명백하지 않은 함수 종속성이나 암시적인 결합 종속성이 존재하지 않는다. 6NF의 릴레이션은 무손실 분해를 할 수 없으므로 튜플의 부분집합에 대해서 생각할 필요가 없기 때문이다.

→ 모든 릴레이션을 6NF가 될 때까지 무손실 분해하고 튜플을 비교해 중복이 없는 것을 확인하면 직교성을 보장할 수 있다.

# 2. 릴레이션 직교화를 위한 전략

## 정규화

원래 DB 설계 시에는 실제 릴레이션(테이블)을 5NF까지 정규화하면 충분하다. 그러나 6NF를 활용하려면 5NF까지 정규화하는 것이 중요하다.

즉, 정규화를 제대로 해두면 단지 릴레이션 그 자체를 다루기 쉬울 뿐만 아니라 직교성을 확인할 때도 도움이 된다.

## 속성(컬럼)의 이름 통일하기

이름 통일하기는 직교화를 하는 데 있어서 꼭 추천하는 방법이다. 직교화를 여부를 확인하려면 다른 릴레이션에 있는 속성(SQL 컬럼)이 같은 것을 의미하고 있는지 확인해야 한다. 같은 의미의 속성인데도 이름이 다르면 얼핏봐선 같은지 알기 어려우므로 놓치게 될 가능성이 커진다.

### 컬럼 명명 주의점

- 명명규칙 통일하기
  - 알파벳 or 한글 || 카멀케이스 or 스네이크 케이스
- 주어를 포함한다.
  - 누구의 속성인지 나타내기 위해 student_name, sns_user_email 등등 구체적인 의미가 필요하다.

## 응용프로그램의 정합성

직교하지 않은 릴레이션은 대부분 응용프로그램의 설계상 문제가 발생한다. 다른 두 가지의 기능에 같은 의미의 데이터가 필요할 때는 공통의 구성 요소를 설계하는 대신에 각 독립된 DB에 데이터를 등록하면 직교하지 않은 DB가 완성된다. 이 경우 발생할 가능성은 응용프로그램의 규모가 커지면 커질수록, or 기능이 늘어나면 늘어날수록 높아질 것이다. (ex: 시스템 통합)

공통적인 데이터가 필요한 경우 응용프로그램 쪽의 코드도 공유할 수 있게 리팩터링을 수행해야 한다.

### 전체를 직교화할 필요는 없다.

직교성은 DB 설계를 고려하는 데 중요한 개념이지만 반드시 모든 릴레이션을 직교화할 필요는 없다.

**예시) A라는 테이블에 데이터를 등록하고 또 다른 조건을 만족하면 B라는 테이블에 데이터를 등록하는 경우**

두 테이블이 나타내는 조건을 만족하는 사용자를 구하는 쿼리

```sql
SELECT user_name FROM A INNER JOIN B USING(user_name);
```

어느 한쪽의 조건을 만족하는 사용자를 구하는 쿼리

```sql
SELECT user_name FROM A
UNION DISTINCT
SELECT user_name FROM B
```

릴레이션은 집합이므로 교집합과 합집합으로 연산하고 싶을 때도 있다.

이와 같이 연산 자체는 중복된 데이터가 서로의 릴레이션에 포함돼 있어도 문제가 되지 않는다.

# 3. 중복을 해결하여 얻는 이점

## 변칙을 막을 수 있다.

변칙이란 DB에 포함된 모순, 즉 DB 내에 상반된 사실을 나타내는 명제가 포함된 것을 말한다.

DB에서 변칙이 발생하면 질의의 결과(SQL의 DML)가 옳다고 보장할 수 없다.

## 필요한 데이터가 어디에 있는지 명확해진다.

필요한 데이터가 어디에 있는지 명확하면 데이터를 갱신할 때도 어느 튜플을 갱신하면 좋은지 명확해지는 장점이 있다. 중복이 없다면 참조, 갱신 어떤 경우도 헤매지 않고 쿼리를 작성할 수 있다.

## 쿼리의 작성이 선언적이 된다

최소한 모든 테이블이 1NF가 되어 있다면 관계형 모델을 기반으로 쿼리를 작성할 수 있다.

→ 쿼리가 술어논리적으로 어떤 데이터가 필요한지 정의할 수 있다는 의미다.

선언적으로 쿼리를 작성함으로써 프로그래밍의 생산성을 크게 높인다.

## 불필요한 무손실 분해는 필요 없다.

릴레이션이 정규화 되어있지 않다는 것은 어떤 두 개 이상의 릴레이션을 결합(JOIN)한 결과가 그 릴레이션이 된다는 의미다.

→ 필연적으로 FROM 절에 서브쿼리를 사용해야 한다.

![image](https://user-images.githubusercontent.com/49144662/111319880-cbb27f80-86a9-11eb-9164-53780858fbfd.png)

![image](https://user-images.githubusercontent.com/49144662/111319923-d66d1480-86a9-11eb-9498-9cf6d9939532.png)

"Ruby On Rails의 수업을 수강하고 있는 학생이 재학 중인 학과의 대표번호 목록" 을 구하는 쿼리를 구해보자.

**Ruby On Rails의 수업을 수강하고 있는 학생이 재학 중인 학과의 목록**

```sql
SELECT DISTINCT '학과' FROM student_school_class as t1
WHERE '수업' = 'Ruby On Rails';
```

그 다음으로 "학과별 대표번호 목록"을 작성한다.

```sql
SELECT DISTINCT '학과', '대표번호' FROM student_school as t2;
```

이 두 개의 쿼리 결과를 결합하면 `Ruby On Rails의 수업을 수강하고 있는 학생이 재학 중인 학과의 대표번호 목록` 을 구할 수 있다.

```sql
SELECT DISTINCT '학과', '대표번호' FROM
(SELECT DISTINCT '학과' FROM student_school_class as t1
WHERE '수업' = 'Ruby On Rails';) ft1
INNER JOIN
(SELECT DISTINCT '학과', '대표번호' FROM student_school as t2;) ft2
USING('학과');
```

무손실 분해를 포함하면 쿼리가 복잡해지고 효율적이지 않다.

쿼리가 복잡하게 작성되는 근본적인 문제는 각 릴레이션(테이블)이 정규화되어 있지 않기 때문이다.

만약 정규화된 테이블이 있다면 다음과 같다.

- student_school을 무손실 분해한 {학과, 대표번호}를 포함하는 schools 테이블
- student_school_class을 무손실 분해한 {학과, 수업}을 포함하는 school_classes 테이블

```sql
SELECT '학과', '대표번호'
FROM schools as t3 INNER JOIN school_classes as t4 USING('학과')
WHERE '수업' = 'Ruby On Rails';
```

SELECT의 기본형에 맞춰져 있어서 쿼리가 무엇을 나타내는지 이해하기 쉽다. DB가 가진 각 릴레이션이 정규화 되어있다면 서브쿼리를 사용할 기회가 줄어들어 쿼리를 단순하게 표현할 수 있다.

## 복잡한 제약은 필요없다

정규화와 직교화가 완료되지 않은 경우에 갱신하더라도 변칙이 생기지 않게 하려면 제약을 걸어야 한다.

그러나 이러한 제약은 단순하게 작성할 수 없다.

어떤 테이블에 명백하지 않은 함수 종속성이 포함돼 있을 때 모순을 만들지 않으려면 함수 종속성이 손상되지 않게 해야 한다.

X → Y라는 함수 종속성이 있다면 같은 X와 Y를 가진 행은 모두 한 번에 같은 값이므로 갱신해야 한다.

UPDATE할 때 모순을 조심해야 한다.

```sql
# 모순이 없는 예
UPDATE t2 SET '대표번호' = 'ww-wwww-wwww'
WHERE '학과' = '데이터베이스';

# 변칙이 발생하는 예
UPDATE t2 SET '대표번호' = 'ww-wwww-wwww'
WHERE '이름' = '오민혁';
```

변칙이 발생하지 않게 제약을 걸지만, SQL은 제약이 아닌 트리거를 사용해야 한다. 다만 이러한 제약을 표현하려면 행별이 아니고 쿼리별로 실행하는 트리거가 필요하다. (트리거 내부에는 GROUP BY를 사용하여 집계를 수행해야 한다.)

직교하지 않는 두 개의 테이블을 갱신할 때 변칙이 생기지 않게 하려면 어떻게 해야할까?

→ 양 쪽의 테이블에 트리거를 걸어 `갱신한 데이터와 같은 값이 다른 쪽 테이블에 있다면 그쪽도 갱신한다` 와 같은 처리가 필요할 것이다.

→ 외부키 사용은 각 테이블에는 다른 테이블에는 존재하지 않는 데이터를 포함하고 있을 가능성이 있어서 외부키를 사용할 수 없다. 외부키를 사용할 수 있는 것은 자식 테이블의 키 데이터가 모든 부모 테이블에 포함돼 있을 때뿐이다.

→ 공통의 행에는 있지만, 그렇지 않는 행이 서로의 테이블에 포함될 때는 외부키를 사용할 수 없다.

정규화되어 있지 않고 다른 테이블과 직교하지 않을 때에 변칙이 발생하지 않게 하려면 어떻게 해야할까?

→ 저자는 이딴 짓할바엔 DB 설계를 다시한다고 한다.

## 응용프로그램의 코드에 낭비가 없어진다

응용프로그램 쪽에서 변칙을 확인하는 방법도 검토할 수 있다. 하지만 이 방법은 생각보다 어렵다.

또한, 응용프로그램 쪽에 새로운 로직을 넣으면 코드가 늘어날 뿐만 아니라 코드 테스트도 필요하게 된다.

## 성능이 향상된다

중복을 해결하는 편이 성능이 좋아지는 경우가 있다. 참조형에는 결합이 늘어날지 모르겠지만 그만큼 불필요한 무손실 분해는 억제된다. 갱신형에는 변칙이 발생하지 않는 것을 보증하는 제약을 걸 필요가 없고 응용프로그램도 변칙을 확인하기 위한 로직을 구현할 필요가 없다.

종종 비정규화를 하는 편이 성능이 좋아지는 경우가 있을지만, 그것은 특별한 경우라고 생각하는 편이 좋다.

> 일반적으로 중복이 있으면 DB의 부담은 비약적으로 상승한다.

# 요약

DB의 중복을 해소하지 않고 사용한다면 올바른 답을 얻을 수 없을 뿐만 아니라 본래의 성능을 끌어낼 수 없다
