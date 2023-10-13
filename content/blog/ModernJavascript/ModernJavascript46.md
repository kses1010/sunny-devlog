---
title: 'Modern Javascript Deep Dive - 46장 제너레이터와 async/await'
date: 2023-09-20 14:55:15
category: 'Javascript'
draft: false
---
# 1. 제너레이터란?

제너레이터(generator)는 코드 블록의 실행을 일시 중지했다가 필요한 시점에 재개할 수 있는 특수한 함수다.

1. 제너레이터 함수는 함수 호출자에게 함수 실행의 제어권을 양도할 수 있다.
2. 제너레이터 함수는 함수 호출자와 함수의 상태를 주고받을 수 있다.
3. 제너레이터 함수를 호출하면 제너레이터 객체를 반환한다.

# 2. 제너레이터 함수의 정의

제너레이터 함수는 `function*` 키워드로 선언한다. 그리고 하나 이상의 yield 표현식을 포함한다.
이것을 제외하면 일반 함수를 정의하는 방법과 같다.

```jsx
// 제너레이터 함수 선언문  
function* genDecFunc() {  
  yield 1;  
}  
  
// 제너레이터 함수 표현식  
const genExpFunc = function* () {  
  yield 1;  
};  
  
// 제너레이터 메서드  
const obj = {  
  * genObjMethod() {  
    yield 1;  
  }  
};  
  
// 제너레이터 클래스 메서드  
class MyClass {  
  * genClsMethod() {  
    yield 1;  
  }  
}
```

애스터리스크(`*`) 의 위치는 function 키워드와 함수 이름 사이라면 어디든지 상관없으나
주로 function 키워드 바로 뒤에 붙인다.

제너레이터 함수는 화살표 함수, new 연산자와 함께 생성자 함수로 정의할 수 없다.

```jsx
const genArrowFunc = * () => {
  yield 1;
}; // SyntaxError

function* genFunc() {
  yield 1;
}

new getFunc(); // TypeError
```

# 3. 제너레이터 객체

제너레이터 함수를 호출하면 일반 함수처럼 함수 코드 블록을 실행하는 것이 아니라 제너레이터 객체를 생성해 반환한다.
제너레이터 함수가 반환한 제너레이터 객체는 이터러블(iterable)이면서 동시에 이터레이터(iterator)다.

```jsx
// 제너레이터 함수  
function* genFunc() {  
  yield 1;  
  yield 2;  
  yield 3;  
}  
  
// 제너레이터 함수를 호출하면 제너레이터 객체를 반환한다.  
const generator = genFunc();  
  
// 제너레이터 객체는 이터러블이면서 동시에 이터레이터다.  
// 이터러블은 Symbol.iterator 메서드를 직접 구현하거나 프로토타입 체인을 통해 상속받은 객체다.  
console.log(Symbol.iterator in generator); // true  
// 이터레이터는 next 메서드를 갖는다.  
console.log('next' in generator); // true
```

제너레이터 객체는 next 메서드를 갖는 이터레이터이지만 이터레이터에는 없는 return, throw 메서드를 갖는다.
제너레이터 객체는 세 개의 메서드를 호출하면 다음과 같이 동작한다.

- next 메서드를 호출하면 제너레이터 함수의 yield 표현식까지 코드 블록을 실행하고 yield된 값을 value 프로퍼티 값으로, false를 done 프로퍼티 값으로 갖는 이터레이터 리절트 객체를 반환한다.
- return 메서드를 호출하면 인수로 전달받은 value 프로퍼티 값으로, true를 done 프로퍼티 값으로 갖는 이터레이터 리절트 객체를 반환한다.
- throw 메서드를 호출하면 인수로 전달받은 에러를 발생시키고 undefined를 value 프로퍼티 값으로, true를 done 프로퍼티 값을 갖는 이터레이터 리절트 객체를 반환한다.

```jsx
function* genFunc() {  
  try {  
    yield 1;  
    yield 2;  
    yield 3;  
  } catch (e) {  
    console.error(e);  
  }  
}  
  
const generator = genFunc();  
  
console.log(generator.next()); // {value: 1, done: false}  
console.log(generator.return('End!')); // {value: "End!", done: true}
console.log(generator.throw('Error!')); // {value: undefined, done: true}
```

# 4. 제너레이터의 일시 중지와 재개

일반 함수처럼 한 번에 코드 블록의 모든 코드를 일괄 실행하는 것이 아니라 yield 표현식까지만 실행한다.
**yield 키워드는 제너레이터 함수의 실행을 일시 중지시키거나 yield 키워드 뒤에 오는 표현식의 평가 결과를 제너레이터 함수 호출자에게 반환한다.**

