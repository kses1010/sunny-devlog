---
title: 'Kotlin In Action - 4장 클래스, 객체, 인터페이스'
date: 2022-10-24
category: 'kotlin'
draft: false
---

# 0. 자바와 코틀린의 클래스, 인터페이스

코틀린의 클래스와 인터페이스는 자바 클래스, 인터페이스와는 약간 다르다.

예를 들어 인터페이스에 프로퍼티 선언이 들어갈 수 있다. 자바와 달리 코틀린 선언은 기본적으로 `final` 이며 `public` 이다. 게다가 중첩 클래스는 기본적으로는 내부 클래스가 아니다.

즉, 코틀린 중첩 클래스에는 외부 클래스에 대한 참조가 없다.

짧은 주 생성자 구문으로도 거의 모든 경우를 잘 처리할 수 있다. 하지만 복잡한 초기화 로직을 수행하는 경우를 대비한 완전한 문법도 있다.

코틀린 컴파일러는 번잡스러움을 피하기 위해 유용한 메소드를 자동으로 만들어준다. 클래스를 `data` 로 선언하면 컴파일러가 일부 표준 메서드를 생성해준다.

코틀린 언어가 제공하는 위임을 사용하면 위임을 처리하기 위한 준비 메소드를 직접 작성할 필요가 없다.

# 1. 클래스 계층 정의

## 1.1 코틀린 인터페이스

간단한 인터페이스 선언하기

```kotlin
interface Clickable {
    fun click()
}

class Button : Clickable {
    override fun click() = println("클릭함")
}

fun main() {
    val button = Button()
    println(button.click())
}
```

코틀린에서는 콜론(:)을 붙이고 인터페이스와 클래스 이름을 적는 것으로 클래스 확장과 인터페이스 구현을 모두 처리한다.

`override` 변경자는 상위 클래스나 상위 인터페이스에 있는 프로퍼티나 메소드를 오버라이드한다는 표시다.

코틀린에서는 이 변경자를 꼭 사용해야 한다.

인터페이스 메소드도 디폴트 구현을 제공할 수 있다. 그런 경우 메소드 앞에 `default` 를 붙일 필요 없이 메소드 본문을 메소드 시그니처 뒤에 추가하면 된다.

```kotlin
interface Clickable {
    fun click()
    fun showOff() = println("I'm Clickable")
}
```

이 인터페이스를 구현하는 클래스는 `click` 에 대한 구현을 제공해야 한다.

`showOff` 메소드의 경우 새로운 동작을 정의할 수도 있고, 그냥 정의를 생략해서 디폴트 구현을 사용할 수 있다.

- 동일한 메소드를 구현하는 다른 인터페이스 정의

```kotlin
interface Focusable {
    fun setFocus(b: Boolean) = println("I ${if (b) "got" else "lost"} focus.")
    fun showOff() = println("I'm focusable!")
}
```

한 클래스에서 이 두 인터페이스를 함께 구현하게 된다면 어느쪽도 선택되지 않는다.

코틀린 컴파일러는 두 메소드를 아우르는 구현을 하위 클래스에 직접 구현하게 강제한다.

```kotlin
class Button : Clickable, Focusable {
    override fun click() = println("클릭함")

    override fun showOff() {
        super<Clickable>.showOff()
        super<Focusable>.showOff()
    }
}
```

- 이름과 시그니처가 같은 멤버 메소드에 대해 둘 이상의 디폴트 구현이 있는 경우 인터페이스를 구현하는 하위 클래스에서 명시적으로 새로운 구현을 제공해야 한다.
- 상위 타입의 이름을 꺾쇠 괄호 `<>` 사이에 넣어서 `super` 를 지정하면 어떤 상위 타입의 멤버 메소드를 호출할지 지정할 수 있다.

```kotlin
fun main(args: Array<String>) {
    val button = Button()
    button.showOff()
    button.click()
    button.setFocus(true)
    button.click()
}

// 출력
I'm Clickable
I'm focusable!
클릭함
I got focus.
클릭함
```

## 1.2 open, final, abstract 변경자: 기본적으로 final

코틀린의 클래스와 메소드는 기본적으로 `final` 이다.

어떤 클래스의 상속을 허용하려면 클래스 앞에 `open` 변경자를 붙여야 한다. 그와 더불어 오버라이드를 허용하고 싶은 메소드나 프로퍼티의 앞에도 `open` 변경자를 붙여야 한다.

```kotlin
open class RichButton : Clickable{

    fun disable() {} // 이 함수는 파이널

    open fun animate() {} // 이 함수는 열려있다. 오버라이드가 가능

    override fun click() {} // 기본적으로 열려있음
}
```

오버라이드하는 메소드의 구현을 하위 클래스에서 오버라이드하지 못하게 금지하려면 `final` 을 명시해야 한다.

```kotlin
final override fun click() {}
```

### 열린 클래스와 스마트 캐스트

`final` 로 함으로써 얻을 수 있는 큰 이익은 다양한 경우에 스마트 캐스트가 가능하다는 점이다. 스마트 캐스트는 타입 검사 뒤에 변경될 수 없는 변수에만 적용 가능하다.

클래스 프로퍼티의 경우 이는 `val` 이면서 커스텀 접근자가 없는 경우에만 스마트 캐스트를 쓸 수 있다.

프로퍼티가 `final` 이 아니라면 그 프로퍼티를 다른 클래스가 상속하면서 커스텀 접근자를 정의함으로써 스마트 캐스트의 요구 사항을 깰 수 있다.

프로퍼티는 기본적으로 `final` 이기 때문에 따로 고민할 필요 없이 대부분의 프로퍼티를 스마트 캐스트에 활용할 수 있다.

---

코틀린에서도 클래스를 `abstract` 선언이 가능하다.

추상 멤버는 항상 열려있기 때문에 `open` 변경자를 명시할 필요가 없다.

```kotlin
abstract class Animated {

    abstract fun animate() // 이 함수는 구현이 없다.

	// 비추상 함수는 기본적으로 파이널이지만 원한다면 open으로 오버라이드가 가능하다.
	open fun stopAnimating() {}

    fun animateTwice() {}
}
```

