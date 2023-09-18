---
title: 'Modern Javascript Deep Dive - 8장 제어문'
date: 2023-07-31 23:06:59
category: 'Javascript'
draft: false
---

제어문은 조건에 따라 코드 블록을 실행(조건문)하거나 반복 실행(반복문)할 때 사용한다.

# 1. 블록문

블록문은 0개 이상의 문을 중괄호로 묶은 것으로, 코드 블록 또는 블록이라고 한다.

자바스크립트는 블록문을 하나의 실행 단위로 취급한다.

```jsx
// 블록문
{
  var foo = 10;
}

// 제어문
var x = 1;
if (x < 10) {
  x++;
}

// 함수 선언문
function sum(a, b) {
  return a + b;
}
```

# 2. 조건문

조건문은 주어진 조건식의 평가 결과에 따라 코드 블록의 실행을 결정한다.

조건식은 불리언 값으로 평가될 수 있는 표현식이다.

## 2.1 if… else 문

`if...else` 문은 주어진 조건식의 평가 결과, 즉 논리적 참 또는 거짓에 따라 실행할 코드 블록을 결정한다.

```jsx
if (조건식) {
  // 조건식이 참이면 실행
} else {
  // 조건식이 거짓이면 실행
}
```

if 문의 조건식은 불리언 값으로 평가되어야 한다. 만약 if 문의 조건식이 불리언 값이 아닌 값으로 평가되면 자바스크립트 엔진에 의해 암묵적으로 불리언 값으로 강제 변환되어 실행할 코드 블록을 결정한다.

조건식을 더 추가하고 싶으면 `else if` 문을 사용한다.

```jsx
var num = 2;
var kind;

// if 문
if (num > 0) {
  kind = '양수'; // 음수를 구별할 수 없다.
}
console.log(kind); // 양수

// if...else 문
if (num > 0) {
  kind = '양수';
} else {
  kind = '음수'; // 0은 음수가 아님
}
console.log(kind); // 양수

// if...else 문
if (num > 0) {
  kind = '양수';
} else if (num < 0) {
  kind = '음수';
} else {
  kind = '영';
}
console.log(kind); // 양수
```

대부분의 `if…else` 문은 삼항 조건 연산자로 바꿔 쓸 수 있다.

## 2.2 switch 문

`switch` 문은 주어진 표현식을 평가하여 그 값과 일치하는 표현식을 갖는 case 문으로 실행 흐름을 옮긴다.

case 문은 상황을 의미하는 표현식을 지정하고 콜론으로 마친다.

```jsx
switch (표현식) {
	case 표현식1:
		switch 문의 표현식과 표현식 1이 일치하면 실행될 문;
		break;
	case 표현식2:
		switch 문의 표현식과 표현식 2이 일치하면 실행될 문;
		break;
	default:
		switch 문의 표현식과 일치하는 case 문이 없을 때 실행하는 문;
}
```

switch 문의 표현식은 불리언 값보다는 문자열이나 숫자 값인 경우가 많다.

break 문을 생략하면 폴스루(fall through)가 발생하는데 이 경우에는 break 문을 만날때까지 모든 케이스를 돌게 된다.

다음 예제는 폴스루가 유용한 경우다.

```jsx
var year = 2000
var month = 2
var day = 0

switch (month) {
  case 1:
  case 3:
  case 5:
  case 7:
  case 8:
  case 10:
  case 12:
    days = 31;
    break;
  case 4:
  case 6:
  case 9:
  case 11:
    days = 30;
    break;
  case 2:
    days = (year % 4 === 0 && year & (100 !== 0)) || year % 400 === 0 ? 29 : 28;
    break;
  default:
    console.log('Invalid month');
}

console.log(days);
```

# 3. 반복문

반복문은 조건식의 평가 결과가 참인 경우 코드 블록을 실행한다.

자바스크립트는 세가지 반복문인 for 문, while 문, do… while 문을 제공한다.

## 3.1 for 문

`for` 문은 조건식이 거짓으로 평가될 때까지 코드 블록을 반복 실행한다.

```jsx
for (변수 선언문 또는 할당문; 조건식; 증감식) {
	조건식이 참일경우 반복 실행될 문
}

for (var i = 0; i < 2; i++) {
	console.log(i); // 1, 2
}
```

for 문의 변수 선언문, 조건식, 증감식은 모두 옵션이므로 반드시 사용할 필요는 없다. 단, 어떤 식도 선언하지 않으면 무한루프가 된다.

```jsx
for (;;) {...}
```

## 3.2 while 문

`while` 문은 주어진 조건식의 평가 결과가 참이면 코드 블록을 계속해서 반복 실행한다.

for 문은 반복 횟수가 명확할 때, while 문은 반복 횟수가 불명확할 때 주로 사용한다.

```jsx
var count = 0;

// count가 3보다 작을 때까지 코드 블록 반복 수행
while (count < 3) {
  console.log(count); //0 1 2
  count++;
}
```

조건식의 평가 결과가 언제나 참이면 무한루프가 된다.

```jsx
while (true) {...}
```

또한 무한루프더라도 break 문으로 코드 블록을 탈출할 수 있다.

```jsx
var count = 0;
while (true) {
  console.log(count);
  count++;

  if (count === 3) break;
} // 0 1 2
```

## 3.3 do…while 문

`do…while` 문은 코드 블록을 먼저 실행하고 조건식을 평가한다.

→ 코드 블록은 무조건 한 번 이상 실행된다.

```jsx
var count = 0;

do {
  console.log(count);
  count++;
} while (count < 3)
// 0 1 2
```

# 4. break 문

`break` 문은 코드 블록을 탈출한다. 레이블 문, 반복문, `switch` 문을 탈출할 때 사용되며 그 외에 블록에 사용될 경우 `SyntaxError` 가 발생한다.

```jsx
if (true) {
  break; // SyntaxError
}

// foo라는 식별자가 붙은 레이블 블록문
foo: {
  console.log(1);
  break foo // foo 레이블 블록문 탈출
  console.log(2);
}

console.log('Done!');

// 1
// Done!
```

중첩된 문의 내부 for 문에서 break 문을 실행하면 내부 for 문을 탈출하여 외부 for 문으로 진입한다.

만약 내부 for 문이 아닌 외부 for 문을 탈출하려면 레이블 문을 사용한다.

```jsx
outer: for (var i = 0; i < 3; i++) {
  for (var j = 0; j < 3; j++) {
    // i + j === 3 이면 outer라는 식별자가 붙은 레이블 for 문을 탈출한다.
    if (i + j === 3) break outer
    console.log(`inner [${i}, ${j}]`);
  }
}
console.log('Done!');

// inner [0, 0]
// inner [0, 1]
// inner [0, 2]
// inner [1, 0]
// inner [1, 1]
// Done!
```

# 5. continue 문

`continue` 문은 반복문의 코드 블록 실행을 현 지점에서 중단하고 반복문의 증감식으로 실행 흐름을 이동시킨다.

`break` 문처럼 반복문을 탈출하지는 않는다.

```jsx
var string = 'Hello World.';
var search = 'l';
var count = 0;

for (var i = 0; i < string.length; i++) {
  if (string[i] !== search) continue;
  count++; // continue 문이 실행되면 이 문은 실행하지 않음.
}

console.log(count); // 3
```
