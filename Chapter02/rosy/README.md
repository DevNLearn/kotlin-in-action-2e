# 2. 코틀린 기초
코틀린에서 가장 기본적인 요소: 함수, 변수, 클래스, 이넘, 프로퍼티, 제어 구조, 스마트 캐스트, 예외 처리 등을 배운다.

## 2.1 기본 요소: 함수와 변수

- `fun` 키워드를 사용해 함수 정의
  ```kotlin
  fun max(a: Int, b: Int): Int = if (a > b) a else b
  ```

- `val`: 읽기 전용 변수 / `var`: 재할당 가능한 변수
  - 단, `val`이더라도 **참조값의 내부 상태는 변경 가능**
- 타입 추론 가능하지만 명시적으로 적을 수도 있음
- 블록 본문(`{}`) 함수는 `return` 필요, 표현식 본문(`=`) 함수는 생략 가능
- **문자열 템플릿**으로 문자열 연결이 간편
  - `$변수명` 또는 `${식}` 사용 가능

---

## 2.2 행동과 데이터 캡슐화: 클래스와 프로퍼티

- `class`로 클래스 정의. 생성자 파라미터는 기본적으로 프로퍼티가 아님
- `val`, `var`를 사용하면 프로퍼티로 자동 선언됨
- `init` 블록은 생성자 호출 시 실행되는 초기화 코드 블록
- 커스텀 getter/setter 지원 (읽기 전용 `val`에도 `get()` 커스터마이징 가능)

    ```kotlin
    class Rectangle(val height: Int, val width: Int) {
        val isSquare: Boolean
            get() = height == width
    }
    ```

- 클래스 내부 로직과 데이터를 캡슐화하여 구조적인 코드 작성 가능
- 패키지와 디렉터리 구조는 자바와 유사

---

## 2.3 선택 표현과 처리: 이넘과 when

코틀린에서는 다양한 선택 표현을 효과적으로 처리할 수 있도록 `enum class`와 `when` 표현식을 제공한다.

- 코틀린의 `enum class`는 열거형 상수에 이름뿐만 아니라 프로퍼티와 메서드도 정의할 수 있어 유연하다.
- `when`은 자바의 `switch`보다 표현력이 강력하며, `enum`, 값, 타입 등 다양한 조건을 처리할 수 있다.

📌 enum 클래스 정의

```kotlin
enum class Color { RED, ORANGE, YELLOW, GREEN, BLUE, INDIGO, VIOLET }
```

📌  프로퍼티와 메서드를 갖는 enum

```kotlin
enum class Color(val r: Int, val g: Int, val b: Int) {
    RED(255, 0, 0),
    GREEN(0, 255, 0),
    BLUE(0, 0, 255);

    fun rgb() = (r * 256 + g) * 256 + b
}
```

📌  when을 사용한 분기 표현

```kotlin
fun getMnemonic(color: Color): String = 
	when (color) {
    Color.RED -> "Richard"
    Color.ORANGE -> "Of"
    Color.YELLOW -> "York"
		Color.GREEN -> "Gave"
		Color.BLUE -> "Battle"
		Color.INDIGO -> "In"
		Color.VIOLET -> "Vain"
}
```

📌 enum 값을 조합한 분기

- `setOf()`처럼 조건을 집합으로 만들면 순서 상관없이 매칭 가능.

```kotlin
fun mix(c1: Color, c2: Color) = 
	when (setOf(c1, c2)) {
    setOf(RED, YELLOW) -> ORANGE
    setOf(YELLOW, BLUE) -> GREEN
    setOf(BLUE, VIOLET) -> INDIGO
    else -> throw Exception("Dirty color")
}
```

📌 인자 없는 when 사용

- 조건 자체가 표현식이 되며, 여러 분기 값을 동시에 반환 가능.