인터페이스 멤버의 경우 `final`, `open`, `abstract` 를 사용하지 않는다.

인터페이스 멤버는 항상 열려 있으며 `final` 로 변경할 수 없다. 인터페이스 멤버에게 본문이 없으면 자동으로 추상 멤버가 되지만, 그렇더라도 따로 멤버 선언 앞에 `abstract` 키워드를 붙일 필요가 없다.

## 1.3 가시성 변경자: 기본적으로 공개

가시성 변경자(visibility modifier)는 코드 기반에 있는 선언에 대한 클래스 외부 접근을 제어한다.

어떤 클래스의 구현에 대한 접근을 제한함으로써 그 클래스에 의존하는 외부 코드를 깨지 않고도 클래스 내부 구현을 변경할 수 있다.

아무 변경자도 없는 경우 선언은 모두 공개(public) 된다. / 패키지를 가시성 제어에 사용하지 않음.

패키지 전용 가시성에 대한 대안으로 코틀린에는 `internal` 이라는 새로운 가시성 변경자를 도입했다.

`internal` 은 “모듈 내부에서만 볼 수 있음" 이라는 뜻이며 모듈은 한 번에 한꺼번에 컴파일되는 코틀린 파일들을 의미한다.

코틀린에서는 최상위 선언에 대해 `private` 가시성을 허용한다. 최상위 선언에는 클래스, 함수, 프로퍼티 등이 포함된다.

비공개 가시성인 최상위 선언은 그 선언이 들어있는 파일 내부에서만 사용할 수 있다.

다음은 가시성 규칙을 위반한다.

```kotlin
internal open class TalkativeButton : Focusable {
    private fun yell() = println("Hey!")
    protected fun whisper() = println("Let's talk")
}

fun TalkativeButton.giveSpeech() { // public 멤버가 자신의 internal 수신 타입인 TalkativeButton 을 추천함
    yell()                          // 접근 불가 private 멤버
    whisper()                       // protected 멤버라 접근 불가
}
```

코틀린 `public` 함수인 `giveSpeech` 안에서 그보다 가시성이 더 낮은 타입인 `TalkativeButton` 을 참조하지 못한다.

여기서 컴파일 오류를 없애려면 `giveSpeech` 확장 함수의 가시성을 `internal` 로 바꾸거나 `TalkativeButton` 클래스의 가시성을 `public` 으로 바꿔야한다.

```kotlin
open class TalkativeButton : Focusable {
    fun yell() = println("Hey!")
    fun whisper() = println("Let's talk")
}

fun TalkativeButton.giveSpeech() { // public 멤버가 자신의 internal 수신 타입인 TalkativeButton 을 추천함
    yell()                          // 접근 불가 private 멤버
    whisper()                       // protected 멤버라 접근 불가
}
```

## 1.4 내부 클래스와 중첩된 클래스: 기본적으로 중첩 클래스

코틀린의 중첩 클래스는 명시적으로 요청하지 않는 한 바깥쪽 클래스 인스턴스에 대한 접근 권한이 없다.

- 직렬화할 수 있는 상태가 있는 뷰 선언

```kotlin
interface State : Serializable {
}

interface View {
    fun getCurrentState() : State
    fun restoreState(state: State) {}
}
```

- 중첩 클래스를 사용해 코틀린에서 View 구현하기

```kotlin
class Button : View {
    override fun getCurrentState() : State = ButtonState()

    override fun restoreState(state: State) {
        super.restoreState(state)
    }

    class ButtonState : State // 자바의 정적 중첩 클래스
}
```

| 클래스 B 안에 정의된 클래스 A                        | 자바           | 코틀린        |
| ---------------------------------------------------- | -------------- | ------------- |
| 중첩 클래스(바깥쪽 클래스에 대한 참조 저장하지 않음) | static class A | class A       |
| 내부 클래스(바깥쪽 클래스에 대한 참조 저장)          | class A        | inner class A |

코틀린에서 바깥쪽 클래스의 인스턴스를 가리키는 참조를 표기하는 방법도 자바와 다르다.

내부 클래스 `inner` 안에서 바깥쪽 클래스 `Outer` 참조에 접근하려면 `this@Outer` 라고 한다.

```kotlin
class Outer {
    inner class Inner {
        fun getOuterReference() : Outer = this@Outer
    }
}
```

## 1.5 봉인된 클래스: 클래스 계층 정의 시 계층 확장 제한

상위 클래스 `Expr` 에는 숫자를 표현하는 `Num` 과 덧셈 연산을 표현하는 `Sum` 이라는 두 하위 클래스가 있다.

`when` 식에서 이 모든 하위 클래스를 처리하면 편리하다.

하지만 `when` 식에서 `Num` 과 `Sum` 이 아닌 경우를 처리하는 `else` 분기를 반드시 넣어야 한다.

```kotlin
interface Expr

class Num(val value: Int) : Expr
class Sum(val left: Expr, val right: Expr) : Expr

fun eval(e: Expr): Int =

    when (e) {
        is Num -> e.value
        is Sum -> eval(e.right) + eval(e.left)
        else -> throw IllegalArgumentException("Unknown expression")
    }
```

코틀린 컴파일러는 `when` 을 사용해 `Expr` 타입의 값을 검사할 때 꼭 디폴트 분기인 `else` 분기를 덧붙이게 강제한다. 이 예제의 `else` 분기에서는 반환할 만한 의미 있는 값이 없으므로 예외를 던진다.

항상 디폴트 분기를 추가하는 게 편하지는 않다. 그리고 디폴트 분기가 있으면 이런 클래스 계층에 새로운 하위 클래스를 추가하더라도 컴파일러가 `when` 이 모든 경우를 처리하는지 제대로 검사할 수 없다.

코틀린은 `sealed` 클래스를 사용한다. 상위 클래스에 `sealed` 변경자를 붙이면 그 상위 클래스를 상속한 하위 클래스 정의를 제한할 수 있다. `sealed` 클래스의 하위 클래스를 정의할 떄는 반드시 상위 클래스 안에 중첩시켜야 한다.

