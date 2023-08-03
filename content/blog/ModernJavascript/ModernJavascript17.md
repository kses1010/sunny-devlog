---
title: 'Modrn Javascript Deep Dive - 17장 생성자 함수에 의한 객체 생성'
date: 2023-08-03
category: 'Javascript'
draft: false
---

# 1. Object 생성자 함수

new 연산자와 함께 Object 생성자 함수를 호출하면 빈 객체를 생성하여 반환한다.

```jsx
// 빈 객체의 생성
const person = new Object()

// 프로퍼티 추가
person.name = 'Son'
person.sayHello = function() {
  console.log(`Hi! My name is ${this.name}`)
}

console.log(person)
person.sayHello

// { name: 'Son', sayHello: [Function (anonymous)] }
// Hi! My name is Son
```

생성자 함수란 new 연산자와 함께 호출하여 객체를 생성하는 함수를 말한다. 생성자 함수로 생성된 객체를 인스턴스라 한다.

자바스크립트는 Object 생성자 함수 이외에도 빌트인 생성자 함수를 제공한다.

```jsx
// String
const strObj = new String('Son')
console.log(typeof strObj)
console.log(strObj) // [String : "Son"]

// Number
const numObj = new Number(123)
console.log(typeof numObj)
console.log(numObj) // [Number : 123]

// Boolean
const boolObj = new Boolean(true)
console.log(typeof boolObj)
console.log(boolObj) // [Boolean: true]

// Function
const func = new Function('x', 'return x * x')
console.log(typeof func) // function
console.dir(func)

// Array
const arr = new Array(1, 2, 3)
console.log(typeof arr)
console.log(arr) // [1, 2, 3]

// RegExp
const regExp = new RegExp(/ab+c/i)
console.log(typeof regExp)
console.log(regExp) // /ab+c/i

// Date
const date = new Date()
console.log(typeof date)
console.log(date) // 2023-08-03T04:11:37.506Z
```

객체를 생성하는 방법은 객체 리터럴을 사용하는 것이 더 간편하다. 굳이 Object 생성자 함수를 사용해 객체를 생성하는 방식은 특별한 이유가 없다면 그다지 유용해 보이지 않는다.

# 2. 생성자 함수

## 2.1 객체 리터럴에 의한 객체 생성 방식의 문제점

객체 리터럴에 의한 객체 생성 방식은 단 하나의 객체만 생성한다. 동일한 프로퍼티를 가진 객체를 여러 개 생성하는 경우 매번 같은 프로퍼티를 기술해야 한다.

```jsx
const circle1 = {
  radius: 5,
  getDiameter() {
    return 2 * this.radius
  },
}

const circle2 = {
  radius: 10,
  getDiameter() {
    return 2 * this.radius
  },
}

console.log(circle1.getDiameter()) // 10
console.log(circle2.getDiameter()) // 20
```

## 2.2 생성자 함수에 의한 객체 생성 방식의 장점

생성자 함수를 사용하여 프로퍼티 구조가 동일한 객체 여러 개를 간편하게 생성할 수 있다.

```jsx
// 생성자 함수
function Circle(radius) {
  // 생성자 함수 내부의 this는 생성자 함수가 생성할 인스턴스를 가리킨다.
  this.radius = radius
  this.getDiameter = function() {
    return 2 * this.radius
  }
}

const circle1 = new Circle(5)
const circle2 = new Circle(10)

console.log(circle1.getDiameter()) // 10
console.log(circle2.getDiameter()) // 20
```

생성자 함수는 이름 그대로 객체(인스턴스)를 생성하는 함수다.

**new 연산자와 함께 호출하면 해당 함수는 생성자 함수로 동작한다.**

```jsx
// new 연산자와 함께 호출하지 않으면 생성자 함수로 동작하지 않는다.
// 일반 함수로 호출됨
const circle3 = Circle(15)

console.log(circle3) // undefined

// 일반 함수로소 호출된 Circle 내의 this는 전역 객체를 가리킨다.
console.log(radius) // 15
```

## 2.3 생성자 함수의 인스턴스 생성 과정

생성자 함수의 역할은 프로퍼티 구조가 동일한 인스턴스를 생성하기 위한 템플릿(클래스)으로서 동작하여 **인스턴스를 생성**하는 것과 **생성된 인스턴스를 초기화(인스턴스 프로퍼티 추가 및 초기값 할당)**하는 것이다.

```jsx
function Circle(radius) {
  this.radius = radius
  this.getDiameter = function() {
    return 2 * this.radius
  }
}

const circle1 = new Circle(5)
```

자바스크립트 엔진은 암묵적인 처리를 통해 인스턴스를 생성하고 반환한다. new 연산자와 함께 생성자 함수를 호출하면 자바스크립트 엔진은 암묵적으로 인스턴스를 초기화한 후 암묵적으로 인스턴스를 반환한다.

→ 생성자 함수 내부에서 `return` 문은 반드시 생략해야 한다.

## 2.4 내부 메서드 [[Call]] 과 [[Construct]]

```jsx
// 함수는 객체다.
function foo() {}

// 함수는 객체이므로 프로퍼티를 소유할 수 있다.
foo.prop = 10

// 함수는 객체이므로 메서드를 소유할 수 있다.
foo.method = function() {
  console.log(this.prop)
}

foo.method() // 10
```

함수는 객체이지만 일반 객체와는 다르다. **일반 객체는 호출할 수 없지만 함수는 호출할 수 있다.**

함수 객체는 일반 객체가 가지고 있는 내부 슬롯과 내부 메서드는 물론, 함수로서 동작하기 위해 함수 객체 만을 위한 Environment, FormalParameter 등의 내부 슬롯과 Call, Construct 같은 내부 메서드를 추가로 가지고 있다.

