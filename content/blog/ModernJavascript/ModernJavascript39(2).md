---
title: 'Modrn Javascript Deep Dive - 39장 DOM(2)'
date: 2023-08-27
category: 'Javascript'
draft: false
---

# 3. 노드 탐색

DOM 트리의 노드를 옮겨 다니며 탐색해야 할 때가 있다.

```html
<ul id="fruits">
  <li class="apple">Apple</li>
  <li class="banana">Banana</li>
  <li class="orange">Orange</li>
</ul>
```

`ul#fruits` 요소는 3개의 자식 요소를 갖는다. 먼저 `ul#fruits` 요소 노드를 취득한 다음, 자식 노드를 모두 탐색하거나 자식 노드 중 하나만 탐색할 수 있다.

노드 탐색 프로퍼티는 모두 접근자 프로퍼티다. 단, 노드 탐색 프로퍼티는 setter없이 getter만 존재하여 참조만 가능한 읽기 전용 접근자 프로퍼티다.

## 3.1 공백 텍스트 노드

공백 문자는 텍스트 노드를 생성한다. 이를 공백 텍스트 노드라 한다.

```html
<!DOCTYPE html>
<html lang="en">
  <head> </head>
  <body>
    <ul id="fruits">
      <li class="apple">Apple</li>
      <li class="banana">Banana</li>
      <li class="orange">Orange</li>
    </ul>
  </body>
</html>
```

HTML 문서의 공백 문자는 공백 텍스트 노드를 생성한다. 노드를 탐색할 때는 공백 문자가 생성한 공백 텍스트 노드에 주의해야 한다.

인위적으로 HTML 문서의 공백 문자를 제거하면 공백 텍스트 노드를 생성하지 않는다.

```html
<ul id="fruits">
  <li class="apple">Apple</li>
  <li class="banana">Banana</li>
  <li class="orange">Orange</li>
</ul>
```

그러나 가독성이 좋지 않으므로 권장하지 않는다.

## 3.2 자식 노드 탐색

자식 노드를 탐색하기 위해서는 다음과 같은 노드 탐색 프로퍼티를 사용한다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <ul id="fruits">
      <li class="apple">Apple</li>
      <li class="banana">Banana</li>
      <li class="orange">Orange</li>
    </ul>
    <script>
      // 노드 탐색의 기점이 되는 #fruits 요소 노드를 취득한다.
      const $fruits = document.getElementById('fruits')

      // #fruits 요소의 모든 자식 노드를 탐색한다.
      // childNodes 프로퍼티가 반환한 NodeList에는 요소 노드뿐만 아니라 텍스트 노드도 포함되어 있다.
      console.log($fruits.childNodes)
      // NodeList(7) [text, li.apple, text, li.banana, text, li.orange, text]

      // #fruits 요소의 모든 자식 노드를 탐색한다.
      // children 프로퍼티가 반환한 HTMLCollection에는 요소 노드만 포함되어 있다.
      console.log($fruits.children)
      // HTMLCollection(3) [li.apple, li.banana, li.orange]

      // #fruits 요소의 첫 번째 자식 노드를 탐색한다.
      // firstChild 프로퍼티는 텍스트 노드를 반환할 수도 있다.
      console.log($fruits.firstChild) // #test

      // #fruits 요소의 마지막 자식 노드를 탐색한다.
      // lastChild 프로퍼티는 텍스트 노드를 반환할 수도 있다.
      console.log($fruits.lastChild)

      // #fruits 요소의 첫 번째 자식 노드를 탐색한다.
      // firstElementChild 프로퍼티는 요소 노드만 반환한다.
      console.log($fruits.firstElementChild) // li.apple

      // #fruits 요소의 마지막 자식 노드를 탐색한다.
      // lastElementChild 프로퍼티는 요소 노드만 반환한다.
      console.log($fruits.lastElementChild) // li.orange
    </script>
  </body>
</html>
```

## 3.3 자식 노드 존재 확인

자식 노드가 존재하는지 확인하려면 Node.prototype.hasChildNodes 메서드를 사용한다. `hasChildNodes` 메서드는 자식 노드가 존재하면 true, 자식 노드가 존재하지 않으면 false를 반환한다.

단, `hasChildNodes` 메서드는 childNodes 프로퍼티와 마찬가지로 텍스트 노드를 포함하여 자식 노드의 존재를 확인한다.

요소 노드를 확인하려면 `childElementCount` 프로퍼티를 사용한다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <ul id="fruits"></ul>
    <script>
      // 노드 탐색의 기점이 되는 #fruits 요소 노드를 취득한다.
      const $fruits = document.getElementById('fruits')

      // #fruits 요소에 자식 노드가 존재하는지 확인
      // hasChildNodes 메서드는 텍스트 노드를 포함하여 자식 노드의 존재 확인
      console.log($fruits.hasChildNodes()) // true

      // 자식 노드 중에 텍스트 노드가 아닌 요소 노드가 존재하는지 확인
      console.log(!!$fruits.childElementCount) // 0 -> false
    </script>
  </body>
</html>
```

