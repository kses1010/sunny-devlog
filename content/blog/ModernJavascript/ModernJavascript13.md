---
title: 'Modrn Javascript Deep Dive - 13장 스코프'
date: 2023-08-02
category: 'Javascript'
draft: false
---

# 1. 스코프란?

스코프(scope)는 자바스크립트를 포함한 모든 프로그래밍 언어의 기본적이며 중요한 개념이다.

자바스크립트의 var 키워드로 선언한 변수와 let 또는 const 키워드로 선언한 변수의 스코프도 다르게 동작한다.

```
function add(x, y) {
    // 매개변수는 함수 몸체 내부에서만 참조할 수 있다.
    // 즉, 매개변수의 스코프(유효범위)는 함수 몸체 내부다.
    console.log(x, y);
    return x + y;
}

add(2, 5);

// 매개변수는 함수 몸체 내부에서만 참조할 수 있다.
console.log(x, y); // ReferenceError
```

변수는 코드의 가장 바깥 영역뿐 아니라 코드 블록이나 함수 몸체 내에서도 선언할 수 있다. 이때 코드 블록이나 함수는 중첩될 수 있다.

```jsx
var var1 = 1 // 코드의 가장 바깥 영역에서 선언한 변수

if (true) {
  var var2 = 2 // 코드 블록 내에서 선언한 변수
  if (true) {
    var var3 = 3 // 중첩된 코드 블록 내에서 선언한 변수
  }
}

function foo() {
  var var4 = 4 // 함수 내에서 선언한 변수

  function bar() {
    var var5 = 5 // 중첩된 함수 내에서 선언한 변수
  }
}

console.log(var1) // 1
console.log(var2) // 2
console.log(var3) // 3
console.log(var4) // ReferenceError
console.log(var5) // ReferenceError
```

모든 식별자는 자신이 선언된 위치에 따라 다른 코드가 식별자 자신을 참조할 수 있는 유효 범위가 결정된다.

**→ 스코프라 한다. 즉, 스코프는 식별자가 유효한 범위를 말한다.**

```jsx
var x = 'global'

function foo() {
  var x = 'local'
  console.log(x)
}

foo()

console.log(x)

// local
// global
```

자바스크립트 엔진은 이름이 같은 두 개의 변수 중에서 어떤 변수를 참조해야 할 것인지를 결정해야 한다.

**→ 식별자 결정**

스코프란 자바스크립트 엔진이 **식별자를 검색할 때 사용하는 규칙**

### var 키워드로 선언한 변수의 중복 선언

```jsx
function foo() {
  var x = 1
  // var 키워드로 선언된 변수는 같은 스코프 내에서 중복 선언을 허용한다.
  // 아래 변수 선언문은 자바스크립트 엔진에 의해 var 키워드가 없는 것처럼 동작한다.
  var x = 2
  console.log(x) // 2
}

foo()

function bar() {
  let x = 1
  // let이나 const 키워드로 선언된 변수는 같은 스코프 내에서 중복 선언을 허용하지 않는다.
  let x = 2 // SyntaxError
}

bar()
```

# 2. 스코프의 종류

코드는 전역과 지역으로 구분할 수 있다.

```jsx
var x = 'global x'
var y = 'global y'

function outer() {
  var z = "outer's local z"

  console.log(x)
  console.log(y)
  console.log(z)

  function inner() {
    var x = "inner's local x"

    console.log(x)
    console.log(y)
    console.log(z)
  }

  inner()
}

outer()

console.log(x)
// console.log(z); -> ReferenceError

// global x
// global y
// outer's local z
// inner's local x
// global y
// outer's local z
// global x
```

## 2.1 전역과 전역 스코프

전역이란 코드의 가장 바깥 영역을 말한다. 전역은 전역 스코프를 만든다.

**전역 변수는 어디서든지 참조할 수 있다.**

## 2.2 지역과 지역 스코프

지역이란 **함수 몸체 내부**를 말한다.

**지역 변수는 자신의 지역 스코프와 하위 스코프에서 유효하다.**

# 3. 스코프 체인

함수 몸체 내부에서 함수가 정의된 것을 ‘함수의 중첩’이라 한다. 그리고 함수 몸체 내부에서 정의한 함수를 중첩 함수, 중첩 함수를 포함하는 함수를 외부 함수라고 한다.

