---
title: 'Modern Javascript Deep Dive - 19장 프로토타입(1)'
date: 2023-08-04 17:48:52
category: 'Javascript'
draft: false
---

자바스크립트는 클래스 기반 객체지향 프로그래밍 언어보다 효율적이며 더 강력한 객체지향 프로그래밍 능력을 지니고 있는 프로토타입 기반의 객체지향 프로그래밍 언어다.

→ ES6에서 클래스가 도입됨.

# 1. 객체지향 프로그래밍

객체지향 프로그래밍은 실세계의 실체를 인식하는 철학적 사고를 프로그래밍에 접목하려는 시도에서 시작한다. 실체는 특징이나 성질을 나타내는 **속성(Attribute / Property)**을 가지고 있고, 이를 통해 실체를 인식하거나 구별할 수 있다.

다양한 속성 중에서 프로그램에 필요한 속성만 간추려 내어 표현하는 것을 **추상화(Abstraction)**라 한다.

```jsx
// 이름과 주소 속성을 갖는 객체
const person = {
  name: 'Son',
  address: 'Seoul',
};
```

**속성을 통해 여러 개의 값을 하나의 단위로 구성한 복합적인 자료구조를 객체**라 하며, 객체지향 프로그래밍은 독립적인 객체의 집합으로 프로그램을 표현하려는 프로그래밍 패러다임이다.

예를 들어 원(Circle)이라는 개념을 객체로 다룬다면, 반지름은 원의 **상태를 나타내는 데이터**이며 원의 지름, 둘레, 넓이를 구하는 것은 **동작**이다.

```jsx
const circle = {
  radius: 5, // 반지름

  // 원의 지름: 2r
  getDiameter() {
    return 2 * this.radius;
  },

  // 원의 둘레: 2PI r
  getPerimeter() {
    return 2 * Math.PI * this.radius;
  },

  // 원의 넓이: PIrr
  getArea() {
    return Math.PI * this.radius ** 2;
  },
}

console.log(circle);

console.log(circle.getDiameter());
console.log(circle.getPerimeter());
console.log(circle.getArea());

//   {
//     radius: 5,
//     getDiameter: [Function: getDiameter],
//     getPerimeter: [Function: getPerimeter],
//     getArea: [Function: getArea]
//   }
//   10
//   31.41592653589793
//   78.53981633974483
```

객체지향 프로그래밍은 객체의 **상태(state)**를 나타내는 데이터와 상태 데이터를 조작할 수 있는 **동작(behavior)**을 하나의 논리적인 단위로 묶어 생각한다.

**→ 객체는 상태 데이터와 동작을 하나의 논리적인 단위로 묶은 복합적인 자료구조**

# 2. 상속과 프로토타입

상속은 객체지향 프로그래밍의 핵심 개념으로, 어떤 객체의 프로퍼티 또는 메서드를 다른 객체가 상속받아 그대로 사용할 수 있는 것을 말한다.

자바스크립트는 프로토타입을 기반으로 상속을 구현하여 불필요한 중복을 제거한다.

```jsx
// 생성자 함수
function Circle(radius) {
  this.radius = radius;
  this.getArea = function() {
    return Math.PI * this.radius ** 2;
  }
}

const circle1 = new Circle(1);
const circle2 = new Circle(2);

// Circle 생성자 함수는 인스턴스를 생성할 때마다 동일한 동작을 하는
// getArea 메서드를 중복 생성하고 모든 인스턴스가 중복 소유한다.
// getArea 메서드는 하나만 생성하여 모든 인스턴스가 공유해서 사용하는 것이 바람직하다.
console.log(circle1.getArea === circle2.getArea); // false

console.log(circle1.getArea());
console.log(circle2.getArea());

// 3.141592653589793
// 12.566370614359172
```

위 예제의 생성자 함수는 문제가 있다.

radius 프로퍼티 값은 일반적으로 인스턴스마다 다르다.

하지만 getArea 메서드는 모든 인스턴스가 동일한 내용의 메서드를 사용하므로 단 하나만 생성하여 모든 인스턴스가 공유해서 사용하는 것이 바람직하다.

