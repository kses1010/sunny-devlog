---
title: 'Kotlin In Action - 2장 코틀린 기초'
date: 2022-10-04
category: 'kotlin'
draft: false
---

# 1. 기본 요소: 함수와 변수

## 1.1 Hello, World!

```kotlin
fun main() {
    println("Hello, world!")
}
```

- 함수를 선언할 때 `fun` 키워드를 사용한다.
- 파라미터 이름 뒤에 그 파라미터의 타입을 쓴다. `fun(str: String)`
- 함수를 최상위 수준에 정의할 수 있다.
  - (자바와 달리) 꼭 클래스 안에 함수를 넣어야 할 필요가 없다.
- 배열도 일반적인 클래스와 마찬가지다.
  - 코틀린에는 자바와 달리 배열 처리를 위한 문법이 따로 존재하지 않는다.
- println이라고 쓴다.
- 줄 끝에 세미콜론(`;`)을 붙이지 않아도 된다.

## 1.2 함수

반환 값을 어디에 지정해야 할까?

```kotlin
fun main() {
    fun max(a: Int, b: Int): Int {
        return if (a > b) a else b
    }
    println(max(1, 2))
}
```

### 문(statement)과 식(expression)의 구분

코틀린에서 if는 식이지 문이 아니다.

식은 **값을 만들어 내며 다른 식의 하위 요소로 계산에 참여**할 수 있는 반면

문은 자신을 둘러싸고 있는 가장 안쪽 블록의 최상위 요소로 존재하며 **아무런 값을 만들어내지 않는다**는 차이가 있다.

자바에서는 모든 제어 구조가 문인 반면

코틀린에서는 루프를 제외한 대부분의 제어 구조가 식이다.

→ 대입문은 자바에서는 식이었으나 코틀린에서는 문이 됐다. 그로인해 대입식과 비교식을 잘못쓰는 버그가 종종 있다.

### 식이 본문인 함수

앞의 함수 본문은 if 식 하나로만 이루어져 있다. 이 경우 중괄호를 없애고 return을 제거하면서 등호(`=`)를 식 앞에 붙이면

더 간결하게 함수를 표현할 수 있다.

```kotlin
fun max(a: Int, b: Int): Int = if (a > b) a else b
```

각각은 `Convert to expression body` 와 `Convert to block body` 로 간단하게 바꿀 수 있다.

코틀린에서는 식이 본문인 함수가 자주 쓰인다.

반환 타입을 생략하면 더 간략하게 만들 수 있다.

```kotlin
fun max(a: Int, b: Int) = if (a > b) a else b
```

식이 본문인 함수의 경우 사용자가 반환 타입을 적지 않아도 컴파일러가 함수 본문 식을 분석해서 식의 결과 타입으로 정해준다.

→ **타입 추론(type inference)** 이다.

실전 프로그램에서는 아주 긴 함수에 return 문이 여럿 들어있는 경우가 자주 있다.

그런 경우 반환 타입을 꼭 명시하고 return 을 반드시 사용한다면
함수가 어떤 타입의 값을 반환하고 어디서 그런값을 반환하는지 더 쉽게 알아볼 수 있다.

## 1.3 변수

코틀린에서는 타입 지정을 생략하는 경우가 흔하다.

```kotlin
val question = "who are you"
val answer = 42

// 타입 선언시
val answer: Int = 42

// 초기화 식을 사용하지 않는 경우에는 타입을 명시할 것
val answer: Int
answer = 42
```

### 변경 가능한 변수와 변경 불가능한 변수

- val(value): **변경 불가능한(Immutable)** 참조를 저장하는 변수.
  - val로 선언된 변수는 일단 초기화하고 나면 재대입이 불가능하다. → 자바로 치면 final 변수
- var(variable) - **변경 가능한(mutable)** 참조. → 자바의 일반 변수에 해당.

기본적으로는 모든 변수를 val 키워드를 사용해 불변 변수로 선언하고, 나중에 필요할 때만 var로 변경할 것.

