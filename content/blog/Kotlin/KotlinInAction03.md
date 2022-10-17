---
title: 'Kotlin In Action - 3장 함수 정의와 호출'
date: 2022-10-17
category: 'kotlin'
draft: false
---

# 1. 코틀린에서 컬렉션 만들기

숫자로 이뤄진 집합 만들기

```kotlin
// Set
val set = hashSetOf(1, 7, 53)
// List
val list = arrayListOf(1, 7, 53)
// map
val map = hashMapOf(1 to "one", 7 to "seven", 53 to "fifty-three")

// 실제 출력
[1, 53, 7]
[1, 7, 53]
{1=one, 53=fifty-three, 7=seven}
```

여기서 `to` 가 언어가 제공하는 특별한 키워드가 아닌 일반 함수이다.

코틀린 컬렉션은 자바 컬렉션과 똑같은 클래스다. 하지만 코틀린에서는 더 많은 기능을 사용할 수 있다.

예를 들어 리스트의 마지막 원소를 가져오거나 수로 이뤄진 컬렉션에서 최댓값을 찾을 수 있다.

```kotlin
val set = hashSetOf(1, 7, 53)
val strings = listOf("first", "second", "fourteenth")
println(strings.last())  // "fourteenth"
println(set.maxOrNull()) // 53
```

# 2. 함수를 호출하기 쉽게 만들기

자바 컬렉션에는 디폴트 `toString` 구현이 있다. 그러나 출력 형식은 원하는 형식이 아닐 수 있다. 원하는 형태로 바꾸려면 다른 라이브러리를 추가해야한다.

코틀린은 다음 같은 함수를 지원한다.

### `joinToString()` 함수의 초기 구현

```kotlin
fun <T> joinToString(
    collection: Collection<T>,
    separator: String,
    prefix: String,
    postfix: String
) : String {
    val result = StringBuilder(prefix)
    for ((index, element) in collection.withIndex()) {
        if (index > 0) result.append(separator) // 첫 원소 앞에는 구분자를 붙이면 안 된다.
        result.append(element)
    }
    result.append(postfix)
    return result.toString()
}

fun main() {
    val list = listOf(1, 2, 3)
    println(joinToString(list, "; ", "(", ")"))
}
// (1; 2; 3)
```

여기서 덜 번잡한 방법은 없을까?

## 2.1 이름 붙인 인자

```kotlin
println(joinToString(collection = list, separator = "; ", prefix = "(", postfix = ")"))
```

코틀린으로 작성한 함수를 호출할 때는 함수에 전달하는 인자 중 일부의 이름을 명시할 수 있다.

호출 시 인자 중 어느 하나라도 이름을 명시하고 나면 혼동을 막기 위해 그 뒤에 오는 모든 인자는 이름을 꼭 명시해야 한다.

## 2.2 디폴트 파라미터 값

코틀린에서는 함수 선언에서 파라미터의 디폴트 값을 지정할 수 있으므로 오버로드 중 상당수를 피할 수 있다.

다음은 디폴트 값을 사용하여 `joinToString` 함수를 개선한 것이다.

```kotlin
fun <T> joinToString(
    collection: Collection<T>,
    separator: String = ", ",
    prefix: String = "",
    postfix: String = ""
): String {
    val result = StringBuilder(prefix)
    for ((index, element) in collection.withIndex()) {
        if (index > 0) result.append(separator)
        result.append(element)
    }
    result.append(postfix)
    return result.toString()
}

fun main() {
	println(joinToString(list, ", ", "", ""))
    println(joinToString(list))
    println(joinToString(list, "; "))
}
1, 7, 53
1, 7, 53
1; 7; 53
```

이름 붙인 인자를 사용하는 경우에는 인자 목록의 중간에 있는 인자를 생략하고, 지정하고 싶은 인자를 이름을 붙여서 순서와 관계없이 지정할 수 있다.

```kotlin
joinToString(list, postfix = ";", prefix = "# ")
// # 1, 2, 3;
```

함수의 디폴트 파라미터 값은 함수를 호출하는 쪽이 아니라 함수 선언 쪽에서 지정된다.

