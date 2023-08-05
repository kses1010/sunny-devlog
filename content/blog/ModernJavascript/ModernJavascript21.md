---
title: 'Modrn Javascript Deep Dive - 21장 빌트인 객체'
date: 2023-08-05
category: 'Javascript'
draft: false
---

# 1. 자바사크립트 객체의 분류

- 표준 빌트인 객체
- 호스트 객체
- 사용자 정의 객체

# 2. 표준 빌트인 객체

Math, Reflect, JSON을 제외한 표준 빌트인 객체는 모두 인스턴스를 생성할 수 있는 생성자 함수 객체다.

생성자 함수 객체인 표준 빌트인 객체는 프로토타입 메서드와 정적 메서드를 제공하고 생성자 함수 객체가 아닌 표준 빌트인 객체는 정적 메서드만 제공한다.

```jsx
// String 생성자 함수에 의한 String 객체 생성
const strObj = new String('Son') // String {"Son"}
console.log(typeof strObj) // object

// Number 생성자 함수에 의한 Number 객체 생성
const numObj = new Number(123) // Number {123}
console.log(typeof numObj) // object

// Boolean 생성자 함수에 의한 Boolean 객체 생성
const boolObj = new Boolean(true) // Boolean {true}
console.log(typeof boolObj) // object

// Function 생성자 함수에 의한 Function 객체(함수) 생성
const func = new Function('x', 'return x * x') // f anonymous(x)
console.log(typeof func) // function

// Array 생성자 함수에 의한 Array 객체(배열) 생성
const arr = new Array(1, 2, 3) // (3) [1, 2, 3]
console.log(typeof object) // object

// RegExp 생성자 함수에 의한 RegExp 객체(정규표현식) 생성
const regExp = new RegExp(/ab+c/i) // /ab+c/i
console.log(typeof regExp) // object

// Date 생성자 함수에 의한 Date 객체 생성
const date = new Date() // 표준시
console.log(typeof date) // object
```

생성자 함수인 표준 빌트인 객체가 생성한 인스턴스의 프로토타입은 표준 빌트인 객체의 prototype 프로퍼티에 바인딩된 객체다.

```jsx
const strObj = new String('Son') // String {"Son"}

// String 생성자 함수를 통해 생성한 strObj 객체의 프로토타입은 String.prototype이다.
console.log(Object.getProtoypeOf(strObj) === String.prototype) // true
```

표준 빌트인 객체의 prototype 프로퍼티에 바인딩된 객체는 다양한 기능의 빌트인 프로토타입 메서드를 제공한다.

표준 빌트인 객체는 인스턴스 없이도 호출 가능한 빌트인 정적 메서드를 제공한다.

```jsx
const numObj = new Number(1.5)

// toFixed는 Number.prototype 의 프로토타입 메서드다.
// Number.prototype.toFixed는 소수점 사리를 반올림하여 문자열로 반환한다.
console.log(numObj.toFixed()) // 2

// isInteger는 Number의 정적 메서드다.
// Number.isInteger는 인수가 정수(integer)인지 검사하여 그 결과를 Boolean으로 반환한다.
console.log(Number.isInteger(0.5)) // false
```

# 3. 원시값과 래퍼 객체

원시값은 객체가 아니므로 프로퍼티나 메서드를 가질 수 없는데도 원시값인 문자열이 마치 객체처럼 동작한다.

```jsx
const str = 'hello'

// 원시 타입인 문자열이 프로퍼티와 메서드를 갖고 있는 객체처럼 동작한다.
console.log(str.length) // 5
console.log(str.toUpperCase()) // HELLO
```

자바스크립트 엔진이 일시적으로 원시값을 연관된 객체로 변환해준다.

**문자열, 숫자, 불리언 값에 대해 객체처럼 접근하면 생성되는 임시 객체를 래퍼 객체라 한다.**

```jsx
const str = 'hello'

// 원시 타입인 문자열이 래퍼 객체인 String 인스턴스로 변환된다.
console.log(str.length) // 5
console.log(str.toUpperCase()) // HELLO

// 래퍼 객체로 프로퍼티에 접근하거나 메서드를 호출한 후, 다시 원시값으로 되돌린다.
console.log(typeof str) // string
```

이때 문자열 래퍼 객체인 String 생성자 함수의 인스턴스는 String.prototype 의 메서드를 상속받아 사용할 수 있다.

→ 객체 처리가 완료되면 해당 식별자는 원시값을 갖도록 되돌리고 래퍼 객체는 가비지 컬렉션의 대상이 된다.

