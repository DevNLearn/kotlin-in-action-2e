# 8. 기본타입, 컬렉션, 배열

### 원시 타입과 기본 타입

- 자바와 달리 코틀린에서는 원시 타입과 래퍼 타입을 구분하지 않는다.
- 코틀린에서는 가능한 한 원시 타입을 사용하고, 래퍼 타입은 필요할 때만 사용한다.
- 예를 들어, `Int` 타입을 컬렉션의 타입 파라미터로 넘기면, 그 컬렉션에는 `Int`의 래퍼 타입에 해당하는 `Integer`가 들어간다.
    - 정수 타입: `Byte`, `Short`, `Int`, `Long`
    - 부동소수점 숫자 타입: `Float`, `Double`
    - 문자 타입: `Char`
    - 불리언 타입: `Boolean`

### 부호 없는 숫자 타입

```kotlin
fun main() {
    val unsignedInt: UInt = 3_000_000_000u
    println("Unsigned Int: $unsignedInt")
}

/*
Unsigned Int: 3000000000
 */
```

- 코틀린에서는 부호 없는 숫자 타입을 지원한다.
- 부호 없는 숫자 타입들은 상응하는 부호 있는 타입의 범위를 '시프트'해서 같은 크기의 메모리를 사용해 더 큰 양수 범위를 표현할 수 있게 해준다.

| 타입       | 크기   | 값 범위         |
|----------|------|--------------|
| `UByte`  | 8비트  | 0 ~ 255      |
| `UShort` | 16비트 | 0 ~ 65535    |
| `UInt`   | 32비트 | 0 ~ 2^32 - 1 |
| `ULong`  | 64비트 | 0 ~ 2^64 - 1 |

### 널이 될 수 있는 기본 타입: `Int?`, `Boolean?` 등

- null 참조를 자바의 참조 타입의 변수에만 대입할 수 있기 때문에 널이 될 수 있는 코틀린 타입은 자바 원시 타입으로 편할 수 없다.
  - **ex. Java의 int에는 기본값 0이 들어감 (null 이 없음)**

```kotlin
data class Person(val name: String,
    val age: Int? = null) {
    fun isOlderThan(other: Person): Boolean? {
        if (age == null || other.age == null) {
            return null
        }

        return age > other.age
    }
}

fun main() {
    println(Person("Sam", 35).isOlderThan(Person("Amy", 42))) // false
    println(Person("Sam", 35).isOlderThan(Person("Jane"))) // null
}
```

- 널이 될 가능성이 있는 `Int?` 타입의 값을 직접 비교할 수 없다.
- 대신, 두 값이 모두 널이 아닌지 검사해야 한다.
- 컴파일러는 `null` 검사를 마친 다음에야 두 값을 일반적인 값처럼 다루도록 허용한다.

```kotlin
val listOfInts: List<Int> = listOf(1, 2, 3)
```

- 제네릭 클래스의 경우, 래퍼 타입을 사용한다.
- 제네릭은 원시타입을 허용하지 않기 때문에 위의 예시에서는 `List<Int>`는 `List<Integer>`로 변환된다.

### 수 변환

- 코틀린과 자바의 가장 큰 차이점 중 하나는 수를 변환하는 방식이다.
- 코틀린은 한 타입의 수를 다른 타입의 수로 자동 변환하지 않는다.

```kotlin
val i = 1
val l: Long = i // Error: Type mismatch
```

- 위의 예시에서 `i`는 `Int` 타입이고, `l`은 `Long` 타입이다.
- `Long`으로 변환하기 위해서는 `toLong()`과 같은 변환 메서드를 호출해야한다.

```kotlin
fun main() {
    val x = 1
    println(x.toLong() in listOf(1L, 2L, 3L))
    // true
}
```

- 코드에서 동시에 여러 숫자 타입을 사용하려면, 예상치 못한 동작을 피하기 위해 각 변수를 명시적으로 변환해야 한다.

```kotlin
fun printALong(l: Long) = println(l)

fun main() {
    val b: Byte = 1
    val l = b + 1L // +는 Byte와 
    printALong(42) // 컴파일러는 42를 Long으로 해석
}
```

- 숫자 리터럴을 사용할 때는 변환 함수를 호출할 필요가 없다. 
- `42L`나 `42.0f` 처럼 상수 뒤에 타입을 표현하는 문자를 붙이면 변환이 필요 없다.

```kotlin
fun main() {
    println(Int.MAX_VALUE + 1) // -2147483648
    println(Int.MIN_VALUE - 1) // 2147483647
}
```
- 자바와 똑같이 숫자 연산 시 오버플로나 언더플로가 발생할 수 있다.

```kotlin
fun main() {
    println("seven".toInt()) // NumberFormatException
    println("seven".toIntOrNull()) // null
}
```
- 문자열을 원시 타입으로 변환하려는 경우, 숫자 형식이 아닐때 `NumberFormatException` 예외가 발생한다.
- 이를 방지하기 위해 `toIntOrNull()` 함수를 사용하면, 변환이 실패할 경우 `null`을 반환한다.

