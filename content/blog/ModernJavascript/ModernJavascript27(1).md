---
title: 'Modrn Javascript Deep Dive - 27장 배열(1)'
date: 2023-08-10
category: 'Javascript'
draft: false
---

# 1. 배열이란?

배열(array)은 여러 개의 값을 순차적으로 나열한 자료구조다.

```jsx
const arr = ['apple', 'banana', 'orange']
```

배열이 가지고 있는 값을 요소(element)라고 부른다. 자바스크립트의 모든 값은 배열의 요소가 될 수 있다.

배열의 요소는 배열에서 자신의 위치를 나타내는 0 이상의 정수인 인덱스(index)를 갖는다.

요소에 접근할 때는 대괄호 표기법을 사용한다.

```jsx
arr[0] // apple
arr[1] // banana
arr[2] // orange
```

배열은 요소의 개수, 즉 배열의 길이를 나타내는 length 프로퍼티를 갖는다.

```jsx
arr.length // 3
```

배열은 인덱스와 length 프로퍼티를 갖기 때문에 for 문을 통해 순차적으로 요소에 접근할 수 있다.

```jsx
// 배열의 순회
for (let i = 0; i < arr.length; i++) {
  console.log(arr[i]) // apple, banana, orange
}
```

자바스크립트에 배열이라는 타입은 존재하지 않는다. 배열은 객체 타입이다.

```jsx
typeof arr // object
```

배열은 배열 리터럴, Array 생성자 함수, Array.of, Array.from 메서드로 생성할 수 있다.

```jsx
const arr = [1, 2, 3]

arr.constructor === Array // true
Object.getPrototypeOf(arr) === Array.prototype // true
```

일반 객체와 배열을 구분하는 가장 명확한 차이는 값의 순서와 length 프로퍼티다.

# 2. 자바스크립트 배열은 배열이 아니다.

자료구조에서 말하는 배열은 동일한 크기의 메모리 공간이 빈틈으로 연속적으로 나열된 자료구조를 말한다.

→ 배열의 요소는 하나의 데이터 타입으로 통일되어 있으며 서로 연속적으로 인접해 있다.

→ **밀집 배열(dense array)**

배열에 인덱스를 통해 단 한 번의 연산으로 임의의 요소에 접근(임의 접근, 시간복잡도 O(1))할 수 있다.

정렬되지 않은 배열에서 특정한 요소를 검색하는 경우 배열의 모든 요소를 처음부터 특정 요소를 발견할 때까지 차례대로 검색(선형 검색, 시간복잡도 O(n))해야 한다.

```jsx
function linearSearch(array, target) {
  const length = array.length

  for (let i = 0; i < length; i++) {
    if (array[i] === target) {
      return i
    }
  }
  return -1
}

console.log(linearSearch([1, 2, 3, 4, 5, 6], 3)) // 2
console.log(linearSearch([1, 2, 3, 4, 5, 6], 0)) // -1
```

자바스크립트의 배열은 일반적인 배열이 아니라 각각의 메모리 공간이 동일한 크기를 갖지 않아도 되며, 연속적으로 이어져 있지 않을 수도 있다. → **희소 배열(sparse array)**

**자바스크립트의 배열은 일반적인 배열의 동작을 흉내 낸 특수한 객체다.**

자바스크립트 배열은 인덱스를 나타내는 문자열을 프로퍼티 키로 가지며, length 프로퍼티를 갖는 특수한 객체다.

자바스크립트 배열의 요소는 사실 프로퍼티 값이다. 자바스크립트에서 사용할 수 있는 모든 값은 객체의 프로퍼티 값이 될 수 있으므로 어떤 타입의 값이라도 배열이 요소가 될 수 있다.

```jsx
const arr = [
  'string',
  10,
  true,
  null,
  undefined,
  NaN,
  Infinity,
  [],
  {},
  function() {},
]
```

자바스크립트의 특징은 다음과 같다.

자바스크립트 배열은 해시 테이블로 구현된 객체이므로 인덱스로 요소에 접근하는 경우 일반적인 배열보다 성능적인 면에서 느릴 수밖에 없는 구조적인 단점이 있다.

→ 특정 요소를 검색하거나 요소를 삽입 또는 삭제하는 경우에는 일반적인 배열보다 빠른 성능을 기대할 수 있다.

# 3. length 프로퍼티와 희소 배열

length 프로퍼티는 요소의 개수, 즉 배열의 길이를 나타내는 0 이상의 정수를 값으로 갖는다.

```jsx
;[].length // 0
;[1, 2, 3].length // 3
```

length 프로퍼티의 값은 배열에 요소를 추가하거나 삭제하면 자동 갱신된다.

```jsx
const arr = [1, 2, 3]
console.log(arr.length) // 3

// 요소 추가
arr.push(4)
// 요소를 추가하면 length 프로퍼티의 값이 자동 갱신된다.
console.log(arr.length) // 4

// 요소 삭제
arr.pop()
// 요소를 삭제하면 length 프로퍼티의 값이 자동 갱신된다.
console.log(arr.length)
```

