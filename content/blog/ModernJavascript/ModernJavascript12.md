---
title: 'Modrn Javascript Deep Dive - 12장 함수'
date: 2023-08-01
category: 'Javascript'
draft: false
---

# 1. 함수란?

```jsx
// f(x, y) = x + y
function add(x, y) {
  return x + y
}

// f(2, 5) = 7
add(2, 5) // 7
```

**함수는 일련의 과정을 문(statement)으로 구현하고 코드 블록으로 감싸서 하나의 실행 단위로 정의한 것.**

함수는 내부로 입력을 전달받는 변수를 **매개변수(parameter)**, 입력을 **인수(argument)**, 출력을 **반환값(return value)** 이라 한다.

함수는 함수 정의를 통해 생성한다.

```jsx
// 함수 정의
function add(x, y) {
  return x + y
}
```

함수를 호출하면 코드 블록에 담긴 문들이 일괄적으로 실행되고 실행 결과, 즉 반환값을 반환한다.

```jsx
// 함수 호출
var result = add(2, 5)

// 함수 add에 인수 2, 5를 전달하면서 호출하면 반환값 7을 반환한다.
console.log(result) // 7
```

# 2. 함수를 사용하는 이유

함수는 몇 번이든 호출할 수 있으므로 **코드의 재사용**이 용이하다.

- 유지보수의 편의성을 높임
- 코드의 신뢰성을 높임
- 코드의 가독성 향상

# 3. 함수 리터럴

함수 리터럴은 function 키워드, 함수 이름, 매개변수 목록, 함수 몸체로 구성된다.

```jsx
// 변수에 함수 리터럴을 할당
var f = function add(x, y) {
  return x + y
}
```

함수 리터럴의 구성 요소는 다음과 같다.

| 구성 요소 | 설명                                                       |
| --------- | ---------------------------------------------------------- |
| 함수 이름 | - 함수 이름은 식별자다. 식별자 네이밍 규칙을 준수해야한다. |

- 함수 이름은 함수 몸체 내에서만 참조할 수 있는 식별자다.
- 함수 이름은 생략할 수 있다. 이름이 없는 함수를 익명함수(annonymous function)이라 한다. |
  | 매개변수 목록 | - 0개 이상의 매개변수를 소괄호로 감싸고 쉼표로 구분한다.
- 각 매개변수에는 함수를 호출할 때 지정한 인수가 순서대로 할당한다. 즉, 매개변수 목록은 순서에 의미가 있다.
- 매개변수는 함수 몸체내에서 변수와 동일하게 취급된다. → 매개변수도 변수와 마찬가지로 식별자 네이밍 규칙을 준수해야한다. |
  | 함수 몸체 | - 함수가 호출되었을 때 일괄적으로 실행될 문들을 하나의 실행 단위로 정의한 코드 블록이다.
- 함수 몸체는 함수 호출에 의해 실행된다. |

함수 리터럴도 평가되 값을 생성하며, 이 값은 객체다.

**→ 함수는 객체다.**

**일반 객체는 함수는 호출할 수 있다. 함수 객체만의 고유한 프로퍼티를 갖는다.**

# 4. 함수 정의

함수를 정의하는 방식은 4가지가 있다.

## 4.1 함수 선언문

```jsx
// 함수 선언문
function add(x, y) {
  return x + y
}

// 함수 참조
console.dir(add) // [Function: add]

// 함수 호출
console.log(add(2, 5)) // 7
```

**함수 선언문은 함수 이름을 생략할 수 없다.**

```jsx
function (x, y) {
	return x + y;
}

// SyntaxError
```

**함수 선언문은 표현식이 아닌 문이다.**

표현식이 아닌 문은 변수에 할당 할 수 없다. 다음 코드에서는 할당되는 것처럼 보인다.

```jsx
var add = function add(x, y) {
  return x + y
}

console.log(add(2, 5))
```

자바스크립트 엔진이 코드의 문맥에 따라 동일한 함수 리터럴을 표현식이 아닌 문인 함수 선언문으로 해석하는 경우와 표현식인 문인 함수 리터럴 표현식으로 해석하는 경우가 있기 때문이다.

→ 코드의 문맥에 따라 해석이 달라질 수 있다.

