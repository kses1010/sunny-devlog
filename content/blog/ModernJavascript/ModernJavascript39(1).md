---
title: 'Modern Javascript Deep Dive - 39장 DOM(1)'
date: 2023-08-26 16:52:06
category: 'Javascript'
draft: false
---

**DOM(Document Object Model)은 HTML 문서의 계층적 구조와 정보를 표현하며 이를 제어할 수 있는 API, 즉 프로퍼티와 메서드를 제공하는 트리 자료구조다.**

# 1. 노드

## 1.1 HTML 요소와 노드 객체

HTML 요소는 HTML 문서를 구성하는 개별적인 요소를 의미한다.

HTML 요소는 렌더링 엔진에 의해 파싱되어 DOM을 구성하는 요소 노드 객체로 변환된다.

이때 HTML 요소의 어트리뷰트는 어트리뷰트 노드로, HTML 요소의 텍스트 콘텐츠는 텍스트 노드로 변환된다.

HTML 요소 간의 부자 관계를 반영하여 HTML 문서의 구성 요소인 HTML 요소를 객체화한 모든 노드 객체들을 트리 자료구조로 구성된다.

## 1.2 노드 객체의 타입

```html
<!DOCTYPE html>
<html lang="kr">
  <head>
    <meta charset="UTF-8" />
    <link rel="stylesheet" href="style.css" />
    <title>Title</title>
  </head>
  <body>
    <ul>
      <li id="apple">Apple</li>
      <li id="banana">Banana</li>
      <li id="orange">Orange</li>
    </ul>
    <script src="app.js"></script>
  </body>
</html>
```

DOM은 노드 객체의 계층적인 구조로 구성된다. 노드 객체는 종류가 있고 상속 구조를 갖는다.

### 문서 노드

문서 노드는 DOM 트리의 최상위에 존재하는 루트 노드로서 document 객체를 가리킨다.

문서 노드, 즉 document 객체는 DOM 트리의 루트 노드이므로 DOM 트리의 노드들에 접근하기 위해 진입점 역할을 담당한다.

→ 요소, 어트리뷰트, 텍스트 노드에 접근하려면 문서 노드를 통해야 한다.

### 요소 노드

요소 노드는 HTML 요소를 가리키는 객체다. 요소 노드는 문서의 구조를 표현한다고 할 수 있다.

### 어트리뷰트 노드

어트리뷰트 노드는 HTML 요소의 어트리뷰트를 가리키는 객체다. 어트리뷰트 노드는 어트리뷰트가 지정된 HTML 요소의 요소 노드와 형제 관계를 갖는다.

단, 요소 노드는 부모 노드와 연결되어 있지만 어트리뷰트 노드는 부모 노드와 연결되어 있지 않고 형제 노드인 요소 노드에만 연결되어 있다.

→ 어트리뷰트 노드에 접근하여 어트리뷰트를 참조하거나 변경하려면 먼저 형제 노드인 요소 노드에 접근해야 한다.

### 텍스트 노드

텍스트 노드는 HTML 요소의 텍스트를 가리키는 객체다. 텍스트 노드는 요소 노드의 자식 노드이며, 자식 노드를 가질 수 없는 리프 노드다.

→ 텍스트 노드는 DOM 트리의 최종단이다.

## 1.3 노드 객체의 상속 구조

DOM을 구성하는 노드 객체는 자신의 구조와 정보를 제어할 수 있는 DOM API를 사용할 수 있다.

노드 객체는 자신의 부모, 형제, 자식을 탐색할 수 있으며, 자신의 어트리뷰트와 텍스트를 조작할 수 있다.

**DOM은 HTML 문서의 계층적 구조와 정보를 표현하는 것은 물론 노드 객체의 종류, 즉 노드 타입에 따라 필요한 기능을 프로퍼티와 메서드의 집합인 DOM API로 제공한다. 이 DOM API를 통해 HTML의 구조나 내용 또는 스타일 등을 동적으로 조작할 수 있다.**

# 2. 요소 노드 취득