length 프로퍼티의 값은 요소의 개수, 즉 배열의 길이를 바탕으로 결정되지만 임의의 숫자 값을 명시적으로 할당할 수도 있다.

현재 length 프로퍼티 값으로 작은 숫자 값을 할당하면 배열의 길이가 줄어든다.

```jsx
const arr = [1, 2, 3, 4, 5]
arr.length = 3
console.log(arr) // 1, 2, 3
```

현재 length 프로퍼티 값보다 큰 숫자를 할당할 경우 length 프로퍼티 값은 변경되지만 실제로 배열의 길이가 늘어나지는 않는다.

```jsx
const arr = [1]
// 현재 length 프로퍼티 값인 1보다 큰 숫자 3을 length 프로퍼티에 할당
arr.length = 3
console.log(arr.length)
console.log(arr)

// 3
// [ 1, <2 empty items> ]
```

현재 length 프로퍼티 값보다 큰 숫자 값을 length 프로퍼티에 할당하는 경우 length 프로퍼티 값은 성공적으로 변경되지만 실제 배열에는 아무런 변함이 없다. → 메모리 공간을 확보하지는 않음

배열의 요소가 연속적으로 위치하지 않고 일부가 비어 있는 배열을 희소 배열이라 한다.

자바스크립트는 희소 배열을 문법적으로 허용한다.

```jsx
// 희소 배열
const sparse = [, 2, , 4]

// 희소 배열의 length 프로퍼티 값은 요소의 개수와 일치하지 않는다.
console.log(sparse.length)
console.log(sparse)

console.log(Object.getOwnPropertyDescriptors(sparse))

// 4
// [ <1 empty item>, 2, <1 empty item>, 4 ]
// {
//  '1': { value: 2, writable: true, enumerable: true, configurable: true },
//  '3': { value: 4, writable: true, enumerable: true, configurable: true },
//  length: { value: 4, writable: true, enumerable: false, configurable: false }
// }
```

**희소 배열은 length와 배열 요소의 개수가 일치하지 않는다. 희소 배열의 length는 희소 배열의 실제 요소 개수보다 언제나 크다.**

배열에는 생성할 경우에는 희소 배열을 생성하지 않도록 주의해야 한다.

# 4. 배열 생성

## 4.1 배열 리터럴

배열 리터럴은 0개 이상의 요소를 쉼표로 구분하여 대괄호([])로 묶는다.

```jsx
const arr = [1, 2, 3]
console.log(arr.length) // 3

// 배열 리터럴에 요소를 하나도 추가하지 않으면 length 프로퍼티 값이 0인 배열이 된다.
const arr = []
console.log(arr.length) // 0

// 배열 리터럴에 요소를 생략하면 희소 배열이 생성된다.
const arr = [1, , 3] // 희소 배열

// 희소 배열의 length는 배열의 실제 요소 개수보다 언제나 크다.
console.log(arr.length) // 3
console.log(arr) // [ 1, <1 empty item>, 3 ]
console.log(arr[1]) // undefined
```

## 4.2 Array 생성자 함수

Object 생성자 함수를 통해 객체를 생성할 수 있듯이 Array 생성자 함수를 통해 배열을 생성할 수도 있다.

전달된 인수가 1개이고 숫자인 경우 length 프로퍼티 값이 인수인 배열을 생성한다.

```jsx
const arr = new Array(10)

console.log(arr) // [ <10 empty items> ]
console.log(arr.length) // 10
```

배열의 요소를 최대 4,294,967,295개를 가질 수 있다. 전달된 인수가 범위를 벗어나면 RangeError가 발생한다.

```jsx
new Array(4294967295)

// 전달된 인수가 0 ~ 4,294,967,295를 벗어나면 RangeError가 발생한다.
new Array(4294967296) // RangeError

// 전달된 인수가 음수면 에러가 발생한다.
new Array(-1) // RangeError
```

전달된 인수가 없는 경우 빈 배열을 생성한다. 즉, 배열 리터럴[]과 같다.

```jsx
new Array() // []
```

전달된 인수가 2개 이상이거나 숫자가 아닌 경우 인수를 요소로 갖는 배열을 생성한다.

```jsx
// 전달된 인수가 2개 이상이면 인수를 요소로 배열을 생성한다.
new Array(1, 2, 3) // [1, 2, 3]

// 전달된 인수가 1개지만 숫자가 아니면 인수를 요소로 갖는 배열을 생성한다.
new Array({}) // [{}]
```

Array 생성자 함수는 new 연산자와 함께 호출하지 않더라도, 즉 일반 함수로서 호출해도 배열을 생성하는 생성자 함수로 동작한다. 이는 Array 생성자 함수 내부에서 new.target 을 확인하기 때문이다.

```jsx
Array(1, 2, 3) // [1, 2, 3]
```

## 4.3 Array.of

ES6에서 도입된 Array.of 메서드는 전달된 인수를 요소로 갖는 배열을 생성한다. Array.of는 Array 생성자 함수와 다르게 전달된 인수가 1개이고 숫자이더라도 인수를 요소로 갖는 배열을 생성한다.

