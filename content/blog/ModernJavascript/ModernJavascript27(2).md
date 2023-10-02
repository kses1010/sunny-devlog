---
title: 'Modern Javascript Deep Dive - 27장 배열(2)'
date: 2023-08-11 12:00:00
category: 'Javascript'
draft: false
---

# 8. 배열 메서드

자바스크립트는 배열을 다룰 때 유용한 다양한 빌트인 메서드를 제공한다. Array 생성자 함수는 정적 메서드를 제공하며, 배열 객체의 프로토타입인 Array.prototype은 프로토타입 메서드를 제공한다.

**배열에는 원본 배열(배열 메서드를 호출한 배열, 즉 배열 메서드의 구현체 내부에서 this가 가리키는 객체)을 직접 변경하는 메서드와 원본 배열을 직접 변경하지 않고 새로운 배열을 생성하여 변환하는 메서드가 있다.**

→ 가급적 원본 배열을 직접 변경하지 않는 메서드를 사용하는 것이 부수 효과를 줄일 수 있다.

## 8.1 Array.isArray

Array.isArray는 Array 생성자 함수의 정적 메서드다.

Array.isArray 메서드는 전달된 인수가 배열이면 true, 배열이 아니면 false를 반환한다.

```jsx
// true
Array.isArray([]);
Array.isArray([1, 2]);
Array.isArray(new Array());

// 이외에는 전부 false
```

## 8.2 Array.prototype.indexOf

indexO 메서드는 원본 배열에서 인수로 전달된 요소를 검색하여 인덱스를 반환한다.

```jsx
const arr = [1, 2, 2, 4];
// 배열 arr에서 요소 2를 검색하여 첫 번째로 검색된 요소의 인덱스를 반환한다.
arr.indexOf(2); // 1
// 배열 arr에 요소 4가 없으므로 -1을 반환한다.
arr.indexOf(4); // -1
// 두 번째 인수는 검색을 시작할 인덱스다. 두 번째 인수를 생략하면 처음부터 검색한다.
arr.indexOf(2, 2); // 2
```

indexOf 메서드는 배열에 특정 요소가 존재하는지 확인할 때 유용하다.

```jsx
const foods = ['apple', 'banana', 'orange'];

// foods 배열에 'orange' 요소가 존재하는지 확인한다.
if (foods.indexOf('orange') === -1) {
  // foods 배열에 'orange' 요소가 존재하지 않으면 'orange' 요소를 추가한다.
  foods.push('orange');
}

console.log(foods); // [ 'apple', 'banana', 'orange' ]
```

ES7에서 도입된 Array.prototype.includes 메서드를 사용하면 가독성이 더 좋다.

```jsx
const foods = ['apple', 'banana', 'orange'];

// foods 배열에 'orange' 요소가 존재하는지 확인한다.
if (foods.includes('orange')) {
  // foods 배열에 'orange' 요소가 존재하지 않으면 'orange' 요소를 추가한다.
  foods.push('orange');
}

console.log(foods); // [ 'apple', 'banana', 'orange' ]
```

## 8.3 Array.prototype.push

push 메서드는 인수로 전달받은 모든 값을 원본 배열의 마지막 요소로 추가하고 변경된 length 프로퍼티 값을 반환한다. → push 메서드는 원본 배열을 직접 변경한다.

```jsx
const arr = [1, 2];

// 인수로 전달받은 모든 값을 원본 배열 arr의 마지막 요소로 추가하고 변경된 length 값을 반환한다.
let result = arr.push(3, 4);
console.log(result);

// push 메서드는 원본 배열을 직접 변경한다.
console.log(arr); // [1, 2, 3, 4]
```

push 메서드는 성능 면에서 좋지 않다. 마지막 요소로 추가할 요소가 하나뿐이라면 push 메서드를 사용하지 않고 length 프로퍼티를 사용하여 배열의 마지막에 요소를 직접 추가할 수 있다.

→ push 메서드보다 빠르다.

```jsx
const arr = [1, 2];

arr[arr.length] = 3;
console.log(arr); // [1, 2, 3]
```

push 메서드는 원본 배열을 직접 변경하는 부수 효과가 있다.

→ push 메서드보다는 ES6의 스프레드 문법을 사용하는 편이 좋다.

스프레드 문법을 사용하면 함수 호출 없이 표현식으로 마지막에 요소를 추가할 수 있으며 부수 효과도 없다.

```jsx
const arr = [1, 2];

// ES6 스프레드 문법
const newArr = [...arr, 3];
console.log(newArr); // [1, 2, 3]
```

## 8.4 Array.prototype.pop

