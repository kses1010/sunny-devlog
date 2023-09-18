---
title: 'Modern Javascript Deep Dive - 9장 타입 변환과 단축 평가'
date: 2023-08-01 00:13:03
category: 'Javascript'
draft: false
---

# 1. 타입 변환이란?

자바스크립트의 모든 값은 타입이 있다.

개발자가 의도적으로 값의 타입을 변환하는 것을 **명시적 타입 변환** 또는 **타입 캐스팅**이라 한다.

```jsx
var x = 10;

// 명시적 타입 변환
// 숫자를 문자열로 타입 캐스팅
var str = x.toString();
console.log(typeof str, str); // string 10

console.log(typeof x, x); // number 10
```

개발자의 의도와는 상관없이 표현식을 평가하는 도중에 자바스크립트 엔진에 의해 암묵적으로 타입이 자동 변환하기도 한다. → **암묵적 타입 변환** 또는 **타입 강제 변환** 이라 한다.

```jsx
var x = 10;

// 암묵적 타입 변환
// 숫자를 문자열로 자동 변환
var str = x + '';
console.log(typeof str, str); // string 10

console.log(typeof x, x); // number 10
```

암묵적 타입 변환은 기존 변수 값을 재할당하여 변경하는 것이 아니라 자바스크립트 엔진이 표현식을 에러 없이 평가하기 위해 피연산자의 값을 암묵적 타입 변환해 새로운 타입의 값을 만들어 단 한 번 사용하고 만다.

명시적 타입 변환은 타입을 변경하겠다는 개발자의 의지가 코드에 명백히 드러난다.

# 2. 암묵적 타입 변환

## 2.1 문자열 타이브로 변환

```jsx
1 + '2'; // '12'
```

자바스크립트 엔진은 문자열 연결 연산자 표현식을 평가하기 위해 문자열 연결 연산자의 피연산자 중에서 문자열 타입이 아닌 피연산자를 문자열 타입으로 암묵적 타입 변환한다.

자바스크립트 엔진은 문자열 타입 아닌 값을 문자열 타입으로 암묵적 타입 변환을 수행할 때 다음 처럼 동작한다.

```jsx
// 숫자 타입
0 + ''               // '0'
-0 + ''              // '0'
1 + ''               // '1'
-1 + ''              // '-1'
NaN + ''             // 'NaN'
Infinity + ''        // 'Infinity'
-Infinity + ''       // '-Infinity'

// 불리언 타입
true + ''            // 'true'
false + ''           // 'false'

// null 타입
null + ''            // 'null'

// undefined 타입
undefined + ''       // 'undefined'

// symbol 타입
(Symbol()) + ''      // TypeError

// 객체 타입
({}) + ''            // '[object Object]'
Math + ''            // '[object Math]'
[] + ''              // ''
[10, 20] + ''        // '10, 20'
(function(){}) + ''  // 'function(){}'
Array + ''           // 'function Array() { [native code] }'
```

## 2.2 숫자 타입으로 변환

```jsx
1 - '1' // 0
1 * '10' // 10
1 / 'one' // NaN
```

산술 연산자의 역할은 숫자 값을 만드는 것이다.

→ 산술 연산자의 모든 피연산자는 코드 문맥상 모두 숫자타입이어야 한다.

피연산자를 숫자 타입으로 변경할 수 없는 경우에는 산술 연산을 할 수 없으므로 NaN이 된다.

```jsx
'1' > 0 // true
```

비교 연산자의 역할은 불리언 값을 만드는 것이다.

자바스크립트 엔진은 숫자 타입이 아닌 값을 숫자 타입으로 암묵적 타입 변환을 수행할 때 다음과 같이 동작한다.

→ `+` 단항 연산자는 피연산자가 숫자 타입의 값이 아니면 숫자 타입의 값으로 암묵적 타입 변환을 수행한다.

```jsx
// 문자열 타입
;+'' + // 0
'0' + // 0
'1' + // 1
'one' + // NaN

// 불리언 타입
true + // 1
false + // 0

// null 타입
null + // 0

// undefined 타입
undefined + // NaN

// symbol 타입
Symbol() + // TypeError

// 객체 타입
{} + // NaN
[] + // 0
[10, 20] + // NaN
  function() {} // NaN
```

## 2.3 불리언 타입으로 변환

```jsx
if ('') console.log('1')
if (true) console.log('2')
if (0) console.log(3)
if ('str') console.log('4')
if (null) console.log('5')

// 2 4
```

**자바스크립트 엔진은 불리언 타입이 아닌 값을 Truthy 값 또는 Falsy 값으로 구분한다**.

→ 제어문의 조건식과 같이 불리언 값으로 평가되어야 할 문맥에 Truthy, Falsy 값에 따라 암묵적 타입변환된다.

