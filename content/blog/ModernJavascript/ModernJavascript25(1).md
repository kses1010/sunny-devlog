---
title: 'Modrn Javascript Deep Dive - 25장 클래스(1)'
date: 2023-08-08
category: 'Javascript'
draft: false
---

# 1. 클래스는 프로토타입의 문법적 설탕인가?

자바스크립트는 프로토타입 기반 객체지향 언어다.

프로토타입 기반 객체지향 언어는 클래스가 필요 없는 객체지향 언어다.

ES5에서는 클래스 없이 생성자 함수와 프로토타입을 통해 객체지향 언어의 상속을 구현할 수 있다.

```jsx
// ES5 생성자 함수
var Person = (function() {
  // 생성자 함수
  function Person(name) {
    this.name = name
  }

  // 프로토타입 메서드
  Person.prototype.sayHi = function() {
    console.log(`Hi! My name is ${this.name}`)
  }

  // 생성자 함수 반환
  return Person
})()

// 인스턴스 생성
var me = new Person('Son')
me.sayHi() // Hi! My name is Son
```

ES6부터 지원하는 클래스는 사실 함수이며 기존 프로토타입 기반 패턴을 클래스 기반 패턴처럼 사용할 수 있도록 하는 문법적 설탕(syntatic sugar)다.

단, 클래스와 생성자 함수는 모두 프로토타입 기반의 인스턴스를 생성하지만 정확히 동일하게 동작하지 않는다.

클래스는 생성자 함수보다 엄격하며 생성자 함수에서는 제공하지 않는 기능도 제공한다.

→ 클래스는 프로토타입 기반 객체 생성 패턴의 단순한 문법적 설탕보단 **새로운 객체 생성 메커니즘**으로 보는 것이 좀 더 합당하다.

# 2. 클래스 정의

클래스는 class 키워드를 사용하여 정의한다. 클래스 이름은 파스칼 케이스를 사용한다.

```jsx
// class 선언문
class Person {}
```

일반적이지는 않지만 함수처럼 표현식으로 클래스를 정의할 수 있다. 이때 클래스는 함수와 마찬가지로 이름을 가질 수 있고, 갖지 않을 수도 있다.

```jsx
// 익명 클래스 표현식
const Person = class {}

// 기명 클래스 표현식
const Person = class MyClass {}
```

클래스를 표현식으로 정의할 수 있다는 것은 클래스가 값으로 사용할 수 있는 일급 객체라는 것을 의미한다.

- 무명의 리터럴로 생성할 수 있다. 즉, 런타임에 생성이 가능하다.
- 변수나 자료구조(객체, 배열 등)에 저장할 수 있다.
- 함수의 매개변수에게 전달할 수 있다.
- 함수의 반환값으로 사용할 수 있다.

클래스 몸체에는 0개 이상의 메서드만 정의할 수 있다.

생성자, 프로토타입 메서드, 정적 메서드로 세가지만 정의가 가능하다.

```jsx
// 클래스 선언문
class Person {
  // 생성자
  constructor(name) {
    // 인스턴스 생성 및 초기화
    this.name = name // name 프로퍼티는 public 이다.
  }

  // 프로토타입 메서드
  sayHi() {
    console.log(`Hi! My name is ${this.name}`)
  }

  // 정적 메서드
  static sayHello() {
    console.log('Hello!')
  }
}

// 인스턴스 생성
const me = new Person('Son')

// 인스턴스의 프로퍼티 참조
console.log(me.name) // Son
// 프로토타입 메서드 호출
me.sayHi() // Hi! My name is Son
// 정적 메서드 호출
Person.sayHello() // Hello
```

위 코드는 생성자 함수의 정의 방식으로 바꾼다면

```jsx
// 클래스 선언문
var Person = (function() {
  // 생성자 함수
  function Person(name) {
    this.name = name
  }

  // 프로토타입 메서드
  Person.prototype.sayHi = function() {
    console.log(`Hi! My name is ${this.name}`)
  }

  // 정적 메서드
  Person.sayHello = function() {
    console.log('Hello!')
  }

  // 생성자 함수 반환
  return Person
})()
```

# 3. 클래스 호이스팅

클래스는 함수로 평가된다.

```jsx
// 클래스 선언문
class Person {}

console.log(typeof Person) // function
```

클래스 선언문으로 정의한 클래스는 프로토타입, 생성자 함수가 쌍으로 생성된다.

