---
title: 'Modrn Javascript Deep Dive - 25장 클래스(2)'
date: 2023-08-08
category: 'Javascript'
draft: false
---

# 6. 클래스의 인스턴스 생성 과정

new 연산자와 함께 클래스를 호출하면 생성자 함수와 마찬가지로 클래스의 내부 메서드 Construct 가 호출된다.

클래스는 new 연산자 없이 호출할 수 없다.

클래스는 다음 과정을 거쳐 인스턴스가 생성된다.

### 1. 인스턴스 생성과 this 바인딩

### 2. 인스턴스 초기화

### 3. 인스턴스 반환

```jsx
class Person {
  // 생성자
  constructor(name) {
    // 1. 암묵적으로 인스턴스가 생성되고 this가 바인딩된다.
    console.log(this) // Person {}
    console.log(Obejct.getPrototypeOf(this) === Person.prototype) // true

    // 2. this에 바인딩되어 있는 인스턴스를 초기화한다.
    this.name = name

    // 3. 완성된 인스턴스가 바인딩된 this가 암묵적으로 반환한다.
  }
}
```

# 7. 프로퍼티

## 7.1 인스턴스 프로퍼티

인스턴스 프로퍼티는 constructor 내부에서 정의해야 한다.

```jsx
class Person {
  constructor(name) {
    // 인스턴스 프로퍼티
    this.name = name // name 프로퍼티는 public 이다.
  }
}

const me = new Person('Son')
console.log(me) // Person { name: 'Son' }
console.log(me.name) // Son
```

## 7.2 접근자 프로퍼티

접근자 프로퍼티는 자체적으로 값을 갖지 않고 다른 데이터 프로퍼티의 값을 읽거나 저장할 때 사용하는 접근자 함수로 구성된 프로퍼티다.

```jsx
class Person {
  constructor(firstName, lastName) {
    this.firstName = firstName
    this.lastName = lastName
  }

  // fullName은 접근자 함수로 구성된 접근자 프로퍼티다.
  // getter 함수
  get fullName() {
    return `${this.firstName} ${this.lastName}`
  }

  // setter 함수
  set fullName(name) {
    ;[this.firstName, this.lastName] = name.split(' ')
  }
}

const me = new Person('Sunny', 'Son')

// 데이터 프로퍼티를 통한 프로퍼티 값 참조
console.log(`${me.firstName} ${me.lastName}`) // Sunny Son

// 접근자 프로퍼티를 통한 프로퍼티 값 저장
me.fullName = 'Cloud Kim'
console.log(me) // Person { firstName: 'Cloud', lastName: 'Kim' }

// 접근자 프로퍼티를 통한 프로퍼티 값의 참조
console.log(me.fullName) // Cloud Kim

// fullName은 접근자 프로퍼티다.
console.log(Object.getOwnPropertyDescriptor(Person.prototype, 'fullName'))

// {
//  get: [Function: get fullName],
//  set: [Function: set fullName],
//  enumerable: false,
//  configurable: true
// }
```

getter와 setter 이름은 인스턴스 프로퍼티처럼 사용된다.

getter, setter는 호출하는 것이 아니라 프로퍼티처럼 참조하는 형식으로 사용된다.

setter는 단 하나의 값만 할당받기 때문에 단 하나의 매개변수만 선언할 수 있다.

## 7.3 클래스 필드 정의 제안

클래스 필드는 클래스 기반 객체지향 언어에서 클래스가 생성할 인스턴스의 프로퍼티를 가리키는 용어다.

자바스크립트의 클래스 몸체에는 메서드만 선언할 수 있다. 자바처럼 클래스 필드를 선언하면 문법적 에러가 발생한다.

```jsx
class Person {
  // 클래스 필드 정의
  name = 'Son'
}

const me = new Person('Son')
```

그러나 이 코드는 정상 동작한다. ECMAScript의 정식 표준 사양으로 승급되진 않았지만 최신 브라우저(Chrome 72 이상), 최신 Node.js(버전 12 이상)는 미리 구현해 놓았다.

