---
title: 'Modern Javascript Deep Dive - 39장 DOM(2)'
date: 2023-08-28 16:40:58
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
<ul id="fruits"><li class="apple">Apple</li><li class="banana">Banana</li><li class="orange">Orange</li>
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
      const $fruits = document.getElementById('fruits');

      // #fruits 요소의 모든 자식 노드를 탐색한다.
      // childNodes 프로퍼티가 반환한 NodeList에는 요소 노드뿐만 아니라 텍스트 노드도 포함되어 있다.
      console.log($fruits.childNodes);
      // NodeList(7) [text, li.apple, text, li.banana, text, li.orange, text]

      // #fruits 요소의 모든 자식 노드를 탐색한다.
      // children 프로퍼티가 반환한 HTMLCollection에는 요소 노드만 포함되어 있다.
      console.log($fruits.children);
      // HTMLCollection(3) [li.apple, li.banana, li.orange]

      // #fruits 요소의 첫 번째 자식 노드를 탐색한다.
      // firstChild 프로퍼티는 텍스트 노드를 반환할 수도 있다.
      console.log($fruits.firstChild); // #test

      // #fruits 요소의 마지막 자식 노드를 탐색한다.
      // lastChild 프로퍼티는 텍스트 노드를 반환할 수도 있다.
      console.log($fruits.lastChild);

      // #fruits 요소의 첫 번째 자식 노드를 탐색한다.
      // firstElementChild 프로퍼티는 요소 노드만 반환한다.
      console.log($fruits.firstElementChild); // li.apple

      // #fruits 요소의 마지막 자식 노드를 탐색한다.
      // lastElementChild 프로퍼티는 요소 노드만 반환한다.
      console.log($fruits.lastElementChild); // li.orange
    </script>
  </body>
</html>
```

## 3.3 자식 노드 존재 확인

자식 노드가 존재하는지 확인하려면 `Node.prototype.hasChildNodes` 메서드를 사용한다. `hasChildNodes` 메서드는 자식 노드가 존재하면 true, 자식 노드가 존재하지 않으면 false를 반환한다.

단, `hasChildNodes` 메서드는 childNodes 프로퍼티와 마찬가지로 텍스트 노드를 포함하여 자식 노드의 존재를 확인한다.

요소 노드를 확인하려면 `childElementCount` 프로퍼티를 사용한다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <ul id="fruits"></ul>
    <script>
      // 노드 탐색의 기점이 되는 #fruits 요소 노드를 취득한다.
      const $fruits = document.getElementById('fruits');

      // #fruits 요소에 자식 노드가 존재하는지 확인
      // hasChildNodes 메서드는 텍스트 노드를 포함하여 자식 노드의 존재 확인
      console.log($fruits.hasChildNodes()); // true

      // 자식 노드 중에 텍스트 노드가 아닌 요소 노드가 존재하는지 확인
      console.log(!!$fruits.childElementCount); // 0 -> false
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
      console.log(document.getElementById('foo').firstChild); // #text
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
      const $banana = document.querySelector('.banana');

      // .banana 요소 노드의 부모 노드를 탐색한다.
      console.log($banana.parentNode); // ul#fruits
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
      const $fruits = document.getElementById('fruits');

      // #fruits 요소의 첫 번째 자식 노드를 탐색
      // firstChild 프로퍼티는 요소 노드뿐만 아니라 텍스트 노드를 반환할 수 있다.
      const { firstChild } = $fruits;
      console.log(firstChild); // #text

      // #fruits 요소의 첫 번째 자식 노드(텍스트 노드)의 다음 형제 노드를 탐색
      // nextSibling 프로퍼티는 요소 노드뿐만 아니라 텍스트 노드를 반환할 수 있다.
      const { nextSibling } = firstChild;
      console.log(nextSibling); // li.apple

      // li.apple 요소의 이전 형제 노드를 탐색
      // previousSibling 프로퍼티는 요소 노드뿐만 아니라 텍스트 노드를 반환할 수 있다.
      const { previousSibling } = nextSibling;
      console.log(previousSibling); // #text

      // #fruits 요소의 첫 번째 자식 노드를 탐색
      // firstElementChild 프로퍼티는 요소 노드만 반환
      const { firstElementChild } = $fruits;
      console.log(firstElementChild); // li.apple

      // #fruits 요소의 첫 번째 자식 노드(li.apple)의 다음 형제 노드를 탐색
      // nextElementSibling 프로퍼티는 요소 노드만 반환
      const { nextElementSibling } = firstElementChild;
      console.log(nextElementSibling); // li.banana

      // li.banana 요소의 이전 요소 노드를 탐색
      // previousElementSibling 프로퍼티는 요소 노드만 반환
      const { previousElementSibling } = nextElementSibling;
      console.log(previousElementSibling); // li.apple
    </script>
  </body>
</html>
```

