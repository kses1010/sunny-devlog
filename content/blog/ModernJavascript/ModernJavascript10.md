---
title: 'Modrn Javascript Deep Dive - 10장 객체 리터럴'
date: 2023-08-01
category: 'Javascript'
draft: false
---

# 1. 객체란?

자바스크립트는 객체(Object) 기반의 프로그래밍 언어이며, 자바스크리브를 구성하는 거의 “모든 것”이 객체다.

원시 값을 제외한 나머지 값(함수, 배열, 정규 표현식 등)은 모두 객체다.

- **원시 타입의 값: 원시 값은 변경 불가능한 값(immutable value)**
- **객체: 변경 가능한 값(mutable value)**

객체는 0개 이상의 프로퍼티로 구성된 집합이며, 프로퍼티는 키(key)와 값(value)으로 구성된다.

```jsx
var person = {
  name: 'Son',
  age: 20,
}

var counter = {
  num: 0, // 프로퍼티
  increase: function() {
    // 메서드
    this.num++
  },
}
```

- 프로퍼티: 객체의 상태를 나타내는 값(data)
- 메서드: 프로퍼티(상태 데이터)를 참조하고 조작할 수 있는 동작(behavior)

# 2. 객체 리터럴에 의한 객체 생성

자바같은 클래스 기반 객체지향 언어는 클래스를 사전에 정의하고 필요한 시점에 `new` 연산자와 함께 생성자를 호출하여 인스턴스를 생성하는 방식으로 객체를 생성한다.

자바스크립트는 프로토타입 기반 객체지향 언어로서 다양한 객체 생성 방법을 지원한다.

- 객체 리터럴
- Object 생성자 함수
- 생성자 함수
- Object.create 메서드
- 클래스(ES6)

객체 리터럴은 중괄호(`{…}`) 내에 0개 이상의 프로퍼티를 정의한다. 변수에 할당되는 시점에 자바스크립트 엔진은 객체 리터럴을 해석해 객체를 생성한다.

```jsx
var person = {
  name: 'Son',
  sayHello: function() {
    console.log(`Hello! My name is ${this.name}.`)
  },
}

console.log(typeof person)
console.log(person)

// object
// { name: 'Son', sayHello: [Function: sayHello] }
```

만약 중괄호 내에 프로퍼티를 정의하지 않으면 빈 객체가 생성된다.

```jsx
var empty = {} // 빈 객체
console.log(typeof empty) // object
```

객체 리터럴의 중괄호는 코드 블록을 의미하지 않는다. 코드 블록의 닫는 중괄호 뒤에는 세미 콜론을 붙이지 않는다. 그러나 객체 리터럴은 값으로 평가되는 표현식이다.

→ 객체 리터럴의 닫는 중괄호 뒤에는 세미콜론을 붙인다.

# 3. 프로퍼티

```jsx
var person = {
  name: 'Son',
  age: 20,
}
```

프로퍼티를 나열할 때는 쉼표(,)로 구분한다. 일반적으로 마지막 프로퍼티 뒤에는 쉼표를 사용하지 않으나 사용해도 무방하다.

프로퍼티 키와 프로퍼티 값으로 사용할 수 있는 값은 다음과 같다.

- key: 빈 문자열을 포함하는 모든 문자열 또는 심벌 값
- value: 자바스크립트에서 사용할 수 있는 모든 값

프로퍼티 키는 프로퍼티 값에 접근할 수 있는 이름으로서 식별자 역할을 한다.

→ 식별자 네이밍 규칙을 준수하는 프로퍼티 키와 그렇지 않은 프로퍼티 키는 미묘한 차이가 있다.

심벌 값도 프로퍼티 키로 사용할 수 있지만 일반적으로 문자열을 사용한다. 이때 프로퍼티 키는 문자열이므로 따옴표(’…’ 또는 “…”)로 묶어야 한다.

하지만 식별자 네이밍 규칙을 준수하는 이름, 즉 자바스크립트에서 사용 가능한 유효한 이름인 경우 따옴표를 생략할 수 있다.

**→ 식별자 네이밍 규칙을 따르지 않는 이름에는 반드시 따옴표를 사용해야 한다.**

```jsx
var person = {
  firstName: 'Sunny', // 식별자 네이밍 규칙 준수
  'last-name': 'Son', // 식별자 네이밍 규칙 미준수
}

console.log(person)

// { firstName: 'Sunny', 'last-name': 'Son' }
```