```jsx
// 기명 함수 리터럴을 단독으로 사용하면 함수 선언문으로 해석된다.
// 함수 선언문에서는 함수 이름을 생략할 수 있다.
function foo() {
  console.log('foo')
}

foo() // foo

// 함수 리터럴을 피연산자로 사용하면 함수 선언문이 아니라 함수 리터럴 표현식으로 해석된다.
// 함수 리터럴에서는 함수 이름을 생략할 수 있다.
;(function bar() {
  console.log('bar')
})
bar() // ReferenceError
```

**자바스크립트 엔진은 생성된 함수를 호출하기 위해 함수 이름과 동일한 이름의 식별자를 암묵적으로 생성하고, 거기에 함수 객체를 할당한다.**

```jsx
// add: 식별자 // add(): 함수이름
var add = function add(x, y) {
  return x + y
}

// 여기의 add는 식별자
console.log(add(2, 5))
```

**함수는 함수 이름으로 호출하는 것이 아니라 함수 객체를 가리키는 식별자로 호출한다.**

## 4.2 함수 표현식

값의 성질을 갖는 객체를 **일급 객체**라 하며 **자바스크립트의 함수는 일급 객체다.**

함수가 일급 객체라는 것은 함수를 값처럼 자유롭게 사용할 수 있다는 의미다.

```jsx
// 함수 표현식
var add = function(x, y) {
  return x + y
}

console.log(add(2, 5)) // 7
```

함수 리터럴의 함수 이름은 생략할 수 있다. 이러한 함수를 익명함수라 한다.

함수 이름은 함수 몸체 내부에서만 유효한 식별자이므로 함수 이름으로 함수를 호출할 수 없다.

```jsx
var add = function foo(x, y) {
  return x + y
}

// 식별자로 호출
console.log(add(2, 5))

// 함수 이름으로 호출하면 ReferenceError가 발생한다.
console.log(foo(2, 5))
```

## 4.3 함수 생성 시점과 함수 호이스팅

```jsx
// 함수 참조
console.dir(add) // [Function: add]
console.dir(sub) // undefined

// 함수 호출
console.log(add(2, 5)) // 7
console.log(sub(2, 5)) // TypeError

// 함수 선언문
function add(x, y) {
  return x + y
}

// 함수 표현식
var sub = function(x, y) {
  return x - y
}
```

함수 선언문으로 정의한 함수는 함수 선언문 이전에 호출할 수 있다. 그러나 함수 표현식으로 정의한 함수는 함수 표현식 이전에 호출할 수 없다.

**→ 함수 선언문으로 정의한 함수와 함수 표현식으로 정의한 함수의 생성 시점이 다르기 때문이다.**

함수 선언문으로 함수를 정의하면 런타임 이전에 함수 객체가 먼저 생성된다. 그리고 자바스크립트 엔진은 함수 이름과 동일한 이름의 식별자를 암묵적으로 생성하고 생성된 함수 객체를 할당한다.

**→ 함수 선언문이 코드의 선두로 끌어 올려진 것처럼 동작하는 자바스크립트 고유의 특징을 함수 호이스팅(function hoisting)이라 한다.**

변수 할당문의 값은 할당문이 실행되는 시점, 즉 런타임에 평가되므로 함수 표현식의 함수 리터럴도 할당문이 실행되는 시점에 평가되어 함수 객체가 된다.

**→ 함수 표현식으로 함수를 정의하면 함수 호이스팅이 아니라 변수 호이스팅이 발생한다.**

## 4.4 Function 생성자 함수

```jsx
var add = new Function('x', 'y', 'return x + y')
console.log(add(2, 5))
```

Function 생성자 함수로 함수를 생성하는 방식은 일반적이지 않으며 바람직하지도 않다.

Function 생성자 함수로 생성한 함수는 클로저를 생성하지 않는 등, 함수 선언문이나 함수 표현식으로 생성한 함수와 다르게 동작한다.

```jsx
var add1 = (function() {
  var a = 10
  return function(x, y) {
    return x + y + a
  }
})()

console.log(add1(1, 2)) // 13

var add2 = (function() {
  var a = 10
  return new Function('x', 'y', 'return x + y + a;')
})()

console.log(add2(1, 2)) // ReferenceError
```

## 4.5 화살표 함수

