---
title: 'Modrn Javascript Deep Dive - 40장 이벤트(1)'
date: 2023-08-29
category: 'Javascript'
draft: false
---

# 1. 이벤트 드리븐 프로그래밍

브라우저는 처리해야 할 특정 사건이 발생하면 이를 감지하여 이벤트를 발생시킨다.

이때 이벤트가 발생했을 때 호출될 함수를 **이벤트 핸들러**라 하고, 이벤트가 발생했을 때 브라우저에게 이벤트 핸들러의 호출을 위임하는 것을 **이벤트 핸들러 등록**이라 한다.

함수를 언제 호출할지 알 수 없으므로 개발자가 명시적으로 함수를 호출하는 것이 아니라 브라우저에게 함수 호출을 위임하는 것이다. 코드는 다음과 같다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <button>Click me!</button>
    <script>
      const $button = document.querySelector('button')

      // 사용자가 버튼을 클릭하면 함수를 호출하도록 요청
      $button.onclick = () => {
        alert('button click')
      }
    </script>
  </body>
</html>
```

이벤트와 그에 대응하는 함수(이벤트 핸들러)를 통해 사용자와 애플리케이션은 상호작용을 할 수 있다.

프로그램의 흐름을 이벤트 중심으로 제어하는 프로그래밍 방식을 **이벤트 드리븐 프로그래밍**이라 한다.

# 2. 이벤트 타입

이벤트 타입은 이벤트의 종류를 나타내는 문자열이다. 이벤트 타입은 약 200여 가지가 있다.

그 중 사용빈도가 높은 이벤트는 다음과 같다.

## 2.1 마우스 이벤트

| 이벤트 타입 | 이벤트 발생 시점                                               |
| ----------- | -------------------------------------------------------------- |
| click       | 마우스 버튼을 클릭했을 때                                      |
| dbclick     | 마우스 버튼을 더블 클릭했을 때                                 |
| mousedown   | 마우스 버튼을 눌렀을 때                                        |
| mouseup     | 누르고 있던 마우스 버튼을 놓았을 때                            |
| mousemove   | 마우스 커서를 움직였을 때                                      |
| mouseenter  | 마우스 커서를 HTML 요소 안으로 이동했을 때(버블링 되지 않는다) |
| mouseover   | 마우스 커서를 HTML 요소 안으로 이동했을 때(버블링 된다)        |
| mouseleave  | 마우스 커서를 HTML 요소 밖으로 이동했을 때(버블링 되지 않는다) |
| mouseout    | 마우스 커서를 HTML 요소 밖으로 이동했을 때(버블링 된다)        |

## 2.2 키보드 이벤트

| 이벤트 타입 | 이벤트 발생 시점         |
| ----------- | ------------------------ |
| keydown     | 모든 키를 눌렀을 때 발생 |

- control, option, shift, tab, delete, 방향 키와 문자, 숫자, 특수 문자 키
- 단, 문자, 숫자, 특수 문자 키를 눌렀을 때 연속적으로 발생하나 그 외엔 한 번만 발생 |
  | keypress | 문자 키를 눌렀을 때 연속적으로 발생(문자, 숫자, 특수 문자 키만 동작) |
  | keyup | 누르고 있던 키를 놓았을 때 한 번만 발생
- keydown 이벤트와 마찬가지로 모든 키로 동작 |

## 2.3 포커스 이벤트

| 이벤트 타입 | 이벤트 발생 시점                                  |
| ----------- | ------------------------------------------------- |
| focus       | HTML 요소가 포커스를 받았을 때(버블링되지 않는다) |
| blur        | HTML 요소가 포커스를 잃었을 때(버블링되지 않는다) |
| focusin     | HTML 요소가 포커스를 받았을 때(버블링된다)        |
| focusout    | HTML 요소가 포커스를 잃었을 때(버블링된다)        |

## 2.4 폼 이벤트

| 이벤트 타입 | 이벤트 발생 시점                                                |
| ----------- | --------------------------------------------------------------- |
| submit      | form 요소 내의 submit 버튼을 클릭했을 때                        |
| reset       | form 요소 내의 reset 버튼을 클릭했을 때(최근에는 사용하지 않음) |

## 2.5 값 변경 이벤트

| 이벤트 타입                                                                                            | 이벤트 발생 시점                                                                                                 |
| ------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------- |
| input                                                                                                  | input(text, checkbox, ratio), select, textarea 요소의 값이 입력되었을 때                                         |
| change                                                                                                 | input(text, checkbox, ratio), select, textarea 요소의 값이 변경되었을 때                                         |
| - change 이벤트는 input 이벤트와는 달리 HTML 요소가 포커스를 잃었을 때 사용자 입력이 종료되었다고 인식 |
| readystatechange                                                                                       | HTML 문서의 로드와 파싱 상태를 나타내는 readyState 프로퍼티 값(’loading’, ‘interactive’, ‘complete’)이 변경될 때 |

## 2.6 DOM 뮤테이션 이벤트

| 이벤트 타입      | 이벤트 발생 시점                                            |
| ---------------- | ----------------------------------------------------------- |
| DOMContentLoaded | HTML 문서의 로드와 파싱이 완료되어 DOM 생성이 완료되었을 때 |

## 2.7 뷰 이벤트

| 이벤트 타입                        | 이벤트 발생 시점                                                     |
| ---------------------------------- | -------------------------------------------------------------------- |
| resize                             | 브라우저 윈도우(window)의 크기를 리사이즈할 때 연속적으로 발생한다.  |
| - 오직 window 객체에서만 발생한다. |
| scroll                             | 웹페이지(document) 또는 HTML 요소를 스크롤할 때 연속적으로 발생한다. |

## 2.8 리소스 이벤트

| 이벤트 타입 | 이벤트 발생 시점                                                                                                      |
| ----------- | --------------------------------------------------------------------------------------------------------------------- |
| load        | DOMContentLoaded 이벤트가 발생한 이후, 모든 리소스(이미지, 폰드 등)의 로딩이 완료되었을 때(주로 window 객체에서 발생) |
| unload      | 리서스가 언로드될 때(주로 새로운 웹페이지를 요청한 경우)                                                              |
| abort       | 리소스 로딩이 중단되었을 때                                                                                           |
| error       | 리소스 로딩이 실패했을 때                                                                                             |

# 3. 이벤트 핸들러 등록

이벤트 핸들러는 이벤트가 발생했을 때 브라우저에 호출을 위임한 함수다.

→ 이벤트가 발생하면 브라우저에 의해 호출될 함수가 이벤트 핸들러다.

이벤트가 발생했을 때 브라우저에게 이벤트 핸들러의 호출을 위임하는 것을 이벤트 핸들러 등록이라 한다.

## 3.1 이벤트 핸들러 어트리뷰트 방식

HTML 요소의 어트리뷰트 중에는 이벤트에 대응하는 이벤트 핸들러 어트리뷰트가 있다.

이벤트 핸들러 어트리뷰트 값으로 함수 호출문 등의 문(statement)을 할당하면 이벤트 핸들러가 등록된다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <button onclick="sayHi('Son')">Click me!</button>
    <script>
      function sayHi(name) {
        console.log(`Hi! ${name}`)
      }
    </script>
  </body>
</html>
```