→ 내용이 동일한 메서드가 중복 생성된다. → 메모리 낭비

**자바스크립트는 프로토타입(prototype)을 기반으로 상속을 구현한다.**

```jsx
// 생성자 함수
function Circle(radius) {
  this.radius = radius;
}

Circle.prototype.getArea = function() {
  return Math.PI * this.radius ** 2;
}

const circle1 = new Circle(1);
const circle2 = new Circle(2);

// Circle 생성자 함수가 생성한 모든 인스턴스는 부모 객체의 역할을 하는
// 프로토타입 Circle.prototype 으로부터 getArea 메서드를 상속받는다.
// Circle 생성자 함수가 생성하는 모든 인스턴스는 하나의 getArea 메서드를 공유한다.
console.log(circle1.getArea === circle2.getArea); // true

console.log(circle1.getArea());
console.log(circle2.getArea());
```

→ 자신의 상태를 나타내는 radius 프로퍼티만 개별적으로 소유하고 내용이 동일한 메서드는 상속을 통해 공유해서 사용하는 것이다.

# 3. 프로토타입 객체

프로토타입 객체란 객체지향 프로그래밍의 근간을 이루는 객체 간 상속을 구현하기 위해 사용된다.

프로토타입은 어떤 객체의 상위 객체의 역할을 하는 객체로서 다른 객체의 공유 프로퍼티를 제공한다.

프로토타입을 상속받은 하위 객체는 상위 객체의 프로퍼티를 자신의 프로퍼티처럼 자유롭게 사용할 수 있다.

모든 객체는 하나의 프로토타입을 갖는다. 그리고 모든 프로토타입은 생성자 함수와 연결되어 있다.

Prototype 내부 슬롯에는 직접 접근할 수 없지만, `__proto__` 접근자 프로퍼티를 통해 자신의 프로토타입, 즉 자신의 Prototype 내부 슬롯이 가리키는 프로토타입에 간접적으로 접근할 수 있다.

## 3.1 `__proto__` 접근자 프로퍼티

**모든 객체는 `__proto__` 접근자 프로퍼티를 통해 자신의 프로토타입, Prototype 내부 슬롯에 간접적으로 접근할 수 있다.**

### `__proto__` 은 접근자 프로퍼티다.

자바스크립트의 내부 슬롯은 프로퍼티가 아니기 때문에 간접적으로 접근할 수 있는 수단을 제공한다.

`Object.prototype` 의 접근자 프로퍼티인 `__proto__` 는 getter/setter 함수라고 부르는 접근자 함수를 통해 Prototype 내부 슬롯의 값, 즉 프로토타입을 취득하거나 할당한다.

```jsx
const obj = {};
const parent = { x: 1 };

// getter 함수인 get __proto__ 가 호출되어 obj 객체의 프로토타입을 취득
obj.__proto__;

// setter 함수인 set __proto__ 가 호출되어 obj 객체의 프로토타입을 고려
obj.__proto__ = parent;

console.log(obj.x); // 1
```

### `__proto__` 접근자 프로퍼티는 상속을 통해 사용된다.

`__proto__` 접근자 프로퍼티는 객체가 직접 소유하는 프로퍼티가 아니라 `Object.prototype` 의 프로퍼티다. 모든 객체는 상속을 통해 `Object.prototype.__proto__` 접근자 프로퍼티를 사용할 수 있다.

```jsx
const person = { name: 'Son' };

// person 객체는 __proto__ 프로퍼티를 소유하지 않는다.
console.log(person.hasOwnProperty('__proto__')); // false

// __proto__ 프로퍼티는 모든 객체의 프로토타입 객체인 Obejct.prototype의 접근자 프로퍼티다.
console.log(Object.getOwnPropertyDescriptor(Object.prototype, '__proto__'));

//   {
//     get: [Function: get __proto__],
//     set: [Function: set __proto__],
//     enumerable: false,
//     configurable: true
//   }

// 모든 객체는 Object.prototype 의 접근자 프로퍼티 __proto__를 상속받아 사용할 수 있다.
console.log({}.__proto__ === Object.prototype); // true
```

### **proto** 접근자 프로퍼티를 통해 프로토타입에 접근하는 이유