## 3.4 요소 노드의 텍스트 노드 탐색

요소 노드의 텍스트 노드는 요소 노드의 자식 노드다. 즉, 요소 노드의 텍스트 노드는 firstChild 프로퍼티로 접근할 수 있다.

firstChild 프로퍼티는 첫 번째 자식 노드를 반환하며, 텍스트 노드이거나 요소 노드다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <div id="foo">Hello</div>
    <script>
      // 요소 노드의 텍스트 노드는 firstChild 프로퍼티로 접근할 수 있다.
      console.log(document.getElementById('foo').firstChild) // #text
    </script>
  </body>
</html>
```

## 3.5 부모 노드 탐색

부모 노드를 탐색하려면 Node.prototype.parentNode 프로퍼티를 사용한다.

텍스트 노드는 DOM 트리의 최종단 노드인 리프 노드이므로 부모 노드가 텍스트 노드인 경우는 없다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <ul id="fruits">
      <li class="apple">Apple</li>
      <li class="banana">Banana</li>
      <li class="orange">Orange</li>
    </ul>
    <script>
      // 노드 탐색의 기점이 되는 .banana 요소 노드를 취득한다.
      const $banana = document.querySelector('.banana')

      // .banana 요소 노드의 부모 노드를 탐색한다.
      console.log($banana.parentNode) // ul#fruits
    </script>
  </body>
</html>
```

## 3.6 형제 노드 탐색

부모 노드가 같은 형제 노드를 탐색하려면 다음과 같은 노드 탐색 프로퍼티를 사용한다.

단, 어트리뷰트 노드는 요소 노드의 형제 노드이지만 부모 노드가 같은 형제 노드가 아니기 때문에 반환되지 않는다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <ul id="fruits">
      <li class="apple">Apple</li>
      <li class="banana">Banana</li>
      <li class="orange">Orange</li>
    </ul>
    <script>
      // 노드 탐색의 기점이 되는 #fruits 요소 노드를 취득한다.
      const $fruits = document.getElementById('fruits')

      // #fruits 요소의 첫 번째 자식 노드를 탐색
      // firstChild 프로퍼티는 요소 노드뿐만 아니라 텍스트 노드를 반환할 수 있다.
      const { firstChild } = $fruits
      console.log(firstChild) // #text

      // #fruits 요소의 첫 번째 자식 노드(텍스트 노드)의 다음 형제 노드를 탐색
      // nextSibling 프로퍼티는 요소 노드뿐만 아니라 텍스트 노드를 반환할 수 있다.
      const { nextSibling } = firstChild
      console.log(nextSibling) // li.apple

      // li.apple 요소의 이전 형제 노드를 탐색
      // previousSibling 프로퍼티는 요소 노드뿐만 아니라 텍스트 노드를 반환할 수 있다.
      const { previousSibling } = nextSibling
      console.log(previousSibling) // #text

      // #fruits 요소의 첫 번째 자식 노드를 탐색
      // firstElementChild 프로퍼티는 요소 노드만 반환
      const { firstElementChild } = $fruits
      console.log(firstElementChild) // li.apple

      // #fruits 요소의 첫 번째 자식 노드(li.apple)의 다음 형제 노드를 탐색
      // nextElementSibling 프로퍼티는 요소 노드만 반환
      const { nextElementSibling } = firstElementChild
      console.log(nextElementSibling) // li.banana

      // li.banana 요소의 이전 요소 노드를 탐색
      // previousElementSibling 프로퍼티는 요소 노드만 반환
      const { previousElementSibling } = nextElementSibling
      console.log(previousElementSibling) // li.apple
    </script>
  </body>
