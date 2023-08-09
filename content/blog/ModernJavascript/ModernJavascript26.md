---
title: 'Modrn Javascript Deep Dive - 26장 ES6 함수의 추가 기능'
date: 2023-08-08
category: 'Javascript'
draft: false
---

# 1. 함수의 구분

자바스크립트의 함수는 일반적인 함수로서 호출할 수도 있고, new 연산자와 함께 호출하여 인스턴스를 생성할 수 있는 생성자 함수로서 호출할 수도 있으며, 객체에 바인딩되어 메서드로서 호출할 수도 있다.

→ 실수를 유발시킬 수 있으며 성능 면에서도 손해다.

```jsx
var foo = function() {
  return 1
}

// 일반적인 함수로서 호출
foo() // 1

// 생성자 함수로서 호출
new foo() // foo {}

// 메서드로서 호출
var obj = { foo: foo }
obj.foo() // 1
```

**ES6 이전의 모든 함수는 일반 함수로서 호출할 수 있는 물론 생성자 함수로서 호출할 수 있다.**

```jsx
var foo = function() {}

// ES6 이전의 모든 함수는 callable이면서 constructor다.
foo() // undefined
new foo() // foo {}
```

ES6 이전에 일반적으로 메서드라고 부르던 객체에 바인딩된 함수도 callable이며 constructor다.

→ 객체에 바인딩된 함수도 일반 함수로서 호출할 수 있는 것은 물론 생성자 함수로서 호출할 수도 있다.

```jsx
// 프로퍼티 f에 바인딩된 함수는 callable이며 constructor다.
var obj = {
  x: 10,
  f: function() {
    return this.x
  },
}

// 프로퍼티 f에 바인딩된 함수를 메서드로서 호출
console.log(obj.f()) // 10

// 프로퍼티 f에 바인딩된 함수를 일반 함수로서 호출
var bar = obj.f
console.log(bar()) // undefined

// 프로퍼티 f에 바인딩된 함수를 생성자 함수로서 호출
console.log(new obj.f()) // f {}
```

함수에 전달되어 보조 함수의 역할을 수행하는 콜백 함수도 마찬가지다. 콜백 함수도 constructor이기 때문에 불필요한 프로토타입 객체를 생성한다.

```jsx
// 콜백 함수를 사용하는 고차 함수 map. 콜백 함수도 constructor이며 프로토타입을 생성한다.
;[1, 2, 3].map(function(item) {
  return item * 2
}) // 2, 4, 6
```

# 2. 메서드

**ES6 사양에서 메서드는 메서드 축약 표현으로 정의된 함수만을 의미한다.**

```jsx
const obj = {
  x: 1,
  // foo는 메서드다.
  foo() {
    return this.x
  },
  // bar에 바인딩된 함수는 메서드가 아닌 일반 함수다.
  bar: function() {
    return this.x
  },
}

console.log(obj.foo()) // 1
console.log(obj.bar()) // 1
```

**ES6 사양에서 정의한 메서드는 인스턴스를 생성할 수 없는 non-constructor다.**

```jsx
new obj.foo() // TypeError
new obj.bar() // bar {}
```

ES6 메서드는 인스턴스를 생성할 수 없으므로 prototype 프로퍼티가 없고 프로토타입도 생성하지 않는다.

```jsx
obj.foo.hasOwnProperty('prototype') // false

// obj.bar는 constructor인 일반 함수이므로 prototype 프로퍼티가 있다.
obj.bar.hasOwnProperty('prototype') // true
```

**ES6 메서드는 자신을 바인딩한 객체를 가리키는 내부 슬롯 HomeObject 를 갖는다.**

ES6 메서드는 super 키워드를 사용할 수 있다.