프로토타입에 접근하기 위해 접근자 프로퍼티를 사용하는 이유는 상호 참조에 의해 프로토타입 체인이 생성되는 것을 방지하기 위해서다.

```jsx
const parent = {};
const child = {};

// child의 프로토타입을 parent로 설정
child.__proto__ = parent;
// parent의 프로토타입을 child로 설정
parent.__proto__ = child; // TypeError
```

프로토타입 체인은 단방향 링크드 리스트로 구현되어야 한다.

→ 프로퍼티 검색 방향이 한쪽 방향으로만 흘러가야 한다.

→ 아무런 체크 없이 무조건적으로 프로토타입을 교체할 수 없도록 `__proto__` 접근자 프로퍼티를 통해 프로토타입에 접근하고 교체하도록 구현되어 있다.

### **proto** 접근자 프로퍼티를 코드 내에서 직접 사용하는 것은 권장하지 않는다.

코드 내에서 `__proto__` 접근자 프로퍼티를 직접 사용하는 것은 권장하지 않는다. 모든 객체가 `__proto__` 접근자 프로퍼티를 사용할 수 있는 것은 아니기 때문이다.

```jsx
// obj는 프로토타입 체인의 종점이다. 따라서 Object.__proto__를 상속받을 수 없다.
const obj = Object.create(null);

// obj는 Object.__proto__를 상속받을 수 없다.
console.log(obj.__proto__); // undefined

// 따라서 __proto__보다 Object.getPrototypeOf 메서드를 사용하는 편이 좋다.
console.log(Object.getPrototypeOf(obj)); // null
```

- `Object.getPrototypeOf()` : 프로토타입의 참조를 취득하고 싶은 경우
- `Object.setPrototypeOf()` : 프로토타입을 교체하고 싶은 경우

```jsx
const obj = {};
const parent = { x: 1 };

// obj 객체의 프로토타입을 취득
Object.getPrototypeOf(obj); // obj.__proto__
// obj 객체의 프로토타입을 교체
Object.setPrototypeOf(obj, parent); // obj.__proto__ = parent

console.log(obj.x); // 1
```

## 3.2 함수 객체의 prototype 프로퍼티

**함수 객체만이 소유하는 prototype 프로퍼티는 생성자 함수가 생성할 인스턴스의 프로토타입을 가리킨다.**

```jsx
// 함수 객체는 prototype 프로퍼티를 소유한다.
(function () {}.hasOwnProperty("prototype")); // true

// 일반 객체는 prototype 프로퍼티를 소유하지 않는다.
({}.hasOwnProperty("prototype")); // false
```

→ non-constructor 인 화살표 함수와 ES6 메서드 축약 표현으로 정의한 메서드는 prototype 프로퍼티를 소유하지 않으며 프로토타입도 생성하지 않는다.

```jsx
// 화살표 함수는 non-constructor 다.
const Person = name => {
  this.name = name
}

// non-constructor는 prototype 프로퍼티를 소유하지 않는다.
console.log(Person.hasOwnProperty('prototype')); // false

// non-constructor는 프로토타입을 생성하지 않는다.
console.log(Person.prototype); // undefined

// ES6의 메서드 축약 표현으로 정의한 메서드는 non-constructor다.
const obj = {
  foo() {},
};

// non-constructor는 prototype 프로퍼티를 소유하지 않는다.
console.log(obj.foo.hasOwnProperty('prototype')); // false

// non-constructor는 프로토타입을 생성하지 않는다.
console.log(obj.foo.prototype); // undefined
```

**모든 객체가 가지고 있는 `__proto__` 접근자 프로퍼티와 함수 객체만이 가지고 있는 prototype 프로퍼티는 결국 동일한 프로토타입을 가리킨다.**

| 구분        | 소유          | 값         | 사용 주체  | 사용 목적                                 |
|-----------|-------------|-----------|--------|---------------------------------------|
| proto     | 모든 객체       | 프로토타입의 참조 | 모든 객체  | 객체가 자신의 프로토타입에 접근 또는 교체하기 위해 사용       |
| prototype | constructor | 프로토타입의 참조 | 생성자 함수 | 생성자 함수가 자신이 생성할 객체의 프로토타입을 할당하기 위해 사용 |