ES6에서 새롭게 도입된 원시값인 심벌도 래퍼 객체를 생성한다. 심벌은 일반적인 원시값과는 달리 리터럴 표기법으로 생성할 수 없고 Symbol 함수를 통해 생성해야 하므로 다른 원시값과는 차이가 있다.

문자열, 숫자, 불리언, 심벌은 암묵적으로 생성되는 래퍼 객체에 의해 마치 객체처럼 사용할 수 있으며, 표준 빌트인 객체인 String, Number, Boolean, Symbol의 프로토타입 메서드 또는 프로퍼티를 참조할 수 있다.

→ String, Number, Boolean 생성자 함수를 new 연산자와 함께 호출하여 문자열, 숫자, 불리언 인스턴스를 생성할 필요가 없으며 권장하지는 않는다.

null과 undefined는 래퍼 객체를 생성하지 않는다. → 객체처럼 사용하면 에러가 발생한다.

# 4. 전역 객체

전역 객체는 코드가 실행되기 이전 단계에 자바스크립트 엔진에 의해 어떤 객체보다도 먼저 생성되는 특수한 객체이며, 어떤 객체에서도 속하지 않은 최상위 객체다.

전역 객체는 자바스크립트 환경에 따라 지칭하는 이름이 제각각이다.

- 브라우저: window or self, this, frames
- Node.js: global

전역 객체는 계층적 구조상 어떤 객체에도 속하지 않은 모든 빌트인 객체의 최상위 객체다.

- 전역 객체는 개발자가 의도적으로 생성할 수 없다. → 생성자 함수가 제공되지 않음
- 전역 객체의 프로퍼티를 참조할 때 window(global)를 생략할 수 있다.

```jsx
// 'F'를 16진수로 해석하여 10진수로 변환
window.parseInt('F', 16) // 15
parseInt('F', 16) // 15

windows.parseInt === parseInt //true
```

```jsx
// var 키워드로 선언한 전역 변수
var foo = 1
console.log(window.foo) // 1

// 선언하지 않은 변수에 값을 암묵적 전역 처리
bar = 2 // window.bar = 2
console.log(window.bar) // 2

// 전역 함수
function baz() {
  return 3
}
console.log(window.baz()) // 3
```

let 이나 const 키워드로 선언한 전역 변수는 전역 객체의 프로퍼티가 아니다. window.foo 처럼 접근할 수 없다.

```jsx
let foo = 123
console.log(window.foo) // undefined
```

## 4.1 빌트인 전역 프로퍼티

빌트인 전역 프로퍼티는 전역 객체의 프로퍼티를 의미한다. 주로 애플리케이션 전역에서 사용하는 값을 제공한다.

### Infinity

Infinity 프로퍼티는 무한대를 나타내는 숫자값 `Infinity` 를 갖는다.

```jsx
// 양의 무한대
console.log(3 / 0) // Infinity
// 음의 무한대
console.log(-3 / 0) // -Infinity
// Infinity는 숫자값이다.
console.log(typeof Infinity) // number
```

### NaN

NaN 프로퍼티는 숫자가 아님(Not-a-Number)을 나타내는 숫자값 NaN을 갖는다.

```jsx
console.log(Number('xyz')) // NaN
console.log(1 * 'string') // NaN
console.log(typeof NaN) // number
```

### undefined

undefined 프로퍼티는 원시 타입 undefind를 값으로 갖는다.

```jsx
var foo
console.log(foo) // undefined
console.log(typeof undefined) // undefined
```

## 4.2 빌트인 전역 함수

빌트인 전역 함수는 애플리케이션 전역에서 호출할 수 있는 빌트인 함수로서 전역 객체의 메서드다.

### eval

eval 함수는 자바스크립트 코드를 나타내는 문자열을 인수로 받는다.

```jsx
// 표현식인 문
eval('1 + 2;') // 3
// 표현식이 아닌 문
eval('var x = 5;') // undefined

// eval 함수에 의해 런타임에 변수 선언문이 실행되어 x 변수가 선언되었다.
console.log(x) // 5

// 객체 리터럴은 반드시 괄호로 둘러싼다.
const o = eval('({a:})')
console.log(o) // {a:1}

// 함수 리터럴은 반드시 괄호로 둘러싼다.
const f = eval('(function() { return 1;})')
console.log(f()) // 1
```

인수로 전달받은 문자열 코드가 여러 개의 문으로 이루어져 있다면 모든 문을 실행한 다음, 마지막 결과값을 반환한다.