만약 따옴표를 생략한다면 `last-name` 을 `-` 연산자가 있는 표현식으로 해석되어 SyntaxError가 발생한다.

문자열 또는 문자열로 평가할 수 있는 표현식을 사용해 프로퍼티 키를 동적으로 생성할 수 있다.

이 경우에는 프로퍼티 키로 사용할 표현식을 대괄호([…])로 묶어야 한다.

```jsx
var obj = {}
var key = 'hello'

// ES5: 프로퍼티 키 동적 생성
obj[key] = 'world'
// ES6: 계산된 프로퍼티 이름
// var obj = {
//     [key]: 'world'
// }

console.log(obj)

// { hello: 'world' }
```

빈 문자열을 프로퍼티 키로 사용해도 에러가 발생하지는 않는다. 그러나 키로서의 의미를 갖지 않으므로 권장하지 않는다.

```jsx
var foo = {
  '': '',
}

console.log(foo) // {"":""}
```

프로퍼티 키에 문자열이나 심벌 값 외에 값을 사용하면 암묵적 타입 변환을 통해 문자열이 된다.

```jsx
var foo = {
  0: 1,
  1: 2,
  2: 3,
}

console.log(foo)

// { '0': 1, '1': 2, '2': 3 }
```

var, function과 같은 예약어를 프로퍼티 키로 사용해도 에러가 발생하지 않으나 예상치 못한 에러가 발생할 여지가 있으므로 권장하지 않는다.

```jsx
var foo = {
  var: '',
  fucntion: '',
}

console.log(foo)
```

이미 존재하는 프로퍼티 키를 중복 선언하면 나중에 선언한 프로퍼티가 먼저 선언한 프로퍼티를 덮어쓴다.

이때 에러가 발생하지 않으므로 주의해야한다.

```jsx
var foo = {
  name: 'Son',
  name: 'Hi',
}

console.log(foo)

// { name: 'Hi' }
```

# 4. 메서드

자바스크립트의 함수는 객체(일급 객체)다.

→ 함수는 값으로 취급할 수 있기 때문에 프로퍼티 값으로 사용할 수 있다.

프로퍼티 값이 함수일 경우 일반 함수와 구분하기 위해 메서드라 부른다.

```jsx
var circle = {
  radius: 5,
  // 원의 지름
  getDiameter: function() {
    return 2 * this.radius
  },
}

console.log(circle.getDiameter()) // 10
```

# 5. 프로퍼티 접근

프로퍼티에 접근하는 방법은 다음과 같이 두가지다.

- 마침표 프로퍼티 접근 연산자(.)를 사용하는 마침표 표기법
- 대괄호 프로퍼티 접근 연산자([…])를 사용하는 대괄호 표기법

```jsx
var person = {
  name: 'Son',
}

console.log(person.name) // Son
console.log(person['name']) // Son
```

대괄호 표기법을 사용하는 경우 **대괄호 프로퍼티 접근 연산자 내부에 지정하는 프로퍼티 키는 반드시 따옴표로 감싼 문자열** 이어야 한다.

대괄호 프로퍼티 접근 연산자 내에 따옴표로 감싸지 않은 이름을 프로퍼티 키로 사용하면 자바스크립트 엔진은 식별자로 해석한다.

```jsx
var person = {
  name: 'Son',
}

console.log(person[name]) // ReferenceError
```

ReferenceError 가 발생한 이유는 대괄호 연산자 내의 따옴표로 감싸지 않은 이름, 즉 식별자 `name` 을 평가하기 위해 선언된 `name`을 찾았지만 찾지 못했기 때문이다.

**객체에 존재하지 않는 프로퍼티에 접근하면 undefined 를 반환한다. → ReferenceError 가 발생하지 않음.**

```jsx
var person = {
  name: 'Son',
}

console.log(person.age) // undefined
```

프로퍼티 키가 식별자 네이밍 규칙을 준수하지 않는 이름, 즉 자바스크립트에서 사용 가능한 유효한 이름이 아니면 반드시 대괄호 표기법을 사용해야 한다.

단, 프로퍼티 키가 숫자로 이뤄진 문자열인 경우 따옴표를 생략할 수 있다.