주의할 점은 이벤트 핸들러 어트리뷰트 값으로 함수 참조가 아닌 함수 호출문 등의 문을 할당한다는 것이다.

**이벤트 핸들러 어트리뷰트 값은 사실 암묵적으로 생성될 이벤트 핸들러의 함수 몸체를 의미한다.**

→ 이벤트 핸들러 어트리뷰트 값으로 여러 개의 문을 할당할 수 있다.

```html
<button onclick="console.log('Hi! '); console.log('Son');">Click me!</button>
```

가능하면 HTML과 자바스크립트는 관심사가 다르므로 혼재하는 것보다 분리하는 것이 좋다.

하지만 모던 자바스크립트에서는 이벤트 핸들러 어트리뷰트 방식을 사용하는 경우가 있다.

CBD(Component Based Development) 방식의 Anguler/React/Svelte/Vue.js 같은 프레임워크/라이브러리는 이벤트 핸들러 어트리뷰트 방식으로 이벤트를 처리한다.

CBD는 HTML, CSS, 자바스크립트를 관심사가 다른 개별적인 요소가 아닌 뷰를 구성하기 위한 구성 요소로 보기 때문에 관심사가 다르다고 생각하지 않는다.

```jsx
// React
<button onClick="{handleClick}">Save</button>
```

