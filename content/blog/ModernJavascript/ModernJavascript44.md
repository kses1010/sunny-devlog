---
title: 'Modrn Javascript Deep Dive - 44장 REST API'
date: 2023-09-17
category: 'Javascript'
draft: false
---

# 1. REST API의 구성

REST API는 자원, 행위, 표현의 3가지 요소로 구성된다.
REST는 자체 표현 구조로 구성되어 REST API만으로 HTTP 요청의 내용을 이해할 수 있다.

| **구성 요소**         | **내용**                       | **표현 방법**    |
| --------------------- | ------------------------------ | ---------------- |
| 자원(resource)        | 자원                           | URI(엔드포인트)  |
| 행위(verb)            | 자원에 대한 행위               | HTTP 요청 메서드 |
| 표현(representations) | 자원에 대한 행위의 구체적 내용 | 페이로드         |

# 2. REST API 설계 원칙

### URI는 리소스를 표현해야 한다.

URI는 리소스를 표현하는 데 중점을 두어야 한다.

```http
GET /todos/1
```

### 2. 리소스에 대한 행위는 HTTP 요청 메서드로 표현한다.

주로 5가지 요청 메서드를 사용하여 CRUD를 구현한다.

| **HTTP 요청 메서드** | **종류**       | **목적**              | **페이로드** |
| -------------------- | -------------- | --------------------- | ------------ |
| GET                  | index/retrieve | 모든/특정 리소스 취득 | X            |
| POST                 | create         | 리소스 생성           | O            |
| PUT                  | replace        | 리소스의 전체 교체    | O            |
| PATCH                | modify         | 리소스의 일부 수정    | O            |
| DELETE               | delete         | 모든/특정 리소스 삭제 | X            |
