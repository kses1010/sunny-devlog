---
title: 'Modern Javascript Deep Dive - 33장 7번째 데이터 타입 Symbol'
date: 2023-08-18 21:31:24
category: 'Javascript'
draft: false
---

# 1. 심벌이란?

심벌은 ES6에서 도입된 7번째 데이터 타입으로 변경 불가능한 원시 타입의 값이다.

심벌 값은 다른 값과 중복되지 않는 유일무이한 값이다. 주로 이름의 충돌 위험이 없는 유일한 프로퍼티 키를 만들기 위해 사용한다.

# 2. 심벌 값의 생성

## 2.1 Symbol 함수

심벌 값은 Symbol 함수를 호출하여 생성한다.

**이때 생성한 심벌 값은 외부로 노출되지 않아 확인할 수 없으며, 다른 값과 절대 중복되지 않는 유일무이한 값이다.**

```jsx
// Symbol 함수를 호출하여 유일무이한 심벌 값을 생성한다.
const mySymbol = Symbol();
console.log(typeof mySymbol); // symbol

// 심벌 값은 외부로 노출되지 않아 확인할 수 없다.
console.log(mySymbol); // Symbol()
```

Symbol 함수는 String, Number, Boolean 생성자 함수와 달리 new 연산자와 함께 호출하지 않는다.

```jsx
new Symbol(); // TypeError
```

Symbol 함수에는 선택적으로 문자열을 인수로 전달할 수 있다. 이 문자열은 생성된 심벌 값에 대한 설명으로 디버깅 용도로만 사용되며, 심벌 값 생성에 어떠한 영향을 주지 않는다.

→ 심벌 값에 대한 설명이 같더라도 생성된 심벌 값은 유일무이한 값이다.

```jsx
// 심벌 값에 대한 설명이 같더라도 유일무이한 심벌 값을 생성한다.
const mySymbol1 = Symbol('mySymbol');
const mySymbol2 = Symbol('mySymbol');

console.log(mySymbol1 === mySymbol2); // false
```

심벌 값도 문자열, 숫자, 불리언과 같이 객체처럼 접근하면 암묵적으로 래퍼 객체를 생성한다.

```jsx
const mySymbol = Symbol('mySymbol!!');

// 심벌도 래퍼 객체를 생성한다.
console.log(mySymbol.description); // mySymbol!!
console.log(mySymbol.toString()); // Symbol(mySymbol!!)
```

심벌 값은 암묵적으로 문자열이나 숫자 타입으로 변환되지 않는다.

```jsx
const mySymbol = Symbol();

// 심벌 값은 암묵적으로 문자열이나 숫자 타입으로 변환되지 않는다.
console.log(mySymbol + ''); // TypeError
console.log(+mySymbol); // TypeError
```

단, 불리언 타입으로는 암묵적으로 타입 변환된다. → if 문 등에서 존재 확인이 가능

```jsx
const mySymbol = Symbol();

// 불리언 타입으로는 암묵적으로 타입 변환된다.
console.log(!!mySymbol); // true

// if 문 등에서 존재 확인이 가능하다.
if (mySymbol) {
  console.log('mySymbol is not empty.');
}
// mySymbol is not empty.
```

## 2.2 Symbol.for / Symbol.keyFor 메서드

Symbol.for 메서드는 인수로 전달받은 문자열을 키로 사용하여 키와 심벌 값의 쌍들이 저장되어 있는 전역 심벌 레지스트리에서 해당 키와 일치하는 심벌 값을 검색한다.

- 검색에 성공하면 새로운 심벌 값을 생성하지 않고 검색된 심벌 값을 반환한다.
- 검색에 실패하면 새로운 심벌 값을 생성하여 Symbol.for 메서드의 인수로 전달된 키로 전역 심벌 레지스트리에 저장한 후, 생성된 심벌 값을 반환한다.

```jsx
// 전역 심벌 레지스트리에 mySymbol 이라는 키로 저장된 심벌 값이 없으면 새로운 심벌 값을 생성
const s1 = Symbol.for('mySymbol');
// 전역 심벌 레지스트리에 mySymbol 이라는 키로 저장된 심벌 값이 있으면 해당 심벌 값을 반환
const s2 = Symbol.for('mySymbol');

console.log(s1 === s2); // true
```

Symbol 함수는 호출될 때마다 유일무이한 심벌 값을 생성한다. 이때 자바스크립트 엔진이 관리하는 심벌 값 저장소인 전역 심벌 레지스트리에서 심벌 값을 검색할 수 있는 키를 지정할 수 없으므로 전역 심벌 레지스트리에 등록되어 관리되지 않는다.

Symbol.for 메서드를 사용하면 애플리케이션 전역에서 중복되지 않는 유일무이한 상수인 심벌 값을 단 하나만 생성하여 전역 심벌 레지스트리를 통해 공유할 수 있다.

Symbol.keyFor 메서드를 사용하면 전역 심벌 레지스트리에 저장된 심벌 값의 키를 추출할 수 있다.