## 3.2 이벤트 핸들러 프로퍼티 방식

window 객체와 Document, HTMLElement 방식의 DOM 노드 객체는 이벤트에 대응하는 이벤트 핸들러 프로퍼티를 가지고 있다.

이벤트 핸들러 프로퍼티에 함수를 바인딩하면 이벤트 핸들러가 등록된다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <button>Click me!</button>
    <script>
      const $button = document.querySelector('button')

      // 이벤트 핸들러 프로퍼티에 이벤트 핸들러를 바인딩
      $button.onclick = function() {
        console.log('button click')
      }
    </script>
  </body>
</html>
```

이벤트 핸들러를 등록하기 위해서는 이벤트를 발생시킬 객체를 **이벤트 타깃**과 이벤트의 종류를 나타내는 문자열인 **이벤트 타입** 그리고 **이벤트 핸들러**를 지정할 필요가 있다.

이벤트 핸들러는 대부분 이벤트를 발생시킬 이벤트 타깃에 바인딩한다. 하지만 반드시 이벤트 타깃에 이벤트 핸들러를 바인딩하지 않아도 된다. 이벤트 타깃 또는 전파된 이벤트를 캐치할 DOM 노드 객체에 바인딩한다.

이벤트 핸들러 프로퍼티 방식은 이벤트 핸들러 어트리뷰트 방식의 HTML과 자바스크립트가 뒤섞이는 문제를 해결할 수 있으나 하나의 이벤트 핸들러만 바인딩할 수 있다는 단점이 있다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <button>Click me!</button>
    <script>
      const $button = document.querySelector('button')

      // 이벤트 핸들러 프로퍼티에 이벤트 핸들러를 바인딩은 하나만 바인딩된다.
      // 두 번째 바인딩된 이벤트 핸들러에 의해 재할당되어 실행되지 않는다.
      $button.onclick = function() {
        console.log('button clicked 1')
      }

      // 두 번째 바인딩된 이벤트 핸들러
      $button.onclick = function() {
        console.log('button clicked 2')
      }
    </script>
  </body>
</html>
```

## 3.3 addEventListener 메서드 방식

EventTarget.prototype.addEventListener 메서드를 사용하여 이벤트 핸들러를 등록할 수 있다.

addEventListener 메서드의 첫 번째 매개변수에는 이벤트의 종류를 나타내는 문자열인 이벤트 타입을 전달한다.

두 번째 매개변수에는 이벤트 핸드럴를 전달한다. 마지막 매개변수에는 이벤트 전파 단계(캡처링, 버블링)를 지정할 수 있다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <button>Click me!</button>
    <script>
      const $button = document.querySelector('button')

      // addEventListener 메서드 방식
      $button.addEventListener('click', function() {
        console.log('button click')
      })
    </script>
  </body>
</html>
```

addEventListener 메서드에는 이벤트 핸들러를 바인딩하지 않고 인수로 전달한다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <button>Click me!</button>
    <script>
      const $button = document.querySelector('button')

      // 이벤트 핸들러 프로퍼티 방식
      $button.onclick = function() {
        console.log('[이벤트 핸들러 프로퍼티 방식]button click')
      }

      // addEventListener 메서드 방식
      $button.addEventListener('click', function() {
        console.log('[addEventListener 방식]button click')
      })
    </script>
  </body>
</html>
```

addEventListener 메서드 방식은 이벤트 핸들러 프로퍼티에 바인딩된 이벤트 핸들러에 아무런 영향을 주지 않는다. → 버튼 요소에서 클릭 이벤트가 발생하면 2개의 이벤트 핸들러가 모두 호출된다.