```jsx
function* genFunc() {  
  yield 1;  
  yield 2;  
  yield 3;  
}  
  
const generator = genFunc();  
  
console.log(generator.next());  
console.log(generator.next());  
console.log(generator.next());
```

제너레이터 객체의 next 메서드를 호출하면 yield 표현식까지 실행되고 일시 중지된다.
이때 함수의 제어권이 호출자로 양도된다.
제너레이터 객체의 next 메서드는 value, done 프로퍼티를 갖는 이터레이터 리절트 객체를 반환한다.
next 메서드가 반환한 이터레이터 리절트 객체의 value 프로퍼티에는 yield 프로퍼티에는 yield 표현식에서 yield된 값이 할당되고 done 프로퍼티에는 제너레이터 함수가 끝까지 실행되었는지를 나타내는 불리언 값이 할당된다.

```
generator.next() -> yield -> generator.next() -> yield -> ...generator.next() -> return 
```

**제너레이터 객체의 next 메서드에 전달한 인수는 제너레이터 함수의 yield 표현식을 할당받는 변수에 할당된다.**

```jsx
function* genFunc() {  
  const x = yield 1;  
  const y = yield (x + 10);  
  
  return x + y;  
}  
  
const generator = genFunc(0);  
  
// 처음 호출하는 next 메서드에는 인수를 전달하지 않는다.  
// next 메서드가 반환하는 이터레이터 리절트 객체의 value 프로퍼티에는 첫 번째 yield된 값 1이 할당된다.  
let res = generator.next();  
console.log(res); // {value: 1, done: false}  
  
// next 메서드에 인수로 전달한 10은 genFunc 함수의 x변수에 할당된다.  
// next 메서드가 반환하는 이터레이터 리절트 객체의 value 프로퍼티에는 두 번째 yield된 값 20이 할당된다.  
res = generator.next(10);  
console.log(res); // {value: 20, done: false}  
  
// next 메서드에 인수로 전달한 20은 genFunc 함수의 x변수에 할당된다.  
// next 메서드가 반환하는 이터레이터 리절트 객체의 value 프로퍼티에는 두 번째 yield된 값 30이 할당된다.  
res = generator.next(20);  
console.log(res); // {value: 30, done: true}
```

# 5. 제너레이터의 활용
## 5.1 이터러블의 활용

다음 코드는 이터레이션 프로토콜을 준수하여 무한 피보나치 수열을 생성하는 함수다.

```jsx
// 무한 이터러블을 생성하는 함수  
const infiniteFibonacci = (function () {  
  let [pre, cur] = [0, 1];  
  return {  
    [Symbol.iterator]() {return this;},  
    next() {  
      [pre, cur] = [cur, pre + cur];  
      // 무한 이터러블이므로 done 프로퍼티를 생략한다.  
      return {value: cur};  
    }  
  };  
})();  
  
for (const num of infiniteFibonacci) {  
  if (num > 10000) break;  
  console.log(num);  
}
```

제너레이터를 사용하여 무한 피보나치 수열을 생성하는 함수다.

```jsx
// 무한 이터러블을 생성하는 제너레이터 함수  
const infiniteFibonacci = (function* () {  
  let [pre, cur] = [0, 1];  
  while (true) {  
    [pre, cur] = [cur, pre + cur];  
    yield cur;  
  }  
})();  
  
for (const num of infiniteFibonacci) {  
  if (num > 10000) break;  
  console.log(num);  
}
```

# 6. async/await

제너레이터를 사용해서 비동기 처리를 동기 처럼 동작이 가능하나 코드가 무척이나 장황해지고 가독성도 나빠진다. ES8에서는 제너레이터보다 간단하고 가독성 좋게 비동기 처리를 동기 처리처럼 동작하도록 구현할 수 있는 `async/await`가 도입되었다.

`async/await`는 프로미스를 기반으로 동작한다.

```jsx
const fetch = require('node-fetch');  
  
async function fetchTodo() {  
  const url = 'https://jsonplaceholder.typicode.com/todos/1';  
  
  const response = await fetch(url);  
  const todo = await response.json();  
  console.log(todo);  
  // {userId: 1, id: 1, title: 'delectus aut autem', completed: false}  }  
  
fetchTodo();
```

## 6.1 async 함수

await 키워드는 반드시 async 함수 내부에서 사용해야 한다.
async 함수는 async 키워드를 사용해 정의하며 언제나 프로미스를 반환한다.
async 함수가 명시적으로 프로미스를 반환하지 않더라도 async 함수는 암묵적으로 반환값을 resolve하는 프로미스를 반환한다.