ES6에서 도입된 화살표 함수는 function 키워드 대신 화살표(`⇒`)를 사용해 좀 더 간략한 방법으로 함수를 선언할 수 있다. 화살표 함수는 항상 익명 함수로 정의한다.

```jsx
// 화살표 함수
const add = (x, y) => x + y
console.log(add(2, 5)) // 7
```

# 5. 함수 호출

## 5.1 매개변수와 인수

함수를 실행하기 위해 필요한 값을 함수 외부에서 함수 내부로 전달할 필요가 있는 경우, 매개변수를 통해 인수를 전달한다. 인수는 값으로 평가될 수 있는 표현식이어야 한다.

인수는 함수를 호출할 때 지정하며, 개수와 타입에 제한이 없다.

```jsx
// 함수 선언문
function add(x, y) {
  return x + y
}

// 함수 호출
// 인수 1과 2가 매개변수 x와 y에 순서대로 할당되고 함수 몸체의 문들이 실행된다.
var result = add(1, 2)
```

매개변수의 스코프는 함수 내부다.

```jsx
function add(x, y) {
  console.log(x, y) // 2 5
  return x + y
}

add(2, 5)

// add 함수의 매개변수 x, y는 함수 몸체 내부에서만 참조할 수 있다.
console.log(x, y) // ReferenceError
```

함수는 매개변수의 개수와 인수의 개수가 일치하는지 체크하지 않는다.

인수가 부족해서 인수가 할당되지 않은 매개변수의 값은 undefined다.

```jsx
function add(x, y) {
  return x + y
}

console.log(add(2)) // NaN
```

매개변수보다 인수가 더 많은 경우 초과된 인수는 무시된다.

```jsx
function add(x, y) {
  return x + y
}

console.log(add(2, 5, 10)) // 7
```

사실 초과된 인수가 그냥 버려지는 것은 아니다.

모든 인수는 암묵적으로 arguments 객체의 프로퍼티로 보관된다.

```jsx
function add(x, y) {
  console.log(arguments) // [Arguments] { '0': 2, '1': 5, '2': 10 }
  return x + y
}

console.log(add(2, 5, 10))
```

arguments 객체는 함수를 정의할 때 매개변수 개수를 확정할 수 없는 가변 인자 함수를 구현할 때 유용하게 사용된다.

## 5.2 인수 확인

```jsx
function add(x, y) {
  return x + y
}

console.log(add(2)) // NaN
console.log(add('a', 'b')) // 'ab'
```

자바스크립트 문법상 어떠한 문제도 없으므로 자바스크립트 엔진은 아무런 이의 제기없이 위 코드를 실행한다.

- 자바스크립트 함수는 매개변수와 인수의 개수가 일치하는지 확인하지 않는다.
- 자바스크립트는 동적 타입 언어다. → 자바스크립트 함수는 매개변수의 타입을 사전에 지정할 수 없다.

→ 자바스크립트의 경우 함수를 정의할 때 적절한 인수가 전달되었는지 확인할 필요가 있다.

```jsx
function add(x, y) {
  if (typeof x !== 'number' || typeof y !== 'number') {
    // 매개변수를 통해 전달된 인수의 타입이 부적절한 경우 에러를 발생시킨다.
    throw new TypeError('인수는 모두 숫자 값이어야 합니다.')
  }

  return x + y
}

console.log(add(2)) // TypeError: 인수는 모두 숫자 값이어야 합니다.
console.log(add('a', 'b')) // TypeError: 인수는 모두 숫자 값이어야 합니다.
```

arguments 객체를 통해 인수 개수를 확인할 수 도 있다.

```jsx
function add(a, b, c) {
  a = a || 0
  b = b || 0
  c = c || 0
  return a + b + c
}

console.log(add(1, 2, 3)) // 6
console.log(add(1, 2)) // 3
console.log(add(1)) // 1
console.log(add()) // 0
```

ES6에서 도입된 매개변수 기본값을 사용하면 함수 내에서 수행하던 인수 체크 및 초기화를 간소화할 수 있다.

```jsx
function add(a = 0, b = 0, c = 0) {
  return a + b + c
}

console.log(add(1, 2, 3)) // 6
console.log(add(1, 2)) // 3
console.log(add(1)) // 1
console.log(add()) // 0
```

