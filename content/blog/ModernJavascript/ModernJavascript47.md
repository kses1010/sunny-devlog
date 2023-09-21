---
title: 'Modern Javascript Deep Dive - 47장 에러 처리'
date: 2023-09-21 16:58:00
category: 'Javascript'
draft: false
---

# 1. 에러 처리의 필요성

에러에 대해 대처하지 않고 방치하면 프로그램은 강제 종료된다.

```jsx
console.log('[Start]');  
  
foo(); // ReferenceError  
  
// 에러에 의해 프로그램이 강제 종료되어 이 코드는 실행되지 않는다.  
console.log('[End]');
```

`try...catch문`을 사용해 발생한 에러에 적절하게 대응하면 프로그램이 강제 종료되지 않고 계속해서 코드를 실행할 수 있다.

```jsx
console.log('[Start]');  
  
try {  
  foo(); // ReferenceError  
} catch (error) {  
  console.error('[에러 발생]', error);  
}

console.log('[End]');
```

직접적으로 에러를 발생시키지 않는 예외적인 상황이 발생할 수도 있다. 예외적인 상황에 적절하게 대응하지 않으면 에러로 이어질 가능성이 크다.

# 2. try...catch...finally 문

에러 처리를 구현하는 방법은 크게 두 가지가 있다.

1. 예외적인 상황이 발생하면 반환하는 값(null or -1)을 if문이나 옵셔널 체이닝 연산자를 통해 확인해서 처리하는 방법
2. `try...catch...finally` 문을 활용하여 처리하는 방법이다.

```jsx
try {
  // 실행할 코드(에러가 발생할 가능성이 있는 코드)
} catch (err) {
  // try 코드 블록에서 에러가 발생하면 이 코드 블록의 코드가 실행된다.
  // err에는 try 코드 블록에서 발생한 Error 객체가 전달된다.
} finally {
  // 에러 발생과 상관없이 반드시 한 번 실행한다.
}
```

# 3. Error 객체

Error 생성자 함수는 에러 객체를 생성한다. Error 생성자 함수에는 에러를 상세히 설명하는 에러 메시지를 인스로 전달할 수 있다.

```jsx
const error = new Error('invalid');
```

Error 생성자 함수가 생성한 에러 객체는 message 프로퍼티와 stack 프로퍼티를 갖는다.
message 프로퍼티의 값은 Error 생성자 함수에 인수로 전달한 에러 메시지이고,
stack 프로퍼티의 값은 에러를 발생시킨 콜스택의 호출 정보를 나타내는 문자열이며 디버깅 목적으로 사용된다.

자바스크립트는 Error 생성자 함수를 포함해 7가지의 에러 객체를 생성할 수 있는 Error 생성자 함수를 제공한다.

| **생성자 함수** | **인스턴스**                                                              |
| --------------- | ------------------------------------------------------------------------- |
| Error           | 일반적 에러                                                               |
| SyntaxError     | 문법 에러                                                                 |
| ReferenceError  | 참조할 수 없는 식별자 참조했을 때 발생하는 에러                           |
| TypeError       | 피연산자 또는 인수의 데이터 타입이 유효하지 않을 때 발생하는 에러         |
| RangeError      | 숫자값의 허용 범위를 벗어났을 때 발생하는 에러                            |
| URIError        | encodeURI 또는 decodeURI 함수에 부적잘한 인수를 전달했을 때 발생하는 에러 |
| EvalError       | eval 함수에서 발생하는 에러                                               |
|                 |                                                                           |

# 4. throw 문

Error 생성자 함수로 에러 객체를 생성한다고 에러가 발생하는 것은 아니다.
즉, 에러 객체 생성과 에러 발생은 의미가 다르다.

```jsx
try {
  // 에러 객체를 생성한다고 에러가 발생하는 것은 아니다.
  new Error('something wrong');
} catch (error) {
  console.log(error);
}
```

에러를 발생시키려면 try 코드 블록에서 throw 문으로 에러 객체를 던져야 한다.

```jsx
try {
  // 에러 객체를 던지면 catch 코드 블록이 실행되기 시작한다.
  throw new Error('something wrong');
} catch (error) {
  console.log(error);
}
```

# 5. 에러의 전파

에러는 호출자(caller) 방향으로 전파된다. 즉, 콜 스택의 아래 방향으로 전파된다.

```jsx
const foo = () => {  
  throw Error('foo에서 발생한 에러');  
};  
  
const bar = () => {  
  foo();  
};  
  
const baz = () => {  
  bar();  
};  
  
try {  
  baz();  
} catch (err) {  
  console.error(err);  
}
```

![image1](https://github.com/kses1010/sunny-devlog/assets/49144662/66d6e480-71f0-4aeb-9c3d-8a2764125b8d)

foo -> bar -> baz -> 전역 실행 컨텍스트 방향으로 전파된다.
throw된 에러를 어디에서도 캐치하지 않으면 프로그램은 강제 종료된다.

주의할 것은 비동기 함수인 `setTimeout`이나 프로미스 후속 처리 메서드의 콜백 함수는 호출자가 없다는 것이다. `setTimeout`이나 프로미스 후속 처리 메서드의 콜백 함수는 태스크 큐나 마이크로태스크 큐에 일시 저장되었다가 콜 스택이 비면 이벤트 루프에 의해 콜 스택으로 푸시되어 실행된다.

이때 콜 스택에 푸시된 콜백 함수의 실행 컨텍스트는 콜 스택의 가장 하부에 존재하게 된다.
따라서 에러를 전파할 호출자가 존재하지 않는다.