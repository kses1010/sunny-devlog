---
title: '관계형 데이터베이스 실전 입문 - 11. 인덱스 설계 전략'
date: 2021-04-13
category: 'MySQL'
draft: false
---

# 1. 인덱스의 동작

인덱스는 검색을 빠르게 하려고 사용하는 것으로 이미지적으로 색인과 가깝다.

## RDB의 인덱스

RDB의 인덱스는 조금 더 똑똑한 방법으로 검색을 수행한다. B+트리를 구조를 가진다.

![02](https://user-images.githubusercontent.com/49144662/114875988-d2fbb300-9e38-11eb-8779-4c15ebb49abf.png)

B+트리는 데이터(인덱스의 값)가 저장된 **리프 노드**와 리프 노드까지의 경로 역할을 하는 **논리프 노드**로 구성된다. 경로의 출발점이 되는 노드는 **루트 노드**라고 한다. 논리프 노드에는 자식 노드가 보유하는 값 중에 최솟값이 저장돼 있다.

또한, 자식 노드에는 포인터(대부분 페이지 ID)가 저장되고 '어떤 경로를 검색하면 원하는 인덱스를 찾는가'를 알 수 있다.

→ B+트리의 검색은 루트 노드에서 어떤 리프 노드에 이르는 한 개의 경로만 검색하면 되므로 굉장히 효율적이다.

이 성질은 아무리 트리의 크기가 크거나 계층이 증가해도 변하지 않는다.

B+트리 인덱스에는 또 다른 재미있는 특징이 있다. 그것은 인덱스에 있는 항목의 수가 테이블의 행 수와 똑같다는 점이다. → 테이블의 행이 늘어나면 인덱스도 커진다.

## 인덱스의 왼쪽과 검색 범위

B+트리 인덱스는 등가 비교와 범위 검색에 사용할 수 있다. 등가 비교는 키의 값이 완전 일치하는 행을 찾는 것이다. 구체적으로 WHERE key = 123처럼 같음(=equal)으로 검색한다. 범위 검색은 부등호나 BETWEEN 절을 사용해 범위를 지정한다.

LIKE 절을 사용한 검색도 범위 검색이다. 예를 들어 WHERE key LIKE 'a%'를 검색하면 'a'이상 'b'미만의 범위에 있는 항목이 검색된다. 그러나 LIKE 절을 이용한 검색은 와일드카드를 배치하는 위치가 문제가 된다.

인덱스를 사용해 검색할 때 와일드카드의 지정은 전방 일치로 와일드카드가 구체적인 문자열 뒤에 놓아야 한다. 전방 일치여야 하는 이유는 B+트리의 물리적인 구조 때문이다.

B+트리는 왼쪽의 문자부터 순서대로 정렬돼 있다.

→ 왼쪽 이외의 문자로 인덱스를 검색하는 것은 구조적으로 불가능하다.

예를 들어 인덱스에서 WHERE name LIKE '가%' 라는 조건으로 검색하는 경우

맨 앞의 문자를 알고 있으므로 논리프 노드를 보면 보면 어떤 자식 노드에 해당하는 항목이 있는지 알 수 있다.

그런데 후방 일치는 WHERE name LIKE '%언어'와 같이 앞에 와일드카드를 사용하면 논리프 노드를 봐도 해당하는 항목이 어디에 있는지 알 수 없다. 값을 검색하려면 인덱스를 스캔해야 한다.

중간 일치도 마찬가지로 스캔하지 않으면 값을 알 수 없다. WHERE name LIKE '%언어%'

다중 컬럼 인덱스(복합 인덱스)도 문자열의 LIKE 검색과 마찬가지로 생각할 수 있다.

예를 들어 (last_name, middle_name, first_name)라고 정의된 인덱스가 있을 때 등가 비교라도 middle_index이나 first_name만 지정한다면 인덱스를 사용해 검색할 수 없다.

검색에 인덱스를 사용하려면 왼쪽 끝의 컬럼부터 순서대로 인덱스를 지정해야 한다.

**왼쪽 끝의 컬럼의 값이 지정되면 나머지 값은 몰라도 범위 검색에 인덱스를 효과적으로 활용할 수 있다.**

예를 들어 last_name, middle_name이 지정된 경우 전방 일치가 되고 first_name이 지정되지 않아도 인덱스를 사용한 범위 검색을 할 수 있다.

B+트리 인덱스는 왼쪽부터 순서대로 값을 지정하지 않으면 소용없다는 사실을 기억해야한다. 역설적이지만 다중 컬럼 인덱스를 작성하는 경우 컬럼의 순서는 매우 중요하다.

## 보조 인덱스의 갱신

대부분 기본키의 값은 한번 데이터가 삽입된 다음에는 수정하는 경우가 거의 없다. 그런데 보조 인덱스는 갱신이 자주 일어날 수 있다. 위 그림에서 '데이터 구조'라는 인덱스 항목을 '객체'로 바꾸는 처리를 생각해보자.

어떤 리프 노드에서 '데이터 구조'라는 항목을 지우고 다른 리프 노드에 '객체'라는 항목을 추가하는 것과 같다.

→ 보조 인덱스의 갱신은 삭제와 삽입의 처리를 수행하는 것과 비용적으로는 다르지 않다.

인덱스를 갱신하는 비용이 비싸다는 것을 기억하자. 인덱스가 늘어나면 늘어날수록 각종 갱신에 드는 오버헤드는 증가하게 된다.

# 2. 인덱스의 종류

인덱스는 B+트리이며, DB 설계 시에 B+인덱스 이외의 인덱스를 채용하는 경우는 드물다. 어떤 인덱스가 구현돼 있는지는 RDB 제품에 따라 다르다. 성능과 특징은 제품에 따라 다르므로 잘 사용하려면 제품을 숙지해야 한다.

## 해시 인덱스

해시 인덱스는 해시 테이블을 이용한 인덱스다. 해시값을 사용하므로 범위 검색에는 사용할 수 없다.

→ 해시 인덱스를 사용할 수 있는 것은 등가 비교에 의한 검색뿐이지만 검색 속도가 굉장히 빠르다는 특성이 있다.

사용처는 간단하여 등가 비교만 필요하고 범위 검색은 필요 없는 곳이며, RDB 제품이 지원한다면 사용을 검토하는 것도 좋다.

해시 인덱스는 범위 검색을 할 수 없어서 B+트리 인덱스와 같이 키의 왼쪽 끝을 지정해 사용하는 방법은 쓸 수 없다. 다중 컬럼 인덱스에도 해시 인덱스를 사용할 수 없다.

→ 키에 포함된 컬럼 모두에 대해서 등가 비교를 해야 하기 때문이다.

## 전문 검색 인덱스

B+트리 인덱스는 후방 일치나 중간 일치 등을 잘 처리하지 못한다. 그렇지만 실제 응용프로그램에서는
'xx 라는 단어를 포함한 행을 검색'와 같은 요구가 많다. 이런 요구에 응하는 것이 전문 검색(풀 텍스트)인덱스다.

전문 검색 인덱스는 전치 인덱스로 구현된다. 전치 인덱스란 행에 포함된 단어나 부분 문자열에 그행의 포인터(ROWID나 클러스터 인덱스의 경우에는 기본키)가 저장된 구조의 인덱스다. 물론 단어와 행은 1:1로 대응하지 않으므로 전치 인덱스의 항목 수는 테이블의 행 수보다 훨씬 많아진다.

### 전치 인덱스

| row_id | text                  |
| ------ | --------------------- |
| 1      | Relational Model      |
| 2      | Model View Controller |
| 3      | Materialized View     |

| word         | row_id |
| ------------ | ------ |
| Controller   | 2      |
| Materialized | 3      |
| Model        | 1      |
| Model        | 2      |
| Relational   | 1      |
| View         | 1      |
| View         | 2      |

스페이스로 단어가 구분되는 영어와 같은 언어라면 문장에서 단어를 구분하는 것은 간단하다. 하지만 띄어쓰기가 없는 언어에서는 전치 인덱스를 어떻게 작성해야 할까?

→ 각 행에 포함된 텍스트에서 단어를 어떻게 구분하는지가 과제다.

### 형태 분석법

형태 분석법은 사전을 사용해 문장의 문법을 분석하여 단어를 추출하는 기술이다. 단어를 딱 맞게 구분할 수 있으므로 낭비가 적고 인덱스의 사이즈가 작다는 장점이 있다. 그러나 작다고 해도 풀 텍스트 인덱스 중에서 작은 것이고 일반적인 B+인덱스와 비교하면 사이즈가 크다.

한편 형태 분석법의 단점으로 단어를 추출하는 능력의 한계다. 사전에 실려 있지 않은 단어는 추출할 수 없다. 또한, 구분 방법에 따라서 의미가 달라지는 경우가 있고 어느 쪽 단어를 추출해야 좋은지 알 수 없다는 문제가 있다.
ex) 아버지가방에들어가신다