### 코틀린 타입 계층의 뿌리: `Any`와 `Any?`

- 자바에서 `Object`가 클래스 계층의 최상위 타입이듯 코틀린에서는 `Any` 타입이 모든 **널이 될 수 없는** 타입의 조상 타입이다.
- 코틀린에서 원시 타입`(Int, Boolean ...)` 또한 `Any` 타입의 하위 타입이다.
- `Any` 타입은 `null`이 될 수 없는 타입임에 유의해야한다.
- 코틀린 함수가 `Any`를 사용하면 자바 바이트 코드의 `Object`로 변환된다.
- `Object`에 있는 `wait`, `notify` 같은 메서드들은 `Any`에서 사용할 수 없으므로, 사용하려면 `Object` 타입으로 캐스트해야한다.

### 코틀린의 void: `Unit`

```kotlin
fun f(): Unit { /* ... */ }
fun f() { /* ... */ }
```

- 코틀린 `Unit` 타입은 자바의 `void`와 같은 기능을 한다.
- 반환 타입 선언 없이 정의한 블록은 `Unit` 타입을 반환한다.
- `void`와 달리 `Unit`은 타입이기 때문에, `Unit` 을 타입 인자로 쓸 수 있다.

```kotlin
interface Processor<T> {
  fun process(): T
}

class NoResultProcessor : Processor<Unit> {
    override fun process() {
        // 업무 처리 코드
    }
}
```

- 인터페이스에서 `process` 함수가 어떤 값을 반환하라고 요구하지만, 제네릭으로 지정된 타입이 `Unit`이기 때문에 반환값이 필요 없다.
- 컴파일러가 암시적으로 `return Unit`을 넣어준다.

### 이 함수는 결코 반환되지 않는다 : `Nothing`

- 코틀린에는 결코 성공적으로 값을 돌려주는 일이 없으므로 **반환값**이라는 개념 자체가 의미가 없는 함수가 일부 존재한다.
- 예를 들어, 예외를 던지거나 무한 루프에 빠지는 함수가 있다.
- 그런 함수를 호출하는 코드를 분석하는 경우에 함수가 정상적으로 끝나지 않는다는 것을 표현하기 위해 `Nothing` 타입을 사용한다.

```kotlin
fun fail(message: String): Nothing {
  throw IllegalStateException(message)
}

fun main() {
  fail("Error occurred")
  // Exception in thread "main" java.lang.IllegalStateException: Error occurred
}
```

### 널이 될 수 있는 값의 컬렉션과 널이 될 수 있는 컬렉션

- 널이 될 수 있는 값의 컬렉션은 `List<String?>`와 같이 표현한다.
- 널이 될 수 있는 컬렉션은 `List<String>?`와 같이 표현한다.

```kotlin
fun readNumbers(text: String): List<Int?> {
    return text.lineSequence()
        .map { line -> line.toIntOrNull() }
        .toList()
}
```

- `List<Int?>`는 `Int?` 타입의 값을 저장할 수 있으므로,
- 이 타입의 리스트에는 `Int`나 `null`을 저장할 수 있다.

```kotlin
fun addValidNumbers(numbers: List<Int?>) {
    var sumOfValidNumbers = 0
    var invalidNumbers = 0
    for (number in numbers) {
        if (number != null) {
            sumOfValidNumbers += number
        } else {
            invalidNumbers++
        }
        /*
        number?.let { sumOfValidNumbers += it }
            ?: invalidNumbers++
         */
    }
}
```

- 위의 예시와 같이 컬렉션을 다룰때 `null`을 핸들링 해줘야한다.

```kotlin
fun addValidNumbers(numbers: List<Int?>) {
    val validNumbers: List<Int> = numbers.filterNotNull()
    println("Sum of valid numbers: ${validNumbers.sum()}")
    println("Invalid numbers: ${numbers.size - validNumbers.size}")
}
```

- `filterNotNull()` 함수를 사용하면 `null`이 아닌 값만 남길 수 있다.
- `filterNotNull()`은 `List<Int?>`를 `List<Int>`로 변환한다.

### 읽기 전용과 변경 가능한 컬렉션

- 코틀린에서는 읽기 전용 컬렉션과 변경 가능한 컬렉션을 구분한다.
- `Collection` 인터페이스는 읽기 전용 컬렉션을 나타내고, `MutableCollection` 인터페이스는 변경 가능한 컬렉션을 나타낸다.

```kotlin
fun <T> copyElements(source: Collection<T>, target: MutableCollection<T>) =
    source.forEach { target.add(it) }

fun main() {
    val source: Collection<Int> = arrayListOf(3, 5, 7)
    val target: MutableCollection<Int> = arrayListOf(1)

    copyElements(source, target)
    println(target) // [1, 3, 5, 7]
}
```