```jsx
var person = {
    'last-name': 'Son',
    1: 10
};

person.'last-name'; // SyntaxError
person.last-name; // 브라우저 환경: NaN // Node.js: ReferenceError

person[last-name]; // ReferenceError
person['last-name']; // Son

// 프로퍼티 키가 숫자로 이뤄진 문자열인 경우 따옴표를 생략할 수 있다.
person.1; // SyntaxError
person.'1'; // SyntaxError
person[1];  // 10
person['1']; // 10
```

# 6. 프로퍼티 값 갱신

존재하는 프로퍼티에 값을 할당하면 프로퍼티 값이 갱신된다.

```jsx
var person = {
  name: 'Son',
}

person.name = 'Sunny'
console.log(person) // { name: 'Sunny' }
```

# 7. 프로퍼티 동적 생성

존재하지 않는 프로퍼티에 값을 할당하면 프로퍼티가 동적으로 생성되어 추가되고 프로퍼티 값이 할당된다.

```jsx
var person = {
  name: 'Son',
}

// person 객체에 age 프로퍼티가 동적으로 생성되고 값이 할당된다.
person.age = 20
console.log(person) // { name: 'Son', age: 20 }
```

# 8. 프로퍼티 삭제

delete 연산자는 개체의 프로퍼티를 삭제한다. 이때 delete 연산자의 피연산자는 프로퍼티 값에 접근할 수 있는 표현식이어야 한다. 만약 존재하지 않는 프로퍼티를 삭제하면 아무런 에러 없이 무시된다.

```jsx
var person = {
  name: 'Son',
}

person.age = 20

// age 프로퍼티 삭제
delete person.age

// delete 연산자로 address 프로퍼티를 삭제할 수 없으나 에러는 발생하지 않는다.
delete person.address

console.log(person) // { name: 'Son' }
```

# 9. ES6에서 추가된 객체 리터럴의 확장 기능

## 9.1 프로퍼티 축약 표현

```jsx
// ES5
var x = 1,
  y = 2

var obj = {
  x: x,
  y: y,
}

console.log(obj) // { x: 1, y: 2 }
```

ES6에서는 프로퍼티 값으로 변수를 사용하는 경우 변수 이름과 프로퍼티 키가 동일한 이름일 때 프로퍼티 키를 생략할 수 있다. 이때 프로퍼티 키는 변수 이름으로 자동 생성된다.

```jsx
let x = 1,
  y = 2

const obj = { x, y }

console.log(obj) // { x: 1, y: 2 }
```

## 9.2 계산된 프로퍼티 이름

문자열 또는 문자열로 타입 변환할 수 있는 값으로 평가되는 표현식을 사용해 프로퍼티 키를 동적을 생성할 수 있다. 단, 프로퍼티 키로 사용할 표현식을 대괄호([…])로 묶어야 한다. → 계산된 프로퍼티 이름이라 한다.

```jsx
// ES5
var prefix = 'prop'
var i = 0

var obj = {}

// 계산된 프로퍼티 이름으로 프로퍼티 키 동적 생성
obj[prefix + '-' + ++i] = i
obj[prefix + '-' + ++i] = i
obj[prefix + '-' + ++i] = i

console.log(obj) // { 'prop-1': 1, 'prop-2': 2, 'prop-3': 3 }
```

ES6에서는 객체 리터럴 내부에서도 계산된 프로퍼티 이름으로 프로퍼티 키를 동적 생성할 수 있다.

```jsx
// ES6
const prefix = 'prop'
let i = 0

// 객체 리터럴 내부에서 계산된 프로퍼티 이름으로 프로퍼티 키를 동적 생성
const obj = {
  [`${prefix}-${++i}`]: i,
  [`${prefix}-${++i}`]: i,
  [`${prefix}-${++i}`]: i,
}

console.log(obj) // { 'prop-1': 1, 'prop-2': 2, 'prop-3': 3 }
```

## 9.3 메서드 축약 표현

```jsx
// ES5
var obj = {
  name: 'Son',
  sayHi: function() {
    console.log('Hi! ' + this.name)
  },
}

obj.sayHi() // Hi! Son
```

ES6에서는 메서드를 정의할 때 function 키워드를 생략한 축약 표현을 사용할 수 있다.

```jsx
// ES6
const obj = {
  name: 'Son',
  sayHi() {
    console.log('Hi! ' + this.name)
  },
}

obj.sayHi() // Hi! Son
```

ES6의 메서드 축약 표현으로 정의한 메서드는 프로퍼티에 할당한 함수와 다르게 동작한다.
