# 고차 함수: 람다를 파라미터와 반환값으로 사용

### 다른 함수를 인자로 받거나 반환하는 함수 : 고차 함수

- 고차 함수는 다른 함수를 인자로 받거나, 함수를 반환하는 함수다.
- 코틀린에서는 람다나 함수 참조를 사용해 함수를 값으로 표현할 수 있다.

```kotlin
list.filter { it > 0 }
```

- 고차함수 타입을 정의하려면 함수 파라미터의 타입을 괄호 안에 넣고, 그 뒤에 화살표`(->)`를 추가한 다음, 함수의 반환 타입을 지정하면 된다.

```kotlin
(Int, String) -> Unit

// ex)
val sum: (Int, Int) -> Int = { x, y -> x + y }
val action: () -> Unit = { println("Hello") }
```

- 다른 함수와 같이 널이 가능한 타입으로도 지정할 수 있다.

```kotlin
var canReturnNull: (Int, Int) -> Int? = { x, y -> null }
```

- 추가로, 함수 자체가 널인 타입도 만들 수 있다. (단, 괄호로 감싸서 물음표를 붙여야함.)

```kotlin
val funOrNull: ((Int, Int) -> Int)? = null
```

```kotlin
fun twoAndThree(operation: (Int, Int) -> Int) {
    val result = operation(2, 3)
    println("The result is $result")
}

fun main() {
    twoAndThree { a, b -> a + b } // The result is 5
    twoAndThree { a, b -> a * b } // The result is 6
}
```

- 함수 타입에서 파라미터 이름을 지정할 수도 있다.

```kotlin
fun twoAndThree(operation: (operandA: Int, operandB: Int) -> Int) {
    val result = operation(2, 3)
    println("The result is $result")
}

fun main() {
    twoAndThree { a, b -> a + b } // The result is 5
    twoAndThree { a, b -> a * b } // The result is 6
}
```

### filter 함수 구현해보기

```kotlin
fun String.filter(predicate: (Char) -> Boolean): String {
    return buildString {
        for (char in this@filter) {
            if (predicate(char)) append(char)
        }
    }
}

fun main() {
    println("ab1c".filter { it in 'a'..'z' })
    // abc
}
```

### 자바에서 코틀린 함수 타입 사용

- 자동 SAM 변환을 통해 코틀린 람다를 함수형 인터페이스를 요구하는 자바 메서드에게 넘길 수 있다.

```kotlin
// kotlin 선언
fun processTheAnswer(f: (Int) -> Int) {
    println(f(42))
}

// java
processTheAnswer(number -> number + 1)

// kotlin
processTheAnswer { number -> number + 1 }
```

### 함수 타입의 파라미터에 대해 기본값을 지정할 수 있고, 널이 될 수도 있다.

- 파라미터를 함수 타입으로 선언할때에도 기본값을 지정할 수 있다.

```kotlin
fun <T> Collection<T>.joinToString(
    separator: String = ", ",
    prefix: String = "",
    postfix: String = "",
): String {
    val result = StringBuilder(prefix)

    for ((index, element) in this.withIndex()) {
        if (index > 0) {
            result.append(separator)
        }
        result.append(element)
    }
    result.append(postfix)
    return result.toString()
}
```

- 이 구현은 유연하지만, 핵심 요소인 컬렉션의 각 원소를 문자열로 변환하는 방법을 제어할 수 없다는 단점이 존재한다.
- 이럴때 `transform` 람다 파라미터를 추가하면 해결이 되지만, 매번 `toString()` 같은 함수를 정의해줘야하기 때문에 오히려 더 불편해진다.
- 이럴때 기본값으로 람다식을 넣어 불편을 해결할 수 있다.

```kotlin
fun <T> Collection<T>.joinToString(
    separator: String = ", ",
    prefix: String = "",
    postfix: String = "",
    transform: (T) -> String = { it.toString() }
): String {
    val result = StringBuilder(prefix)

    for ((index, element) in this.withIndex()) {
        if (index > 0) {
            result.append(separator)
        }
        result.append(transform(element))
    }
    result.append(postfix)
    return result.toString()
}

fun main() {
    val letters = listOf("Alpha", "Beta")
    println(letters.joinToString())
    // Alpha, Beta

    println(letters.joinToString { it.lowercase() })
    // alpha, beta

    println(letters.joinToString(separator = "! ", postfix = "! ") { it.uppercase() })
    // ALPHA! BETA!
}
```