```jsx
const base = {
  name: 'Son',
  sayHi() {
    return `Hi! ${this.name}`
  },
}

const derived = {
  __proto__: base,
  // sayHi는 ES6 메서드다.
  // sayHi의 HomeObject는 derived.prototype을 가리키고
  // super는 sayHi의 HomeObject의 프로토타입인 base.prototype을 가리킨다.
  sayHi() {
    return `${super.sayHi()}. how are you doing?`
  },
}

console.log(derived.sayHi()) // Hi! Son. how are you doing?
```

ES6 메서드가 아닌 함수는 super 키워드를 사용할 수 없다.

```jsx
const derived = {
  __proto__: base,
  // sayHi는 ES6 메서드다.
  // 따라서 sayHi의 HomeObject를 갖지 않으므로 super 키워드를 사용할 수 없다.
  sayHi: function() {
    // SyntaxError
    return `${super.sayHi()}. how are you doing?`
  },
}
```

# 3. 화살표 함수

화살표 함수는 function 키워드 대신 화살표(⇒ , fat arrow)를 사용하여 기존의 함수 정의 방식보다 간략하게 함수를 정의할 수 있다.

## 3.1 화살표 함수 정의

### 함수 정의

화살표 함수는 함수 선언문으로 정의할 수 없고 함수 표현식으로 정의해야 한다.

```jsx
const multiply = (x, y) => x * y
multiply(2, 3) // 6
```

### 매개변수 선언

```jsx
// 매개변수가 여러 개인 경우 소괄호 ()안에 매개변수를 선언한다.
const arrow = (x, y) => {...};

// 매개변수가 한 개인 경우 소괄호 ()를 생략할 수 있다.
const arrow = x => {...};

// 매개변수가 없는 경우 소괄호 ()를 생략할 수 없다.
const arrow = {} => {...};
```

### 함수 몸체 정의

함수 몸체가 하나의 문으로 구성된다면 함수 몸체를 감싸는 중괄호 {}를 생략할 수 있다. 이때 함수 몸체 내부의 문이 값으로 평가될 수 있는 표현식인 문이라면 암묵적으로 반환된다.

```jsx
// concise body
const power = x => x ** 2
power(2) // 4

// 위 표현은 다음과 같다.
// block body
const power = x => {
  return x ** 2
}
```

함수 몸체를 감싸는 중괄호 {}를 생략한 경우 함수 몸체 내부의 문이 표현식이 아닌 문이라면 에러가 발생한다.

```jsx
const arrow = () => const x = 1; // SyntaxError

// 위 표현은 다음과 같이 해석된다.
const arrow = () => { return const x = 1; };
```

함수 몸체가 하나의 문으로 구성된다 해도 함수 몸체의 문이 표현식이 아닌 문이라면 중괄호를 생략할 수 없다.

```jsx
const arrow = () => {
  const x = 1
}
```

객체 리터럴을 반환하는 경우 객체 리터럴을 소괄호 ()로 감싸 주어야 한다.

```jsx
const create = (id, content) => ({ id, content })
create(1, 'javascript') // { id:1, content:"javascript"}

// 위 표현은 다음과 같다.
const create = (id, content) => {
  return { id, content }
}
```

객체 리터럴을 소괄호 ()로 감싸지 않으면 객체 리터럴의 중괄호 {}를 함수 몸체를 감싸는 중괄호 {}로 잘못 해석한다.

```jsx
// {id, content} 를 함수 몸체 내의 수미표 연산자문으로 해석한다.
const create = (id, content) => {
  id, content
}
create(1, 'javascript') // undefined
```

함수 몸체가 여러 개의 문으로 구성된다면 함수 몸체를 감싸는 중괄호 {}를 생략할 수 없다.

이떄 반환값이 있다면 명시적으로 반환해야 한다.

```jsx
const sum = (a, b) => {
  const result = a + b
  return result
}
```

화살표 함수도 즉시 실행 함수로 사용할 수 있다.

```jsx
const person = name =>
  ({
    sayHi() {
      return `Hi? My name is ${name}`
    },
  }('Son'))

console.log(person.sayHi()) // Hi? My name is Son.
```