요소 노드의 취득은 HTML 요소를 조작하는 시작점이다.

## 2.1 id를 이용한 요소 노드 취득

`Document.prototype.getElementById` 메서드는 인수로 전달한 id 어트리뷰트 값을 갖는 하나의 요소 노드를 탐색하여 반환한다.

→ 반드시 문서 노드인 document를 통해 호출해야 한다.

```html
<!DOCTYPE html>
<html lang="kr">
  <head>
    <meta charset="UTF-8" />
    <link rel="stylesheet" href="style.css" />
    <title>Title</title>
  </head>
  <body>
    <ul>
      <li id="apple">Apple</li>
      <li id="banana">Banana</li>
      <li id="orange">Orange</li>
    </ul>
    <script>
      // id 값이 banana 인 요소 노드를 탐색하여 반환
      // 두 번째 li 요소가 파싱되어 생성된 요소 노드가 반환
      const $elem = document.getElementById('banana');

      // 취득한 요소 노드의 style.color 프로퍼티 값을 변경한다.
      $elem.style.color = 'red';
    </script>
  </body>
</html>
```

id 값은 HTML 문서 내에서 유일한 값이어야 하며, class 어트리뷰트와는 달리 공백 문자로 구분하여 여러 개의 값을 가질 수 없다.

단, HTML 문서 내에 중복된 id 값을 갖는 HTML 요소가 여러 개 존재하더라도 어떠한 에러도 발생하지 않는다.

언제나 단 하나의 요소 노드를 반환한다.

```html
<body>
  <ul>
    <li id="banana">Apple</li>
    <li id="banana">Banana</li>
    <li id="banana">Orange</li>
  </ul>
  <script>
    // getElementById 메서드는 언제나 단 하나의 요소 노드를 반환.
    // 첫 번째 li 요소가 파싱되어 생성된 요소 노드가 반환
    const $elem = document.getElementById('banana');

    // 취득한 요소 노드의 style.color 프로퍼티 값을 변경한다.
    $elem.style.color = 'red';
  </script>
</body>
```

만약 인수로 전달된 id 값을 갖는 HTML 요소가 존재하지 않는 경우 `getElementId` 메서드는 null을 반환한다.

```html
<body>
  <ul>
    <li id="apple">Apple</li>
    <li id="banana">Banana</li>
    <li id="orange">Orange</li>
  </ul>
  <script>
    // id가 grape 인 요소 노드를 탐색하여 반환. null이 반환된다.
    const $elem = document.getElementById('grape');

    // 취득한 요소 노드의 style.color 프로퍼티 값을 변경한다.
    $elem.style.color = 'red'; // TypeError
  </script>
</body>
```

HTML 요소에 id 어트리뷰트를 부여하면 id 값과 동일한 이름의 전역 변수가 암묵적으로 선언되고 해당 노드 객체가 할당되는 부수 효과가 있다.

```html
<body>
  <div id="foo"></div>
  <script>
    // id 값과 동일한 이름의 전역 변수가 암묵적으로 선언되고 해당 노드 객체가 할당.
    console.log(foo === document.getElementById('foo')); // true

    // 암묵적 전역으로 생성된 전역 프로퍼티는 삭제되지만 전역 변수는 삭제되지 않는다.
    delete foo;
    console.log(foo); // <div id="foo"></div>
  </script>
</body>
```

단, id 값과 동일한 이름의 전역 변수가 이미 선언되어 있으면 이 전역 변수에 노드 객체가 재할당되지 않는다.

```html
<body>
  <div id="foo"></div>
  <script>
    let foo = 1;

    // id 값과 동일한 이름의 전역 변수가 이미 선언되어 있으면 노드 객체가 재할당되지 않는다.
    console.log(foo);
  </script>
</body>
```

## 2.2 태그 이름을 이용한 요소 노드 취득

`Document.prototype/Element.prototype.getElementsByTagName` 메서드는 인수로 전달한 태그 이름을 갖는 모든 요소 노드들을 탐색하여 반환한다.