pop 메서드는 원본 배열에서 마지막 요소를 제거하고 제거한 요소를 반환한다. 원본 배열이 빈 배열이면 undefined를 반환한다. → pop 메서드는 원본 배열을 직접 변경한다.

```jsx
const arr = [1, 2];

// 원본 배열에서 마지막 요소를 제거하고 제거한 요소를 반환한다.
let result = arr.pop();
console.log(result); // 2

// pop 메서드는 원본 배열을 직접 변경한다.
console.log(arr); // [1]
```

## 8.5 Array.prototype.unshift

unshift 메서드는 인수로 전달받은 모든 값을 원본 배열의 선두에 요소를 추가하고 변경된 length 프로퍼티 값을 반환한다. → unshift 메서드는 원본 배열을 직접 변경한다.

```jsx
const arr = [1, 2];

// 인수로 전달받은 모든 값을 원본 배열의 선두에 요소를 추가하고 변경된 length 값을 반환한다.
let result = arr.unshift(3, 4);
console.log(result); // 4

// unshift 메서드는 원본 배열을 직접 변경한다.
console.log(arr); // [ 3, 4, 1, 2 ]
```

ES6의 스프레드 문법

```jsx
const arr = [1, 2];

// ES6 스프레드 문법
const newArr = [3, ...arr];
console.log(newArr); // [3, 1, 2]
```

## 8.6 Array.prototype.shift

shift 메서드는 원본 배열에서 첫 번째 요소를 제거하고 제거한 요소를 반환한다. 원본 배열이 빈 배열이면 undefined를 반환한다. → shift 메서드는 원본 배열을 직접 변경한다.

```jsx
const arr = [1, 2];

// 원본 배열에서 첫 번째 요소를 제거하고 제거한 요소를 반환한다.
let result = arr.shift();
console.log(result); // 1

// shift 메서드는 원본 배열을 직접 변경한다.
console.log(arr); // [2]
```

## 8.7 Array.prototype.concat

concat 메서드는 인수로 전달된 값들을 원본 배열의 마지막 요소로 추가한 새로운 배열을 반환한다.

인수로 전달한 값이 배열인 경우 배열을 해체하여 새로운 배열의 요소로 추가한다.

→ 원본 배열은 변경되지 않는다.

```jsx
const arr1 = [1, 2];
const arr2 = [3, 4];

// 배열 arr2를 원본 배열 arr1의 마지막 요소로 추가한 새로운 배열을 반환한다.
// 인수로 전달한 값이 배열인 경우 배열을 해체하여 새로운 배열의 요소로 추가한다.
let result = arr1.concat(arr2);
console.log(result); // [1, 2, 3, 4]

// 숫자를 원본 배열 arr1의 마지막 요소로 추가한 새로운 배열을 반환한다.
result = arr1.concat(3);
console.log(result); // [1, 2, 3]

// 배열 arr2와 숫자를 원본 배열 arr1의 마지막 요소로 추가한 새로운 배열을 반환한다.
result = arr1.concat(arr2, 5);
console.log(result); // [1, 2, 3, 4, 5]

// 원본 배열은 변경되지 않는다.
console.log(arr1); // [1, 2]
```

push와 unshift 메서드는 concat 메서드로 대체할 수 있다.

```jsx
const arr1 = [3, 4];

arr1.unshift(1, 2);
console.log(arr1); // [1, 2, 3, 4]

arr1.push(5, 6);
console.log(arr1); // [1, 2, 3, 4, 5, 6]

// concat 메서드로 대체하기
const arr2 = [3, 4];

let result = [1, 2].concat(arr2);
console.log(arr2); // [1, 2, 3, 4]

result = result.concat(5, 6);
console.log(arr2); // [1, 2, 3, 4, 5, 6]
```

인수로 전달받은 값이 배열인 경우 push와 unshift 메서드는 배열을 그대로 원본 배열의 마지막/첫 번째 요소로 추가한다.

concat 메서드는 인수로 전달받은 배열을 해체하여 새로운 배열의 마지막 요소로 추가한다.

```jsx
const arr1 = [3, 4];

arr1.unshift([1, 2]);
arr1.push([5, 6]);
console.log(arr1); // [ [ 1, 2 ], 3, 4, [ 5, 6 ] ]

// concat 메서드로 대체하기
const arr2 = [3, 4];

let result = [1, 2].concat([3, 4]);
result = result.concat([5, 6]);
console.log(arr2); // [1, 2, 3, 4, 5, 6]
```

concat 메서드는 ES6의 스프레드 문법으로 대체할 수 있다.

```jsx
let result = [...[1, 2], ...[3, 4]];
console.log(result); // [1, 2, 3, 4]
```

## 8.8 Array.prototype.splice

splice 메서드는 3개의 매개변수가 있으며 원본 배열을 직접 변경한다.