# 4. 노드 정보 취득

노드 객체에 대한 정보를 취득하려면 다음과 같은 노드 정보 프로퍼티를 사용한다.

| 프로퍼티                    | 설명                                                          |
| ----------------------- | ----------------------------------------------------------- |
| Node.prototype.nodeType | 노드 객체의 종류, 즉 노드 타입을 나타내는 상수를 반환한다. 노드 타입 상수는 Node에 정의되어 있다. |
|                         | - Node.ELEMENT_NODE: 요소 노드 타입을 나타내는 상수 1을 반환                |
|                         | - Node.TEXT_NODE: 텍스트 노드 타입을 나타내는 상수 3을 반환                  |
|                         | - Node.DOCUMENT_NODE: 문서 노드 타입을 나타내는 상수 9를 반환               |
| Node.prototype.nodeName | 노드의 이름을 문자열로 반환한다.                                          |
|                         | - 요소 노드: 대문자 문자열로 태그이름(”UL”, “LI” 등)을 반환                    |
|                         | - 텍스트 노드: 문자열 “#text”를 반환                                   |
|                         | - 문서 노드: 문자열 “#document”를 반환                                |

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <div id="foo">Hello</div>
    <script>
      // 문서 노드의 노드 정보를 취득
      console.log(document.nodeType); // 9
      console.log(document.nodeName); // #document

      // 요소 노드의 노드 정보를 취득
      const $foo = document.getElementById('foo');
      console.log($foo.nodeType); // 1
      console.log($foo.nodeName); // DIV

      // 텍스트 노드의 노드 정보를 취득
      const $textNode = $foo.firstChild;
      console.log($textNode.nodeType); // 3
      console.log($textNode.nodeName); // #text
    </script>
  </body>
</html>
```

# 5. 요소 노드의 텍스트 조작

## 5.1 nodeValue

`Node.prototype.nodeValue` 프로퍼티는 setter, getter 모두 존재하는 접근자 프로퍼티다.

노드 객체의 nodeValue 프로퍼티를 참조하면 노드 객체의 값을 반환한다. 노드 객체의 값이란 텍스트 노드의 텍스트다.

→ 텍스트 노드가 아닌 노드, 즉 문서 노드나 요소 노드의 nodeValue 프로퍼티를 참조하면 null을 반환한다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <div id="foo">Hello</div>
    <script>
      // 문서 노드의 nodeValue 프로퍼티를 참조한다.
      console.log(document.nodeValue); // null

      // 요소 노드의 nodeValue 프로퍼티를 참조한다.
      const $foo = document.getElementById('foo');
      console.log($foo.nodeValue); // null

      // 텍스트 노드의 nodeValue 프로퍼티를 참조한다.
      const $textNode = $foo.firstChild;
      console.log($textNode.nodeValue); // Hello
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
      const $textNode = document.getElementById('foo').firstChild;

      // 2. nodeValue 프로퍼티를 사용하여 텍스트 노드의 값을 변경한다.
      $textNode.nodeValue = 'World';

      console.log($textNode.nodeValue); // World
    </script>
  </body>
</html>
```

## 5.2 textContent