```jsx
eval('1 + 2; 3 + 4;') // 7
```

eval 함수는 자신이 호출된 위치에 해당하는 기존의 스코프를 런타임에 동적으로 수정한다.

```jsx
const x = 1

function foo() {
  // eval 함수는 런타임에 foo 함수의 스코프를 동적으로 수정한다.
  eval('var x = 2;')
  console.log(x) // 2
}

foo()
console.log(x) // 1
```

**eval 함수는 기존의 스코프를 동적으로 수정한다.**

strict mode 에서 eval 함수는 기존의 스코프를 수정하지 않고 eval 함수 자신의 자체적인 스코프를 생성한다.

```jsx
const x = 1

function foo() {
  'use strict'

  eval('var x = 2; console.log(x);') // 2
  console.log(x) // 1
}

foo()
console.log(x) // 1
```

인수로 전달받은 문자열 코드가 let, const 키워드를 사용한 변수 선언문이라면 암묵적으로 strict mode가 적용된다.

```jsx
const x = 1

function foo() {
  eval('var x = 2; console.log(x);') // 2
  // let, const 선언문은 strict mode가 적용된다.
  eval('const x = 3; console.log(x);') // 3
  console.log(x) // 2
}

foo()
console.log(x) // 1
```

eval 함수를 통해 사용자로부터 입력받은 콘텐츠를 실행하는 것은 보안에 매우 취약하다.

eval 함수를 통해 실행되는 코드는 자바스크립트 엔진에 의해 최적화가 수행되지 않으므로 일반적인 코드 실행에 비해 처리 속도가 느리다. **→ eval 함수의 사용은 금지해야 한다.**

### isFinite

전달받은 인수가 정상적인 유한수인지 검사하여 유한수이면 true를 반환하고, 무한수이면 false를 반환한다.

```jsx
// 인수가 유한수이면 true를 반환한다.
isFinite(0) // true
isFinite(2e64) // true
isFinite('10') // true "10" -> 10
isFinite(null) // true null -> 0

// 인수가 무한수 또는 NaN 평가되는 값이면 false를 반환한다.
isFinite(Infinity) // false
isFinite(NaN) // false
isFinite('Hello') // false
isFinite('2005/12/25') // false
```

### isNaN

전달받은 인수가 NaN인지 검사하여 그 결과를 불리언 타입으로 반환한다.

```jsx
// 숫자
isNaN(NaN) // true
isNaN(10) // false

// 문자열
isNaN('blabla') // true : "blabla" -> NaN
isNaN('10') // false
isNaN('10.12') // false
isNaN('') // false : 0
isNaN(' ') // false : 0

// 불리언
isNaN(true) // false : true -> 1
isNaN(null) // false : false -> 0

// undefined
isNaN(undefined) // true : undefined -> NaN

// 객체
isNaN({}) // true: {} -> NaN

// date
isNaN(new Date()) // false : new Date() -> Number
isNaN(new Date().toString()) // true : String -> NaN
```

### parseFloat

전달받은 문자열 인수를 부동 소수점 숫자, 실수로 해석하여 반환한다.

```jsx
// 문자열을 실수로 해석하여 반환한다.
parseFloat('3.14') // 3.14
parseFloat('10.00') // 10

// 공백으로 구분된 문자열은 첫 번째 문자열만 반환
parseFloat('34 45 66') // 34
parseFloat('40 years') // 40

// 첫 번째 문자열을 숫자로 변환할 수 없다면 NaN 반환
parseFloat('He was 40') // NaN

// 앞뒤 공백은 무시
parseFloat(' 60 ') // 60
```

### parseInt

전달받은 문자열 인수를 정수로 해석

```jsx
// 문자열을 정수로 해석
parseInt('10') // 10
parseInt('10.123') // 10

// 인수가 문자열이 아니면 무낮열로 변환한 다음, 정수로 해석하여 반환
parseInt(10) // 10
parseInt(10.123) // 10

// 10을 10진수로 해석 -> 그 결과를 10진수 정수로 반환
parseInt('10') // 10
// 10을 2진수로 해석 -> 그 결과를 10진수 정수로 반환
parseInt('10', 2) // 2
// 10을 8진수로 해석 -> 그 결과를 10진수 정수로 반환
parseInt('10', 8) // 8
// 10을 16진수로 해석 -> 그 결과를 10진수 정수로 반환
parseInt('10', 16) // 16
```

기수를 지정하여 10진수 숫자를 해당 기수의 문자열로 변환하여 반환하고 싶을 때는 `Number.prototype.toString` 메서드를 사용한다.

