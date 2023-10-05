---
title: 'Modern Javascript Deep Dive - 31장 RegExp'
date: 2023-08-16 23:02:08
category: 'Javascript'
draft: false
---

# 1. 정규 표현식이란?

정규 표현식(Regular expression)은 일정한 패턴을 가진 문자열의 집합을 표현하기 위해 사용하는 형식 언어다.

정규 표현식은 문자열을 대상으로 패턴 매칭 기능을 제공한다.

```jsx
// 사용자로부터 입력받은 휴대폰 전화번호
const tel = '010-1234-567팔';

// 정규 표현식 리터럴로 휴대폰 전화번호 패턴 정의
const regExp = /^\d{3}-\d{4}-\d{4}$/;

// 패턴 매칭 확인
regExp.test(tel); // false
```

# 2. 정규 표현식의 생성

정규 표현식 객체를 생성하기 위해서는 정규 표현식 리터럴과 RegExp 생성자 함수를 사용할 수 있다.

일반적인 방식은 정규 표현식 리터럴을 사용하는 것이다.

```jsx
const target = 'Is this all there is?';

// 패턴: is
// 플래그: i => 대소문자를 구별하지 않고 검색한다.
const regexp = /is/i;

// test 메서드는 target 문자열에 대해 정규 표현식 regexp의 패턴을 검색하여
// 매칭 결과를 불리언 값으로 반환한다.
regexp.test(target); // true

// RegExp 생성자 함수를 사용하여 RegExp 객체를 생성할 수도 있다.
const target = 'Is this all there is?';

const regexp = new RegExp(/is/i); // ES6

regexp.test(target); // true
```

# 3. RegExp 메서드

## 3.1 RegExp.prototype.exec

exec 메서드는 인수로 전달받은 문자열에 대해 정규 표현식의 패턴을 검색하여 매칭 결과를 배열로 반환한다.

매칭 결과가 없으면 null을 반환한다.

```jsx
const target = 'Is this all there is?';
const regexp = /is/i;

regexp.exec(target);
// [ 'Is', index: 0, input: 'Is this all there is?', groups: undefined ]
```

exec 메서드는 문자열 내의 모든 패턴을 검색하는 g 플래그를 지정해도 첫 번째 매칭 결과만 반환하므로 주의해야 한다.

## 3.2 RegExp.prototype.test

test 메서드는 인수로 전달받은 문자열에 대해 정규 표현식의 패턴을 검색하여 매칭 결과를 불리언으로 반환한다.

```jsx
const target = 'Is this all there is?';
const regexp = /is/i;

regexp.test(target); // true
```

## 3.3 RegExp.prototype.match

String 표준 빌트인 객체가 제공하는 match 메서드는 대상 문자열과 인수로 전달받은 정규 표현식과의 매칭 결과를 배열로 반환한다.

```jsx
const target = 'Is this all there is?';
const regexp = /is/;

target.match(regexp);

/*
[
  'Is this all there is',
  index: 0,
  input: 'Is this all there is?',
  groups: undefined
]
*/
```

exec 메서드는 문자열 내의 모든 패턴을 검색하는 g 플래그를 지정해도 첫 번째 매칭 결과를 반환한다.

하지만 String.prototype.match 메서드는 g 플래그를 지정되면 모든 매칭 결과를 배여롤 반환한다.

```jsx
const target = 'Is this all there is?';
const regexp = /is/g;

target.match(regexp); // ["is", "is"]
```

# 4. 플래그

| 플래그 | 의미          | 설명                                   |
| --- | ----------- | ------------------------------------ |
| i   | Ignore case | 대소문자를 구별하지 않고 패턴을 검색한다.              |
| g   | Global      | 대상 문자열 내에서 패턴과 일치하는 모든 문자열을 전역 검색한다. |
| m   | Multi line  | 문자열의 행이 바뀌더라도 패턴 검색을 계속한다.           |

```jsx
const target = 'Is this all there is?';

// target 문자열에서 is 문자열을 대소문자를 구별하여 한 번 만 검색한다.
target.match(/is/);
// [ 'is', index: 5, input: , groups: undefined ]?'

// target 문자열에서 is 문자열을 대소문자를 구별하지 않고 한 번 만 검색한다.
target.match(/is/i);
// [ 'Is', index: 0, input: 'Is this all there is?', groups: undefined ]

// target 문자열에서 is 문자열을 대소문자를 구별하여 전역 검색한다.
target.match(/is/g); // [ 'is', 'is' ]

// target 문자열에서 is 문자열을 대소문자를 구별하지 않고 전역 검색한다.
target.match(/is/gi); // [ 'Is', 'is', 'is' ]
```

# 5. 패턴

패턴은 / 로 열고 닫으며 문자열의 따옴표는 생략한다.

## 5.1 문자열 검색