`Node.prototype.textContent`프로퍼티는 setter, getter 모두 존재하는 접근자 프로퍼티로서 요소 노드의 텍스트와 모든 자손 노드의 텍스트를 모두 취득하거나 변경한다.

HTML 마크업은 무시하고 모든 텍스트를 반환한다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <div id="foo">Hello <span>world!</span></div>
    <script>
      // #foo 요소 노드의 텍스트를 모두 취득한다. 이때 HTML 마크업은 무시된다.
      console.log(document.getElementById('foo').textContent); // Hello world!
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
      document.getElementById('foo').textContent = 'Hi <span>there!</span>';
    </script>
  </body>
</html>
```

![image](https://github.com/kses1010/sunny-devlog/assets/49144662/39b3c040-5571-43c7-84d3-a8d92a1e057f)

# 6. DOM 조작

DOM 조작은 새로운 노드를 생성하여 DOM에 추가하거나 기존 노드를 삭제 또는 교체하는 것을 말한다.

DOM 조작에 의해 DOM에 새로운 노드가 추가되거나 삭제되면 리플로우와 리페인트가 발생하는 원인이 되므로 성능에 영향을 준다.

## 6.1 innerHTML

Element.prototype.innerHTML 프로퍼티는 setter, getter 모두 존재하는 접근자 프로퍼티다.

요소 노드의 HTML 마크업을 취득하거나 변경한다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <div id="foo">Hello <span>world!</span></div>
    <script>
      // #foo 요소의 콘텐츠 영역 내의 HTML 마크업을 문자열로 취득한다.
      console.log(document.getElementById('foo').innerHTML);
      // "Hello <span>world!</span>"
    </script>
  </body>
</html>
```

요소 노드의 innerHTML 프로퍼티에 문자열을 할당하면 요소 노드의 모든 자식 노드가 제거되고 할당한 문자열에 포함되어 있는 HTML 마크업이 파싱되어 요소 노드의 자식 노드로 DOM에 반영된다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <div id="foo">Hello <span>world!</span></div>
    <script>
      document.getElementById('foo').innerHTML = 'Hi <span>there!</span>'
    </script>
  </body>
</html>
```

이처럼 innerHTML 프로퍼티를 사용하면 HTML 마크업 문자열로 간단히 DOM 조작이 가능하다.

요소 노드의 innerHTML 프로퍼티에 할당한 HTML 마크업 문자열은 렌더링 엔진에 의해 파싱되어 요소 노드의 자식으로 DOM에 반영된다.

이때 사용자로부터 입력받은 데이터를 그대로 innerHTML 프로퍼티에 할당하는 것은 **크로스 사이트 스크립팅 공격(XSS)** 에 취약하므로 위험하다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <div id="foo">Hello</div>
    <script>
      // 에러 이벤트를 강제로 발생시켜서 자바스크립트 코드가 실행되도록 한다.
      document.getElementById(
        'foo'
      ).innerHTML = `<img src="x" onerror="alert(document.cookie)">`;
    </script>
  </body>
</html>
```

innerHTML 프로퍼티의 또 다른 단점은 요소 노드의 innerHTML 프로퍼티에 HTML 마크업 문자열을 할당하는 경우 요소 노드의 모든 자식 노드를 제거하고 할당한 HTML 마크업 문자열을 파싱하여 DOM을 변경시키는 것이다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <ul id="fruits">
      <li class="apple">Apple</li>
    </ul>
  </body>
  <script>
    const $fruits = document.getElementById("fruits");

    // 노드 추가
    $fruits.innerHTML += "<li class="banana">Banana</li>";
  </script>
</html>
```

`#fruits` 요소에 자식 요소 li.banana 를 추가한다. 이때 `#fruits` 요소의 자식 요소 li.apple은 아무런 변경이 없으므로 다시 생성할 필요가 없다.

다만 새롭게 추가할 li.banana 요소 노드만 생성하여 `#fruits` 요소의 자식 요소로 추가하면 된다.