val과 var을 조합해 사용하면 코드가 함수형 코드에 가까워진다.

val 변수는 블록을 실행할 때 정확히 한 번만 초기화돼야 한다.

→ 어떤 블록이 실횅될 때 오직 한 초기화 문장만 실행됨을 컴파일러가 확인할 수 있다면 조건에 따라 val 값을 여러 값으로 초기화할 수도 있다.

```kotlin
val message: String
if (canPerformOperation()) {
    message = "Success"
}
else {
    message = "Failed"
}
```

val 참조 자체는 불변일지라도 그 참조가 가리키는 객체의 내부 값은 변경될 수 있다.

```kotlin
val languages = arrayListOf("Java") // 불변 참조를 선언
languages.add("Kotlin")            // 참조가 가리키는 객체 내부를 변경한다.
```

var 키워드를 사용하면 변수의 값은 변경할 수 있지만 변수의 타입은 고정돼 바뀌지 않는다.

```kotlin
var answer = 42
answer = "no answer" // 컴파일 에러
```

이 경우 강제 형 변환(`coerce`) 필요하다.

## 1.4 더 쉽게 문자열 형식 지정: 문자열 템플릿

```kotlin
fun main(args: Array<String>) {
    val name = if (args.size > 0) args[0] else "Kotlin"
    println("Hello, $name!")
}
```

인자를 넣으면 "Hello, 인자"를 출력하고 아무 인자도 없으면 "Hello, Kotlin"을 출력한다.

문자열 리터럴의 필요한 곳에 변수를 넣되 변수 앞에 `$` 를 추가해야 한다.

`$` 문자를 문자열에 넣고 싶으면 `println("\$x")` 와 같이 `\` 를 사용해 `$` 를 이스케이프(escape)시켜야 한다.

복잡한 식도 중괄호(`{}`)로 둘러싸서 문자열 템플릿 안에 넣을 수 있다.

```kotlin
fun main(args: Array<String>) {
    val arrayOf = arrayOf("Java", "C++")
    println("Hello, ${arrayOf[0]}!") // 배열의 첫번째 원소를 넣기 위해 ${}구문을 사용한다.
}
```

### 한글을 문자열 템플릿에서 사용할 경우 주의할 점

코틀린 컴파일러는 영문자와 한글을 한꺼번에 식별자로 인식해서 `unresolved reference` 오류를 발생시킨다.

```kotlin
fun main(args: Array<String>) {
    val name = "Sunny"
    println("$name님 반가와요!") // 컴파일 에러
		// 중괄호 사용
    println("${name}님 반가와요!")
}
```

이 문제를 해결하는 방법은 `${name}` 처럼 `{}` 로 감싸는 것이다.

문자열 템플릿 안에서 변수이름만 사용하더라도 `${name}` 처럼 중괄호로 변수명을 감싸는 습관을 들이자.

# 2. 클래스와 프로퍼티

간단한 자바 클래스 class

Person에는 name이라는 프로퍼티(property)만 들어있다.

```java
public class Person {
		private final String name;

		public Person(name) {
			this.name = name;
		}