```jsx
// 생성자 함수
function Person(name) {
  this.name = name;
}

const me = new Person('Son');

// 결국 Person.prototype과 me.__proto__는 결국 동일한 프로토타입을 가리킨다.
console.log(Person.prototype === me.__proto__); // true
```

## 3.3 프로토타입의 constructor 프로퍼티와 생성자 함수

모든 프로토타입은 constructor 프로퍼티를 갖는다. 이 constructor 프로퍼티는 prototype 프로퍼티로 자신을 참조하고 있는 생성자 함수를 가리킨다.

→ 생성자 함수가 생성될 때, 함수 객체가 생성될 때 이뤄진다.

```jsx
// 생성자 함수
function Person(name) {
  this.name = name;
}

const me = new Person('Son');

// me 객체의 생성자 함수는 Person이다.
console.log(me.constructor === Person); // true
```

→ me 객체는 프로토타입인 `Person.prototype` 의 constructor 프로퍼티를 상속받아 사용할 수 있다.

# 4. 리터럴 표기법에 의해 생성된 객체의 생성자 함수와 프로토타입

생성자 함수에 의해 생성된 인스턴스는 프로토타입의 constructor 프로퍼티에 의해 생성자 함수가 연결된다.

→ constructor 프로퍼티가 가리키는 생성자 함수는 인스턴스를 생성한 생성자 함수다.

```jsx
// obj 객체를 생성한 생성자 함수는 Object다.
const obj = new Object();
console.log(obj.constructor === Object); // true

// add 함수 객체를 생성한 생성자 함수는 Function이다.
const add = new Function('a', 'b', 'return a + b');
console.log(add.constructor === Function); // true

// 생성자 함수
function Person(name) {
  this.name = name;
}

// me 객체를 생성한 생성자 함수는 Person이다.
const me = new Person('Son');
console.log(me.constructor === Person); // true
```

하지만 리터럴 표기법에 의한 객체 생성 방식과 같이 명시적으로 new 연산자와 함께 생성자 함수를 호출하여 인스턴스를 생성하지 않는 객체 생성 방식도 있다.

```jsx
// 객체 리터럴
const obj = {};

// 함수 리터럴
const add = function(a, b) {
  return a + b;
}

// 배럴 리터럴
const arr = [1, 2, 3];

// 정규 표현식 리터럴
const regexp = /is/gi;
```

리터럴 표기법에 의해 생성된 객체도 물론 프로토타입이 존재한다.

하지만 리터럴 표기법에 의해 생성된 객체인 경우 프로토타입의 constructor 프로퍼티가 가리키는 생성자 함수가 반드시 객체를 생성한 생성자 함수라고 단정할 수 없다.

```jsx
// obj 객체는 Object 생성자 함수로 생성한 객체가 아니라 객체 리터럴로 생성.
const obj = {};

// 하지만 obj 객체의 생성자 함수는 Object 생성자 함수다.
console.log(obj.constructor === Object); // true
```

```jsx
// Object 생성자 함수에 의한 객체 생성
// 인수가 전달되지 않았을 때 추상 연산 OrdinaryObjectCreate를 호출하여 빈 객체를 생성한다.
let obj = new Object();
console.log(obj); // {}

// new.target이 undefined나 Object가 아닌 경우
// 인스턴스 -> Foo.prototype -> Object.prototype 순으로 프로토타입 체인이 생성된다.
class Foo extends Object {};
new Foo(); // Foo {}

// 인수가 전달된 경우에는 인수를 객체로 변환한다.
// Number 객체 생성
obj = new Object(123);
console.log(obj); // Number {123}

// String 객체 생성
obj = new Object('123');
console.log(obj); // String {"123"}
```

함수 선언문과 함수 표현식을 평가하여 함수 객체를 생성한 것은 Function 생성자 함수가 아니다.

→ constructor 프로퍼티를 통해 확인해보면 foo 함수의 생성자 함수는 Function 생성자 함수다.

