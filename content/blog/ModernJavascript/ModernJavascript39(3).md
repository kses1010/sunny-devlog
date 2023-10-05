---
title: 'Modern Javascript Deep Dive - 39장 DOM(3)'
date: 2023-08-29 13:53:50
category: 'Javascript'
draft: false
---

# 7. 어트리뷰트

## 7.1 어트리뷰트 노드와 attributes 프로퍼티

HTML 문서의 구성 요소인 HTML 요소는 여러 개의 어트리뷰트를 가질 수 있다. HTML 요소의 동작을 제어하기 위한 추가적인 정보를 제공하는 HTML 어트리뷰트는 HTML 요소의 시작 태그에
`어트리뷰트 이름="어트리뷰트 값"` 형식으로 정의한다.

```html
<input id="user" type="text" value="Sunny" />
```

요소 노드의 모든 어트리뷰트는 요소 노드의 `Element.prototype.attributes` 프로퍼티로 취득 할 수 있다.

attributes 프로퍼티는 getter만 존재하는 읽기 전용 접근자 프로퍼티이며, 요소 노드의 모든 어트리뷰트 노드의 참조가 담긴 NamedNodeMap 객체를 반환한다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <input id="user" type="text" value="sunny" />
    <script>
      // 요소 노드의 attribute 프로퍼티는 요소 노드의 모든 어트리뷰트 노드의 참조가 담긴
      // NamedNodeMap 객체를 반환한다.
      const { attributes } = document.getElementById('user');
      console.log(attributes);

      // 어트리뷰트 값 취득
      console.log(attributes.id.value); // user
      console.log(attributes.type.value); // text
      console.log(attributes.value.value); // sunny
    </script>
  </body>
</html>
```

## 7.2 HTML 어트리뷰트 조작

`Element.prototype.getAttribute/setAttribute` 메서드를 사용하면 attributes 프로퍼티를 통하지 않고 요소 노드에서 메서드를 통해 직접 HTML 어트리뷰트 값을 취득하거나 변경할 수 있어서 편리하다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <input id="user" type="text" value="sunny" />
    <script>
      const $input = document.getElementById('user');

      // value 어트리뷰트 값을 취득
      const inputValue = $input.getAttribute('value');
      console.log(inputValue); // sunny

      // value 어트리뷰트 값을 변경
      $input.setAttribute('value', 'foo');
      console.log($input.getAttribute('value')); // foo
    </script>
  </body>
</html>
```

특정 HTML어트리뷰트가 존재한는지 확인하려면 `Element.prototype.hasAttribute(attributeName)` 메서드를 사용한다.

특정 HTML 어트리뷰트를 삭제하려면 `Element.prototype.removeAttribute(attributeName)` 메서드를 사용한다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <input id="user" type="text" value="sunny" />
    <script>
      const $input = document.getElementById('user');

      // value 어트리뷰트의 존재 확인
      if ($input.hasAttribute('value')) {
        // value 어트리뷰트 삭제
        $input.removeAttribute('value');
      }

      // value 어트리뷰트가 삭제되었다.
      console.log($input.hasAttribute('value')); // false
    </script>
  </body>
</html>
```

## 7.3 HTML 어트리뷰트 vs DOM 프로퍼티

요소 노드 객체에는 HTML 어트리뷰트에 대응하는 프로퍼티(DOM 프로퍼티)가 존재한다.

이 DOM 프로퍼티들은 HTML 어트리뷰트 값을 초기값으로 가지고 있다.

DOM 프로퍼티는 setter, getter 모두 존재하는 접근자 프로퍼티다. 참조와 변경이 가능하다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <input id="user" type="text" value="sunny" />
    <script>
      const $input = document.getElementById('user');

      // 요소 노드의 value 프로퍼티 값을 변경
      $input.value = 'foo';

      // 요소 노드의 프로퍼티값을 참조
      console.log($input.value); // foo
    </script>
  </body>
</html>
```

HTML 어트리뷰트는 DOM에서 중복 관리되는것처럼 보이지만 그렇지 않다.