```kotlin
sealed class Expr {
    class Num(val value: Int) : Expr()
    class Sum(val left: Expr, val right: Expr) : Expr()
}

fun eval(e:Expr) : Int =
    when(e) {
        is Expr.Num -> e.value
        is Expr.Sum -> eval(e.right) + eval(e.left)
    } // else 분기 필요가 없다.
```

`sealed` 클래스는 자동으로 `open` 이다.

`sealed` 클래스에 속한 값에 대해 디폴트 분기를 사용하지 않고 `when` 식을 사용하면 나중에 `sealed` 클래스의 상속 계층에 새로운 하위 클래스를 추가해도 `when` 식이 컴파일되지 않는다.

내부적으로 `Expr` 클래스는 `private` 생성자를 가진다. 그 생성자는 클래스 내부에서만 호출할 수 있다.

`sealed` 인터페이스를 정의할 수는 없다.

# 2. 뻔하지 않은 생성자와 프로퍼티를 갖는 클래스 선언

코틀린은 주 생성자와 부 생성자를 구분한다.

또한 코틀린에서는 초기화 블록을 통해 초기화 로직을 추가할 수 있다.

## 2.1 클래스 초기화: 주 생성자와 초기화 블록

```kotlin
class User(val nickname: String)
```

클래스 이름 뒤에 오는 괄호로 둘러싸인 코드를 주 생성자라고 한다.

주 생성자는 생성자 파라미터를 지정하고 그 생성자 파라미터에 의해 초기화되는 프로퍼티를 정의하는 두 가지 목적에 쓰인다.

```kotlin
class User constructor(_nickname : String) { // 파라미터가 하나만 있는 주 생성자
    val nickName: String

    init { // 초기화 블록
        nickName = _nickname
    }
}
```

`constructor` 키워드는 주 생성자와 부 생성자 정의를 시작할 때 사용한다.

`init` 키워드는 초기화 블록을 시작한다. 초기화 블록에는 클래스의 객체가 만들어질 때 실행되는 초기화 코드가 들어간다.

초기화 블록은 주 생성자와 함께 사용된다. 주 생성자는 제한적이기 때문에 별도의 코드를 포함할 수 없으므로 초기화 블록이 필요하다. 필요하다면 클래스 안에 여러 초기화 블록을 선언할 수 있다.

생성자 파라미터 `_nickname` 에서 맨 앞의 밑줄(`_`)은 프로퍼티와 생성자 파라미터를 구분해준다.

`this` 와 같은 역할

주 생성자 앞에 별다른 애노테이션이나 가시성 변경자가 없다면 `constructor` 를 생략해도 된다.

다음처럼 바꿀 수 있다.

```kotlin
class User constructor(_nickname : String) {    // 파라미터가 하나뿐인 주 생성자
    val nickname = _nickname // 프로퍼티를 주 생성자의 파라미터로 초기화
}
```

같은 클래스를 정의하는 여러 방법 중 하나.

프로퍼티를 초기화하는 식이나 초기화 블록 안에서만 주 생성자의 파라미터를 참조할 수 있다.

주 생성자의 파라미터로 프로퍼티를 초기화한다면 그 주 생성자 파라미터 이름앞에 `val` 을 추가하는 방식으로 프로퍼티 정의와 초기화를 간략히 쓸 수 있다.

```kotlin
class User(val nickname: String)
```

디폴트 값도 지정이 가능하다.

```kotlin
class User(val nickname: String, val isSubscribed: Boolean = true)

fun main() {
    val hyun = User("현석")
    println(hyun.isSubscribed)
}
// true
```

클래스에 기반 클래스가 있다면 주 생성자에서 기반 클래스의 생성자를 호출해야 할 필요가 있다.

기반 클래스를 초기화하려면 기반 클래스 이름 뒤에 괄호를 치고 생성자 인자를 넘긴다.

```kotlin
open class User(val nickname: String, val isSubscribed: Boolean = true)

class TwitterUser(nickname: String) : User(nickname) {}
```

클래스를 정의할 때 별도로 생성자를 정의하지 않으면 컴파일러가 자동으로 아무 일도 하지 않는 인자가 없는 디폴트 생성자를 만들어준다.

`Button` 클래스를 상속한 하위 클래스는 반드시 `Button` 생성자를 호출해야 한다.

```kotlin
open class Button
class RadioButton: Button()
```

이 규칙으로 인해 기반 클래스의 이름 뒤에는 꼭 빈 괄호가 들어간다.

반면 인터페이스는 생성자가 없기 때문에 어떤 클래스가 인터페이스를 구현하는 경우 그 클래스의 상위 클래스 목록에 있는 인터페이스 이름 뒤에는 아무 괄호도 없다.

클래스 정의에 있는 상위 클래스 및 인터페이스 목록에서 이름 뒤에 괄호가 붙었는지 살펴보면 쉽게 기반 클래스와 인터페이스를 구별할 수 있다.

어떤 클래스를 클래스 외부에서 인스턴스화하지 못하게 막고 싶다면 모든 생성자를 `private` 로 만들면 된다.

```kotlin
class Secretive private constructor() {}
```

`Secretive` 클래스 안에는 주 생성자밖에 없고 그 주 생성자는 비공개이므로 외부에서는 `Secretive` 를 인스턴스화 할 수 없다.

## 2.2 부 생성자: 상위 클래스를 다른 방식으로 초기화

```kotlin
open class View {
    constructor(ctx: Context) {

    }

    constructor(ctx: Context, attr AttributeSet) {

    }

}
```

이 클래스는 주 생성자를 선언하지 않고 부 생성자만 2가지 선언한다. 부 생성자는 `constructor` 키워드로 시작한다. 필요에 따라 얼마든지 부 생성자를 많이 선언해도 된다.

다음은 확장이다.

```kotlin
class MyButton: View {
    constructor(ctx: Context) : super(ctx) {

    }
    constructor(ctx: Context, attr: AttributeSet) : super(ctx, attr) {

    }
}
```

자바와 마찬가지로 생성자에서 `this()` 를 통해 클래스 자신의 다른 생성자를 호출할 수 있다.

