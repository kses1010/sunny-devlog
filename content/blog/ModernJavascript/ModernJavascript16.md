---
title: 'Modern Javascript Deep Dive - 16장 프로퍼티 어트리뷰트'
date: 2023-08-03 12:38:39
category: 'Javascript'
draft: false
---

# 1. 내부 슬롯과 내부 메서드

ECMAScript 사양에 등장하는 이중 대괄호(`[[…]]`) 로 감싼 이름들이 내부 슬롯과 내부 메서드다.

내부 슬롯과 내부 메서드는 ECMAScript 사양에 정의된 대로 구현되어 자바스크립트 엔진에서 실제로 동작하지만 개발자가 직접 접근할 수 있도록 외부로 공개된 객체의 프로퍼티는 아니다.

→ 원칙적으로 자바스크립트는 내부 슬롯과 내부 메서드에 직접적으로 접근하거나 호출할 수 있는 방법을 제공하지 않는다.

```jsx
const o = {};

// 내부 슬롯은 자바스크립트 엔진의 내부 로직이므로 직접 접근할 수 없다.
o.[[Prototype]]; // Uncaught SyntaxError
// 단, 일부 내부 슬롯과 내부 메서드에 한하여 간접적으로 접근할 수 있는 수단을 제공한다.
o.__proto__; // Object.prototype
```

# 2. 프로퍼티 어트리뷰트와 프로퍼티 디스크립터 객체

**자바스크립트 엔진은 프로퍼티를 생성할 때 프로퍼티의 상태를 나타내는 프로퍼티 어트리뷰트를 기본적으로 자동 정의한다.**

```jsx
const person = {
  name: 'Son',
};

// 프로퍼티 어트리뷰트 정보를 제공하는 프로퍼티 디스크립터 객체를 반환한다.
console.log(Object.getOwnPropertyDescriptor(person, 'name'));

// { value: 'Son', writable: true, enumerable: true, configurable: true }
```

`Object.getOwnPropertyDescriptor()` 호출할 때 첫 번째 매개변수에는 객체의 참조를 전달하고, 두 번째 매개변수에는 프로퍼티 키를 문자열로 전달한다.

`Object.getOwnPropertyDescriptor()` 프로퍼티 어트리뷰트 정보를 제공하는 **프로퍼티 디스크립터 객체**를 반환한다. 만약 존재하지 않는 프로퍼티나 상속받은 프로퍼티라면 undefined를 반환된다.

# 3. 데이터 프로퍼티와 접근자 프로퍼티

- 데이터 프로퍼티: 키와 값으로 구성된 일반적인 프로퍼티.
- 접근자 프로퍼티: 자체적으로 값을 갖지 않고 다른 데이터 프로퍼티의 값을 읽거나 저장할 때 호출되는 접근자 함수로 구성된 프로퍼티.

## 3.1 데이터 프로퍼티

```jsx
// { value: 'Son', writable: true, enumerable: true, configurable: true }
```

- value: 프로퍼티 키를 통해 프로퍼티 값에 접근하면 반환되는 값.
  - 프로퍼티가 없다면 동적 생성하고 생성된 프로퍼티의 [[value]]에 값을 갖는다.
- writable: 프로퍼티 값의 변경 가능 여부를 나타내며 불리언 값을 갖는다.
- enumerable: 프로퍼티의 열거 가능 여부를 나타내며 불리언 값을 갖는다.
  - false인 경우 `for...in` 문이나 `Object.keys` 등으로 열거할 수 없다.
- configurable: 프로퍼티의 재정의 가능 여부를 나타내며 불리언 값을 갖는다.

## 3.2 접근자 프로퍼티

- get: 접근자 프로퍼티를 통해 데이터 프로퍼티의 값을 읽을 때 호출되는 접근자 함수다.
  - 접근자 프로퍼티 키로 프로퍼티 값에 접근하면 프로퍼티 어트리뷰트 `getter()` 호출되고 그 결과가 프로퍼티 값으로 반환된다.
- set: 접근자 프로퍼티를 통해 데이터 프로퍼티의 값을 저장할 때 호출되는 접근자 함수다.
  - 접근자 프로퍼티 키로 프로퍼티 값에 저장하면 프로퍼티 어트리뷰트 `setter()` 호출되고 그 결과가 프로퍼티 값으로 반환된다.
