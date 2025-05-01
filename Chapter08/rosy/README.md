# 8장 기본 타입, 컬렉션, 배열
코틀린 타입 시스템은 기존 언어들의 문제를 반영해 설계되었으며, 읽기 쉬운 코드 작성을 돕는다. 읽기 전용 컬렉션과 배열 지원을 강화해 타입 시스템을 더 깔끔하고 직관적으로 다듬었다.

---

## 8.1 원시 타입과 기본 타입

- 코틀린에서는 모든 기본 타입이 클래스다. (`Int`, `Double`, `Char` 등)

    ```kotlin
    val i: Int = 1
    val list: List<Int> = listOf(1, 2, 3)
    ```

- 성능을 위해 컴파일 시 JVM의 기본형(primitive type) (`int`, `double` 등)으로 변환해서 사용한다.

- 코틀린 타입
    
    | 타입 종류 | 설명 |
    | --- | --- |
    | Byte, Short, Int, Long | 정수 타입 |
    | Float, Double | 실수 타입 |
    | Char | 문자 타입 |
    | Boolean | 참/거짓 논리 타입 |

### 부호 없는 숫자 타입

- 코틀린은 JVM의 일반적인 원시 타입을 확장해 부호 없는 타입도 제공한다.

    | 타입 | 크기 |
    | --- | --- |
    | UByte | 부호 없는 8비트 정수 (0 ~ 255) |
    | UShort | 부호 없는 16비트 정수 (0 ~ 65535) |
    | UInt | 부호 없는 32비트 정수 (0 ~ 2³²-1) |
    | ULong | 부호 없는 64비트 정수 (0 ~ 2⁶⁴-1) |
- 명시적으로 전체 비트 범위가 필요한 경우가 아니라면 일반적으로 보통의 정수를 사용하고 음수 범위가 함수에 전달됐는지 검사하는 편이 낫다.

### 널이 될 수 있는 기본 타입

- 널이 될 수 있는 코틀린은 자바 원시 타입으로 표현할 수 없다.
- 널이 될 수 있는 원시 타입을 사용하면 그 타입은 자바의 래퍼 타입으로 컴파일 된다.

```kotlin
data class Person(val name: String,
									val age: Int? = null) {
	fun isOlderThan(other: Person): Boolean? {
		if (age == null || other.age == null)
			return null
		return age > other.age
	}
}

fun main() {
	println(Person("Sam", 35).isOlderThan("Amy", 42)) // false
	println(Person("Sam", 35).isOlderThan("Jane")) // null
}
```

- 널이 될 수 있는 기본 타입은 내부적으로 **객체 타입(래퍼 타입)** 으로 컴파일된다.
    - `Int?` → `java.lang.Integer`
- JVM은 타입 인자로 원시 타입을 허용하지 않는다.
- 자바나 코틀린 모두 제네릭 클래스는 항상 박스 타입을 사용해야한다.
- 원시 타입으로 이뤄진 대규모 컬렉션을 효율적으로 저장하고자 한다면 원시 타입으로 이루어진 효율적인 컬렉션을 제공하는  이클립스 컬렉션즈 등의 서드파티 라이브러리를 사용하거나 배열을 사용해야 한다.

### 수 변환

- 코틀린과 자바의 가장 큰 차이점 중 하나는 수를 변환하는 방식이다.
- 코틀린에서는 타입 간 자동 변환이 일어나지 않는다. (자바와 다름)
- 명시적으로 변환 메서드를 호출해야 한다.

    | 변환 메서드 | 설명 |
    | --- | --- |
    | `toByte()` | Byte로 변환 |
    | `toShort()` | Short로 변환 |
    | `toInt()` | Int로 변환 |
    | `toLong()` | Long으로 변환 |
    | `toFloat()` | Float으로 변환 |
    | `toDouble()` | Double로 변환 |

> **원시 타입 리터럴**
> - L 접미사가 붙은 Long 타입 리터럴: 123L
> - 표준 부동소수점 표기법을 사용한 Double 타입 리터럴: 0.12, 2.0, 1.2e10, 1.2e-10
> - f나 F접미사가 붙은 Float 타입 리터럴: 123.4f, .456F, 1e3f
> - 0x나 0x 접두사가 붙은 16진 리터럴: 0xCAFEBABE, 0xbcdl
> - 0b나 0B 접두사가 붙은 2진 리터럴: 0b000000101
> - U 접미사가 붙은 부호 없는 정수 리터럴: 123U, 123UL, 0x10cU
>
- 코틀린 산술 연산자에서도 자바와 똑같이 숫자 연산 시 오버플로나 언더플로가 발생할 수 있다.

    ```kotlin
    fun main() {
    	println(Int.MAX_VALUE + 1) // -2147483648
    	println(Int.MIN_VALUE - 1) // 2147483647
    ```