		public String getName() {
			return name;
		}
}
```

코틀린으로 변환한 Person 클래스

```kotlin
class Person(val name: String)
```

이런 유형의 클래스(코드가 없이 데이터만 저장하는 클래스)를 **값 객체(Value Object)** 라 부르며, 다양한 언어가 값 객체를 간결하게 기술할 수 있는 구문을 제공한다.

코틀린의 기본 가시성은 public 이므로 변경자를 생략한다.

## 2.1 프로퍼티

자바에서는 데이터를 필드(field)에 저장하며, 멤버 필드의 가시성은 보통 비공개(private)다.

클래스는 자신을 사용하는 클라이언트가 그 데이터에 접근하는 통로로 쓸 수 있는 **접근자 메소드(accessor method)** 를 제공한다.

보통 게터(getter), 세터(setter)를 추가 제공한다.

자바에서는 필드와 접근자를 한데 묶어 프로퍼티(property)라고 부르며, 프로퍼티라는 개념을 활용하는 프레임워크가 많다.

코틀린은 프로퍼티를 언어 기본 기능으로 제공하며, 코틀린 프로퍼티는 자바의 필드와 접근자 메소드를 완전히 대신한다.

```kotlin
class Person(
    val name: String,           // 읽기 전용. getter만 만들어 낸다.
    var isMarried: Boolean      // 쓸 수 있는 프로퍼티. 비공개필드, 게터, 세터를 만들어 낸다.
)
```

코틀린은 값을 저장하기 위한 비공개 필드와 그 필드에 값을 저장하기 위한 세터, 필드의 값을 읽기 위한 게터로 이뤄진 간단한 디폴트 접근자 구현을 제공한다.

다음은 Person을 자바 코드에서 사용하는 방법을 보여준다.

```java
public class Main {
    public static void main(String[] args) {
        Person person = new Person("Bob", true);
        System.out.println(person.getName());
        System.out.println(person.isMarried());
    }
}
```

코틀린

```kotlin
fun main() {
    val person = Person("Bob", true)
    println(person.name)
    println(person.isMarried)
}
```

게터를 호출하는 대신 프로퍼티를 직접 사용했다.

setter 사용법

```kotlin
//Java
person.setMarried(false)
// kotlin
person.isMarried = false
```

대부분의 프로퍼티에는 그 프로퍼티의 값을 저장하기 위한 필드가 있다. 이를 프로퍼티를 뒷바침하는 필드(backing field)라고 부른다.

## 2.2 커스텀 접근자

직사각형 클래스인 Rectangle을 정의하면서 자신이 정사각형인지 알려주는 기능을 만들어보자

```kotlin
class Rectangle(val height: Int, val width: Int) {
    val isSquare: Boolean
        get() {                    // 프로퍼티 게터 선언
            return height == width
        }
}
```

`isSquare` 프로퍼티에는 자체 값을 저장하는 필드가 필요 없다. 이 프로퍼티에는 자체 구현을 제공하는 게터만 존재한다.

클라이언트가 프로퍼티에 접근할 때마다 게터가 프로퍼티 값을 매번 다시 계산한다.

```kotlin
fun main() {
    val rectangle = Rectangle(41, 43)
    println(rectangle.isSquare)
}
```

구현이나 성능상 차이는 없다. 차이가 나는건 가독성뿐이다.

## 2.3 코틀린 소스코드 구조: 디렉터리와 패키지

자바의 경우 모든 클래스를 패키지 단위로 관리한다.

코틀린에도 자바와 비슷한 개념의 패키지가 있다. 모든 코틀린 파일의 맨 앞에 package 문을 넣을 수 있다.

그러면 그 파일 안에 있는 모든 선언(클래스, 함수, 프로퍼티 등)이 해당 패키지에 들어간다.

같은 패키지에 속해 있다면 다른 파일에서 정의한 선언일지라도 직접 사용할 수 있다.

반면 다른 패키지에 정의한 선언을 사용하려면 임포트를 통해 선언을 불러와야 한다. → 자바와 마찬가지

```kotlin
package shape

import java.util.Random

class Rectangle(val height: Int, val width: Int) {
    val isSquare: Boolean
        get() {
            return height == width
        }
}