**HTML 어트리뷰트의 역할은 HTML 요소의 초기 상태를 지정하는 것이다. 즉, HTML 어트리뷰트 값은 HTML 요소의 초기 상태를 의미하며 이는 변하지 않는다.**

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <input id="user" type="text" value="sunny" />
    <script>
      const $input = document.getElementById('user');

      // attributes 프로퍼티에 저장된 value 어트리뷰트 값
      console.log($input.getAttribute('value')); // sunny

      // 요소 노드의 value 프로퍼티에 저장된 value 어트리뷰트 값
      console.log($input.value); // sunny
    </script>
  </body>
</html>
```

현재는 동일하지만 첫 렌더링 이후 사용자가 input 요소에 무언가를 입력하기 시작하면 상황이 달라진다.

**요소 노드는 상태를 가지고 있다.**

사용자가 input 요소의 입력 필드에 “foo”라는 값을 입력한 경우, 최신 상태(”foo”)를 관리해야하는 물론, HTML 어트리뷰트로 지정한 초기상태(”sunny”)도 관리해야 한다.

**요소 노드는 2개의 상태, 즉 초기 상태와 최신 상태를 관리해야 한다. 요소 노드의 초기 상태는 어트리뷰트 노드가 관리하며, 요소 노드의 최신 상태는 DOM 프로퍼티가 관리한다.**

### 어트리뷰트 노드

**HTML 어트리뷰트로 지정한 HTML 요소의 초기 상태는 어트리뷰트 노드에서 관리한다.**

HTML 요소에 지정한 어트리뷰트 값은 사용자의 입력에 의해 변하지 않으므로 결과는 언제나 동일하다.

```jsx
// attributes 프로퍼티에 지정된 value 어트리뷰트 값을 취득한다. 결과는 언제나 동일
document.getElementById('user').getAttribute('value'); // sunny
```

setAttribute 메서드는 어트리뷰트 노드에서 관리하는 HTML 요소에 지정한 어트리뷰트 값, 즉 초기 상태 값을 변경한다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <input id="user" type="text" value="sunny" />
    <script>
      document.getElementById('user').setAttribute('value', 'foo');
    </script>
  </body>
</html>
```

### DOM 프로퍼티

**사용자가 입력한 최신 상태는 HTML 어트리뷰트에 대응하는 요소 노드의 DOM 프로퍼티가 관리한다.**

**DOM 프로퍼티는 사용자의 입력에 의한 상태 변화에 반응하여 언제나 최신 상태를 유지한다.**

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <input id="user" type="text" value="sunny" />
    <script>
      const $input = document.getElementById('user');

      // 사용자가 input 요소의 입력 필드에 값을 입력할 때마다 input 요소 노드의 value 프로퍼티 값,
      // 즉 최신 상태 값을 취득한다. value 프로퍼티 값은 사용자의 입력에 의해 동적으로 변경된다.
      $input.oninput = () => {
        console.log('value 프로퍼티 값', $input.value);
      }

      // getAttribute 메서드로 취득한 HTML 어트리뷰트 값, 초기 상태 값은 변하지 않고 유지된다.
      console.log('value 어트리뷰트 값', $input.getAttribute('value'));
    </script>
  </body>
</html>
```

DOM 프로퍼티에 값을 할당하는 것은 HTML 요소의 최신 상태 값을 변경하는 것을 의미한다.

즉, 사용자가 상태를 변경하는 행위와 같다. HTML 요소에 지정한 어트리뷰트 값에는 어떠한 영향도 주지 않는다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <input id="user" type="text" value="sunny" />
    <script>
      const $input = document.getElementById('user');

      // DOM 프로퍼티에 값을 할당하여 HTML 요소의 최신 상태를 변경한다.
      $input.value = 'foo';
      console.log($input.value);

      // getAttribute 메서드로 취득한 HTML 어트리뷰트 값, 초기 상태 값은 변하지 않고 유지된다.
      console.log('value 어트리뷰트 값', $input.getAttribute('value'));
    </script>
  </body>
</html>
```

사용자 입력에 의한 상태 변화와 관계없는 id 어트리뷰트와 id 프로퍼티는 사용자 입력과 관계없이 항상 동일한 값을 유지한다. → id 어트리뷰트 값이 변하면 id 프로퍼티 값도 변하고 그 반대도 마찬가지다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <input id="user" type="text" value="sunny" />
    <script>
      const $input = document.getElementById('user');

      // id 프로퍼티와 id 프로퍼티는 사용자 입력과 관계없이 항상 동일한 값으로 연동한다.=
      $input.id = 'foo';

      console.log($input.id); // foo
      console.log($input.getAttribute('id')); // foo
    </script>
  </body>