```jsx
// foo 함수는 Function 생성자 함수로 생성한 함수 객체가 아니라 함수 선언문으로 생성한다.
function foo() {};

// constructor 프로퍼티를 통해 확인해보면 함수 foo의 생성자 함수는 Function 생성자 함수다.
console.log(foo.constructor === Function); // true
```

**프로토타입과 생성자 함수는 단독으로 존재할 수 없고 언제나 쌍으로 존재한다.**

함수 리터럴에 의해 생성한 함수와 Function 생성자 함수에 의해 생성한 함수는 생성 과정과 스코프, 클로저 등의 차이가 있지만 결국 함수로서 동일한 특성을 갖는다.

→ 프로토타입의 constructor 프로퍼티를 통해 연결되어 있는 생성자 함수를 리터럴 표기법으로 생성한 객체를 생성한 생성자 함수로 생각해도 크게 무리는 없다.

# 5. 프로토타입의 생성 시점

객체는 리터럴 표기법 또는 생성자 함수에 의해 생성되므로 결국 모든 객체는 생성자 함수와 연결되어 있다.

**프로토타입은 생성자 함수가 생성되는 시점에 더불어 생성된다.**

## 5.1 사용자 정의 생성자 함수와 프로토타입 생성 시점

**생성자 함수로서 호출할 수 있는 함수, 즉 constructor는 함수 정의가 평가되어 함수 객체를 생성하는 시점에 프로토타입도 더불어 생성된다.**

```jsx
// 함수 정의가 평가되어 함수 객체를 생성하는 시점에 프로토타입도 더불어 생성된다.
console.log(Person.prototype); // {constructor: f}

// 생성자 함수
function Person(name) {
  this.name = name;
}
```

```jsx
// 화살표 함수는 non-constructor다.
const Person = name => {
  this.name = name;
}

// non-constructor 는 프로토타입이 생성되지 않는다.
console.log(Person.prototype);
```

## 5.2 빌트인 생성자 함수와 프로토타입 생성 시점

모든 빌트인 생성자 함수는 전역 객체가 생성되는 시점에 생성된다. 생성된 프로토타입은 빌트인 생성자 함수의 prototype 프로퍼티에 바인딩된다.

### 전역 객체

전역 객체는 표준 빌트인 객체들과 환경에 따른 호스트 객체, 그리고 var 키워드로 선언한 전역 변수와 전역 함수를 프로퍼티로 갖는다. → (Math, Reflect, JSON)을 제외한 표준 빌트인 객체는 모두 생성자 함수다.

```jsx
window.Object === Object; // true
```

`Object` 도 전역 객체의 프로퍼티이며, 전역 객체가 생성되는 시점에 생성된다.

객체가 생성되기 이전에 생성자 함수와 프로토타입은 이미 객체화되어 존재한다. **이후 생성자 함수 또는 리터럴 표기법으로 객체를 생성하면 프로토타입은 생성된 객체의 Prototype 내부 슬롯에 할당된다.**

# 6. 객체 생성 방식과 프로토타입의 결정

## 6.1 객체 리터럴에 의해 생성된 객체의 프로토타입

```jsx
const obj = { x: 1};

// 객체 리터럴에 의해 생성된 obj 객체는 Object.prototype을 상속받는다.
console.log(obj.constructor === Object); // true
console.log(obj.hasOwnProperty("x"); // true
```

위 객체 리터럴이 평가되면 Object 함수와 `Object.prototype` 과 생성된 객체 사이에 연결이 만들어 진다.

## 6.2 Object 생성자 함수에 의해 생성된 객체의 프로토타입

```jsx
const obj = new Object();
obj.x = 1;

// Object 생성자 함수에 의해 obj 객체는 Object.prototype을 상속받는다.
console.log(obj.constructor === Object); // true
console.log(obj.hasOwnProperty("x"); // true
```

Object 생성자 함수에 의해 생성된 obj 객체는 Object.prototype을 프로토타입으로 갖게 되며, 이로써 Object.prototype을 상속받는다.

객체 리터럴 방식은 객체 리터럴 내부에 프로퍼티를 추가하지만 Object 생성자 함수 방식은 일단 빈 객체를 생성한 이후 프로퍼티를 추가해야 한다.

