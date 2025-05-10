## 8장

### 원시타입과 래퍼타입?
- 자바에서는 원시타입과 참조타입을 구분한다.
- 원시타입은 값을 직접 저장하고, 참조타입은 메모리상의 객체 위치가 들어간다.
- 원시타입은 메서드를 호출하거나, 컬렉션에 담을 수 없기 때문에 참조 타입이 필요한 경우, 래퍼 타입으로 원시 타입을 감싸서 사용한다 (`int` -> `Integer` 등)
- 하지만 코틀린은 원시 타입과 래퍼 타입을 구분하지 않는다.
- 항상 참조타입을 사용하는것이 아니라, 실행 시점에 가장 효율적인 값으로 표현된다
  - 대부분의 경우 원시타입으로 사용
  - 제네릭 클래스, 컬렉션은 래퍼 타입으로 사용

| 항목          | 원시 타입 (Primitive) | 래퍼 타입 (Wrapper)    |
|-------------|-------------------|--------------------|
| 저장 방식       | 값 자체 저장           | 객체로 감싸서 저장         |
| 메모리 사용량     | 적음                | 상대적으로 많음           |
| null 허용 여부  | 불가                | 가능                 |
| 메서드/프로퍼티 사용 | 불가                | 가능                 |
| 컬렉션/제네릭 클래스 | 불가                | 가능                 |
| 예시          | int, boolean 등    | Integer, Boolean 등 |

---

#### 부호 없는 숫자 타입
- 동일한 메모리를 사용하지만, 부호 구분이 아닌 더 큰 양수 범위를 표현하도록 한다.
- `Int` 가 약 -21억에서 21억까지를 표현한다고 한다면, `UInt`는 0에서 약 42억까지를 표현한다.
- 하지만 명시적으로 전체 비트 범위가 필요한 경우가 아니라면 일반적으로 보통의 정수를 사용하고, 음수 범위가 함수에 전달되었는지 검사하는게 낫다. 

---

#### 널이 될수 있는 기본 타입
- 코틀린에서는 원시 타입과 래퍼 타입을 구분하지 않는다고 했다.
- 따라서 코틀린에서 `null`이 들어갈 수 있는 원시 타입을 사용하면 그 타입은 자바의 래퍼 타입으로 컴파일 된다.
```kotlin
val a: Int = 1    // 원시타입인 int로 컴파일
val b: Int? = 1   // null이 가능하므로 Integer 객체로 컴파일
```

---
#### 수 변환
- 코틀린은 한 타입의 수를 다른 타입의 수로 자동 변환하지 않는다.
- 변환할 타입의 범위가 더 넓더라도 자동 변환은 불가능하다
- 코틀린은 명시적 변환만 허용하여, 의도하지 않은 버그가 생기는걸 방지한다.
- 대신 원시 타입에 대해 변환 함수를 제공한다 (`toByte()`, `toShort()`, `toChar()` 등)
  - 표현 범위가 더 넓은 타입으로 변환하는 함수도 있고(`Int.toLong()`), 범위가 더 좁은 범위로 변환하며 일부를 잘라내는 함수도 있다(`Long.toInt()`)
  - 변환함수를 사용하다보면 각 타입 포맷에 맞지 않아 `NumberFormatException` 등이 발생할 수 있는데, 변환 실패시 `null`을 반환하는 함수들도 있다 (`toIntOrNull`, `toByteOrNull`)

---

### Any, Unit, Nothing
- 자바를 했더라도 추측하기가 힘든 3인방이다.


#### Any
- Kotlin 의 `null` 이 될수 없는 모든 타입의 최상위 타입.
- Java 의 Object 와 비슷하지만, `equals`, `hashCode`, `toString`만 기본으로 제공한다.
- Java 의 Object 는 원시 타입은 포함되지 않지만, 코틀린의 Any 는 자동으로 값을 객체로 감싼다

---

#### Unit
- 자바에서의 `void` 역할을 하며, 반환 타입을 명시하지 않더라도 `Unit` 반환형을 사용했다고 볼 수 있다
- 자바의 `void` 와 달리 타입 인자로 사용할 수 있다.

```kotlin
interface Processor<T> {
  fun process(): T
}

class NoResultProcessor : Processor<Unit> {
  override fun process() {  // Unit 반환 타입은 생략할 수 있다. void 와 동일하기 때문에 return 이 없다
    // Todo
  }
}
```

---