양쪽의 단어에 전치 인덱스를 작성하면 양쪽 단어 모두 검색에 걸리므로 의도하지 않은 결과가 반환될지도 모른다. 또한, 분석의 정확도가 완전하지 않아서 구어체 등은 잘 처리하기 어려울 때도 있다.

이 이유로 검색에 실수가 발생하거나 사전의 품질에 따라 검색의 정확도가 다를 수 있다.

### N 그램

N 그램은 문장을 단순히 N 문자씩 부분 문자열로 나누는 방법이다. N에는 단어를 구분하는 문자의 크기를 지정한다. 2문자의 경우는 바이그램, 3문자의 경우는 트리그램이라고 한다.

형태 분석법처럼 단어 단위로 부분 문자열을 추출할 수 없으므로 의미가 없는 문자열로 잘릴 때가 있다. 예를 들어 바이그램이라면 '외국인참정권' 이라는 단어는 '외국', '국인', '인참', '참정', '정권'의 5개의 부분 문자열로 분해된다.이로 인해 의도하지 않은 '인참'이라는 단어도 검색이 된다.

N 그램은 포괄적이지만 검색 노이즈가 많다는 문제가 있다. 또한, 상당히 많은 문자열로 분해하므로 인덱스의 크기도 커진다.

## R트리 인덱스

R트리 인덱스는 지도상의 지점을 검색하는 데 사용하는 인덱스다. 공간(Spatial) 인덱스라고도 하며 한 개의 평면에서 어떤 도형이 다른 도형에 포함되는지? 중복되는지 등을 판별할 때 사용한다.

R트리의 R은 Rectangle의 앞 문자로 **최소 경계 사각형(Minimum Bounding Rectangle, MBR)**이라는 개념을 사용해 인덱스를 구성한다. MBR이란 어떤 도형을 둘러 싼 최소의 사각형(직사각형)이다.