addEventListener 메서드는 하나 이상의 이벤트 핸들러를 등록 할 수 있으며, 등록된 순서대로 호출된다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <button>Click me!</button>
    <script>
      const $button = document.querySelector('button')

      // 이벤트 둘 모두 동작한다.
      $button.addEventListener('click', function() {
        console.log('[1]button click')
      })

      $button.addEventListener('click', function() {
        console.log('[2]button click')
      })
    </script>
  </body>
</html>
```

단, addEventListener 메서드를 통해 참조가 동일한 이벤트 핸들러를 중복 등록하면 하나의 이벤트 핸들러만 등록된다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <button>Click me!</button>
    <script>
      const $button = document.querySelector('button')

      const handleClick = () => console.log('button click')

      // 참조가 동일한 이벤트 핸들러를 중복 등록하면 하나의 핸들러만 등록된다.
      $button.addEventListener('click', handleClick)
      $button.addEventListener('click', handleClick)
    </script>
  </body>
</html>
```

# 4. 이벤트 핸들러 제거

addEventListener 메서드로 등록한 이벤트 핸들러를 제거하려면 EventTarget.prototype.removeEventListener 메서드를 사용한다.

removeEventListener 메서드에 전달할 인수는 addEventListener 메서드와 동일하나 addEventListener 인수와 일치하지 않으면 이벤트 핸들러가 제거되지 않는다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <button>Click me!</button>
    <script>
      const $button = document.querySelector('button')

      const handleClick = () => console.log('button click')

      // 이벤트 핸들러 등록
      $button.addEventListener('click', handleClick)

      // 이벤트 핸들러 제거
      $button.removeEventListener('click', handleClick, true) // 실패
      $button.removeEventListener('click', handleClick) // 성공
    </script>
  </body>
</html>
```

무명 함수를 이벤트 핸들러로 등록한 경우 제거할 수 없다.

이벤트 핸들러를 제거하려면 이벤트 핸들러의 참조를 변수나 자료구조에 저장하고 있어야 한다.

```jsx
$button.addEventListener('click', () => cnosole.log('button click'))
// 삭제 불가
```

단, 기명 이벤트 핸들러 내부에서 removeEventListener 메서드를 호출하여 이벤트 핸들러를 제거하는 것은 가능하며 이때 이벤트 핸들러는 단 한 번만 호출된다.

```jsx
$button.addEventListner('click', function foo() {
  console.log('button click')
  // 이벤트 핸들러를 제거하며 단 한 번만 호출
  $button.removeEventListener('click', foo)
})
```

기명 함수를 이밴트 핸들러로 등록할 수 없다면 호출된 함수, 함수 자신을 가리키는 `arguments.callee` 를 사용할 수도 있다.

```jsx
$button.addEventListner('click', function() {
  console.log('button click')
  // 이벤트 핸들러를 제거하며 단 한 번만 호출
  // arguments.callee는 호출된 함수, 즉 함수 자신을 가리킨다.
  $button.removeEventListener('click', arguments.callee)
})
```

그러나 `arguments.callee` 는 코드 최적화를 방해하므로 strict mode에서 사용이 금지된다.

→ 가급적 이벤트 핸들러의 참조를 변수나 자료구조에 저장하여 제거하는 편이 좋다.

이벤트 핸들러 프로퍼티 방식으로 등록한 이벤트 핸들러는 removeEventListener 메서드로 제거할 수 없다.

이벤트 핸들러 프로퍼티에 null을 할당하여 제거해야 한다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <button>Click me!</button>
    <script>
      const $button = document.querySelector('button')

      const handleClick = () => console.log('button click')

      // 이벤트 핸들러 프로퍼티 방식으로 등록
      $button.onclick = handleClick

      // 이벤트 핸들러 제거 불가
      $button.removeEventListener('click', handleClick)

      // null을 할당하여 핸들러 제거
      $button.onclick = null
    </script>
  </body>
</html>
```

# 5. 이벤트 객체

이벤트가 발생하면 이벤트에 관련한 다양한 정보를 담고 있는 이벤트 객체가 동적으로 생성된다.

**생성된 이벤트 객체는 이벤트 핸들러의 첫 번째 인수로 전달된다.**