```jsx
Array.of(1) // [1]

Array.of(1, 2, 3) // [1, 2, 3]

Array.of('string') // ['string']
```

## 4.4 Array.from

ES6에서 도입된 Array.from 메서드는 유사 배열 객체 또는 이터러블 객체를 인수로 전달받아 배열로 변환하여 반환한다.

```jsx
// 유사 배열 객체를 변환하여 배열을 생성한다.
Array.from({
  length: 2,
  0: 'a',
  1: 'b',
})

// 이터러블을 변환하여 배열을 생성한다. 문자열을 이터러블이다.
Array.from('Hello') // ["H", "e", "l", "l", "o"]
```

Array.from을 사용하면 두 번 째 인수로 전달한 콜백 함수를 통해 값을 만들면서 요소를 채울 수 있다.

Array.from 메서드는 두 번째 인수로 전달한 콜백 함수에 첫 번째 인수에 의해 생성된 배열의 요소값과 인덱스를 순차적으로 전달하면서 호출하고, 콜백 함수의 반환값으로 구성된 배열을 반환한다.

```jsx
// Array.from에 length만 존재하는 유사 배열 객체를 전달하면 undefined를 요소로 채운다.
Array.from({ length: 3 })

// Array.from은 두 번째 인수로 전달한 콜백 함수의 반환값으로 구성된 배열을 반환한다.
Array.from({ length: 3 }, (_, i) => i)
```

# 5. 배열 요소의 참조

배열의 요소를 참조할 때에는 대괄호([]) 표기법을 사용한다.

```jsx
const arr = [1, 2]

console.log(arr[0]) // 1
console.log(arr[1]) // 2
```

존재하지 않는 요소에 접근하면 undefined가 반환된다.

```jsx
const arr = [1, 2]

console.log(arr[3]) // undefined
```

# 6. 배열 요소의 추가와 갱신

객체에 프로퍼티를 동적으로 추가할 수 있는 것처럼 배열에도 요소를 동적으로 추가할 수 있다.

존재하지 않는 인덱스를 사용해 값을 할당하면 새로운 요소가 추가된다.

이때 length 프로퍼티 값은 자동 갱신된다.

```jsx
const arr = [0]

// 배열 요소의 추가
arr[1] = 1

console.log(arr) // [0, 1]
console.log(arr.length) // 2
```

만약 현재 배열의 length 프로퍼티 값보다 큰 인덱스로 새로운 요소를 추가하면 희소 배열이 된다.

```jsx
const arr = [0]

arr[1] = 1
arr[3] = 3

console.log(arr) // [ 0, 1, <1 empty item>, 3 ]
console.log(arr.length) // 4
```

이때 인덱스로 요소에 접근하여 명시적으로 값을 할당하지 않은 요소는 생성되지 않는다.

이미 요소가 존재하는 요소에 값을 재할당하면 요소값이 갱신된다.

```jsx
const arr = [0]

// 배열 요소의 추가
arr[1] = 1
arr[1] = 10

console.log(arr) // [0, 10]
console.log(arr.length) // 2
```

인덱스는 요소의 위치를 나타내므로 반드시 0 이상의 정수를 사용해야 한다.

만약 정수 이외의 값을 인덱스처럼 사용하면 요소가 생성되는 것이 아니라 프로퍼티가 생성된다.

→ 추가된 프로퍼티는 length 프로퍼티 값에 영향을 주지 않는다.

```jsx
const arr = []

// 배열 요소의 추가
arr[0] = 1
arr['1'] = 2

// 프로퍼티 추가
arr['foo'] = 3
arr.bar = 4
arr[1.1] = 5
arr[-1] = 6

console.log(arr) // [ 1, 2, foo: 3, bar: 4, '1.1': 5, '-1': 6 ]

// 프로퍼티는 length에 영향을 주지 않는다.
console.log(arr.length) // 2
```

# 7. 배열 요소의 삭제

배열은 사실 객체이기 때문에 배열의 특정 요소를 삭제하기 위해 delete 연산자를 사용할 수 있다.

```jsx
const arr = [1, 2, 3]

// 배열 요소의 삭제
delete arr[1]
console.log(arr) // [1, empty, 3]

// length 프로퍼티에 영향을 주지 않는다. 즉, 희소 배열이 된다.
console.log(arr.length) // 3
```

delete 연산자는 객체의 프로퍼티를 삭제한다. 이때 배열은 희소 배열이 되며 length 프로퍼티 값은 변하지 않는다. → delete 연산자는 사용하지 않는 것이 좋다.

→ Array.prototype.splice 를 사용한다.

```jsx
const arr = [1, 2, 3]

// 배열 요소의 삭제
// Array.prototype.splice(삭제를 시작할 인덱스, 삭제할 요소 수)
arr.splice(1, 1)
console.log(arr) // [1, 3]

// length 프로퍼티에 영향을 주지 않는다. 즉, 희소 배열이 된다.
console.log(arr.length) // 2
```