`getElementsByTagName` 메서드는 여러 개의 요소 노드 객체를 갖는 DOM 컬렉션 객체인 HTMLCollection 객체를 반환한다.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
  </head>
  <body>
    <ul>
      <li id="apple">Apple</li>
      <li id="banana">Banana</li>
      <li id="orange">Orange</li>
    </ul>
    <script>
      // HTMLCollection 객체는 유사 배열 객체이면서 이터러블이다.
      const $elem = document.getElementsByTagName('li');

      // HTMLCollection 객체를 배열로 변환하여 순회하면서 color 프로퍼티 값을 변경한다.
      [...$elem].forEach(elem => {
        elem.style.color = 'red';
      });
    </script>
  </body>
</html>
```

`getElementsByTagName` 메서드가 반환하는 DOM 컬렉션 객체인 HTMLCollection 객체는 유사 배열 객체이면서 이터러블이다.

HTML 문서의 모든 요소 노드를 취득하려면 `getElementsByTagName` 메서드의 인수로 `*` 를 전달한다.

```jsx
const $all = document.getElementsByIdTagName('*');
```

Document의 `getElementsByTagName` 메서드는 DOM의 루트 노드인 문서 노드, 즉 document를 통해 호출하며 DOM 전체에서 요소 노드를 탐색하여 반환한다.

Element의 `getElementsByTagName` 메서드는 특정 요소 노드를 통해 호출하며, 특정 요소 노드의 자손 노드 중에서 요소 노드를 탐색하여 반환한다.

```html
<!DOCTYPE html>
<html lang="kr">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
  </head>
  <body>
    <ul id="fruits">
      <li>Apple</li>
      <li>Banana</li>
      <li>Orange</li>
    </ul>
    <ul>
      <li>HTML</li>
    </ul>
    <script>
      const $lisFromDocument = document.getElementsByTagName('li');
      console.log($lisFromDocument); // HTMLCollection(4) [li, li, li, li]

      // ul#fruits 요소의 자손 노드 중에서 태그 이름이 li인 요소 노드를 모두 탐색하여 반환
      const $fruits = document.getElementById('fruits');
      const $lisFromFruits = $fruits.getElementsByTagName('li');
      console.log($lisFromDocument); // HTMLCollection(3) [li, li, li]
    </script>
  </body>
</html>
```

만약 인수로 전달된 태그 이름을 갖는 요소가 존재하지 않는 경우 `getElementsByTagName` 메서드는 빈 HTMLCollection 객체를 반환한다.

## 2.3 class를 이용한 요소 노드 취득

`Doclument.prototype/Element.prototype.getElementsByClassName` 메서드는 인수로 전달한 class 어트리뷰트 값을 갖는 모든 요소 노드들을 탐색하여 반환한다.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
  </head>
  <body>
    <ul>
      <li class="fruit apple">Apple</li>
      <li class="fruit banana">Banana</li>
      <li class="fruit orange">Orange</li>
    </ul>
    <script>
      // class 값이 fruit 인 요소 노드를 모두 탐색하여 HTMLCollection 객체를 담아 반환
      const $elem = document.getElementsByClassName('fruit');

      // 취득한 모든 요소의 CSS color 프로퍼티 값을 변경.
      [...$elem].forEach(elem => {
        elem.style.color = 'red';
      });

      // class 값이 fruit apple 인 요소 노드를 모두 탐색하여 HTMLCollection 객체를 담아 반환
      const $apples = document.getElementsByClassName('fruit apple');

      [...$apples].forEach(elem => {
        elem.style.color = 'blue';
      });
    </script>
  </body>
</html>
```

Document의 `getElementsByClassName` 메서드는 DOM의 루트 노드인 문서 노드, 즉 document를 통해 호출하며 DOM 전체에서 요소 노드를 탐색하여 반환한다.