화살표 함수도 일급 객체이므로 고차 함수에 인수로 전달할 수 있다.

```jsx
;[1, 2, 3].map(v => v * 2) // [2, 4, 6]
```

## 3.2 화살표 함수와 일반 함수의 차이

### 1. 화살표 함수는 인스턴스를 생성할 수 없는 non-constructor다.

```jsx
const Foo = () => {}
// 화살표 함수는 생성자 함수로서 호출할 수 없다.
new Foo() // TypeError
// 화살표 함수는 prototype 프로퍼티가 없다.
Foo.hasOwnProperty('prototype') // false
```

### 2. 중복된 매개변수 이름을 선언할 수 없다.

```jsx
function normal(a, a) {
  return a + a
}
console.log(normal(1, 2)) // 4

// 단, strict mode에서 중복된 매개변수 이름을 선언하면 에러가 발생한다.
;('use strict')

function normal(a, a) {
  return a + a
} // SyntaxError
```

화살표 함수에서도 중복된 매개변수 이름을 선언하면 에러가 발생한다.

```jsx
const arrow = (a, a) => a + a // SyntaxError
```

### 3. 화살표 함수는 함수 자체의 this, arguments, super, new.target 바인딩을 갖지 않는다.

## 3.3 this

화살표 함수의 this는 일반 함수의 this와 다르게 동작한다.

this 바인딩은 함수를 정의할 때 정적으로 결정되는 것이 아니고, 함수를 호출할 때 함수가 어떻게 호출되었는지에 따라 this에 바인딩할 객체가 동적으로 결정된다.

```jsx
class Prefixer {
  constructor(prefix) {
    this.prefix = prefix
  }

  add(arr) {
    return arr.map(function(item) {
      return this.prefix + item
      // TypeError
    })
  }
}

const prefixer = new Prefixer('-webkit-')
console.log(prefixer.add(['transition', 'user-select']))
```

일반 함수로서 호출되는 모든 함수 내부의 this는 전역 객체를 가리킨다. 그런데 클래스 내부의 모든 코드에는 strict mode가 암묵적으로 적용된다.

Array.prototype.map 메서드의 콜백 함수 내부의 this에는 undefined가 바인딩된다.

**→ 콜백 함수 내부의 this 문제 라 한다.**

ES6에서는 화살표 함수를 사용해서 “콜백 함수 내부의 this 문제”를 해결할 수 있다.

```jsx
class Prefixer {
  constructor(prefix) {
    this.prefix = prefix
  }

  add(arr) {
    return arr.map(item => this.prefix + item)
  }
}

const prefixer = new Prefixer('-webkit-')
console.log(prefixer.add(['transition', 'user-select']))
// [ '-webkit-transition', '-webkit-user-select' ]
```

**화살표 함수는 함수 자체의 this 바인딩을 갖지 않는다. 따라서 화살표 함수 내부에서 this를 참조하면 상위 스코프 this를 그대로 참조한다. → lexical this 라 한다.**

화살표 함수를 다음과 같이 표현이 가능하다.

```jsx
// 화살표 함수는 상위 스코프의 this를 참조한다.
;() => this.x

// 익명 함수에 상위 스코프의 this를 주입한다.
// 위 화살표 함수와 동일하게 동작한다.
;(function() {
  return this.x
}.bind(this))
```

만약 화살표 함수와 화살표 함수가 중첩되어 있다면 상위 화살표 함수에도 this 바인딩이 없으므로 스코프 체인 상에서 가장 가까운 상위 함수 중에서 화살표 함수가 아닌 함수의 this를 참조한다.

```jsx
;(function() {
  const foo = () => console.log(this)
  foo()
}.call({ a: 1 })) // {a:1}

;(function() {
  const bar = () => () => console.log(this)
  bar()()
}.call({ a: 1 })) // {a:1}
```