단, 클래스는 클래스 정의 이전에 참조할 수 없다.

```jsx
console.log(Person) // ReferenceError

// 클래스 선언문
class Person {}
```

클래스 선언문은 호이스팅이 발생하지 않은 것처럼 보이나 그렇지 않다.

```jsx
const Person = ''
{
  // 호이스팅이 발생하지 않는다면 '' 이 출력되어야 한다.
  console.log(Person) // ReferenceError 발생
  // 클래스 선언문
  class Person {}
}
```

클래스는 let, const 키워드로 선언한 변수처럼 호이스팅된다.

# 4. 인스턴스 생성

클래스 생성자 함수이며 new 연산자와 함께 호출되어 인스턴스를 생성한다.

```jsx
class Person {}

// 인스턴스 생성
const me = new Person()
console.log(me) // Person {}
```

만약 new 키워드를 뺀다면

```jsx
class Person {}

// TypeError
const me = Person()
```

클래스 표현식으로 정의된 클래스의 경우 식별자를 상용해 인스턴스를 생성하지 않고 기명 클래스 표현식의 클래스 이름을 사용해 인스턴스를 생성하면 에러가 발생한다.

```jsx
const Person = class MyClass {}

const me = new Person()

// 클래스 몸체 내부에서만 유효한 식별자다.
console.log(MyClass) // ReferenceError

const you = new MyClass() // ReferenceError
```

기명 함수 표현식과 마찬가지로 클래스 포현식에서 사용한 클래스 이름은 외부 코드에서 접근 불가능하다.

# 5. 메서드

클래스 몸체에는 0개 이상의 메서드만 선언할 수 있다.

constructor(생성자), 프로토타입 메서드, 정적 메서드 세가지가 있다.

## 5.1 constructor

constructor는 인스턴스를 생성하고 초기화하기 위한 특수한 메서드다.

constructor는 이름을 변경할 수 없다.

```jsx
class Person {
  // 생성자
  constructor(name) {
    // 인스턴스 생성 및 초기화
    this.name = name
  }
}
```

constructor는 메서드로 해석되는 것이 아니라 클래스가 평가되어 생성한 함수 객체 코드의 일부가 된다.

constructor는 생성자 함수와 유사하지만 몇가지 차이가 있다.

- constructor는 클래스 내에 최대 한 개만 존재할 수 있다.

```jsx
class Person {
  constructor() {}
  constructor() {}
}
// SyntaxError
```

- constructor는 생략할 수 있다.

```jsx
class Person {}
```

constructor는 생략하면 클래스에 빈 constructor가 암묵적으로 정의된다.

```jsx
class Person {
  // constructor는 생략하면 빈 constructor가 암묵적으로 정의된다.
  constructor() {}
}

// 빈 객체가 생성된다.
const me = new Person()
console.log(me) // Person {}
```

프로퍼티가 추가되어 초기화된 인스턴스를 생성하려면 constructor 내부에서 this에 인스턴스 프로퍼티를 추가한다.

```jsx
class Person {
  constructor() {
    // 고정값으로 인스턴스 초기화
    this.name = 'Son'
    this.address = 'Seoul'
  }
}

const me = new Person()
console.log(me) // Person { name: 'Son', address: 'Seoul' }
```

클래스 외부에서 인스턴스 프로퍼티의 초기값을 전달하려면 constructor에 매개변수를 선언하고 인스턴스를 생성할 때 초기값을 전달한다.

```jsx
class Person {
  constructor(name, address) {
    this.name = name
    this.address = address
  }
}

const me = new Person('Son', 'Seoul')
console.log(me) // Person { name: 'Son', address: 'Seoul' }
```

→ 인스턴스를 초기화하려면 constructor를 생략해서는 안 된다.

constructor는 별도의 반환문을 갖지 않아야 한다.

```jsx
class Person {
  constructor(name) {
    this.name = name

    // 명시적으로 객체를 반환하면 암묵적인 this 반환이 무시된다.
    // return  {};
    // 명시적으로 원시값을 반환하면 원시값 반환은 무시되고 암묵적으로 this가 반환한다.
    // return 100;
  }
}

const me = new Person('Son')
console.log(me)
// return {} 시: {}
// return 100 시: Person {name: "Son"}
```

→ constructor 내부에서 return 문을 반드시 생략해야 한다.

## 5.2 프로토타입 메서드

클래스 몸체에서 정의한 메서드는 prototype 프로퍼티에 메서드를 추가하지 않아도 기본적으로 프로토타입 메서드가 된다.