Element의 `getElementsByClassName` 메서드는 특정 요소 노드를 통해 호출하며, 특정 요소 노드의 자손 노드 중에서 요소 노드를 탐색하여 반환한다.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
  </head>
  <body>
    <ul id="fruits">
      <li class="apple">Apple</li>
      <li class="banana">Banana</li>
      <li class="orange">Orange</li>
    </ul>
    <div class="banana">Banana</div>
    <script>
      // DOM 전체에서 class 값이 banana 인 요소 노드를 모두 탐색하여 반환
      const $bananasFromDocument = document.getElementsByClassName('banana');
      console.log($bananasFromDocument);
      // HTMLCollection(2) [li.banana, div.banana]

      // #fruits 요소의 자손 노드 중에서 class 값이 banana 인 요소 노드를 모두 탐색하여 반환
      const $fruits = document.getElementById('fruits');
      const $bananaFromFruits = $fruits.getElementsByClassName('banana');

      console.log($bananaFromFruits); // HTMLCollection [li.banana]
    </script>
  </body>
</html>
```

만약 인수로 전달된 태그 이름을 갖는 요소가 존재하지 않는 경우 `getElementsByClassName` 메서드는 빈 HTMLCollection 객체를 반환한다.

## 2.4 CSS 선택자를 이용한 요소 노드 취득

CSS 선택자는 스타일을 적용하고자 하는 HTML 요소를 특정할 때 사용하는 문법이다.

```css
/* 전체 선택자 */
* {
  ...;
}
/* 태그 선택자 */
p {
  ...;
}
/* id 선택자 */
#foo {
  ...;
}
/* 등등 */
```

`Document.prototype/Element.prototype.querySelector` 메서드는 인수로 전달한 CSS 선택자를 만족시키는 하나의 요소 노드를 탐색하여 반환한다.

- 인수로 전달한 CSS 선택자를 만족시키는 요소 노드가 여러 개인 경우 첫번째 요소 노드만 반환
- 인수로 전달한 CSS 선택자를 만족시키는 요소 노드가 존재하지 않는 경우 null 반환
- 인수로 전달한 CSS 선택자가 문법에 맞지 않는 경우 DOMException 에러가 발생

```html
<!DOCTYPE html>
<html lang="kr">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
  </head>
  <body>
    <ul>
      <li class="apple">Apple</li>
      <li class="banana">Banana</li>
      <li class="orange">Orange</li>
    </ul>
    <script>
      // class 어트리뷰트 값이 banana 인 첫 번째 요소 노드를 탐색하여 반환
      const $elem = document.querySelector('.banana');

      // 취득한 요소 노드의 style.color 프로퍼티 값을 변경
      $elem.style.color = 'red';
    </script>
  </body>
</html>
```

`Document.prototype/Element.prototype.querySelectorAll` 메서드는 인수로 전달한 CSS 선택자를 만족시키는 모든 요소 노드를 탐색하여 반환한다.

`querySelectorAll` 메서드는 여러 개의 요소 노드 객체를 갖는 DOM 컬렉션 객체인 NodeList 객체를 반환한다. NodeList 객체는 유사 배열 객체이면서 이터러블이다.

- 인수로 전달한 CSS 선택자를 만족시키는 요소 노드가 존재하지 않는 경우 null 반환
- 인수로 전달한 CSS 선택자가 문법에 맞지 않는 경우 DOMException 에러가 발생

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
  </head>
  <body>
    <ul>
      <li class="apple">Apple</li>
      <li class="banana">Banana</li>
      <li class="orange">Orange</li>
    </ul>
    <script>
      // ul 요소의 자식 요소인 li 요소를 모두 탐색하여 반환
      const $elem = document.querySelectorAll('ul > li');
      // 취득한 요소 노드들은 NodeList 객체에 담겨 반환
      console.log($elem); // NodeList(3) [li.apple, li.banana, li.orange]

      // 취득한 모든 요소 노드의 style.color 프로퍼티 값을 변경
      $elem.forEach(elem => {
        elem.style.color = 'red';
      });
    </script>
  </body>
</html>
```

HTML 문서의 모든 요소 노드를 취득하려면 `querySelectorAll` 메서드의 인수로 전체 선택자 `*` 을 전달한다.