어떤 클래스 안에 정의된 함수의 디폴트 값을 바꾸고 그 클래스가 포함된 파일을 재컴파일하면 그 함수를 호출하는 코드 중에 값을 지정하지 않은 모든 인자는 자동으로 바뀐 디폴트 값을 적용받는다.

### 디폴트 값과 자바

자바에는 디폴트 파라미터 값이라는 개념이 없어서 코틀린 함수를 자바에서 호출하는 경우에는 그 코틀린 함수가 디폴트 파라미터 값을 제공하더라도 모든 인자를 명시해야 한다.

자바에서 코틀린 함수를 자주 호출해야 한다면 자바 쪽에서 좀 더 편하게 코틀린 함수를 호출하고 싶을 때
`@JvmOverloads` 애노테이션을 함수에 추가할 수 있다.

`@JvmOverloads` 를 함수에 추가하면 코틀린 컴파일러가 자동으로 맨 마지막 파라미터로부터 파라미터를 하나씩 생략한 오버로딩한 자바 메소드를 추가해준다.

## 2.3 정적인 유틸리티 클래스 없애기: 최상위 함수와 프로퍼티

자바에서는 모든 코드를 클래스의 메소드로 작성해야 한다. 보통 그런 구조는 잘 작동하나 실전에서 어느 한 클래스에 포함시키기 어려운 코드가 많이 생긴다. 중요한 객체는 하나뿐이지만 그 연산을 객체의 인스턴스 API에 추가해서 API를 너무 크게 만들고 싶지 않은 경우도 있다.

그 결과 다양한 정적 메소드를 모아두는 역할만 담당하며, 특별한 상태나 인스턴스 메소드는 없는 클래스가 생겨난다. → JDK의 `Collections` 클래스가 전형적인 예

---

코틀린에서는 이런 무의미한 클래스가 필요 없다.

대신 함수를 직접 소스 파일의 최상위 수준, 모든 다른 클래스의 밖에 위치시면 된다. 그런 함수들은 여전히 그 파일의 맨 앞에 정의된 패키지의 멤버 함수이므로 다른 패키지에서 그 함수를 사용하고 싶을 때는 그 함수가 정의된 패키지를 임포트해야 한다.

하지만 임포트 시 유틸리티 클래스 이름이 추가로 들어갈 필요는 없다.

```kotlin
package strings

fun joinToString(...) : String {...}
```

JVM이 클래스 안에 들어있는 코드만을 실행할 수 있기 때문에 컴파일러는 이 파일을 컴파일할 때 새로운 클래스를 정의해준다. 코틀린만 사용하는 경우에는 그냥 그런 클래스가 생긴다는 사실만 기억하면 된다.

코틀린 컴파일러가 생성하는 클래스의 이름은 최상위 함수가 들어있던 코틀린 소스 파일의 이름과 대응한다. 코틀린 파일의 모든 최상위 함수는 이 클래스의 정적인 메소드가 된다.

### 파일에 대응하는 클래스의 이름 변경하기

코틀린 최상위 함수가 포함되는 클래스의 이름을 바꾸고 싶다면 파일에 `@JvmName` 애노테이션을 추가하면된다.

`@JvmName` 애노테이션은 파일의 맨 앞, 패키지 이름 선언 이전에 위치해야 한다.

```kotlin
@file:JvmName("StringFunctions") // 클래스 이름을 지정하는 애노테이션

package strings // `@file:JvmName 애노테이션 뒤에 패키지 문이 와야 한다.

fun joinToString(...) : String {...}

// Java
import strings.StringFuctions;
StringFunctions.joinToString(list, "");
```

### 최상위 프로퍼티

함수와 마찬가지로 프로퍼티도 파일의 최상위 수준에 놓을 수 있다. 어떤 데이터를 클래스 밖에 위치시켜야 하는 경우는 흔하지는 않지만, 그래도 가끔 유용할 때가 있다.

```kotlin
var opCount = 0

fun performOperation() {
    opCount++
}

fun reportOperationCount() {
    println("$opCount times")
}

