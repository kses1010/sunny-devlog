---
title: 'Modrn Javascript Deep Dive - 35장 스프레드 문법'
date: 2023-08-20
category: 'Javascript'
draft: false
---

ES6에서 도입된 스프레드 문법 `...` 은 하나로 뭉쳐 있는 여러 값들의 집합을 펼쳐서 개별적인 값들의 목록으로 만든다.

스프레드 문법을 사용할 수 있는 대상은 `for...of` 문으로 순회할 수 있는 이터러블에 한정된다.

```jsx
// ...[1, 2, 3]은 [1, 2, 3]을 개별 요소로 분리한다.(-> 1, 2, 3)
console.log(...[1, 2, 3]) // 1 2 3

// 문자열은 이터러블이다.
console.log(...'Hello') // H e l l o

// Map과 Set은 이터러블이다.
console.log(
  ...new Map([
    ['a', '1'],
    ['b', '2'],
  ])
) // [ 'a', '1' ] [ 'b', '2' ]
console.log(...new Set([1, 2, 3])) // 1 2 3

// 이터러블이 아닌 일반 객체는 스프레드 문법의 대상이 될 수 없다.
console.log(...{ a: 1, b: 2 }) // TypeError
```

스프레드 문법의 결과는 값이 아니라 변수에 할당할 수 없다.

```jsx
// 스프레드 문법의 결과는 값이 아니다.
const list = ...[1, 2, 3]; // SyntaxError
```

사용하는 문맥은 다음과 같다.

# 1. 함수 호출문의 인수 목록에서 사용하는 경우

```jsx
const arr = [1, 2, 3]

// 배열 arr의 요소 중에서 최대값을 구하기 위해 Math.max를 사용한다.
const max = Math.max(arr) // NaN
```

Math.max 메서드는 매개변수 개수를 확정할 수 없는 가변 인자 함수다.

Math.max 메서드에 숫자가 아닌 배열을 인수로 전달하면 최대값을 구할 수 없어 NaN을 반환한다.

이 같은 문제를 해결하기 위해 배열을 펼쳐서 요소들을 개별적인 값들의 목록으로 만들고 Math.max 메서드의 인수로 전달해야 한다.

```jsx
var arr = [1, 2, 3]

var max = Math.max.apply(null, arr) // 3

// 스프레드 문법을 사용한다면
const arr = [1, 2, 3]

const max = Math.max(...arr) // 3
```

스프레드 문법은 Rest 파라미터와 형태가 동일하여 혼동할수 있으므로 주의할 필요가 있다.

```jsx
// Rest 파라미터는 인수들의 목록을 배열로 전달 받는다.
function foo(...rest) {
	console.log(rest); // 1, 2, 3 -> [1, 2, 3]
}

// 스프레드 문법은 배열과 같은 이터러블을 펼쳐서 개별적인 값들의 목록을 만든다.
// 1, 2, 3
foo(...[1, 2, 3];
```

# 2. 배열 리터럴 내부에서 사용하는 경우

ES5에서 사용하는 방식과 비교하여 좀 더 간결하고 가독성좋게 표현할 수 있다.

## 2.1 concat

```jsx
// ES5
var arr = [1, 2].concat([3, 4])
console.log(arr) // [1, 2, 3, 4]

// ES6
const arr = [...[1, 2], ...[3, 4]]
console.log(arr) // [1, 2, 3, 4];
```

## 2.2 splice

```jsx
// ES5
var arr1 = [1, 4]
var arr2 = [2, 3]

// 세 번째 인수 arr2를 해체하여 전달해야 한다.
// 그렇지 않으면 arr1에 arr2 배열 자체가 추가된다.
arr1.splice(1, 0, arr2)

// 기대값은 [1, 2, 3, 4]지만
console.log(arr1) // [1, [2, 3], 4];

// 다음처럼 코드가 추가되어야 한다.
Array.prototype.splice.apply(arr1, [1, 0].concat(arr2))
console.log(arr1) // [1, 2, 3, 4]
```

스프레드 문법을 사용하면 더욱 간결하고 가독성 좋게 표현할 수 있다.