```jsx
const $all = document.querySelectorAll('*');
```

CSS 선택자 문법을 사용하는 querySelector, querySelectorAll 메서드는 `getElementById`, `getElementsBy***` 메서드보다 다소 느린것으로 알려져 있다.

하지만 CSS 선택자 문법을 사용하여 좀 더 구체적인 조건으로 요소 노드를 취득할 수 있고 일관된 방식으로 요소 노드를 취득할 수 있다는 장점이 있다.

→ id 어트리뷰트가 있는 요소 노드를 취득하는 경우에는 `getElementById` 메서드를 사용하고 그 외의 경우에는 `querySelector`, `querySelectorAll` 메서드를 사용하는 것이 권장된다.

## 2.5 특정 요소 노드를 취득할 수 있는 지 확인

`Element.prototype.matches` 메서드는 인수로 전달한 CSS 선택자를 통해 특정 요소 노드를 취득할 수 있는지 확인한다.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
  </head>
  <body>
    <ul id="fruits">
      <li class="apple">Apple</li>
      <li class="banana">Banana</li>
      <li class="orange">Orange</li>
    </ul>
    <script>
      const $apple = document.querySelector('.apple');

      // $apple 노드는 #fruits > li.apple 로 취득할 수 있다.
      console.log($apple.matches('#fruits > li.apple')); // true

      // $apple 노드는 #fruits > li.banana 로 취득할 수 없다.
      console.log($apple.matches('#fruits > li.banana')); // false
    </script>
  </body>
</html>
```

`Element.prototype.matches` 메서드는 이벤트 위임을 사용할 때 유용하다.

## 2.6 HTMLCollection과 NodeList

DOM 컬렉션 객체인 HTMLCollection과 NodeList는 DOM API가 여러 개의 결과값을 반환하기 위한 DOM 컬렉션 객체다.

HTMLCollection과 NodeList는 모두 유사 배열 객체이며 이터러블이다.

→ `for...of` 문이나 스프레드 문법을 사용하여 간단히 배열로 변환할 수 있다.

HTMLCollection과 NodeList의 중요한 특성은 노드 객체의 상태 변화를 실시간으로 반영하는 살아 있는 객체라는 것이다.

- HTMLCollection: 언제나 live 객체로 동작
- NodeList: 대부분의 경우 노드 객체의 상태 변화를 실시간으로 반영하지 않고 과거의 정적 상태를 유지하는 non-live 객체로 동작하지만 경우에 따라 live 객체로 동작할 때가 있다.

### HTMLCollection

`getElementsByTagName`, `getElementsByClassName` 메서드가 반환하는 HTMLCollection 객체는 노드 객체의 상태 변화를 실시간으로 반영하는 살아 있는 DOM 컬렉션 객체다.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <style>
      .red {
        color: red;
      }
      .blue {
        color: blue;
      }
    </style>
  </head>
  <body>
    <ul id="fruits">
      <li class="red">Apple</li>
      <li class="red">Banana</li>
      <li class="red">Orange</li>
    </ul>
    <script>
      // class 값이 red 인 요소 노드를 모두 탐색하여 HTMLCollection 객체에 담아 반환
      const $elems = document.getElementsByClassName('red');
      // 이 시점에 HTMLCollection 객체에는 3개의 요소 노드가 담겨 있다.
      console.log($elems) // HTMLCollection(3) [li.red, li.red, li.red]

      // HTMLCollection 객체의 모든 요소의 class 값을 blue 로 변경
      for (let i = 0; i < $elems.length; i++) {
        $elems[i].className = 'blue';
      }

      // HTMLCollection 객체의 요소가 3개에서 1개로 변경
      console.log($elems); // HTMLCollection(1) [li.red]
    </script>
  </body>
</html>
```

