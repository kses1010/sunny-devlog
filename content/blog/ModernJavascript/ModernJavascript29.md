---
title: 'Modern Javascript Deep Dive - 29장 Math'
date: 2023-08-14 16:52:01
category: 'Javascript'
draft: false
---

# 1. Math 프로퍼티

## 1.1 Math.PI

원주율 PI값(3.14)을 반환한다.

```jsx
Math.PI; // 3.141592
```

# 2. Math 메서드

```jsx
// Math.abs 메서드는 인수로 전달된 숫자의 절대값을 반환한다. 절대값은 반드시 0 또는 양수다.
Math.abs(-1); // 1
Math.abs('1'); // 1
Math.abs(''); // 0
Math.abs([]); // 0
Math.abs(null); // 0

// NaN으로 표기된다.
Math.abs(undefined);
Math.abs({});
Math.abs('string');
Math.abs();

// Math.Round 메서드는 인수로 전달된 숫자의 소수점 이하를 반올림한 정수를 반환한다.
Math.round(1.4); // 1
Math.round(1.6); // 2
Math.round(-1.4); // -1
Math.round(-1.6); // -2
Math.round(1); // 1
Math.round(); // NaN

// Math.ceil 메서드는 인수로 전달된 숫자의 소수점 이하를 올림한 정수를 반환한다.
Math.ceil(1.4); // 2
Math.ceil(1.6); // 2

// Math.floor 메서드는 인수로 전달된 숫자의 소수점 이하를 내림한 정수를 반환한다.
// 인수가 양수인 경우 소수점 이하를 떼어 버린 다음 정수를 반환한다.
Math.floor(1.4); // 1
Math.floor(2.6); // 2
// 인수가 음수인 경우 소수점 이하를 떼어 버린 다음 -1을 곱한 정수를 반환한다.
Math.floor(-1.9); // -2
Maht.floor(-9.1); // 10

// Math.sqrt 메서드는 인수로 전달된 숫자의 제곱근을 반환한다.
Math.sqrt(9); // 3
Math.sqrt(-9); // NaN
Math.sqrt(2); // 1.414213....
Math.sqrt(1); // 1
Math.sqrt(0); // 0
Math.sqrt(); // NaN
```

## 2.6 Math.random

Math.random 메서드는 임의의 난수(랜덤 숫자)를 반환한다. Math.random 메서드가 반환한 난수는 0에서 1미만의 실수다. → 0은 포함되지만 1은 포함되지 않는다.

```jsx
Math.random(); // 0 ~ 1 미만의 랜덤 실수

const random = Math.floor(Math.random() * 10 + 1);
console.log(random); // 1 ~ 10 범위의 정수
```

## 2.7 Math.pow

Math.pow 메서드는 첫 번째 인수를 밑으로, 두 번째 인수를 지수로 거듭제곱한 결과를 반환한다.

```jsx

Math.pow(2, 8); // 256
Math.pow(2 - 1); // 0.5
Math.pow(2); // NaN

// ES7에서 도입된 지수 연산자를 사용하면 가독성이 더 좋다.
2 ** (2 ** 2); // 16
Math.pow(Math.pow(2, 2), 2); // 16
```

## 2.8 Math.max // Math.min

Math.max 메서드는 전달받은 인수 중에서 가장 큰 수를 반환한다. 인수가 전달되지 않으면 -Infinity를 반환한다.

Math.min 메서드는 전달받은 인수 중에서 가장 작은 수를 반환한다. 인수가 전달되지 않으면 Infinity를 반환한다.

```jsx
Math.max(1); // 1
Math.max(1, 2); // 2
Math.max(1, 2, 3); // 3
Math.max(); // -Infinity

Math.min(1); // 1
Math.min(1, 2); // 1
Math.min(1, 2, 3); // 1
Math.min(); // Infinity
```

배열로 인수로 전달받아 배열의 요소 중에서 최대값/최소값을 구하려면 `Function.prototype.apply` 메서드나 스프레드 문법을 사용해야 한다.

```jsx
Math.max.apply(null, [1, 2, 3]); // 3
Math.max(...[1, 2, 3]); // 3

Math.min.apply(null, [1, 2, 3]); // 1
Math.min(...[[1, 2, 3]); // 1
```