그럴듯 하지만 사실은 `#fruits` 요소의 모든 자식 노드(li.apple)를 제거하고 새롭게 요소 노드 li.apple 과 li.banana 를 생성하여 `#fruits` 요소의 자식요소로 추가한다.

innerHTML 프로퍼티의 단점은 이뿐만 아니다. 

새로운 요소를 삽입할 때 삽입될 위치를 지정할 수 없다는 단점도 있다.

```html
<ul id="fruits">
  <li class="apple">Apple</li>
  <li class="orange">Orange</li>
</ul>
```

li.apple 요소와 li.orange 요소 사이에 새로운 요소를 삽입하고 싶은 경우 innerHTML 프로퍼티를 사용하면 삽입 위치를 지정할 수 없다.

이처럼 innerHTML 프로퍼티는 복잡하지 않은 요소를 새롭게 추가할 때 유용하지만 기존 요소를 제거하지 않으면서 위치를 지정해 새로운 요소를 삽입해야 할 때는 사용하지 않는 것이 좋다.

## 6.2 insertAdjacentHTML 메서드

`Element.prototype.insertAdjacentHTML(position, DOMString)` 메서드는 기존 요소를 제거하지 않으면서 위치를 지정해 새로운 요소를 삽입한다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <!--    beforebegin-->
    <div id="foo">
      <!--    afterbegin-->
      text
      <!--    beforeend-->
    </div>
    <!--    afterend-->
  </body>
  <script>
    const $foo = document.getElementById('foo')

    $foo.insertAdjacentHTML('beforebegin', '<p>beforebigin</p>')
    $foo.insertAdjacentHTML('afterbegin', '<p>afterbigin</p>')
    $foo.insertAdjacentHTML('beforeend', '<p>beforeend</p>')
    $foo.insertAdjacentHTML('afterend', '<p>afterend</p>')
  </script>
</html>
```

insertAdjacentHTML 메서드는 기존 요소에는 영향을 주지 않고 새롭게 삽입될 요소만을 파싱하여 자식 요소로 추가하므로 기존의 자식 노드를 모두 제거하고 다시 처음부터 새롭게 자식 노드를 생성하여 자식 요소로 추가하는 innerHTML 프로퍼티보다 효율적이고 빠르다.

그러나, insertAdjacentHTML 메서드도 또한 크로스 사이트 스크립팅 공격에 취약하다.

## 6.3 노드 생성과 추가

DOM은 노드를 직접 생성/삽입/삭제/치환하는 메서드도 제공한다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <ul id="fruits">
      <li>Apple</li>
    </ul>
  </body>
  <script>
    const $fruits = document.getElementById('fruits');

    // 1. 요소 노드 생성
    const $li = document.createElement('li');

    // 2. 텍스트 노드 생성
    const textNode = document.createTextNode('Banana');

    // 3. 텍스트 노드를 $li 요소 노드의 자식 노드로 추가
    $li.appendChild(textNode);

    // 4. $li 요소 노드를 #fruits 요소 노드의 마지막 자식 노드로 추가
    $fruits.appendChild($li);
  </script>
</html>
```

### 요소 노드 생성

Document.prototype.createElement(tagName) 메서드는 요소 노드를 생성하여 반환한다.

```jsx
// 1. 요소 노드 생성
const $li = document.createElement('li');
// 생성된 요소 노드는 아무런 자식 노드가 없다.
console.log($li.childNodes); // NodeList []
```

### 텍스트 노드 생성

`Document.prototype.createTextNode(text)` 메서드는 텍스트 노드를 생성하여 반환한다.

```jsx
// 2. 텍스트 노드 생성
const textNode = document.createTextNode('Banana');
```

createTextNode 메서드는 텍스트 노드를 생성할 뿐 요소 노드에 추가하지는 않는다.

→ 이후에 생성된 텍스트 노드를 요소 노드에 추가하는 처리가 별도로 필요하다.

### 텍스트 노드를 요소 노드의 자식 노드로 추가

Node.prototype.appendChild(childNode) 메서드는 매개변수 childNode에게 인수로 전달한 노드를 appendChild 메서드를 호출한 노드의 마지막 자식 노드로 추가한다.