```kotlin
class MyButton: View {
    constructor(ctx: Context) : this(ctx, MY_STYLE) {

    }
    constructor(ctx: Context, attr: AttributeSet) : super(ctx, attr) {

    }
}
```

`MyButton` 클래스의 생성자 중 하나가 파라미터의 디폴트 값을 넘겨서 같은 클래스의 다른 생성자에게 생성을 위임한다. 두번째 생성자는 여전히 `super()` 를 호출한다.

클래스에 주 생성자가 없다면 모든 부 생성자는 반드시 상위 클래스를 초기화하거나 다른 생성자에게 생성을 위임해야 한다.

부 생성자가 필요한 주된 이유는 자바 상호운용성이다.

## 2.3 인터페이스에 선언된 프로퍼티 구현

코틀린에서는 인터페이스에 추상 프로퍼티 선언을 넣을 수 있다.

```kotlin
interface User {
    val nickname: String
}
```

`User` 인터페이스를 구현하는 클래스가 `nickname` 의 값을 얻을 수 있는 방법을 제공해야 한다.

인터페이스에 있는 프로퍼티 선언에는 뒷받침하는 필드나 게터 등의 정보가 들어있지 않다.

사실 인터페이스는 아무 상태도 포함할 수 없으므로 상태를 저장할 필요가 있다면 인터페이스를 구현한 하위 클래스에서 상태 저장을 위한 프로퍼티 등을 만들어야 한다.

- 인터페이스의 프로퍼티 구현하기

```kotlin
// 주 생성자에 있는 프로퍼티
class PrivateUser(override val nickname: String) : User

class SubscribingUser(val email: String) : User {
    override val nickname: String
        get() = email.substringBefore('@') // 커스텀 게터
}

class FacebookUser(val accountId: Int) : User {
		// 프로퍼티 초기화 식
    override val nickname = getFacebookName(accountId)
}

fun main() {
    println(PrivateUser("test@kotlinlang.org").nickname)
    println(SubscribingUser("test@kotlinlang.org").nickname)
}

//test@kotlinlang.org
//test
```

`PrivateUser` 는 주 생성자 안에 있는 프로퍼티를 직접 선언하는 간결한 구문을 사용한다.

이 프로퍼티는 `User` 의 추상 프로퍼티를 구현하고 있으므로 `override` 를 표시해야 한다.

`SubscribingUser` 는 커스텀 게터로 nickname 프로퍼티를 설정한다. 이 프로퍼티는 뒷받침하는 필드에 값을 저장하지 않고 매번 이메일 주소에서 별명을 계산한다.

`FacebookUser` 에서는 초기화 식으로 `nickname` 값을 초기화 한다. 이 때 페이스북 사용자 ID를 받아서 그 사용자의 이름을 반환해주는 `getFacebookName` 함수(다른곳에 정의돼 있다고 가정)를 호출해서 `nickname` 을 초기화 한다.

`getFacebookName` 은 페이스북에 접근해서 인증을 거친 후 원하는 데이터를 가져와야 하기 때문에 비용이 많이 들 수 도 있다.

- `SubscribingUser` 의 `nickname` : 매번 호출될 때마다 `substringBefore` 를 호출해 계산하는 커스템 게터를 사용한다.
- `FacebookUser` 의 `nickname` : 객체 초기화 시 계산한 데이터를 뒷받침하는 필드에 저장했다가 불러오는 방식을 활용.

인터페이스에는 추상 프로퍼티뿐 아니라 게터와 세터가 있는 프로퍼티를 선언할 수 있다. 물론 그런 게터와 세터는 뒷받침하는 필드를 참조할 수 없다.

```kotlin
interface User {
    val email: String
    val nickname: String
    get() = email.substringBefore('@') // 프로퍼티에 뒷받침하는 필드가 없어 매번 계산하여 돌려줌.
}
```

`User` 인터페이스는 추상 프로퍼티인 `email` 과 커스텀 게터가 있는 `nickname` 프로퍼티가 함께 들어있다.

하위 클래스는 추상 프로퍼티인 `email` 을 반드시 오버라이드해야 한다. 반면 `nickname` 은 오버라이드하지 않고 상속할 수 있다. 인터페이스에 선언된 프로퍼티와 달리 클래스에 구현된 프로퍼티는 뒷받침하는 필드를 원하는 대로 사용할 수 있다.

## 2.4 게터와 세터에서 뒷받침하는 필드에 접근

값을 저장하는 동시에 로직을 실행할 수 있게 하기 위해서는 접근자 안에서 프로퍼티를 뒷받침하는 필드에 접근할 수 있어야 한다.

다음은 값의 변경 이력을 로그에 남기는 경우다.

```kotlin
class User(val name: String) {
    var address: String = "unspecified"
        set(value: String) {
           println("""
               Address was changed for $name:
               "$field" -> "$value".""".trimIndent()) // 뒷받침하는 필드 값 읽기
            field = value   // 뒷받침하는 필드 값 변경하기
        }
}

fun main() {
    val user = User("Alice")
    user.address = "Elsenheimerstrasse 47, 80687 Muehchen"
    println(user.address)
}

// Address was changed for Alice:
// "unspecified" -> "Elsenheimerstrasse 47, 80687 Muehchen".
// Elsenheimerstrasse 47, 80687 Muehchen
```

코틀린에서 프로퍼티의 값을 바꿀 때는 `user.address = "new value"` 처럼 필드 설정 구문을 사용한다.

이 구문은 내부적으로는 `address` 의 세터를 호출한다.

접근자의 본문에서는 `field` 라는 특별한 식별자를 통해 뒷받침하는 필드에 접근할 수 있다. 게터에서는 `field` 값을 읽을 수만 있고, 세터에서는 `field` 값을 읽거나 쓸 수 있다.

변경 가능 프로퍼티의 게터와 세터 중 한쪽만 직접 정의해도 된다는 점을 기억해야한다.

`address` 의 게터는 필드 값을 그냥 반환해주는 뻔한 게터다. → 굳이 직접 정의할 필요가 없음

---

## 2.5 접근자의 가시성 변경

