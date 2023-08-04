---
title: 'Modrn Javascript Deep Dive - 19장 프로토타입(2)'
date: 2023-08-04
category: 'Javascript'
draft: false
---

# 11. 직접 상속

## 11.1 Object.create에 의한 직접 상속

`Object.create` 메서드는 명시적으로 프로토타입을 지정하여 새로운 객체를 생성한다.

```jsx
// 프로토타입이 null인 객체를 생성한다. 생성된 객체는 프로토타입 체인의 종점에 위치한다.
// obj -> null
let obj = Object.create(null)
console.log(Object.getPrototypeOf(obj) === null) // true
// Object.prototype을 상속받지 못한다.
console.log(obj.toString()) // TypeError

// obj -> Object.prototype -> null
// obj = {}; 와 동일하다.
obj = Object.create(Object.prototype)
console.log(Object.getPrototypeOf(obj) === Object.prototype) // true

// obj -> Object.prototype -> null
// obj = { x: 1 }; 와 동일하다.
obj = Object.create(Object.prototype, {
  x: { value: 1, writable: true, enumerable: true, configurable: true },
})

console.log(obj.x) // 1
console.log(Object.getPrototypeOf(obj) === Object.prototype) // true

const myProto = { x: 10 }
// 임의의 객체를 직접 상속받는다.
// obj -> myProto -> Object.prototype -> null
obj = Object.create(myProto)
console.log(obj.x) // 10
console.log(Object.getPrototypeOf(obj) === myProto) // true

// 생성자 함수
function Person(name) {
  this.name = name
}

// obj -> Person.prototype -> Object.prototype -> null
// obj = new Person("Son") 와 동일
obj = Object.create(Person.prototype)
obj.name = 'Son'
console.log(obj.name) // Son
console.log(Object.getPrototypeOf(obj) === Person.prototype) // true
```

Object.create 메서드는 첫 번째 매개변수에 전달한 객체의 프로토타입 체인에 속하는 객체를 생성한다.

→ 객체를 생성하면서 직접적으로 상속을 구현하는 것이다.

- new 연산자가 없이도 객체를 생성할 것이다.
- 프로토타입을 지정하면서 객체를 생성할 수 있다.
- 객체 리터럴에 의해 생성된 객체도 상속받을 수 있다.

```jsx
const obj = { a: 1 }

obj.hasOwnProperty('a') // true
obj.propertyIsEnumerable('a') // true
```

그러나 Object.prototype의 빌트인 메서드를 객체가 직접 호출하는 것을 권장하지 않는다.

Object.create 메서드를 통해 프로토타입 체인의 종점에 위치하는 객체를 생성할 수 있기 때문이다.

프로토타입 체인의 종점에 위치하는 객체는 Object.prototype의 빌트인 메서드를 사용할 수 없다.

```jsx
// 프로토타입이 null인 객체, 즉 프로토타입 체인의 좀점에 위치하는 객체를 생성한다.
const obj = Object.create(null)
obj.a = 1

console.log(Object.getPrototypeOf(obj) === null) // true

// obj는 Object.prototype의 빌트인 메서드를 사용할 수 없다.
console.log(obj.hasOwnProperty('a')) // TypeError
```

Object.prototype의 빌트인 메서드는 다음과 같이 간접적으로 호출하는 것이 좋다.

```jsx
// 프로토타입이 null인 객체, 즉 프로토타입 체인의 좀점에 위치하는 객체를 생성한다.
const obj = Object.create(null)
obj.a = 1

console.log(Object.getPrototypeOf(obj) === null) // true

// Object.prototype의 빌트인 메서드는 객체로 직접 호출하지 않는다.
console.log(Object.prototype.hasOwnProperty.call(obj, 'a')) // true
```

## 11.2 객체 리터럴 내부에서 `__proto__` 에 의한 직접 상속

ES6에서는 객체 리터럴 내부에서 `__proto__` 접근자 프로퍼티를 사용하여 직접 상속을 구현할 수 있다.

