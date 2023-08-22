---
title: 'Modrn Javascript Deep Dive - 37장 Set과 Map'
date: 2023-08-22
category: 'Javascript'
draft: false
---

# 1. Set

**Set 객체는 중복되지 않는 유일한 값들의 집합이다.**

## 1. 1 Set 객체의 생성

```jsx
const set = new Set()
console.log(set) // Set(0) {}
```

**Set 생성자 함수는 이터러블을 인수로 전달받아 Set 객체를 생성한다. 이때 이터러블의 중복된 값은 Set 객체에 요소로 저장되지 않는다.**

```jsx
const set1 = new Set([1, 2, 3, 3])
console.log(set1) // Set(3) {1, 2, 3}

const set2 = new Set('hello')
console.log(set2) // Set(4) { 'h', 'e', 'l', 'o' }
```

중복을 허용하지 않는 Set 객체의 특성을 활용하여 배열에서 중복된 요소를 제거할 수 있다.

```jsx
const array = [2, 1, 2, 3, 4, 3, 4]

const uniq1 = array => array.filter((v, i, self) => self.indexOf(v) === i)
console.log(uniq1(array)) // [2, 1, 3, 4]

const uniq2 = array => [...new Set(array)]
console.log(uniq2(array)) // [2, 1, 3, 4]
```

## 1.2 요소 개수 확인

Set 객체의 요소 개수를 확인할 때는 Set.prototype.size 프로퍼티를 사용한다.

```jsx
const { size } = new Set([2, 1, 3, 3])
console.log(size) // 3
```

size 프로퍼티는 getter 함수만 존재하는 접근자 프로퍼티다.

size 프로퍼티에 숫자를 할당하여 Set 객체의 요소 개수를 변경할 수 없다.

```jsx
const set = new Set([1, 2, 3])

set.size = 10 // 무시됨.
console.log(set.size)
```

## 1.3 요소 추가

Set 객체에 요소를 추가할 때는 Set.prototype.add 메서드를 사용한다.

```jsx
const set = new Set();
console.log(set); // Set(0) {}

set.add(1);
console.log(set); // Set(1) {1}

// add 메서드는 새로운 요소가 추가된 Set 객체를 반환한다.
// add 메서드를 연속적으로 호출이 가능하다.
set.add(1).add(2);
console.log(set); // Set(2) { 1, 2 }

----
const set = new Set();

// 일치 비교 연산자 ===을 사용하면 NaN과 NaN을 다르다고 평가한다.
// Set 객체는 NaN과 NaN을 같다고 평가하여 중복 추가를 허용하지 않는다.
// +0, -0은 일치 비교 연산자 ===와 마찬가지로 같다고 평가한다.
console.log(NaN === NaN); // false
console.log(+0 === -0); // true

set.add(NaN).add(NaN);
console.log(set); // Set(1) { NaN }

set.add(0).add(-0);
console.log(set); // Set(1) { 0 }
```

Set 객체는 객체나 배열과 같이 자바스크립트의 모든 값을 요소로 저장할 수 있다.

```jsx
const set = new Set()
set
  .add(1)
  .add('a')
  .add(true)
  .add(undefined)
  .add(null)
  .add({})
  .add([])
  .add(() => {})

console.log(set)

/*
	Set(8) {
	  1,
	  'a',
	  true,
	  undefined,
	  null,
	  {},
	  [],
	  [Function (anonymous)]
	}
*/
```

## 1.4 요소 존재 여부 확인

Set 객체에 특정 요소가 존재하는지 확인하려면 Set.prototype.has 메서드를 사용한다.

has 메서드는 특정 요소의 존재 여부를 나타내는 불리언 값을 반환한다.

```jsx
const set = new Set([1, 2, 3])

console.log(set.has(2)) // true
console.log(set.has(4)) // false
```

## 1.5 요소 삭제

Set 객체에 특정 요소를 삭제하려면 Set.prototype.delete 메서드를 사용한다.

delete 메서드는 삭제 성공 여부를 나타내는 불리언 값을 반환한다. delete 메서드는 인덱스가 아니라 삭제하려는 요소값을 인수로 전달해야 한다.

```jsx
const set = new Set([1, 2, 3])

// 요소 2 삭제
set.delete(2)
console.log(set) // Set(2) { 1, 3 }

// 존재하지 않는 요소 0을 삭제하면 무시된다.
set.delete(0)
console.log(set) // Set(2) { 1, 3 }
```

## 1.6 요소 일괄 삭제

Set 객체에 모든 요소를 일괄 삭제하려면 Set.prototype.clear 메서드를 사용한다.

clear 메서드는 언제나 undefined를 반환한다.