화살표 함수가 전역 함수라면 화살표 함수의 this는 전역 객체를 가리킨다.

```jsx
const foo = () => console.log(this)
foo() // window
```

프로퍼티에 할당한 화살표 함수도 스코프 체인 상에서 가장 가까운 상위 함수 중에서 화살표 함수가 아닌 함수의 this를 참조한다.

```jsx
const counter = {
  num: 1,
  increase: () => ++this.num, // 여기서의 this는 전역 객체를 가리킨다.
}

console.log(counter.increase()) // NaN
```

화살표 함수는 함수 객체의 this 바인딩을 갖지 않기 때문에 call, apply, bind 메서드를 사용해도 화살표 함수 내부의 this를 교체할 수 없다.

```jsx
window.x = 1

const normal = function() {
  return this.x
}
const arrow = () => this.x

console.log(normal.call({ x: 10 })) // 10
console.log(arrow.call({ x: 10 })) // 1
```

화살표 함수는 함수 자체의 this 바인딩을 갖지 않기 때문에 this를 교체할 수 없고 언제나 상위 스코프의 this 바인딩을 참조한다.

```jsx
const add = (a, b) => a + b

console.log(add.call(null, 1, 2)) // 3
console.log(add.apply(null, [1, 2])) // 3
console.log(add.bind(null, 1, 2)()) // 3
```

메서드를 정의할 때는 ES6 메서드 축약 표현으로 정의한 ES6 메서드를 사용하는 것이 좋다.

```jsx
const person = {
  name: 'Son',
  sayHi() {
    console.log(`Hi! ${this.name}`)
  },
}

person.sayHi() // Hi! Son
```

프로퍼티를 동적 추가할 때는 ES6 메서드 정의를 사용할 수 없으므로 일반 함수를 할당한다.

```jsx
function Person(name) {
  this.name = name
}

Person.prototype.sayHi = function() {
  console.log(`Hi ${this.name}`)
}

const person = new Person('Son')
person.sayHi() // Hi Son
```

클래스 필드 정의 제안을 사용하여 클래스 필드에 화살표 함수를 할당할 수도 있다.

```jsx
class Person {
  constructor() {
    this.name = 'Son'
    this.sayHi = () => console.log(`Hi ${this.name}`)
  }
}
```

클래스 필드에 할당한 화살표 함수는 프로토타입 메서드가 아니라 인스턴스 메서드가 된다.

→ 메서드를 정의할 때는 ES6 메서드 축약 표현으로 정의한 ES6 메서드를 사용하는 것이 좋다.

```jsx
class Person {
  // 클래스 필드 정의
  name = 'Son'
  sayHi() {
    console.log(`Hi ${this.name}`)
  }
}

const person = new Person()
person.sayHi()
```

## 3.4 super

화살표 함수는 함수 자체의 super 바인딩을 갖지 않는다.

→ 화살표 함수 내부에서 super를 참조하면 this와 마찬가지로 상위 스코프의 super를 참조한다.

```jsx
class Base {
  constructor(name) {
    this.name = name
  }

  sayHi() {
    return `Hi! ${this.name}`
  }
}

class Derived extends Base {
  // 화살표 함수의 super는 상위 스코프인 constructor의 super를 가리킨다.
  sayHi = () => `${super.sayHi()}. how are you doing?`
}

const derived = new Derived('Son')
console.log(derived.sayHi())
//Hi! Son. how are you doing?
```

## 3.5 arguments

화살표 함수는 함수 자체의 arguments 바인딩을 갖지 않는다.

→ 화살표 함수 내부에서 arguments를 참조하면 this와 마찬가지로 상위 스코프의 arguments를 참조한다.

```jsx
;(function() {
  // 화살표 함수 foo의 argumetns는 상위 스코프인 즉시 실행 함수의 arguments를 가리킨다.
  const foo = () => console.log(arguments)
  foo(3, 4)
})(1, 2)

// 전역에는 arguments 객체가 존재하지 않는다.
const foo = () => console.log(arguments)
foo(1, 2) // ReferenceError
```

