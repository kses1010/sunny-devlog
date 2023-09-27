---
title: 'Modern Javascript Deep Dive - 24장 클로저'
date: 2023-08-07 18:27:57
category: 'Javascript'
draft: false
---

클로저는 자바스크립트 고유의 개념이 아니다. 함수를 일급 객체로 취급하는 함수형 프로그래밍 언어(하스켈, 리스프, 얼랭, 스칼라 등)에서 사용되는 중요한 특성이다.

클로저는 자바스크립트 고유의 개념이 아니라 클로저의 정의가 ECMAScript 사양에 등장하지 않는다.

MDN에서는 클로저에 대해 다음과 같이 정의하고 있다.

> 클로저는 함수와 그 함수가 선언된 렉시컬 환경과의 조합이다.

```jsx
const x = 1;

function outerFunc() {
  const x = 10;

  function innerFunc() {
    console.log(x); // 10;
  }

  innerFunc();
}

outerFunc();
```

outerFunc 함수 내부에서 중첩 함수 innerFunc가 정의되고 호출되었다

중첩 함수 innerFunc의 상위 스코프는 외부 함수 outerFunc의 스코프다.

→ 중첩 함수 innerFunc 내부에서 자신을 포함하고 있는 외부 함수 outerFunc의 x 변수에 접근할 수 있다.

```jsx
const x = 1;

function outerFunc() {
  const x = 10;
  innerFunc();
}

function innerFunc() {
  console.log(x); // 1;
}

outerFunc();
```

innerFunc 함수가 outerFunc 함수의 내부에서 정의한 중첩 함수가 아니라면 innerFunc 함수를 outerFunc 함수의 내부에서 호출한다 하더라도 outerFunc 함수의 변수에 접근할 수 없다.

→ 자바스크립트가 렉시컬 스코프를 따르는 프로그래밍 언어이기 때문이다.

# 1. 렉시컬 스코프

**자바스크립트 엔진은 함수를 어디서 호출했는지가 아니라 함수를 어디에 정의했는지에 따라 상위 스코프를 결정한다. → 렉시컬 스코프(정적 스코프)라 한다.**

```jsx
const x = 1;

function foo() {
  const x = 10;
  bar();
}

function bar() {
  console.log(x);
}

foo(); // 1
bar(); // 1
```

foo 함수와 bar 함수의 상위 스코프는 전역이다.

→ 함수의 상위 스코프는 함수를 정의한 위치에 의해 정적으로 결정되고 변하지 않는다.

**렉시컬 환경의 “외부 렉시컬 환경에 대한 참조” 에 저장할 참조값, 즉 상위 스코프에 대한 참조는 함수 정의가 평가되는 시점에서 함수가 정의한 환경(위치)에 따라 결정된다. → 렉시컬 스코프**

# 2. 함수 객체의 내부 슬롯 Environment

**함수는 자신의 내부 슬롯 Environment 에 자신이 정의된 환경, 즉 상위 스코프의 참조를 저장한다.**

함수 객체의 내부 슬롯 Environment 에 저장된 현재 실행 중인 실행 컨텍스트의 렉시컬 환경의 참조가 바로 상위 스코프다. 또한 자신이 호출되었을 때 생성될 함수 렉시컬 환경의 “외부 렉시컬 환경에 대한 참조”에 저장될 참조값이다.

함수 객체는 내부 슬롯 Environment 에 저장한 렉시컬 환경의 참조, 상위 스코프를 자신이 존재하는 한 기억한다.

```jsx
const x = 1;

function foo() {
  const x = 10;

  // 상위 스코프는 함수 정의 환경(위치) 에 따라 결정된다.
  // 함수 호출 위치와 상위 스코프는 아무런 관계가 없다.
  bar();
}

// 함수 bar는 자신의 상위 스코프, 즉 전역 렉시컬 환경을 Environment 에 저장하여 기억한다.
function bar() {
  console.log(x);
}

foo();
bar();
```

# 3. 클로저와 렉시컬 환경

```jsx
const x = 1;

// 1번
function outer() {
  const x = 10;
  // 2번
  const inner = function() {
    console.log(x);
  }
  return inner;
}

// outer 함수를 호출하면 중첩 함수 inner를 반환한다.
// 그리고 outer 함수의 실행 컨텍스트는 실행 컨텍스트 스택에서 팝되어 제거된다.
const innerFunc = outer(); // 3번

// 4번
innerFunc();

// 10
```

- outer 함수(3)를 호출하면 outer 함수는 중첩 함수 inner를 반환하고 생명 주기를 마감한다.
- outer 함수의 지역 변수 x와 변수 값 10을 저장하고 있던 outer 함수의 실행 컨텍스트가 제거되었으므로 outer 함수의 지역 변수 x는 생명주기를 마감한다.
  - outer 함수의 지역 변수 x는 더는 유효하지 않게 되어 x 변수에 접근할 수 없어 보인다.