## 6.3 생성자 함수에 의해 생성된 객체에 프로토타입

생성자 함수에 의해 생성되는 객체의 프로토타입은 생성자 함수의 prototype 프로퍼티에 바인딩되어 있는 객체다.

```jsx
function Person(name) {
  this.name = name;
}

const me = new Person('Son');
```

프로토타입 Person.prototype 에 프로퍼티를 추가하여 하위 객체가 상속받을 수 있도록 구현한다면

```jsx
function Person(name) {
  this.name = name;
}

// 프로토타입 메서드
Person.prototype.sayHello = function() {
  console.log(`Hi! My name is ${this.name}`);
}

const me = new Person('Son');
const you = new Person('Kim');

me.sayHello();
you.sayHello();

// Hi! My name is Son
// Hi! My name is Kim
```

Person 생성자 함수를 통해 생성된 모든 객체는 프로토타입에 추가된 `sayHello` 메서드를 상속받아 자신의 메서드처럼 사용할 수 있다.

# 7. 프로토타입 체인

```jsx
function Person(name) {
  this.name = name;
}

// 프로토타입 메서드
Person.prototype.sayHello = function() {
  console.log(`Hi! My name is ${this.name}`);
}

const me = new Person('Son');

console.log(me.hasOwnProperty('name')); // true
```

Person 생성자 함수에 의해 생성된 me 객체는 Object.prototype의 메서드인 hasOwnProperty를 호출할 수 있다.

me 객체가 Person.prototype 뿐만 아니라 Object.prototype 도 상속받았다는 것을 의미한다.

```jsx
Object.getPrototypeOf(me) === Person.prototype; // true
Object.getPrototypeOf(Person.prototype) === Object.prototype; // true
```

자바스크립트는 객체의 프로퍼티(메서드 포함)에 접근하려고 할 때 해당 객체에 접근하려는 프로퍼티가 없다면 Prototype 내부 슬롯의 참조를 따라 자신의 부모 역할을 하는 프로토타입의 프로퍼티를 순차적으로 검색한다.

→ 프로토타입 체인이라 한다.

프로토타입 체인은 자바스크립트가 객체지향 프로그래밍의 상속을 구현하는 메커니즘이다.

```jsx
// hasOwnProperty는 Object.prototype의 메서드다.
// me 객체는 프로토타입 체인을 따라 hasOwnProperty 메서드를 검색하여 사용한다.
me.hasOwnProperty('name'); // true
```

프로토타입 체인의 최상위에 위치하는 객체는 언제나 Object.prototype이다.

→ 모든 객체는 Object.prototype을 상속받는다. **Object.prototype을 프로토타입 체인의 종점이라 한다.**

Object.prototype의 프로토타입, 즉 Prototype 내부 슬롯의 값은 null 이다.

프로토타입 체인의 종점인 Object.prototype에서도 프로퍼티를 검색할 수 없는 경우 undefined를 반환한다.

```jsx
console.log(me.foo); // undefined
```

자바스크립트 엔진은 프로토타입 체인을 따라 프로퍼티 / 메서드를 검색한다.

**→ 프로토타입 체인은 상속과 프로퍼티 검색을 위한 메커니즘이다.**

**→ 스코프 체인은 식별자 검색을 위한 메커니즘이다.**

스코프 체인과 프로토타입 체인은 서로 연관없이 별도로 동작하는 것이 아니라 서로 협력하여 식별자와 프로퍼티를 검색하는데 사용된다.

# 8. 오버라이딩과 프로퍼티 섀도잉

```jsx
const Person = (function() {
  // 생성자 함수
  function Person(name) {
    this.name = name;
  }

  // 프로토타입 메서드
  Person.prototype.sayHello = function() {
    console.log(`Hi! My name is ${this.name}`);
  }

  // 생성자 함수를 반환
  return Person
})();

const me = new Person('Son');

// 인스턴스 메서드
me.sayhello = function() {
  console.log(`Hey! My name is ${this.name}`);
}

// 인스턴스 메서드가 호출된다. 프로토타입 메서드는 인스턴스 메서드에 의해 가려진다.
me.sayHello();

// Hey! My name is Son
```