# 4. Rest 파라미터

## 4.1 기본문법

Rest 파라미터는 매개변수 이름 세개의 점 `...` 을 붙여서 정의한 매개변수를 의미한다.

**Rest 파라미터는 함수에 전달된 인수들의 목록을 배열로 전달받는다.**

```jsx
function foo(...rest) {
  // 매개변수 rest는 인수들의 목록을 배열로 전달받는 Rest 파라미터다.
  console.log(rest)
}

foo(1, 2, 3, 4, 5)
// [ 1, 2, 3, 4, 5 ]
```

일반 매개변수와 Rest 파라미터는 함께 사용할 수 있다. 이때 함수에 전달된 인수들은 매개변수와 Rest 파라미터에 순차적으로 할당된다.

```jsx
function foo(param, ...rest) {
  console.log(param) // 1
  console.log(rest) // [2, 3, 4, 5]
}

foo(1, 2, 3, 4, 5)

function bar(param1, param2, ...rest) {
  console.log(param1) // 1
  console.log(param2) // 2
  console.log(rest) // [3, 4, 5]
}

bar(1, 2, 3, 4, 5)
```

Rest 이름 그대로 먼저 선언된 매개변수에 할당된 인수를 제외한 나머지 인수들로 구성된 배열이 할당된다.

→ Rest 파라미터는 반드시 마지막 파라미터이어야 한다.

```jsx
function foo(...rest, param1, param2);

foo(1, 2, 3, 4, 5); // SyntaxError
```

Rest 파라미터는 단 하나만 선언할 수 있다.

```jsx
function foo(...rest1, ...rest2);

foo(1, 2, 3, 4, 5); // SyntaxError
```

Rest 파라미터는 함수 정의 시 선언한 매개변수 개수를 타나내는 함수 객체의 length 프로퍼티에 영향을 주지 않는다.

```jsx
function foo(...rest) {}
console.log(foo.length) // 0

function bar(x, ...rest) {}
console.log(bar.length) // 1

function baz(x, y, ...rest) {}
console.log(baz.length) // 2
```

## 4.2 Rest 파라미터와 arguments 객체

ES6에서는 Rest 파라미터를 사용해서 가변 인자 함수의 인수 목록을 배열로 직접 전달 받을 수 있다.

→ 유사 배열 객체인 arguments 객체를 배열로 변환하는 번거로움을 피할 수 있다.

```jsx
function sum(...args) {
  // Rest 파라미터 args에는 배열 [1, 2, 3, 4, 5]가 할당된다.
  return args.reduce((pre, cur) => pre + cur, 0)
}
console.log(sum(1, 2, 3, 4, 5)) // 15
```

# 5. 매개변수 기본값

자바스크립트 엔진이 매개변수의 개수와 인수의 개수를 체크하지 않는다.

인수가 전달되지 않은 매개변수의 값은 undefined다.

```jsx
function sum(x, y) {
  return x + y
}

console.log(sum(1)) // NaN
```

이 때문에 방어 코드가 필요하다. ES6에서 도입된 매개변수 기본값을 사용하면 함수 내에서 수행하던 인수 체크 및 초기화를 간소화할 수 있다.

```jsx
function sum(x = 0, y = 0) {
  return x + y
}

console.log(sum(1, 2)) // 3
console.log(sum(1)) // 1
```

매개변수 기본값은 매개변수에 인수를 전달하지 않은 경우와 undefined를 전달한 경우에만 유효하다.

```jsx
function logName(name = 'Son') {
  console.log(name)
}

logName() // Son
logName(undefined) // Son
logName(null) // null
```

Rest 파라미터에는 기본값을 지정할 수 없다.

```jsx
function foo(...rest = []) {
	console.log(rest);
}
// SyntaxError
```