</html>
```

### HTML 어트리뷰트와 DOM 프로퍼티의 대응 관계

- id 어트리뷰트와 id 프로퍼티는 1:1 대응하며, 동일한 값으로 연동한다.
- input 요소의 value 어트리뷰트는 value 프로퍼티와 1:1 대응한다. 하지만 value 어트리뷰트는 초기 상태를, value 프로퍼티는 최신 상태를 갖는다.
- class 어트리뷰트는 className, classList 프로퍼티와 대응한다.
- for 어트리뷰트는 htmlFor 프로퍼티와 1:1 대응한다.
- td 요소의 colspan 어트리뷰트는 대응하는 프로퍼티가 존재하지 않는다.
- textContent 프로퍼티는 대응하는 어트리뷰트가 존재하지 않는다.
- 어트리뷰트 이름은 대소문자를 구별하지 않지만 대응하는 프로퍼티 키는 카멜 케이스를 따른다.

### DOM 프로퍼티 값의 타입

getAttribute 메서드로 취득한 어트리뷰트 값은 언제나 문자열이다. 하지만 DOM 프로퍼티로 취득한 최신 상태 값은 문자열이 아닐 수도 있다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <input type="checkbox" checked />
    <script>
      const $checkbox = document.querySelector('input[type=checkbox]');

      // getAttribute 메서드로 취득한 어트리뷰트 값은 언제나 문자열이다.
      console.log($checkbox.getAttribute('checked')); // ''

      // DOM 프로퍼티로 취득한 최신 상태 값은 문자열이 아닐 수도 있다.
      console.log($checkbox.checked); // true
    </script>
  </body>
</html>
```

## 7.4 data 어트리뷰트와 dataset 프로퍼티

data 어트리뷰트와 dataset 프로퍼티를 사용하면 HTML 요소에 정의한 사용자 정의 어트리뷰트와 자바스크립트 간에 데이터를 교환할 수 있다.

data 어트리뷰트는 `data-user-id` , `data-role` 와 같이 `data-` 접두사 다음에 임의의 이름을 붙여 사용한다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <ul class="users">
      <li id="1" data-user-id="1234" data-role="admin">Son</li>
      <li id="2" data-user-id="5678" data-role="subscriber">Kim</li>
    </ul>
  </body>
</html>
```

data 어트리뷰트의 값은 HTMLElement.dataset 프로퍼티로 취득할 수 있다.

dataset 프로퍼티는 HTML 요소의 모든 data 어트리뷰트의 정보를 제공하는 DOMStringMap 객체를 반환한다. DOMStringMap 객체는 data 어트리뷰트의 `data-` 접두사 다음에 붙인 임의의 이름을 카멜 케이스로 변환한 프로퍼티를 가지고 있다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <ul class="users">
      <li id="1" data-user-id="1234" data-role="admin">Son</li>
      <li id="2" data-user-id="5678" data-role="subscriber">Kim</li>
    </ul>
    <script>
      const users = [...document.querySelector('.users').children];

      // user-id 가 '1234'인 요소 노드를 취득한다.
      const user = users.find(user => user.dataset.userId === '1234');
      // user-id가 '1234' 인 요소 노드에서 data-role의 값을 취득한다.
      console.log(user.dataset.role);

      // user-id가 '1234'인 요소 노드의 data-role 값을 변경한다.
      user.dataset.role = 'subscriber';
      // dataset 프로퍼티는 DOMStringMap 객체를 반환한다.
      console.log(user.dataset);
      // DOMStringMap {userId: "1234", role: "subscriber"}
    </script>
  </body>
</html>
```

data 어트리뷰트의 `data-` 접두사 다음에 존재하지 않는 이름을 키로 사용하여 dataset 프로퍼티에 값을 할당하면 HTML 요소에 data 어트리뷰트가 추가된다.