</html>
```

# 4. 노드 정보 취득

노드 객체에 대한 정보를 취득하려면 다음과 같은 노드 정보 프로퍼티를 사용한다.

| 프로퍼티                | 설명                                                                                              |
| ----------------------- | ------------------------------------------------------------------------------------------------- |
| Node.prototype.nodeType | 노드 객체의 종류, 즉 노드 타입을 나타내는 상수를 반환한다. 노드 타입 상수는 Node에 정의되어 있다. |

- Node.ELEMENT_NODE: 요소 노드 타입을 나타내는 상수 1을 반환
- Node.TEXT_NODE: 텍스트 노드 타입을 나타내는 상수 3을 반환
- Node.DOCUMENT_NODE: 문서 노드 타입을 나타내는 상수 9를 반환 |
  | Node.prototype.nodeName | 노드의 이름을 문자열로 반환한다.
- 요소 노드: 대문자 문자열로 태그이름(”UL”, “LI” 등)을 반환
- 텍스트 노드: 문자열 “#text”를 반환
- 문서 노드: 문자열 “#document”를 반환 |

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <div id="foo">Hello</div>
    <script>
      // 문서 노드의 노드 정보를 취득
      console.log(document.nodeType) // 9
      console.log(document.nodeName) // #document

      // 요소 노드의 노드 정보를 취득
      const $foo = document.getElementById('foo')
      console.log($foo.nodeType) // 1
      console.log($foo.nodeName) // DIV

      // 텍스트 노드의 노드 정보를 취득
      const $textNode = $foo.firstChild
      console.log($textNode.nodeType) // 3
      console.log($textNode.nodeName) // #text
    </script>
  </body>
</html>
```

# 5. 요소 노드의 텍스트 조작

## 5.1 nodeValue

Node.prototype.nodeValue 프로퍼티는 setter, getter 모두 존재하는 접근자 프로퍼티다.

노드 객체의 nodeValue 프로퍼티를 참조하면 노드 객체의 값을 반환한다. 노드 객체의 값이란 텍스트 노드의 텍스트다.

→ 텍스트 노드가 아닌 노드, 즉 문서 노드나 요소 노드의 nodeValue 프로퍼티를 참조하면 null을 반환한다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <div id="foo">Hello</div>
    <script>
      // 문서 노드의 nodeValue 프로퍼티를 참조한다.
      console.log(document.nodeValue) // null

      // 요소 노드의 nodeValue 프로퍼티를 참조한다.
      const $foo = document.getElementById('foo')
      console.log($foo.nodeValue) // null

      // 텍스트 노드의 nodeValue 프로퍼티를 참조한다.
      const $textNode = $foo.firstChild
      console.log($textNode.nodeValue) // Hello
    </script>
  </body>
</html>
```

텍스트 노드의 nodeValue 프로퍼티에 값을 할당하면 텍스트 노드의 값, 즉 텍스트를 변경할 수 있다.

→ 요소 노드의 텍스트를 변경하려면 다음과 같은 순서의 처리가 필요하다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <div id="foo">Hello</div>
    <script>
      // 1. #foo 요소 노드의 자식 노드인 텍스트 노드를 취득한다.
      const $textNode = document.getElementById('foo').firstChild

      // 2. nodeValue 프로퍼티를 사용하여 텍스트 노드의 값을 변경한다.
      $textNode.nodeValue = 'World'

      console.log($textNode.nodeValue) // World
    </script>
  </body>
</html>
```

## 5.2 textContent

Node.prototype.textContent 프로퍼티는 setter, getter 모두 존재하는 접근자 프로퍼티로서 요소 노드의 텍스트와 모든 자손 노드의 텍스트를 모두 취득하거나 변경한다.

HTML 마크업은 무시하고 모든 텍스트를 반환한다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <div id="foo">Hello <span>world!</span></div>
    <script>
      // #foo 요소 노드의 텍스트를 모두 취득한다. 이때 HTML 마크업은 무시된다.
      console.log(document.getElementById('foo').textContent) // Hello world!
    </script>
  </body>
</html>
```

만약 요소 노드의 콘텐츠 영역에 자식 요소 노드가 없고 텍스트만 존재한다면 `firstChild.nodeValue` 와 `textContent` 프로퍼티는 같은 결과를 반환한다.

이 경우 textContent 프로퍼티를 사용하는 편이 코드가 더 간단하다.

요소 노드의 textContent 프로퍼티에 문자열을 할당하면 요소 노드의 모든 자식 노드가 제거되고 할당한 문자열이 텍스트로 추가된다.

이때 할당한 문자열에 HTML 마크업이 포함되어 있더라도 문자열 그대로 인식되어 텍스트로 취급된다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <div id="foo">Hello <span>world!</span></div>
    <script>
      document.getElementById('foo').textContent = 'Hi <span>there!</span>'
    </script>
  </body>
</html>
```

![image](https://github.com/kses1010/sunny-devlog/assets/49144662/39b3c040-5571-43c7-84d3-a8d92a1e057f)
