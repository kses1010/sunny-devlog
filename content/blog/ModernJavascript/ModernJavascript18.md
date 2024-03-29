---
title: 'Modern Javascript Deep Dive - 18장 함수와 일급 객체'
date: 2023-08-03 14:48:41 
category: 'Javascript'
draft: false
---

# 1. 일급 객체

- 무명의 리터럴로 생성할 수 있다. 즉, 런타임에 생성이 가능하다.
- 변수나 자료구조(객체, 배열 등)에 저장할 수 있다.
- 함수의 매개변수에 전달할 수 있다.
- 함수의 반환값으로 사용될 수 있다.

```jsx
// 1. 함수는 무명의 리터럴로 생성할 수 있다.
// 2. 함수는 변수에 저장할 수 있다.
// 런타임(할당 단계)에 함수 리터럴이 평가되어 함수 객체가 생성되고 변수에 할당된다.
const increase = function(num) {
  return ++num;
}

const decrease = function(num) {
  return --num;
}

// 2. 함수는 객체에 저장할 수 있다.
const predicates = { increase, decrease };

// 3. 함수의 매개변수에 전달할 수 있다.
// 4. 함수의 반환값으로 사용할 수 있다.
function makeCounter(predicate) {
  let num = 0;

  return function() {
    num = predicate(num);
    return num;
  }
}

// 3. 함수는 매개변수에게 함수를 전달할 수 있다.
const increaser = makeCounter(predicates.increase);
console.log(increaser()); // 1
console.log(increaser()); // 2

const decreaser = makeCounter(predicates.decrease);
console.log(decreaser()); // -1
console.log(decreaser()); // -2
```

일급 객체로서 함수가 가지는 가장 큰 특징은 일반 객체와 같이 함수의 매개변수에 전달할 수 있으며, 함수의 반환값으로 사용할 수도 있다는 것이다. → 함수형 프로그래밍을 가능하게 해주는 자바스크립트의 장점 중 하나다.

함수는 객체이지만 일반 객체와는 차이가 있다. 일반 객체는 호출할 수 없지만 함수 객체는 호출할 수 있다.

함수 객체는 일반 객체에는 없는 함수 고유의 프로퍼티를 소유한다.

# 2. 함수 객체의 프로퍼티

```jsx
function square(number) {
  return number * number;
}

console.log(Object.getOwnPropertyDescriptors(square));
```

arguments, caller, length, name, prototype 프로퍼티는 모두 함수 객체의 데이터 프로퍼티다.

이들 프로퍼티는 일반 객체에는 없는 함수 객체 고유의 프로퍼티다.

## 2.1 arguments 프로퍼티

arguments 객체는 함수 호출 시 전달된 인수들의 정보를 담고 있는 순회 가능한 유사 배열 객체이며, 함수 내부에서 지역 변수처럼 사용된다. → 함수 외부에서는 참조할 수 없다.

자바스크립트는 함수의 매개변수와 인수의 개수가 일치하는지 확인하지 않는다.

→ 함수 호출 시 매개변수 개수만큼 인수를 전달하지 않아도 에러가 발생하지 않는다.

```jsx
function multiply(x, y) {
  console.log(arguments);
  return x * y;
}

console.log(multiply()); // NaN
console.log(multiply(1)); // NaN
console.log(multiply(1, 2)); // 2
console.log(multiply(1, 2, 3)); // 2
```

arguments 객체는 인수를 프로퍼티 값으로 소유하며 프로퍼티 키는 인수의 순서를 나타낸다.

선언된 매개변수의 개수와 함수를 호출할 때 전달하는 인수의 개수를 확인하지 않는 자바스크립트의 특성 때문에 함수가 호출되면 인수 개수를 확인하고 이에 따라 함수의 동작을 달리 정의할 필요가 있을 수 있다.

→ arguments 객체는 매개변수 개수를 확정할 수 없는 가변 인자 함수를 구현할 때 유용하다.

```jsx
function sum() {
  let res = 0;

  // arguments 객체는 length 프로퍼티가 있는 유사 배열 객체이므로 for 문으로 순회할 수 있다.
  for (let i = 0; i < arguments.length; i++) {
    res += arguments[i];
  }

  return res;
}

console.log(sum()); // 0
console.log(sum(1, 2)); // 3
console.log(sum(1, 2, 3)); // 6
```