- 널이 될 수 있는 함수 타입으로 사용할 수도 있다.
- 그대로 호출하면 NPE가 발생하기 때문에, 호출하기 전에 널 체크를 해줘야한다.

```kotlin
fun <T> Collection<T>.joinToString(
    separator: String = ", ",
    prefix: String = "",
    postfix: String = "",
    transform: ((T) -> String)? = null,
): String {
    val result = StringBuilder(prefix)

    for ((index, element) in this.withIndex()) {
        if (index > 0) {
            result.append(separator)
        }
        val str = transform?.invoke(element) ?: element.toString()
        result.append(str)
    }
    result.append(postfix)
    return result.toString()
}
```

### 함수를 함수에서 반환

- 함수를 반환하는 함수가 사용되는 경우는 적지만, 아래와 같은 상황에서 유용하게 사용할 수 있다.
- 예를 들어, 사용자가 선택한 배송 수단에 따라 배송비를 계산하는 방법이 다른 경우.

```kotlin
enum class Delivery {
    STANDARD, EXPEDITED;
}

class Order(val itemCount: Int)

fun getShippingCostCalculator(delivery: Delivery): (Order) -> Double {
    return when (delivery) {
        Delivery.STANDARD -> { order: Order -> 1.2 * order.itemCount }
        Delivery.EXPEDITED -> { order: Order -> 6 + 2.1 * order.itemCount }
    }
}

fun main() {
    val calculator = getShippingCostCalculator(Delivery.EXPEDITED)
    println("Shipping costs ${calculator(Order(3))}")
    // Shipping costs 12.3
}
```

### 람다를 활용해 중복을 줄여 코드 재사용성 높이기

```kotlin
data class SiteVisit(
    val path: String,
    val duration: Double,
    val os: OS
)

enum class OS {
    WINDOWS,
    MAC,
    LINUX,
    IOS,
    ANDROID
}

val log = listOf(
    SiteVisit("/", 34.0, OS.WINDOWS),
    SiteVisit("/", 22.0, OS.MAC),
    SiteVisit("/login", 12.0, OS.WINDOWS),
    SiteVisit("/signup", 8.0, OS.IOS),
    SiteVisit("/", 16.3, OS.ANDROID)
)

val averageWindowsDuration = log
    .filter { it.os == OS.WINDOWS }
    .map(SiteVisit::duration)
    .average()

fun main() {
    println(averageWindowsDuration)
}
```

- 일반 함수를 통해 중복 제거

```kotlin
fun List<SiteVisit>.averageDurationFor(os: OS) =
    filter { it.os == os }.map(SiteVisit::duration).average()

fun main() {
    println(log.averageDurationFor(OS.WINDOWS))
    println(log.averageDurationFor(OS.MAC))
}
```

- 고차함수를 사용해 중복을 제거

```kotlin
fun List<SiteVisit>.averageDurationFor(predicate: (SiteVisit) -> Boolean) =
    filter(predicate).map(SiteVisit::duration).average()

fun main() {
    println(log.averageDurationFor { it.os == OS.WINDOWS })
    println(log.averageDurationFor { it.os in setOf(OS.WINDOWS, OS.MAC) })
    println(log.averageDurationFor { it.os == OS.WINDOWS && it.path == "/login" })
}
```

---

### 인라인 함수를 사용해 람다의 부가 비용 없애기

- 람다를 인자로 받는 고차 함수는 람다를 호출할 때마다 부가 비용이 발생한다. **(변수 캡처 시 호출 시점마다 새로운 객체 생성)**
- 이 부가 비용을 없애기 위해 인라인 함수를 사용한다.
- 인라인 함수는 람다를 호출하는 대신, 람다의 내용을 함수 본문에 직접 삽입한다.

```kotlin
import java.util.concurrent.locks.Lock
import java.util.concurrent.locks.ReentrantLock

inline fun <T> synchronized(lock: Lock, action: () -> T): T {
    lock.lock()
    try {
        return action()
    } finally {
        lock.unlock()
    }
}

fun main() {
    val l = ReentrantLock()
    synchronized(l) {
        100
    }
}

// 바이트 코드
fun main() {
    val l = ReentrantLock()
    lock.lock()
    val result: Int = try {
        100
    } finally {
        lock.unlock()
    }
}
```