- start: 원본 배열의 요소를 제거하기 시작할 인덱스다. start가 음수인 경우 배열의 끝에서의 인덱스를 나타낸다. 만약 start가 -1이면 마지막 요소를 가리키고 -n이면 마지막에서 n번째 요소를 가리킨다.
- deleteCount: 원본 배열의 요소를 제거하기 시작할 인덱스인 start부터 제거할 요소의 개수다. deleteCount가 0인 경우 아무런 요소도 제거되지 않는다.
- items: 제거한 위치에 삽입할 요소들의 목록이다. 생략할 경우 원본 배열에서 요소들을 제거하기만 한다.

```jsx
const arr = [1, 2, 3, 4];

// 원본 배열의 인덱스 1부터 2개의 요소를 제거하고 그 자리에 새로운 요소 20, 30을 삽입한다.
const result = arr.splice(1, 2, 20, 30);

// 제거한 요소가 배열로 반환된다.
console.log(result); // [2, 3]
// splice 메서드는 원본 배열을 직접 변경한다.
console.log(arr); // [ 1, 20, 30, 4 ]
```

splice 메서드의 두 번째 인수, 제거할 요소의 개수를 0으로 지정하면 아무런 요소도 제거하지 않고 새로운 요소들을 삽입한다.

```jsx
const arr = [1, 2, 3, 4];

const result = arr.splice(1, 0, 100);

console.log(arr); // [1, 100, 2, 3, 4]
// 제거한 요소가 배열로 반환된다.
console.log(result); // []
```

splice 메서드의 세 번째 인수, 즉 제거한 위치에 추가할 요소들의 목록을 전달하지 않으면 원본 배열에서 지정된 요소를 제거하기만 한다.

```jsx
const arr = [1, 2, 3, 4];

const result = arr.splice(1, 2);

console.log(arr); // [1, 4]
// 제거한 요소가 배열로 반환된다.
console.log(result); // [2, 3]
```

splice 메서드의 두 번째 인수, 제거할 요소의 개수를 생력하면 첫 번째 인수로 전달된 시작 인덱스부터 모든 요소를 제거한다.

```jsx
const arr = [1, 2, 3, 4];

const result = arr.splice(1);

console.log(arr); // [1]
// 제거한 요소가 배열로 반환된다.
console.log(result); // [2, 3, 4]
```

배열에서 특정 요소를 제거하려면 indexOf 메서드를 통해 특정 요소의 메서드를 취득한 다음 splice 메서드를 사용한다.

```jsx
const arr = [1, 2, 3, 1, 2];

function remove(array, item) {
  const index = array.indexOf(item);

  if (index !== -1) {
    array.splice(index, 1);
  }
  return array;
}

console.log(remove(arr, 2)); // [ 1, 3, 1, 2 ]
console.log(remove(arr, 10)); // [ 1, 3, 1, 2 ]
```

filter 메서드를 사용해서 특정 요소를 제거할 수도 있다. 하지만 특정 요소가 중복된 경우 모두 제거된다.

```jsx
const arr = [1, 2, 3, 1, 2];

function removeAll(array, item) {
  return array.filter(v => v !== item);
}

console.log(removeAll(arr, 2)); // [1, 3, 1]
```

## 8.9 Array.prototype.slice

slice 메서드는 인수로 전달된 범위의 요소들을 복사하여 배열로 반환한다.

- start: 복사를 시작할 인덱스다. 음수인 경우 배열의 끝에서의 인덱스를 나타낸다.
- end: 복사를 종료할 인덱스다. 이 인덱스에 해당하는 요소는 복사되지 않는다. end는 생략 가능하며 생략 시 기본값은 length 프로퍼티 값이다.

```jsx
const arr = [1, 2, 3];

// arr[0] 부터 arr[1] 이전(arr[1] 미포함)까지 복사하여 반환한다.
arr.slice(0, 1); // [1]

// arr[1]부터 arr[2] 이전(arr[2] 미포함)까지 복사하여 반환한다.
arr.slice(1, 2); // [2]

// 원본은 변경되지 않는다.
console.log(arr); // [1, 2, 3]
```

slice 메서드의 두 번째 인수를 생략하면 첫 번째 인수로 전달받은 인덱스부터 모든 요소를 복사하여 배열로 반환한다.

```jsx
const arr = [1, 2, 3];

// arr[1]부터 전부 복사하여 반환한다.
arr.slice(1); // [2, 3]
```

slice 메서드의 첫 번째 인수가 음수인 경우 배열의 끝에서부터 요소를 복사하여 배열로 반환한다.