함수는 중첩될 수 있으므로 지역 스코프도 중첩될 수 있다.

**→ 스코프가 함수의 중첩에 의해 계층적 구조를 갖는다.**

모든 스코프는 하나의 계층적 구조로 연결되며, 모든 지역 스코프의 최상위 스코프는 전역 스코프다.

스코프가 계층적 연결된 것을 **스코프 체인** 이라 한다.

변수를 참조할 때 자바스크립트 엔진은 스코프 체인을 통해 변수를 참조하는 코드의 스코프에서 시작하여 상위 스코프 방향으로 이동하며 선언된 변수를 검색 한다.

## 3.1 스코프 체인에 의한 변수

```jsx
var x = 'global x'

function outer() {
  var z = "outer's local z"

  console.log(z)
}

outer()

console.log(x) // global x
// console.log(z); -> ReferenceError
```

**상위 스코프에서 유효한 변수는 하위 스코프에서 자유롭게 참조할 수 있지만 하위 스코프에서 유효한 변수를 상위 스코프에서 참조할 수 없다.**

## 3.2 스코프 체인에 의한 함수 검색

```jsx
// 전역 함수
function foo() {
  console.log('global function foo')
}

function bar() {
  // 중첩 함수
  function foo() {
    console.log('local function foo')
  }
  foo()
}

bar()

// local function foo
```

함수도 식별자에 할당되기 때문에 스코프를 갖는다.

# 4. 함수 레벨 스코프

**코드 블록이 아닌 함수에 의해서 지역 스코프가 생성된다.**

대부분의 프로그래밍 언어는 함수 몸체만이 아니라 모든 코드 블록(if , for, while, try/catch 등)이 지역 스코프를 만든다. 이러한 특성을 **블록 레벨 스코프**라 한다.

**하지만 var 키워드로 선언된 변수는 오로지 함수의 코드 블록만을 지역 스코프로 인정한다.**

→ 함수 레벨 스코프라 한다.

```jsx
var x = 1

if (true) {
  // var 키워드로 선언된 변수는 함수의 코드 블록(함수 몸체)만을 지역 스코프로 인정한다.
  // 함수 밖에서 var 키워드로 선언된 변수는 코드 블록 내에서 선언되었다 할지라도 모두 전역 변수다.
  // 따라서 x는 지역 변수다. 이미 선언된 전역 변수 x가 있으므로 x변수는 중복 선언된다.
  var x = 10
}

console.log(x) // 10
```

var 키워드를 사용할 경우 의도치 않은 전역 변수의 값이 재할당 될 수 있다.

```jsx
var i = 10

for (var i = 0; i < 5; i++) {
  console.log(i) // 0 1 2 3 4
}

console.log(i) // 5
```

# 5. 렉시컬 스코프

```jsx
var x = 1

function foo() {
  var x = 10
  bar()
}

function bar() {
  console.log(x)
}

foo()
bar()

// 1
// 1
```

bar 함수의 상위 스코프가 무엇인지에 따라 결정된다.

1. 함수를 어디서 호출했는지에 따라
2. 함수를 어디서 정의했는지에 따라

첫 번째 방식을 **동적 스코프**라 한다. 함수를 정의하는 시점에서 함수가 어디서 호출될 지 알 수 없다.

→ 함수가 호출되는 시점에 동적으로 상위 스코프를 결정해야 하기 때문에 동적 스코프라 한다.

두번째 방식을 **렉시컬 스코프 또는 정적 스코프** 라 한다. 함수 정의가 평가되는 시점에 상위 스코프가 정적으로 결정되기 때문에 정적 스코프라 한다.

자바스크립트를 비롯한 대부분의 프로그래밍 언어는 렉시컬 스코프를 따른다.

위 예제는 bar 함수는 전역에서 정의된 함수다. 함수 선언문으로 정의된 bar 함수는 전역 코드가 실행되기 전에 먼저 평가되어 함수 객체를 생성한다.

이때 생성된 bar 함수 객체는 자신이 정의된 스코프, 즉 전역 스코프를 기억한다.

그리고 bar 함수가 호출되면 호출된 곳이 어디인지 관계없이 언제나 자신이 기억하고 있는 전역 스코프를 상위 스코프로 사용한다. → 전역 변수 x의 값 1을 두 번 출력한다.