```jsx
// 3. 텍스트 노드를 $li 요소 노드의 자식 노드로 추가
$li.appendChild(textNode);
```

요소 노드에 자식 노드가 하나도 없는 경우에는 텍스트 노드를 생성하여 요소 노드의 자식 노드로 텍스트 노드로 추가하는 것보다 textContent 프로퍼티를 사용하는 편이 더욱 간편하다.

```jsx
// 텍스트 노드를 생성하여 요소 노드의 자식 노드로 추가
$li.appendChild(document.createTextnode('Banana'));

// $li 요소 노드에 자식 노드가 하나도 없는 위 코드와 동일하게 동작
$li.textContent = 'Banana';
```

단, 요소 노드에 자식 노드가 있는 경우 요소 노드의 textContent 프로퍼티에 문자열을 할당하면 요소 노드의 모든 자식 노드가 제거되고 할당한 문자열이 텍스트로 추가되므로 주의해야 한다.

### 요소 노드를 DOM에 추가

Node.prototype.appendChild 메서드를 사용하여 텍스트 노드와 부자 관계로 연결한 요소 노드를 `#fruits` 요소 노드의 마지막 자식 요소로 추가한다.

```jsx
// 4. $li 요소 노드를 #fruits 요소 노드의 마지막 자식 노드로 추가
$fruits.appendChild($li);
```

단 하나의 요소 노드를 생성하여 DOM에 한번 추가하므로 DOM은 한 번 변경된다.

→ 리플로우와 리페인트가 실행된다.

## 6.4 복수의 노드 생성과 추가

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <ul id="fruits"></ul>
  </body>
  <script>
    const $fruits = document.getElementById('fruits');

    ['Apple', 'Banana', 'Orange'].forEach(text => {
      // 1. 요소 노드 생성
      const $li = document.createElement('li');

      // 2. 텍스트 노드 생성
      const textNode = document.createTextNode(text);

      // 3. 텍스트 노드를 $li 요소 노드의 자식 노드로 추가
      $li.appendChild(textNode);

      // 4. $li 요소 노드를 #fruits 요소 노드의 마지막 자식 노드로 추가
      $fruits.appendChild($li);
    })
  </script>
</html>
```

위 예제와 같이 기존 DOM에 요소 노드를 반복하여 추가하는 것은 비효율적이다.

컨테이너 요소를 미리 생성한 다음, DOM에 추가해야 할 3개의 요소 노드를 컨테이너 요소에 자식 노드로 추가하고, 컨테이너 `#fruits` 요소에 자식으로 추가한다면 DOM은 한 번만 변경된다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <ul id="fruits"></ul>
  </body>
  <script>
    const $fruits = document.getElementById('fruits');

    // 컨테이너 요소 노드 생성
    const $container = document.createElement('div');

    ['Apple', 'Banana', 'Orange'].forEach(text => {
      // 1. 요소 노드 생성
      const $li = document.createElement('li');

      // 2. 텍스트 노드 생성
      const textNode = document.createTextNode(text);

      // 3. 텍스트 노드를 $li 요소 노드의 자식 노드로 추가
      $li.appendChild(textNode);

      // 4. $li 요소 노드를 #container 요소 노드의 마지막 자식 노드로 추가
      $container.appendChild($li);
    })

    // 5. 컨테이너 요소 노드를 #fruits 요소 노드의 마지막 자식 노드로 추가
    $fruits.appendChild($container);
  </script>