fun main() {
    reportOperationCount()
}
// 1 times
```

기본적으로 최상위 프로퍼티도 다른 모든 프로퍼티처럼 접근자 메소드를 통해 자바 코드에 노출된다.(val의 경우 게터, var의 경우 게터 세터가 발생)

겉으론 상수처럼 보이는데, 실제로는 게터를 사용해야 한다면 자연스럽지 못하다. 더 자연스럽게 사용하려면 이 상수를 `public static final` 필드로 컴파일해야 한다.

`const` 변경자를 추가하면 프로퍼티를 `public static final` 필드로 컴파일하게 만들 수 있다.

```kotlin
const val UNIX_LINE_SEPERATOR = "\n"
```

# 3. 메소드를 다른 클래스에 추가: 확장 함수와 확장 프로퍼티

기존 코드와 코틀린 코드를 자연스럽게 통합하는 것은 코틀린의 핵심 목표 중 하나다.

완전히 코틀린으로만 이뤄진 프로젝트조차도 JDK나 안드로이드 프레임워크 또는 다른 서드파티 프레임워크 등의 자바 라이브러리를 기반으로 만들어진다.

또 코틀린을 기존 자바 프로젝트에 통합하는 경우에는 코틀린으로 직접 변환할 수 없거나 미처 변환하지 않은 기존 자바 코드를 처리할 수 있어야 한다.

확장 함수(`extension function`)을 사용하면 된다.

확장 함수는 어떤 클래스의 멤버 메소드인 것처럼 호출할 수 있지만 그 클래스의 밖에 선언된 함수다.

확장 함수를 보여주기 위해 어떤 문자열의 마지막 문자를 돌려주는 메소드를 추가해보자.

```kotlin
fun String.lastChar(): Char = this.get(this.length - 1)
```

확장 함수를 만들려면 추가하려는 함수 이름 앞에 그 함수가 확장할 클래스의 이름을 덧붙이기만 하면 된다.

클래스 이름을 **수신 객체 타입** 이라 부르며, 확장 함수가 호출되는 대상이 되는 값(객체)을 **수신 객체** 라고 한다

```kotlin
println("Kotlin".lastChar()) // n
```

이 예제에서는 String이 수신 객체 타입이고 `Kotlin` 이 수신 객체다.

어떤 면에서는 String 클래스에 새로운 메소드를 추가하는 것과 같다. String 클래스를 우리가 작성한것도 아니지만 원하는 메소드를 String 클래스에 추가할 수 있다.

자바 클래스로 컴파일한 클래스 파일이 있는 한 그 클래스에 원하는 대로 확장을 추가할 수 있다.

확장 함수 본문에서도 this를 생략 할 수 있다.

```kotlin
fun String.lastChar(): Char = get(length - 1)
```

확장 함수 내부에서는 일반적인 인스턴스 메소드의 내부에서와 마찬가지로 수신 객체의 메소드나 프로퍼티를 바로 사용할 수 있다.

하지만 확장 함수가 캡슐화를 깨지는 않는다는 사실을 기억해야한다. 클래스 안에서 정의한 메소드와 달리 확장 함수 안에서는 클래스 내부에서만 사용할 수 있는 비공개(private) 멤버나 보호된(protected) 멤버를 사용할 수 없다.

## 3.1 임포트와 확장 함수

확장 함수를 정의했다고 해도 자동으로 프로젝트 안의 모든 소스코드에서 그 함수를 사용할 수 있지는 않다. 확장 함수를 사용하기 위해서는 그 함수를 다른 클래스나 함수와 마찬가지로 임포트해야만 한다.

코틀린에서는 클래스를 임포트할 때와 동일한 구문을 사용해 개별함수를 임포트 할 수 있다.

```kotlin
import strings.lastChar

val c = "kotlin".lastChar()

// *도 가능
import strings.*

// as 키워드를 사용하면 다른 이름으로 부를 수 있음
import strings.lastChar as last