```kotlin
class LengthCounter {
    var counter: Int = 0
        private set // 이 클래스 밖에서 이 프로퍼티의 값을 바꿀 수 없음
    fun addWord(word: String) {
        counter += word.length
    }
}

fun main() {
    val lengthCounter = LengthCounter()
    lengthCounter.addWord("Hi!")
    println(lengthCounter.counter)
}
// 3
```

전체 길이를 저장하는 프로퍼티는 클라이언트에게 제공하는 API의 일부분이므로 `public` 으로 외부에 공개된다.

하지만 외부 코드에서 단어 길이의 합을 마음대로 바꾸지 못하게 이 클래스 내부에서만 길이를 변경하고자 한다면 기본 가시성을 가진 게터를 컴파일러가 생성하게 내버려 두는 대신 세터의 가시성을 `private` 으로 지정한다.

# 3. 컴파일러가 생성한 메소드: 데이터 클래스와 클래스 위임

코틀린 컴파일러는 `equals`, `hashCode`, `toString` 등을 기계적으로 생성해준다.

## 3.1 모든 클래스가 정의해야 하는 메소드

- `Client` 클래스의 초기 정의

```kotlin
class Client(val name: String, val postalCode: Int)
```

### 문자열 표현 : `toString()`

`toString` 메소드를 오버라이드 하면 된다.

```kotlin
class Client(val name: String, val postalCode: Int) {
    override fun toString() = "Client (name=$name, postalCode=$postalCode)"
}

fun main() {
    val client1 = Client("오현석", 4122)
    println(client1)
}

//Client (name=오현석, postalCode=4122)
```

### 객체의 동등성: `equals()`

서로 다른 두 객체가 내부에 동일한 데이터를 포함하는 경우 그 둘을 동등한 객체로 간주해야할 때가 있다.

```kotlin
fun main() {
    val client1 = Client("오현석", 4122)
    val client2 = Client("오현석", 4122)
    println(client1 == client2) // == 코틀린에서는 equals 호출
}
// false
```

다음은 `equals` 를 추가한 `Client` 클래스이다.

```kotlin
class Client(val name: String, val postalCode: Int) {
    override fun toString() = "Client (name=$name, postalCode=$postalCode)"

    override fun equals(other: Any?): Boolean {
        if (other == null || other !is Client)
            return false
        return name == other.name &&
             postalCode == other.postalCode
    }
}
```

- `Any` : `java.lang.Object` 에 대응하는 클래스로, 코틀린의 모든 클래스의 최상위 클래스다. `Any?` 는 널이 될 수 있는 타입이므로 `other` 은 `null` 일 수 있다.

코틀린의 `is` 검사는 `instanceof` 와 같다. 이제 구현했을 경우 true를 반환한다.

그러나 hashCode 정의가 빠져서 제대로 동작을 안할 수 있으니 구현해보자.

### 해시 컨테이너: `hashCode()`

프로퍼티가 모두 일치하므로 새 인스턴스와 집합(set)에 있는 기존 인스턴스가 동등한가 테스트이다.

```kotlin
val processed = hashSetOf(Client("오현석", 4122))
println(processed.contains(Client("오현석", 4122)))
// false
```

원소 객체들이 해시 코드에 대한 규칙을 지키지 않은 경우 `HashSet` 은 제대로 동작하지 않는다.

```kotlin
override fun hashCode(): Int {
        var result = name.hashCode()
        result = 31 * result + postalCode
        return result
    }
```

## 3.2 데이터 클래스: 모든 클래스가 정의해야 하는 메소드 자동 생성

코틀린에서는 이젠 이 메소드를 생성할 필요가 없다. `data` 라는 변경자를 붙이면 컴파일러가 자동으로 생성한다.

```kotlin
data class Client(val name: String, val postalCode: Int)
```

### 데이터 클래스와 불변성: `copy()` 메소드

데이터 클래스의 프로퍼티가 꼭 `val` 일 필요는 없다. `var` 를 사용해도 되나 데이터 클래스의 모든 프로퍼티를 읽기 전용으로 만들어서 데이터 클래스를 불변 클래스로 만들라고 권장한다.

`HashMap` 등의 컨테이너에 데이터 클래스 객체를 담는 경우엔 불변성이 필수적이다. 다중스레드 프로그램의 경우 이런 성질은 더욱 중요하다.

코틀린 컴파일러는 한가지 편의 메소드를 제공한다.

객체를 복사하면서 일부 프로퍼티를 바꿀 수 있게 해주는 `copy` 메소드다. 객체를 메모리상에서 직접 바꾸는 대신 복사본을 만드는 편이 더 낫다. 복사본은 원본과 다른 생명주기를 가지며, 복사를 하면서 일부 프로퍼티 값을 바꾸거나 복사본을 제거해도 프로그램에서 원본을 참조하는 다른 부분에 전혀 영향을 끼치지 않는다.

```kotlin
fun copy(name: String = this.name, postalCode: Int = this.postalCode) =
		Client(name, postalCode) // 이미 이렇게 구현되어있음

fun main() {
    val lee = Client("이씨", 4122)
    println(lee.copy(postalCode = 4000))
}
//Client(name=이씨, postalCode=4000)
```

## 3.3 클래스 위임: by 키워드 사용

대규모 객체지향 시스템을 설계할 때 시스템을 취약하게 만드는 문제는 보통 구현 상속에 의해 발생한다.

하위 클래스가 상위 클래스의 메소드 중 일부를 오버라이드하면 하위 클래스의 상위 클래스의 세부 구현 사항에 의존하게 된다. 시스템이 변함에 따라 상위 클래스의 구현이 바뀌거나 상위 클래스에 새로운 메소드가 추가된다.

그 과정에서 하위 클래스가 상위 클래스에 대해 갖고 있던 가정이 깨져서 코드가 정상적으로 작동하지 못하는 경우가 발생할 수 있다.

코틀린은 기본적으로 클래스를 `final` 로 취급한다. `open` 변경자로 열어둔 클래스만 확장할 수 있다.

하지만 종종 상속을 허용하지 않는 클래스에 새로운 동작을 추가해야 할 때가 있다. 이럴 때 사용하는 방법이 데코레이터 패턴이다.