클래스 몸체에서 클래스 필드를 정의하는 경우 this에 클래스 필드를 바인딩해서는 안된다.

```jsx
class Person {
	this.name = ""; // SyntaxError
}
```

자바스크립트에서는 this를 반드시 사용해야 한다.

클래스 필드에 초기값을 할당하지 않으면 undefined를 갖는다.

외부의 초기값으로 클래스 필드를 초기화 한다면 constructor를 사용하여 초기화해야 한다.

```jsx
class Person {
  name
  constructor(name) {
    // 클래스 필드 초기화
    this.name = name
  }
}

const me = new Person('Son')
console.log(me) // Person { name: 'Son' }
```

함수는 일급 객체이므로 함수를 클래스 필드에 할당할 수 있다.

→ 클래스 필드를 통해 메서드를 정의할 수도 있다.

```jsx
class Person {
  name = 'Son'

  // 클래스 필드에 함수를 할당
  getName = function() {
    return this.name
  }

  // 화살표로도 가능
  // getName = () => this.name;
}

const me = new Person()
console.log(me) // Person { name: 'Son', getName: [Function: getName] }
console.log(me.getName()) // Son
```

클래스 필드에 함수를 할당하는 경우, 이 함수는 프로퍼티 메서드가 아닌 인스턴스 메서드가 된다.

모든 클래스 필드는 인스턴스 프로퍼티가 되기 때문이다.

→ 클래스 필드에 함수를 할당하는 것은 권장하지 않는다.

## 7.4 private 필드 정의 제안

자바스크립트의 인스턴스 프로퍼티는 인스턴스를 통해 클래스 외부에서 언제나 참조할 수 있다.

→ 언제나 public 이다.

```jsx
class Person {
  constructor(name) {
    this.name = name // 기본적으로 public
  }
}

const me = new Person('Son')
console.log(me.name) // Son
```

현재 private 필드를 정의할 수 있는 새로운 표준 사양이 제안되었으며, 최신 브라우저(Chrome 74 이상), Node.js(버전 12이상)에 이미 구현되어 있다.

private 필드의 선두에는 #을 붙여준다. private 필드를 참조할 때도 #을 붙여주어야 한다.

```jsx
class Person {
	// private 필드 정의
	#name = "";
	constructor(name) {
		// private 필드 참조
		this.#name = name;
	}
}

const me = new Person("Son");
// 외부에서 참조 불가
console.log(me.#name); // SyntaxError
```

클래스 외부에서 private 필드에 직접 접근할 수 있는 방법은 없다. 다만 접근자 프로퍼티를 통해 간접적으로 접근하는 방법은 유효하다.

```jsx
class Person {
  // private 필드 정의
  #name = ''
  // #name; 도 가능
  constructor(name) {
    this.#name = name
  }

  // name 은 접근자 프로퍼티다.
  get name() {
    // private 필드를 참조하여 trime한 다음 반환한다.
    return this.#name.trim()
  }
}

const me = new Person(' Son ')
console.log(me.name) // Son
```

private 필드는 반드시 클래스 몸체에 정의해야 한다.

private 필드를 직접 constructor에 정의하면 에러가 발생한다.

## 7.5 static 필드 정의 제안

static 필드도 정의 가능하도록 표준 사양이 제안되었으며, 최신 브라우저(Chrome 72 이상), Node.js(버전 12이상)에 이미 구현되어 있다.

```jsx
class MyMath {
  // static public 필드 정의
  static PI = 22 / 7

  // static private 필드 정의
  static #num = 10

  // static 메서드
  static increment() {
    return ++MyMath.#num
  }
}

console.log(MyMath.PI) // 3.142857142857143
console.log(MyMath.increment()) // 11
```

# 8. 상속에 의한 클래스 확장

## 8.1 클래스 상속과 생성자 함수 상속

**상속에 의한 클래스 확장은 기존 클래스를 상속받아 새로운 클래스를 확장하여 정의하는 것이다.**