```jsx
function foo() {}

// 일반적인 함수로서 호출: Call 호출
foo()

// 생성자 함수로서 호출: Construct 호출
new foo()
```

내부 메서드 Call 을 갖는 함수 객체를 callable 이라 하며, 내부 메서드 Construct 를 갖는 함수 객체를 constructor, Construct 를 갖지 않은 함수 객체를 non-constructor라고 부른다.

callable 은 호출할 수 있는 객체, 즉 함수를 말하며

constructor 는 생성자 함수로서 호출할 수 있는 함수

non-constructor 는 객체를 생성자 함수로서 호출할 수 없는 함수

호출할 수 없는 객체는 함수 객체가 아니므로 함수로서 가능하는 객체, 즉 함수 객체는 반드시 callable 이어야 한다. → 모든 함수 객체는 내부 메서드 Call을 갖고 있으므로 호출할 수 있다.

## 2.5 constructor와 non-constructor의 구분

- constructor: 함수 선언문, 함수 표현식, 클래스(클래스도 함수)
- non-constructor: 메서드, 화살표 함수

```jsx
// 일반 함수 정의: 함수 선언문, 함수 표현식
function foo() {}
const bar = function() {}
// 프로퍼티 x의 값으로 할당된 것은 일반 함수로 정의된 함수다. 이는 메서드로 인정하지 않음
const baz = {
  x: function() {},
}

// 일반 함수로 정의된 함수만이 constructor.
new foo() // foo {}
new bar() // bar {}
new bar.x() // x {}

// 화살표 함수 정의
const arrow = () => {}

new arrow() // TypeError

// 메서드 정의: ES6 의 메서드 축약 표현만 메서드로 인정
const obj = {
  x() {},
}

new obj.x() // TypeError:
```

생성자 함수로서 호출될 것을 기대하고 정의하지 않은 일반 함수(callble / constructor)에 new 연산자를 붙여 호출하면 생성자 함수처럼 동작할 수 있다.

## 2.6 new 연산자

new 연산자와 함께 함수를 호출하면 해당 함수는 생성자 함수로 동작한다.

```jsx
// 생성자 함수로서 정의하지 않은 일반 함수
function add(x, y) {
  return x + y
}

// 생성자 함수로서 정의하지 않은 일반 함수를 new 연산자와 함께 호출
let inst = new add()

// 함수가 객체를 반환하지 않았으므로 반환문이 무시된다. -> 빈 객체가 생성
console.log(inst) // {}

// 객체를 반환하는 일반 함수
function createUser(name, role) {
  return { name, role }
}

// 일반 함수를 new 연산자와 함께 호출
inst = new createUser('Son', 'admin')
// 함수가 생성된 객체 반환
console.log(inst) // { name: 'Son', role: 'admin' }
```

반대로 new 연산자 없이 생성자 함수를 호출하면 일반 함수로 호출된다.

```jsx
function Circle(radius) {
  this.radius = radius
  this.getDiameter = function() {
    return 2 * this.radius
  }
}

const circle1 = Circle(5)

console.log(circle1) // undefined

// 일반 함수 내부의 this는 전역 객체 window를 가리킨다.
console.log(radius) // 5
console.log(getDiameter()) // 10

circle1.getDiameter() // TypeError
```

위 예제의 Circle 함수는 일반 함수로서 호출되었기 때문에 Circle 함수 내부의 this는 전역 객체 window를 가리킨다. → radius 프로퍼티, getDiameter 메서드는 전역 객체의 프로퍼티와 메서드가 된다.

## 2.7 new.target

생성자 함수가 new 연산자 없이 호출되는 것을 방지하기 위해 파스칼 케이스 컨벤션을 사용한다 하더라도 실수가 발생할 수 있다. → ES6에서는 `new.target` 을 지원한다.

`new.target` 은 this와 유사하게 constructor인 모든 함수 내부에서 암묵적인 지역 변수와 같이 사용되며 메타 프로퍼티라고 부른다.

**new 연산자와 함께 생성자 함수로서 호출되면 함수 내부의 `new.target` 은 함수 자신을 가리킨다. new 연산자없이 일반 함수로서 호출된 함수 내부의 `new.target` 은 undefined다.**

```jsx
// 생성자 함수
function Circle(radius) {
  // 이 함수가 new 연산자와 함께 호출되지 않았다면 new.target은 undefined다.
  if (!new.target) {
    // new 연산자와 함께 생성자 함수를 재귀 호출하여 생성된 인스턴스를 반환한다.
    return new Circle(radius)
  }
  this.radius = radius
  this.getDiameter = function() {
    return 2 * this.radius
  }
}

// new 연산자 없이 생성자 함수를 호출하여도 new.target을 통해 생성자 함수로서 호출된다.
const circle = Circle(5)
console.log(circle.getDiameter()) // 10
```

참고로 대부분의 빌트인 생성자 함수는 new 연산자와 함께 호출되었는지를 확인한 후 적절한 값을 반환한다.

```jsx
let obj = new Object()
console.log(obj) // {}

obj = Object()
console.log(obj) // {}

let f = new Function('x', 'return x ** x')
console.log(f) // [Function: anonymous]

f = Function('x', 'return x ** x')
console.log(f) // [Function: anonymous]
```

하지만 String, Number, Boolean 생성자 함수는 new 연산자와 함께 호출했을 때 객체를 생성하여 반환하지만 new 연산자 없이 호출하면 문자열, 숫자, 불리언 값을 반환한다.

```jsx
const str = String(123)
console.log(str, typeof str) // 123 string

const num = Number('123')
console.log(num, typeof num) // 123 number

const bool = Boolean(1)
console.log(bool, typeof bool) // true boolean
```