프로토타입 프로퍼티와 같은 이름의 프로퍼티를 인스턴스에 추가하면 프로토타입 체인에 따라 프로토타입 프로퍼티를 검색하여 프로토타입 프로퍼티를 덮어쓰는 것이 아니라 인스턴스 프로퍼티로 추가한다.

인스턴스 메서드 sayHello는 프로토타입 메서드 sayHello를 오버라이딩햇고 프로토타입 메서드 sayHello는 가려진다. → 상속 관계에 의해 프로퍼티가 가려지는 현상을 프로퍼티 섀도잉이라 한다.

```jsx
// 인스턴스 메서드를 삭제
delete me.sayHello;
// 인스턴스에는 sayHello 메서드가 없으므로 프로토타입 메서드가 호출
me.sayHello();
```

하위 객체를 통해 프로토타입의 프로퍼티를 변경 또는 삭제는 불가능하다.

프로토타입 프로퍼티를 변경 또는 삭제하려면 하위 객체를 프로토타입 체인으로 접근하는 것이 아니라 프로토타입에 직접 접근해야 한다.

```jsx
// 프로토타입 메서드 삭제
delete Person.prototype.sayHello;
me.sayHello(); // TypeError
```

# 9. 프로토타입의 교체

프로토타입은 생성자 함수 또는 인스턴스에 의해 교체할 수 있다.

## 9.1 생성자 함수에 의한 프로토타입의 교체

```jsx
const Person = (function() {
  function Person(name) {
    this.name = name;
  }

  // 생성자 함수의 prototype 프로퍼티를 통해 프로토타입을 교체
  Person.prototype = {
    sayHello() {
      console.log(`Hi! My name is ${this.name}`);
    },
  }

  return Person;
})();

const me = new Person('Son');
```

Person 생성자 함수가 생성할 객체의 프로토타입을 객체 리터럴로 교체한 것이다.

프로토타입으로 교체한 객체 리터럴에는 constructor 프로퍼티가 없다.

→ me 객체의 생성자 함수를 검색하면 Person이 아닌 Object가 나온다.

```jsx
// 프로토타입을 교체하면 constructor 프로퍼티와 생성자 함수 간의 연결이 파괴된다.
console.log(me.constructor === Person); // false
// 프로토타입 체인을 따라 Object.prototype의 constructor 프로퍼티가 검색된다.
console.log(me.constructor === Object); // true
```

프로토타입으로 교체한 객체 리터럴에 constructor 프로퍼티를 추가하여 프로토타입의 constructor 프로퍼티를 되살린다.

```jsx
const Person = (function() {
  function Person(name) {
    this.name = name;
  }

  // 생성자 함수의 prototype 프로퍼티를 통해 프로토타입을 교체
  Person.prototype = {
    // constructor 프로퍼티와 생성자 함수 간의 연결을 설정
    constructor: Person,
    sayHello() {
      console.log(`Hi! My name is ${this.name}`);
    },
  }

  return Person;
})();

const me = new Person('Son');

// constructor 프로퍼티와 생성자 함수를 가리킨다.
console.log(me.constructor === Person); // true
console.log(me.constructor === Object); // false
```

## 9.2 인스턴스에 의한 프로토타입의 교체

프로토타입은 생성자 함수의 prototype 프로퍼티뿐만 아니라 인스턴스의 `__proto__` 접근자 프로퍼티를 통해 접근 할 수 있다.

```jsx
function Person(name) {
  this.name = name;
}

const me = new Person('Son');

// 프로토타입으로 교체할 객체
const parent = {
  sayHello() {
    console.log(`Hi! My name is ${this.name}`);
  },
};

// me 객체의 프로토타입을 parent 객체로 교체한다.
Object.setPrototypeOf(me, parent);
// 위 코드는 아래의 코드와 동일하게 동작한다.
// me.__proto__ = parent;

me.sayHello(); // Hi! My name is Son
```

프로토타입으로 교체한 객체에는 constructor 프로퍼티가 없으므로 constructor 프로퍼티와 생성자 함수 간의 연결이 파괴된다.

→ 프로토타입 constructor 프로퍼티가 me 객체의 생성자 함수를 검색하면 Person이 아닌 Object가 나온다.