arguments 객체는 배열 형태로 인자 정보를 담고 있지만 실제 배열이 아닌 유사 배열 객체다.

유사 배열 객체란 length 프로퍼티를 가진 객체로 for 문으로 순회할 수 있는 객체를 말한다.

유사 배열 객체는 배열이 아니므로 배열 메서드를 사용할 경우 에러가 발생한다.

→ 배열 메서드를 사용하려면 `Function.prototype.call` , `Function.prototype.apply` 를 사용해 간접 호출해야 하는 번거로움이 있다.

```jsx
function sum() {
  // arguments 객체를 배열로 변환
  const array = Array.prototype.slice.call(arguments);
  return array.reduce(function(pre, cur) {
    return pre + cur;
  }, 0);
}

console.log(sum(1, 2)); // 3
console.log(sum(1, 2, 3, 4, 5)); // 15
```

ES6에서는 Rest 파라미터를 도입했다.

```jsx
// ES6 Rest parameter
function sum(...args) {
  return args.reduce((pre, cur) => pre + cur, 0);
}

console.log(sum(1, 2)); // 3
console.log(sum(1, 2, 3, 4, 5)); // 15
```

## 2.2 caller 프로퍼티

caller 프로퍼티는 ECMAScript 사양에 포함되지 않은 비표준 프로퍼티다. 넘어가도록 하자.

## 2.3 length 프로퍼티ㅎ

함수 객체의 length 프로퍼티는 함수를 정의할 때 선언한 매개변수의 개수를 가리킨다.

```jsx
function foo() {};
console.log(foo.length); // 0

function bar(x) {
  return x;
}
console.log(bar.length); // 1

function baz(x, y) {
  return x * y;
}
console.log(baz.length); // 2
```

arguments 객체의 length 프로퍼티와 함수 객체의 length 프로퍼티의 값은 다를 수 있다.

- arguments 객체: 인자의 개수
- 함수 객체: 매개 변수의 개수

## 2.4 name 프로퍼티

함수 객체의 name 프로퍼티는 함수 이름을 나타낸다.

name 프로퍼티는 ES5 와 ES6에서 동작을 다르기 때문에 주의해야 한다.

ES5: 익명 함수 표현식의 경우 빈 문자열을 값을 갖는다.

ES6: 함수 객체를 가리키는 식별자를 값으로 갖는다.

```jsx
// 기명 함수 표현식
var namedFunc = function foo() {};
console.log(namedFunc.name); // foo

// 익명 함수 표현식
var anonymousFunc = function() {};
console.log(anonymousFunc.name); // anonymousFunc

// 함수 선언문
function bar() {};
console.log(bar.name); // bar
```

## 2.5 `__proto__` 접근자 프로퍼티

`__proto__` 프로퍼티는 Prototype 내부 슬롯이 가리키는 프로토타입 객체에 접근하기 위해 사용하는 접근자 프로퍼티다. 내부 슬롯에는 직접 접근할 수 없고 간접적인 접근 방법을 제공하는 경우에 한하여 접근할 수 있다.

```jsx
const obj = { a: 1 };

// 객체 리터럴 방식으로 생성한 객체의 프로토타입 객체는 Object.prototype 이다.
console.log(obj.__proto__ === Object.prototype); // true

// 객체 리터럴 방식으로 생성한 객체는 프로토타입 객체인 Object.prototype의 프로퍼티를 상속받는다.
// hasOwnProperty 메서드는 Object.prototype의 메서드다.
console.log(obj.hasOwnProperty('a')); // true
console.log(obj.hasOwnProperty('__proto__')); // false
```

## 2.6 prototype 프로퍼티

prototype 프로퍼티는 생성자 함수로 호출할 수 있는 함수 객체로 non-constructor 에는 존재하지 않는다.

```jsx
// 함수 객체는 prototype 프로퍼티를 소유한다.
(function() {}.hasOwnProperty('prototype')); // true

// 일반 객체는 prototype 프로퍼티를 소유하지 않는다.
({}.hasOwnProperty('prototype')); // false
```

prototype 프로퍼티는 함수가 객체를 생성하는 생성자 함수로 호출될 때 생성자 함수가 생성할 인스턴스의 프로토타입 객체를 가리킨다.
