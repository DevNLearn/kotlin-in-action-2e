# 2. 코틀린 기초

### Hello World!

```kotlin
fun main() {
    println("Hello, World!")
}
```

- `fun`: 함수 선언
- `main`: 함수 이름
- `println`: 콘솔에 출력하는 함수
- 세미콜론 없음 `(안붙이는걸 권장)`

### 반환 값이 있는 함수 선언

```kotlin
fun max(a: Int, b: Int): Int {
    return if (a > b) a else b
}
```

- a, b 둘다 `Int` 타입
- 매개변수 뒤에 오는 객체가 리턴할 객체이다.
- 코틀린에서 if는 결과를 만드는 표현식이라서 값이 리턴이된다.

### 간단한 함수 간결하게 정의

```kotlin
fun max(a: Int, b: Int): Int = if (a > b) a else b
fun max(a: Int, b: Int) = if (a > b) a else b // 리턴을 안붙여도 Int로 추론됨.
```

### val과 var의 사용
- `val`: 읽기 전용 변수 (final)
  - 단 한번만 초기화 가능
- `var`: 읽기/쓰기 가능 변수
  - 초기화가 이뤄진 다음이라도 다른 값을 대입할 수 있다.

기본적으로 `val`로 선언하는게 좋고, 변수가 꼭 변경이 필요한 경우에만 `var`로 선언한다.

`val` 에 선언된 객체는 바꿀 수 없지만, 객체 내부의 값은 변경될 수 있다.

```kotlin
// MutableList 는 내부의 값을 변경할 수 있다.
fun main() {
  val languages: MutableList<String> = mutableListOf("Java", "Kotlin", "Scala")
  languages.add("Groovy")

  println(languages)
}

/*
[Java, Kotlin, Scala, Groovy]
 */
```

### 문자열 템플릿 (최고!)

```kotlin
fun main() {
    val input = readln()
    val name = if (input.isNotBlank()) input else "Kotlin"
    println("Hello, $name!")
    println("Hello, ${name.uppercase()}!") // 변경도 가능
}
```

$를 출력하고 싶은 경우에는 이스케이프 문자를 넣어줘야한다.

```kotlin
fun main() {
    println("\$x")
}

/*
$x
 */
```

```kotlin
// 나의 경우 Spring의 @Value 어노테이션을 사용할때 주로 사용한다.

@Service
class MyService(
    @Value("\${my.property}") // 자바에서는 @Value("${my.property}")로 사용
    private val file: File,
)
```

### 클래스와 프로퍼티

```kotlin
class Person(
    val name: String,
    var isStudent: Boolean,
)
```

- `val`: 읽기 전용 프로퍼티, 게터가 자동으로 만들어진다.
- `var`: 읽기/쓰기 가능 프로퍼티, 게터와 세터가 자동으로 만들어진다.

```kotlin
class Person(
    val name: String,
    var isStudent: Boolean,
)

fun main() {
    val person = Person("철수", true)
    println(person.name)
    // 철수
    println(person.isStudent)
    // true
    person.isStudent = false
    println(person.isStudent)
    // false
}
```

```kotlin
// 커스텀 게터
class Rectangle(val height: Int, val width: Int) {
  val isSquare: Boolean
    get() {
      return height == width
    }
}

fun main() {
  val rectangle = Rectangle(41, 43)
  println(rectangle.isSquare)
}

/*
false
 */
```

### Enum 클래스

```kotlin
enum class Color {
    RED, ORANGE, YELLOW, GREEN, BLUE, INDIGO, VIOLET
}
```

```kotlin
enum class Color(
    val r: Int,
    val g: Int,
    val b: Int,
) {
    RED(255, 0, 0),
    ORANGE(255, 165, 0),
    YELLOW(255, 255, 0),
    GREEN(0, 255, 0),
    BLUE(0, 0, 255),
    INDIGO(75, 0, 130),
    VIOLET(238, 130, 238);
  
    fun rgb() = (r * 256 + g) * 256 + b
    fun print() {
        println("r: $r, g: $g, b: $b")
    }
}
```

### when 절

```kotlin
fun getWarmthFromSensor(): String {
    val color = Color.ORANGE
    return when (color) {
        Color.RED, Color.ORANGE, Color.YELLOW -> "warm (red = ${color.r})"
        Color.GREEN -> "neutral (green = ${color.g})"
        Color.BLUE, Color.INDIGO, Color.VIOLET -> "cold (blue = ${color.b})"
    }
}

fun main() {
    println(getWarmthFromSensor())
}
```

좀더 줄여쓰면 아래와 같이 줄일 수 있다.

```kotlin
fun getWarmthFromSensor(): String {
    return when (val color = Color.ORANGE) {
        Color.RED, Color.ORANGE, Color.YELLOW -> "warm (red = ${color.r})"
        Color.GREEN -> "neutral (green = ${color.g})"
        Color.BLUE, Color.INDIGO, Color.VIOLET -> "cold (blue = ${color.b})"
    }
}
```