### Any와 Any?

- Any: 코틀린 클래스 계층의 최상위 타입. (자바의 `Object`와 비슷)
- Any?: Any에 `null`을 허용한 버전.
- `Any`가 제공하는 기본 메서드
    - `equals()`
    - `hashCode()`
    - `toString()`

```kotlin
val answer: Any = 42
```

### 코틀린의 void: Unit 타입

- 코틀린은 반환 타입이 없는 함수에 `Unit`을 사용한다.
- 자바의 `void`와 비슷하지만, `Unit`은 타입 인자이다.
- `Unit` 타입에는 유일한 값인 `Unit` 객체가 존재한다.

```kotlin
interface Processor<T> {
	fun process(): T
}

class NoResultProcessor: Processor<Unit> {
	override fun process() {
		// ..
	}
}
```

### Nothing 타입

- Nothing 타입은 결코 정상적으로 끝나지 않는 함수의 반환 타입이다.
    - 예를 들어, 예외를 던지는 함수가 해당된다.

    ```kotlin
    fun fail(message: String): Nothing {
        throw IllegalArgumentException(message)
    }
    ```

- `Nothing` 은 어떤 타입의 하위 타입이기 때문에, 타입 추론시 편리하게 사용할 수 있다.

---

## 8.2 컬렉션과 배열

### 널이 될 수 있는 값의 컬렉션과 널이 될 수 없는 컬렉션

- **널이 될 수 있는 값**을 담는 컬렉션: 리스트 안에 null이 있을 수 있다.
- **널이 될 수 있는 컬렉션**: 리스트 자체가 null일 수 있다.

    ```kotlin
    fun readNumbers(text: String): List<Int?> {
        val result = mutableListOf<Int?>()
        for (line in text.lineSequence()) {
            val numberOrNull = line.toIntOrNull()
            result.add(numberOrNull)
        }
        return result
    }
    ```

- `List<Int?>` 타입은 Int 또는 null을 담을 수 있는 리스트다.
- 각 줄을 읽어 정수로 변환할 수 있으면 Int로 저장하고, 변환할 수 없으면 null로 저장한다.
- `filterNotNull()` 은 컬렉션에서 null 값을 제거해준다.
- 널이 어디에 붙는지 주의❕
    - `List<Int?>`: 리스트 안의 요소가 Int나 null.
    - `List<Int>?`: 리스트 자체가 null일 수 있음.
- 널이 될 수 있는 값으로 이뤄진 리스트 다루기

    ```kotlin
    fun addValidNumbers(numbers: List<Int?>): Int {
        var sumOfValidNumbers = 0
        for (number in numbers) {
            number?.let { sumOfValidNumbers += it }
        }
        return sumOfValidNumbers
    }
    ```

    - `number?.let {}`를 사용해서 null이 아닌 경우에만 합계에 추가한다.
    - null 값은 무시된다.

### 읽기 전용과 변경 가능한 컬렉션

- Kotlin 컬렉션은 크게 **읽기 전용(read-only)** 컬렉션과 **변경 가능(mutable)** 컬렉션으로 나뉜다.
- `List`, `Set`, `Map`은 읽기 전용 인터페이스이고, `MutableList`, `MutableSet`, `MutableMap`은 변경 가능한 컬렉션을 위한 인터페이스이다.

```kotlin
val source: Collection<Int> = arraylistOf(3, 5, 7)
val target: MutableCollection<Int> = arraylistOf(1)
target.addAll(source)
```

- `source`는 읽기 전용 리스트이고, `target`은 변경 가능한 리스트이다.

> **Kotlin에서는 읽기 전용 컬렉션이어도 내부적으로는 변경 가능한 구조를 가질 수 있음.**
>
>
> (진짜 "불변(immutable)"과는 구별해야 한다.)
>

### 코틀린 컬렉션과 자바 컬렉션

- 코틀린 컬렉션은 내부적으로 **자바 컬렉션**을 기반으로 만들어졌다.
- 따라서 자바와의 상호운용이 쉽다.

    | 컬렉션 타입 | 읽기 전용 타입 | 변경 가능 타입 |
    | --- | --- | --- |
    | List | listOf, List | mutableListOf, MutableList, arrayListOf, buildList |
    | Set | setOf | mutableSetOf, hashSetOf, linkedSetOf, sortedSetOf, buildSet |
    | Map | mapOf | mutableMapOf, hashMapOf, linkedMapOf, sortedMapOf, buildMap |
