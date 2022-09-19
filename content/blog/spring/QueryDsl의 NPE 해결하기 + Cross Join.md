---
title: 'QueryDSL NPE 문제 해결하기 + Cross Join'
date: 2022-09-19 13:22:00
category: 'Spring'
draft: false
---

## 1. 발생

조회 쿼리를 작성 도중에 500에러가 발생했습니다.

에러코드를 확인해보니 NPE가 발생했습니다.

![picture1](https://user-images.githubusercontent.com/49144662/190952320-c206d333-7d3b-427a-8788-ddef8b1b2138.png)

해당 쿼리에서 Null이 발생했다고 하는군요.

```java
@Override
  public List<PlandGroupSaleItem> findByUserAndPlandGroupAndSaleItemIds(
      User user, PlandGroup plandGroup, Set<Long> saleItemIds) {
    return jpaQueryFactory
        .selectFrom(plandGroupSaleItem)
        .where(
            plandGroupSaleItem.saleItem.commonInfo.register.eq(user),
            plandGroupSaleItem.plandGroup.eq(plandGroup),
            plandGroupSaleItem.saleItem.id.in(saleItemIds)
        )
        .fetch();
  }
```

디버킹으로 찾아본 결과 register에서 Null이 발생했다고 합니다. 단순한 쿼리인데 왜 Null이 발생한 걸까요?

## 2. 원인

StackOverflow을 찾아보니 바로 객체그래프 문제였습니다.

링크 :
[java.lang.NullPointerException: while filtering data using QueryDsl](https://stackoverflow.com/questions/53425631/java-lang-nullpointerexception-while-filtering-data-using-querydsl)

답변:

The NPE that you get is probably related to the limitation of QueryDsl where you cannot exceed four levels.

In the first link there is a solution to overcome this issue using `QueryInit` annotation.

If you do not want to alter your entity class you can perform a join (if possible) with an alias for the first 2 or 3 levels and then use this alias to complete the query.

공식문서에 따르면 객체그래프가 4개이상 넘어가면 NPE가 발생한다고 합니다.

링크 : [3.3. Code generation](https://querydsl.com/static/querydsl/3.5.0/reference/html/ch03s03.html#d0e2181)

**3.3.1. Path initialization**

기본적으로 Querydsl은 처음 두 수준의 참조 속성만 초기화합니다. 더 긴 초기화 경로가 필요한 경우 `com.mysema.query.annotations.QueryInit` 주석을 통해 도메인 유형에 주석을 추가해야 합니다. QueryInit는 깊은 초기화가 필요한 속성에 사용됩니다. 다음 예는 사용법을 보여줍니다.

```java
@Entity
class Event {
    @QueryInit("customer.address")
    Account account;
}

@Entity
class Account{
    Customer customer;
}

@Entity
class Customer{
    String name;
    Address address;
    // ...
}
```

이 예는 이벤트 경로가 루트 경로/변수로 초기화될 때 account.customer 경로의 초기화를 강제 실행합니다. 경로 초기화 형식은 와일드카드도 지원합니다. "customer._" 아무 도메인 "_". 자동 경로 초기화는 엔터티 필드가 최종적이지 않아야 했던 수동 경로 초기화를 대체합니다. 선언적 형식은 쿼리 유형의 모든 최상위 인스턴스에 적용되고 최종 엔터티 필드의 사용을 가능하게 하는 이점이 있습니다. 자동 경로 초기화가 선호되는 초기화 전략이지만 수동 초기화는 아래에 설명된 Config 주석을 통해 활성화할 수 있습니다.

## 3. 해결

`@QueryInit` 을 이용하면 더 긴 경로사용에 도움을 받을 수 있게 됩니다. 그래서 저희 도메인에 적용했습니다.

실제 NPE가 발생하는 where 절인

```java
plandGroupSaleItem.saleItem.commonInfo.register.eq(user)
```

에서 plandGroupSaleItem에서 연결되는 경로에 초기화를 합니다.

```java
// PlandGroupSaleItem.class

	@ManyToOne(fetch = FetchType.LAZY)
  @JoinColumn(name = "sale_item_id")
  @QueryInit("commonInfo.register")
  private SaleItem saleItem;
```

테스트 결과는 정상적으로 잘 동작합니다

![picture2](https://user-images.githubusercontent.com/49144662/190952649-a20dd1cd-8ebf-4209-b381-cd7c6b23c9ed.png)

## 그 외 Cross Join 해결하기

테스트는 통과했으니 위 사진을 보면 알 수 있듯이 cross join이 발생합니다. cross join은 성능 문제를 일으키는 주 원인이기 때문에 cross join을 해결해야합니다.

querydsl의 cross join은 다음 블로그에서 참고했습니다.

링크 : [Querydsl (JPA) 에서 Cross Join 발생할 경우](https://jojoldu.tistory.com/533)

바로 innerJoin을 명시하면 됩니다.

```java
@Override
  public List<PlandGroupSaleItem> findByUserAndPlandGroupAndSaleItemIds(
      User user, PlandGroup plandGroup, Set<Long> saleItemIds) {
    return jpaQueryFactory
        .selectFrom(plandGroupSaleItem)
        // innerJoin 명시
        .innerJoin(plandGroupSaleItem.saleItem, saleItem)
        .where(
            plandGroupSaleItem.saleItem.commonInfo.register.eq(user),
            plandGroupSaleItem.plandGroup.eq(plandGroup),
            plandGroupSaleItem.saleItem.id.in(saleItemIds)
        )
        .fetch();
  }
```

진한 글씨인 innerJoin을 명시적으로 넣으면?

![picture3](https://user-images.githubusercontent.com/49144662/190952780-edc4f1a5-4a40-40ba-8fc4-e59f53338071.png)

바로 cross join이 사라진걸 확인 할 수 있습니다.