```jsx
const set = new Set([1, 2, 3])

set.clear()
console.log(set) // Set(0) {}
```

## 1.7 요소 순회

Set 객체에 요소를 일괄 순회하려면 Set.prototype.forEach 메서드를 사용한다.

Array의 forEach 메서드처럼 3개의 인수와 함께 콜백 함수를 전달한다.

- 현재 순회 중인 요소값
- 현재 순회 중인 요소값
- 현재 순회 중인 Set 객체 자체

```jsx
const set = new Set([1, 2, 3])

set.forEach((v, v2, set) => console.log(v, v2, set))

/*
	1 1 Set(3) { 1, 2, 3 }
	2 2 Set(3) { 1, 2, 3 }
	3 3 Set(3) { 1, 2, 3 }
*/
```

**Set 객체는 이터러블이다.**

- `for…of` 문
- 스프레드 문법
- 배열 디스트럭처링

대상이 될 수 있다.

```jsx
const set = new Set([1, 2, 3])

// for...of 문
for (const value of set) {
  console.log(value) // 1 2 3
}

// 스프레드 문법
console.log([...set]) // [1, 2, 3]

// 배열 디스트럭처링
const [a, ...rest] = set
console.log(a, rest) // 1 [2, 3]
```

Set 객체는 요소의 순서에 의미를 갖지 않지만 Set 객체를 순회하는 순서는 요소가 추가된 순서를 따른다.

## 1.8 집합 연산

Set 객체는 수학적 집합을 구현하기 위한 자료구조다.

집합 연산을 수행하는 프로토타입 메서드를 구현하면 다음과 같다.

### 교집합

집합 A와 집합 B의 공통 요소로 구성한다.

```jsx
Set.prototype.intersection = function(set) {
  const result = new Set()

  for (const value of set) {
    // 2개의 set의 요소가 공통되는 요소이면 교집합의 대상이다.
    if (this.has(value)) {
      result.add(value)
    }
  }

  return result
}

const setA = new Set([1, 2, 3, 4])
const setB = new Set([2, 4])

console.log(setA.intersection(setB)) // Set(2) { 2, 4 }
console.log(setB.intersection(setA)) // Set(2) { 2, 4 }

// 다음과 같은 방법으로도 가능하다.
Set.prototype.intersection = function(set) {
  return new Set([...this].filter(v => set.has(v)))
}
```

### 합집합

집합 A와 집합 B의 중복 없는 모든 요소로 구성된다.

```jsx
Set.prototype.usage = function(set) {
  // this(Set 객체 복사)
  const result = new Set(this)

  for (const value of set) {
    // 중복된 요소는 포함되지 않도록 한다.
    result.add(value)
  }

  return result
}

const setA = new Set([1, 2, 3, 4])
const setB = new Set([2, 4])

console.log(setA.usage(setB)) // Set(4) { 1, 2, 3, 4 }
console.log(setB.usage(setA)) // Set(4) { 2, 4, 1, 3 }

// 다음과 같은 방법으로도 가능하다.
Set.prototype.usage = function(set) {
  return new Set([...this, ...set])
}
```

### 차집합

A-B 는 집합 A에는 존재하지만 집합 B에는 존재하지 않는 요소로 구성된다.

```jsx
Set.prototype.difference = function(set) {
  // this(Set 객체 복사)
  const result = new Set(this)

  for (const value of set) {
    // 어느 한쪽 집합에는 존재하지만 다른 한쪽 집합에는 존재하지 않는 요소로 구성한 집합이다.
    result.delete(value)
  }

  return result
}

const setA = new Set([1, 2, 3, 4])
const setB = new Set([2, 4])

console.log(setA.difference(setB))
console.log(setB.difference(setA))

// 다음과 같은 방법으로도 가능하다.
Set.prototype.difference = function(set) {
  return new Set([...this].filter(v => !set.has(v)))
}
```

### 부분집합과 상위 집합

```jsx
Set.prototype.isSuperset = function(subset) {
  for (const value of subset) {
    // superset의 모든 요소가 subset의 모든 요소를 포함하는지 확인
    if (!this.has(value)) {
      return false
    }
  }
  return true
}

const setA = new Set([1, 2, 3, 4])
const setB = new Set([2, 4])

console.log(setA.isSuperset(setB))
console.log(setB.isSuperset(setA))

// 다음과 같은 방법으로도 가능하다.
Set.prototype.isSuperset = function(subset) {
  const supersetArr = [...this]
  return [...subset].every(v => supersetArr.includes(v))
}
```

# 2. Map

**Map 객체는 키와 쌍으로 이루어진 컬렉션이다.** Map 객체는 객체와 유사하나 다음과 같은 차이가 있다.