![03](https://user-images.githubusercontent.com/49144662/114875993-d3944980-9e38-11eb-9b83-0637f9e9000a.png)

R트리를 구성하려면 먼저 요소가되는 도형 전부를 MBR로 바꾼다. 다음은 주변에 있는 임의의 사각형을 모두 포함하는 사각형을 새롭게 작성한다. 이 사각형은 내부에 포함된 사각형의 부모 노드가 된다.

자식 노드가 되는 MBR이 많은 경우 거리가 떨어져 있는 자식 노드 MBR을 각각 다른 부모 노드 MBR에 포함시킨다. 이렇게 하여 모든 자식 노드 MBR이 어딘가의 부모 노드 MBR에 포함되게 한다. 부모 노드에 포함된 자식 노드는 적당한 수가 되도록 조정한다.

부모 노드의 수가 많을 때는 상위의 부모 노드가 되는 MBR을 만들도록 트리를 구성한다. 최종적으로 모든 사각형을 내부에 포함하는 MBR이 루트 노드가 된다.

![04](https://user-images.githubusercontent.com/49144662/114875995-d3944980-9e38-11eb-9ad5-950c1e7519b4.png)

## 함수 인덱스

일반적으로 WHERE year(date_col) = 2013과 같이 컬럼 값에 함수를 사용한 WHERE 절은 B+트리 인덱스를 사용한 검색을 할 수 없다. 함수를 적용한 쿼리에도 인덱스를 사용하고 싶을 때 편리한 것이 함수 인덱스다.

일반 인덱스는 테이블 내의 컬럼의 값 자체에 인덱스를 생성하지만, 함수 인덱스는 year(date_col)과 같이 함수를 평가한 다음의 값에 대해 B+트리 인덱스를 작성한다.

→ 함수를 평가한 값을 사용해 검색할 수 있으므로 WHERE year(date_col) = 2013이라는 검색 조건도 인덱스를 사용하게 된다.

## 비트맵 인덱스

OLAP(Online Analytical Processing, 온라인 분석 처리) 등에 사용되는 인덱스다.

B+트리 인덱스는 행별로 대응하는 인덱스 항목이 생성되고 각 항목에 행의 포인터(또는 ROWID)가 저장된다. 비트맵 인덱스는 이것과는 다르다.

컬럼별로 여러 개의 비트맵이 생성된다. 컬럼별로 여러 개의 비트맵이 생성된다. 비트 중에 1이 있으면 그 행의 값이 비트맵의 값이 되어 있다는 것을 나타낸다.

### 비트맵 인덱스

| name   | grade |
| ------ | ----- |
| 이유정 | 1     |
| 정은오 | 2     |
| 한혜린 | 4     |
| 홍윤서 | 3     |
| 서정은 | 2     |
| 정승효 | 3     |
| 백하은 | 4     |

| value | bitmap  |
| ----- | ------- |
| 1     | 1000000 |
| 2     | 0100100 |
| 3     | 0001010 |
| 4     | 0010001 |

grade가 2인 비트맵은 2번째와 5번째의 비트가 1이므로 2번째와 5번째 행의 학생이 2학년이라는 것을 알 수 있다. 각 비트맵은 B+트리에 비해 훨씬 크기가 작아지므로 인덱스 스캔이 매우 빠르다.

그러나 비트맵의 갱신에 시간이 걸리므로 OLTP(Online Transaction Processing, 온라인 트랜잭션 처리)와 같이 갱신이 많은 작업에는 적합하지 않다. 또한, 컬럼이 얻는 값이 많을 때도 필요한 비트맵의 수가 증가하므로 원하는 성능이 나오지 않을 수 있다.

## 클러스터 인덱스

테이블 자체가 인덱스 되어 있는 것을 **클러스터 인덱스** 또는 **인덱스 구성 테이블(Index Organized Table)**이라고 한다.

클러스터 인덱스는 기본키의 리프 노드에 키 값뿐 아니라 다른 컬럼의 값도 같이 저장한다. 따라서 기본키에 값을 검색하면 같은 페이지에 데이터가 들어있어서 자체에 엑세스 할 필요가 없으므로 오버헤드가 줄어든다.

한편 보조 인덱스를 사용한 검색, 즉 일반적인 테이블은 행의 포인터(페이지 번호, 오프셋 등)로 엑세스하면 되므로 1페이지만 엑세스하면 되지만, 클러스터 인덱스는 트리를 거치지 않으면 안 되므로 약간 오버헤드가 커진다. 기본키에 의한 엑세스가 메인이라면(RDB 제품이 클러스터 인덱스를 지원하는 경우) 클러스터 인덱스는 매우 좋은 선택지가 될 것이다.

MySQL에서 가장 많이 사용되는 InnoDB 스토리지 엔진은 모든 테이블이 클러스터 인덱스로 작성되며 그 외의 구조를 선택할 수 없다.

# 3. 파티셔닝

## 파티셔닝이란

인덱스는 임의의 키 값에 따라서 행 데이터의 위치를 식별하는 데 사용하는 기능이다. 한편 파티셔닝은 테이블을 여러 개의 파티션으로 분할하고 키의 값에 따라 어떤 파티션에 속하는 행인지 배분한다.

파티션은 각각 같은 구조를 가진 테이블이며 인덱스도 파티션마다 존재한다.

![05](https://user-images.githubusercontent.com/49144662/114875997-d42ce000-9e38-11eb-8264-275dd86a26fb.png)

세 개의 파티션이 정의돼 있고 날짜에 따라 파티션이 나뉘어져 있을 것이다. 어떤 파티션에 속할 것인지 정하는 키(파티션 키)를 검색 조건으로 지정하면 해당 파티션만 검색하면 되므로 검색 효율이 향상된다.

이처럼 검색 대상 파티션을 좁혀가는 동작을 파티션 프루닝(Patition Pruning)이라고 한다. 프루닝은 가지치기를 의미한다.

파티션 프루닝은 범위 검색에도 유용하다. 예를 들어

```sql
key BETWEEN '2013-10-01' AND '2014-02-01';
```

조건을 달면 해당하는 파티션만 검색 대상이 될 것이다.

또한 각 파티션에는 각 인덱스가 있으므로 파티션 프루닝을 한 다음에 대상의 인덱스를 사용한 검색을 수행할 수 있다. 파티셔닝은 어떤 방법으로 분할할 것인지에 따라 세가지 방법으로 나뉠수 있다.

- 레인지: 키의 범위가 어디에 있는지에 따라 파티션을 정한다. (날짜별로 데이터 나눌 때)
- 리스트: 사전에 부여된 키의 값에 따라 파티션을 정한다. (컬럼의 값이 아주 적을 때)
- 해시: 키로 계산한 해시값의 나머지 연산 등에 따라 파티션을 정한다. 키가 정수일 때 키 파티셔닝이라 한다.

## 파티셔닝이 적합한 경우

파티셔닝을 사용할지의 가장 큰 판단 기준은 파티션 프루닝을 할 수 있는 검색 조건을 포함한 쿼리의 실행빈도다. 파티션 프루닝이 유효한 쿼리가 대부분일 때는 파티셔닝에 의한 처리 전체의 효율화가 기대된다.

그러나 파티셔닝이 적합하다고 생각될 때도 인덱스를 붙이는 것만으로 충분할 때가 대부분이다. 파티셔닝을 쓰는 편이 명확하게 성능이 향상될 때는 키의 카디널리티가 낮을 때

→ 검색으로 대량의 데이터가 패치 되는 경우다.

카디널리티가 낮은 인덱스를 사용한 검색은 대량의 행이 일치하므로 인덱스 페이지와 데이터를 저장한 페이지를 몇 번이고 왕복하게 되어 효율이 높지 않다. 파티션이라면 한 개의 파티션을 스캔하면 되므로 훨씬 비용이 낮아진다.

그러나 OLTP 형 응용프로그램에서 이와 같은 대량의 행을 패치해야 하는 경우는 드물어서 파티셔닝을 사용하는 경우도 적을 것이다. 만약 OLTP 형 응용프로그램에서 스캔을 여러 번 수행해야 하면 테이블의 정규화가 충분하지 않은 등의 문제가 있을 수 있으므로 먼저 이를 해결해야 한다.

최신 데이터를 항상 참조할 때는 파티셔닝이 유용한 경우가 있다. 새로운 데이터를 차례로 추가하는 응용프로그램이며 과거의 데이터에 그다지 접근하지 않을 때는 레인지 파티셔닝을 수행해 엑세스되는 대상의 데이터 국소성을 최대한 활용할 수 있다. 파티셔닝을 사용하면 파티셔닝별로 인덱스가 작성되기 때문이다.

**엑세스의 국소성이 있다면 캐시의 효율이 매우 향상된다.**

→ 대부분 검색이 메모리만으로 처리되고 디스크 I/O를 줄여서 성능 향상을 실현할 수 있다.

## 파티셔닝과 고유성 제약

인덱스가 파티셔닝별로 작성돼 있다. 이와 같은 인덱스를 **로컬 인덱스** 라고 한다. 인덱스가 파티션별로 존재하므로 가장 곤란한 것이 고유성 제약이다.

고유성 제약은 인덱스를 사용해 구현한다.로컬 인덱스에서 고유성을 보장하는 범위는 그 인덱스가 속한 파티셔닝으로 제한된다.

→ 테이블 전체의 고유성을 보장할 수 없다.

이 문제 때문에 많은 DB에서 파티셔닝된 테이블에 대해 기본키나 유니크 인덱스를 반드시 파티션 키에 포함해야 한다는 제한이 있다.

파티션 키가 기본키나 유니크 인덱스를 구성하는 컬럼에 포함돼 있으면 파티션별로 파티션 키의 값이 다르므로 다른 파티션에는 같은 값의 키가 포함될 가능성은 없을 것이다.

이 방법을 사용하면 고유성 제약의 기능은 바르게 동작할 것이다.

그러나 파티션 키를 포함한 키는 그 테이블의 설계상 정말로 유일하지 않으면 안된다. 이 경우 파티션 키를 유니크 인덱스에 넣어도 아무런 해결책이 되지 않는다.

**파티셔닝을 수행해 고유성을 보장할 수 없다면 파티셔닝을 하지 않는 것이 좋다.**

파티셔닝을 수행하고 있을 때도 정상적으로 고유성 제약을 사용하려면 개별 파티션이 아닌 테이블 전체를 대상으로 하는 인덱스가 필요하다. → **글로벌 인덱스** 라고 한다.

글로벌 인덱스를 사용할 수 있는 제품이라면 파티셔닝에 의한 고유성 제약이 손상되지 않고 큰 문제 없이 파티셔닝을 사용할 수 있다. 그러나 이런 글로벌 인덱스는 모든 파티션의 데이터가 포함되므로 이 인덱스를 사용한 검색은 파티션 프루닝을 할 수 없다. 또한, 어느 파티션의 행을 갱신해도 반드시 글로벌 인덱스도 갱신해야 한다. 엑세스의 국소성도 활용 할 수 없다.

→ 파티셔닝의 의미가 줄어들 가능성이 있다.

## 파티셔닝에 관한 일반적인 오해

파티셔닝에 관한 가장 일반적인 오해는 파티셔닝만으로 엑세스가 빨라진다는 생각이다.

파티셔닝은 기본적으로 파티션 프루닝을 잘 사용하지 않으면 의미가 없다. 파티셔닝을 수행해 데이터를 저장할 디스크를 나누거나 처리를 파티셔닝별로 병행할 수 있는 제품도 있지만, 이런 제품이 아닌 이상 무조건 파티셔닝을 해서는 의미가 없다.

오히려 파티셔닝 때문에 처리가 늦어질 가능성이 크다. 파티션 프루닝을 사용하는 것으로 고속화가 실현되지만 파티션 프루닝을 사용할 수 없을 때는 모든 파티션에 대한 검색을 실행해야 한다.

인덱스를 사용한 검색도 마찬가지다. 파티셔닝을 하지 않는 경우, 범위 검색은 인덱스를 한 번 검색하면 충분하다. 그러나 예를 들어 100개의 파티션이 있는 테이블을 파티션 프루닝해서 사용할 수 없다면 로컬 인덱스를 100번 검색해야 한다.

이와 같은 오버헤드는 패치된 행 수가 적을 때 문제가 된다. 로컬 인덱스를 검색하더라도 빠질(해당 데이터가 없는)가능성이 커지기 때문이다.

# 4. 관계형 모델과 인덱스

인덱스는 논리적인 설계가 아니고 물리적인 설계의 이야기(구현)다.

논리적인 설계는 어떤 값이 테이블에 저장되는가 뿐이다. 쿼리의 결과에 따라서 어떤 값을 얻을 수 있는지, 즉 테이블에 포함된 각 값이 무엇인지가 테이블의 논리적인 설계다.

한편 인덱스는 실행 계획과 관련된 오브젝트다. 인덱스를 사용한 엑세스는 빠르지만, 인데스ㅡ이 유무에 따라 쿼리 결과가 달라지지는 않는다. 인덱스는 물리적 설계(구현)의 일부다.

원래는

> 쿼리가 정해지고 나서(원하는 데이터를 결정하고 나서) 고속화하기 위한 인덱스를 설계한다.

이지만 실무에서는 가끔

> 인덱스를 정한 다음에 이 인덱스를 사용한 쿼리를 어떻게 작성할까

라는 모습을 볼 수 있다. 대부분 SQL 숙련자가 자주 하는 실수 중 하나다.

## 인덱스는 관계형 모델의 일부가 아니다

인덱스는 관계형 모델 외부 세계의 개념이다.

기본적으로 먼저 DB의 논리 설계를 하고 그 다음에 쿼리를 작성하고, 인덱스에 대해 생각하는 것이 좋다. DB 설계나 쿼리로 무엇을 얻을 것인지는 논리설계이며 인덱스는 물리 설계이기 때문이다.

DB를 설계할 때 후보키는 알 수 있으므로 그것만은 처음부터 기본키로 정의해도 된다.

직감으로 '이 컬럼은 검색할 일이 많을 거야' 라면 인덱스를 작성해도 괜찮으나 결정한 인덱스에 얽매여 쿼리를 작성하는 일이 없도록 해야한다.

특히 인덱스는 리팩터링도 간단하므로 처음에 정한 인덱스에 집착할 필요는 없다.

> 인덱스의 정의는 유연하게 재검토해도 상관없다.

## 정규화와 인덱스

인덱스를 효과적으로 사용하려면 DB 설계가 매우 중요하다. 특히 정규화는 제대로 구현해야 한다.

### 컬럼수가 줄어든다

정규화되지 않은 테이블은 컬럼이 매우 많다. 그 결과 필요한 인덱스도 늘어나게 된다. 인덱스가 많으면 많을수록 갱신 성능이 나빠지고 필요한 디스크 공간도 많아지므로 인덱스 관점에서 보더라도 정규화는 필요하다.

정규화된 테이블에는 적어도 한 개의 후보키가 존재하므로 그 테이블에는 명시적으로 기본키를 작성할 수 있다. SQL에서는 키본키가 없는 테이블도 허용하지만, 중복하는 행을 가지게 되므로 바람직하지 않다.

중복하는 행을 가진 테이블은 관계형 모델을 기반으로 한 연산을 적용할 수 없기 때문이다.

### 문제아 NULL

정규화하지 않으면 문제가 되는 것이 NULL의 존재다.

1NF 되어 있지 않은 테이블에는 NULL이 나올 가능성이 있다.

NULL은 관계형 모델에서는 천적일 뿐만아니라 인덱스와도 궁합이 좋지 않다. SQL에서는 NULL 끼리의 비교는 할 수 없고, 인덱스의 구현에도 NULL과 같은 값을 가진 항목은 인덱스의 맨 앞 또는 맨 뒤에 모아서 놓이게 된다.

맨 앞과 맨 뒤 중 어느 쪽에 배치할지는 제품의 구현(정렬 순서)에 달려있다. NULL의 값을 검색하는 것은 물리적으로 연속된 영역을 스캔하는 것에 불과하다. 그 컬럼의 값 중에 어느 정도의 비율이 NULL인지에 따라, NULL의 비율이 높을 때는 그 인덱스를 사용한 검색이 매우 비효율적이게 된다.

컬럼의 값이 NULL일 때의 문제는 비교가 3VL이 된다는 것이다. 예를 들어 NOT NULL이 아닌 age라는 컬럼이 있고 20세 미만의 모든 사람을 찾는 경우 age가 NULL인 사람도 포함할 것인지 아닌지에 따라 검색이 달라진다.

NULL이 포함돼 있으면 인덱스의 효과를 최대한 활용할 수 없다.

# 5. 지령: 최적의 인덱스를 찾아라!

인덱스 설계에 있어 가장 중요한 점은 많은 컬럼중에서 어떤 컬럼에 인덱스를 생성하는 것이 좋은가이다.

## 필요한 인덱스

인덱스의 검색을 빠르게 하려면 검색조건에 적합한 인덱스가 필요하다. 조건에 맞는 인덱스가 없으면 테이블을 스캔해 검색하는 방법밖에 없다. 테이블의 크기가 크면 스캔에 드는 비용도 커진다. 크기가 큰 테이블은 인덱스에 의한 검색 속도의 개선 효과가 크다.

그러나 검색이 빨라진다고 해서 어떤 것이든지 인덱스를 작성하면 안된다. 인데스가 늘어날수록 테이블을 갱신할 때의 오버헤드도 커지고 필요한 디스크 공간도 늘어난다. 데이터 사이즈가 증가해 버퍼풀의 히트율도 낮아질 것이다. 검색에 필요한 컬럼이 포함된 인덱스의 조합 중에 가장 효율이 높은 것을 찾는 것이 중요하다.

## 인덱스의 엑세스 특성

인덱스가 필요한 이유는 테이블 스캔을 피하기 위해서지만 테이블 스캔이 실행 계획상 반드시 나쁜 선택이라고는 할 수 없다. 다음과 같은 상황이면 테이블 스캔을 하는 편이 시스템 전체의 처리량이 많아진다.

- 어떤 인덱스를 사용한 쿼리의 실행 빈도가 매우 낮음(1일 1회)
- 테이블의 사이즈가 매우 작을 때(100행 등)
- 검색 결과가 매우 많은 행을 가져올 때

마지막의 경우는 뒤에 설명하는 커버링 인덱스가 있다면 빠르다. 그러나 그렇지 않을 때는 실행 과정에서 인덱스와 테이블을 오가며 엑세스하게 되므로 테이블 자체에 엑세스하기 위해 랜덤 엑세스가 발생하게 되고, 인덱스를 사용하는 것이 오히려 느릴 때가 많다.

## 인덱스가 사용되는 구문

어떤 컬럼의 인덱스를 작성해야 할지 고민하기 전에 어떤 SQL의 구문에 대해 인덱스를 사용할 수 있는지 알고 있어야 한다. 지금은 SELECT를 대상으로 설명하지만, UPDATE, DELETE, INSERT 등의 구문도 방법은 같다.

### WHERE 절

가장 기본적인 것은 WHERE 절이다. WHERE 절에 컬럼이 등호나 부등호로 비교하고 있으면 인덱스로 빨라질 가능성이 있다. 같은 테이블의 여러 개의 컬럼이 WHERE 절로 지정돼 있다면 해당 컬럼에 대해서 다중 컬럼 인덱스를 사용하면 가장 좋은 성능을 얻을 가능성이 커질 것이다.

다중 컬럼 인덱스를 사용할 때 주의해야 할 점은 검색 조건이 전방 일치해야 한다는 점이다. 특히 등호나 부등호가 섞여 있을 때는 주의가 필요하다.

```sql
## 다중 컬럼 인덱스가 적합한 범위 검색
SELECT * FROM t WHERE col1 > 100 AND col2 = 'abc';
```

이 쿼리를 인덱스를 사용해 빠르게 처리하려면 (col2, col1)의 순서로 인덱스를 작성해야 한다.

(col2, col1)의 순서로 인덱스를 작성해도 col1>100의 부분만 B+트리로 해결할 수 있기 때문이다. 이 부등식은 한쪽이 열려있어서 많은 행이 검색 결과로 나올 가능성이 있다.

이에 따라 앞에서 말한 것과 같이 인덱스를 사용하는 장점이 없을 가능성이 있다.

반대로 (col2, col1)의 순서로 작성된 인덱스라면 양쪽의 검색 조건을 인덱스로 해결할 수 있다.

예시) ('abc', 100) < (col2, col1) < ('abd', col1의 최솟값)이라는 범위를 인덱스에서 검색 가능

### JOIN

JOIN에서 중요한 것은 결합하는 테이블의 순서다. 내부 테이블(결합하는 쪽의 테이블)에 대한 엑세스는 인덱스가 사용된다. 이때 중요한 것이 JOIN의 결합 조건 이외의 검색 조건이다.

```sql
SELECT * FROM t1
JOIN t2 ON t1.col1 = t2.col2
WHERE t1.col3 = 100 AND t2.col4 = 'abc';
```

이 쿼리에서 구동 테이블이 t1, 내부 테이블이 t2라는 순서의 실행 계획이 선택했을 때 t2 테이블에서 (col2, col4)라는 다중 컬럼 인덱스가 있다면 col2에 인덱스보다 엑세스가 효과적일 수 있다.

col2는 JOIN의 ON절에, col4는 WHERE 절에 각각 위치하고 있다. 이 것은 t2.col2가 기본키거나 유닉스 인덱스가 아닐 때 특히 효과적이다.

기본키나 유니크 인덱스에서는 등가 비교로 테이블에서 선택된 데이터는 최대 1줄이다.

→ 다른 조건이 있다고 해도 효율은 크게 차이 없다. 역시 내부 테이블에 대한 엑세스가 기본키나 유닉스 인덱스를 사용해 수행하는 것이 효과적이다.

그러나 WHERE 의 검색 조건으로 좁혀진 정도(좁혀진 후의 행 수)에 따라 옵티마이저는 JOIN을 수행하는 순서

→ 어느 쪽이 구동 테이블이 될 것인지를 바꿀 수 있다.

이 경우는 t2.col2 컬럼에 유니크가 아닌 보조 인덱스가 있을지도 모른다. t2.col2가 유니크가 아니라면 JOIN의 t1.col1 = t2.col2라는 조건에 해당하는 행이 여러 개 나온다. 이 쿼리를 해결하려면 JOIN의 조건에 따라서 t2에서 행을 선택한 다음에 WHERE 절의 t2.col4 = 'abc'라는 조건으로 좁혀서 검색해야 한다.

만약 t2 테이블에 (col2, col4)라는 다중 컬럼 인덱스가 있다면 테이블에서 데이터를 선택한 시점에 JOIN의 t1.col1 = t2.col2, WHERE 절의 t2.col4 = 'abc'이라는 두 조건 모두가 적용되게 되고 테이블에서 행을 선택한 다음에 WHERE 절의 조건으로 대상을 좁혀나가는 불필요한 작업이 없어지게 된다.

JOIN에서 다중 컬럼 인덱스가 유용할 때가 있다는 것은 의외의 맹점이므로 명심해야 한다.

### 상관 서브쿼리

JOIN의 경우와 마찬가지로 상관 서브쿼리도 WHERE 절의 조건과 서브쿼리 양쪽 모두를 고려한 인덱스 설계가 요구된다.

```sql
SELECT * FROM t1 WHERE t1.col1 IN (
	SELECT col2 FROM t2 WHERE col3 = 100)
AND t1.col4 = 'abc';
```

서브쿼리의 WEHRE 절에 지정된 col3와 select list에 있는 col2가 인덱스의 대상이 된다. col2는 t1.col1과 비교되기 때문이다.

→ t2 테이블에 (col2, col3) 또는 (col3, col2)라는 인덱스가 있으면 이 상관 서브쿼리는 인덱스를 사용해 해결할 수 있다.

참고로 IN 서브쿼리는 EXISTS 서브쿼리로 간단하게 재작성할 수 있다.

```sql
SELECT * FROM t1 WHERE EXISTS (
	SELECT col2 FROM t2 WHERE col3 = 100
		AND t2.col2 = t1.col1)
AND t1.col4 = 'abc';
```

이렇게 보면 t2 테이블에 다중 컬럼 인덱스가 필요한 이유를 알 수 있다.

### 정렬

B+트리는 키의 순서로 정렬되어 저장돼 있으므로 인덱스의 순서로 항목을 읽으면 정렬의 고속화에 도움이 된다. WHERE 절에서 전방일치하도록 조건을 지정하면 범위 검색과 정렬 모두를 한 번에 인덱스로 해결할 수 있다.

```sql
SELECT * FROM t WHERE name LIKE '아%'
ORDER BY name;
```

이 쿼리는 (name)이라는 인덱스가 있으면 name에 대한 검색이 전방일치이므로 인덱스를 사용한 범위 검색이 가능하다. 정렬은 이 범위의 앞에서부터 순서대로 인덱스를 따라가면 되므로 정렬에도 인덱스를 사용할 수 있다. 범위가 명확하다면 정렬의 순서는 ASC여도 되고, DESC여도 된다.

다중 컬럼 인덱스도 마찬가지로 범위가 명확하다는 것과 그 범위의 인덱스가 정렬키가 순서로 나열한다는 두 가지 조건을 만족한다면 인덱스로 정렬을 해결할 수 있다.

예를 들어 (col1, col2, col3)라는 인덱스가 있을 때 WHERE col1 = 100 AND col2 = 'abc' ORDER BY col3라는 조건이 있다면 정렬을 인덱스로 해결할 수 있다. 그러나 WHERE col1 = 100 ORDER BY col3와 같이 col2가 빠져있을 때는 인덱스를 사용하면 정렬을 할 수 없다. WHERE 절에 지정된 범위 인덱스 항목은 정렬키의 col3가 아니고, col2가 앞쪽에 오는 순서로 정렬돼 있기 때문이다.

### 정렬을 인덱스로 해결한 예시

| col1 | col2 | col3 |
| ---- | ---- | ---- |
| 99   | abc  | 1    |
| 99   | xyz  | 1    |
| 100  | aaa  | 1    |
| 100  | aaa  | 2    |
| 100  | abc  | 1    |
| 100  | abc  | 2    |
| 100  | abc  | 3    |
| 100  | abc  | 4    |
| 100  | xyz  | 1    |
| 101  | abc  | 1    |
| 101  | abc  | 2    |

WHERE col1 = 100 AND col2 = 'abc' ORDER BY col3라는 조건에 해당하는 4줄이다. 정렬의 순서가 ASC라면 네 개의 행을 위에서부터 순서대로, DESC라면 아래에서부터 순서대로 읽으면 된다는 것을 알 수 있다.

한편 WHERE col1 = 100 ORDER BY col3라는 조건에 해당하는 행은 7줄이다. 그러나 그 7줄에 대해 col3의 순서로 정렬돼 있지 않기 때문에 인덱스 순서대로 데이터를 읽어도 정렬되지 않는다. 이럴 땐 테이블에서 데이터를 읽은 다음에 다시 정렬을 수행해야 한다.

### 커버링 인덱스

쿼리의 실행에 필요한 컬럼이 인덱스에 모두 포함돼 있다면 테이블 자체에 엑세스하지 않고 인덱스에만 엑세스하며 쿼리를 해결할 수 있다. → **커버링 인덱스** 라고 부른다.

커버링 인덱스를 사용한 쿼리(인덱스 온리 스캔)는 매우 빠르다.

→ 인덱스의 사이즈가 커질 것을 각오하고 일부러 여분의 컬럼을 인덱스에 넣어서 자주 실행하는 쿼리가 인덱스만 검색되게 하는 기술이 있다.

디스크 공간과 갱신 성능은 조금 떨어지지만, 커버링 인덱스의 효과는 크다. 높은 빈도로 실행되는 쿼리가 있다면 적극적으로 사용해보자.

### OR과 인덱스

다중 컬럼 인덱스로 효과가 있는 것은 기본적으로 검색 조건이 AND로 결합한 경우뿐이다.

(col1, col2)이라는 인덱스는 col1 = val1 AND col2 = val2와 같이 AND로 조건을 결합했을 때만 효과가 있다.

한편 col1 = val1 OR col2 = val2 등 OR로 결합했을 때는 다중 컬럼 인덱스로는 해결할 수 없다. OR을 해결할 수 있는 것은 OR로 결합한 각 조건에 인덱스가 있을 때다.

col1 = val1 OR col2 = val2라는 검색조건은 (col1)과 (col2)라는 두 개의 인덱스가 있어야한다. 이 실행 계획은 각 인덱스를 사용해 구한 ROWID의 집합에 대해서 합집합(UNION)을 실행해 최종적으로 필요한 행이 어느 것인지 판단한다.

→ **인덱스 머지** 라고 한다.

인덱스 머지가 구현돼 있는지는 제품에 달려있다. 사용할 수 없는 제품일 경우 WHERE 절의 검색 조건에 OR를 사용해 결합하는 대신 UNION DISTINCT를 사용해 두 개의 SELECT를 연결하는 방법을 선택하면 좋다.

OR에 대해서 인덱스 머지가 유용할 때 AND에도 응용이 될까?

col1 = val1 AND col2 = val2라는 검색 조건에 대해 (col1)과 (col2)라는 두 개의 인덱스가 있을 때 양쪽의 인덱스를 사용해 해결할 수 없는지다. 그러나 AND를 두 개의 인덱스로 해결하는 것은 효율적이지 않다.

한쪽의 인덱스, col1을 사용해 행 데이터를 구한 다음에 행 데이터에 포함된 컬럼의 값에 대해서 다른 검색 조건인
(col2 = val2)를 적용하는 것이 효율적일 때가 많기 때문이다.

## 최적의 인덱스를 찾기 위한 전략

지금까지는 주로 '각 인덱스가 어떻게 사용될까? 라는 점을 중점으로 설명했다. 이 다음은 최적의 인덱스를 찾는데 필요한 전략을 소개한다.

### 인덱스 ≠ 후보키

릴레이션의 후보키만 인덱스가 되는 것은 아니다. 대부분 인덱스는 WHERE 절의 검색 조건과 같은 **제약을 빠르게 실행하는 데 필요** 하다. 제약은 보조키 = 기본키에 대한 등가비교를 수행하는 것이 아니면 인덱스의 구조상 범위 검색이 된다. 이것은 인덱스의 값이 중복될 가능성이 있기 때문이다.

인덱슬르 잘 설계하기 위한 최대의 목적은

> 어떻게 하면 범위 검색에 필요한 인덱스를 설계하는가

범위 검색을 위한 보조 인덱스가 제한이라면 **커버링 인덱스는 사용(프로젝션)을 빠르게 하기 위한 것**이다. 인덱스에 포함된 컬럼을 적절하게 설정하면 커버링 인덱스에 의해 제한과 사영을 동시에 빨라지게 할 수 있다.

### 컬럼의 정렬 순서

B+트리 인덱스는 컬럼의 순서가 중요하지만, 한편으로 관계형 모델에서 후보키나 슈퍼키는 집합이므로 요소 사이의 순서가 없다.

→ 어떤 순서로 컬럼을 정렬할 것인지가 중요하다.

어떤 순서로 컬럼을 정렬해야 할지는 쿼리에 달려있다. 범위 검색이나 정렬이 있을 때는 그러한 검색 조건이 전방일치가 되게 컬럼을 배치해야 한다. 같은 컬럼으로 조합한 인덱스라도 순서가 다른 것이 필요할 때도 있다.

**전체의 성능을 향상시킬 수 있다면 같은 컬럼의 조합으로 순서만 다른 인덱스를 작성해야 한다.**

### 카디널리티

어떤 컬럼을 인덱스에 포함시킬 것인가를 판단하는 데 있어 포인트가 되는 것이 **카디널리티(집합의 밀도)**이다. 카디널리티란 집합에 포함된 요소의 개수다. 집합의 요소수이므로 중복은 배제된다.

예를 들어 WHERE 절에서 col1, col2, col3라는 세 개의 컬럼이 사용되는 경우 col1과 col2의 카디널리티가 충분히 높다면 col3은 굳이 인덱스에 넣지 않아도 된다.

충분히 빠르다면 일부러 모든 컬럼을 포함한 인덱스를 작성할 필요는 없다.

**여러 가지 검색 조건에 대해 최대공약수와 같이 카디널리티가 높은 컬럼을 인덱스로 만드는 기술도 중요하다.**

### 최적의 조합을 찾아라

SQL 구문상 기본적으로 인덱스가 사용될 가능성이 있는 곳에 대해서는 모두 인덱스 작성을 검토할 필요가 있다.

**→ 우선 필요할 것 같은 인덱스를 찾아낼 수 있는지가 중요한 포인트가 된다.**

다음은 필요한 인덱스만 남긴다. 인덱스가 늘어나면 갱신에 오버헤드가 커지고, 많을수록 좋지 않다.

→ 필요한 인덱스를 판별하는 것이 중요하다.

**어떤 컬럼을 인덱스에 포함해야 하는지에 대한 판단은 각 컬럼의 카디널리티, 테이블의 사이즈 또는 쿼리의 실행빈도 등을 검토해 실시한다.**

인덱스를 설계할 때의 어려운 점은 데이터의 품질(사이즈나 카디널리티)이나 쿼리의 실행빈도에 따라 결정이 좌우된다는 점이다. 테이블 설계, DB 설계 또는 쿼리만을 봐서는 최적의 인덱스 조합이 어떤 것인지 판단할 수 없다.

또한 한 개의 쿼리가 빨라도 그 때문에 갱신이 늦어져 전체적으로 성능이 저하된다면 주객이 전도된 꼴이 된다. 인덱스 튜닝을 할 때 전체적인 성능이 좋아지게 균형을 잡아야 한다.

각 쿼리를 따로따로 봐서는 전체적인 성능이 어떤지 판단할 수 없다.

→ 벤치마크를 사용한다. 벤치마크를 수행해 응용프로그램 전체의 성능이 개선되도록 균형을 조정해야 한다.

**인덱스의 설계는 조합 최적화의 문제**라고 할 수 있다.

### 어려운 작업에 맞서기

여기서 소개한 기본을 제대로 습득하고 인덱스를 설계해보자.

- B+트리 인덱스의 기능
- B+트리 이외의 인덱스의 종류
- 인덱스 설계에 앞서 제대로 정규화를 실시
- DB 설계와 쿼리가 정해지고 나서 인덱스를 결정
- SQL의 어떤 부분에 인덱스가 필요한가
- 컬럼이 정렬 순서만 다른 인덱스가 있어야 하는가
- 최대공약수의 카디널리티가 높은 컬럼만 포함한 인덱스를 작성
- 필요한 인덱스의 조합을 늘어놓는 방법

## 이러한 인덱스 설계는 쓰레기통 행!

- 모든 컬럼에 인덱스가 있다.
- 인덱스에 포함된 컬럼이 한 개 밖에 없다.(멀티 컬럼 인덱스가 아니다)
- 여러 개의 다중 컬럼 인덱스에 같은 컬럼의 조합이 있고 모두 같은 정렬로 되어 있다.
- 0 또는 1 밖에 값이 없는 플래그와 같은 컬럼에 인덱스가 있다.

# 요약

인덱스 설계에서 가장 중요한 것은 기반이 되는 DB의 논리설계(DB설계와 쿼리)가 제대로 되어 있어야 한다.