</html>
```

DOM을 한 번만 변경하므로 성능에 유리하기는 하지만 불필요한 컨테이너 요소(div)가 DOM에 추가되는 부작용이 있다.

이 문제는 `DocumentFragment` 노드를 통해 해결할 수 있다.

`DocumentFragment` 노드는 자식 노드들의 부모 노드로서 별도의 서브 DOM을 구성하여 기존 DOM에 추가하기 위한 용도로 사용한다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <ul id="fruits"></ul>
  </body>
  <script>
    const $fruits = document.getElementById('fruits');

    // DocumentFragment 노드 생성
    const $fragment = document.createDocumentFragment();

    ['Apple', 'Banana', 'Orange'].forEach(text => {
      // 1. 요소 노드 생성
      const $li = document.createElement('li');

      // 2. 텍스트 노드 생성
      const textNode = document.createTextNode(text);

      // 3. 텍스트 노드를 $li 요소 노드의 자식 노드로 추가
      $li.appendChild(textNode);

      // 4. $li 요소 노드를 DocumentFragment 요소 노드의 마지막 자식 노드로 추가
      $fragment.appendChild($li);
    })

    // 5. DocumentFragment 노드를 #fruits 요소 노드의 마지막 자식 노드로 추가
    $fruits.appendChild($fragment);
  </script>
</html>
```

실제로 DOM 변경이 발생하는 것은 한 번뿐이며 리플로우와 리페인트도 한 번만 실행된다.

→ 여러 개의 요소 노드를 DOM에 추가하는 경우 DocumentFragment 노드를 사용하는 것이 더 효율적이다.

## 6.5 노드 삽입

### 마지막 노드 추가

`Node.prototype.appendChild` 메서드는 인수로 전달받은 노드를 자신을 호출한 노드의 마지막 자식 노드로 DOM에 추가한다. 

→ 노드를 추가할 위치를 지정할 수 없고 언제나 마지막 자식 노드로 추가한다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <ul id="fruits">
      <li>Apple</li>
      <li>Banana</li>
    </ul>
  </body>
  <script>
    // 요소 노드 생성
    const $li = document.createElement('li');

    // 텍스트 노드를 $li 요소 노드의 마지막 자식 노드로 추가
    $li.appendChild(document.createTextNode('Orange'));

    // $li 요소 노드를 #fruits 요소 노드의 마지막 자식 노드로 추가
    document.getElementById('fruits').appendChild($li);
  </script>
</html>
```

### 지정한 위치에 노드 삽입

`Node.prototype.insertBefore(newNode, childNode)` 메서드는 첫 번째 인수로 전달받은 노드를 두 번째 인수로 전달받은 노드 앞에 삽입한다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <ul id="fruits">
      <li>Apple</li>
      <li>Banana</li>
    </ul>
  </body>
  <script>
    const $fruits = document.getElementById('fruits');

    // 요소 노드 생성
    const $li = document.createElement('li');

    // 텍스트 노드를 $li 요소 노드의 마지막 자식 노드로 추가
    $li.appendChild(document.createTextNode('Orange'));

    // $li 요소 노드를 #fruits 요소 노드의 마지막 자식 노드로 추가
    $fruits.insertBefore($li, $fruits.lastElementChild);
    // Apple - Orange - Banana
  </script>
</html>
```

두 번째 인수로 전달받은 노드는 반드시 insertBefore 메서드를 호출한 노드의 자식 노드이어야 한다.

그렇지 않으면 DOMException이 발생한다.

두 번째 인수로 전달받은 노드가 null이면 첫 번째 인수로 전달받은 노드를 insertBefore 메서드를 호출한 노드의 마지막 자식 노드로 추가된다. → appendChild 메서드처럼 동작한다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <ul id="fruits">
      <li>Apple</li>
      <li>Banana</li>
    </ul>
  </body>
  <script>
    const $fruits = document.getElementById('fruits')

    // 요소 노드 생성
    const $li = document.createElement('li')

    // 텍스트 노드를 $li 요소 노드의 마지막 자식 노드로 추가
    $li.appendChild(document.createTextNode('Orange'))

    // 두 번째 인수로 전달받은 노드는 반드시 #fruits 요소 노드의 자식 노드이어야 한다.
    $fruits.insertBefore($li, document.querySelector('div')); // DOMException
    // null 이면 apppendChild 메서드 처럼 동작
    $fruits.insertBefore($li, null);
  </script>