```jsx
// 전역 심벌 레지스트리에 mySymbol 이라는 키로 저장된 심벌 값이 없으면 새로운 심벌 값을 생성
const s1 = Symbol.for('mySymbol');
// 전역 심벌 레지스트리에 저장된 심벌 값의 키를 추출
Symbol.keyFor(s1); // mySymbol

// Symbol 함수를 호출하여 생성한 심벌 값은 전역 심벌 레지스트리에 등록되어 관리되지 않는다.
const s2 = Symbol('foo');
Symbol.keyFor(s2); // undefined
```

# 3. 심벌과 상수

4방향, 즉 위, 아래, 왼쪽, 아래쪽을 나타내는 상수를 심벌로 정의한다면

```jsx
const Direction = {
  UP: Symbol('up'),
  DOWN: Symbol('down'),
  LEFT: Symbol('left'),
  RIGHT: Symbol('right'),
};

const myDirection = Direction.UP;

if (myDirection === Direction.UP) {
  console.log('위로 갑니다.'); // 위로 갑니다.
}
```

# 4. 심벌과 프로퍼티 키

객체의 프로퍼티 키는 빈 문자열을 포함하는 모든 문자열 또는 심벌 값으로 만들 수 있으며, 동적으로 생성할 수도 있다.

심벌 값으로 프로퍼티 키를 동적 생성하여 프로퍼티를 만든다면 다음과 같다.

심벌 값을 프로퍼티 키로 사용하려면 프로퍼티 키로 사용할 심벌 값에 대괄호를 사용해야 한다.

프로퍼티에 접근할 때도 마찬가지로 대괄호를 사용해야 한다.

```jsx
const obj = {
  // 심벌 값으로 프로퍼티 키를 생성
  [Symbol.for('mySymbol')]: 1,
};

console.log(obj[Symbol.for('mySymbol')]); // 1
```

**심벌 값은 유일무이한 값이므로 심벌 값으로 프로퍼티 키를 만들면 다른 프로퍼티 키와 절대 충돌하지 않는다.**

# 5. 심벌과 프로퍼티 은닉

심벌 값을 프로퍼티 키로 생성한 프로퍼티는 `for…in` , `Object.keys` , `Object.getOwnPropertyNames` 메서드로 찾을 수 없다.

이처럼 심벌 값을 프로퍼티 키로 사용하여 프로퍼티를 생성하면 외부에 노출할 필요가 없는 프로퍼티를 은닉할 수 있다.

```jsx
const obj = {
  // 심벌 값으로 프로퍼티 키를 생성
  [Symbol('mySymbol')]: 1,
};

for (const key in obj) {
  console.log(key); // 아무것도 출력되지 않는다.
}

console.log(Object.keys(obj)); // []
console.log(Object.getOwnPropertyNames(obj)); // []
```

ES6에서 도입된 Object.getOwnPropertySymbols 메서드를 사용하면 심벌 값을 프로퍼티 키로 사용하여 생성한 프로퍼티를 찾을 수 있다.

```jsx
const obj = {
  // 심벌 값으로 프로퍼티 키를 생성
  [Symbol('mySymbol')]: 1,
};

console.log(Object.getOwnPropertySymbols(obj)); // [Symbol(mySymbol)]

// getOwnPropertySymbols 메서드로 심벌 값도 찾을 수 있다.
const symbolKey = Object.getOwnPropertySymbols(obj)[0];
console.log(obj[symbolKey]); // 1
```

# 6. 심벌과 표준 빌트인 객체 확장

표준 빌트인 객체는 읽기 전용으로 사용하는 것이 좋다.

```jsx
// 표준 빌트인 객체를 확장하는 것은 권장하지 않는다.
Array.prototype.sum = function() {
  return this.reduce((acc, cur) => acc + cur, 0);
}
[1, 2].sum(); // 3
```

그 이유는 개발자가 직접 추가한 메서드와 미래에 표준 사용으로 추가될 메서드의 이름이 중복될 수 있기 때문이다.

하지만 중복될 가능성이 없는 심벌 값으로 프로퍼티 키를 생성하여 표준 빌트인 객체를 확장하면 표준 빌트인 객체의 기존 프로퍼티 키와 충돌하지 않는 것은 물론, 표준 사양의 버전이 올라감에 따라 추가될지 모르는 어떤 프로퍼티 키와도 충돌할 위험이 없어 안전하게 표준 빌트인 객체를 확장할 수 있다.

```jsx
Array.prototype[Symbol.for('sum')] = function() {
  return this.reduce((acc, cur) => acc + cur, 0)
}
[1, 2][Symbol.for('sum')](); // 3
```

# 7. Well-known Symbol

자바스크립트가 기본 제공하는 빌트인 심벌 값이 있다.

브라우저 콘솔에서 Symbol 함수를 참조하면 다음과 같다.

![image](https://github.com/kses1010/sunny-devlog/assets/49144662/de509a12-0b9e-46eb-b9de-08fbff58c562)

자바스크립트가 기본 제공하는 심벌 값을 ECMAScript 사양에서는 **Well-known Symbol** 이라 부른다.

Well-known Symbol 은 자바스크립트 엔진의 내부 알고리즘에 사용된다.

> 심벌은 중복되지 않는 상수 값을 생성하는 것은 물론 기존에 작성된 코드에 영향을 주지 않고 새로운 프로퍼티를 추가하기 위해, 즉 하위 호환성을 보장하기 위해 도입되었다.