val c = "kotlin".last
```

코틀린 문법상 확장 함수는 반드시 짧은 이름을 써야 한다. → 임포트할 때 이름을 바꾸는 것이 확장 함수 이름 충돌을 해결할 수 있는 유일한 방법이다.

## 3.2 자바에서 확장 함수 호출

확장 함수는 수신 객체를 첫 번째 인자로 받는 정적 메소드다.

→ 확장 함수를 호출해도 다른 어댑터객체나 실행 시점 부가 비용이 들지 않는다.

이런 설계로 인해 자바에서 확장 함수를 사용하기 도 편하다.

확장 함수를 `StringUtil.kt` 파일에 정의했다면 다음과 같이 호출할 수 있다.

```java
char c = StringUtilkt.lastChar("Java")
```

## 3.3 확장 함수로 유틸리티 함수 정의

```kotlin
fun <T> Collection<T>.joinToString( // Collection<T>에 대한 확장 함수를 선언
    separator: String = ", ",
    prefix: String = "",
    postfix: String = ""
): String {
    val result = StringBuilder(prefix)
    for ((index, element) in this.withIndex()) { // "this" 는 수신 객체를 기리킨다. 여기는 T 타입의 원소 컬렉션
        if (index > 0) result.append(separator)
        result.append(element)
    }
    result.append(postfix)
    return result.toString()
}

fun main() {
    val list = listOf(1, 2, 3)
    println(list.joinToString(separator = "; ", prefix = "(", postfix = ")"))
}
// (1; 2; 3)
```

확장 함수는 단지 정적 메소드 호출에 대한 문법적인 편의일 뿐이다. 그래서 클래스가 아닌 더 구체적인 타입을 수신 객체 타입으로 지정할 수 있다.

→ 문자열의 컬렉션에 대해서만 호출할 수 있는 `join` 함수를 정의하고 싶다면 다음과 같이 한다.

```kotlin
fun Collection<String>.join(
    separator: String = ", ",
    prefix: String = "",
    postfix: String = ""
) = joinToString(separator, prefix, postfix)

println(listOf("One", "TWO", "three").join(" "))
// One TWO three
```

이 함수를 객체의 리스트에 대해 호출할 수는 없다. 확장 함수가 정적 메소드와 같은 특징을 가지므로, 확장 함수를 하위 클래스에서 오버라이드 할 수 는 없다.

## 3.4 확장 함수는 오버라이드할 수 없다

다음은 View와 그 하위 클래스인 Button이 있는데, Button이 이 상위 클래스의 click 함수를 오버라이드하는 경우를 보자.

```kotlin
open class View {
    open fun click() = println("View clicked")
}

class Button : View() {
    override fun click() {
        println("Button clicked")
    }
}
```

Button 이 View의 하위 타입이기 때문에 View 타입 변수를 선언해도 Button 타입 변수를 그 변수에 대입할 수 있다. View 타입 변수에 대해 click과 같은 일반 메소드를 호출했는데, click을 Button 클래스가 오버라이드 했다면 실제로는 Button이 오버라이드한 click이 호출된다.

```kotlin
fun main() {
    val view: View = Button() // view 에 저장된 값의 실제 타입에 따라 호출할 메소드가 결정된다.
    view.click()
}
// Button clicked
```

확장 함수는 클래스의 일부가 아니다. 확장 함수는 클래스 밖에 선언된다. 이름과 파라미터가 완전히 같은 확장 함수를 기반 클래스와 하위 클래스에 대해 정의해도 실제로는 확장 함수를 호출할 때 수신 객체로 지정된 변수의 정적 타입에 의해 어떤 확장 함수가 호출될지 결정된다. → 동적으로 확장 함수가 정의되지 않음

```kotlin
fun View.showOff() = println("View View")
fun Button.showOff() = println("Button Button")