상속을 허용하지 않는 클래스 대신 사용할 수 있는 새로운 클래스를 만들되 기존 클래스와 같은 인터페이스를 데코레이터가 제공하게 만들고, 기존 클래스를 데코레이터 내부에 필드로 유지하는 것이다.

이때 새로 정의해야 하는 기능은 데코레이터의 메소드에 새로 정의하고 기존 기능이 그대로 필요한 부분은 데코레이터의 메소드가 기존 클래스의 메소드에게 요청을 전달한다.

단점은 준비 코드가 상당히 많이 필요한다는 점이다. `Collection` 같이 비교적 단순한 인터페이스를 구현하면서 아무 동작도 변경하지 않는 데코레이터를 만들 때조차도 다음과 같이 복잡한 코드를 작성해야 한다.

```kotlin
class DelegatingCollection<T> : Collection<T> {
    private val innerList = arrayListOf<T>()
    override val size: Int
        get() = TODO("Not yet implemented")

    override fun contains(element: T): Boolean {
        TODO("Not yet implemented")
    }

    override fun containsAll(elements: Collection<T>): Boolean {
        TODO("Not yet implemented")
    }

    override fun isEmpty(): Boolean {
        TODO("Not yet implemented")
    }

    override fun iterator(): Iterator<T> {
        TODO("Not yet implemented")
    }
}
```

이런 위임을 언어가 제공하는 일급 시민 기능으로 지원한다는 점이 코틀린의 장점이다.

인터페이스를 구현할 때 `by` 키워드를 통해 그 인터페이스에 대한 구현을 다른 객체에 위임 중이라는 사실을 명시 할 수 있다. 다음은 위임을 이용하여 재작성한 코드다.

```kotlin
class DelegatingCollection<T>(innerList: Collection<T> = ArrayList())
    : Collection<T> by innerList {

    }
```

컴파일러가 전달 메소드를 자동으로 생성하며 자동 생성한 코드의 구현은 `DelegatingCollection` 에 있던 구현과 비슷하다.

메소드 중 일부의 동작을 변경하고 싶은 경우 메소드를 오버라이드하면 컴파일러가 생성한 메소드 대신 오버라이드한 메소드가 쓰인다. 기존 클레스의 메소드에 위임하는 기본 구현으로 충분한 메소드는 따로 할 필요가 없다.

다음은 원소를 추가하려고 시도한 횟수를 기록하는 컬렉션을 구현한 것이다.

예를 들어 중복을 제거하는 프로세스를 설계하는 중이라면 원소 추가 횟수를 기록하는 컬렉션을 통해 최종 컬렉션 크기와 원소 추가 시도 횟수 사이의 비율을 살펴봄으로써 중복 제거 프로세스의 효율성을 판단할 수 있다.

```kotlin
class CountingSet<T>(
    val innerSet: MutableCollection<T> = HashSet()
) : MutableCollection<T> by innerSet { // MutableCollection 의 구현을 innerSet에게 위임.
    var objectsAdded = 0

    override fun add(element: T): Boolean {
        objectsAdded++
        return innerSet.add(element)
    }

    override fun addAll(elements: Collection<T>): Boolean {
        objectsAdded += elements.size
        return innerSet.addAll(elements)
    }
}

fun main() {
    val cset = CountingSet<Int>()
    cset.addAll(listOf(1, 1, 2))
    println("${cset.objectsAdded} objects were added, ${cset.size} remain")
}
// 3 objects were added, 2 remain
```

`MutableCollection` 인터페이스의 나머지 메소드는 내부 컨테이너(`innerSet`) 에게 위임한다.

# 4. object 키워드: 클래스 선언과 인스턴스 생성

`object` 키워드를 다양한 상황에서 사용하지만 모든 경우 클래스를 정의하면서 동시에 인스턴스를 생성한다는 공통점이 있다. `object` 키워드를 사용하는 여러 상황은 다음과 같다.

- 객체 선언(object declaration): 싱글턴을 정의하는 방법 중 하나다.
- 동반 객체(companion object): 인스턴스 메소드는 아니지만 어떤 클래스와 관련 있는 메소드와 팩토리 메소드를 담을 때 쓰인다. 동반 객체 메소드에 접근할 때는 동반 객체가 포함된 클래스의 이름을 사용할 수 있다.
- 객체 식은 자바의 무명 내부 클래스(anonymous inner class) 대신 쓰인다.

## 4.1 객체 선언: 싱글턴을 쉽게 만들기

객체지향 시스템을 설계하다 보면 인스턴스가 하나만 필요한 클래스가 유용한 경우가 많다. 자바에서는 보통 클래스의 생성자를 `private` 으로 제한하고 정적인 필드에 그 클래스의 유일한 객체를 저장하는 싱글턴 패턴을 통해 이를 구현한다.

코틀린은 객체 **선언 기능**을 통해 싱글턴을 언어에서 기본 지원한다. 객체 선언은 클래스 선언과 그 클래스에 속한 단일 인스턴스의 선언을 합친 선언이다.

예를 들어 객체 선언을 사용해 회사 급여 대장을 만들 수 있다. 한 회사에 여러 급여 대장이 필요하지 않을 테니 싱글턴을 쓰는게 정당해 보인다.

```kotlin
object Payroll {
    val allEmployees = arrayListOf<Person>()

    fun calculateSalary() {
        for (person in allEmployees) {
            //
        }
    }
}
```

객체 선언은 `object` 키워드로 시작한다. 객체 선언은 클래스를 정의하고 그 클래스의 인스턴스를 만들어서 변수에 저장하는 모든 작업을 단 한 문장으로 처리한다.

클래스와 마찬가지로 객체 선언 안에도 프로퍼티, 메소드, 초기화 블록 등이 들어 갈 수 있다.

하지만 생성자(주, 부 모두) 객체 선언에 쓸 수 없다. 일반 클래스 인스턴스와 달리 싱글턴 객체는 객체 선언문이 있는 위치에서 생성자 호출 없이 즉시 만들어진다. 따라서 객체 선언에는 생성자 정의가 필요 없다.