| 구분                   | 객체                    | Map 객체              |
| ---------------------- | ----------------------- | --------------------- |
| 키로 사용할 수 있는 값 | 문자열 또는 심벌 값     | 객체를 포함한 모든 값 |
| 이터러블               | X                       | O                     |
| 요소 개수 확인         | Object.keys(obj).length | map.size              |

## 2.1 Map 객체의 생성

Map 객체는 Map 생성자 함수로 생성한다. Map 생성자 함수에 인수를 전달하지 않으면 빈 Map이 생성된다.

```jsx
const map = new Map()
console.log(map) // Map(0) {}
```

Map 생성자 함수는 이터러블을 인수로 전달받아 Map 객체를 생성한다.

인수로 전달되는 이터러블은 키와 쌍으로 이루어진 요소로 구성되어야 한다.

```jsx
const map1 = new Map([
  ['key1', 'value1'],
  ['key2', 'value2'],
])
console.log(map1) // Map(2) { 'key1' => 'value1', 'key2' => 'value2' }

const map2 = new Map([1, 2]) // TypeError
```

중복된 키는 Map 객체에 요소로 저장되지 않는다.

```jsx
const map = new Map([
  ['key1', 'value1'],
  ['key1', 'value2'],
])
console.log(map) // Map(1) { 'key1' => 'value2'}
```

## 2.2 요소 개수 확인

Map 객체의 요소 개수를 확인할 때는 Map.prototype.size 프로퍼티를 사용한다.

```jsx
const { size } = new Map([
  ['key1', 'value1'],
  ['key2', 'value2'],
])
console.log(size) // 2
```

size 프로퍼티는 getter 함수만 존재하는 접근자 프로퍼티다.

size 프로퍼티에 숫자를 할당하여 Set 객체의 요소 개수를 변경할 수 없다.

```jsx
const map = new Map([
  ['key1', 'value1'],
  ['key2', 'value2'],
])

map.size = 10
console.log(map.size) // 2
```

## 2.2 요소 추가

Map 객체의 요소를 추가할 때는 Map.prototype.set 프로퍼티를 사용한다.

```jsx
const map = new Map()
console.log(map) // Map(0) {}

map.set('key1', 'value1')
console.log(map) // Map(1) { 'key1' => 'value1' }

// set 메서드는 새로운 요소가 추가된 Map 객체를 반환한다.
// set 메서드는 연속적으로 호출할 수 있다.
map.set('key1', 'value1').set('key2', 'value2')
console.log(map) // Map(2) { 'key1' => 'value1', 'key2' => 'value2' }

// Map 객체에 중복된 키를 갖는 요소의 추가는 허용되지 않는다.
// 에러는 발생하지 않고 무시한다.
map.set('key1', 'value1').set('key1', 'value2')
console.log(map) // Map(1) { 'key1' => 'value2' }

// NaN, +0,, -0도 Set과 마찬가지로 같다고 평가하여 중복 추가를 허용하지 않는다.
map.set(NaN, 'value1').set(NaN, 'value2')
console.log(map) // Map(1) { NaN => 'value2' }

map.set(0, 'value1').set(-0, 'value2')
console.log(map) // Map(1) { 0 => 'value2' }
```

객체는 문자열 또는 심벌 값만 키로 사용할 수 있다.

Map 객체는 키 타입에 제한이 없다. → 객체를 포함한 모든 값을 키로 사용할 수 있다.

```jsx
const map = new Map()

const lee = { name: 'Lee' }
const son = { name: 'Son' }

map.set(lee, 'developer').set(son, 'designer')
console.log(map)

/*
	Map(2) 
	{
	  { name: 'Lee' } => 'developer',
	  { name: 'Son' } => 'designer'
	}
*/
```

## 2.4 요소 취득

Map 객체에서 특정 요소를 취득하려면 Map.prototype.get 메서드를 사용한다.

get 메서드의 인수로 키를 전달하면 Map 객체에서 인수로 전달한 키를 갖는 값을 반환한다.

존재하지 않으면 undefined를 반환한다.

```jsx
const map = new Map()

const lee = { name: 'Lee' }
const son = { name: 'Son' }

map.set(lee, 'developer').set(son, 'designer')

console.log(map.get(lee)) // developer
console.log(map.get('key')) // undefined
```

## 2.5 요소 존재 여부 확인

Map 객체에서 특정 요소가 존재하는지 확인하려면 Map.prototype.has 메서드를 사용한다.

has 메서드는 특정 요소의 존재 여부를 나타내는 불리언 값을 반환한다.

```jsx
const map = new Map()

const lee = { name: 'Lee' }
const son = { name: 'Son' }

map.set(lee, 'developer').set(son, 'designer')

console.log(map.has(lee)) // true
console.log(map.has('key')) // false
```