- 실행 결과(4) outer 함수의 지역 변수 x의 값인 10이다.
  - 생명 주기가 종료되어 실행 컨텍스트 스택에서 제거된 outer 함수의 지역 변수 x가 부활한 것처럼 보인다.

**외부 함수보다 중첩 함수가 더 오래 유지되는 경우 중첩 함수는 이미 생명 주기가 종료한 외부 함수의 변조를 참조 할 수 있다. → 이러한 중첩 함수를 클로저(Closure)라고 한다.**

중첩 함수 inner는 자신의 Environment 내부 슬롯에 현재 실행 중인 실행 컨텍스트의 렉시컬 환경

→ outer 함수의 렉시컬 환경을 상위 스코프로서 저장한다.

outer 함수의 실행이 종료하면 inner 함수를 반환하면서 outer 함수의 생명주기가 종료된다.

**→ outer 함수의 실행 컨텍스트는 실행 컨텍스트 스택에서 제거되지만 outer 함수의 렉시컬 환경까지 소멸하지는 않는다.**

inner 함수는 전역 변수 innerFunc에 의해 참조되고 있으므로 가비지 컬렉션의 대상이 되지 않는다.

자바스크립트의 모든 함수는 상위 스코프를 기억하므로 이론적으로 모든 함수는 클로저다.

하지만 일반적으로 모든 함수를 클로저라고 하지는 않는다.

외부 함수보다 중첩 함수가 더 오래 유지되는 경우 중첩 함수는 이미 생명 주기가 종료한 외부 함수의 변수를 참조할 수 있다.

**클로저는 중첩 함수가 상위 스코프의 식별자를 참조하고 있고 중첩 함수가 외부 함수보다 더 오래 유지되는 경우에 한정한다.**

# 4. 클로저의 활용

**클로저는 상태를 안전하게 변경하고 유지하기 위해 사용한다.**

**상태를 안전하게 은닉하고 특정 함수에게만 상태 변경을 허용하기 위해 사용한다.**

```jsx
// 카운트 상태 변수
let num = 0;

// 카운트 상태 변경 함수
const increase = function() {
  // 카운트 상태를 1만큼 증가시킨다.
  return ++num;
}

console.log(increase()); // 1
console.log(increase()); // 2
console.log(increase()); // 3
```

코드는 잘 동작하지만 오류를 발생시킬만한 코드다. 다음 조건을 지켜야 한다.

- 카운트 상태(num)는 increase 함수가 호출되기 전까지 변경되지 않고 유지되어야 한다.
- 이를 위해 카운트 상태(num)는 increase 함수만이 변경할 수 있어야 한다.

그러나 카운트 상태는 전역 변수를 통해 관리되고 있기 때문에 언제든지 누구나 접근할 수 있고 변경이 가능하다.

increase만이 num 변수를 참조하도록 변경해보자.

```jsx
// 카운트 상태 변경 함수
const increase = function() {
  // 카운트 상태 변수
  let num = 0;

  // 카운트 상태를 1만큼 증가시킨다.
  return ++num;
}

console.log(increase()); // 1
console.log(increase()); // 1
console.log(increase()); // 1
```

이 경우 increase 함수가 호출될 때마다 지역 변수 num이 다시 선언되어 0으로 초기화된다.

클로저를 사용한다면

```jsx
// 카운트 상태 변경 함수
const increase = (function() {
  // 카운트 상태 변수
  let num = 0;

  // 클로저
  return function() {
    // 카운트 상태를 1만큼 증가시킨다.
    return ++num;
  }
})();

console.log(increase()); // 1
console.log(increase()); // 2
console.log(increase()); // 3
```

클로저는 상태가 의도치 않게 변경되지 않도록 안전하게 은닉하고 특정 함수에게만 상태 변경을 허용하여 상태를 안전하게 변경하고 유지하기 위해 사용한다.

```jsx
// 카운트 상태 변경 함수
const counter = (function() {
  // 카운트 상태 변수
  let num = 0;

  // 클로저
  // 객체 리터럴은 스코프를 만들지 않는다.
  // 따라서 아래 메서드들의 상위 스코프는 즉시 실행 함수의 렉시컬 환경이다.
  return {
    increase() {
      // 카운트 상태를 1만큼 증가시킨다.
      return ++num;
    },
    decrease() {
      // 카운트 상태를 1만큼 감소시킨다.
      return --num;
    },
  }
})()

console.log(counter.increase()); // 1
console.log(counter.increase()); // 2
console.log(counter.decrease()); // 1
console.log(counter.decrease()); // 0
```

즉시 실행 함수가 반환하는 객체 리터럴은 즉시 실행 함수의 실행 단계에서 평가되어 객체가 된다.