변수와 마찬가지로 객체 선언에 사용한 이름 뒤에 마침표(.)를 붙이면 객체에 속한 메소드나 프로퍼티에 접근할 수 있다.

```kotlin
Payroll.allEmployees.add(Person("김", true))
Payroll.calculateSalary()
```

객체 선언도 클래스나 인터페이스를 상속할 수 있다. 프레임워크를 사용하기 위해 특정 인터페이스를 구현해야 하는데, 그 구현 내부에 다른 상태가 필요하지 않은 경우에 이 기능이 유용하다.

예를 들어 `java.util.Comparator` 인터페이스를 살펴보자. `Comparator` 구현은 두 객체를 인자로 받아 그중 어느 객체가 더 큰지 알려주는 정수를 반환한다. `Comparator` 안에는 데이터를 저장할 필요가 없다.

→ 어떤 클래스에 속한 객체를 비교할 때 사용하는 `Comparator` 는 보통 클래스마다 단 하나씩만 있으면 된다.

→ `Comparator` 인스턴스를 만드는 방법으로는 객체 선언이 가장 좋은 방법이다.

```kotlin
object CaseInsensitiveFileComparator : Comparator<File> {
    override fun compare(o1: File, o2: File): Int {
        return o1.path.compareTo(o2.path, ignoreCase = true)
    }
}

fun main() {
    println(CaseInsensitiveFileComparator.compare(File("/User"), File("/user")))
}
```

일반 객체를 사용할 수 있는 곳에서는 항상 싱글턴 객체를 사용할 수 있다. 예를 들어 이 객체를 `Comparator` 를 인수로 받는 함수에게 인자로 넘길 수 있다.

```kotlin
val files = listOf(File("/Z"), File("/a"))
println(files.sortedWith(CaseInsensitiveFileComparator))
```

이 예제는 전달받은 `Comparator` 에 따라 리스트를 정렬하는 `sortedWith` 함수를 사용한다.

### 싱글턴과 의존관계 주입

싱글턴 패턴과 마찬가지 이유로 대규모 소프트웨어 시스템에서는 객체 선언이 항상 적합하지는 않다. 의존관계가 별로 많지 않은 소규모 소프트웨어에서는 싱글턴이나 객체 선언이 유용하지만, 시스템을 구현하는 다양한 구성 요소와 상호작용하는 대규모 컴포넌트에는 싱글턴이 적합하지 않다. 이유는 객체 생성을 제어할 방법이 없고 생성자 파라미터를 지정할 수 없어서다.

생성을 제어할 수 없고 생성자 파라미터를 지정할 수 없으므로 단위 테스트를 하거나 소프트웨어 시스템의 설정이 달라질 때 객체를 대체하거나 객체의 의존관계를 바꿀 수 없다. 따라서 그런 기능이 필요하다면 자바와 마찬가지로 의존관계 주입 프레임워크와 코틀린 클래스를 함께 사용해야 한다.

---

클래스 안에서 객체를 선언할 수 있다. 그런 객체도 인스턴스는 단 하나뿐이다.

어떤 클래스의 인스턴스를 비교하는 `Comparator` 를 클래스 내부에 정의하는게 더 바람직하다.

```kotlin
data class Person(val name: String) {
    object NameComparator : Comparator<Person> {
        override fun compare(o1: Person, o2: Person): Int {
            return o1.name.compareTo(o2.name)
        }
    }
}

fun main() {
    val persons = listOf(Person("Bob"), Person("Alice"))
    println(persons.sortedWith(Person.NameComparator))
}
```

### 코틀린 객체를 자바에서 사용하기

코틀린 객체 선언은 유일한 인스턴스에 대한 정적인 필드가 있는 자바 클래스로 컴파일된다. 이 때 인스턴스 필드의 이름은 항상 `INSTANCE` 다. 싱글턴 패턴을 자바에도 구현해도 비슷한 필드가 필요하다. 자바 코드에서 코틀린 싱글턴 객체를 사용하려면 정적인 `INSTANCE` 필드를 통하면된다.

```java
CaseInsensitiveFileComparator.INSTANCE.compare(file1,file2);
```

## 4.2 동반 객체: 팩토리 메소드와 정적 멤버가 들어갈 장소

코틀린 언어는 자바 `static` 키워드를 지원하지 않는다. 그 대신 코틀린에서는 패키지 수준의 최상위 함수와 객체 선언을 활용한다. 대부분의 경우 최상위 함수를 활용하는 편을 더 권장한다.

하지만 최상위 함수는 `private` 으로 표시된 클래스 비공개 멤버에 접근할 수 없다. 그래서 클래스의 인스턴스와 관계없이 호출해야 하지만, 클래스 내부 정보에 접근해야 하는 함수가 필요할 때는 클래스에 중첩된 객체 선언의 멤버 함수로 정의해야 한다. 대표적인 예로 팩토리 메소드를 들 수 있다.

클래스 안에 정의된 객체 중 하나에 `companion` 이라는 특별한 표시를 붙이면 그 클래스의 동반 객체로 만들 수 있다.

동반 객체의 프로퍼티나 메소드에 접근하려면 그 동반 객체가 정의된 클래스 이름을 사용한다. 이 때 객체의 이름을 따로 지정할 필요가 없다. 그 결과 동반 객체의 멤버를 사용하는 구문은 자바의 정적 메소드 호출이나 정적 필드 사용 구문과 같아진다.

```kotlin
class A {
    companion object {
        fun bar() {
            println("Companion object called")
        }
    }
}

fun main() {
    A.bar()
}
//Companion object called
```

동반 객체가 `private` 생성자를 호출하기 좋은 위치다. 동반 객체는 자신을 둘러싼 클래스의 모든 `private` 멤버에 접근할 수 있다. 따라서 동반 객체는 바깥쪽 클래스의 `private` 생성자도 호출할 수 있다.

→ 동반 객체는 팩토리 패턴을 구현하기 가장 적합한 위치다.

다음은 부 생성자가 여럿 있는 클래스 정의한 방식이다.