```kotlin
fun mixOptimized(c1: Color, c2: Color) =
    when {
        (c1 == RED && c2 == YELLOW) || (c1 == YELLOW && c2 == RED) -> ORANGE
        (c1 == YELLOW && c2 == BLUE) || (c1 == BLUE && c2 == YELLOW) -> GREEN
        (c1 == BLUE && c2 == VIOLET) || (c1 == VIOLET && c2 == BLUE) -> INDIGO
        else -> throw Exception("Dirty color")
    }
```

📌 타입 검사와 스마트 캐스트

- 타입 검사 후 `as` 없이 자동으로 그 타입으로 캐스팅된다.

```kotlin
interface Expr
class Num(val value: Int) : Expr
class Sum(val left: Expr, val right: Expr) : Expr

fun eval(e: Expr): Int =
    when (e) {
        is Num -> e.value
        is Sum -> eval(e.left) + eval(e.right)
        else -> throw IllegalArgumentException("Unknown expression")
    }
```

---

## 2.4 대상 이터레이션: while과 for 루프

- `while`, `do-while`은 다른 언어와 동일한 구조를 사용

  → 조건을 만족할 때까지 반복 수행

- `break`, `continue`로 루프 제어 가능

  → 레이블(`@label`)을 사용해 바깥 루프 제어도 가능

    ```kotlin
    outer@ while (true) {
        while (true) {
            if (needBreak) break@outer
        }
    }
    ```


### 📌 범위와 순열

- `..` 연산자로 범위 생성 (예: `1..10`)
- `downTo`, `step`을 이용해 역순 또는 간격 지정 가능

```kotlin
for (i in 1..100) {
    print(fizzBuzz(i))
}

for (i in 100 downTo 1 step 2) {
    print(fizzBuzz(i))
}
```

### 📌 맵에 대한 이터레이션

- 키/값 쌍을 구조 분해하여 순회 가능

```kotlin
val binaryReps = mutableMapOf<Char, String>()
for (char in 'A'..'F') {
    val binary = char.code.toString(2)
    binaryReps[char] = binary
}
for ((letter, binary) in binaryReps) {
    println("$letter = $binary")
}

```

### 📌 인덱스와 함께 이터레이션

- `withIndex()`로 인덱스 + 값 동시 순회 가능
```kotlin
val list = listOf("10", "11", "1001")
for ((index, element) in list.withIndex()) {
    println("$index: $element")
}
```

### 📌 in과 !in을 사용한 범위 검사

```kotlin
fun isLetter(c: Char) = c in 'a'..'z' || c in 'A'..'Z'
fun isNotDigit(c: Char) = c !in '0'..'9'
```

---

## 2.5 코틀린에서 예외 던지고 잡아내기

- 코틀린은 예외를 **자바처럼** `try`, `catch`, `finally`로 처리한다.
- `throw`를 사용해 예외를 던질 수 있다. (코틀린은 **함수가 던지는 예외를 선언할 필요가 없다.**)
- `throw`는 식이므로 `if`와 함께 사용 가능
  ```kotlin
  if (percentage !in 0..100) {
    throw IllegalArgumentException("Invalid: $percentage")
  }
  ```

📌 예외 처리 기본 구조

```kotlin
fun readNumber(reader: BufferedReader): Int? {
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

📌 try를 식으로 사용

```kotlin
val number = try {
    Integer.parseInt(reader.readLine())
} catch (e: NumberFormatException) {
    null
}
println(number)
```

- `try`와 `catch` 블록 모두 값을 반환해야 함
- 예외가 발생하지 않으면 `try`의 마지막 식이 반환됨
- 예외 발생 시 `catch`의 마지막 식이 반환됨

---

## 🟢 흥미로웠던 포인트

- **문자열 템플릿** 덕분에 `+`로 문자열 연결할 필요 없음

  → `${변수}` 또는 `$변수`만으로 간단하게 삽입 가능

- **for 루프의 다양성**

  → 맵을 순회하거나 인덱스 포함 이터레이션이 **자바보다 훨씬 직관적**

- 코틀린의 `throw`는 **식**이다

  → 다른 식 안에서도 사용할 수 있음