```jsx
class Person {
  // 생성자
  constructor(name) {
    this.name = name
  }

  // 프로토타입 메서드
  sayHi() {
    console.log(`Hi! My name is ${this.name}`)
  }
}

const me = new Person('Son')
me.sayHi() // Hi! My name is Son
```

생성자 함수와 마찬가지로 클래스가 생성한 인스턴스는 프로토타입 체인의 일원이 된다.

```jsx
// me 객체의 프로토타입은 Person.prototype이다.
Object.getPrototypeOf(me) === Person.prototype // true
me instanceof Person // true

// Person.prototype의 프로토타입은 Object.prototype이다.
Object.getPrototypeOf(Person.prototype) === Object.prototype // true
me instanceof Object // true

// me 객체의 constructo는 Person 클래스다.
me.constructor === Person // true
```

→ 클래스는 생성자 함수와 같이 인스턴스를 생성하는 생성자 함수라고 볼 수 있다.

→ 클래스는 생성자 함수와 마찬가지로 프로토타입 기반의 객체 생성 메커니즘이다.

## 5.3 정적 메서드

정적 메서드는 인스턴스를 생성하지 않아도 호출할 수 있는 메서드를 말한다.

```jsx
class Person {
  // 생성자
  constructor(name) {
    // 인스턴스 생성 및 초기화
    this.name = name
  }

  // 정적 메서드
  static sayHi() {
    console.log('hi')
  }
}
```

정적 메서드는 프로토타입처럼 인스턴스로 호출하지 않고 클래스로 호출한다.

```jsx
Person.sayHi() // hi
```

정적 메서드는 인스턴스로 호출할 수 없다.

인스턴스의 프로토타입 체인 상에는 클래스가 존재하지 않기 때문에 인스턴스로 클래스의 메서드를 상속받을 수 없다.

```jsx
// 인스턴스 생성
const me = new Person('Son')
me.sayHi() // TypeError
```

## 5.4 정적 메서드와 프로토타입 메서드의 차이

1. 정적 메서드와 프로토타입 메서드는 자신이 속해 있는 프로토타입 체인이 다르다.
2. 정적 메서드는 클래스로 호출하고 프로토타입 메서드는 인스턴스로 호출한다.
3. 정적 메서드는 인스턴스 프로퍼티를 참조할 수 없지만 프로토타입 메서드는 인스턴스 프로퍼티를 참조 할 수 있다.

```jsx
class Square {
  // 정적 메서드
  static area(width, height) {
    return width * height
  }
}

console.log(Square.area(10, 10)) // 100
```

정적 메서드 area는 2개의 인수를 전달받아 면적을 계산한다. 이때 정적 메서드 area는 인스턴스 프로퍼티를 참조하지 않는다.

만약 인스턴스 프로퍼티를 참조해야 한다면 정적 메서드 대신 프로토타입 메서드를 사용해야 한다.

```jsx
class Square {
  constructor(width, height) {
    this.width = width
    this.height = height
  }

  // 프로토타입 메서드
  area() {
    return this.width * this.height
  }
}

const square = new Square(10, 10)
console.log(square.area())
```

표준 빌트인 객체인 Math, Number, JSON, Object, Reflect 등은 다양한 정적 메서드를 가지고 있다.

정적 메서드는 애플리케이션 전역에서 사용할 유틸리티 함수다.

```jsx
// 표준 빌트인 객체의 정적 메서드
Math.max(1, 2, 3) // 3
Number.isNaN(NaN) // true
JSON.stringify({ a: 1 }) // "{"a": 1}"
Object.is({}, {}) // false
Reflect.has({ a: 1 }, 'a') // true
```

## 5.5 클래스에서 정의한 메서드의 특징

1. function 키워드를 생략한 메서드 축약 표현을 사용한다.
2. 객체 리터럴과는 다르게 클래스에 메서드를 정의할 때는 콤마가 필요 없다.
3. 암묵적으로 strict mode로 실행된다.
4. `for...in` 문이나 `Object.keys` 메서드 등으로 열거 할 수 없다.
   1. 프로퍼티의 열거 기능 여부를 나타내며, 불리언 값을 갖는 프로퍼티 어트리뷰트 Enumerable 값은 false다.
5. 내부 메서드 Construct 를 갖지 않은 non-constructor다.
   1. new 연산자와 함께 호출할 수 없다.