### 인라인 함수의 제약

- 인라이닝을 하는 방식으로 인해 람다를 사용하는 모든 함수를 인라이닝할 수는 없다.
- 함수가 인라이닝될 때 그 함수에 인자로 전달된 람다식의 본문은 결과 코드에 직접 들어갈 수 있다.
- 하지만, 이렇게 람다가 본문에 직접 펼쳐지기 때문에 함수를 파라미터로 전달받은 람다를 본문에 사용하는 방식이 한정될 수밖에 없다.
- 예를 들어, 람다를 변수에 저장하거나, 다른 함수에 전달하는 방식은 사용할 수 없다.

```kotlin
class FunctionStorage {
    var myStoredFunction: ((Int) -> Unit)? = null
    inline fun storeFunction(f: (Int) -> Unit) {
        myStoredFunction = f // Compile error!
    }
}
```

- 둘 이상의 람다를 인자로 받는 함수에서 일부 람다만 인라이닝하고 싶은 경우
- `noinline` 키워드를 파라미터 이름 앞에 사용하면 인라이닝을 금지할 수 있다.

```kotlin
class FunctionStorage {
    var myStoredFunction: ((Int) -> Unit)? = null
    inline fun storeFunction(noinline f: (Int) -> Unit, block: () -> Unit) {
        myStoredFunction = f // Compile error!
        block.invoke()
    }
}
```

### 컬렉션 연산 인라이닝

- 코틀린 표중 라이브러리의 컬렉션 함수는 대부분 람다를 인자로 받는다.
- 이런 함수들은 인라이닝을 통해 성능을 높일 수 있다.

```kotlin
// 람다를 사용한 필터링
data class Person(val name: String, val age: Int)

val people = listOf(Person("Alice", 29), Person("Bob", 31))

fun main() {
    println(people.filter { it.age < 30 })
    // [Person(name=Alice, age=29)]
}

// 컬렉션을 직접 거르기
fun main() {
    val result = mutableListOf<Person>()
    for (person in people) {
        if (person.age < 30) result.add(person)
    }
    println(result)
    // [Person(name=Alice, age=29)]
}
```

- 표준 라이브러리의 `filter` 함수는 인라인 함수로 구현되어 있기 때문에 성능에 신경쓰지 않아도 된다.
- 람다를 사용한 필터링의 예제를 써서 생긴 바이트 코드는 컬렉션을 직접 거른 코드의 바이트 코드와 비슷하게 생성된다.
- 하지만, 이런 `filter`, `map` 같은 함수를 체이닝 형태로 만드는 경우 중간 리스트를 만들기 때문에 `Sequence`로 변경해 부가 비용을 줄여야한다.

```kotlin
public inline fun <T> Iterable<T>.filter(predicate: (T) -> Boolean): List<T> {
    return filterTo(ArrayList<T>(), predicate)
}
```

### 언제 함수를 인라인으로 선언할까?
인라이닝을 사용하면 성능이 향상되지만, 바이트 코드의 크기가 커지기 때문에 무조건 인라이닝을 사용하는 것은 좋지 않다.
- 고차 함수를 최적화할 때: 람다를 인자로 받는 함수에서 객체 생성 오버헤드 제거

```kotlin
inline fun executeWithLogging(action: () -> Unit) {
    println("실행 전")
    action()
    println("실행 후")
}
```

  - 타입 파라미터의 구체화가 필요할 때:

```kotlin
inline fun <reified T> isType(value: Any): Boolean {
    return value is T // 일반 함수에서는 불가능
}
```

  - 자주 호출되는 작은 함수에서 호출 오버헤드 제거할 때
  - 컬렉션 확장 함수처럼 짧고 자주 사용되는 유틸리티 함수
  - 람다 내에서 비지역 반환을 사용해야 할 때:

```kotlin
inline fun retry(times: Int, action: () -> Boolean): Boolean {
    for (i in 0 until times) {
        if (action()) return true // 람다에서 외부 함수 반환 가능
    }
    return false
}

// 또다른 예
fun findPerson(people: List<Person>) {
    people.forEach { person ->
        if (person.name == "Kim") {
            return@forEach  // 람다만 종료됨
        }
    }
    println("검색 완료")  // 항상 실행됨
}

fun findPerson(people: List<Person>) {
    people.forEach { person ->  // forEach는 inline 함수
        if (person.name == "Kim") {
            return  // findPerson 함수 자체를 종료
        }
    }
    println("Kim을 찾지 못했을 때만 실행")
}
```