- enumerable: 데이터 프로퍼티와 같음
- configurable: 데이터 프로퍼티와 같음

접근자 함수는 `getter/setter` 함수라고도 부른다.

```jsx
const person = {
  // 데이터 프로퍼티
  firstName: 'Sunny',
  lastName: 'Son',

  // fullName은 접근자 함수로 구성된 접근자 프로퍼티다.
  get fullName() {
    return `${this.firstName} ${this.lastName}`;
  },
  set fullName(name) {
    // 배열 디스트럭처링 할당
    [this.firstName, this.lastName] = name.split(' ');
  },
};

// 데이터 프로퍼티를 통한 프로퍼티 값의 참조
console.log(person.firstName + ' ' + person.lastName); // Sunny Son

// 접근자 프로퍼티를 통한 프로퍼티 값 저장 및 참조
person.fullName = 'Cloud Kim';
console.log(person); // {firstName: "Cloud", lastName: "Kim"}

console.log(person.fullName); // Cloud Kim

let descriptor = Object.getOwnPropertyDescriptor(person, 'firstName');
console.log(descriptor);
//   {
//     value: 'Cloud',
//     writable: true,
//     enumerable: true,
//     configurable: true
//   }

descriptor = Object.getOwnPropertyDescriptor(person, 'fullName');
console.log(descriptor);
//   {
//     get: [Function: get fullName],
//     set: [Function: set fullName],
//     enumerable: true,
//     configurable: true
//   }
```

### 프로토타입

프로토타입은 어떤 객체의 상위 객체의 역할을 하는 객체다. 프로토타입 객체의 프로퍼티나 메서드를 상속받은 하위 객체는 자신의 프로퍼티 또는 메서드인 것처럼 자유롭게 사용할 수 있다.

접근자 프로퍼티와 데이터 프로퍼티를 구별하는 방법은 다음과 같다.

```jsx
// 일반 객체의 __proto__는 접근자 프로퍼티다.
Object.getOwnPropertyDescriptor(Object.prototype, '__proto_-');

// 함수 객체의 prototype은 데이터 프로퍼티다.
Object.getOwnPropertyDescriptor(function() {}, 'prototype');
```

# 4. 프로퍼티 정의

프로퍼티 정의란 새로운 프로퍼티를 추가하면서 프로퍼티 어트리뷰트를 명시적으로 정의하거나, 기존 프로퍼티의 프로퍼티 어트리뷰트를 재정의하는 것을 말한다.

```jsx
const person = {};

// 데이터 프로퍼티 정의
Object.defineProperty(person, 'firstName', {
  value: 'Sunny',
  writable: true,
  enumerable: true,
  configurable: true,
});

Object.defineProperty(person, 'lastName', {
  value: 'Son',
});

let descriptor = Object.getOwnPropertyDescriptor(person, 'firstName');
console.log('firstName', descriptor);
// firstName {
//     value: 'Sunny',
//     writable: true,
//     enumerable: true,
//     configurable: true
//   }

// 디스크립터 객체의 프로퍼티를 누락시키면 undefined, false가 기본값이다.
descriptor = Object.getOwnPropertyDescriptor(person, 'lastName');
console.log('lastName', descriptor);
// lastName {
//     value: 'Son',
//     writable: false,
//     enumerable: false,
//     configurable: false
//   }

// lastName 프로퍼티는 [[Writable]] 의 값이 false 이므로 변경 할 수 없다.
person.lastName = 'Kim';
// lastName 프로퍼티는 [[Enumerable]] 값이 false이므로 열거되지 않음
console.log(Object.keys(person)); // ["firstName"]
// lastName 프로퍼티는 [[Configurable]] 값이 false이므로 삭제하거나 재정의할 수 없다.
delete person.lastName;
Object.defineProperty(person, 'lastName', { enumerable: true }); // SyntaxError

// 접근자 프로퍼티 정의
Object.defineProperty(person, 'fullName', {
  get() {
    return `${this.firstName} ${this.lastName}`;
  },
  set(name) {
    [this.firstName, this.lastName] = name.split(' ');
  },
  enumerable: true,
  configurable: true,
});
```

`Object.defineProperties()` 사용하면 여러 개의 프로퍼티를 한 번에 정의할 수 있다.