```jsx
class Animal {
  constructor(age, weight) {
    this.age = age
    this.weight = weight
  }

  eat() {
    return 'eat'
  }

  move() {
    return 'move'
  }
}

// 상속을 통해 Animal 클래스를 확장한 Bird 클래스
class Bird extends Animal {
  fly() {
    return 'fly'
  }
}

const bird = new Bird(1, 5)
console.log(bird) // Bird { age: 1, weight: 5 }
console.log(bird instanceof Bird) // true
console.log(bird instanceof Animal) // true

console.log(bird.eat()) // eat
console.log(bird.move()) // move
console.log(bird.fly()) // fly
```

## 8.2 extends 키워드

상속을 통해 클래스를 확장하려면 extends 키워드를 사용하여 상속받을 클래스를 정의한다.

```jsx
// 수퍼(파생/부모) 클래스
class Base {}

// 서브(파생/자식) 클래스
class Derived extends Base {}
```

extends 키워드의 역할은 수퍼클래스와 서브클래스 간의 상속 관계를 설정하는 것이다.

클래스도 프로토타입을 통해 상속 관계를 구현한다.

수퍼클래스와 서브클래스는 인스턴스의 프로토타입 체인뿐 아니라 클래스 간의 프로토타입 체인도 생성한다.

→ 프로토타입 메서드, 정적 메서드 모두 상속이 가능하다.

## 8.3 동적 상속

extends 키워드는 클래스뿐만 아니라 생성자 함수를 상속받아 클래스를 확장할 수도 있다.

단, extends 키워드 앞에는 반드시 클래스가 와야 한다.

```jsx
// 생성자 함수
function Base(a) {
  this.a = a
}

// 생성자 함수를 상속받는 서브클래스
class Derived extends Base {}

const derived = new Derived(1)
console.log(derived) // Derived { a: 1 }
```

extends 키워드 다음에는 클래스뿐만이 아니라 Construct 내부 메서드를 갖는 함수 객체로 평가될 수 있는 모든 표현식을 사용할 수 있다. → 동적으로 상속받을 대상을 결정할 수 있다.

```jsx
function Base1() {}

class Base2 {}

let condition = true

// 조건에 따라 동적으로 상속 대상을 결정하는 서브클래스
class Derived extends (condition ? Base1 : Base2) {}

const derived = new Derived()
console.log(derived) // Derived {}

console.log(derived instanceof Base1) // true
console.log(derived instanceof Base2) // false
```

## 8.4 서브클래스의 constructor

서브클래스에서 constructor를 생략하면 클래스는 다음과 같은 constructor가 암묵적으로 정의된다.

args는 new 연산자와 함께 클래스를 호출할 때 전달한 인수의 리스트다.

```jsx
constructor(...args) {super(...args);}
```

super()는 수퍼클래스의 constructor(super-constructor)를 호출하여 인스턴스를 생성한다.

```jsx
// 수퍼클래스
class Base {}

// 서브클래스
class Derived extends Base {}

// 위 코드는 아래의 코드를 생략한 코드다.

class Base {
  constructor() {}
}

class Derived extends Base {
  constructor(...args) {
    super(...args)
  }
}

const derived = new Derived()
console.log(derived) // Derived {}
```

프로퍼티를 소유하는 인스턴스를 생성하려면 constructor 내부에서 프로퍼티를 추가해야 한다.

## 8.5 super 키워드

super 는 다음과 같이 동작한다.

- super를 호출하면 수퍼클래스의 constructor(super-constructor)를 호출한다.
- super를 참조하면 수퍼클래스의 메서드를 호출할 수 있다.

### super 호출

**super를 호출하면 수퍼클래스의 constructor(super-constructor)를 호출한다.**

```jsx
// 수퍼클래스
class Base {
  constructor(a, b) {
    this.a = a
    this.b = b
  }
}

// 서브클래스
class Derived extends Base {
  // 다음과 같이 암묵적으로 constructor 가 정의된다.
  // constructor(...args) {
  // 	super(...args);
  // }
}

const derived = new Derived(1, 2)
console.log(derived) // Derived { a: 1, b: 2 }
```