### `withLock`, `use`, `useLines`로 자원 관리를 위해 인라인된 람다 사용

- 람다로 중복을 없앨 수 있는 일반적인 패턴 중 한 가지는 어떤 작업을 하기 전에 자원을 획득하고, 작업을 마친 후 자원을 해제하는 자원 관리다.
- 자원 관리 패턴을 만들 때 보통 사용하는 방법은 `try/finally` 문을 사용한다.
- 하지만, 이 방법은 코드가 길어지고 가독성이 떨어진다.
- `withLock`은 자바의 `ReentrantLock`을 사용해 락을 획득하고 해제하는 자원 관리 패턴을 구현한 것이다.

```kotlin
fun main() {
    val l: Lock = ReentrantLock()
    l.withLock {
        // 락에 의해 보호되는 자원을 사용
    }
}

public inline fun <T> Lock.withLock(action: () -> T): T {
    contract { callsInPlace(action, InvocationKind.EXACTLY_ONCE) }
    lock()
    try {
        return action()
    } finally {
        unlock()
    }
}
```

- `use`는 자바의 `Closeable` 인터페이스를 구현한 객체에 사용된다.
- 이 함수는 람다를 호출하고 사용 후 자원이 확실히 닫히게 한다.

```kotlin
import java.io.BufferedReader
import java.io.FileReader

fun readFirstLineFromFile(fileName: String): String {
    BufferedReader(FileReader(fileName)).use { br ->
        return br.readLine()
    }
}
```

- 여기서 사용된 `return`은 `use` 함수의 람다를 종료하는 것이 아니라, `readFirstLineFromFile` 함수를 종료한다.
- 그 이유는, `use` 함수가 인라인 함수이기 때문이다.

---

### 고차 함수에서 흐름 제어

루프와 같은 명령형 코드를 람다 형식의 함수형 코드로 변경하면서 `return` 문제에 부딪힌다.

### 람다 안의 return 문: 람다를 둘러싼 함수에서 반환

```kotlin
data class Person(val name: String, val age: Int)

val people = listOf(Person("Alice", 29), Person("Bob", 31))

fun lookForAlice(people: List<Person>) {
    for (person in people) {
        if (person.name == "Alice") {
            println("Found!")
            return
        }
    }
    println("Alice is not found")
}

fun main() {
    lookForAlice(people)
    // Found!
}
```

- `forEach`를 사용하더라도 마찬가지로 동일한 지점에서 함수가 끝난다.
- 그 이유는 `forEach`가 인라인 함수이기 때문이다.
- 이렇게 `return`이 바깥쪽 함수를 반환시킬 수 있는 때는 람다를 인자로 받는 함수가 인라인 함수인 경우 뿐이다.
- 이러한 `return` 문을 비로컬 `return`이라 부른다.

```kotlin
fun lookForAlice(people: List<Person>) {
    people.forEach {
        if (it.name == "Alice") {
            println("Found!")
            return
        }
    }
    println("Alice is not found")
}

fun main() {
    lookForAlice(people)
    // Found!
}
```

### 람다로부터 반환: 레이블을 사용한 return

- `return` 문에 레이블을 붙여서 바깥쪽 함수가 아닌 람다를 종료할 수 있다.
- 람다를 종료하는 `return` 문을 로컬 `return`이라고 부른다.
- 로컬 `return`은 `for`루프의 `break`와 비슷한 역할을 한다.

```kotlin
fun lookForAlice(people: List<Person>) {
    people.forEach label@{
        if (it.name == "Alice") {
            println("Found!")
            return@label
        }
    }
    println("This line reached!")
}

fun main() {
    lookForAlice(people)
    // Found!
    // This line reached!
}
```

- 함수 이름 자체를 레이블로 사용할 수도 있다.

```kotlin
fun lookForAlice(people: List<Person>) {
    people.forEach {
        if (it.name == "Alice") {
            println("Found!")
            return@forEach
        }
    }
    println("This line reached!")
}
```

- `this` 식의 레이블에도 마찬가지 규칙이 적용된다.

```kotlin
fun main() {
    println(StringBuilder().apply sb@{ 
        listOf(1, 2, 3).apply { 
            this@sb.append(this.toString())
        }
    })
    // [1, 2, 3]
}
```
