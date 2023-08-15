---
title: 'Modrn Javascript Deep Dive - 30장 Date'
date: 2023-08-15
category: 'Javascript'
draft: false
---

표준 객체인 Date는 날짜와 시간을 위한 메서드를 제공하는 빌트인 객체이면서 생성자 함수다.

UTC는 국제 표준시를 말하며 GMT로 불리기도 한다.

KST는 UTC에 9시간을 더한 시간이다.

현재 날짜와 시간은 자바스크립트 코드가 실행된 시스템의 시계에 의해 결정된다.

# 1. Date 생성자 함수

Date는 생성자 함수다. Date 생성자 함수로 생성한 Date 객체는 기본적으로 현재 날짜와 시간을 나타내는 정수값을 가진다.

현재 날짜와 시간이 아닌 다른 날짜와 시간을 다루고 싶은 경우 Date 생성자 함수에 명시적으로 해당 날짜와 시간 정보를 인수로 지정한다.

## 1.1 new Date()

Date 생성자 함수를 인수 없이 new 연산자와 함께 호출하면 현재 날짜와 시간을 가지는 Date 객체를 반환한다.

Date 객체는 내부적으로 날짜와 시간을 나타내는 정수값을 갖지만 Date 객체를 호출하면 기본적으로 날짜와 시간 정보를 출력한다.

```jsx
new Date() // 2023-08-15T12:50:28.906Z
```

Date 생성자 함수를 new 연산자 없이 호출하면 Date 객체를 반환하지 않고 날짜와 시간 정보를 나타내는 문자열을 반환한다.

```jsx
Date() // Tue Aug 15 2023 21:52:47 GMT+0900 (Korean Standard Time)
```

## 1.2 new Date(milliseconds)

Date 생성자 함수에 숫자 타입의 밀리초를 인수로 전달하면 UTC 기점으로 인수로 전달된 밀리초만큼 경과한 날짜와 시간을 나타내는 Date 객체를 반환한다.

```jsx
new Date(0) // 1970-01-01T00:00:00.000Z

new Date(86400000) // 1970-01-02T00:00:00.000Z
```

## 1.3 new Date(dateString)

Date 생성자 함수에 날짜와 시간을 나타내는 문자열을 인수로 전달하면 지정된 날짜와 시간을 Date 객체를 반환한다. 이때 인수로 전달한 문자열은 Date.parse 메서드에 의해 해석 가능한 형식이어야 한다.

```jsx
new Date('May 26, 2020 10:00:00') // 2020-05-26T01:00:00.000Z

new Date('2023/08/15/10:00:00') // 2023-08-15T01:00:00.000Z
```

## 1.4 new Date(year, month, day…)

Date 생성자 함수에 연, 월, 일, 시, 분, 초, 밀리초를 의미하는 숫자를 인수로 전달하면 지정된 날짜와 시간을 나타내는 Date 객체를 반환한다. 이때 연, 월은 반드시 지정해야 한다.

| 인수        | 내용                                                                   |
| ----------- | ---------------------------------------------------------------------- |
| year        | 연을 나타내는 1900년 이후의 정수. 0부터 99는 1900부터 1999로 처리된다. |
| month       | 월을 나타내는 0 ~ 11까지의 정수(주의: 0부터 시작, 0 = 1월)             |
| day         | 일을 나타내는 1 ~ 31까지의 정수                                        |
| hour        | 시를 나타내는 0 ~ 23까지의 정수                                        |
| minute      | 분을 나타내는 0 ~ 59까지의 정수                                        |
| second      | 초를 나타내는 0 ~ 59까지의 정수                                        |
| millisecond | 밀리초를 나타내는 0 ~ 999까지의 정수                                   |

```jsx
new Date(2020, 2) // 2020-02-29T15:00:00.000Z
new Date(2020, 2, 26, 10, 0, 0, 0) // 2020-03-26T01:00:00.000Z

// 가독성이 훨씬 좋다.
new Date('2020/2/26/10:00:00:00') // 2020-02-26T01:00:00.000Z
```