```jsx
// 수퍼클래스
class Base {
  constructor(a, b) {
    this.a = a
    this.b = b
  }
}

// 서브클래스
// 서브클래스의 프로퍼티를 추가하려면 super와 함께 호출한다.
class Derived extends Base {
  constructor(a, b, c) {
    super(a, b)
    this.c = c
  }
}

const derived = new Derived(1, 2, 3)
console.log(derived)
```

**super를 호출할 때 주의할 사항은 다음과 같다.**

1. 서브클래스에서 constructor를 생략하지 않는 경우 서브클래스의 constructor에서는 반드시 super를 호출해야 한다.

```jsx
// 수퍼클래스
class Base {}

// 서브클래스
class Derived extends Base {
  constructor() {
    // ReferenceError
    console.log('constructor call')
  }
}

const derived = new Derived()
```

1. 서브클래스의 constructor에서 super를 호출하기 전에는 this를 참조할 수 없다.

```jsx
// 수퍼클래스
class Base {}

// 서브클래스
class Derived extends Base {
  constructor() {
    // ReferenceError
    console.log('constructor call')
    this.a = 1
    super()
  }
}

const derived = new Derived(1)
```

1. super는 반드시 서브클래스의 constructor에서만 호출한다.

```jsx
class Base {
  constructor() {
    // SyntaxError
    super()
  }
}
```

### super 참조

**메서드 내에서 super를 참조하면 수퍼클래스의 메서드를 호출할 수 있다.**

1. 서브클래스의 프로토타입 메서드 내에서 super.sayHi 는 수퍼클래스의 프로토타입 메서드 sayHi를 가리킨다.

```jsx
// 수퍼클래스
class Base {
  constructor(name) {
    this.name = name
  }

  sayHi() {
    return `Hi! ${this.name}`
  }
}

// 서브클래스
class Derived extends Base {
  sayHi() {
    // super.sayHi는 수퍼클래스의 프로토타입 메서드를 가리킨다.
    return `${super.sayHi()}. how are you?`
  }
}

const derived = new Derived('Son')
console.log(derived.sayHi()) // Hi! Son. how are you?
```

1. 서브클래스의 정적 메서드 내에서 super.sayHi는 수퍼클래스의 정적 메서드 sayHi를 가리킨다.

```jsx
// 수퍼클래스
class Base {
  static sayHi() {
    return `Hi!`
  }
}

// 서브클래스
class Derived extends Base {
  static sayHi() {
    // super.sayHi는 수퍼클래스의 정적 메서드를 가리킨다.
    return `${super.sayHi()}. how are you?`
  }
}

console.log(Derived.sayHi()) // Hi!. how are you?
```

## 8.6 상속 클래스의 인스턴스 생성 과정

다음은 직사각형을 추상화한 Rectangle 클래스와 상속을 통해 Rectangle 클래스를 확장한 ColorRectangle 클래스를 정의한 것이다.

```jsx
// 수퍼클래스
class Rectangle {
  constructor(width, height) {
    this.width = width
    this.height = height
  }

  getArea() {
    return this.width * this.height
  }

  toString() {
    return `width = ${this.width}, height = ${this.height}`
  }
}

// 서브클래스
class ColorRectangle extends Rectangle {
  constructor(width, height, color) {
    super(width, height)
    this.color = color
  }

  // 메서드 오버라이딩
  toString() {
    return super.toString + `, color = ${this.color}`
  }
}

const colorRectangle = new ColorRectangle(2, 4, 'Blue')
console.log(colorRectangle)
// ColorRectangle { width: 2, height: 4, color: 'Blue' }

// 상속을 통해 getArea 메서드 호출
console.log(colorRectangle.getArea()) // 8
// 오버라이딩된 toString 메서드를 호출
console.log(colorRectangle.toString()) // width = 2, height = 4, color = Blue
```

