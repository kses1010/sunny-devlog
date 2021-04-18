---
title: '관계형 데이터베이스 실전 입문 - 12-1. 웹 응용프로그램을 위한 데이터 구조'
date: 2021-04-18
category: 'MySQL'
draft: false
---

> 성능을 얻기 위해 관계형 모델을 유지하면서 물리적인 한계에 대응할 수 있을까?

# 1. 캐시라는 개념

## 장점/단점

### 장점

캐시의 본질은 비용이 많이 드는 A라는 처리를 같은 효과가 있으며 비용이 낮은 A라는 별도의 처리로 대체하는 것이다. 높은 비용의 처리를 생략함으로써 전체적인 성능을 향상시킨다.

캐시의 효과는 절대적이다. 때로는 캐시 덕분에 수백 배, 수천배나 성능이 향상되는 경우도 많다.

### 단점

캐시는 성능 향상의 대가로 시스템의 새로운 복잡성을 만들어낸다. 캐시라는 새로운 레이어가 추가됨에 따라 시스템의 동작은 더욱 복잡해진다.

하드웨어상의 캐시, 소프트웨어가 관리하는 캐시든 캐시를 도입함으로써 시스템의 개발 및 유지 보수 비용이 증가한다.

→ 캐시를 사용하는 것은 그에 알맞는 성능 향상을 기대할 수 있을 때 한다.

## DB 응용프로그램에서의 캐시

캐시의 기본적인 발상은 매우 간단하다.

> 대체할 수 있는 낮은 비용의 처리로 지속해서 발생하는 높은 비용의 처리를 가능한 한 실행하지 않게 하기 위한 것.

DB 응용프로그램에서 캐시는 DB 상의 데이터가 대상이 된다. 단순하게 버퍼풀이라는 물리적인 캐시를 늘릴 뿐만 아니라 데이터의 구조를 개선해 전체적인 처리 효과를 향상시킬 수 있다.

## 캐시는 어디까지나 캐시

캐시를 도입할 때 조심해야 할 점은 **캐시는 어디까지나 캐시로써 다뤄야 한다.**

파일 시스템 캐시나 DB의 버퍼 풀처럼 디스크상의 데이터를 메모리에 캐시할 때는 시스템 문제 등으로 인해서 메모리 상의 데이터가 손실된 때도 있다. 하지만 디스크라는 영속화된(엑세스가 느림) 장치에 데이터가 남아있어서 데이터가 손실되는 일은 없다.

캐시에 있는 데이터는 기본적으로 사라져도 상관없다는 전제로 설계해야 한다. 캐시를 중심으로 하는 설계가 아닌 캐시의 바탕이 되는 논리 데이터와 캐시는 명확하게 구별해서 운영해야 한다.

→ 캐시의 바탕이 되는 데이터를 논리 데이터라고 한다.

## 캐시로 사용하기 요건

DB상의 데이터를 다른 무언가로 캐시하려면 명확한 절차가 필요하다.

- 캐시에 대한 질의가 캐시에 없을 때의 대처법
- 캐시에 없을 때 DB에 질의하는 방법
- 캐시에 없을 때 새롭게 캐시에 항목을 추가하는 방법
- 데이터가 갱신됐을 때 캐시와 동기화하는 방법
- 데이터가 모두 손실됐을 때 1회 재구성하는 방법
- 캐시의 정합성을 확인하는 방법

논리 데이터와 캐시를 동기화하는 방법으로 트리거를 사용해 논리 데이터가 갱신될 때마다 동기화하는 방법과 배치 처리를 이용해 정기적으로 동기화하는 방법을 생각할 수 있다.

## 캐시할만한 데이터의 종류

### 캐시하면 안 되는 데이터

대표적으로 **트랜잭션에서 참조하는 데이터**가 있다. 트랜잭션에서 참조하는 데이터와 갱신하는 데이터는 완전 동기화하지 않으면 안 되기 때문이다.

설계에 따라서 캐시는 논리 데이터보다 갱신이 늦어지거나 정합성이 맞지 않을 가능성이 있다. 트랜잭션은 논리 데이터만을 사용해 수행해야 한다. 또한, 논리 데이터는 가능하면 관계형 모델에 따라 설계해야 한다.

### 캐시할 수 있는 데이터

캐시하려면 다음의 조건이 필요하다.

- 매우 빈번하게 엑세스하는 데이터
- 엑세스에 편차가 있음
- 장기간 변경될 여정이 없음
- 표시하는 것이 목적인 데이터

이와 같은 특징이 있는 데이터로는 사용자 인증, 블로그, CMS(Content Management System) 등의 페이지 데이터, 검색용 데이터 등을 들 수 있다.