```jsx
const arr = [1, 2, 3];

// 배열의 끝에서부터 요소를 한 개 복사하여 반환한다.
arr.slice(-1); // [3]

// 배열의 끝에서부터 요소를 두 개 복사하여 반환한다.
arr.slice(-2); // [2, 3]
```

slice 메서드의 인수를 모두 생략하면 원본 배열의 복사본을 생성하여 반환한다.

```jsx
const arr = [1, 2, 3];

// 인수를 모두 생략하면 원본 배열의 복사본을 생성하여 반환한다.
const copy = arr.slice();
console.log(copy); // [1, 2, 3]
console.log(copy === arr); // false
```

slice 메서드가 복사본을 생성하는 것을 이용하여 유사 배열 객체를 배열로 변환할 수 있다.

```jsx
function sum() {
  // 유사 배열 객체를 배열로 변환(ES5)
  var arr = Array.prototype.slice.call(arguments);
  console.log(arr); // [1, 2, 3]

  return arr.reduce(function(pre, cur) {
    return pre + cur;
  }, 0)
}

console.log(sum(1, 2, 3)); // 6
```

Array.from 메서드를 사용하면 더욱 간단하게 유사 배열 객체를 배열로 변환할 수 있다.

```jsx
function sum() {
  const arr = Array.from(arguments);
  console.log(arr); // [1, 2, 3]

  return arr.reduce((pre, cur) => pre + cur, 0);
}

console.log(sum(1, 2, 3)); // 6
console.log(sum()); // 0
```

ES6 스프레드 문법을 사용해서 간단하게 배열로 변환할 수 있다.

```jsx
function sum() {
  const arr = [...arguments];
  console.log(arr); // [1, 2, 3]

  return arr.reduce((pre, cur) => pre + cur, 0);
}

console.log(sum(1, 2, 3)); // 6
```

## 8.10 Array.prototype.join

join 메서드는 원본 배열의 모든 요소를 문자열로 변환한 후, 인수로 전달받은 문자열, 즉 구분자로 연결한 문자열을 반환한다. 구분자는 생략 가능하며 기본 구분자는 콤마(,)다.

```jsx
const arr = [1, 2, 3];

arr.join(); // '1,2,3,4'
arr.join(''); // '1234'
arr.join(':'); // '1:2:3:4'
```

## 8.11 Array.prototype.reverse

reverse 메서드는 원본 배열의 순서를 반대로 뒤집는다. 이때 원본 배열이 변경된다. 변경값은 변경된 배열이다.

```jsx
const arr = [1, 2, 3, 4];
const result = arr.reverse();

// reverse 메서드는 원본 배열을 직접 변경한다.
console.log(arr); // [4, 3, 2, 1]
console.log(result); // [4, 3, 2, 1]
```

## 8.12 Array.prototype.fill

ES6에서 도입된 fill 메서드는 인수로 전달 받은 값을 배열의 처음부터 끝까지 요소로 채운다. 이때 원본 배열이 변경된다.

```jsx
const arr = [1, 2, 3, 4];

// 인수로 전달받은 값 0을 배열의 처음부터 끝까지 요소로 채운다.
arr.fill(0);

// fill 메서드는 원본 배열을 직접 변경한다.
console.log(arr); // [0, 0, 0]
```

두 번째 인수로 요소 채우기를 시작할 인덱스를 전달할 수 있다.

```jsx
const arr = [1, 2, 3];

// 인수로 전달받은 값 0을 배열의 인덱스 1부터 끝으로 요소로 채운다.
arr.fill(0, 1);

// fill 메서드는 원본 배열을 직접 변경한다.
console.log(arr); // [1, 0, 0]
```

세 번째 인수로 요소 채우기를 멈출 인덱스를 전달할 수 있다.

```jsx
const arr = [1, 2, 3, 4, 5];

// 인수로 전달받은 값 0을 배열의 인덱스 1부터 3이전(인덱스 3미포함)으로 요소로 채운다.
arr.fill(0, 1, 3);

// fill 메서드는 원본 배열을 직접 변경한다.
console.log(arr); // [1, 0, 0, 4, 5]
```

fill 메서드를 사용하면 배열을 생성하면서 특정 값으로 요소를 채울 수 있다.

```jsx
const arr = new Array(3);
arr.fill(0);
console.log(arr); // [0, 0, 0]
```

fill 메서드는 하나의 값만으로 채울 수밖에 없다. 만약 Array.from 메서드를 활용하면 두 번째 인수로 전달한 콜백 함수를 통해 요소값을 만들면서 배열을 채울 수 있다.

```jsx
const sequences = (length = 0) => Array.from({ length }, (_, i) => i);
console.log(sequences(3)); // [0, 1, 2]
```

## 8.13 Array.prototype.includes