fun main() {
    val view: View = Button()
    view.showOff()
}
// View View <- 확장 함수는 정적으로 정의된다.
```

## 3.5 확장 프로퍼티

확장 프로퍼티를 사용하면 기존 클래스 객체에 대한 프로퍼티 형식의 구문으로 사용할 수 있는 API를 추가할 수 있다.

프로퍼티라는 이름으로 불리기는 하지만 상태를 저장할 적절한 방법이 없기 때문에 실제로 확장 프로퍼티는 아무 상태도 가질 수 없다. 하지만 프로퍼티 문법으로 더 짧게 코드를 작성할 수 있어서 편한 경우가 있다.

`lastChar` 함수를 프로퍼티로 바꿔보자.

```kotlin
val String.lastChar: Char get() = get(length - 1)
```

확장 프로퍼티도 일반적인 프로퍼티와 같은데, 단지 수신 객체 클래스가 추가됐을 뿐이다.

뒷받침하는 필드가 없어서 기본 게터 구현을 제공할 수 없으므로 최소한 게터는 꼭 정의해야한다.

초기화 코드에서 계산한 값을 담을 장소가 전혀 없어서 초기화 코드도 쓸 수 없다.

`StringBuilder` 에 같은 프로퍼티를 정의한다면 `StringBuilder` 의 맨 마지막 문자는 변경 가능하므로 프로퍼티를 `var` 로 만들 수있다.

```kotlin
var StringBuilder.lastChar: Char
    get() = get(length - 1) // 프로퍼티 게터
    set(value: Char) {
        this.setCharAt(length - 1, value) // 프로퍼티 세터
    }
		println("kotlin".lastChar)
    val sb = StringBuilder("kotlin?")
    sb.lastChar = '!'
    println(sb)
// n
// kotlin!
```

# 4. 컬렉션 처리: 가변 길이 인자, 중위 함수 호출, 라이브러리 지원

코틀린 언어 특성

- varag 키워드를 사용하면 호출시 인자 개수가 달라질 수 있는 함수를 정의할 수 있다.
- 중위 함수 호출 구문을 사용하면 인자가 하나뿐인 메소드를 간편하게 호출할 수 있다.
- 구조 분해 선언을 사용하면 복합적인 분해해서 여러 변수에 나눠 담을 수 있다.

## 4.1 자바 컬렉션 API 확장

```kotlin
fun main() {
    val strings: List<String> = listOf("first", "second", "fourteenth")
    println(strings.last()) // fourteenth
    val numbers: Collection<Int> = setOf(1, 14, 2)
    println(numbers.maxOrNull()) // 14
}
```

코틀린 확장 라이브러리는 수많은 확장 함수를 포함하므로 아주 많다. 코틀린 표준 라이브러리를 모두 알 필요가 없다. IDE가 표시해주는 목록에서 그냥 원하는 함수를 선택하기만 하면 된다.

## 4.2 가변 인자 함수: 인자의 개수가 달라질 수 있는 함수 정의

가변 길이 인자는 메소드를 호출할 때 원하는 개수만큼 값을 인자로 넘기면 자바 컴파일러가 배열에 그 값들을 넣어주는 기능이다.

코틀린은 타입 뒤에 `...` 대신 파라미터 앞에 `vararg` 변경자를 붙인다.

코틀린은 배열을 명시적으로 풀어서 배열의 각 원소가 인자로 전달되게 해야 한다. 스프레드 연산자가 해결해준다.

실제로 전달하려는 배열 앞에 `*` 를 붙이기만 하면 된다.

```kotlin
fun main(args: Array<String>) {
    val list = listOf("args: ", *args) // 스프레드 연산자가 배열의 내용을 펼쳐준다.
    println(list)
}
```

## 4.3 값의 쌍 다루기: 중위 호출과 구조 분해 선언

맵을 만드려면 `mapOf` 함수를 사용한다.

```kotlin
val map = mapOf(1 to "one", 2 to "two")
```

중위 호출 이라는 특별한 방식으로 `to` 라는 일반 메소드를 호출한다.

중위 호출 시에는 수신 객체와 유일한 메소드 인자 사이에 메소드 이름을 넣는다.(이때 객체, 메소드 이름, 유일한 인자 사이에는 공백이 들어가야 한다.)

```kotlin
1.to("one")
1 to "one"
```

인자가 하나뿐인 일반 메소드나 인자가 하나뿐인 확장 함수에 중위 호출을 사용할 수 있다.

함수(메소드)를 중위 호출에 사용하게 허용하고 싶으면 `infix` 변경자를 함수 선언 앞에 추가해야 한다.

다음은 `to` 함수의 정의를 간략하게 줄인 코드다.

```kotlin
infix fun Any.to(other: Any) = Pair(this, other)
```

이 `to` 함수는 `Pair` 인스턴스를 반환한다. `Pair` 는 코틀린 표준 라이브러리 클래스로, 두 원소로 이뤄진 순서쌍을 표현한다.

`Pair` 의 내용으로 두 변수를 즉시 초기화할 수 있다.

```kotlin
val (number, name) = 1 to "one" // 구조 분해 선언
```

루프에서도 구조 분해 선언을 활용할 수 있다. `joinToString` 에서 본 `withIndex` 를 구조 분해 선언과 조합하면 컬렉션 원소의 인덱스와 값을 따로 변수에 담을 수 있다.

```kotlin
for ((index, element) in collection.withIndex()) {
        println("$index : $element")
    }