- 겉으로는 읽기 전용처럼 보이지만, **내부 객체는 여전히 변경 가능**할 수 있다.
- 코틀린 타입 시스템은 "읽기 전용 인터페이스"만 제공할 뿐, **컬렉션 객체 자체를 불변으로 만들진 않는다.**

```kotlin
// CollectionUtils. java
public class CollectionUtils (
	public static List‹String> uppercaseAll(List<String› items) { 
		for (int i = 0; i ‹ Items.Size(); i++) { 
			items. set (i, items.get (1) • toUpperCase());
		}
		return items;
	}
}
	
// 코틀린 코드
// collections. kt
fun printInuppercase(list: List string>) {
	println(CollectionUtils.uppercaseAll(list)) 
	printin(list. first())
}
fun main() {
	val list = listof("a", "b", "C")
	printInUppercase(list)
	// [A, B, C]
	// A
}
```

- 코틀린에서는 `List`, `Set`, `Map`이 읽기 전용 인터페이스를 제공하지만,
- 자바 코드로 넘기면 내부 객체는 자유롭게 수정될 수 있다.
- 즉, 코틀린 타입만 믿고 리스트가 절대 안 바뀔 거라고 생각하면 안 된다.

### 자바에서 선언한 컬렉션은 코틀린에서 플랫폼 타입

- 자바 컬렉션을 코틀린으로 가져오면 **널 가능성(nullability)** 을 명확히 알 수 없다.
- 이럴 때 코틀린은 **플랫폼 타입**(ex: `List<String!>`)으로 취급한다.
- 플랫폼 타입은 널이 될 수도 있고 안 될 수도 있는 **"알 수 없는 타입"** 을 의미한다.
- 플랫폼 타입 컬렉션을 코틀린에서 다룰때 체크해야할 질문
    - 컬렉션이 null이 될 수 있는가?
    - 컬렉션의 원소가 null이 될 수 있는가?
    - 컬렉션을 변경할 수 있는가?

  > 위의 질문에 따라 타입 선언을 정확히 해야 안전한 코드를 만들 수 있다.

- FileContentProcessor 인터페이스

    ```java
    /* 자바 */
    interface FileContentProcessor (
    	void processcontents (
    		File path,
    		byte(] binaryContents,
    		List«String> textContents
    	);
    }
    ```

    - 이 인터페이스를 코틀린으로 구현하려면 다음을 선택해야한다.
        - **binaryContents**: 이진 파일이면 존재하지만, 아닐 수도 있으므로 → `ByteArray?`
        - **textContents**: 이진 파일이면 텍스트 없음 → 리스트 전체가 null 가능 → `List<String>?`
        - **리스트 안 원소**: 각 줄이 "문자열"이므로 원소는 null이 될 수 없음 → `String`
        - **리스트 수정 여부**: 파일 내용을 표현하는 거니까 읽기 전용 → `List`
    - 다음은 이를 코틀린으로 구현한 모습이다.

        ```kotlin
        class FileIndexer : FileContentProcessor {
        	override fun processContents (
        		path: File, 
        		binaryContents: ByteArray?,
        		textContents: List<String›?
        	){
        		// ...
        	}
        }
        ```

- DataParser 인터페이스

    ```kotlin
    /* 자바 */
    interface DataParser<T> {
    	void parseData(
    		String input, 
    		List<T> output, 
    		List<String> errors
    	);
    }
    ```

    - 이 인터페이스를 코틀린으로 구현하려면 다음을 선택해야한다.
        - **output**: 출력 결과를 저장할 리스트 → 추가/수정해야 하니까 → `MutableList<T>`
        - **errors**: 파싱 오류를 기록할 리스트
            - 리스트 자체는 **null이 되면 안 된다** (오류 항상 받을 것)
            - 하지만 리스트 안 **원소(String)는 null 가능** (오류가 없을 수도)
        - **리스트 수정 여부**: 오류 메시지를 추가해야 하므로 → `MutableList`
    - 다음은 이를 코틀린으로 구현한 모습이다.

        ```kotlin
        class PersonParser : DataParser‹Person> {
        	override fun parseData( 
        		input: String, 
        		output: MutableList<Person>, 
        		errors: MutableList‹String?› 
        	){
        		// ...
        	}
        }
        ```

- 플랫폼 타입을 코틀린에서 사용할 때는 **null 가능성 + 수정 가능성**을 명확히 판단하고 타입을 선언해야 한다.
- **List<String>?** : 리스트 전체가 null 가능
- **List<String?>** : 리스트 안 원소가 null 가능
- **MutableList** 사용 여부: 리스트를 수정해야 하면 변경 가능 타입 사용
- 자바 API를 사용할 때 **"어떻게 사용할지를 고민한 후"** 코틀린 타입을 정확히 매핑하자.