# 2. 캐시의 구축 방법

## NoSQL을 캐시로 사용

NoSQL을 메인 DB로 사용하는 것은 저자는 회의적으로 보고 있다. 그러나 NoSQL을 RDB의 캐시로 병용할 때는 전혀 문제가 없다. NoSQL의 종류에 따라 다르지만, 다음과 같은 특성이 있는 DB가 있기 때문이다.

- 오버헤드가 적고 엑세스가 매우 빠름
- RDB가 잘 처리 못하는 종류의 검색이 특기
- 여러 대의 서버로 분산처리할 수 있음

엑세스 시에 오버헤드가 적은 것의 예로 KVS(Key-Value Store)를 들 수 있다. KVS는 오버헤드가 매우 적고 대량의 엑세스에도 안정적인 응답과 처리를 제공할 수 있다. 1대로는 성능이나 캐시 사이즈에 한계가 있어도, 여러 대의 호스트를 사용해 스케일 아웃하기가 쉽다.

KVS의 사용 방법은 MySQL과 멤캐시드(memcached)를 조합해 사용하는 방법이 일반적이다.

![01](https://user-images.githubusercontent.com/49144662/115144698-05035400-a089-11eb-8854-007b91bcb33c.png)

클라이언트에서 인덱스로 검색시작 →

**SQL**

- 캐시를 사용하지 않는 쿼리
- 캐시 불일치시의 쿼리
- 트랜잭션

**맴캐시드 클러스터**

- cache get(캐시 일치시)
- cache set(캐시 불일치)

단순하게 특정 데이터에 대한 엑세스를 고속화할 때나 DB에서 가져온 데이터를 임의의 가공을 거친 내용(웹 페이지 등)이 저장된다. KVS를 사용할 수 있는 것은 등가비교로 데이터로 가져온 경우로 한정하는 것이 요점이다.

응용프로그램이 캐시에 데이터를 저장하는 구조로 되어 있지만 UDF(User Defined Function, 사용자 정의 함수)와 트리거를 사용해 데이터를 갱신할 때 MySQL에서 맴캐시드로 데이터를 동기화하는 구조도 가능하다.

좀 더 복잡한 검색이 필요할 때는 그래프 DB나 전문 검색 엔진, 분석 DB, 도큐먼트형 DB 등을 사용하면 좋다. 이러한 NoSQL 제품은 KVS에서는 할 수 없는 복잡한 검색을 RDB 대신 수행할 수 있다.

**NoSQl이 자랑하는 검색은 RDB의 검색과는 다르므로 NoSQL이 장점으로 가지고 있는 검색에 필요한 데이터를 NoSQL 쪽에 캐시해 전반적인 처리의 성능 향상을 기대할 수 있다.**

### 논리 데이터는 RDB에서 관리

중요한 점은 논리 데이터는 RDB에서 관리하는 것이다. NoSQL쪽에서도 유연한 스키마를 표현할 수 있다고 해도 NoSQL에 중요한 데이터를 관리해서는 안 된다.

캐시는 어디까지나 캐시로 다뤄야 한다. NoSQL은 데이터에 이상이 생겨도 막을 수 없지만 RDB라면 데이터에 이상이 생기는 것을 억제하고 데이터의 품질을 유지할 수 있다.

NoSQL은 논리 데이터를 저장하는 RDB와는 다른 제품이며 다른 프로토콜이나 드라이버를 사용해 엑세스하므로 응용프로그램에서는 두 개 또는 그 이상의 DB 제품을 취급하게 되며 그만큼 복잡성은 증가한다.

### 데이터를 동기화할 때의 주의사항

데이터를 동기화하기 위한 알고리즘도 응용프로그램에서 구현해야 한다. NoSQL에 데이터를 전송할 때 JOIN(결합) 등을 수행해 저장하는 등의 가공이 필요한 경우 NoSQL에 부정합이 생기지 않게 주의를 기울여야 한다.

데이터의 동기화는 ETL(Extract/Transform/Load) 제품을 활용하는 것도 좋다.

대표적 ETL 도구: Splunk, 하둡기반(Flume, Sqoop)

### NoSQL로 RDB를 대체할 수 있을까?

RDB를 도큐먼트형 DB(예시: MongoDB)로 바꾸는 이유는 스키마가 없는 것이 큰 이유다. RDB에 있어 DB 설계는 어려운 일이므로 스키마를 사용하는 어려움에서 벗어날 수 있지 않을까라고 생각하기 때문이다.

도큐먼트형 DB와 같은 NoSQL은 언제든지 유연하게 데이터의 구조를 바꿀 수 있지만 데이터의 중복이나 모순과 같은 문제에서 벗어날 수 없다.

RDB는 정규화나 직교화라는 해결 방법이 있지만, 도큐먼트형 DB는 이 방법이 없어서 데이터의 모순을 해결하기가 매우 어렵다.

→ 도큐먼트형 DB의 데이터를 정규화하면 원래 JOIN이 필요한 쿼리는 다른 난해한 회피 방법에 의존해 구축할 수 밖에 없다. → NoSQL에서 정규화라는 방법을 사용할 수 없다.

또한, 스키마가 없는 경우 데이터 구조의 변경을 막을 수단이 없다. 도큐먼트형 데이터 구조의 변화에 제약을 걸 수 없기 때문에 작업에 착오가 생겨 데이터 구조가 바뀌고, 데이터의 갱신에 누락이 발생할 가능성이 있다.

## 테이블을 캐시로 사용

테이블을 캐시로 사용한다고 하면 테이블 안의 데이터를 테이블에 캐시한다고 해도 엑세스 속도가 같으니 의미는 없지 않을까?

하지만 일단 데이터에 대한 어떠한 처리를 한 다음에 다른 테이블에 저장한다면 다르다. 적어도 쿼리마다 데이터를 가공하는 비용은 줄어들 것이다.

가공한 비용에 따라서는 수천 행을 처리하지 않으면 얻을 수 없는 답을 단 몇 줄에 엑세스 하는 것만으로 얻는 경우도 있다. 쿼리에 따라서는 엑세스해야 하는 행 수가 줄어들면 그만큼 확실하게 성능이 향상된다.

데이터를 일단 가공하는 것은 관계형 모델이나 정규화와 같은 정석에서 벗어나는 것을 의미한다.

→ 테이블을 비정규화 하는 경우도 있지만 어디까지나 캐시로 사용하는 것뿐이라면 비정규화 된 상태의 데이터를 가지고 있어도 문제가 없다. 이 경우에 캐시와 논리 데이터를 혼동하지 않는게 중요하다.

특히 비정규화된 테이블에 엑세스가 빠르다고 해서 그 테이블만으로 데이터를 표현하려고 하는 것은 잘못된 생각이다. 관계형 모델에 따라 제대로 설계되고 정규화된 논리 데이터가 있으므로 캐시로서 비정규화 된 상태의 데이터를 가지는 것이 허락되는 것이다.

테이블을 캐시로 사용할 때의 장점은 NoSQL의 경우처럼 여러 개의 프로토콜을 구분하지 않아도 된다는 점이다. SQL만으로 처리하기 위해서 NoSQL을 병용하는 경우와 비교했을 때 개발 비용을 줄일 수 있다. 또한, RDB만 관리하면 되므로 운영 비용도 줄일 수 있다.

### 단점

한 개의 DB 서버에 관리해야 할 데이터 사이즈가 늘어나므로 버퍼풀의 캐시 적중률이 낮아질 가능성이 있다.

또한, 갱신할 테이블이 늘어나기 때문에 갱신의 오버헤드도 커진다.

**→ 테이블을 캐시로 사용할 때 장점이 있는 경우는 갱신보다 참조가 압도적으로 많을 때**

## 테이블을 캐시로 사용하는 예

## 집계 테이블

테이블에 미리 가공한 데이터를 저장하는 용도로 사용되는 것 중에 대표적인 것이 어떤 테이블에서 구한 집계 결과를 별도의 테이블에 저장해두는 것이다. → **집계 테이블**

집계에서 사용하는 집계 함수는 굉장히 비용이 많이 드는 처리다. 집계 처리는 여러 행에 엑세스하지 않으면 결과를 얻을 수 없기 때문이다.

→ 집계된 데이터를 미리 준비해 두면, 극소수의 행에 엑세스하는 것만으로 쿼리를 해결할 수 있다는 전략에 매우 효과적이다.

웹 응용프로글매에서 이런 패턴이 잘 맞는것이 랭킹이다. 테이블에서 그룹별로 발생횟수(COUNT)를 계산하고 그 결과에 따라 정렬한다.

```sql
SELECT item_name, COUNT(*)
FROM orders
GROUP BY item_name
ORDER BY COUNT(*) DESC
LIMIT 10;
```

이 쿼리는 테이블 전체를 스캔하여 많은 컴퓨터 자원을 소비한다. 엑세스가 조금 늘어나는 것만으로도 순식간에 DB 서버의 리소스를 낭비하게 된다.

그런데 이런 집계 결과를 위에서부터 순서대로 표시하는 것과 같은 기능이 요구되는 경우가 많다.

→ GROUP BY보다 훨씬 자원의 소비가 적은 해결책이 필요하다.

그래서 필요한 것이 캐시다. 다음 테이블처럼 집계 테이블에 쿼리를 수행해보자.

### 집계 결과가 저장된 테이블

| item_name    | order_count |
| ------------ | ----------- |
| 아령세트     | 120         |
| 악력기       | 550         |
| 턱걸이 기계  | 25          |
| 복근운동기구 | 44          |

```sql
SELECT item_name, order_count
FROM order_summary
ORDER BY order_count DESC
LIMIT 10;
```

이 쿼리는 order_count 컬럼에 인덱스가 있다면 정렬 인덱스를 사용해 해결할 수 있으므로 매우 빠른 속도로 처리될 것이다. 단 10줄만 인덱스 항목을 읽으면 쿼리를 해결할 수 있기 때문이다.

10줄 정도를 가져오는 쿼리라면 상당한 빈도로 실행해도 웬만해서는 성능의 한계에 도달하지 못할 것이다.

그렇다면 이 테이블의 데이터는 어떻게 작성해야 할까?

### 1. 원 테이블에 트리거를 걸어서 동기화하는 방법

order 테이블에 데이터가 추가될 때마다 order_count 테이블의 데이터를 갱신하는 방법을 생각할 수 있다.

```sql
## 데이터가 추가될 때마다 다른 테이블에 데이터를 추가
delimiter //
CREATE TRIGGER bi_orders
BEFORE INSERT ON orders FOR EACH ROW
BEGIN
DECLARE c INT DEFAULT 0;

SELECT COALESCE(order_count, 0) INTO c FROM order_count
	WHERE item_name = NEW.item_name;
IF c = 0
THEN
	INSERT INTO order_summary (item_name, order_count)
	VALUES(NEW.item_name, 1);
ELSE
	UPDATE order_summary
		SET order_count = c + 1;
		WHERE item_name = NEw.item_name;
	END IF;
END;//
delimiter;
```

### 2. 정기적인 배치 처리로 집계 테이블을 갱신하는 방법

이 경우 전체에 대해 GROUP BY를 실행한 결과를 테이블에 저장하는 것이 가장 쉬운 구현.

```sql
# 정기적인 배치처리로 집계 테이블을 갱신
REPLACE INTO order_summary
	SELECT item_name, COUNT(*)
	FROM orders
	GROUP BY item_name;
```

- REPLACE 문은 MySQL 고유 문법

만약 배치 처리에 따른 전체 테이블의 스캔이 싫다면 마지막으로 집계한 이후에 갱신된 행만 옮겨서 추가된 데이터만 집계 테이블에 반영할 수도 있다. 쿼리의 실행 횟수가 증가하겠지만, 마지막으로 집계 테이블을 갱신한 이후의 경과 시간이 아주 짧고, 집계 대상이 되는 행이 적다면 전체 스캔보다 이 처리가 빠르다.

다음은 order_id = 10000까지의 데이터를 지난번에 배치 처리로 집계했다고 가정한 경우다.

```sql
INSERT INTO order_summary
	SELECT item_name, 0
	FROM (
		SELECT DISTINCT item_name
		FROM orders
		WHERE order_id > 10000
	) t1
	LEFT JOIN order_summary USING (item_name)
	WHERE order_summary.item_name IS NULL:
UPDATE order_summary
	JOIN (
		SELECT DISTINCT item_name
		FROM orders
		WHERE order_id > 10000) t1
		USING (item_name)
	SET order_count = order_count + (
		SELECT COUNT(*) AS c FROM orders
		WHERE item_name t1.item_name
		AND order_id > 10000);
SELECT MAX(order_id) FROM orders;
```

이러한 쿼리는 같은 트랜잭션에서 실행해야 하므로 주의해야 한다. 배치 처리로 집계 테이블의 갱신을 수행하면 논리 데이터와는 약간의 시간 차이가 발생한다. 사용자가 검색 결과를 기준으로 무언가 판단하는 경우라면 논리 데이터와 검색 결과가 동기화되지 않아도 상관없을 케이스도 있을 수 있다.

배치 처리로 집계 테이블을 갱신하는 것은 이런 케이스에 사용할 수 있다.

스스로 기본 테이블을 유지 보수하는 대신에 실체화 뷰(Materialized VIew)를 사용하는 방법이 있다.

## 조인(JOIN)된 데이터

결합은 관계형 모델에서는 일반적인 작업이며 결합하는 작업 자체가 나쁜 것은 아니다.

그러나 굉장히 엑세스가 많을 때는 단 한 번의 연산도 허용되지 않는 경우가 있다. 이럴 때 고속화를 위해서 유용한 것이 캐시다. 조인된 데이터를 테이블을 사용해 캐시하고, 엑세스를 극한으로 고속화하면 시스템이 갖는 성능의 한계를 끌어올릴 수 있다.

집계 테이블과 달리 조인된 데이터는 사이즈가 조인 전의 데이터보다 커지게 된다.

그러나 아무리 캐시의 데이터가 크다고 해도 캐스는 어디까지나 캐시이며 논리 데이터에서 데이터를 복사하자.

캐시의 갱신에 관해서는 기존 데이터 값의 갱신과 삭제, 외부키의 연결을 사용하고, 새로운 데이터를 삽입할 때에는 트리거로 결합한 다음에 테이블에 동기화하는 것이 좋다.

다음은 order 테이블에 행이 삽입될 때 orders_cache 테이블에 데이터를 복사하는 트리거의 예다.

```sql
delimiter //
CREATE TRIGGER bi_orders
BEFORE INSERT ON orders FOR EACH ROW
BEGIN
	INSERT INTO orders_cache
		(order_id, order_timestamp, order_qty,
		user_id, user_name,
		shipping_address, payment_address,
		item_id, item_name, price_id, unit_price)
		SELECT order_id, ordered_timestamp, qty,
			user_id, user_name
			a1.address, a2.address,
			item_id, item_name, price_id, unit_price
			FROM orders
				INNER JOIN users USING (user_id)
				INNER JOIN items USING (item_id)
				INNER JOIN prices USING (price_id)
				INNER JOIN addresses a1
					ON orders.shipping_address_id = a1.address_id
				INNER JOIN addresses a2
					ON orders.payment_address_id = a2.address_id;
	END;//
delimiter ;
```

여러 개의 테이블을 조인하고 있지만, 캐시에 엑세스하는 것으로 이러한 조인을 피할 수 있다.

조인된 데이터를 사용했을 때의 장점은 다음과 같다.

### 디스크 I/O를 줄일 수 있다

조인하기 전의 상태로 데이터를 저장하는 경우 조인을 해결하려면 각 테이블에 엑세스해야 한다. 엑세스에 필요한 데이터는 버퍼 풀에 전혀 캐시돼 있지 않으면 각 테이블에 엑세스할 때마다 디스크 I/O가 발생한다.

조인된 테이블의 데이터는 한 군데에 저장되므로 캐시에 적중하지 않았을 때의 I/O 횟수를 줄일 확률이 높아진다.

그러나 큰 캐시를 사용하므로 버퍼 풀의 캐시 적중률 자체가 저하될 가능성이 있다.

### 정렬과 궁합이 좋다

두 개의 테이블에 있는 데이터를 조인한 다음 정렬하는 경우에 실행 계획에 따라서는 정렬을 인덱스로 해결하지 못하고 조인한 다음에 다시 퀵정렬 등과 같은 알고리즘을 사용해 정렬해야 하는 경우가 있다.

→ 이러한 실행 계획은 인덱스를 사용한 정렬보다 비효율적이다.

조인된 테이블에 대해 인덱스를 생성해두면 그 인덱스를 사용해 정렬을 해결할 수 있다. 특히 상위 몇 건만 필요할 때는 인덱스를 사용해 정렬을 해결하는 장점이 커진다.

### NewSQL과 궁합이 좋다

최근에는 RDB에 대해 KVS를 위한 인터페이스를 제공하는 제품이 늘어나고 있다. 이처럼 순수한 RDB도 NoSQL도 아닌 하이브리드 형 DB 제품을 NewSQL이라 한다.

MySQL에도 KVS의 인터페이스를 제공하는 기능이 있지만 기본키를 사용한 등가 비교라면 SQL 인터페이스를 통해 SELECT로 엑세스하는 것보다 훨씬 빨라진다. 사전에 데이터를 결합해두면 KVS 인터페이스를 사용해 엑세스한다는 선택지도 선택할 수 있다.

## 태그

태그란 개체를 나타내는 데이터에 연상되는 속성을 나타내고자 붙이는 라벨이다.

태그는 원래 논리적인 데이터의 구조로는 완벽하게 관계형 모델에 적합하다.

→ 태그를 테이블로 표현해도 DB 설계상에는 아무런 문제가 없다.

### 태그를 나타내는 테이블

| tag      | item_name      |
| -------- | -------------- |
| 초보자용 | 튜브 세트      |
| 초보자용 | 복근 벤치      |
| 초보자용 | 푸쉬업 바      |
| 초보자용 | 아령 2kg       |
| 상급자용 | 아령 세트 60kg |
| 상급자용 | 턱걸이 기계    |
| 상급자용 | 악력기 80kg    |
| 큰사이즈 | 턱걸이 기계    |
| 큰사이즈 | 복근 벤치      |
| 큰사이즈 | 헬스 자전거    |

태그는 검색 조건을 지정하는 데는 매우 편리하다. 한 개 또는 여러 개의 태그를 지정해 원하는 아이템 등을 찾을 수 있다. → 그런데 무슨 문제일까?

문제는 데이터의 구조(스키마)가 아니고 데이터 그 자체에 있다. 한 개의 아이템에 대해 여러 개의 태그를 지정할 수 있으므로 태그의 데이터는 매우 커지게 된다. 또한, 인기 태그가 있을 때는 매우 많은 아이템에 같은 태그를 붙게 된다.

다음은 '초보자용' 이라는 태그가 있는 제품을 찾는 쿼리다.

```sql
SELECT item_name FROM tags
WHERE tag = '초보자용';
```

초보자용 상품이 대량이라고 생각되므로 이 쿼리는 매우 많은 행을 반환할 것이다. 이 쿼리보다 더 까다로운 것은 여러 개의 태그를 동시에 지정하는 경우다.

다음은 '초보자용', '상급자용' 두 개의 태그가 지정된 아이템을 찾는 쿼리다.

```sql
SELECT item_name FROM tags t1
INNER JOIN tags t2 USING (item_name)
WHERE t1.tag = '초보자용'
AND t2.tag = '상급자용';
```

각 태그로 찾은 두 가지 아이템 목록에서 교집합인 공통 아이템을 구할 때 필요한 작업이다. 상급자의 요구사항은 다양하게 나뉘므로 상급자용 아이템의 제품군은 많을 것이다.

→ 이 쿼리는 두 개의 큰 집합 사이의 교집합이므로 연산을 위한 비용이 매우 비싸다.

태그 검색을 실용적으로 하려면 연산 비용을 줄이기 위한 노력이 필요하다. 검색할 때 지정하는 태그가 한 개뿐일 때는 비교적 쉽다. 태그를 사용한 검색의 경우 그 검색 결과 전체를 사용자가 필요로 하는 경우는 거의 없다.

검색 결과 중에 인기가 있는 아이템부터 순서대로 표시하는 식으로 사용될 것이다.

다음은 점수가 붙은 태그의 예다.

### 점수가 붙은 태그

| tag      | item_name      | score |
| -------- | -------------- | ----- |
| 초보자용 | 튜브 세트      | 100   |
| 초보자용 | 복근 벤치      | 150   |
| 초보자용 | 푸쉬업 바      | 120   |
| 초보자용 | 아령 2kg       | 80    |
| 상급자용 | 아령 세트 60kg | 110   |
| 상급자용 | 턱걸이 기계    | 90    |
| 상급자용 | 악력기 80kg    | 50    |
| 큰사이즈 | 턱걸이 기계    | 90    |
| 큰사이즈 | 복근 벤치      | 150   |
| 큰사이즈 | 헬스 자전거    | 70    |

이 테이블은 item_name → score 라는 함수 종속성이 존재하므로 정규화 되어 있다고 할 수 없다.

비정규화된 테이블은 원래 테이블의 구조는 비슷해도 캐시로 취급한다.

이 테이블에 (tag.score)라는 인덱스가 있다면 점수 상위 10건인 초보자용 아이템을 빠르게 구할 수 있을 것이다.

```sql
SELECT item_name, score FROM scored_tags
WHERE tag = '초보자용'
ORDER BY score DESC
LIMIT 10;
```

만약 '초보자용', '상급자용' 이라는 두 개의 태그를 동시에 지정하면 어떻게 될까?

이 테이블은 이와 같은 검색을 고속화할 수 없다. 여러 개의 태그를 지정한 경우에는 다른 구조의 캐시가 필요하다.

다음은 두 개의 태그를 위한 테이블이다.

### 두개의 태그 테이블

| tag1     | tag2     | item_name     | score |
| -------- | -------- | ------------- | ----- |
| 초보자용 | 상급자용 | 복근 벤치     | 150   |
| 초보자용 | 상급자용 | 웨이 단백질   | 180   |
| 초보자용 | 상급자용 | 푸쉬업 바     | 120   |
| 상급자용 | 큰사이즈 | 턱걸이 기계   | 90    |
| 상급자용 | 보조     | 웨이 단백질   | 180   |
| 상급자용 | 보조     | 카제인 단백질 | 140   |

다음 쿼리는 두 개의 태그를 지정하면서 인기가 있는 10개의 아이템을 구하는 쿼리다.

```sql
SELECT item_name FROM scored_double_tags
WHERE tag1 = '초보자용'
	AND tag2 = '상급자용'
ORDER BY score DESC
LIMIT 10;
```

캐시를 사용한 쿼리는 매우 빠르지만, 이 테이블의 데이터를 어떻게 작성해야 할까?

어떤 아이템이 n개의 태그가 있을 때 이 아이템에 대해 지정할 수 있는 두개의 태그 조합은 $nC2$ 가 된다.

이런 조합 전체에 대해 미리 캐시에 데이터를 저장해야 한다.

이 캐시를 SQL로 구하려면 자체조인(self-JOIN)을 사용한 쿼리를 쓰는 것이 좋다.

다음 쿼리는 tag 테이블에서 '턱걸이 기계' 라는 아이템에 대해 두 개의 태그 조합을 모두 열거하고 scored_double_tags 테이블에 추가한 것이다.

```sql
# 두 개의 태그 조합을 모두 열거하고 테이블에 추가
DELETE FROM scored_double_tags
	WHERE item_name = '턱걸이 기계'
CREATE TEMPORARY TABLE tmp_tags (tag VARCHAR(100));

INSERT INTO tmp_tags SELECT tag FROM tags
	WHERE item_name = '턱걸이 기계';

INSERT INTO scored_double_tags
	(tag1, tag2, item_name,score)
	SELECT t1.tag, t2.tag, '턱걸이 기계', 90
		FROM tmp_tags t1 INNER JOIN tmp_tags t2
		WHERE t1.tag < t2.tag;
```

이 방식의 문제점은 캐시의 사이즈가 매우 커진다는 것이다.

어떤 아이템에 대한 태그의 수가 m개라고 하면 그 중에 n개를 취득하는 조합은 $nCm$ 개다.
예를 들어 m은 10, n은 5라면 252개다.

특히 n이 늘어날 수록 비약적으로 커진다. 만약 태그의 수를 제한하지 않는다면 이 테이블에 태그를 캐시하는 방식은 포기하는 게 좋다. 태그의 수를 가능한 5개, 많아도 10개 정도로 한정할 수 있다면 테이블에 태그의 검색에 유리한 데이터를 미리 만들 수 있다.

### 5개까지 태그를 허용하는 캐시 테이블의 설계

태그별로 유니크한 정수형의 ID, 즉 대리키를 붙이는 방법이 있다. 이렇게 함으로써 한 개의 태그가, MySQL의 정수형을 사용한 경우 4바이트로 표현할 수 있다.

논리 데이터의 대리키를 도입하는 것은 신중해야 하지만 캐시로 사용한다면 부담 없이 대리키를 사용해도 괜찮다.

대리키로 tag_id를 추가했다고 하고, 아이템에 대해서도 대리키를 적용했다고 하자. 그러면 검색시에 5개까지 테그를 허용하는 캐시 테이블은 다음처럼 정의할 수 있다.

```sql
CREATE TABLE tag_index (
	tag1 INT UNSIGNED NOT NULL,
	tag2 INT UNSIGNED NOT NULL,
	tag3 INT UNSIGNED NOT NULL,
	tag4 INT UNSIGNED NOT NULL,
	tag5 INT UNSIGNED NOT NULL,
	item_id INT UNSIGNED NOT NULL,
	score INT UNSIGNED NOT NULL,
	PRIMARY KEY (tag1, tag2, tag3, tag4, tag5, item_id),
	INDEX ix1 (tag1, tag2, tag3, tag4, tag5, score, item_id),
	INDEX ix2 (item_id))
```

인덱스 ix1은 커버링 인덱스다. 태그 검색은 이 인덱스를 사용해 실시된다. 동시에 5개까지 태그를 검색할 목적으로 태그를 나타내는 컬럼이 5개 있다. 만약 태그 1~4개밖에 지정하지 않았을 때는 어떻게 표현할까?

하나의 가능성으로 생각할 수 있는 설계는 5개 미만의 태그의 조합에 대해서도 이 테이블에 미리 데이터를 지정하는 것이다. 5개 미만일 때는 태그를 반드시 tag1부터 순서대로 오름차순으로 지정하고 검색에 사용되지 않는 태그에 대해서 0을 지정하면 된다.

논리 데이터에 대해서 NULL 대신 기본값을 사용하는 것은 좋지 않은 방법이지만, 캐시로 사용된다면 이와 같은 설계도 허용된다. 다음은 '턱걸이 기계'에 대한 태그의 데이터로 추가하는 것이다.

```sql
# MySQL 문법으로 SQL의 트랜잭션으로 수행
SELECT @item_id := item_id, @score := score
	FROM items WHERE item_name = '턱걸이 기계'
DELETE FROM tag_index WHERE item_id = @item_id;

CREATE TEMPORARY TABLE tmp_tags (tag INT);

INSERT INTO tmp_tags
	SELECT tag_id FROM tags WHERE item_name = '턱걸이 기계';

INSERT INTO tag_index
	(tag1, tag2, tag3, tag4, tag5, tag6, item_id, score)
	SELECT tag, 0, 0, 0, 0, 0, @item_id, @score
		FROM tmp_tags
	UNION SELECT t1.tag, t2.tag, 0, 0, 0, @item_id, @score
		FROM tmp_tags t1 INNER JOIN tmp_tags t2
		WHERE t1.tag < t2.tag
	UNION SELECT t1.tag, t2.tag, t3.tag, 0, 0, @item_id, @score
		FROM tmp_tags t1 INNER JOIN tmp_tags t2
			INNER JOIN tmp_tags t3
		WHERE t1.tag < t2.tag AND t2.tag < t3.tag
	UNION SELECT t1.tag, t2.tag, t3.tag, t4.tag, 0, @item_id, @score
		FROM tmp_tags t1 INNER JOIN tmp_tags t2
			INNER JOIN tmp_tags t3
			INNER JOIN tmp_tags t4
		WHERE t1.tag < t2.tag AND t2.tag < t3.tag
			AND t3.tag < t4.tag
	UNION SELECT t1.tag, t2.tag, t3.tag, t4.tag, t5.tag, @item_id, @score
		FROM tmp_tags t1 INNER JOIN tmp_tags t2
			INNER JOIN tmp_tags t3
			INNER JOIN tmp_tags t4
			INNER JOIN tmp_tags t5
		WHERE t1.tag < t2.tag AND t2.tag < t3.tag
			AND t3.tag < t4.tag AND t4.tag < t5.tag;
DROP TEMPORARY TABLE tmp_tags;
```

이와 같은 방법으로 모든 아이템에 대해 태그 검색용 캐시 데이터를 작성한다.

이 데이터를 사용해 '초보자용', '상급자용', '큰사이즈' 3개의 태그를 지정해 검색할 때는 다음의 쿼리를 실행한다.

```sql
CREATE TEMPORARY TABLE tmp_tags (tag INT);

INSERT INTO tmp_tags
	SELECT tag_id FROM tags
		WHERE tag IN ('초보자용', '상급자용', '큰 사이즈');

SELECT item_name, score
	FROM tag_index INNER JOIN items USING (item_id)
	WHERE (tag1, tag2, tag3, tag4, tag5) =
	(SELECT t1.tag, t2.tag, t3.tag, 0, 0
		FROM tmp_tags t1
			INNER JOIN tmp_tags t2
			INNER JOIN tmp_tags t3
		WHERE t1.tag < t2.tag AND t2.tag < t3.tag)
	ORDER BY score DESC;
DROP TEMPORARY TABLE tmp_tags;
```

3개의 다른 값의 곱집합 중에 t1.tag < t2.tag AND t2.tag < t3.tag 라는 조건을 만족하는 행은 한 개밖에 없다.

이 SELECT에 포함된 서브쿼리는 행 서브쿼리가 된다.

또한 이 쿼리는 tag_index 테이블의 검색을 ix1 인덱스를 사용해서 빠르게 처리할 수 있다.

이처럼 거대한 테이블을 캐시라고 부르기에는 거부감이 있을지 모르지만, 사전에 검색에 유리한 구조를 준비해 검색 속도가 크게 개선할 수 있다.

### 전치 인덱스를 사용한 검색

아이템 수가 그리 많지 않을 때는 전치 인덱스를 사용해 태그의 검색을 빠르게 할 수 있다. RDB 제품 중에는 배열 형식을 지원하는 제품이 있어서 태그를 배열로 표현하고 그 배열에 대해 전치 인덱스를 붙여 태그의 검색 속도를 높일 수 있다.

그러나 전치 인덱스를 사용할 때도 이 검색의 본질은 B+트리를 사용한 인덱스와 마찬가지로 거대한 집합 사이의 조인이 된다. 단순히 데이터가 일반적인 B+트리 인덱스보다 작게 저장될 뿐이다.

같은 태그가 붙은 아이템 수가 많이 늘어나면 연산속도는 저하된다.

다음 포스트에 이어 추가하겠습니다.