```html
<!DOCTYPE html>
<html lang="kr">
  <head>
    <meta charset="UTF-8" />
    <title>Practice</title>
  </head>
  <body>
    <p>클릭하세요. 클릭한 곳의 좌표가 표시됩니다.</p>
    <em class="message"></em>
    <script>
      const $msg = document.querySelector('.message')

      // 클릭 이벤트에 의해 생성된 이벤트 핸들러의 첫 번째 인수로 전달된다.
      function showCoords(e) {
        $msg.textContent = `clientX: ${e.clientX}, clientY: ${e.clientY}`
      }

      document.onclick = showCoords
    </script>
  </body>
</html>
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0428e8f7-e31e-485c-83c2-d0f754e05f1f/Untitled.png)

클릭 이벤트에 의해 생성된 이벤트 객체는 이벤트 핸들러의 첫 번쨰 인수로 전달되어 매개변수 e에 암묵적으로 할당된다. 이는 브라우저가 이벤트 핸들러를 호출할 떄 이벤트 객체를 인수로 전달하기 때문이다.

이벤트 객체를 전달받으려면 이벤트 핸들러를 정의할 때 이벤트 객체를 전달받을 매개변수를 명시적으로 선언해야 한다.

```html
<!DOCTYPE html>
<html lang="kr">
  <head>
    <meta charset="UTF-8" />
    <title>Practice</title>
    <style>
      html,
      body {
        height: 100%;
      }
    </style>
  </head>
  <!--이벤트 핸들러 어트리뷰트 방식의 경우 event가 아닌 다른 이름으로는 이벤트 객체를 전달받지 못한다.-->
  <body onclick="showCoords(event)">
    <p>클릭하세요. 클릭한 곳의 좌표가 표시됩니다.</p>
    <em class="message"></em>
    <script>
      const $msg = document.querySelector('.message')

      // 클릭 이벤트에 의해 생성된 이벤트 핸들러의 첫 번째 인수로 전달된다.
      function showCoords(e) {
        $msg.textContent = `clientX: ${e.clientX}, clientY: ${e.clientY}`
      }
    </script>
  </body>
</html>
```

이벤트 핸들러 어트리뷰트 방식의 경우 이벤트 객체를 전달받으려면 이벤트 핸들러의 첫 번째 매개변수 이름이 반드시 `event` 이어야 한다. 만약 다른 이름일 경우 이벤트를 전달 받지 못한다.

## 5.1 이벤트 객체의 상속 구조

다음 코드와 같이 생성자 함수를 호출하여 이벤트 객체를 생성할 수 있다.

```html
<!DOCTYPE html>
<html lang="kr">
  <head>
    <meta charset="UTF-8" />
    <title>Practice</title>
  </head>
  <body>
    <script>
      // Event 생성자 함수를 호출하여 foo 이벤트 타입의 Event 객체를 생성한다.
      let e = new Event('foo')
      console.log(e)
      // Event {...}
      console.log(e.type) // "foo"
      console.log(e instanceof Event) // true
      console.log(e instanceof Object) // true

      // FocusEvent 생성자 함수를 호출하여 focus 이벤트 타입의 FocusEvent 객체를 생성한다.
      e = new FocusEvent('focus')
      console.log(e)
      // FocusEvent {...}

      // MouseEvent 생성자 함수를 호출하여 click 이벤트 타입의 MouseEvent 객체를 생성한다.
      e = new MouseEvent('click')
      console.log(e)
      // MouseEvent {...}

      // KeyboardEvent 생성자 함수를 호출하여 keyup 이벤트 타입의 KeyboardEvent 객체를 생성
      e = new KeyboardEvent('keyup')
      console.log(e)
      // KeyBoardEvent {...}

      // InputEvent 생성자 함수를 호출하여 change 이벤트 타입의 InputEvent 객체를 생성
      e = new InputEvent('change')
      console.log(e)
      // InputEvent {...}
    </script>
  </body>