```jsx
const myProto = { x: 10 }

// 객체 리터럴에 의해 객체를 생성하면서 프로토타입을 지정하여 직접 상속받을 수 있다.
const obj = {
  y: 20,
  // 객체를 직접 상속받는다.
  // obj -> myProto -> Object.prototype -> null
  __proto__: myProto,
}

console.log(obj.x, obj.y) // 10 20
console.log(Object.getPrototypeOf(obj) === myProto) // true
```

# 12. 정적 프로퍼티/메서드

정적 프로퍼티/메서드는 생성자 함수로 인스턴스를 생성하지 않아도 참조/호출할 수 있는 프로퍼티/메서드를 말한다.

```jsx
// 생성자 함수
function Person(name) {
  this.name = name
}

// 프로토타입 메서드
Person.prototype.sayHello = function() {
  console.log(`Hi! My name is ${this.name}`)
}

// 정적 프로퍼티
Person.staticProp = 'static prop'

// 정적 메서드
Person.staticMethod = function() {
  console.log('staticMethod')
}

const me = new Person('Son')

// 생성자 함수에 추가한 정적 프로퍼티/메서드는 생성자 함수로 참조/호출한다.
Person.staticMethod() // staticMethod

// 정적 프로퍼티 메서드는 생성자 함수가 생성한 인스턴스로 참조/호출할 수 없다.
me.staticMethod() // TypeError
```

Object.prototype의 메서드이므로 모든 객체가 호출할 수 있다.

```jsx
// Object.create는 정적 메서드다.
const obj = Object.create({ name: 'Son' })

// Object.prototype.hasOwnProperty는 프로토타입 메서드다.
obj.hasOwnProperty('name') // false
```

프로토타입 메서드를 호출하려면 인스턴스를 생성해야 하지만 정적 메서드는 인스턴스를 생성하지 않아도 호출할 수 있다.

```jsx
function foo() {}

// 프로토타입 메서드
// this를 참조하지 않는 프로토타입 메서드는 정적 메서드로 변경하여도 동일한 효과를 얻을 수 있다.
foo.prototype.x = function() {
  console.log('x')
}

const foo = new foo()
// 프로토타입 메서드를 호출하려면 인스턴스를 생성해야 한다.
foo.x() // x

// 정적 메서드
Foo.x = function() {
  console.log('x')
}

// 정적 메서드는 인스턴스를 생성하지 않아도 호출할 수 있다.
Foo.x() // x
```

# 13. 프로퍼티 존재 확인

## 13.1 in 연산자

in 연산자는 객체 내에 특정 프로퍼티가 존재하는지 여부를 확인한다.

```jsx
const person = {
  name: 'Son',
  address: 'Seoul',
}

// person 객체에 name 프로퍼티가 존재한다.
console.log('name' in person) // true
// person 객체에 address 프로퍼티가 존재한다.
console.log('address' in person) // true
// person 객체에 age 프로퍼티가 존재하지 않는다.
console.log('age' in person) // false
```

in 연산자는 확인 대상 객체의 프로퍼티뿐만 아니라 확인 대상 객체가 상속받은 모든 프로퍼티를 확인하므로 주의가 필요하다.

```jsx
console.log('toString' in person) // true
```

in 연산자가 프로토타입 체인 상에 존재하는 모든 프로토타입에서 `toString` 프로퍼티를 검색하기 때문에 `true` 가 된다.

ES6에서는 `Reflect.has` 메서드를 사용할 수 있다.

```jsx
const person = { name: 'Son' }

console.log(Reflect.has(person, 'name')) // true
console.log(Reflect.has(person, 'toString')) // true
```

## 13.2 Object.prototype.hasOwnProperty 메서드

객체에 특정 프로퍼티가 존재하는지 확인할 수 있다.