dataset 프로퍼티에 추가한 카멜케이스(fooBar)의 프로퍼티 키는 data 어트리뷰트의 `data-` 접두사 다음에 케밥케이스(data-foo-bar)로 자동 변경되어 추가된다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <ul class="users">
      <li id="1" data-user-id="1234" data-role="admin">Son</li>
      <li id="2" data-user-id="5678" data-role="subscriber">Kim</li>
    </ul>
    <script>
      const users = [...document.querySelector('.users').children];

      // user-id 가 '1234'인 요소 노드를 취득한다.
      const user = users.find(user => user.dataset.userId === '1234');

      // user-id가 '1234'인 요소 노드에 새로운 data 어트리뷰트를 추가한다.
      user.dataset.fooBar = 'abc';
      console.log(user.dataset);
      // DOMStringMap {userId: "1234", role: "subscriber", fooBar: "abc"}
    </script>
  </body>
</html>
```

# 8. 스타일

## 8.1 인라인 스타일 조작

HTMLElement.prototype.style 프로퍼티는 setter, getter 모두 존재하는 접근자 프로퍼티로서 요소 노드의 인라인 스타일을 취득하거나 추가 또는 변경한다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <div style="color: red">Hello World</div>
    <script>
      const $div = document.querySelector('div');

      // 인라인 스타일 취득
      console.log($div.style);

      // 인라인 스타일 변경
      $div.style.color = 'blue';

      // 인라인 스타일 추가
      $div.style.width = '100px';
      $div.style.height = '100px';
      $div.style.backgroundColor = 'yellow';
    </script>
  </body>
</html>
```