### 1. 서브클래스의 super 호출

**서브클래스는 자신이 직접 인스턴스를 생성하지 않고 수퍼클래스에게 인스턴스 생성을 위임한다.**

**→ 서브클래스의 constructor에서 반드시 super를 호출해야 하는 이유다.**

### 2. 수퍼클래스의 인스턴스 생성과 this 바인딩

인스턴스는 new.target이 가리키는 서브클래스가 생성한 것으로 처리된다.

### 3. 수퍼클래스의 인스턴스 초기화

### 4. 서브클래스 constructor로의 복귀와 this 바인딩

**super가 반환한 인스턴스가 this에 바인딩된다. 서브클래스는 별도의 인스턴스를 생성하지 않고 super가 반환한 인스턴스를 this에 바인딩하여 그대로 사용한다.**

### 5. 서브클래스의 인스턴스 초기화

### 6. 인스턴스 반환

클래스의 모든 처리가 끝나면 완성된 인스턴스가 바인딩된 this가 암묵적으로 반환된다.

## 8.7 표준 빌트인 생성자 함수 확장

String, Number, Array 같은 표준 빌트인 객체도 Construct 내부 메서드를 갖는 생성자 함수이므로 extends 키워드를 사용하여 확장이 가능하다.

```jsx
// Array 생성자 함수를 상속받아 확장한 MyArray
class MyArray extends Array {
  // 중복된 배열 요소를 제거하고 반환한다.
  uniq() {
    return this.filter((v, i, self) => self.indexOf(v) === i)
  }

  // 모든 배열 요소의 평균을 구한다
  average() {
    return this.reduce((pre, cur) => pre + cur, 0) / this.length
  }
}

const myArray = new MyArray(1, 1, 2, 3)
console.log(myArray) // MyArray(4) [ 1, 1, 2, 3 ]

// MyArray.prototype.uniq 호출
console.log(myArray.uniq()) // MyArray(3) [ 1, 2, 3 ]
// MyArray.prototype.average 호출
console.log(myArray.average()) // 1.75
```

이때 주의할 것은 Array.prototype의 메서드 중에서 map, filter와 같이 새로운 배열을 반환하는 메서드가 MyArray 클래스의 인스턴스를 반환한다.

새로운 배열을 반환하는 메서드가 MyArray 클래스의 인스턴스를 반환하지 않고 Array의 인스턴스를 반환하면 MyArray 클래스의 메서드와 메서드 체이닝이 불가능하다.

```jsx
// 메서드 체이닝
console.log(
  myArray
    .filter(v => v % 2)
    .uniq()
    .average()
) // 2
```

`myArray.filter` 가 반환하는 인스턴스는 MyArray 클래스가 생성한 인스턴스, 즉 MyArray 타입이다.

→ 메서드 체이닝이 가능하다.

만약 MyArray 클래스가 생성한 인스턴스가 아닌 Array가 생성한 인스턴스를 반환하게 하려면 다음과 같이 Symbol.species를 사용하여 정적 접근자 프로퍼티를 추가한다.

```jsx
// Array 생성자 함수를 상속받아 확장한 MyArray
class MyArray extends Array {
  // 모든 메서드가 Array 타입의 인스턴스를 반환하도록 한다.
  static get [Symbol.species]() {
    return Array
  }

  // 중복된 배열 요소를 제거하고 반환한다.
  uniq() {
    return this.filter((v, i, self) => self.indexOf(v) === i)
  }

  // 모든 배열 요소의 평균을 구한다
  average() {
    return this.reduce((pre, cur) => pre + cur, 0) / this.length
  }
}

const myArray = new MyArray(1, 1, 2, 3)
console.log(myArray.uniq() instanceof MyArray) // false
console.log(myArray.uniq() instanceof Array) // true

// 메서드체이닝
// uniq 메서드는 Array 인스턴스를 반환하므로 average 메서드를 호출할 수 없다.
console.log(myArray.uniq().average()) // TypeError
```
