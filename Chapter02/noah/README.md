# 코틀린 기초
## 함수

---

함수를 선언할 때는 `fun` 를 사용한다. 함수는 `블록 본문 함수`와 `식 본문 함수` 로 표현할 수 있다.

### 블록 본문 함수(Block body function)

본문이 중괄호로 둘러싸인 함수

```kotlin
fun max(a: Int, b: Int): Int {
    return if (a > b) a else b
}
```

### 식 본문 함수(Expression body function)

등호와 식으로 이루어진 함수로 간결하게 표현할 수 있다. 식 본문 함수는 반환 타입만 생략 가능하다.

> 코틀린에서는 식 본분 함수가 자주 사용된다. 조건 검사나 자주 사용되는 연산에 기억하기 쉬운 이름을 부여 할 수 있고, `if`, `when`, `try` 에서 유용하게 사용된다.
> 

```kotlin
fun max(a: Int, b: Int): Int = if (a > b) a else b
```

## 변수

---

코틀린 변수 선언은 `val`과 `var` 이 있다. 

```kotlin
{키워드} {변수명}: {타입} = {값}
```

### val(value)

`val`은 `읽기 전용 참조(read-only reference)`를 선언하여 단 한 번만 대입 될 수 있다. 초기화하고 나면 다른 값을 대입 할 수 없다.

```kotlin
val question: String = "가나다"
```

 | `읽기 전용 참조`와 `변경 불가능한 객채`를 부수 효과가 없는 함수와 조합해 사용하면 함수형 프로그래밍이 제공하는 이점을 살릴 수 있다.

### var(variable)

`var`은 `재대입 가능한 참조(reassignable reference)`를 선언하여 초기화가 이뤄진 이후에도 다른 값을 대입할 수 있다.

```kotlin
var question = "가나다"
question = "abc"
```

## 클래스와 프로퍼티

코틀린에서 기본 `가시성 제어자(접근 제어자)`는 `public` 으로 생략 가능하다.

```kotlin
class Person(val name: String)
```

코틀린에서는 기본 기능으로 `프로퍼티`를 제공하여 자바의 필드와 접근자 메서드를 대신한다.

> 자바에서는 필드와 접근자를 묶어 프로퍼티(property)라고 한다.
> 
- 읽기 전용 프로퍼티(val):  비공개 필드와 Getter를 생성
- 쓸 수 있는 프로퍼티(var): 비공개 필드, Getter, Setter 생성

```kotlin
class Perseon(
   val name: String,
   var isStudent: Boolean
)
```

### 커스텀 접근자

- 커스텀 Getter 정의: 클래스의 특성을 기술하고 싶을때
- 멤버함수:  클래스의 행동을 기술하고 싶을때

```kotlin
class Rectangle(val height: Int, val width: Int) {
    val isSquare: Boolean
        get() { // 커스텀 getter 블록 본문
            return height == width
        }
}

class Rectangle(val height: Int, val width: Int) {
    val isSquare: Boolean
        get() = height == width // 커스텀 getter 식 본문
}

class Rectangle(val height: Int, val width: Int) {
    fun isSquare(): Boolean { // 멤버 함수(메서드) 정의
        return height == width
    }
}
```

## enum 과 when

### Enum

코틀린에서 `enum` 은 소프트 키워드이다.

> 소프트 키워드(Soft Keyword)는 문맥이 맞을 때만 예약어처럼 동작하고, 그 외에는 일반 식별자로 사용 가능한 키워드이다.
> 

```kotlin
enum class Color(
    val r: Int,
    val g: Int,
    val b: Int
){
    RED(255, 0, 0),
    GREEN(0, 255, 0),
    BLUE(0, 0, 255);
}
```

### when

`when` 은 자바의 `switch` 을 대신한다.

```kotlin
fun getMnemonic(color: Color) = 
    when (color) {
        Color.RED -> "빨강"
        Color.Green -> "초록"
        Color.BLUE -> "파랑"
    }
    
// 하나의 분기에 여러 값 사용 가능
fun getMnemonic(color: Color) = 
    when (color) { 
        Color.RED, Color.GREEN, Color.BLUE  -> "삼원색"
    }

// 조합을 검증 가능
fun mix(c1: Color, c2: Color) = 
    when (setOf(c1, c2)) {  // setOf는 코틀린 표준 라이브러리, 원소 순서는 무시된다.    
        setOf(RED, BLUE) -> "빨파"
        setOf(RED, GREEN) -> "빨초"
        setOf(BLUE, GREEN) -> "파초"
    }

//인자 없이 사용 가능
fun mixOptimized(c1: Color, c2: Color) = 
    when {
        (c1 == RED && c2 == BLUE) ||
        (c1 == BLUE && c2 == RED) -> "빨파"
        ...
    }
```

## while 과 for 루프

코틀린의 범위는 폐구간(닫힌구간)으로 양쪽을 포함하는 구간을 사용한다.

### 레이블 지정

코틀린에서 내포된 루프의 경우 레이블을 지정할 수 있다

```kotlin
outer@ while (outerCondition) { //outer라는 레이블 지정
    while (innerCondition) {
         if(shouldExitInner) break
         if(shouldSkipInner) continue
         if(shouldExit) break@outer
         if(shouldSkip) continue@outer
    }
}
   
```

### 범위와 순열

```kotlin
fun main() {
    for(i in 1..100) { // 1~100
        print(i)
    }
}

fun main() {
    for(i in 100 downTo 1 step 2) { // 역방향으로 증가 값 범위 이터레이션
        print(i)
    }
}

```

### 컬렉션에 대해 이터레이션

```kotlin
fun main() {
    val collection = listOf("red", "green", "blue")
    for (color in collection) {
        print("$color ")
    }
}
```

### 맵에 대해 이터레이션

```kotlin
fun main() {
    val binaryReps = mutableMapOf<Char, String>()
    for (char in 'A'..'F') {
        val binary = char.code.toString(redix = 2)
        binaryReps[char] = binary
    }
    
    for((letter, binary) in binaryReps) {
        println("$letter = $binary")
    }
}
```

### in으로 컬렉션이나 범위의 원소 검사

```kotlin
fun isLetter(c: char) = c in 'a'..'z' || c in 'A'..'Z'
fun main() {
    println(isLetter('q'))
}

fun main() {
    println("Kotlin" in setOf("Java", "Scala"))
}
```

## 예외 처리

코틀린에서는 예외 처리를 할때 `new` 키워드를 사용하지 않는다.

```kotlin
if (percentage !in 0..100) {
    throw IllegalArgumentException("exception")
}
```

### try-catch-finally

```kotlin
fun readNumber(reader: BufferedReader): Int? {
    try { // 블록 본문 처리
        val line = reader.readLine()
    } catch(e: NumberFormatException) {
        return null
    } finally{
        reader.close()
    }
}

fun readNumber(reader: BufferedReader) {
    val number = try {
        Integer.parserInt(reader.readLine())
    } catch(e: numberformatException) {
        return
    }
}
```