- 컬렉션과 마찬가지로 Map 클래스도 코틀린에서 Map과 MutableMap으로 구분된다.

| 컬렉션 타입 | 읽기 전용 타입 | 변경 가능 타입                                         |
|--------|----------------|--------------------------------------------------|
| List   |listOf, List | mutableListOf, MutableList, arrayListOf, buildList |
| Set    |setOf|mutableSetOf, hashSetOf, linkedSetOf, sortedSetOf, buildSet|
| Map    |mapOf|mutableMapOf, hashMapOf, linkedMapOf, sortedMapOf, buildMap|

- setOf()와 mapOf()는 읽기 전용 인터페이스의 인스턴스를 반환하지만, 내부적으로는 변경 가능한 클래스다.
- 따라서, 자바 메서드를 호출할때 컬렉션을 인자로 넘겨야하는 경우, 따로 변환하거나 복사하는 등의 추가 작업 없이 직접 컬렉션을 넘기는 경우 위험하다.

### 자바에서 선언한 컬렉션은 코틀린에서는 플랫폼 타입으로 보인다.

- 자바 코드에서 정의한 타입은 `@Nullable` 같은 어노테이션을 붙지지 않는한 코틀린에서는 플랫폼 타입으로 보인다.
- 자바에서 선언한 컬렉션도 마찬가지로 컬렉션 타입의 변수를 코틀린에서는 플랫폼 타입으로 본다.
- 컬렉션 타입이 시그니처에 들어간 자바 메서드 구현을 오버라이드 하는 경우, 문제가 된다.
  - 어떤 코틀린 컬렉션 타입으로 표현할지를 결정해야함.
    - 컬렉션이 `null`이 될 수 있는지
    - 컬렉션의 원소가 `null`이 될 수 있는지
    - 작성할 메서드가 컬렉션을 변경할 수 있는지

```java
/* Java */
interface FileContentProcessor {
    void processContents(
            File path,
            byte[] binaryContents,
            List<String> textContents
    );
}
```

```kotlin
/* Kotlin */
class FileIndexer : FileContentProcessor {
    override fun processContents(
      path: File, 
      binaryContents: ByteArray?,  
      textContents: List<String>?
    ) {
        // ...
    }
}
```

- 일부 파일이 이진 파일이고, 이진 파일 내용은 텍스트로 표현할 수 없는 경우가 있으므로 리스트는 `null`이 될 수 있다.
- 파일의 각줄은 `null`이 될 수 없으므로, 리스트의 원소는 `null`이 될 수 없다.
- 리스트는 파일의 내용을 표현하고, 그 내용을 바꿀 필요가 없기 때문에 읽기 전용인 `List`다.

➡️ `List<String>?` 

### 성능과 상호운용을 위한 원시 타입 배열

```kotlin
// 배열 사용 방법
fun main(args: Array<String>) {
    for (i in args.indices) { // 배열의 인덱스 값의 범위에 대한 이터레이션
        println("Argument $i is: ${args[i]}") // array[index]로 인덱스를 사용해 배열 원소에 접근
    }
}
```

- 코틀린에서는 배열을 인자로 받는 자바 함수를 호출하거나, `vararg` 파라미터를 받는 코틀린 함수를 호출하기 위해 배열을 만든다.
- 이때 데이터가 이미 컬렉션에 들어가있는 경우, 컬렉션을 배열로 변환해야 한다.

```kotlin
fun main() {
    val strings = listOf("a", "b", "c")
  println("%s/%s/%s".format(*strings.toTypedArray())) // vararg 인자를 넘기기 위해 스프레드 연산자(*) 사용
}
```

- `toTypedArray()` 메서드는 리스트를 배열로 변환한다.
- 제네릭 타입에서처럼 배열 타입의 타입 인자도 항상 객체 타입이 된다.
- 따라서, `Array<Int>` 같은 타입을 선언하면, 그 배열은 박싱된 정수의 배열 `Integer[]`이 된다.
- 박싱하지 않은 원시 타입의 배열 `int[]` 등이 필요한 경우, `ByteArray(byte[])`, `IntArray(int[])`, `BooleanArray(boolean[])` 특별한 배열 클래스를 사용해야한다.

### 원시 타입 배열

```kotlin
val fiveZeros = IntArray(5)
val fiveZerosToo = intArrayOf(0, 0, 0, 0, 0)
```

- 각 배열 타입의 생성자는 `size` 인자를 받아 해당 원시 타입의 기본값이 0으로 초기화된 `size` 크기의 배열을 반환한다.
- 박싱된 값이 들어있는 컬렉션이나 배열이 있다면, `toIntArray` 등의 변환 함수를 사용해 박싱하지 않은 원시 타입 값이 들어있는 배열로 변환할 수도 있다.