ES7에서 도입된 includes 메서드는 배열 내에 특정 요소가 포함되어 있는지 확인하여 true 또는 false를 반환한다.

첫 번째 인수로 검색할 대상을 지정한다.

```jsx
const arr = [1, 2, 3];

arr.includes(2); // true
arr.includes(100); // false
```

두 번째 인수로 검색을 시작할 인덱스를 전달할 수 있다. 생략할 경우 기본값 0이 설정된다.

만약 두 번째 인수에 음수를 전달하면 length 프로퍼티 값과 음수 인덱스를 합산하여 (length + index) 검색 시작 인덱스를 설정한다.

```jsx
const arr = [1, 2, 3];

arr.includes(1, 1); // false
// 인덱스(arr.length - 1)부터 확인한다.
arr.includes(3, -1); // true
```

includes 메서드를 사용하면 NaN이 포함되어 있는지 확인할 수 있다.

```jsx
[NaN].indexOf(NaN) !== -(1)[NaN].includes(NaN); // false // true
```

## 8.14 Array.prototype.flat

ES10에서 도입된 flat 메서드는 인수로 전달한 깊이만큼 재귀적으로 배열을 평탄화한다.

```jsx
[1, [2, 3, 4, 5]].flat(); // [1, 2, 3, 4, 5]
```

중첩 배열을 평탄화할 인수로 전달할 수 있다. 인수를 생략할 경우 기본값은 1이다.

인수로 Infinity를 전달하면 중첩 배열 모두를 평탄화한다.

```jsx
const arr = [1, [2, [3, [4]]]].flat();
console.log(arr); // [ 1, 2, [3, [ 4 ] ]]
console.log(arr.flat(2)); // [ 1, 2, 3, [ 4 ] ]
console.log(arr.flat(Infinity)); // [ 1, 2, 3, 4 ]
```

# 9. 배열 고차 함수

고차 함수는 외부 상태의 변경이나 가변 데이터를 피하고 **불변성**을 지향하는 함수형 프로그래밍에 기반을 두고 있다. 순수 함수를 통해 부수 효과를 최대한 억제하여 오류를 피하고 프로그램의 안정성을 높이려는 노력의 일환.

## 9.1 Array.prototype.sort

sort 메서드는 배열의 요소를 정렬한다. 기본적으로 오름차순으로 정렬한다.

```jsx
const fruits = ['banana', 'orange', 'apple'];
fruits.sort();
console.log(fruits); // [ 'apple', 'banana', 'orange' ]
```

한글 문자열인 요소도 오름차순으로 정렬된다.

```jsx
const fruits = ['바나나', '오렌지', '사과'];
fruits.sort();
console.log(fruits); // [ '바나나', '사과', '오렌지' ]
```

내림차순으로 정렬하고 싶다면 오름차순으로 정렬한 뒤 reverse 메서드를 사용한다.

```jsx
const fruits = ['banana', 'orange', 'apple'];
fruits.sort();
fruits.reverse();
console.log(fruits); // [ 'orange', 'banana', 'apple' ]
```

문제는 숫자 요소로 이루어진 배열을 정렬할 때는 주의가 필요하다.

```jsx
const points = [40, 100, 1, 5, 2, 25, 10];
points.sort();
console.log(points); // [1, 10, 100, 2, 25, 40, 5]
```

sort 메서드는 기본 정렬 순서는 유니코드 코드 포인트의 순서를 따른다.

숫자 요소를 정렬할 때는 sort 메서드에 정렬 순서를 정의하는 비교 함수를 인수로 전달해야 한다.

```jsx
const points = [40, 100, 1, 5, 2, 25, 10];

// 숫자 배열의 오름차순 정렬. 비교 함수의 반환값이 0보다 작으면 a를 우선하여 정렬한다.
points.sort((a, b) => a - b);
console.log(points); // [1, 2, 5, 10, 25, 40, 100]

// 내림차순 정렬
points.sort((a, b) => b - a);
console.log(points); // [100, 40, 25, 10, 5, 2, 1]
```

객체를 요소로 갖는 배열을 정렬한다면

```jsx
const todos = [
  { id: 4, content: 'Javascript' },
  { id: 1, content: 'HTML' },
  { id: 2, content: 'CSS' },
];

// 비교 함수, 매개변수 key는 프로퍼티다.
function compare(key) {
  // 프로퍼티의 값이 문자열인 경우 - 산술 연산으로 비교하면 NaN이 나오므로 비교 연산을 사용한다.
  // 비교 함수는 양수/음수/0을 반환하면 되므로 - 산술 연산 대신 비교 연산을 사용할 수 있다.
  return (a, b) => (a[key] > b[key] ? 1 : a[key] < b[key] ? -1 : 0);
}

// id를 기준으로 오름차순 정렬
todos.sort(compare('id'));
console.log(todos);
// [
//     { id: 1, content: 'HTML' },
//     { id: 2, content: 'CSS' },
//     { id: 4, content: 'Javascript' }
// ]

// content를 기준으로 오름차순 정렬
todos.sort(compare('content'));
console.log(todos);
// [
//     { id: 2, content: 'CSS' },
//     { id: 1, content: 'HTML' },
//     { id: 4, content: 'Javascript' }
// ]
```