```kotlin
class User {
    val nickname: String

    constructor(email: String) { // 부생성자
        nickname = email.substringBefore('@')
    }

    constructor(facebookAccountId: Int) { // 부생성자
        nickname = getFacebookName(facebookAccountId)
    }
}
```

이런 로직을 표현하는 더 유용한 방법으로 클래스의 인스턴스를 생성하는 팩토리 메소드가 있다.

```kotlin
class User private constructor(val nickname: String) { // 주 생성자 비공개
    companion object { // 동반 객체 선언
        fun newSubscribingUser(email: String) = User(email.substringBefore('@'))
    }
    // 팩토리 메소드
    fun newFacebookUser(accountId: Int) = User(getFacebookName(accountId))
}

fun main {
	val subscribingUser = User.newSubscribingUser("bob@gmail.com")
	val facebookUser = User.newFacebookUser(4)
	println(subscribingUser.nickname) // bob
}
```

목적에 따라 팩토리 메소드 이름을 정할 수 있다. 게다가 팩토리 메소드는 그 팩토리 메소드가 선언된 클래스의 하위 클래스 객체를 반환할 수도 있다.

팩토리 메소드는 생성할 필요가 없는 객체를 생성하지 않을 수도 있다. 예를 들어 이메일 주소별로 유일한 `User` 인스턴스를 만드는 경우 팩토리 메소드가 이미 존재하는 인스턴스에 해당하는 이메일 주소를 전달받으면 새 인스턴스를 만들지 않고 캐시에 있는 기존 인스턴스를 반활할 수 있다.

하지만 클래스를 확장해야만 하는 경우에는 동반 객체 멤버를 하위 클래스에서 오버라이드할 수 없으므로 여러 생성자를 사용하는 편이 더 나은 해법이다.

## 4.3 동반 객체를 일반 객체처럼 사용

동반 객체는 클래스 안에 정의된 일반 객체다. 따라서 동반 객체에 이름을 붙이거나, 동반 객체가 인터페이스를 상속하거나, 동반 객체 안에 확장 함수와 프로퍼티를 정의할 수 있다.

다음은 객체를 JSON으로 직렬화하거나 역직렬화하는 코드다.

```kotlin
class Person(val name: String) {
    companion object Loader { // 동반 객체에 이름을 붙임
        fun fromJSON(jsonText: String) : Person = Person("JSON")
    }
}

fun main() {
    val person = Person.Loader.fromJSON("{name: 'Dmitry'}")
    person.name
	val person2 = Person.fromJSON("{name: 'Dmitry'}")
	person2.name
}
```

대부분의 경우 클래스 이름을 동반 객체에 속한 멤버를 참조할 수 있으므로 객체의 이름을 짓느라 고심할 필요가 없다. 하지만 필요하다면 동반 객체에도 이름을 붙일 수 있다. 특별히 이름을 지정하지 않으면 동반 객체 이름은 자동으로 `Companion` 이 된다.

### 동반 객체에서 인터페이스 구현

동반 객체도 인터페이스를 구현할 수 있다.

다음은 Person을 다음과같이 `JSONFactory` 구현으로 제공한 코드다.

```kotlin
interface JSONFactory<T> {
    fun fromJSON(jsonText: String) : T
}

class Person(val name: String) {
    companion object : JSONFactory<Person> {
        override fun fromJSON(jsonText: String) : Person = Person("JSON $jsonText")
    }


}
```

### 코틀린 동반 객체와 정적 멤버

클래스의 동반 객체는 일반 객체와 비슷한 방식으로, 클래스에 정의된 인스턴스를 가리키는 정적 필드로 컴파일된다. 동반 객체에 이름을 붙이지 않았다면 자바쪽에서 `Companion` 이라는 이름으로 그 참조에 접근할 수 있다.

```java
Person.Companion.fromJSON("...");
```

때로 자바에서 사용하기 위해 코틀린 클래스의 멤버를 정적인 멤버로 만들어야 할 필요가 있다. 그런 경우 `@JvmStatic` 애노테이션을 코틀린 멤버에 붙이면 된다.

정적 필드가 필요하다면 `@JvmField` 애노테이션을 최상위 프로퍼티나 객체에서 선언된 프로퍼티 앞에 붙인다. 이 기능은 자바와의 상호운용성을 존재하며 코틀린 핵심언어가 제공하는 기능은 아니다.

### 동반 객체 확장

클래스에 동반 객체가 있으면 그 객체 안에 함수를 정의함으로써 클래스에 대해 호출할 수 있는 확장 함수를 만들 수 있다.

```kotlin
class Person(val firstName: String, val lastName: String) {
    companion object {
        // 비어있는 동반 객체 선언
    }
}

fun Person.Companion.fromJSON(json: String) : Person { // 확장 함수 선언
    return Person("JSON $json", json)
}

fun main() {
    val p = Person.fromJSON("json")
}
```

동반 객체에 대한 확장 함수를 작성할 수 있으려면 원래 클래스에 동반 객체를 꼭 선언해야 한다. 빈 객체라도 동반 객체가 꼭 있어야 한다.

## 4.4 객체 식: 무명 내부 클래스를 다른 방식으로 작성

무명 객체는 자바의 무명 내부 클래스를 대신한다.

다음은 무명 객체로 이벤트 리스너를 구현한 코드다.

```kotlin
window.addMouseListener {
        object : MouseAdapter() { // MouseAdapter를 확장하는 무명객체 선언

						// MouseAdapter의 메소드 오버라이드함
            override fun mouseClicked(e: MouseEvent) {

            }

            override fun mouseEntered(e: MouseEvent) {
                super.mouseEntered(e)
            }
        }

fun main() {
		val listener = object : MouseAdapter () {
            override fun mouseClicked(e: MouseEvent) {

            }

            override fun mouseEntered(e: MouseEvent) {
                super.mouseEntered(e)
            }
		}
}
```

한 가지 차이는 객체 이름이 빠졌다는 점이다. 객체 식은 클래스를 정의하고 그 클래스에 속한 인스턴스를 생성하지만, 그 클래스나 인스턴스에 이름을 붙이지 않는다. 객체에 이름을 붙여야 한다면 변수에 무명 객체를 대입하면 된다.