객체 리터럴의 중괄호는 코드 블록이 아니므로 별도의 스코프를 생성하지 않는다.

생성자 함수로 표현하면 다음과 같다.

```jsx
const Counter = (function() {
  // 카운트 상태 변수
  let num = 0;

  function Counter() {
    // this.num = 0; // 프로퍼티는 public하므로 은닉하지 않는다.
  }

  Counter.prototype.increase = function() {
    return ++num;
  }

  Counter.prototype.decrease = function() {
    return --num;
  }

  return Counter;
})();

const counter = new Counter();

console.log(counter.increase()); // 1
console.log(counter.increase()); // 2
console.log(counter.decrease()); // 1
console.log(counter.decrease()); // 0
```

num 변수의 값은 increase, decrease 메서드 만이 변경할 수 있다.

상태 변경이나 가변 데이터를 피하고 불변성을 지향하는 함수형 프로그래밍에서 부수 효과를 최대한 억제하여 오류를 피하고 프로그램의 안정성을 높이기 위해 클로저는 적극적으로 사용된다.

```jsx
// 함수를 인수로 전달받고 함수를 반환하는 고차 함수
// 이 함수는 카운트 상태를 유지하기 위한 자유 변수 counter를 기억하는 클로저를 반환한다.
function makeCounter(predicate) {
  // 카운트 상태를 유지하기 위한 자유 변수
  let counter = 0

  // 클로저를 반환
  return function() {
    // 인수로 전달받은 보조 함수에 상태 변경을 위임한다.
    counter = predicate(counter)
    return counter
  }
}

// 보조 함수
function increase(n) {
  return ++n
}

function decrease(n) {
  return --n
}

// 함수로 함수를 생성한다.
// makeCounter 함수는 보조 함수를 인수로 전달 받아 함수를 반환한다.
const increaser = makeCounter(increase)
console.log(increaser()) // 1
console.log(increaser()) // 2
console.log(increaser) // [Function (anonymous)]

const decreaser = makeCounter(decrease)
console.log(decreaser()) // -1
console.log(decreaser()) // -2
console.log(decreaser) // [Function (anonymous)]
```

makeCounter 함수는 보조 함수를 인자로 전달받고 함수를 반환하는 고차 함수다.

makeCounter 함수는 인자로 전달받은 보조 함수를 합성하여 자신이 반환하는 함수의 동작을 변경할 수 있다.

**이때 makeCounter 함수를 호출해 함수를 반환할 때 반환한 함수는 자신만의 독립된 렉시컬 환경을 갖는다.**

전역 변수 increaser와 decreaser에 할당된 함수는 각각 자신만의 독립된 렉시컬 환경을 갖기 대문에 카운트를 유지하기 위한 자유 변수 counter를 공유하지 않아 카운터의 증감이 연동되지 않는다.

→ 증감이 가능한 카운터를 만들려면 렉시컬 환경을 공유하는 클로저를 만들어야 한다.

```jsx
// 함수를 인수로 전달받고 함수를 반환하는 고차 함수
// 이 함수는 카운트 상태를 유지하기 위한 자유 변수 counter를 기억하는 클로저를 반환한다.
const counter = (function() {
  // 카운트 상태를 유지하기 위한 자유 변수
  let counter = 0;

  // 클로저를 반환
  return function(predicate) {
    // 인수로 전달받은 보조 함수에 상태 변경을 위임한다.
    counter = predicate(counter);
    return counter;
  }
})()

// 보조 함수
function increase(n) {
  return ++n;
}

function decrease(n) {
  return --n;
}

// 보조 함수를 전달하여 호출
console.log(counter(increase)); // 1
console.log(counter(increase)); // 2

console.log(counter(decrease)); // 1
console.log(counter(decrease)); // 0
```

# 5. 캡슐화와 정보 은닉

캡슐화는 객체의 상태를 나타내는 프로퍼티와 프로퍼티를 참조하고 조작할 수 있는 동작인 메서드를 하나로 묶는 것을 말한다.

캡슐화는 객체의 특정 프로퍼티나 메서드를 감출 목적으로 사용하기도 하는데 이를 정보 은닉이라 한다.

정보 은닉은 객체의 상태가 변경되는 것을 방지해 정보를 보호하고, 객체 간의 상호 의존성, 즉 결합도(coupling)를 낮추는 효과가 있다.

자바스크립트는 public, private, protected 같은 접근 제한자를 제공하지 않는다.

→ 자바스크립트 객체의 모든 프로퍼티와 메서드는 기본적으로 외부에 공개되어 있다.