## 9.2 Array.prototype.forEach

forEach 메서드는 for문을 대체할 수 있는 고차 함수다.

```jsx
const number = [1, 2, 3];
const pows = [];

// for 문으로 배열 순회
for (let i = 0; i < number.length; i++) {
  pows.push(number[i] ** 2);
}

// forEach 메서드는 numbers 배열의 모든 요소를 순회하면서 콜백 함수를 반복 호출한다.
numbers.forEach(item => pows.push(item ** 2));
console.log(pows); // [1, 4, 9]
```

forEach 메서드를 호출한 배열의 요소값과 인덱스, forEach 메서드를 호출한 배열(this)를 순차적으로 전달한다.

```jsx
// forEach 메서드는 콜백 함수를 호출하면서 3개(요소값, 인덱스, this)의 인수를 전달한다.
[1, 2, 3].forEach((item, index, arr) => {
  console.log(`요소값: ${item}, 인덱스: ${index}, this: ${JSON.stringify(arr)}`)
});

/*
요소값: 1, 인덱스: 0, this: [1,2,3]
요소값: 2, 인덱스: 1, this: [1,2,3]
요소값: 3, 인덱스: 2, this: [1,2,3]
*/
```

forEach 메서드는 원본 배열을 변경하지 않는다. 하지만 콜백 함수를 통해 원본 배열을 변경할 수는 있다.

```jsx
const numbers = [1, 2, 3];

// 콜백 함수의 세 번째 매개변수 arr은 원본 배열 numbers를 가리킨다.
// 따라서 콜백 함수의 세 번째 매개변수 arr을 직접 변경하면 원본 배열 numbers가 변경된다.
numbers.forEach((item, index, arr) => {
  arr[index] = item ** 2;
});
console.log(numbers); // [1, 4, 9]
```

forEach 메서드의 반환값은 언제나 undefined다.

```jsx
const result = [1, 2, 3].forEach(console.log);
console.log(result); // undefined
```

forEach 메서드의 두 번째 인수로 forEach 메서드의 콜백 함수 내부에서 this로 사용할 객체를 전달할 수 있다.

```jsx
class Numbers {
  numberArray = [];

  multiply(arr) {
    arr.forEach(function(item) {
      // 그러나 TypeError 가 발생한다.
      this.numberArray.push(item * item);
    });
  }
}

const numbers = new Numbers();
numbers.multiply([1, 2, 3]);
```

화살표 함수로 고친다면

```jsx
class Numbers {
  numberArray = [];

  multiply(arr) {
    arr.forEach(item => this.numberArray.push(item * item));
  }
}

const numbers = new Numbers();
numbers.multiply([1, 2, 3]);
console.log(numbers.numberArray); // [1, 4, 9]
```

forEach 문은 for문과는 다르게 break, continue 문을 사용할 수 없다.

```jsx
[1, 2, 3].forEach(item => {
  console.log(item)
  if (item > 1) break // SyntaxError
  if (item === 1) continue // SyntaxError
})
```

희소 배열의 경우 존재하지 않는 요소는 순회 대상에서 제외된다.

forEach 메서드는 for 문에 비해 성능이 좋지는 않지만 가독성은 더 좋다.

## 9.3 Array.prototype.map

map 메서드는 자신을 호출한 배열의 모든 요소를 순회하면서 인수로 전달받은 콜백 함수를 반복 호출한다.

**콜백 함수의 반환값들로 구성된 새로운 배열을 반환한다.**

```jsx
const numbers = [1, 4, 9];

// map 메서드는 numbers 배열의 모든 요소를 순회하면서 콜백 함수를 반복 호출한다.
// 그리고 콜백 함수의 반환값들로 구성된 새로운 배열을 반환한다.
const roots = numbers.map(item => Math.sqrt(item));
console.log(roots); // [1, 2, 3]
console.log(numbers); // [1, 4, 9]
```

forEach 메서드는 단순히 반복문을 대체하기 위한 고차함수다.

map 메서드는 요소값을 다른 값으로 매핑한 새로운 배열을 생성하기 위한 고차 함수다.

map 메서드도 요소값, 인덱스 map 메서드를 호출한 배열(this)를 순차적으로 전달한다.