```kotlin
fun mix(c1: Color, c2: Color) =
    when (setOf(c1, c2)) { // Set<Color>
        setOf(Color.RED, Color.YELLOW) -> Color.ORANGE
        setOf(Color.YELLOW, Color.BLUE) -> Color.GREEN
        setOf(Color.BLUE, Color.VIOLET) -> Color.INDIGO
        else -> throw Exception("Dirty color")
    }

fun main() {
    println(mix(Color.BLUE, Color.YELLOW))
}
```
인자 없는 when 절도 가능하다.

> 읽기는 더 어려워 졌다.

```kotlin
fun mixOptimized(c1: Color, c2: Color) = 
    when {
      (c1 == RED && c2 == YELLOW) || (c1 == YELLOW && c2 == RED) -> ORANGE
      (c1 == YELLOW && c2 == BLUE) || (c1 == BLUE && c2 == YELLOW) -> GREEN
      (c1 == BLUE && c2 == VIOLET) || (c1 == VIOLET && c2 == BLUE) -> INDIGO
      else -> throw Exception("Dirty color")
    }
```

### 이터레이션

```kotlin
val oneToTen: IntRange = 1..10 // 1 <= x <= 10
val oneToNine: IntRange = 1..<10 // 1 <= x < 10
```

```kotlin
for (i in 1..100) {
    // 1부터 100까지 순회
}

for (i in 100 downTo 1 step 2) {
    // 100부터 1까지 2씩 감소하며 순회
}
```

컬렉션에 대한 이터레이션은 `java`의 `for-each`와 유사하다.

```kotlin
fun main() {
  val collection = listOf("red", "green", "blue")
  for (color in collection) {
    print("$color ")
  }
}
/*
red green blue 
 */
```

문자도 이터레이션이 가능하고, `for-each` 문에서는 키와 맵을 동시에 받을 수도 있다.

```kotlin
fun main() {
    val binaryReps = mutableMapOf<Char, String>()

    for (char in 'A'..'F') {
        val binary = char.code.toString(radix = 2)
        binaryReps[char] = binary
    }

    for ((letter, binary) in binaryReps) {
        println("$letter = $binary")
    }
}
/*
A = 1000001
B = 1000010
C = 1000011
D = 1000100
E = 1000101
F = 1000110
 */
```

인덱스와 함께 컬렉션을 이터레이션 할 수도 있다.

```kotlin
fun main() {
  val list = listOf("10", "11", "1001")
  for ((index, element) in list.withIndex()) {
    println("$index: $element")
  }
}
/*
0: 10
1: 11
2: 1001
 */
```

이터레이션에서 사용되었던 in을 통해 어떤 값이 범위 속하는지 아닌지 검증가능!

```kotlin
fun isLetter(c: Char) = c in 'a'..'z' || c in 'A'..'Z'
fun isNotDigit(c: Char) = c !in '0'..'9'

fun main() {
  println(isLetter('q'))
  println(isNotDigit('x'))
}

/*
true
true
 */
```

### 문제 ###

책에서 나온 예시를 보면, `print("Kotlin" in setOf("Java", "Scala")`의 결과는 `false`가 나왔다.<br>
그렇다면, 아래의 결과는 어떻게 될까요?

```kotlin
fun main() {
  val map = mapOf(
    "Java" to "자바",
    "Kotlin" to "코틀린",
    "Python" to "파이썬",
    "Cpp" to "씨쁠쁠",
  )

  when ("Java" to "자바") {
    in map -> println("in map")
    else -> throw IllegalArgumentException("not in map")
  }
}

// 참고로 for-each 구문을 사용하면 아래와 같이 사용할 수 있다.
for ((key, value) in map) {
    println("$key = $value")
}
```
<details>
<summary>정답 확인하기</summary>

> 정답은 컴파일 오류!
> 
> `in` 키워드는 `if, when`과 같이 조건절에서 쓰일때와 `for` 문과 같이 순회할때 달라지는데<br>
> `if, when` 에서는 `in` 키워드가 `contains` 메서드를 호출하는 반면<br>
> `for` 문에서는 `iterator` 메서드를 호출한다.<br>
> 
> `in` 키워드가 `contains` 메서드를 호출하는 경우에는 `Pair` 객체를 인자로 받지 않기 때문에<br>
> 컴파일 오류가 발생한다.<br>
> 
> 만약, 해당 구문이 정상적으로 작동하기를 원한다면 아래와 같은 `contains` 확장함수를 구현해줘야 한다.
```kotlin
operator fun <K, V> Map<K, V>.contains(pair: Pair<K, V>): Boolean {
  return this[pair.first] == pair.second
}
```
</details>