![image](https://github.com/kses1010/sunny-devlog/assets/49144662/d808a21a-4f5d-422b-a78f-b5d0dc8a9075)

style 프로퍼티를 참조하면 CSSStyleDeclaration 타입의 객체를 반환한다. CSSStyleDeclaration 객체는 다양한 CSS 프로퍼티에 대응하는 프로퍼티를 가지고 있으며, 이 프로퍼티에 값을 할당하면 해당 CSSS 프로퍼티가 인라인 스타일로 HTML 요소에 추가되거나 변경된다.

CSS 프로퍼티는 케밥 케이스를 따른다. 이에 대응하는 CSSStyleDeclaration 객체의 프로퍼티는 카멜 케이스를 따른다.

```jsx
$div.style.backgroundColor = "yellow";
// 케밥 케이스의 CSS 프로퍼티를 그대로 사용하려면
$div.style.["background-color"] = "yellow";
```

## 8.2 클래스 조작

`.` 으로 시작하는 클래스 선택자를 사용하여 CSS class를 미리 정의한 다음, HTML 요소의 class 어트리뷰트 값을 변경하여 HTML 요소의 스타일을 변경할 수도 있다.

단, class 어트리뷰트에 대응하는 DOM 프로퍼티는 class가 아니라 className과 classList다.

### className

`Element.prototype.className` 프로퍼티는 setter, getter 모두 존재하는 접근자 프로퍼티로서 HTML 요소의 class 어트리뷰트 값을 취득하거나 변경한다.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <style>
      .box {
        width: 100px;
        height: 100px;
        background-color: antiquewhite;
      }
      .red {
        color: red;
      }
      .blue {
        color: blue;
      }
    </style>
  </head>
  <body>
    <div class="box red">Hello World</div>
    <script>
      const $box = document.querySelector('.box');

      // .box 요소의 class 어트리뷰트 값을 취득
      console.log($box.className); // "box red"

      // .box 요소의 class 어트리뷰트 값 중에서 "red"만 "blue" 로 변경
      $box.className = $box.className.replace('red', 'blue');
    </script>
  </body>
</html>
```

className 프로퍼티는 문자열을 반환하므로 공백으로 구분된 여러 개의 클래스를 반환하는 경우 다루기가 불편하다.

### classList

`Element.prototype.classList` 프로퍼티는 class 어트리뷰트의 정보를 담은 DOMTokenList 객체를 반환한다.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <style>
      .box {
        width: 100px;
        height: 100px;
        background-color: antiquewhite;
      }
      .red {
        color: red;
      }
      .blue {
        color: blue;
      }
    </style>
  </head>
  <body>
    <div class="box red">Hello World</div>
    <script>
      const $box = document.querySelector('.box')

      // .box 요소의 class 어트리뷰트 정보를 담은 DOMTokenList 객체를 취득
      console.log($box.classList);
      // DOMTokenList(2) [length: 2, value: "box blue", 0: "box", 1: "blue"]

      // .box 요소의 class 어트리뷰트 값 중에서 "red"만 "blue" 로 변경
      $box.classList.replace('red', 'blue');
    </script>
  </body>
</html>
```

DOMTokenList 객체는 class 어트리뷰트의 정보를 나타내는 컬렉션 객체로서 유사 배열이면서 이터러블이다.

DOMTokenList 객체는 다음과 같은 메서드를 제공한다.

```jsx
// add(...className)
// 인수로 전달한 1개 이상의 문자열을 class 어트리뷰트 값으로 추가한다.
$box.classList.add('foo'); // class="box red foo"

// remove(...className)
// 인수로 전달한 1개 이상의 문자열과 일치하는 클래스를 class 어트리뷰트에서 삭제한다.
$box.classList.remove('foo'); // class="box red"

// item(index)
// 인수로 전달한 index에 해당하는 클래스를 class 어트리뷰트에서 반환한다.
$box.classList.item(0); // "box"
$box.classList.item(1); // "red"

// contains(className)
// 인수로 전달한 문자열과 일치하는 클래스가 class 어트리뷰트에 포함되어 있는지 확인한다.
$box.classList.contains('box'); // true
$box.classList.contains('blue'); // false

// replace(oldClassName, newClassName)
// 첫 번째 인수로 전달한 문자열을 두 번째 인수로 전달한 문자열로 변경한다.
$box.classList.replace('red', 'blue'); // class="box blue"

// toggle(className[force])
// 인수로 전달한 문자열과 일치하는 클래스가 존재하면 제거하고, 존재하지 않으면 추가한다.
$box.classList.toggle('foo'); // class="box red foo"
$box.classList.toggle('foo'); // class="box red"

// 두 번째 인수로 불리언 값으로 평가되는 조건식을 전달할 수 있다.
// true 면 class 어트리뷰트에 강제로 첫 번째 인수로 전달받은 문자열을 추가
// false 면 class 어트리뷰트에 강제로 첫 번째 인수로 전달받은 문자열을 제거한다.
$box.classList.toggle('foo', true); // class="box red foo"
$box.classList.toggle('foo', false); // class="box red"
```

## 8.3 요소에 적용되어 있는 CSS 스타일 참조

style 프로퍼티는 인라인 스타일만 반환한다.

HTML 요소에 적용되어 있는 모든 CSS 스타일을 참조해야 할 경우 `getComputedStyle` 메서드를 사용한다.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <style>
      .box {
        width: 100px;
        height: 100px;
        background-color: cornsilk;
        border: 1px solid black;
      }
      .red {
        color: red;
      }
    </style>
  </head>
  <body>
    <div class="box red">Hello World</div>
    <script>
      const $box = document.querySelector('.box');

      // .box 요소에 적용된 모든 CSS 스타일을 담고 있는 CSSStyleDeclaration 객체를 취득
      const computedStyle = window.getComputedStyle($box);
      console.log(computedStyle) // CSSStyleDeclaration

      // 임베딩 스타일
      console.log(computedStyle.width); // 100px
      console.log(computedStyle.height); // 100px
      console.log(computedStyle.backgroundColor); // rgb(255, 248, 220)
      console.log(computedStyle.border); // 1px solid rgb(0, 0, 0)

      // 상속 스타일(body -> .box)
      console.log(computedStyle.color); // rgb(255, 0, 0)

      // 기본 스타일
      console.log(computedStyle.display); // block
    </script>
  </body>
</html>
```

`getComputedStyle` 메서드의 두 번째 인수(pseudo)로 `:after`, `:before` 와 같은 의사 요소를 지정하는 문자열을 전달할 수 있다. 의사 요소가 아닌 일반 요소의 경우 두 번째 인수는 생략한다.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <style>
      .box:before {
        content: 'Hello';
      }
    </style>
  </head>
  <body>
    <div class="box">Box</div>
    <script>
      const $box = document.querySelector('.box')

      // 의사 요소 :before의 스타일을 취득한다.
      const computedStyle = window.getComputedStyle($box, ':before');
      console.log(computedStyle.content); // "Hello"
    </script>
  </body>
</html>
```