```

`to` 함수는 확장 함수다. `to` 를 사용하면 타입과 관계없이 임의의 순서쌍을 만들 수 있다.

다음은 `mapOf` 함수의 선언이다.

```kotlin
fun <K, V> mapOf(vararg values: Pair<K, V>) : Map<K, V>
```

# 5. 문자열과 정규식 다루기

코틀린 문자열은 자바 문자열과 같다.

코틀린은 다양한 확장 함수를 제공함으로써 표준 자바 문자열을 더 쉽게 다룰 수 있다.

## 5.1 문자열 나누기

자바의 `split` 메소드는 빈 배열을 반환한다. 구분 문자열이 정규식이기 때문이다. 따라서 `.` 모든 문자를 나타내는 정규식으로 해석한다.

코틀린에서 정규식을 파라미터로 받는 함수는 String이 아닌 Regex 타입의 값을 받는다. 따라서 코틀린에서는 split 함수에 전달하는 값의 타입에 따라 정규식이나 일반 텍스트 중 어느 것으로 문자열을 분리하는지 쉽게 알 수 있다. 다음 코드는 마침표나 대시(-) 문자열을 분리하는 예다.

```kotlin
println("12.345-6.A".split("\\.|-".toRegex()))
//[12, 345, 6, A]
```

코틀린 정규식 문법은 자바와 똑같다. 코틀린에서는 `toRegex` 확장 함수를 사용해 문자열을 정규식으로 변환할 수 있다. 간단한 경우는 다음처럼 바꿀 수 있다.

```kotlin
println("12.345-6.A".split(".", "-"))
```

## 5.2 정규식과 3중 따옴표로 묶은 문자열

코틀린 표준 라이브러리에는 어떤 문자열에서 구분 문자열이 맨 나중에 나타난 곳의 부분 문자열을 반환하는 함수가 있다. 이런 함수를사용해 경로 파싱을 구현한 버전은 다음과 같다.

```kotlin
fun parsePath(path: String) {
    val directory = path.substringBeforeLast("/")
    val fullName = path.substringAfterLast("/")
    val fileName = fullName.substringBeforeLast(".")
    val extension = fullName.substringAfterLast(".")

    println("Dir: $directory, name: $fileName, ext: $extension")
}