정규 표현식의 패턴에 문자 또는 문자열을 지정하면 검색 대상 문자열에서 패턴으로 지정한 문자 또는 문자열을 검색한다.

```jsx
const target = 'Is this all there is?';

// target 문자열에서 is 문자열을 대소문자를 구별하여 한 번 만 검색한다.
target.match(/is/);
// [ 'is', index: 5, input: , groups: undefined ]?'

// target 문자열에서 is 문자열을 대소문자를 구별하지 않고 한 번 만 검색한다.
target.match(/is/i);
// [ 'Is', index: 0, input: 'Is this all there is?', groups: undefined ]

// target 문자열에서 is 문자열을 대소문자를 구별하여 전역 검색한다.
target.match(/is/g); // [ 'is', 'is' ]

// target 문자열에서 is 문자열을 대소문자를 구별하지 않고 전역 검색한다.
target.match(/is/gi); // [ 'Is', 'is', 'is' ]
```

## 5.2 임의의 문자열 검색

`.` 은 임의의 문자 한 개를 의미한다.

```jsx
const target = 'Is this all there is?';

// 임의의 3자리 문자열을 대소문자를 구별하여 전역 검색한다.
const regExp = /.../g;

console.log(target.match(regExp));

/*
[
  'Is ', 'thi',
  's a', 'll ',
  'the', 're ',
  'is?'
]
*/
```

## 5.3 반복 검색

{m, n} 은 앞선 패턴이 최소 m번, 최대 n번 반복되는 문자열을 의미한다. 콤마 뒤에 공백이 있으면 정상 동작하지 않으므로 주의해야 한다.

```jsx
const target = 'A AA B BB Aa Bb AAA';

// A가 최소 1번, 최대 2번 반복되는 문자열을 전역 검색한다.
let regExp = /A{1,2}/g;

target.match(regExp) // [ 'A', 'AA', 'A', 'AA', 'A' ]

// {n}은 앞선 패턴이 n번 반복되는 문자열을 의미한다.
// -> {n} === {n, n}
regExp = /A{2}/g;

target.match(regExp); // ['AA', 'AA']

// {n,}은 앞선 패턴이 최소 n번 이상 반복되는 문자열을 의미한다.
regExp = /A{2,}/g;

target.match(regExp); // ['AA', 'AAA']

// +는 앞선 패턴이 최소 한번 이상 반복되는 문자열을 의미한다. +는 {1,}
// 'A'가 최소 한 번 이상 반복되는 문자열('A', 'AA', 'AAA',....)을 전역 검색한다.
regExp = /A+/g;

target.match(regExp); // [ 'A', 'AA', 'A', 'AAA' ]

// ?는 앞선 패턴이 최대 한 번(0번 이상) 이상 반복되는 문자열을 의미한다.
// ?는 {0,1}과 같다.
// 'colo' 다음 'u'가 최대 한 번(0번 포함) 이상 반복되고 'r'이 이어지는
// 문자열 'color', 'colour'를 전역 검색한다.
const target = 'color colour';

const regExp = /colou?r/g;
target.match(regExp); // ['color', 'colour']
```

## 5.4 OR 검색

`|` 은 OR의 의미를 갖는다.

```jsx
const target = 'A AA B BB Aa Bb';

// 'A' 또는 'B'를 전역 검색한다.
let regExp = /A|B/g;

target.match(regExp);

/*
[
  'A', 'A', 'A',
  'B', 'B', ,B'
  'A', 'B'
]
*/

// 분해되지 않은 단어 레벨로 검색하기 위해서는 +를 함께 사용한다.
// 'A', 'AA', 'AAA', 'B', 'BB'...
regExp = /A+|B+/g;

target.match(regExp); // [ 'A', 'AA', 'B', 'BB', 'A', 'B' ]

// []내의 문자는 or로 동작한다. 그 뒤에 +를 사용하면 앞선 패턴을 한 번 이상 반복한다.
regExp = /[AB]+/g;

target.match(regExp); // [ 'A', 'AA', 'B', 'BB', 'A', 'B' ]

// 범위를 지정하려면 []내에 -를 사용한다.
const target = 'A AA BB ZZ Aa Bb';

// 'A' ~ 'Z'가 한 번 이상 반복되는 문자열을 전역 검색한다.
regExp = /[A-Z]+/g;

target.match(regExp); // [ 'A', 'AA', 'BB', 'ZZ', 'A', 'B' ]

// 대소문자를 구별하지 않고 알파벳을 검색하는 방법은 다음과 같다.
const target = 'AA BB Aa Bb 12';

// 'A' ~ 'Z' 또는 'a' ~ 'z'가 한 번 이상 반복되는 문자열을 전역 검색한다.
regExp = /[A-Za-z]+/g;

target.match(regExp); // [ 'AA', 'BB', 'Aa', 'Bb' ]
```