```jsx
let arr1 = [1, 4]
let arr2 = [2, 3]

arr1.splice(1, 0, ...arr2)
console.log(arr1) // [1, 2, 3, 4];
```

## 2.3 배열 복사

ES5에선 배열을 복사하려면 slice 메서드를 사용해야 한다.

```jsx
var origin = [1, 2]
var copy = origin.slice()

console.log(copy) // [1, 2]
console.log(origin === copy)
```

스프레드 문법 사용

```jsx
let origin = [1, 2]
let copy = [...origin]

console.log(copy) // [1, 2]
console.log(origin === copy)
```

## 2.4 이터러블을 배열로 변환

ES5에서 이터러블을 배열로 변환하려면 apply 또는 call 메서드를 사용하여 slice 메서드를 호출해야 한다.

```jsx
function sum() {
  // 이터러블이면서 유사 배열 객체인 arguments 를 배열로 변환
  var args = Array.prototype.slice.call(arguments)

  return args.reduce(function(pre, cur) {
    return pre + cur
  }, 0)
}

console.log(sum(1, 2, 3)) // 6
```

이터러블뿐만 아니라 이터러블이 아닌 유사 배열 객체도 배열로 변환할 수 있다.

```jsx
const arrayLike = {
  0: 1,
  1: 2,
  2: 3,
  length: 3,
}

const arr = Array.prototype.slice.call(arrayLike) // [1, 2, 3]
console.log(Array.isArray(arr)) // true
```

스프레드 문법을 사용하면 좀 더 간편하게 이터러블을 배열로 변환할 수 있다.

```jsx
function sum() {
  // 이터러블이면서 유사 배열 객체인 arguments 를 배열로 변환
  return [...arguments].reduce((pre, cur) => pre + cur, 0)
}

console.log(sum(1, 2, 3)) // 6
```

이보다 더 나은 방법은 Rest 파라미터를 사용하는 것이다.

```jsx
const sum = (...args) => args.reduce((pre, cur) => pre + cur, 0)

console.log(sum(1, 2, 3))
```

단, 이터러블이 아닌 유사 배열 객체는 스프레드 문법의 대상이 될 수 없다.

```jsx
const arrayLike = {
  0: 1,
  1: 2,
  2: 3,
  length: 3,
}

const arr = [...arrayLike] // TypeError
```

이터러블이 아닌 유사 배열 객체를 배열로 변경하려면 ES6에서 도입된 Array.from 메서드를 사용한다.

```jsx
// Array.from 은 유사 배열 객체 또는 이터러블을 배열로 변환한다.
Array.from(arrayLike) // [1, 2, 3]
```

# 3. 객체 리터럴 내부에서 사용하는 경우

스프레드 문법의 대상은 이터러블이어야 하지만 스프레드 프로퍼티 제안은 일반 객체를 대상으로도 스프레드 문법의 사용을 허용한다.

```jsx
// 스프레드 프로퍼티
// 객체 복사(얕은 복사)
const obj = { x: 1, y: 2 }
const copy = { ...obj }
console.log(copy) // { x: 1, y: 2 }
console.log(obj === copy) // false

// 객체 병합
const merged = { x: 1, y: 2, ...{ a: 3, b: 4 } }
console.log(merged) // { x: 1, y: 2, a: 3, b: 4 }
```

스프레드 프로퍼티는 Object.assign 메서드를 대체할 수 있는 간편한 문법이다.

```jsx
// 객체 병합, 프로퍼티가 중복되는 경우 뒤에 위치한 프로퍼티가 우선권을 갖는다.
const merged = { ...{ x: 1, y: 2 }, ...{ y: 10, z: 3 } }
console.log(merged) // { x: 1, y: 10, z: 3 }

// 특정 프로퍼티 변경
const changed = { ...{ x: 1, y: 2 }, y: 100 }
console.log(changed) // { x: 1, y: 100 }

// 프로퍼티 추가
const added = { ...{ x: 1, Y: 2 }, z: 0 }
console.log(added) // { x: 1, Y: 2, z: 0 }
```