아래 값이 false로 평가되는 Falsy 값이다.

- false
- undefined
- null
- 0, -0
- NaN
- ‘’(빈문자열)

# 3. 명시적 타입 변환

표준 빌트인 생성자 함수(String, Number, Boolean)를 new 연산자 없이 호출하는 방법,

빌트인 메서드를 사용하는 방법, 암묵적 타입 변환을 이용하는 방법이 있다.

## 3.1 문자열 타입으로 변환

```jsx
// 1. String 생성자 함수를 new 연산자 없이 호출하는 방법
// 숫자 타입 -> 문자열 타입
String(1) // '1'
String(NaN) // 'NaN'
String(Infinity) // 'Infinity'

// 불리언 타입 -> 문자열 타입
String(true) // 'true'
String(false) // 'false'

// 2. Object.prototype.toString 메서드를 사용하는 방법
// 숫자 타입 -> 문자열 타입
;(1).toString() // '1'
NaN.toString() // 'NaN'
Infinity.toString() // 'Infinity'

// 불리언 타입 -> 문자열 타입
true.toString() // 'true'
false.toString() // 'false'

// 3. 문자열 연결 연산자를 이용하는 방법
// 숫자 타입 -> 문자열 타입
1 + ''
NaN + ''
Infinity + ''

// 불리언 타입 -> 문자열 타입
true + ''
false + ''
```

## 3.2 숫자 타입으로 변환

숫자 타입으로 변환하는 방법은 다음과 같다.

- Number 생성자 함수를 new 연산자 없이 호출하는 방법
- parseInt, parseFloat 함수를 사용하는 방법(문자열만 숫자 타입으로 변환 가능)
- `+` 단항 산술 연산자를 이용하는 방법
- `*` 산술 연산자를 이용하는 방법

```jsx
// 1. Number 생성자 함수를 new 연산자 없이 호출하는 방법
// 문자열 타입 -> 숫자 타입
Number('0') // 0
Number('-1') // -1
Number('10.53') // 10.53
// 불리언 타입 -> 숫자 타입
Number(true) // 1
Number(false) // 0

// 2. parseInt, parseFloat 함수를 이용하는 방법(문자열만 가능)
parseInt('0') // 0
parseInt('-1') // -1
parseInt('10.53') // 10.53

// 3. + 단항 산술 연산자를 이용하는 방법
// 문자열 타입 -> 숫자 타입
+'0' // 0
+'-1' // -1
+'10.53' // 10.53

// 불리언 타입 -> 숫자 타입
+true // 1
+false // 0

// 4. * 산술 연산자를 이용하는 방법
'0' * 1 // 0
'-1' * 1 // -1
'10.53' * 1 // 10.53

// 불리언 타입 -> 숫자 타입
true * 1 // 1
false * 1 // 0
```

## 3.3 불리언 타입으로 변환

- Boolean 생성자 함수를 new 연산자 없이 호출하는 방법
- `!` 부정 논리 연산자를 두 번 사용하는 방법

```jsx
// Boolean 생성자 함수를 new 연산자 없이 호출하는 방법은 위와 똑같다.

// 2. ! 부정 논리 연산자를 두 번 사용하는 방법
// 문자열 타입 -> 불리언 타입
!!'x' // true
!!'' // false
!!false // true

// 숫자타입 -> 불리언 타입
!!0 // false
!!1 // true
!!NaN // false
!!Infinity // true

// null 타입 -> 불리언 타입
!!null // false

// undefined 타입 -> 불리언 타입
!!undefined // false

// 객체 타입 -> 불리언 타입
!!{} // true
!![] // true
```

# 4. 단축 평가

## 4.1 논리 연산자를 사용한 단축 평가

```jsx
'Cat' && 'Dog' // 'Dog'
```

논리곱(&&) 연산자는 두 개의 피연산자가 모두 true 로 평가될 때 true를 반환한다.

**논리곱 연산자는 논리 연산의 결과를 결정하는 두 번째 피연산자, 즉 문자열 ‘Dog’를 그대로 반환한다.**

```jsx
'Cat' || 'Dog' // 'Cat'
```

**논리합(||) 연산자는 논리 연산의 결과를 결정한 첫 번째 피연산자, 즉 문자열 ‘Cat’을 그대로 반환한다.**

단축 평가는 다음 규칙을 따른다.

| 단축 평가 표현식  | 평가 결과 |
| ----------------- | --------- |
| true              |           | anything | true |
| false             |           | anything | anything |
| true && anything  | anything  |
| false && anything | false     |

단축 평가를 사용하면 if 문을 대체할 수 있다.

조건이 Truthy 값일 때 논리곱(&&) 연산자 표현식으로 대체가 가능하다.