</html>
```

이벤트 객체 중 일부는 사용자의 행위에 의해 생성된 것이고 일부는 자바스크립트 코드에 의해 인위적으로 생성된 것이다.

Event 인터페이스는 DOM 내에서 발생한 이벤트에 의해 생성되는 객체를 나타낸다.

Event 인터페이스에는 모든 이벤트 객체의 공통 프로퍼티가 정의되어 있고 FocusEvent, MouseEvent, KeyboardEvent, WheelEvent 같은 하위 인터페이스에는 이벤트 타입에 따라 고유한 프로퍼티가 정의되어 있다.

## 5.2 이벤트 객체의 공통 프로퍼티

Event 인터페이스, 즉 Event.prototype에 정의되어 있는 이벤트 관련 프로퍼티는 UIEvent, CustomEvent, MouseEvent 등 모든 파생 이벤트 객체에 상속된다.

| 공통 프로퍼티                                                  | 설명                                                                | 타입          |
| -------------------------------------------------------------- | ------------------------------------------------------------------- | ------------- |
| type                                                           | 이벤트 타입                                                         | string        |
| target                                                         | 이벤트를 발생시킨 DOM 요소                                          | DOM 요소 노드 |
| currentTarget                                                  | 이벤트 핸들러가 바인딩된 DOM 요소                                   | DOM 요소 노드 |
| eventPhase                                                     | 이벤트 전파 단계                                                    |
| - 0: 이벤트 없음, 1: 캡처링 단계, 2: 타깃 단계, 3: 버블링 단계 | number                                                              |
| bubble                                                         | 이벤트를 버블링으로 전파하는지 여부. 다음 이벤트는 bubbles: false로 |

버블링하지 않는다.

- 포커스 이벤트 focus/blur
- 리소스 이벤트 load/unload/abort/error
- 마우스 이벤트 mouseenter/mouseleave | boolean |
  | cancelable | preventDefault 메서드를 호출하여 이벤트의 기본 동작을 취소할 수 있는지
  여부. 다음 이벤트는 cancelable: false로 취소할 수 없다.
- 포커스 이벤트 focus/blur
- 리소스 이벤트 load/unload/abort/error
- 마우스 이벤트 mouseenter/mouseleave | boolean |
  | defaultPrevented | preventDefault 메서드를 호출하여 이벤트를 취소했는지 여부 | boolean |
  | isTrusted | 사용자의 행위에 의해 발생한 이벤트인지 여부.
  click 메서드를 통해 인위적으로 발생시킨 이벤트인 경우 isTrusted는 false다. | boolean |
  | timeStamp | 이벤트가 발생한 시각(1970/01/01/00:00:0부터 경과한 밀리초) | number |

다음 코드는 체크박스 요소의 체크 상태가 변경되면 현재 체크 상태를 출력하는 코드다

```html
<!DOCTYPE html>
<html lang="kr">
  <head>
    <meta charset="UTF-8" />
    <title>Practice</title>
  </head>
  <body>
    <input type="checkbox" />
    <em class="message">off</em>
    <script>
      const $checkbox = document.querySelector('input[type=checkbox]')
      const $msg = document.querySelector('.message')

      // change 이벤트가 발생하면 Event 타입의 이벤트 객체가 생성된다.
      $checkbox.onchange = e => {
        console.log(Object.getPrototypeOf(e) === Event.prototype) // true

        // e.target.checked는 체크박스 요소의 현재 체크 상태를 나타낸다.
        $msg.textContent = e.target.checked ? 'on' : 'off'
      }
    </script>
  </body>