## 5.3 매개변수의 최대 개수

매개변수의 개수가 많다는 것은 함수가 여러 가지 일을 한다는 증거이므로 바람직하지 않다.

**이상적인 함수는 한 가지 일만 해야 하며 가급적 작게 만들어야 한다.**

→ 매개변수는 최대 3개 이상을 넘지 않는 것이 좋다.

## 5.4 반환문

함수는 return 키워드와 표현식(반환값)으로 이뤄진 반환문을 사용해 실행 결과를 함수 외부로 반환할 수 있다.

```jsx
function multiply(x, y) {
  return x * y // 반환문
}

// 함수 호출은 반환값으로 평가된다.
var result = multiply(3, 5)
console.log(result) // 15
```

반환문은 두 가지 역할을 한다.

- 반환문은 함수의 실행을 중단하고 함수 몸체를 빠져나간다.

```jsx
function multiply(x, y) {
  return x * y // 반환문
  // 반환문 이후에 다른 문이 존재하면 그 문은 실행되지 않고 무시한다.
  console.log('실행되지 않음')
}

console.log(multiply(3, 5)) // 15
```

- 반환문은 return 키워드 뒤에 오는 표현식을 평가해 반환한다. return 키워드 뒤에 반환값으로 사용할 표현식을 명시하지 않으면 undefined가 반환한다.

```jsx
function foo() {
  return
}

console.log(foo()) // undefined
```

반환문은 생략이 가능하다. 함수는 함수 몸체의 마지막 문까지 실행한 후 암묵적으로 undefined를 반환한다.

```jsx
function foo() {}

console.log(foo()) // undefined
```

return 키워드와 반환값으로 사용할 표현식 사이에 줄바꿈이 있으면 세미콜론 자동 삽입이 된다.

```jsx
function multiply(x, y) {
  return // 세미콜론이 자동 삽입이 되어 return 문이 실행된다.
  x + y // 무시된다.
}

console.log(multiply(3, 5)) // undefined
```

# 6. 참조에 의한 전달과 외부 상태의 변경

```jsx
// 매개변수 primitive는 원시 값을 전달받고, 매개변수 obj는 객체를 전달받는다.
function changeVal(primitive, obj) {
  primitive += 100
  obj.name = 'Son'
}

// 외부 상태
var num = 100
var person = { name: 'sunny' }

console.log(num) // 100
console.log(person) // {name: "sunny"}

// 원시 값은 값 자체가 복사되어 전달되고 객체는 참조 값이 복사되어 전달된다.
changeVal(num, person)

// 원시 값은 원본이 훼손되지 않는다.
console.log(num) // 100

// 객체는 원본이 훼손된다.
console.log(person) // {name: "Son"}
```

외부 참조를 통해 객체의 상태 훼손을 막기위해서는 객체를 불변 객체로 만들어 사용하는 것이다.

객체의 복사본을 새롭게 생성하는 비용은 들지만 객체를 마치 원시 값처럼 변경 불가능한 값으로 동작하게 만드는 것이다. 이를 통해 객체의 상태 변경을 원천봉쇄하고 객체의 상태 변경이 필요한 경우에는 객체의 방어적 복사를 통해 원본 객체를 완전히 복제 → 깊은 복사를 통해 새로운 객체를 생성하고 재할당을 통해 교체한다.

외부 상태를 변경하지 않고 외부 상태에 의존하지도 않는 함수를 순수 함수라 한다.

# 7. 다양한 함수의 형태

## 7.1 즉시 실행 함수

함수 정의와 동시에 즉시 호출되는 함수를 **즉시 실행 함수** 한다.

```jsx
// 익명 즉시 실행 함수
;(function() {
  var a = 3
  var b = 5
  return a * b
})()
```

즉시 실행 함수는 함수 이름이 없는 익명 함수를 사용하는 것이 일반적이다.

그룹 연산자(…) 내의 기명 함수는 함수 선언문이 아니라 함수 리터럴로 평가되며 함수 이름은 함수 몸체에서만 참조할 수 있는 식별자이므로 즉시 실행 함수를 다시 호출할 수 없다.

```jsx
// 익명 즉시 실행 함수
;(function foo() {
  var a = 3
  var b = 5
  return a * b
})()

foo() // ReferenceError
```