```jsx
const person = {
  name: 'Son',
  address: 'Seoul',
}

console.log(person.hasOwnProperty('name')) // true
console.log(person.hasOwnProperty('age')) // false

console.log(person.hasOwnProperty('toString')) // false
```

인수로 전달받은 프로퍼티 키가 객체 고유의 프로퍼티 키인 경우에만 true를 반환하고 상속받은 프로퍼티 키인 경우 false를 반환한다.

# 14. 프로퍼티 열거

## 14.1 for…in 문

객체의 모든 프로퍼티를 순회하며 열거하려면 `for...in` 문을 사용한다.

```jsx
const person = {
  name: 'Son',
  address: 'Seoul',
}

// for...in 문의 변수 prop에 person 객체의 프로퍼티 키가 할당된다.
for (const key in person) {
  console.log(key + ': ' + person[key])
}

// name: Son
// address: Seoul
```

`for...in` 문은 in 연산자처럼 상속받은 프로토타입의 프로퍼티까지 열거한다.

→ toString 과 같은 Object.prototype의 프로퍼티가 열거되지 않는다.

→ Enumerable 의 값이 true 인 프로퍼티를 순회하며 열거 한다.

```jsx
const person = {
  name: 'Son',
  address: 'Seoul',
  __proto__: { age: 20 },
}

// for...in 문의 변수 prop에 person 객체의 프로퍼티 키가 할당된다.
for (const key in person) {
  console.log(key + ': ' + person[key])
}

// name: Son
// address: Seoul
// age: 20
```

`for...in` 문은 프로퍼티 키가 심벌인 프로퍼티는 열거하지 않는다.

```jsx
const sym = Symbol()
const obj = {
  a: 1,
  [sym]: 10,
}

for (const key in obj) {
  console.log(key + ': ' + obj[key])
}

// a: 1
```

배열에는 `for...in` 문을 사용하지 말고 일반적인 for 문이나 `for...of` 문, Array.prototype.forEach 메서드를 사용하는 것이 좋다.

```jsx
const arr = [1, 2, 3]
arr.x = 10 // 배열도 객체이므로 프로퍼티를 가질 수 있다.

for (const i in arr) {
  // 프로퍼티 x도 출력된다.
  console.log(arr[i]) // 1 2 3 10
}

// arr.length는 3이다.
for (let i = 0; i < arr.length; i++) {
  console.log(arr[i]) // 1 2 3
}

// forEach 메서드는 요소가 아닌 프로퍼티는 제외한다.
arr.forEach(i => console.log(i)) // 1 2 3

// for...of 는 변수 선언문에서 선언한 변수에 키가 아닌 값을 할당한다.
for (const value of arr) {
  console.log(value) // 1 2 3
}
```

## 14.2 Object.keys/values/entries 메서드

객체 자신의 고유 프로퍼티만 열거하기 위해서는 `for...in` 문을 사용하는 것보다 Object.keys/values/entries 메서드를 사용하는 것을 권장된다.

Object.keys 메서드는 객체 자신의 열거 가능한 프로퍼티 키를 배열로 반환한다.

```jsx
const person = {
  name: 'Son',
  address: 'Seoul',
  __proto__: { age: 20 },
}

console.log(Object.keys(person)) // [ 'name', 'address' ]
```

ES8에서 도입된 Object.values 메서드는 객체 자신의 열거 가능한 프로퍼티 값을 배열로 반환한다.

```jsx
console.log(Object.values(person)) // [ 'Son', 'Seoul' ]
```

ES8에서 도입된 Object.entries 메서드는 객체 자신의 열거 가능한 키와 값의 쌍의 배열을 반환한다.

```jsx
const person = {
  name: 'Son',
  address: 'Seoul',
  __proto__: { age: 20 },
}

console.log(Object.entries(person)) // [ [ 'name', 'Son' ], [ 'address', 'Seoul' ] ]

Object.entries(person).forEach(([key, value]) => console.log(key, value))
// name Son
// address Seoul
```