# 5. 객체 변경 방지

자바스크립트는 객체의 변경을 방지하는 다양한 메서드를 제공한다. 객체 변경 방지 메서드들은 객체의 변경을 금지하는 정도가 다르다.

## 5.1 객체 확장 금지

`Object.preventExtensions()` 객체의 확장을 금지한다.

**확장이 금지된 객체는 프로퍼티 추가가 금지된다.**

```jsx
const person = {
  name: 'Sunny',
};

// 확장이 금지되지 않음
console.log(Object.isExtensible(person)); // true

// 확장 금지
Object.preventExtensions(person);

// 프로퍼티 추가가 되지 않음
person.age = 20;
console.log(person); // {name: "Sunny"}

// 프로퍼티 삭제는 가능
delete person.name;
console.log(person); // {}

// 프로퍼티 정의에 의한 프로퍼티 추가도 금지
Object.defineProperty(person, 'age', { value: 20 }); // TypeError
```

## 5.2 객체 밀봉

`Object.seal()` 객체를 밀봉한다.

**밀봉된 객체는 읽기와 쓰기만 가능하다.**

```jsx
const person = {
  name: 'Sunny',
};

// 밀봉되지 않음
console.log(Object.isSealed(person)); // false

// 밀봉
Object.seal(person);

// 프로퍼티 추가가 되지 않음
person.age = 20;
console.log(person); // {name: "Sunny"}

// 프로퍼티 삭제 금지
delete person.name;
console.log(person); // {name: "Sunny"}

// 프로퍼티 값 갱신은 가능
person.name = 'Kim';
console.log(person); // {name: "Kim"}

// 프로퍼티 어트리뷰트 재정의가 금지
Object.defineProperty(person, 'name', { configurable: true }); // TypeError
```

## 5.3 객체 동결

`Object.freeze()` 객체를 동결한다.

**동결된 객체는 읽기만 가능하다.**

```jsx
const person = {
  name: 'Sunny',
};

// 동결되지 않음
console.log(Object.isFrozen(person)); // false

// 동결
Object.freeze(person);

// 프로퍼티 추가가 되지 않음
person.age = 20;
console.log(person); // {name: "Sunny"}

// 프로퍼티 삭제 금지
delete person.name;
console.log(person); // {name: "Sunny"}

// 프로퍼티 값 갱신 금지
person.name = 'Kim';
console.log(person); // {name: "Sunny"}

// 프로퍼티 어트리뷰트 재정의가 금지
Object.defineProperty(person, 'name', { configurable: true }); // TypeError
```

## 5.4 불변 객체

위 메서드들은 얕은 변경 방지(shallow only)로 직속 프로퍼티만 변경이 방지되고 중첩 객체까지는 영향을 주지는 못한다. → `Object.freeze` 로 객체를 동결하여도 중첩 객체까 동결할 수 없다.

```jsx
const person = {
  name: 'Sunny',
  address: { city: 'Seoul' },
};

// 얕은 동결
Object.freeze(person);

// 중첩 객체까지 동결하지 못함
console.log(Object.isFrozen(person)); // true
console.log(Object.isFrozen(person.address)); // false

person.address.city = 'Busan';
console.log(person); // { name: 'Sunny', address: { city: 'Busan' } }
```

객체의 중첩 객체까지 동결하여 변경이 불가능한 읽기 전용의 불변 객체를 구현하려면 객체를 값으로 갖는 모든 프로퍼티에 대해 재귀적으로 `Object.freeze()` 메서드를 호출해야 한다.

```jsx
function deepFreeze(target) {
  // 객체가 아니거나 동결된 객체는 무시하고 객체이며 동결되지 않은 객체만 동결
  if (target && typeof target === 'object' && !Object.isFrozen(target)) {
    Object.freeze(target)
    Object.keys(target).forEach(key => deepFreeze(target[key]))
  }

  return target;
}

const person = {
  name: 'Sunny',
  address: { city: 'Seoul' },
};

// 깊은 동결
deepFreeze(person);

// 중첩 객체까지 동결하지 못함
console.log(Object.isFrozen(person)); // true
console.log(Object.isFrozen(person.address)); // true

person.address.city = 'Busan';
console.log(person); // { name: 'Sunny', address: { city: 'Seoul' } }
```