즉시 실행 함수도 일반 함수처럼 값을 반환할 수 있고 인수를 전달할 수도 있다.

```jsx
// 즉시 실행 함수도 일반 함수처럼 값을 반환할 수 있다.
var res = (function () {
	var a = 3;
	var b = 5;
	return a * b;
})();

console.log(res); // 15

// // 즉시 실행 함수도 일반 함수처럼 인수를 전달할 수 있다.
res = (fucntion (a, b) {
    return a * b;
}(3, 5));

console.log(res); // 15
```

## 7.2 재귀 함수

함수가 자기 자신을 호출하는 것을 재귀 호출이라 한다. 재귀 함수는 자기 자신을 호출하는 행위, 즉 재귀 호출을 수행하는 함수를 말한다.

```jsx
function countdown(n) {
  for (var i = n; i >= 0; i--) console.log(i)
}
countdown(10)

// 재귀 호출
function countdown(n) {
  if (n < 0) return
  console.log(n)
  countdown(n - 1) // 재귀 호출
}
countdown(10)
```

## 7.3 중첩 함수

함수 내부에 정의된 함수를 중첩 함수(nested function) 또는 내부 함수(inner function)라 한다. 중첩 함수를 포함하는 함수는 외부 함수(outer function)라 한다.

일반적으로 중첩 함수는 자신을 포함하는 외부 함수를 돕는 헬퍼 함수(helper function)의 역할을 한다.

```jsx
function outer() {
  var x = 1

  // 중첩 함수
  function inner() {
    var y = 2
    // 외부 함수의 변수를 참조할 수 있다.
    console.log(x + y) // 3
  }

  inner()
}

outer()
```

ES6부터 함수 정의는 문이 위치할 수 있는 문맥이라면 어디든 지 가능하다. ES6부터는 if 문이나 for 문 등의 코드 블록 내에서도 정의할 수 있다.

단, 호이스팅으로 인해 혼란이 발생할 수 있으므로 if 문이나 for 문 등의 코드 블록에서 함수 선언문을 통해 함수를 정의하는것은 바람직하지 않다.

## 7.4 콜백 함수

어떤 일을 반복 수행하는 repeat 함수를 정의를 합니다.

```jsx
// n만큼 어떤 일을 반복한다.
function repeat(n) {
  // i를 출력한다.
  for (var i = 0; i < n; i++) console.log(i)
}

repeat(5)
```

repeat 함수는 `console.log(i)` 에 강하게 의존하고 있어 다른 일을 할 수 없다.

→ 만약 repeat 함수의 반복문 내부에서 다른 일을 하고 싶다면 함수를 새롭게 정의해야 한다.

```jsx
// n만큼 어떤 일을 반복한다.
function repeat(n) {
  // i를 출력한다.
  for (var i = 0; i < n; i++) console.log(i)
}

repeat(5)

function repeat2(n) {
  for (var i = 0; i < n; i++) {
    // i가 홀수만 출력한다.
    if (i % 2) console.log(i)
  }
}

repeat2(5) // 1 3
```

함수들은 반복하는 일은 변하지 않고 공통적으로 수행하지만 반복하면서 하는 일의 내용은 다르다.

→ 함수의 일부분만이 다르기 때문에 매번 함수를 새롭게 정의해야 한다. 이 문제는 함수를 합성하는 것으로 해결할 수 있다.

함수의 변하지 않는 공통 로직은 미리 정의해 두고, 경우에 따라 변경되는 로직은 추상화해서 함수 외부에서 함수 내부로 전달하는 것이다.

```jsx
// 외부에서 전달받은 f를 n만큼 반복 호출한다.
function repeat(n, f) {
  for (var i = 0; i < n; i++) {
    f(i) // i를 전달하면서 f를 호출
  }
}

var logAll = function(i) {
  console.log(i)
}

// 반복 호출할 함수를 인수로 전달한다.
repeat(5, logAll) // 0 1 2 3 4

var logOdds = function(i) {
  if (i % 2) console.log(i)
}

// 반복 호출할 함수를 인수로 전달한다.
repeat(5, logOdds) // 1 3
```