fun createRandomRectangle(): Rectangle {
    val random = Random()
    return Rectangle(random.nextInt(), random.nextInt())
}
```

코틀린에서는 클래스 임포트와 함수 임포트에 차이가 없으며, 모든 선언을 import 키워드로 가져올 수 있다.

자바에서는 디렉터리 구조가 패키지 구조를 그대로 따라가야한다.

코틀린에서는 패키지 구조와 디렉터리 구조가 맞아 떨어질 필요는 없다.

그러나! 대부분의 경우 자바와 같이 패키지별로 디렉터리를 구성하는 편이 낫다.

자바의 방식을 따르지 않으면 자바 클래스를 코틀린 클래스로 마이그레이션할 때 문제가 발생할 수 있다.

# 3. 선택 표현과 처리: enum과 when

when은 자바의 switch를 대치하되 훨씬 더 강력하다.

## 3.1 enum 클래스 정의

다음은 색을 표현하는 enum이다.

```kotlin
enum class Color {
    RED, ORANGE, YELLOW, GREEN, BLUE, INDIGO, VIOLET
}
```

enum은 자바보다 코틀린이 더 많은 키워드를 써야하는 흔치 않는 예다.

enum 클래스 안에도 프로퍼티나 메소드를 정의할 수 있다.

```kotlin
enum class Color(
    val r: Int, val g: Int, val b: Int
) {
    RED(255, 0, 0), ORANGE(255, 165, 0),
    YELLOW(255, 255, 0), GREEN(0, 255, 0), BLUE(0, 0, 255),
    INDIGO(75, 0, 130), VIOLET(238, 130, 238);

    fun rgb() = (r * 256 + g) * 256 + b
}
```

여기서 유일하게 세미콜론(`;`)이 필수인 부분을 볼 수 있다.

enum 클래스 안에 메소드를 정의하는 경우 반드시 enum 상수 목록과 메소드 정의 사이에 세미콜론을 넣어야 한다.

## 3.2 when으로 enum 클래스 다루기

switch 에 해당하는 코틀린 구성요소는 `when` 이다.

다음은 when을 사용해 올바른 enum 값을 찾는 코드다.

```kotlin
fun getMnemonic(color: Color) =      // 함수의 반환 값으로 when 식을 직접 사용
    when (color) {
        Color.RED -> "red"
        Color.ORANGE -> "주"
        Color.YELLOW -> "노"
        Color.GREEN -> "초"
        Color.BLUE -> "파"
        Color.INDIGO -> "남"
        Color.VIOLET -> "보"
    }