```jsx
function Person(name, age) {
  this.name = name; // public
  let _age = age; // private

  // 인스턴스 메서드
  this.sayHi = function() {
    console.log(`Hi! My name is ${this.name}. I am ${_age}.`);
  }
}

const me = new Person('Son', 20);
me.sayHi() // Hi! My name is Son. I am 20.
console.log(me.name); // name
console.log(me._age); // undefined
```

name 프로퍼티는 public이다. \_age 변수는 Person 생성자 함수의 지역 변수이므로 Person 생성자 함수 외부에서 참조하거나 변경할 수 없다.

위 예제의 sayHi 메서드는 인스턴스 메서드이므로 Person 객체가 생성될 때마다 중복 생성된다.

중복 생성을 방지한다면

```jsx
function Person(name, age) {
  this.name = name; // public
  let _age = age; // private
}

// 프로토타입 메서드
// Person 생성자 함수의 지역 변수 _age를 참조할 수 없다.
Person.prototype.sayHi = function() {
  console.log(`Hi! My name is ${this.name}. I am ${_age}.`);
}
```

즉시 실행 함수를 사용하여 하나의 함수 내에 모은다면

```jsx
const Person = (function() {
  let _age = 0; // private

  // 생성자 함수
  function Person(name, age) {
    this.name = name; // public
    _age = age; // private
  }

  // 프로토타입 메서드
  Person.prototype.sayHi = function() {
    console.log(`Hi! My name is ${this.name}. I am ${_age}.`);
  }

  // 생성자 함수를 반환
  return Person;
})();

const me = new Person('Son', 20);
me.sayHi() // Hi! My name is Son. I am 20.
console.log(me.name); // name
console.log(me._age); // undefined
```

즉시 실행 함수가 반환하는 Person 생성자 함수와 Person 생성자 함수의 인스턴스가 상속받아 호출할 Person.prototype.sayHi 메서드는 즉시 실행 함수가 종료된 이후 호출된다.

`_age`를 참조 할 수 있는 클로저다.

하지만 이 코드도 문제가 있다. Person 생성자 함수가 여러 개의 인스턴스를 생성할 경우 다음과 같이 `_age` 변수의 상태가 유지되지 않는다.

```jsx
const me = new Person('Son', 20);
me.sayHi() // Hi! My name is Son. I am 20.

const you = new Person('Sunny', 30);
you.sayHi() // Hi! My name is Sunny. I am 30.

// _age 값이 변경되었다.
me.sayHi(); // Hi! My name is Son. I am 30.
```

Person.prototype.sayHi 메서드가 단 한 번 생성되는 클로저이기 때문에 발생하는 현상이다.

# 6. 자주 발생하는 실수

```jsx
var funcs = [];

for (var i = 0; i < 3; i++) {
  funcs[i] = function() {
    return i;
  }
}

for (var j = 0; j < funcs.length; j++) {
  console.log(funcs[j]());
}
// 3 3 3
```

funcs 배열의 요소로 추가된 3개의 함수가 0, 1, 2를 반환하기를 바라지만 그렇진 않다.

for 문의 변수 선언문에서 var 키워드로 선언한 i 변수는 블록 레벨 스코프가 아닌 함수 레벨 스코프를 갖기 때문에 전역 변수다. 전역 변수 i에는 0, 1, 2가 순차적으로 할당된다.

→ funcs 배열의 요소로 추가한 함수를 호출하면 전역 변수 i를 참조하여 i의 값 3이 출력된다.

클로저를 사용한다면

```jsx
var funcs = [];

for (var i = 0; i < 3; i++) {
  funcs[i] = (function(id) {
    return function() {
      return id;
    }
  })(i);
}

for (var j = 0; j < funcs.length; j++) {
  console.log(funcs[j]());
}
// 0 1 2
```

자바스크립트의 함수 레벨 스코프 특성으로 인해 for 문의 변수 선언문에서 var 키워드로 선언한 변수가 전역 변수가 되기 때문에 발생하는 현상이다.

ES6의 let 키워드를 사용하면 해결된다.

```jsx
var funcs = [];

for (let i = 0; i < 3; i++) {
  funcs[i] = function() {
    return i;
  }
}

for (var j = 0; j < funcs.length; j++) {
  console.log(funcs[j]());
}

// 0 1 2
```

또 다른 방법으로 함수형 프로그래밍 기법인 고차 함수를 사용하는 방법이 있다.

```jsx
// 요소가 3개인 배열을 생성하고 배열의 인덱스를 반환하는 함수를 요소로 추가한다.
// 배열의 요소로 추가한 함수들은 모두 클로저다.
const func = Array.from(new Array(3), (_, i) => () => i)
console.log(func)
// [
// 	[Function (anonymous)],
// 	[Function (anonymous)],
// 	[Function (anonymous)]
// ]

// 배열의 요소로 추가된 함수들을 순차적으로 호출한다.
func.forEach(f => console.log(f())) // 0 1 2
```
