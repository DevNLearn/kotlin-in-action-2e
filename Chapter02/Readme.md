### 함수와 변수

1. 함수

```kotlin
fun main() { // fun : 함수 선언, main 인자 생략 가능
	println("Hello, World!") // 세미콜론 X
}

fun max(a: Int, b: Int): Int { // 블록 본문 함수
	return if (a > b) a else b
}

// return 을 없앨 수 있음

fun max(a: Int, b: Int): Int = if (a > b) a else b // 본문 함수
fun max(a: Int, b: Int = if (a > b) a else b // 타입 추론을 할 수 있어서 반환 타입이 필요 없음
```

문과 식의 구분 :

- ‘식’은 값을 만들어냄. 다른 식의 하위 요소로 계산에 참여 가능.
- ‘문’은 자신을 둘러싸는 가장 안쪽 블록의 최상위 요소로 존재하며 아무 값을 만들어내지 않음.
- 코틀린에서는 for, while, do/while을 제외한 대부분의 제어구조가 식 !!

```kotlin
val x = if (flag) 3 else 5
val dir = when (inputString) {
	"u" -> UP
	"d" -> DOWN
	else -> UNKNOWN
}
val num = try {
	inputString.toInt()
} catch (nfe: NumberFormatException) {
	-1
}
```

1. 변수

```kotlin
val question: String = "질문"
val answer: Int = 40

// 이렇게도 가능
val question = "질문"
val answer = 40

// 근데 즉시 초기화 하는게 아니고 나중에 대입하는거는 타입 추론 못함
fun main() {
	val answer: Int // 이렇게 해야함
	answer = 40
}
```

- `val`: 읽기 전용 함수 (val-ue) 자바에서 final
    - 읽기 전용이라 한 번 대입된 다음 그 값을 바꿀 순 없지만, 참조가 가리키는 객체의 내부 값은 변경될 수 있음. (예 : val이 가리키는 가변 리스트에 원소 추가)

        ```kotlin
        fun canPerformOperation(): Boolean {
        	return true
        }
        val result: String
        if (canPerformOperation()) { // canPerformOperation으로 한번만 초기화 된다면, 이렇게 사용도 가능
        	result = "Success"
        } else {
        	result = "Can't perform operation"
        }
        
        fun main() {
        	val languages = mutableListOf("Java") // 읽기 전용 참조 선언
        	languages.add("Kotlin") // 원소 추가 가능
        }
        ```

- `var` : 재대입 가능한 참조 (var-iable) 자바에서 일반 변수. 반드시 필요할 때만 !!! var로 써야 함.
    - var은 값 변경은 되지만 타입은 고정됨
    - 다른 타입 값을 저장하고 싶으면 변환 함수로 값을 바꾸거나, 강제 형 변환을 해야 함.

    ```kotlin
    fun main() {
    	var answer = 40
    	answer = "노답" // type mismatch 에러
    }
    ```


### 문자열 템플릿

```kotlin
fun main() {
	val input = readln()
	val name = if (input.isNotBlank()) input else "Kotlin"
	println("Hello, $name") // Hello Kotlin
}

// $을 문자처럼 쓰려면 백슬래시로 이스케이프
fun main() {
	println("\$x") // $x
}

// {} 내에 식도 사용 가능
fun main() {
	val name = readln()
	if (name.isNotBlank())
		println("Hello, ${name.length}-letter person!") // name 변수의 길이에 접근 가능 (한글 사이에 변수를 넣으려면 중괄호로 감싸야함)
}

// 아예 이렇게도 가능
fun main() {
	val name = readln()
	println("Hello ${if (name.isBlank()) "someone" else name}!")
}
```

### 클래스와 프로퍼티

```kotlin
// Java POJO
public class Person {
	private final String name;
	public Person(String name) {
		this.name = name;
	}
	public String getName() {
		return name;	
	}
}

// Kotlin
class Person(val name: String) // 코틀린 기본이 public이라서 생략해도 됨
```

- 프로퍼티 : 필드  + 접근자. 코틀린 기본 기능.

91p Preson 클래스 오탈자

```kotlin
class Person {
	val name: String, // getter만
	var isStudent: Boolean // getter & setter
}

// Java에서 하려면
person.getName()
person.setName("name")
// 근데 이름이 is로 시작하는 프로퍼티는 getter는 이름 그대로, setter는 is를 set으로 바꿔서
person.isStudent()
person.setStudent()

// Kotlin에서 하려면
person.name
println(person.isStudent) // true
person.isStudent = false // 이렇게 바로 setter 호출 가능
```

### 커스텀 접근자

어떤 프로퍼티가 같은 객체 안의 다른 프로퍼티에서 계산된 직접적인 결과일 때

```kotlin
class Rectangle(val height: Int, val width: Int) {
	val isSquare: Boolean
		get() { // 프로퍼티 getter 선언
			return height == width
		}
		
		// 이렇게도 됨
		// get() = height == width
}
```

### 디렉토리, 패키지

```kotlin
package geometry.example

import geometry.shapes.Rectangle
import geometry.shapes.createUnitSquare // 다른 패키지의 함수 임포트

fun main() {
	println(Rectangle(3, 4).isSquare) // false
	println(createUnitSqure().isSquare) // true
}
```

### enum, when
```kotlin
    enum class Color {
        RED, ORANGE, YELLOW, GREEN, BLUE
    }

    enum class Color (
            val r: Int,
            val g: Int,
            val b: Int
    ) {
        RED(255, 0, 0),
        ORANGE(255, 165, 0),
        YELLOW(255, 255, 0),
        GREEN(0, 255, 0),
        BLUE(0, 0, 255);
    
        fun rgb() = (r*256 + g) * 256 + b
        fun printColor() = println("$this is $rgb")
    }

    fun main() {
        println(Color.BLUE.rgb) // 255
        Color.GREEN.printColor() // GREEN is 65280
    }
    
    fun getMnemonic(color: Color) =
            when (color) {
                Color.RED -> "Richard" // break 없어도 됨
                Color.Orange -> "Of"
                Color.Yellow -> "York"
                Color.GREEN -> "Gave"
                Color.BLUE -> "Battle"
            }

    fun main() {
        println(getMnemonic(Color.BLUE)) // Battle
    }
```