</html>
```

이벤트 객체의 currentTarget 프로퍼티는 이벤트 핸들러가 바인딩된 DOM 요소를 가리킨다.

→ 이벤트 객체의 target 프로퍼티와 currentTarget 프로퍼티는 동일한 객체 \$checkbox를 가리킨다.

```jsx
$checkbox.onchange = e => {
  // e.currentTarget은 이벤트 핸들러가 바인딩된 DOM 요소 $checkbox를 가리킨다.
  console.log(e.target === e.currentTarget) // true

  $msg.textContent = e.target.checked ? 'on' : 'off'
}
```

## 5.3 마우스 정보 취득

click, dbclick, mousedown, mouseup, mousemove, mouseenter, mouseleave 이벤트가 발생하면 생성되는 MouseEvent 타입의 이벤트 객체는 다음과 같은 고유의 프로퍼티를 갖는다.

- 마우스 포인터의 좌표 정보를 나타내는 프로퍼티
  - screenX/screenY
  - clientX/clientY
  - pageX/pageY
  - offsetX/offsetY
- 버튼 정보를 나타내는 프로퍼티
  - altKey, ctrlKey, shiftKey, button

```html
<!DOCTYPE html>
<html lang="kr">
  <head>
    <meta charset="UTF-8" />
    <title>Practice</title>
    <style>
      .box {
        width: 100px;
        height: 100px;
        background-color: #fff700;
        border: 5px solid orange;
        cursor: pointer;
      }
    </style>
  </head>
  <body>
    <div class="box"></div>
    <script>
      // 드래그 대상 요소
      const $box = document.querySelector('.box')

      // 드래그 시작 시점의 마우스 포인터 위치
      const initialMousePos = { x: 0, y: 0 }
      // 오프셋: 이동할 거리
      const offset = { x: 0, y: 0 }

      // mousemove 이벤트 핸들러
      const move = e => {
        // 오프셋 = 현재 마우스 포인터 좌표 - 드래그 시작 시점의 마우스 포인터 좌표
        offset.x = e.clientX - initialMousePos.x
        offset.y = e.clientY - initialMousePos.y

        // translage3d는 GPU를 사용하므로 absolute의 top, left를 사용하는 것보다 빠르다.
        // top, left는 레이아웃에 영향을 준다.
        $box.style.transform = `translate3d(${offset.x}px, ${offset.y}px, 0)`
      }

      // mousedown 이벤트 핸들러가 발생하면 드래그 시작 시점의 마우스 포인터 좌표를 저장
      $box.addEventListener('mousedown', e => {
        // 이동 거리를 계산하기 위해 mousedown 이벤트가 발생
        // 마우스 포인터 좌표(clientX/clientY)를 저장
        // 한 번 이상 드래그로 이동한 경우 move에서 translate3d로
        // 이동한 상태이므로 offset.x/offset.y를 빼야한다.
        initialMousePos.x = e.clientX - offset.x
        initialMousePos.y = e.clientY - offset.y

        // mousedown 이벤트가 발생한 상태에서 mousemove 이벤트가 발생하면 box 요소를 이동시킨다.
        document.addEventListener('mousemove', move)
      })

      // mouseup 이벤트가 발생하면 mousemove 이벤트를 제거해 이동을 멈춘다.
      document.addEventListener('mouseup', () => {
        document.removeEventListener('mousemove', move)
      })
    </script>
  </body>
</html>
```

## 5.4 키보드 정보 취득

keydown, keyup, keypress 이벤트가 발생하면 생성되는 KeyboardEvent 타입의 이벤트 객체는

altKey, ctrlKey, metaKey, key, keyCode(폐지되어 key를 사용할 것) 같은 고유의 프로퍼티를 갖는다.

다음 코드는 input 요소의 입력 필드에 엔터 키가 입력되면 현재까지 입력 필드에 입력된 값을 출력하는 예제다.

```html
<!DOCTYPE html>
<html lang="kr">
  <head>
    <meta charset="UTF-8" />
    <title>Practice</title>
  </head>
  <body>
    <input type="text" />
    <em class="message"></em>
    <script>
      const $input = document.querySelector('input[type=text]')
      const $msg = document.querySelector('.message')

      $input.onkeyup = e => {
        // e.key는 입력한 키 값을 문자열로 반환
        // 입력한 키가 'Enter', 엔터 키가 아니면 무시한다.
        if (e.key !== 'Enter') return

        // 엔터키가 입력되면 현재까지 입력 필드에 입력된 값을 출력한다.
        $msg.textContent = e.target.value
        e.target.value = ''
      }
    </script>
  </body>
</html>
```

input 요소의 입력 필드에 한글을 입력하고 엔터 키를 누르면 keyup 이벤트 핸들러가 두 번 호출되는 현상이 발생한다. → keyup 이벤트 대신 keydown 이벤트를 캐치한다.