## 2.6 요소 삭제

Map 객체에서 요소를 삭제하려면 Map.prototype.delete 메서드를 사용한다.

delete 메서드는 삭제 성공 여부를 나타내는 불리언 값을 반환한다.

```jsx
const map = new Map()

const lee = { name: 'Lee' }
const son = { name: 'Son' }

map.set(lee, 'developer').set(son, 'designer')

map.delete(lee)
console.log(map) // Map(1) { { name: 'Son' } => 'designer' }

// 만약 존재하지 않는 키로 삭제하려하면 에러 없이 무시된다.
const map = new Map()

const lee = { name: 'Lee' }
const son = { name: 'Son' }

map.set(lee, 'developer').set(son, 'designer')

map.delete('key2')
console.log(map)

/*
	Map(2) 
	{
	  { name: 'Lee' } => 'developer',
	  { name: 'Son' } => 'designer'
	}
*/

// delete 메서드는 연속적으로 호출할 수 없다.
map.delete(lee).delete(son) // TypeError
```

## 2.7 요소 일괄 삭제

Map 객체에서 요소를 일괄 삭제하려면 Map.prototype.clear 메서드를 사용한다.

clear 메서드는 언제나 undefined 를 반환한다.

```jsx
const map = new Map()

const lee = { name: 'Lee' }
const son = { name: 'Son' }

map.set(lee, 'developer').set(son, 'designer')

map.clear()
console.log(map) // Map(0) {}
```

## 2.8 요소 순회

Map 객체에서 요소를 순회하려면 Map.prototype.forEach 메서드를 사용한다.

Array의 forEach 메서드처럼 3개의 인수와 함께 콜백 함수를 전달한다.

- 현재 순회 중인 요소값
- 현재 순회 중인 요소키
- 현재 순회 중인 Map 객체 자체

```jsx
const map = new Map()

const lee = { name: 'Lee' }
const son = { name: 'Son' }

map.set(lee, 'developer').set(son, 'designer')

map.forEach((v, k, map) => console.log(v, k, map))

/*
	developer { name: 'Lee' } Map(2) {
	  { name: 'Lee' } => 'developer',
	  { name: 'Son' } => 'designer'
	}
	designer { name: 'Son' } Map(2) {
	  { name: 'Lee' } => 'developer',
	  { name: 'Son' } => 'designer'
	}
*/
```

Map 객체는 이터러블이다.

- `for…of` 문
- 스프레드 문법
- 배열 디스트럭처링

대상이 될 수 있다.

```jsx
const map = new Map()

const lee = { name: 'Lee' }
const son = { name: 'Son' }

map.set(lee, 'developer').set(son, 'designer')

// for...of 문
for (const entry of map) {
  console.log(entry)
  // [ { name: 'Lee' }, 'developer' ]
  // [ { name: 'Son' }, 'designer' ]
}

// 스프레드 문법
console.log([...map])
// [ [ { name: 'Lee' }, 'developer' ], [ { name: 'Son' }, 'designer' ] ]

// 배열 디스트럭처링
const [a, b] = map
console.log(a, b)
// [ { name: 'Lee' }, 'developer' ] [ { name: 'Son' }, 'designer' ]
```

Map 객체는 이터러블이면서 동시에 이터레이터인 객체를 반환하는 메서드를 제공한다.

| Map 메서드            | 설명                                                                                |
| --------------------- | ----------------------------------------------------------------------------------- |
| Map.prototype.keys    | Map 객체에서 요소키를 값으로 갖는 이터러블 / 이터레이터인 객체를 반환한다.          |
| Map.prototype.values  | Map 객체에서 요소값을 값으로 갖는 이터러블 / 이터레이터인 객체를 반환한다.          |
| Map.prototype.entries | Map 객체에서 요소키와 요소값을 값으로 갖는 이터러블 / 이터레이터인 객체를 반환한다. |

```jsx
const map = new Map()

const lee = { name: 'Lee' }
const son = { name: 'Son' }

map.set(lee, 'developer').set(son, 'designer')

// Map.prototype.keys 문
for (const key of map.keys()) {
  console.log(key) // { name: 'Lee' } { name: 'Son' }
}

// Map.prototype.values 문
for (const value of map.values()) {
  console.log(value) // developer, designer
}

// Map.prototype.entries 문
for (const entry of map.entries()) {
  console.log(entry)
  // [ { name: 'Lee' }, 'developer' ] [ { name: 'Son' }, 'designer' ]
}
```

Map 객체는 요소의 순서에 의미를 갖지 않지만 Map 객체를 순회하는 순서는 요소가 추가된 순서를 따른다.