```jsx
var done = true;
var message = '';

// 주어진 조건이 true일 때
if (done) message = '완료';

// if 문은 단축 평가로 대체 가능하다.
// done이 true라면 message에 '완료'를 할당
message = done && '완료';
console.log(message); // 완료
```

조건이 Falsy 값일 때 논리합(||) 연산자 표현식으로 대체가 가능하다.

```jsx
var done = false;
var message = '';

// 주어진 조건이 false일 때
if (done) message = '미완료';

// if 문은 단축 평가로 대체 가능하다.
// done이 false라면 message에 '미완료'를 할당
message = done || '미완료';
console.log(message);
```

### 객체를 가리키기를 기대하는 변수가 null 또는 undefined가 아닌지 확인하고 프로퍼티를 참조할 때

객체는 키와 값으로 구성된 프로퍼티의 집합이다. 만약 객체를 가리키기를 기대하는 변수의 값이 객체가 아니라 null 또는 undefined인 경우 객체의 프로퍼티를 참조하면 타입 에러가 발생한다.

```jsx
var elem = null;
var value = elem.value; // TypeError
```

이때 단축 평가를 사용하면 에러를 발생시키지 않는다.

```jsx
var elem = null;
// elem이 null이나 undefined 값이라 false일 경우 elem으로 평가받고
// elem이 true값이면 elem.value로 평가된다.
var value = elem && elem.value; // null
```

### 함수 매개변수에 기본값을 설정할 때

함수를 호출할 때 인수를 전달하지 않으면 매개변수에는 undefined 가 할당된다.

단축 평가를 사용해 매개변수의 기본값을 설정하면 undefined 로 인해 발생할 수 있는 에러를 방지할 수 있다.

```jsx
// 단축 평가를 사용한 매개변수의 기본값 설정
function getStringLength(str) {
  str = str || '';
  return str.length;
}

getStringLength(); // 0
getStringLength('hi'); // 2

// ES6의 매개변수의 기본값 설정
function getStringLength(str = '') {
  return str.length;
}

getStringLength(); // 0
getStringLength('hi'); // 2
```

## 4.2 옵셔널 체이닝 연산자

ES11에서 도입된 옵셔널 체이닝 연산자 `?.` 는 좌항의 피연산자가 null 또는 undefined 인 경우 undefined 를 반환하고, 그렇지 않으면 우항의 프로퍼티 참조를 이어간다.

```jsx
var elem = null;

// elem이 null 또는 undefined 이면 undfined를 반환하고,
//그렇지 않으면 우항의 프로퍼티 참조를 이어간다.
var value = elem?.value;
console.log(value); // undefined
```

옵셔닝 체이닝 연산자 `?.` 는 객체를 가리키기를 기대하는 null 또는 undefined 가 아닌지 확인하고 프로퍼티를 참조할 때 유용하다.

옵셔닝 체이닝 연산자 `?.` 가 도입되기 이전에는 논리 연산자 && 를 사용한 단축 평가를 통해 변수가 null 또는 undefined 인지 확인했다.

논리 연산자 && 는 피연산자가 false로 평가되는 Falsy 값이면 좌항 피연산자를 그대로 반환한다.

좌항 피연산자가 Falsy 값인 0이나 ‘’인 경우도 마찬가지다. 하지만 0이나 ‘’은 객체로 평가될 때도 있다.

```jsx
var str = '';

// 문자열의 길이를 참조한다.
var length = str && str.length;

// 문자열의 길이를 참조하지 못한다.
console.log(length); // ''
```

하지만 옵셔닝 체이닝 연산자 `?.` 는 좌항 피연산자가 false로 평가되는 Falsy 값이라도 null 또는 undefined 가 아니면 우항의 프로퍼티 참조를 이어간다.

```jsx
var str = '';

var length = str?.length;
console.log(length); // 0
```

## 4.3 null 병합 연산자

ES11에서 도입된 null 병합 연산자 `??` 는 좌항의 피연산자가 null 또는 undefined 인 경우 우항의 피연산자를 반환하고, 그렇지 않으면 좌항의 피연산자를 반환한다.

null 병합 연산자 `??` 는 변수에 기본값을 설정할 때 유용하다.

```jsx
var foo = null ?? 'default string';
console.log(foo); // "default string"
```

null 병합 연산자 `??` 이전에는 논리 연산자 || 를 사용한 단축 평가를 통해 변수에 기본값을 설정했다.

Falsy 값인 0이나 ‘’ 기본값으로서 유효하다면 원하지 않은 동작이 발생할 수 있다.

```jsx
var foo = '' || 'default string';
console.log(foo) // default string";
```

null 병합 연산자 `??` 을 사용하면 Falsy 값이라도 그대로 반환한다.

```jsx
var foo = '' ?? 'default string';
console.log(foo); // ""
```