</html>
```

## 6.6 노드 이동

DOM에 이미 존재하는 노드를 appendChild 또는 insertBefore 메서드를 사용하여 DOM에 다시 추가하면 현재 위치에서 노드를 제거하고 새로운 위치에 노드를 추가한다. → 노드가 이동한다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <ul id="fruits">
      <li>Apple</li>
      <li>Banana</li>
      <li>Orange</li>
    </ul>
  </body>
  <script>
    const $fruits = document.getElementById('fruits')

    // 이미 존재하는 요소 노드를 취득
    const [$apple, $banana] = $fruits.children;

    // 이미 존재하는 $apple 요소 노드를 #fruits 요소 노드의 마지막 노드로 이동
    $fruits.appendChild($apple);

    $fruits.insertBefore($banana, $fruits.lastElementChild);
    // Orange - Banana - Apple
  </script>
</html>
```

## 6.7 노드 복사

`Node.prototype.cloneNode([deep: true | false])` 메서드는 노드의 사본을 생성하여 반환한다.

매개변수 deep에 true를 인수 전달하면 노드를 깊은 복사하여 모든 자손 노드가 포함된 사본을 생성한다.

얕은 복사로 생성된 요소 노드는 자손 노드를 복사하지 않으므로 텍스트 노드도 없다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <ul id="fruits">
      <li>Apple</li>
    </ul>
  </body>
  <script>
    const $fruits = document.getElementById('fruits');
    const $apple = $fruits.firstElementChild;

    // $apple 요소를 얕은 복사하여 사본을 생성. 텍스트 노드가 없는 사본이 생성
    const $shallowClone = $apple.cloneNode();
    // 사본 요소 노드에 텍스트 추가
    $shallowClone.textContent = 'Banana';
    // 사본 요소 노드를 #fruits 요소 노드의 마지막 노드로 추가
    $fruits.appendChild($shallowClone);

    // #fruits 요소를 깊은 복사하여 모든 자손 노드가 포함된 사본을 생성
    const $deepClone = $fruits.cloneNode(true);
    // 사본 요소 노드를 #fruits 요소 노드의 마지막 노드로 추가
    $fruits.appendChild($deepClone);
  </script>
</html>
```

![image](https://github.com/kses1010/sunny-devlog/assets/49144662/ae8f64f6-8b8f-45c3-b765-d62fdb672981)

## 6.8 노드 교체

`Node.prototype.replaceChild(newChild, oldChild)` 메서드는 자신을 호출한 노드의 자식 노드를 다른 노드로 교체한다.

첫 번째 매개변수 newChild에는 교체할 새로운 노드를 인수로 전달하고, 두 번째 매개변수 oldChild에는 이미 존재하는 교체될 노드를 인수로 전달한다.

oldChild 매개변수에 인수로 전달한 노드는 replaceChild 메서드를 호출한 노드의 자식 노드이어야 한다.

→ replaceChild 메서드는 자신을 호출한 노드의 자식 노드인 oldChild 노드를 newChild 노드로 교체한다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <ul id="fruits">
      <li>Apple</li>
    </ul>
  </body>
  <script>
    const $fruits = document.getElementById('fruits');

    // 기존 노드와 교체할 요소 노드를 생성
    const $newChild = document.createElement('li');
    $newChild.textContent = 'Banana';

    // #fruits 요소 노드의 첫 번째 요소 노드를 $newChild 요소 노드로 교체
    $fruits.replaceChild($newChild, $fruits.firstElementChild);
  </script>
</html>
```

## 6.9 노드 삭제

`Node.prototype.removeChild(child)` 메서드는 child 매개변수에 인수로 전달한 노드를 DOM에서 삭제한다.

인수로 전달한 노드는 removeChild 메서드를 호출한 노드의 자식 노드이어야 한다.

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <ul id="fruits">
      <li>Apple</li>
      <li>Banana</li>
    </ul>
  </body>
  <script>
    const $fruits = document.getElementById('fruits');

    // #fruits 요소 노드의 마지막 요소를 삭제
    $fruits.removeChild($fruits.lastElementChild);
  </script>
</html>
```