프로토타입으로 교체한 객체리터럴에 constructor 프로퍼티를 추가하고 생성자 함수의 prototype 프로퍼티를 재설정하여 파괴된 생성자 함수와 프로토타입 간의 연결을 되살린다면

```jsx
function Person(name) {
  this.name = name;
}

const me = new Person('Son');

// 프로토타입으로 교체할 객체
const parent = {
  // constructor 프로퍼티와 생성자 함수 간의 연결을 설정
  constructor: Person,
  sayHello() {
    console.log(`Hi! My name is ${this.name}`)
  },
};

// 생성자 함수의 prototype 프로퍼티와 프로토타입 간의 연결을 설정
Person.prototype = parent;

// me 객체의 프로토타입을 parent 객체로 교체한다.
Object.setPrototypeOf(me, parent);
// 위 코드는 아래의 코드와 동일하게 동작한다.
// me.__proto__ = parent;

me.sayHello();

// constructor 프로퍼티와 생성자 함수를 가리킨다.
console.log(me.constructor === Person); // true
console.log(me.constructor === Object); // false

// 생성자 함수의 prototype 프로퍼티가 교체된 프로토타입을 가리킨다.
console.log(Person.prototype === Object.getPrototypeOf(me)); // true
```

프로토타입 교체를 통해 객체 간의 상속 관계를 동적으로 변경하는 것은 꽤나 번거롭다.

→ 프로토타입은 직접 교체하지 않는 것이 좋다.

상속 관계를 인위적으로 설정하려면 직접 상속을 사용하거나 ES6에서 도입된 클래스를 간편하고 직관적으로 상속 관계를 구현하면 된다.

# 10. instanceof 연산자

```jsx
객체 instanceof 생성자 함수
```

우변의 생성자 함수의 prototype에 따라 바인딩된 객체가 좌변의 객체의 프로토타입 체인 상에 존재하면 true로 평가되고, 그렇지 않은 경우에는 false로 평가된다.

```jsx
// 생성자 함수
function Person(name) {
  this.name = name;
}

const me = new Person('Son');

// Person.prototype이 me 객체의 프로토타입 체인 상에 존재하므로 true로 평가된다.
console.log(me instanceof Person); // true

// Object.prototype이 me 객체의 프로토타입 체인 상에 존재하므로 true로 평가된다.
console.log(me instanceof Object); // true
```

만약 프로토타입을 교체한다면

```jsx
// 생성자 함수
function Person(name) {
  this.name = name;
}

const me = new Person('Son');

// 프로토타입으로 교체할 객체
const parent = {};

// 프로토타입으로 교체
Object.setPrototypeOf(me, parent);

// Person 생성자 함수와 parent 객체는 연결되어 있지 않다.
console.log(Person.prototype === parent); // false
console.log(parent.constructor === Person); // false

// Person.prototype이 me 객체의 프로토타입 체인 상에 존재하지 않아 false로 평가된다.
console.log(me instanceof Person); // false

// Object.prototype이 me 객체의 프로토타입 체인 상에 존재하므로 true로 평가된다.
console.log(me instanceof Object); // true
```

프로토타입으로 교체한 parent 객체를 Person 생성자 함수의 prototype 프로퍼티에 바인딩 하면 true로 평가가 된다.

```jsx
// 생성자 함수
function Person(name) {
  this.name = name;
}

const me = new Person('Son');

// 프로토타입으로 교체할 객체
const parent = {};

// 프로토타입으로 교체
Object.setPrototypeOf(me, parent);

// Person 생성자 함수와 parent 객체는 연결되어 있지 않다.
console.log(Person.prototype === parent); // false
console.log(parent.constructor === Person); // false

// parent 객체를 Person 생성자 함수의 prototype 프로퍼티에 바인딩한다.
Person.prototype = parent;

// Person.prototype이 me 객체의 프로토타입 체인 상에 존재하므로 true로 평가된다.
console.log(me instanceof Person); // true

// Object.prototype이 me 객체의 프로토타입 체인 상에 존재하므로 true로 평가된다.
console.log(me instanceof Object); // true
```