숫자를 검색하는 방법

```jsx
const target = 'AA BB 12,345';

// '0' ~ '9'가 한 번 이상 반복되는 문자열을 전역 검색한다.
let regExp = /[0-9]+/g;

target.match(regExp); // ['12', '345']

// 쉼표를 패턴에 포함한다면
regExp = /[0-9,]+/g;

target.match(regExp); // ['12,345']

// \d는 숫자를 의미한다.
let regExp = /[\d,]+/g;

target.match(regExp); // ['12,345']

// \D는 숫자가 아닌 문자를 의미한다.
// 숫자가 아닌 문자 또는 ','가 한 번 이상 반복되는 문자열을 전역 검색한다.
regExp = /[\D,]+/g;

target.match(regExp); // [ 'AA BB ', ',' ]

// \w는 알파벳, 숫자, 언더스코어를 의미한다.
// \w는 [A-Za-z0-9_]와 같다.
const target = 'Aa Bb 12,345 _$%&';

// 알파벳, 숫자, 언더스코어, ','가 한 번 이상 반복되는 문자열을 전역 검색한다.
let regExp = /[\w,]+/g;

target.match(regExp); // [ 'Aa', 'Bb', '12,345', '_' ]

// 알파벳, 숫자, 언더스코어가 아닌 문자 또는 ','가 한 번 이상 반복되는 문자열을 전역 검색한다.
regExp = /[\W,]+/g;

target.match(regExp); // [ ' ', ' ', ',', ' ', '$%&' ]
```

## 5.5 NOT 검색

[…] 내의 ^은 not의 의미를 갖는다.

```jsx
const target = 'AA BB 12 Aa Bb';

// 숫자를 제외한 문자열을 전역 검색한다.
const regExp = /[^\d]+/g;

target.match(regExp); // [ 'AA BB ', ' Aa Bb' ]
```

## 5.6 시작 위치로 검색

[…] 밖의 ^은 문자열의 시작을 의미한다. 단, […]내의 ^은 not의 의미를 가진다.

```jsx
const target = 'https://naver.com';

// 'https'로 시작되는지 검사한다.
const regExp = /^https/g;

regExp.test(target); // true
```

## 5.7 마지막 위치로 검색

\$는 문자열의 마지막을 의미한다.

```jsx
const target = 'https://naver.com';

// 'com'로 시작되는지 검사한다.
const regExp = /com$/g;

regExp.test(target); // true
```

# 6. 자주 사용하는 정규 표현식

## 6.1 특정 단어로 시작하는지 검사

```jsx
const url = 'https://naver.com';

// 'http://' 또는 'https://'로 시작하는지 검사한다.
const regExp = /^https?:\/\//;

regExp.test(target); // true
```

## 6.2 특정 단어로 끝나는지 검사

```jsx
const fileName = 'index.html';

const regExp = /html$/;

regExp.test(fileName); // true
```

## 6.3 숫자로만 이루어진 문자열인지 검사

처음과 끝이 숫자이고 최소 한 번 이상 반복되는 문자열검사는 다음과 같다.

```jsx
const target = '12345';

const regExp = /^\d+$/;

regExp.test(target); // true
```

## 6.4 하나 이상의 공백으로 시작하는지 검사

`\s` 는 여러 가지 공백 문자(스페이스, 탭 등)를 의미한다.

```jsx
const target = ' Hi!';

const regExp = /^[\s]+/;

regExp.test(target); // true
```

## 6.5 아이디로 사용 가능한지 검사

알파벳 대소문자 또는 숫자로 시작하고 끝나며 4~10자리인지 검사한다.

```jsx
const id = 'abc123';

const regExp = /^[A-Za-z0-9]{4,10}$/;

regExp.test(id); // true
```

## 6.6 메일 주소 형식에 맞는지 검사

인터넷 메시지 형식 규약인 RFC 5322에 맞는 정교한 패턴 매칭을 사용한다면 무척이나 복잡한 패턴을 가진다.

```jsx
const email = 'abc123@gmail.com';

const regExp = /^[-0-9A-Za-z!#$%&'*+/=?^_`{|}~.]+@[-0-9A-Za-z!#$%&'*+/=?^_`{|}~]+[.]{1}[0-9A-Za-z]/;

regExp.test(email); // true
```

## 6.7 휴대폰 번호 형식에 맞는지 검사

```jsx
const cellPhone = "010-1234-5678";

const regExp = /^\d{3,4}-\d{3,4}-\d{4,4}$;

regExp.test(cellPhone); // true
```

## 6.8 특수 문자 포함 여부 검사

```jsx
const target = 'abc#123';

const regExp = /[^A-Za-z0-9]/gi;

regExp.test(target); // true

// 특수 문자를 제거한다면 String.prototype.replace 메서드를 사용한다.
target.replace(regExp, ''); // abc123
```