**함수의 매개변수를 통해 다른 함수의 내부로 전달되는 함수를 콜백 함수(callback function)라고 하며, 매개변수를 통해 함수의 외부에서 콜백 함수를 전달받은 함수를 고차 함수(Higher-Order Function, HOF)라고 한다.**

고차 함수는 콜백 함수를 자신의 일부분으로 합성한다.

고차 함수는 매개변수를 통해 전달받은 콜백 함수의 호출 시점을 결정해서 호출한다.

**→ 콜백 함수는 고차 함수에 의해 호출되며 이때 고차 함수는 필요에 따라 콜백 함수에 인수를 전달할 수 있다.**

따라서 고차 함수에 콜백 함수를 전달할 때 콜백 함수를 호출하지 않고 함수 자체를 전달해야 한다.

```jsx
// 익명 함수 리터럴을 콜백 함수로 고차 함수에 전달한다.
// 익명 함수 리터럴은 repeat 함수를 호출할 때마다 평가되어 함수 객체를 생성한다.
repeat(5, function(i) {
  if (i % 2) console.log(i) // 1 3
})
```

콜백 함수를 다른 곳에서도 호출할 필요가 있거나, 콜백 함수를 전달받는 함수가 자주 호출된다면 함수 외부에서 콜백 함수를 정의한 후 함수 참조를 고차 함수에 전달하는 편이 효율적이다.

```jsx
// logOdds 함수는 단 한 번만 생성된다.
var logOdds = function(i) {
  if (i % 2) console.log(i)
}

// 고차 함수에 함수 참조를 전달한다.
repeat(5, logOdds)
```

위 예제는 logOdds 함수는 단 한 번만 생성된다. 하지만 콜백 함수를 익명 함수 리터럴로 정의하면서 곧바로 고차 함수에 전달하면 고차 함수가 호출될 때마다 콜백 함수가 생성된다.

콜백 함수는 비동기 처리뿐 아니라 배열 고차 함수에서도 사용된다. 자바스크립트에서 배열은 사용 빈도가 매우 높은 자료구조이고 배열을 다룰 때 배열 고차 함수는 매우 중요하다.

```jsx
// 콜백 함수를 사용하는 고차 함수 map
var res = [1, 2, 3].map(function(item) {
  return item * 2
})

console.log(res) // [2, 4, 6]

// 콜백 함수를 사용하는 고차 함수 filter
res = [1, 2, 3].filter(function(item) {
  return item % 2
})

console.log(res) // [1, 3]

// 콜백 함수를 사용하는 고차 함수 reduce
res = [1, 2, 3].reduce(function(acc, cur) {
  return acc + cur
})

console.log(res) // 6
```

## 7.5 순수 함수와 비순수 함수

부수 효과가 없는 함수를 순수 함수라 하고, 부수효과가 있는 함수를 비순수 함수라 한다.

순수 함수는 동일한 인수가 전달되면 언제나 동일한 값을 반환하는 함수다. 즉, 순수 함수는 어떤 외부 상태에도 의존하지 않고 오직 매개변수를 통해 함수 내부로 전달된 인수에게만 의존해 반환값을 만든다.

함수의 외부 상태에 의존하는 함수는 외부 상태에 따라 반환값이 달라진다.

순수 함수의 또 하나의 특징은 함수의 외부 상태를 변경하지 않는다는 것이.

→ 순수 함수는 어떤 외부 상태에도 의존하지 않으며 외부 상태를 변경하지도 않는 함수다.

```jsx
var count = 0 // 현재 카운트를 나타내는 상태

// 순수 함수 increase 는 동일한 인수가 전달되면 언제나 동일한 값을 반환한다.
function increase(n) {
  return ++n
}

// 순수 함수가 반환한 결과값을 변수에 재할당해서 상태를 변경
count = increase(count)
console.log(count) // 1

count = increase(count)
console.log(count) // 2
```

외부 상태에 의존하는 함수를 비순수 함수라고 한다.

```jsx
var count = 0 // 현재 카운트를 나타내는 상태

// 순수 함수 increase 는 동일한 인수가 전달되면 언제나 동일한 값을 반환한다.
function increase(n) {
  return ++count
}

// 비순수 함수가 외부 상태(count)를 변경하므로 상태 변화를 추적하기 어려워진다.
increase()
console.log(count) // 1

increase()
console.log(count) // 2
```