```

자바와 달리 각 분기의 끝에 break를 넣지 않아도 된다.

한 분기 안에서 여러 값을 매치 패턴으로 사용할 경우 값 사이에 콤마(`,`)로 분리한다.

```kotlin
fun getMnemonic(color: Color) =
    when (color) {
        Color.RED, Color.ORANGE, Color.YELLOW -> "warm"
```

## 3.3 when과 임의의 객체를 함께 사용

코틀린 when의 분기 조건은 임의의 객체를 허용한다.

```kotlin
fun mix(c1: Color, c2: Color) =
    when (setOf(c1,c2)) {
        setOf(RED, YELLOW) -> ORANGE
        setOf(YELLOW, BLUE) -> GREEN
        setOf(BLUE, VIOLET) -> INDIGO
        else -> throw Exception("더러운 색깔") // 매치되는 분기 조건이 없으면 이 문장 실행
    }
```

when의 분기 조건 부분에 식을 넣을 수 있기 때문에 많은 경우 코드를 더 간결하고 아름답게 작성할 수 있다.

이 예제에서는 조건에서 동등성을 검사했다.

## 3.4 인자 없는 when 사용

위 코드는 약간 비효율적이다. when의 분기조건에 있는 다른 두 색과 같은지 비교하기 위해 여러 set 인스턴스를 생성한다.

인자가 없는 when 식을 사용하면 불필요한 객체 생성을 막을 수 있다.

```kotlin
fun mix(c1: Color, c2: Color) =
    when {
        (c1 == RED && c2 == YELLOW) || (c1 == YELLOW && c2 == RED) -> ORANGE
        (c1 == YELLOW && c2 == BLUE) || (c1 == BLUE && c2 == YELLOW) -> GREEN

        else -> throw Exception("더러운 색깔")
    }
```

when에 아무 인자도 없으려면 각 분기의 조건이 불리언 결과를 계산하는 식이어야 한다.

추가 객체를 만들지 않는다는 장점이 있지만 가독성이 더 떨어진다.

## 3.5 스마트 캐스트: 타입 검사와 타입 캐스트를 조합

이번 예제는 (1 + 2) + 4 같은 간단한 산술식을 계산하는 함수를 만들어보자

다음은 식을 표현하는 클래스 계층을 표현한 코드다

```kotlin
interface Expr
class Num(val value: Int) : Expr // value라는 프로퍼티만 존재하는 단순한 클래스로 Expr 인터페이스를 구현한다.

// Expr 타입의 객체라면 어떤것이나 sum 연산의 인자가 될 수 있다. -> Num이나 다른 sum이 인자로 올 수 있다.
class Sum(val left: Expr, val right: Expr) : Expr
```

자바 스타일로 짰다면 if문을 사용했을 것이다. 다음은 자바스타일로 짠 코틀린 코드다.

```kotlin
fun main() {
    println(evalJava(Sum(Sum(Num(1), Num(2)), Num(4))))
}

fun evalJava(e: Expr): Int {
    if (e is Num) {
        val n = e as Num // 불필요한 중복
        return n.value
    }
    if (e is Sum) {
        return evalJava(e.right) + evalJava(e.left) // 변수 e에 대해 스마트 캐스트 사용
    }

    throw IllegalArgumentException("알수 없는 표현식")
}
```

코틀린에서는 `is`를 사용해 변수 타입을 검사한다.

코틀린에서는 자바와는 다르게 일단 `is`로 검사하고 나면 굳이 변수를 원하는 타입으로 캐스팅하지 않아도 컴파일러가 캐스팅을 해준다.

이를 스마트 캐스트라고 한다.

앞 예제처럼 클래스의 프로퍼티에 대해 스마트 캐스트를 사용한다면 그 프로퍼티는 반드시 val이어야 하며 커스텀 접근자를 사용해선 안된다.

원하는 타입으로 명시적으로 타입 캐스팅하려면 `as` 키워드를 사용한다.

## 3.6 리팩토링: if를 when으로 변경

코틀린에서는 if가 값을 만들어내기 때문에 자바와 달리 3항 연산자가 따로 없다.

```kotlin
// 값을 만들어내는 if 식
fun evalJava(e: Expr): Int {
    return if (e is Num) {
        e.value
    } else if (e is Sum) {
        evalJava(e.right) + evalJava(e.left)
    } else {
        throw IllegalArgumentException("알수 없는 표현식")
    }
}
// if 중첩 대신 when 사용하기
fun evalJava(e: Expr): Int {
    return when (e) {
        is Num -> {
            e.value
        }
        is Sum -> {
            evalJava(e.right) + evalJava(e.left)
        }
        else -> {
            throw IllegalArgumentException("알수 없는 표현식")
        }
    }
}
```

## 3.7 if문과 when의 분기에서 블록 사용

```kotlin
fun evalWithLogging(e: Expr): Int {
    return when (e) {
        is Num -> {
            println("num: ${e.value}")
            e.value
        }
        is Sum -> {
            val left = evalWithLogging(e.left)
            val right = evalWithLogging(e.right)
            println("sum: $left + $right")
            left + right
        }
        else -> throw IllegalArgumentException("알수 없는 표현식")
    }
}
```

# 4. 대상을 이터레이션: while 과 for 루프

자바와 가장 비슷한 것이 이터레이션이다.

for는 자바의 for-each 루프에 해당하는 형태만 존재한다.

## 4.1 while 루프

자바와 동일

## 4.2 수에 대한 이터레이션: 범위와 수열

코틀린에는 자바의 for 루프에 해당하는 요소가 없다. 초기값, 증가값, 최종값을 사용한 루프를 대신하기 위해 코틀린에서는 범위(range)를 사용한다.

`..` 연산자를 사용한다.

```kotlin
val oneToTen = 1..10
```

다음은 피즈버즈 게임 구현이다.

```kotlin
fun main() {
    for (i in 1..100) {
        print(fizzBuzz(i))
    }
}

fun fizzBuzz(i: Int) = when {
    i % 15 == 0 -> "fizzBuzz"
    i % 3 == 0 -> "Fizz"
    i % 5 == 0 -> "Buzz"
    else -> "$i"
}
```

다음은 100부터 거꾸로 세되 짝수만으로 게임을 진행해보자

```kotlin
for (i in 100 downTo 1 step 2) {
        print(fizzBuzz(i))
    }
```

`..` 는 항상 범위의 끝 값을 포함한다. 하지만 끝값을 포함하지 않는 닫힌 범위에 대해 구하고 싶으면

`until` 함수를 사용하면 된다.

## 4.3 맵에 대한 이터레이션

다음은 2진 표현을 출력하는 프로그램이다.

```kotlin
// 맵을 초기화하고 이터레이션 하기
fun main() {
    val binaryReps = TreeMap<Char, String>()

    for (c in 'A'..'F') {
        val binary = Integer.toBinaryString(c.code) // 책에서는 toInt()라고 했지만 deprecated 되어 변경
        binaryReps[c] = binary
    }

    for ((letter, binary) in binaryReps) {     // 맵에 대해 이터레이션한다. 맵의 키와 값을 두 변수에 각각 대입한다.
        println("$letter = $binary")
    }
}
```

자바의 get과 put 대신 map[key] 나 map[key] = value를 사용해 값을 가져오고 설정할 수 있다.

맵에 사용했던 구조 분해 구문을 맵이 아닌 컬렉션에도 활용할 수 있다.

인덱스를 저장하기 위한 변수를 별도로 선언하고 루프에서 매번 그 변수를 증가시킬 필요가 없다.

```kotlin
fun main() {
    val list = listOf("10", "11", "1001")
    for ((index, element) in list.withIndex()) {
        println("$index: $element")
    }
}
```

## 4.4 in으로 컬렉션이나 범위의 원소 검사

`in` 연산자를 사용해 어떤 값이 범위에 속하는지 검사할 수 있다.

반대로 `!in` 을 사용하면 어떤 값이 범위에 속하지 않는지 검사할 수 있다.

```kotlin
// in을 사용해 값이 범위에 속하는지 검사하기
fun main() {
    println(isLetter('q'))   // true
    println(isNotDigit('x')) // true
}

fun isLetter(c: Char) = c in 'a'..'z' || c in 'A'..'Z'
fun isNotDigit(c: Char) = c !in '0'..'9'
```

when 식에서도 사용해도 된다.

```kotlin
fun main() {
    println(recognize('d'))
}

fun recognize(c: Char) = when(c) {
    in '0'..'9' -> "숫자"
    in 'a'..'z', in 'A'..'Z' -> "문자열"
    else -> "잘모르겠읍니다"
}
```

# 5. 코틀린의 예외처리

코틀린의 기본 예외 처리구문은 자바와 비슷하다.

## 5.1 try, catch, finally

```kotlin
fun readNumber(reader: BufferedReader) : Int? {   // 함수가 던질 수 있는 예외를 명시할 필요가 없다.
    try {
        val line = reader.readLine()
        return Integer.parseInt(line)
    } catch (e: NumberFormatException) {
        return null
    } finally {
        reader.close()
    }
}
```

자바에서는 함수 선언 뒤에 `throws IOException` 을 붙여야한다. IOException은 체크 예외이기 때문이다.

자바에서 체크 예외는 명시적으로 처리해야한다.

코틀린은 체크 예외와 언체크 예외를 구별하지 않는다. 코틀린에서는 함수가 던지는 예외를 지정하지 않고 발생한 예외를 잡아내도되고 안잡아도 된다.

자바 7의 자원을 사용하는 `try-with-resource` 는 어떻게 사용할까?

코틀린은 그런 경우를 위한 특별한 문법을 제공하지 않고 라이브러리 함수로 같은 기능을 구현한다.

## 5.2 try를 식으로 사용

```kotlin
fun readNumber(reader: BufferedReader) {
    val number = try {
        Integer.parseInt(reader.readLine())
    } catch (e: NumberFormatException) {
        return
    }
    println(number)
}
```