```jsx
class Prefixer {
  constructor(prefix) {
    this.prefix = prefix;
  }

  add(arr) {
    return arr.map(item => this.prefix + item);
  }
}

const prefixer = new Prefixer('-webkit-');
console.log(prefixer.add(['transition', 'user-select']));
// [ '-webkit-transition', '-webkit-user-select' ]
```

## 9.4 Array.prototype.filter

**콜백 함수의 반환값이 true인 요소로만 구성된 새로운 배열을 반환한다.**

```jsx
const numbers = [1, 2, 3, 4, 5];

const odds = numbers.filter(item => item % 2 === 1);
console.log(odds); // [1, 3, 5]
```

filter 메서드도 요소값, 인덱스 filter 메서드를 호출한 배열(this)를 순차적으로 전달한다.

filter 메서드는 자신을 호출한 배열에서 특정 요소를 제거하기 위해 사용할 수도 있다.

```jsx
class Users {
  constructor() {
    this.users = [
      { id: 1, name: 'Son' },
      { id: 2, name: 'Kim' },
    ];
  }

  // 요소 추출
  findById(id) {
    // id가 일치하는 사용자만 반환한다.
    return this.users.filter(user => user.id === id);
  }

  // 요소 제거
  remove(id) {
    this.users = this.users.filter(user => user.id !== id);
  }
}

const users = new Users();
let user = users.findById(1);
console.log(user); // [ { id: 1, name: 'Son' } ]

// id가 1인 사용자를 제거
users.remove(1);

user = users.findById(1);
console.log(user); // []
```

## 9.5 Array.prototype.reduce

콜백 함수의 반환값을 다음 순회 시에 콜백 함수의 첫 번째 인수로 전달하면서 콜백 함수를 호출하여 **하나의 결과값을 만들어 반환한다.** 이때 원본 배열은 변경되지 않는다.

reduce 메서드는 첫 번째 인수로 콜백 함수, 두 번째 인수로 초기값을 전달받는다.

reduce 메서드의 콜백 함수에는 4개의 인수, 초기값 or 콜백 함수의 이전 반환값, reduce 메서드를 호출한 배열의 요소값과 인덱스, reduce 메서드를 호출한 배열 자체 즉 this가 전달된다.

```jsx
// 1부터 4까지 누적을 구한다.
const sum = [1, 2, 3, 4].reduce(
  (accumulator, currentValue, index, array) => accumulator + currentValue,
  0
);

console.log(sum);
```

**reduce 메서드는 하나의 결과값을 반환한다.**

다음은 reduce 메서드 활용법이다.

```jsx
// 평균 구하기
const values1 = [1, 2, 3, 4, 5, 6];

const average = values1.reduce((acc, cur, i, { length }) => {
  // 마지막 순회가 아니면 누적값을 반환하고 마지막 순회면 누적값으로 평균을 구해 반환
  return i === length - 1 ? (acc + cur) / length : acc + cur
}, 0);

console.log(average); // 3.5

// 최대값 구하기
const values2 = [1, 2, 3, 4, 5, 6];

const max = values2.reduce((acc, cur) => (acc > cur ? acc : cur), 0);
console.log(max); // 6

// Math.max 메서드 활용
const max2 = Math.max(...values2);
console.log(max2); // 6

// 요소의 중복 횟수 구하기
const fruits = ['banana', 'apple', 'orange', 'orange', 'apple'];

const count = fruits.reduce((acc, cur) => {
  // 첫 번째 순회 시 acc는 초기값인 {}이고 cur은 첫 번째 요소인 "banana"
  // 초기값으로 전달받은 빈 객체에 요소값인 cur을 프로퍼티 키로, 요소의 개수를 프로퍼티값으로 할당
  // 만약 프로퍼티 값이 undefined(처음 등장하는 요소)이면 프로퍼티 값을 1로 초기화
  acc[cur] = (acc[cur] || 0) + 1;
  return acc;
}, {});

console.log(count); // { banana: 1, apple: 2, orange: 2 }

// 중첩 배열 평탄화
const values = [1, [2, 3], 4, [5, 6]];

const flatten = values.reduce((acc, cur) => acc.concat(cur), []);
console.log(flatten);

// flat 메서드
const flatten2 = values.flat(Infinity);
console.log(flatten2);

// 중복 요소 제거
const values = [1, 2, 1, 3, 5, 4, 5, 3, 4, 4]

const result = values.reduce((acc, cur, i, arr) => {
  if (arr.indexOf(cur) === i) {
    acc.push(cur);
  }
  return acc;
}, []);

console.log(result);

// filter
const result = values.filter((v, i, arr) => arr.indexOf(v) === i);
console.log(result);

// Set
const result2 = [...new Set(values)];
console.log(result2);
```