#### Nothing
- 절대 정상적으로 반환하지 않는 함수의 반환 타입
- 리턴값이 없다는 뜻이 아니라, 예를들어 무조건 예외가 발생하거나 무한 루프를 도는 함수로, 값을 반환하며 정상적으로 종료되지 않는다는 뜻.
```kotlin
fun fail(message: String): Nothing {
    throw IllegalArgumentException(message)
}

val address = company.address ?: fail("No Address")
print(address.city)
```
- `Nothing` 타입은 아무 값도 포함하지 않고, 변수를 선언하더라도 아무 값도 저장할 수 없기에, 위와같은 식은 항상 값이 존재한다고 볼 수 있다.

---

### 컬렉션과 배열
- `null`이 될 수 있는 값의 컬렉션과 `null`이 될 수 없는 컬렉션은 당연하게도 데이터 타입에 따른다. 
- `?` 기호가 존재한다면 `null` 혹은 해당 데이터타입이 들어갈 수 있는 컬렉션이다.
  - `List<Int>` -> `null` 불가
  - `List<Int?>` -> `null` 가능
- 그런데 컬렉션 원소가 아니라, 컬렉션 자체가 `null` 이 될수 있냐는 전혀 다른 이야기이다.
  - `List<Int?>` -> 컬렉션은 null이 될수 없고, 컬렉션 내부 원소는 null이 들어갈 수 있다.
  - `List<Int>?` -> 컬렉션은 null이 될수 있고, 컬렉션 내부 원소는 null이 들어갈 수 없다.

---
- 코틀린에서 `null` 을 다루는게 중점적인것 처럼, 표준 라이브러리에서도 다양하게 `null` 인 원소를 거르는 함수를 제공한다.
- `filterNotNull` -> null 이 아닌 리스트 반환
```kotlin
fun printValidNumbers(numbers: List<Int?>) {
    numbers.filterNotNull().map { println(it) }
}

// 1, 3 만 출력
fun call() = printValidNumbers(listOf(1, null, 3))
```

---
#### 읽기 전용과 변경 가능한 컬렉션
- 코틀린에서 `var` 와 `val` 가 있고, 그중에서 불변 타입인 `val` 을 지향하라는것 처럼, 컬렉션에서도 불변 사용을 지향한다.
- 변경 가능한 컬렉션인 `MutableCollection` 은 읽기 전용 인터페이스인 `Collection` 을 확장하여 추가, 삭제, 전체삭제 등의 메서드를 지원한다.

```kotlin
val readOnly = listOf(1, 2, 3)
val mutable = mutableListOf(1, 2, 3)

mutable.add(4) // 가능
// readOnly.add(4) // 컴파일 에러
```

---

#### 코틀린 컬렉션과 자바컬렉션
- 코틀린의 컬렉션 기본 구조는 자방 컬렉션 인터페이스 구조와 같다.
- 변경 가능 인터페이스는 자신과 대응하는 읽기 전용 인터페이스를 상속하고 있다.
- 코틀린은 자바 표준클래스가 코틀린의 변경 가능한 컬렉션을 상속한 것 처럼 취급한다.
- 이로, 자바 호환성을 제공하는 한편 읽기 전용 인터페이스와 변경 가능 인터페이스를 분리한다.


| 컬렉션 타입 | 읽기 전용 타입     | 변경 가능 타입                                                    |
|--------|--------------|-------------------------------------------------------------|
| List   | listOf, List | mutableListOf, MutableList, arrayListOf, buildList          |
| Set    | setOf        | mutableSetOf, hashSetOf, linkedSetOf, sortedSetOf, buildSet |
| Map    | mapOf        | mutableMapOf, hashMapOf, linkedMapOf, sortedmapOf, buildMap |

- 자바는 코틀린과 달리 읽기 전용 컬렉션과, 변경 가능 컬렉션을 구분하지 않기 때문에, 코틀린에서 읽기전용으로 선언되더라도 자바에서는 변경 가능하다.
- 자바 코드가 컬렉션을 변경할지 여부에 따라 파라미터 타입 사용의 책임은 사용자에게 있다....
- `toList()`, `toMutableList()` 등 복사본을 사용하여 별도의 복사본으로 만드는것이 안전하다.
---
- 자바에서 코틀린으로 넘어와서, `null` 인지 알수없는 플랫폼타입들이 존재한다.
- 해당 타입은 컴파일러가 `null` 관련 오류를 강제하지 않는다.
- 사용자가 명시적으로 `?` 기호를 추가하여 `null` 가능 타입으로 받거나, `null` 검사 방어로직을 추가해야한다.
- 