# 2. Date 메서드

## 2.1 Date.now

1970년 1월 1일 00:00:00(UTC)을 기점으로 현재 시간까지 경과한 밀리초를 숫자로 반환한다.

```jsx
Date.now() // 1692105562943
```

## 2.2 Date.parse

1970년 1월 1일 00:00:00(UTC)을 기점으로 인수로 전달된 지정시간(new Date(dateString))까지의 밀리초를 숫자로 반환한다.

```jsx
Date.parse('Jan 2, 1970 00:00:00 UTC') // 86400000
Date.parse('Jan 2, 1970 00:00:00') // 54000000
Date.parse('2023/08/16/10:00:00') // 1692147600000
```

## 2.3 Date.UTC

1970년 1월 1일 00:00:00(UTC)을 기점으로 인수로 전달된 지정 시간까지의 밀리초를 숫자로 반환한다.

Date.UTC 메서드는 new Date(year, month[, day…]와 같은 형식의 인수를 사용해야 한다.

```jsx
Date.UTC(1970, 0, 2) // 86400000
Date.UTC('2020/02/02') // NaN
```

## 2.4 Date 메서드들

```jsx
// Date.prototype.getFullYear
// Date 객체의 연도를 나타내는 정수를 반환한다.
new Date('2020/07/23').getFullYear() // 2020

// Date.prototype.setFullYear
// Date 객체의 연도를 나타내는 정수를 설정한다. 연도 이외에 옵션으로 월, 일도 설정할 수 있다.
const today = new Date()

// 년도 지정
today.setFullYear(2000)
today.getFullYear() // 2000

// 년도/월/일 지정
today.setFullYear(2023, 0, 2)
today.getFullYear() // 2023

// Date.prototype.getMonth
// Date 객체의 월을 나타내는 0 ~ 11의 정수를 반환한다. 1월은 0, 12월은 11이다.
new Date('2023/08/14').getMonth() // 6

// Date.prototype.setMonth
// Date 객체의 월을 나타내는 0 ~ 11의 정수를 설정한다.
const today = new Date()

// 월 지정
today.setMonth(0) // 1월
today.getMonth() // 0

// 월/일 지정
today.setMonth(11, 1) // 12월 1일
today.getMonth() // 11

// Date.prototype.getDate
// Date 객체의 날짜(1 ~ 31)를 나타내는 정수를 반환한다.
new Date('2023/08/15').getDate() // 24

// Date.prototype.setDate
// Date 객체의 날짜(1 ~ 31)를 나타내는 정수를 설정한다.
const today = new Date()

// 날짜 지정
today.setDate(1)
today.getDate() // 1

// Date.prototype.getDay
// Date 객체의 요일(0 ~ 6)을 나타내는 정수를 반환한다.
new Date('2023/08/15').getDay() // 2

// Date.prototype.getHours
// Date 객체의 시간(0 ~ 23)을 나타내는 정수를 반환한다.
new Date('2023/08/15/12:00').getHours() // 12

// Date.prototype.setHours
// Date 객체의 시간(0 ~ 23)을 나타내는 정수를 설정한다
// 시간 이외에 옵션으로 분, 초, 밀리초도 설정할 수 있다.
const today = new Date()

// 시간 지정
today.setHours(7)
today.getHours() // 0

// 시간/분/초/밀리초 지정
today.setHours(0, 0, 0, 0) // 00:00:00:00
today.getHours()

// Date.prototype.getMinutes
// Date 객체의 분(0 ~ 59)을 나타내는 정수를 반환한다.
new Date('2023/08/15/12:20').getMinutes() // 20

// Date.prototype.setMinutes
// Date 객체에 분(0 ~ 59)을 나타내는 정수를 설정한다.
// 분 이외에 옵션으로 초, 밀리초도 설정할 수 있다.
const today = new Date()

// 시간 지정
today.setMinutes(5)
today.getMinutes() // 5

// 시간/분/초/밀리초 지정
today.setMinutes(5, 10, 0) // HH:05:10:00
today.getMinutes() // 5

// Date.prototype.getSeconds
// Date 객체의 초(0 ~ 59)를 나타내는 정수를 반환한다.
new Date('2023/08/15/12:20:10').getSeconds() // 10

// Date.prototype.setSeconds
// Date 객체의 초(0 ~ 59)를 나타내는 정수를 설정한다.
const today = new Date()

// 시간 지정
today.setSeconds(5)
today.getSeconds() // 5

// 시간/분/초/밀리초 지정
today.setSeconds(5, 10) // HH:MM:05:10
today.getSeconds() // 5

// Date.prototype.getMilliseconds
// Date 객체의 밀리초(0 ~ 999)를 나타내는 정수를 반환한다.
new Date('2023/08/15/12:20:10:150').getMilliseconds() // 150
```

## 2.5 Date Time, Timezone 메서드들

```jsx
// Date.prototype.getTime
// 1970년 1월 1일 00:00:00(UTC)를 기점으로 Date 객체의 시간까지 경과된 밀리초를 반환한다.
new Date("2023/08/15/12:30").getTime(); // 1692070200000

// Date.prototype.setTime
// 1970년 1월 1일 00:00:00(UTC)를 기점으로 Date 객체의 시간까지 경과된 밀리초를 설정한다.
const today = new Date();

today.setTime(86400000); // 86400000은 1day를 의미

// Date.prototype.getTimezoneOffset
// UTC와 Date 객체에 지정된 locale 시간과의 차이를 분 단위로 반환한다.
// KST는 UTC에 9시간을 더한 시간이다.
const today = new Date();

today.getTimezoneOffset() / 60; // -9

// Date.prototype.toDateString
// 사람이 읽을 수 있는 문자열로 Date 객체의 날짜를 반환한다.
const today = new Date("2023/08/15/12:30");

today.toString(); // Tue Aug 15 2023 12:30:00 GMT+0900 (Korean Standard Time)
today.toDateString(); // Tue Aug 15 2023

// Date.prototype.toTimeString
// 사람이 읽을 수 있는 문자열로 Date 객체의 날짜를 반환한다.
const today = new Date("2023/08/15/12:30");

today.toTimeString(); // 12:30:00 GMT+0900 (Korean Standard Time)

// Date.prototype.toISOString
// ISO 8601 형식으로 Date 객체의 날짜와 시간을 표현한 문자열을 반환한다.
const today = new Date("2023/08/15/12:30");

today.toISOString(); // 2023-08-15T03:30:00.000Z
today.toISOString().slice(0, 10); // 2023-08-15
today.toISOString().slice(0, 10).replace(/-/g, ""); // 20230815

// Date.prototype.toLocaleString
// 인수로 전달한 로캘을 기준으로 Date 객체의 날짜와 시간을 문자열을 반환한다.
// 인수를 생략한 경우 브라우저가 동작 중인 시스템의 로캘을 적용한다.
const today = new Date("2023/08/15/12:30");

today.toLocaleString(); // 8/15/2023, 12:30:00 PM
today.toLocaleString("ko-KR"); // 2023. 8. 15. 오후 12:30:00
today.toLocaleString("en-US"); // 8/15/2023, 12:30:00 PM
today.toLocaleString("ja-JP"); // 2023/8/15 12:30:00

// Date.prototype.toLocaleTimeString
// 인수로 전달한 로캘을 기준으로 Date 객체의 시간을 문자열을 반환한다.
// 인수를 생략한 경우 브라우저가 동작 중인 시스템의 로캘을 적용한다.
const today = new Date("2023/08/15/12:30");

today.toLocaleTimeString()); // 12:30:00 PM
today.toLocaleTimeString("ko-KR"); // 오후 12:30:00
today.toLocaleTimeString("en-US"); // 12:30:00 PM
today.toLocaleTimeString("ja-JP"); // 12:30:00
```