//Dir: /Users/yole/kotlin-book, name: chapter, ext: adoc
```

코틀린에서는 정규식을 사용하지 않아도 문자열을 쉽게 파싱할 수 있다.

정규식을 사용해야할 때 라이브러리를 사용하면 더 편하다.

다음은 경로 파싱에 정규식 사용을 했다.

```kotlin
fun parsePathRegex(path: String) {
    val regex = """(.+)/(.+)\.(.+)""".toRegex()
    val matchResult = regex.matchEntire(path)
    if (matchResult != null) {
        val (directory, fileName, extension) = matchResult.destructured
        println("Dir: $directory, name: $fileName, ext: $extension")
    }
}
```

3중 따옴표 문자열을 사용해 정규식을 사용했다.

## 5.3 여러 줄 3중 따옴표 문자열

3중 따옴표 문자열을 문자열 이스케이프를 피하기 위해서만 사용하지는 않는다. 3중 따옴표 문자열에는 줄 바꿈을 표현하는 아무 문자열이나 그대로 들아간다.

따라서 3중 따옴표를 쓰면 줄 바꿈이 들어있는 프로그램 텍스트를 쉽게 문자열로 만들 수 있다.

```kotlin
val kotlinLogo =
      """|  //
        .| //
        .|/ \""".trimMargin(".")
    println(kotlinLogo)
```

프로그래밍 시 여러 줄 문자열이 요긴한 분야로 테스트를 꼽을 수 있다. 여러 줄 문자열은 테스트의 예상 출력을 작성할 때 사용한다.

# 6. 코드 다듬기: 로컬 함수와 확장

코틀린에는 깔끔한 해법이 있다. 코틀린에서는 함수에서 추출한 함수를 원 함수 내부에 중첩시킬 수 있다. 그렇게 하면 문법적인 부가 비용을 들이지 않고도 깔끔하게 코드를 조작할 수 있다.

다음은 리스트에서 사용자를 저장하는 데이터베이스에 저장하는 함수다.

```kotlin
class User(val id: Int, val name: String, val address: String)

fun saveUser(user: User) {
    if (user.name.isEmpty()) {
        throw IllegalArgumentException(
            "Can't save user ${user.id}: empty name"
        )
    }
    if (user.address.isEmpty()) {
        throw IllegalArgumentException(
            "Can't save user ${user.address}: empty address"
        )
    }
    // 데이터베이스 저장 코드
}

fun main() {
    saveUser(User(1, "","")) // 에러 발생
}
```

다음은 중복제거다.

```kotlin
fun saveUser(user: User) {
    fun validate(
        user: User,
        value: String,
        fieldName: String
    ) {
        if (value.isEmpty()) {
            throw IllegalArgumentException(
                "Can't save user ${user.id}: empty $fieldName"
            )
        }
    }
    validate(user, user.name, "Name")
    validate(user, user.address, "Address")
    // 데이터베이스 저장 코드
}
```

로컬 함수는 자신이 속한 바깥 함수의 모든 파라미터와 변수를 사용할 수 있다.

이런 성질을 이용해 불필요한 User 파라미터를 없애보자.

```kotlin
fun saveUser(user: User) {
    fun validate(
        value: String,
        fieldName: String
    ) {
        if (value.isEmpty()) {
            throw IllegalArgumentException(
				// 바깥 함수의 파라미터에 직접 접근할 수 있다.
                "Can't save user ${user.id}: empty $fieldName"
            )
        }
    }
    validate(user.name, "Name")
    validate(user.address, "Address")
    // 데이터베이스 저장
}
```

좀 더 개선하고 싶다면 검증 로직을 User 클래스를 확장한 함수로 만들 수도 있다.

```kotlin
class User(val id: Int, val name: String, val address: String)

fun User.validateBeforeSave() {
    fun validate(
        value: String,
        fieldName: String
    ) {
        if (value.isEmpty()) {
            throw IllegalArgumentException(
                "Can't save user ${id}: empty $fieldName"
            )
        }
    }
    validate(name, "Name")
    validate(address, "Address")
    // 데이터베이스 저장
}

fun saveUser(user: User) {
    user.validateBeforeSave()
}

fun main() {
    saveUser(User(1, "",""))
}
```

`User` 는 라이브러리에 있는 클래스가 아니라 나만의 클래스지만, 이 경우 검증 로직은 `User` 를 사용하는 다른 곳에서는 쓰이지 않는 기능이기 때문에 `User` 에 포함시키고 싶지는 않다.

`User` 를 간결하게 유지하면 생각해야 할 내용이 줄어들어서 더 쉽게 코드를 파악할 수 있다.

반면 한 객체만을 다루면서 객체의 비공개 데이터를 다룰 필요는 없는 함수는 확장 함수로 만들면 `객체.멤버` 처럼 수신 객체를 지정하지 않고도 공개된 멤버 프로퍼티나 메소드에 접근할 수 있다.

확장 함수를 로컬 함수로 정의할 수도 있다. 즉 `User.validateBeforeSave` 를 `saveuser` 내부에 로컬 함수로 넣을 수 있다. 하지만 중첩된 함수의 깊이가 길어지면 코드를 읽기가 상당히 어려워진다.

일반적으로 한 단계만 함수를 중첩시키면 된다.