```jsx
// async 함수 선언문  
async function foo(n) { return n;}  
foo(1).then(v => console.log(v)); // 1  
  
// async 함수 표현식  
const bar = async function (n) { return n;};  
bar(2).then(v => console.log(v)); // 2  
  
// async 화살표 함수  
const baz = async n => n;  
baz(3).then(v => console.log(v)); // 3  
  
// async 메서드  
const obj = {  
  async foo(n) {return n;}  
};  
obj.foo(4).then(v => console.log(v)); // 4  
  
// async 클래스 메서드  
class MyClass {  
  async bar(n) {return n;}  
}  
  
const myClass = new MyClass();  
myClass.bar(5).then(v => console.log(v)); // 5
```

## 6.2 await 키워드

await 키워드는 프로미스가 settled 상태(비동기 처리가 수행된 상태)가 될 때까지 대기하다가 settled 상태가 되면 프로미스가 resolve한 처리 결과를 반환한다.
await 키워드는 반드시 프로미스 앞에서 사용해야 한다.

```jsx
const fetch = require('node-fetch');  
  
const getGithubUserName = async id => {  
  const res = await fetch(`https://api.github.com/users/${id}`);  
  const {name} = await res.json();  
  console.log(name); // sunny-son  
}  
  
getGithubUserName('sunny');
```

**프로미스가 settled 상태가 되면 프로미스가 resolve한 처리 결과가 res 변수에 할당된다.**
await 키워드는 다음 실행을 일시 중지시켰다가 프로미스가 settled 상태가 되면 다시 재개한다.

```jsx
async function foo() {  
  const a = await new Promise(resolve => setTimeout(() => resolve(1), 3000));  
  const b = await new Promise(resolve => setTimeout(() => resolve(2), 2000));  
  const c = await new Promise(resolve => setTimeout(() => resolve(3), 1000));  
  console.log([a, b, c]); // [1, 2, 3]  
}  
  
foo(); // 6초소요
```

모든 프로미스에 await 키워드를 사용하는 것은 주의해야 한다. 위 예제는 비동기 처리이므로 굳이 순차적으로 처리 할 필요가 없다.

```jsx
async function foo() {  
  const res = await Promise.all([  
    await new Promise(resolve => setTimeout(() => resolve(1), 3000)),  
    await new Promise(resolve => setTimeout(() => resolve(2), 2000)),  
    await new Promise(resolve => setTimeout(() => resolve(3), 1000))  
  ]);  
  console.log(res); // [1, 2, 3]  
}  
  
foo(); // 3초 소요
```

다음 함수는 앞선 비동기 처리의 결과를 가지고 다음 비동기 처리를 수행해야 한다.
비동기 처리의 처리 순서가 보장되어야 하므로 모든 프로미스에 await 키워드를 써서 순처작으로 처리할 수밖에 없다.

```jsx
async function foo(n) {  
  const a = await new Promise(resolve => setTimeout(() => resolve(n), 3000));  
  const b = await new Promise(resolve => setTimeout(() => resolve(a + 1), 2000));  
  const c = await new Promise(resolve => setTimeout(() => resolve(b + 1), 1000));  
  console.log([a, b, c]); // [1, 2, 3]  
}  
  
foo(1); // 6초소요
```

## 6.3 에러 처리

비동기 처리를 위한 콜백 패턴의 단점 중 가장 심각한 것은 에러 처리가 곤란하다는 것이다.
비동기 함수의 콜백함수를 호출한 것은 비동기 함수가 아니기 때문에 `try...catch문`을 사용해 에러를 캐치할 수 없다.

```jsx
try {  
  setTimeout(() => {throw new Error('Error!');}, 1000);  
} catch (e) {  
  // 에러를 캐치하지 못한다.  
  console.error('캐치한 에러', e);  
}
```

async/await 에서 에러 처리는 `try...catch문`을 사용할 수 있다.
프로미스를 반환하는 비동기 함수는 명시적으로 호출할 수 있기 때문에 호출자가 명확하다.

```jsx
const fetch = require('node-fetch');  
  
const foo = async () => {  
  try {  
    const wrongUrl = 'https://wrong.url';  
  
    const response = await fetch(wrongUrl);  
    const data = await response.json();  
    console.log(data);  
  } catch (e) {  
    console.error(e); //TypeError: Failed to fetch  
  }  
};  
  
foo();
```

foo 함수의 catch 문은 HTTP 통신에서 발생한 네트워크 에러뿐 아니라 try 코드 블록 내의 모든 문에서 발생한 일반적인 에러까지 모두 캐치할 수 있다.

**async 함수 내에서 catch 문을 사용해서 에러 처리를 하지 않으면 async 함수는 발생한 에러를 reject 하는 프로미스를 반환한다.**

```jsx
const fetch = require('node-fetch');  
  
const foo = async () => {  
    const wrongUrl = 'https://wrong.url';  
  
    const response = await fetch(wrongUrl);  
    const data = await response.json();  
    return data;
};  
  
foo()
  .then(console.log)
  .catch(console.error); // Type
```