두 번째 인수로 전달하는 초기값이 옵션이다. 생략 할 수 있다.

**그러나 reduce 메서드를 호출할 때는 언제나 초기값을 전달하는 것이 안전하다.**

```jsx
const sum = [].reduce((acc, cur) => acc + cur) // TypeError
```

초기값을 전달하면 에러가 발생하지 않는다.

```jsx
const sum = [].reduce((acc, cur) => acc + cur, 0);
console.log(sum); // 0

// 객체인 경우
const products = [
  { id: 1, price: 100 },
  { id: 2, price: 200 },
  { id: 3, price: 300 },
]

// 1번째 순회 시 acc는 {id: 1, price: 100}, cur {id:2, price: 200}인데
// 2번째 순회 시 acc 는 300, cur {id: 3, price: 300}이다.
// undefined가 되어 더하게 되었을 때 NaN이 발생한다.
const priceSum = products.reduce((acc, cur) => acc.price + cur.price);
console.log(priceSum); // NaN

const priceSum = products.reduce((acc, cur) => acc + cur.price, 0);
console.log(priceSum); // 600
```

## 9.6 Array.prototype.some

some 메서드는 콜백 함수의 반환값이 단 한번이라도 참이면 true, 모두 거짓이면 false를 반환한다.

```jsx
// 배열의 요소 중 10보다 큰 요소가 1개 이상 존재하는지 확인
[5, 10, 15].some(item => item > 10); // true

// 배열의 요소 중 0보다 큰 요소가 1개 이상 존재하는지 확인
[5, 10, 15].some(item => item > 0); // false

// some 메서드를 호출한 배열이 빈 배열인 경우 언제나 false를 반환한다.
[].some(item => item > 3); // false
```

## 9.7 Array.prototype.every

every 메서드는 콜백 함수의 반환값이 모두 참이면 true, 단 한번이라도 거짓이면 false를 반환한다.

```jsx
// 배열의 모든 요소가 3보다 큰지 확인
[5, 10, 15].every(item => item > 3); // true

// 배열의 모든 요소가 10보다 큰지 확인
[5, 10, 15].every(item => item > 10); // false

// every 메서드를 호출한 배열이 빈 배열인 경우 언제나 true를 반환한다.
[].every(item => item > 3); // true
```

## 9.8 Array.prototype.find

콜백 함수를 호출하면서 반환값이 true인 첫 번째 요소를 반환한다.

true인 요소가 없다면 undefined를 반환한다.

```jsx
const users = [
  { id: 1, name: 'Son' },
  { id: 2, name: 'Kim' },
  { id: 2, name: 'Park' },
  { id: 3, name: 'Choi' },
]

users.find(user => user.id === 2); // { id: 2, name: "Kim" }
```

find 메서드는 콜백 함수의 반환값이 true인 첫 번째 요소를 반환하므로 true의 결과값은 해당 요소다.

```jsx
// filter 메서드는 배열을 반환한다.
[1, 2, 2, 3].filter(item => item === 2); // [2, 2]

// find 메서드는 요소를 반환한다.
[1, 2, 2, 3].find(item => item === 2); // 2
```

## 9.9 Array.prototype.findIndex

콜백 함수를 호출하면서 반환값인 true인 첫 번째 요소의 인덱스를 반환한다.

true인 요소가 없다면 -1을 반환한다.

```jsx
const users = [
  { id: 1, name: 'Son' },
  { id: 2, name: 'Kim' },
  { id: 2, name: 'Park' },
  { id: 3, name: 'Choi' },
]

// id가 2인 요소의 인덱스
users.findIndex(user => user.id === 2); // 1

// name이 "Park"인 요소의 인덱스
users.findIndex(user => user.name === 'Park'); // 2

// 다음과 같은 콜백 함수를 추상화가 가능하다.
function predicate(key, value) {
  return item => item[key] === value;
}

// id가 2인 요소의 인덱스
useres.findIndex(predicate('id', 2)); // 1

// name이 "Park"인 요소의 인덱스
users.findIndex(predicate('name', 'Park')); // 2
```

## 9.10 Array.prototype.flatMap

map 메서드를 통해 생성된 새로운 배열을 평탄화한다.

→ map 메서드와 flat 메서드를 순차적으로 실행하는 효과가 있다.

```jsx
const arr = ['hello', 'world'];

// map과 flat을 순차적으로 실행
arr.map(x => x.split('').flat());

// flatMap
arr.flatMap(x => x.split(''));
```

flatMap은 1단계만 평탄화가 가능하다. map 메서드와 flat메서드를 각각 호출해야 한다.