### 객체의 배열이나 원시타입의 배열을 만들기

- 코틀린에서 배열은 특정 타입의 값들을 순서대로 저장하는 컨테이너이다.
- 코틀린의 배열은 제너릭 클래스 `Array<T>` 로 표현한다.

    ```kotlin
    fun main(args: Array<String>) {
    	for (i in args. indices) {
    		printin ("Argument $i is: ${args[i]}")
    	}
    }
    ```

- 코틀린에서 배열을 만드는 방법은 다양하다.
    - arrayof 함수는 배열에 직접 값을 넣어서 생성한다.

        ```kotlin
        val numbers = arrayOf(1, 2, 3)
        ```

    - arrayOfNulls 함수는 null로 채워진 배열을 만든다.

        ```kotlin
        val nulls = arrayOfNulls<String>(3) 
        // [null, null, null]
        ```

    - Array 생성자는 배열의 크기, 초기화 함수를 지정할 수 있다.

        ```kotlin
        val letters = Array<String>(26) { i -> ('a' + i).toString() }
        println(letters.joinToString("")) 
        // 출력: abcdefghijklmnopqrstuvwxyz
        ```

- 컬렉션이 있을 때 배열로 변환하려면 `toTypedArray()` 를 사용한다.

    ```kotlin
    fun main() {
    	val strings = listof ("a", "b", "с") 
    	pnintan ("%5/%5/%s" . format (*strings. toTypedArray ())
    	// a/b/c
    }
    ```

- 코틀린은 박싱(boxing) 비용을 줄이기 위해, 원시 타입용 배열 클래스를 별도로 제공한다.

    | 타입 | 원시 배열 클래스 |
    | --- | --- |
    | Int | IntArray |
    | Byte | ByteArray |
    | Char | CharArray |
    | Boolean | BooleanArray |
    | Long | LongArray |
    | Short | ShortArray |
    | Float | FloatArray |
    | Double | DoubleArray |
- 코틀린은 부호 없는 타입(UByte, UShort, UInt, ULong)에 대해서도 배열 클래스를 제공한다.
- 원시 타입 배열을 만드는 방법은 다음과 같다.
    - 기본값으로 초기화

        ```kotlin
        val fiveZeros = IntArray(5) // [0, 0, 0, 0, 0]
        ```

    - 값 명시

        ```kotlin
        val fiveZerosToo = intArrayOf(0, 0, 0, 0, 0)
        ```

    - 람다로 초기화

        ```kotlin
        val squares = IntArray(5) { i -> (i + 1) * (i + 1) }
        println(squares.joinToString())
        // 출력: 1, 4, 9, 16, 25
        ```

- 배열에도 컬렉션용 확장 함수(`filter`, `map`, `forEach` 등)를 사용할 수 있다.
- 단, **배열을 변환하면 결과는 리스트**가 된다.

    ```kotlin
    fun main(args: Array<String>) {
        args.forEachIndexed { index, element ->
            println("Argument $index is: $element")
        }
    }
    ```

## 요약
- **코틀린 기본 타입** (`Int`, `Double` 등)은 일반 클래스처럼 보이지만, **자바의 원시 타입**으로 컴파일된다.
- **부호 없는 수 클래스**(`UInt`, `ULong`)는 JVM에 대응 타입이 없지만, **인라인 클래스**를 이용해 **성능 손실 없이** 표현된다.
- **널이 될 수 있는 원시 타입** (`Int?`, `Boolean?`)은 **자바의 박싱 타입**(`Integer`, `Boolean`)으로 변환된다.
- **Any**는 모든 타입의 최상위 타입이고, **자바의 Object**에 대응된다.
- **Unit**은 자바의 `void`에 대응된다.
- **Nothing** 타입은 **함수가 정상 종료되지 않음을 나타내는 타입**이다.
- **플랫폼 타입**: 자바에서 넘어온 타입은 코틀린에서 **널 가능성 여부를 확정할 수 없다**. 개발자가 직접 널 가능성(?, 없음)을 판단해야 한다.
- **코틀린 컬렉션**은 **읽기 전용**과 **변경 가능** 컬렉션을 구분하여, 더 안전하고 명확하게 설계되었다.
- 자바 클래스를 상속하거나 인터페이스를 구현할 때는, **널 가능성과 변경 가능성**을 **주의 깊게 고려**해야 한다.
- **코틀린 배열**(`Array`)은 일반 제네릭처럼 보이지만 **자바 배열**로 컴파일된다.
- **원시 타입 배열**은 `IntArray`, `CharArray` 등 **전용 배열 클래스**로 표현되며, **박싱 없이 효율적**으로 처리된다.