![image](https://github.com/kses1010/sunny-devlog/assets/49144662/c8a9f220-09af-4874-89bc-ddd25ec57ae6)

그러나, 두 번째 `li` 요소만 class 값이 변경되지 않는다.

HTMLCollection 객체는 실시간으로 노드 객체의 상태 변경을 반영하여 요소를 제거할 수 있기 때문에 HTMLCollection 객체를 for 문으로 순회하면서 노드 객체의 상태를 변경해야 할 때 주의해야 한다.

```jsx
// for 문을 역방향으로 순회
for (let i = $elems.length - 1; i >= 0; i--) {
  $elems[i].className = 'blue';
}

// while문으로 HTMLCollection에 요소가 남아 있지 않을 때까지 무한 반복
let i = 0;
while ($elems.length > i) {
  $elems[i].className = 'blue';
}
```

더 간단한 해결책은 부작용을 발생시키는 원인인 HTMLCollection 객체를 사용하지 않는 것이다.

유사 배열 객체이면서 이터러블인 HTMLCollection 객체를 변환하면 부작용을 발생시키지 않고 고차 함수를 사용할 수 있다.

```jsx
[...$elems].forEach(elem => (elem.className = 'blue'));
```

### NodeList

`querySelectorAll` 메서드는 DOM 컬렉션 객체인 NodeList 객체를 반환한다.

이때 NodeList 객체는 실시간으로 노드 객체의 상태 변경을 반영하지 않는 객체다.

```jsx
const $elems = document.querySelectorAll('.red');

$elems.forEach(elem => (elem.className = 'blue'));
```

NodeList 객체는 대부분의 경우 노드 객체의 상태 변경을 실시간으로 반영하지 않고 과거의 정적 상태를 유지하는 non-live 객체로 동작한다.

**하지만 childNodes 프로퍼티가 반환하는 NodeList 객체는 HTMLCollection 객체와 같이 실시간으로 노드 객체의 상태 변경을 반영하는 live 객체로 동작하므로 주의가 필요하다.**

```html
<!DOCTYPE html>
<html lang="en">
  <head> </head>
  <body>
    <ul id="fruits">
      <li>Apple</li>
      <li>Banana</li>
      <li>Orange</li>
    </ul>
    <script>
      const $fruits = document.getElementById('fruits');

      // childNodes 프로퍼티는 NodeList 객체(live)를 반환한다.
      const { childNodes } = $fruits;
      console.log(childNodes instanceof NodeList); // true

      // $fruits 요소의 자식 노드는 공백 텍스트 노드를 포함해 모두 5개다.
      console.log(childNodes); // NodeList(5) [text, li, text, li, text]

      for (let i = 0; i < childNodes.length; i++) {
        // removeChild 메서드는 $fruits 요소의 자식 노드를 DOM 에서 삭제
        // removeChild 메서드는 호출할 때마다 NodeList 객체인 childNodes 실시간으로 변경
        // 첫 번째, 세 번째, 다섯 번째 요소만 삭제된다.
        $fruits.removeChild(childNodes[i]);
      }

      // $fruits 요소의 모든 자식 노드가 삭제되지 않는다.
      console.log(childNodes); // NodeList(2) [li, li]
    </script>
  </body>
</html>
```

**노드 객체의 상태 변경과 상관없이 안전하게 DOM 컬렉션을 사용하려면 HTMLCollection이나 NodeList 객체를 배열로 변환하여 사용하는 것을 권장한다.**

```html
<!DOCTYPE html>
<html lang="en">
  <head> </head>
  <body>
    <ul id="fruits">
      <li>Apple</li>
      <li>Banana</li>
      <li>Orange</li>
    </ul>
    <script>
      const $fruits = document.getElementById('fruits');

      // childNodes 프로퍼티는 NodeList 객체(live)를 반환한다.
      const { childNodes } = $fruits

      // 스프레드 문법을 ㅅ가용하여 NodeList 객체를 배열로 반환
      [...childNodes].forEach(node => {
        $fruits.removeChild(node);
      });

      // $fruits 요소의 모든 자식 노드가 삭제
      console.log(childNodes); // NodeList []
    </script>
  </body>
</html>
```