```jsx
const x = 15

// 10진수 15를 2진수로 변환하여 문자열로 반환
x.toString(2) // '1111'
parseInt(x.toString(2), 2) // 15

// 10진수 15를 8진수로 변환하여 문자열로 반환
x.toString(8) // '17'
parseInt(x.toString(8), 8) // 15

// // 10진수 15를 16진수로 변환하여 문자열로 반환
x.toString(16) // 'f'
parseInt(x.toString(16), 16) // 16

// 숫자값을 문자열로 변환
x.toString() // '15'
parseInt(x.toString()) // 15
```

인수로 진법을 나타내는 기수를 지정하지 않더라도 첫 번째 인수로 전달된 문자열이 “0x” or “0X” 로 시작하는 16진수 리털이라면 16진수로 해석하여 10진수 정수로 반환한다.

```jsx
parseInt('0xf') // 15
parseInt('f', 16) // 15
```

그러나 2진수, 8진수는 제대로 해석하지 못한다.

```jsx
// 2진수
parseInt('0b10') // 0
// 8진수
parseInt('0o10') // 0
```

첫 번째 인수로 전달한 문자열의 첫 번째 문자가 해당 지수의 숫자로 변환될 수 없다면 NaN을 반환한다.

```jsx
// "A"는 10진수로 해석할 수 없다.
parseInt('A0') // NaN
// "2"는 2진수로 해석할 수 없다.
parseInt('20', 2) // NaN
```

첫 번째 인수로 전달한 문자열의 두 번째 문자부터 해당 진수를 나타내는 숫자가 아닌 문자와 마주치면 이 문자와 계속되는 문자들은 전부 무시되며 해석된 정수값만 반환한다.

```jsx
// 10진수로 해석할 수 없는 "A" 이후의 문자는 모두 무시된다.
parseInt('1A0') // 1
// 2진수로 해석할 수 없는 "2" 이후의 문자는 모두 무시된다.
parseInt('102', 2) // 2
// 8진수로 해석할 수 없는 "8" 이후의 문자는 모두 무시된다.
parseInt('58', 8) // 5
// 16진수로 해석할 수 없는 "G" 이후의 문자는 모두 무시된다.
parseInt('FG', 16) // 15
```

### encodeURI / decodeURI

encodeURI 함수는 완전한 URI를 문자열로 전달받아 이스케이프 처리를 위해 인코딩한다.

decodeURI 함수는 인코딩한 URI를 인수로 전달받아 이스케이프 처리 이전으로 디코딩한다.

```jsx
// 완전한 URI
const uri = 'http://example.com?name=손&job=developer'
// encodeURI 함수는 완전한 URI를 전달받아 이스케이프 처리를 위해 인코딩한다.
const enc = encodeURI(uri)
console.log(enc)
// http://example.com?name=%EC%86%90&job=developer

// decodeURI 함수는 이스케이프 ㅇ처리 이전으로 디코딩한다.
const dec = decodeURI(uri)
console.log(dec)
// "http://example.com?name=손&job=developer"
```

## 4.3 암묵적 전역

```jsx
var x = 10 // 전역 변수

function foo() {
  // 선언하지 않은 식별자에 값을 할당
  y = 20 // window.y = 20;
}

foo()

// 선언하지 않은 식별자 y를 전역에서 참조할 수 있다.
console.log(x + y) // 30
```

```jsx
// 전역 변수는 x는 호이스팅이 발생한다.
console.log(x) // undefined
// 전역 변수가 아니라 단지 전역 객체의 프로퍼티인 y는 호이스팅이 발생하지 않음
console.log(y) // ReferenceError

var x = 10 // 전역 변수

function foo() {
  // 선언하지 않은 식별자에 값을 할당
  y = 20 // window.y = 20;
}
foo()

// 선언하지 않은 식별자 y를 전역에서 참조할 수 있다.
console.log(x + y) // 30
```

단지 프로퍼티인 y는 delete 연산자로 삭제할 수 있다. 전역 변수는 프로퍼티이지만 delete 연산자로 삭제할 수 없다.

```jsx
var x = 10 // 전역 변수

function foo() {
  // 선언하지 않은 식별자에 값을 할당
  y = 20 // window.y = 20;
  console.log(x + y)
}
foo()

console.log(x) // 10
console.log(y) // 20

delete x // 전역 변수는 삭제되지 않는다.
delete y // 삭제

console.log(x) // 10
console.log(y) // undefined
```
