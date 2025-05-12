# 10장 고차 함수: 람다를 파라미터와 반환값으로 사용

## 10.1 다른 함수를 인자로 받거나 반환하는 함수 정의: 고차 함수

### **고차 함수란?**

- 고차 함수(Higher-Order Function)란 **다른 함수를 인자로 받거나, 함수를 반환하는 함수**를 말한다.
- 코틀린에서는 **람다(lambda)**나 **함수 참조**를 사용해 함수를 값처럼 전달할 수 있다.
- 예를 들어, 표준 라이브러리 함수인 `filter`는 조건에 맞는 요소를 걸러내기 위한 함수(술어 함수, Predicate)를 인자로 받기 때문에 고차 함수다.

    ```kotlin
    // 0보다 큰 값만 필터링
    val list = listOf(-2, 3, 0, 7, -5)
    val positives = list.filter { it > 0 } 
    println(positives) // [3, 7]
    ```


### 함수 타입 정의하기

- 고차 함수를 정의하려면, 먼저 **함수 타입(Function Type)**을 이해해야 한다.
- 함수 타입은 **파라미터 타입과 반환 타입**을 조합하여 정의한다.

**기본적인 함수 타입**

```kotlin
// 두 숫자를 더하는 람다 정의
val sum = { x: Int, y: Int -> x + y }

// 숫자를 출력하는 람다 정의
val action = { println(42) }

// 함수 타입을 명시적으로 지정
val sum: (Int, Int) -> Int = { x, y -> x + y }
val action: () -> Unit = { println(42) }
```

- `(Int, Int) -> Int`: 두 개의 `Int`를 받아서 `Int`를 반환하는 함수 타입
- `() -> Unit`: 인자가 없고 아무것도 반환하지 않는 함수 타입

**널이 될 수 있는 함수 타입**

- 함수 타입도 널(`null`)이 될 수 있다.
- 널이 될 수 있는 경우, 타입 전체를 괄호로 감싸고 `?`를 붙여야 한다.

    ```kotlin
    // 반환 값이 널일 수 있는 함수 타입
    var canReturnNull: (Int, Int) -> Int? = { x, y -> null }
    
    // 함수 자체가 널일 수 있는 함수 타입
    var funOrNull: ((Int, Int) -> Int)? = null
    ```


### 인자로 전달 받은 함수 호출하기

- 고차 함수는 함수 타입 파라미터를 받아서 호출할 수 있다.

**두 숫자를 받아 연산을 수행하는 고차 함수**

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

- `twoAndThree` 함수는 `(Int, Int) -> Int` 타입의 함수를 받아 실행한다.
- 고차 함수에 넘기는 람다 표현식으로 덧셈, 곱셈을 각각 전달할 수 있다.

**`filter` 함수를 단순하게 만든 버전**

- 코틀린의 표준 라이브러리에는 `filter` 함수가 있다.
- 이 함수는 리스트나 문자열에서 특정 조건에 맞는 값만 걸러낸다.
- 이 구조를 이해하기 쉽게 단순하게 구현해보자.

    ```kotlin
    fun String.filter(predicate: (Char) -> Boolean): String {
        // buildString은 StringBuilder를 생성하고, 최종 결과를 String으로 반환합니다.
        return buildString {
            // this@filter: filter의 수신 객체인 현재 문자열(String)을 의미합니다.
            for (char in this@filter) {
                if (predicate(char)) append(char)
            }
        }
    }
    
    fun main() {
        println("abc".filter { it in 'a'..'z' }) // abc
    }
    
    ```

    1. **buildString**
        - `buildString`은 내부적으로 `StringBuilder`를 사용하여 효율적으로 문자열을 누적하고, 최종적으로 `String`으로 반환한다.
    2. **this@filter**
        - `this@filter`는 현재 확장 함수가 호출된 **String 객체**를 가리킨다.
        - `for (char in this@filter)`는 문자열의 각 문자를 순회한다.
    3. **predicate**
        - `predicate`는 `(Char) -> Boolean` 타입의 함수이다.
        - 각 문자를 검사하고, 조건이 `true`인 경우에만 결과 문자열에 추가된다.

### 파라미터 이름과 함수 타입

- 코틀린에서는 **함수 타입의 각 파라미터에 이름을 지정할 수 있다.**
- 이 이름들은 IDE의 **코드 자동 완성** 시 유용하게 활용될 수 있다.

**함수 타입에 파라미터 이름 지정하기**

```kotlin
fun twoAndThree(
    operation: (operandA: Int, operandB: Int) -> Int
) {
    val result = operation(2, 3)
    println("The result is $result")
}

fun main() {
    // 지정한 이름으로 람다 표현식 작성
    twoAndThree { operandA, operandB -> operandA + operandB } // The result is 5
    
    // 다른 이름으로도 작성 가능
    twoAndThree { alpha, beta -> alpha + beta } // The result is 5
}
```

- 함수 타입에서 정의한 **파라미터 이름은 타입 검사 시 무시된다**.
- 예를 들어, `operandA`, `operandB` 대신 `alpha`, `beta`를 사용해도 문제없다.
- 하지만 **가독성**을 높이고, IDE에서 **코드 자동 완성** 시 유리하기 때문에 의미 있는 이름을 지정하는 것이 좋다.

### 자바에서 코틀린 함수 타입 사용

- 코틀린에서 작성한 함수 타입은 **자바에서 사용할 수 있다.**
- 코틀린의 함수 타입은 자바의 **SAM 인터페이스**로 변환되어 호출된다.

```kotlin
/* 코틀린 선언 */
fun processTheAnswer(f: (Int) -> Int) {
    println(f(42))
}

/* 자바 호출 */
processTheAnswer(number -> number + 1);
// 43
```

- `processTheAnswer`는 `(Int) -> Int` 타입의 함수를 받아 42에 적용한다.
- 자바에서는 **람다 표현식**을 그대로 전달할 수 있다.

### 함수 타입의 파라미터에 대해 기본값을 지정할 수 있고, 널이 될 수도 있다.

- 코틀린은 함수 타입의 파라미터에 **기본값(default value)**을 지정할 수 있다.
- 또한, **널이 될 수 있는 함수 타입**도 선언이 가능하다.

**함수 타입의 기본값 지정**

- 기본적으로 사용하는 로직이 있을 때, 이를 기본값으로 설정할 수 있다.

```kotlin
fun <T> Collection<T>.joinToString(
    separator: String = ", ",
    prefix: String = "",
    postfix: String = "",
    transform: (T) -> String = { it.toString() } // 기본값 설정
): String {
    val result = StringBuilder(prefix)
    for ((index, element) in this.withIndex()) {
        if (index > 0) result.append(separator)
        result.append(transform(element))
    }
    result.append(postfix)
    return result.toString()
}

fun main() {
    val letters = listOf("Alpha", "Beta")
    
    // 기본값으로 toString() 사용
    println(letters.joinToString()) // Alpha, Beta
    
    // 커스텀 변환 사용
    println(letters.joinToString(separator = "! ", postfix = "! ",
		    transform = { it.uppercase() })) // ALPHA! BETA!
}

```

**널이 될 수 있는 함수 타입**

- 함수 타입을 널이 될 수 있도록 선언하려면 **전체 타입을 괄호로 감싸고 `?`를 붙인다.**
- 호출 시에는 안전 호출(`?.invoke()`)을 사용해야 한다.

```kotlin
fun <T> Collection<T>.joinToString(
	separator: String = ", ",
	prefix: String = "",
	postfix: String = "",
	transform: ((T) -> String)? = null
): String {
	val result = StringBuilder(prefix)
	
	for((index, element) in this.withIndex())
		if (index > 0) result.append(separator)
		val str = transform?.invoke(element)
			?: element.toString()
		result.append(str)
	}
	
	result.append(postfix)
	return result.toString()
}
```

### 함수를 함수에서 반환

- 코틀린의 고차 함수는 함수를 **반환**할 수도 있다.
- 함수를 반환할 때도 `(파라미터 타입) -> 반환 타입`으로 정의한다.

**배송 옵션에 따른 배송비 계산 함수 반환하기**

```kotlin
enum class Delivery { STANDARD, EXPEDITED }

class Order(val itemCount: Int)

fun getShippingCostCalculator(delivery: Delivery): (Order) -> Double {
    if (delivery == Delivery.EXPEDITED) {
      return { itemCount -> 6 + 2.1 * itemCount }
    }
    return { itemCount -> 1.2 * itemCount }
   
}

fun main() {
    val calculator = getShippingCostCalculator(Delivery.EXPEDITED)
    println("Shipping costs: ${calculator(3)}") // Shipping costs: 12.3
}

```

- `getShippingCostCalculator`는 `Delivery` 타입에 맞춰 다른 계산 로직의 함수를 반환한다.
- 반환된 함수는 나중에 필요할 때 실행할 수 있다.

### 람다를 활용해 중복을 줄여 코드 재사용성 높이기

- 람다를 활용하면 **반복되는 로직을 추상화**하여 중복을 줄일 수 있다.
- 조건만 바꿔서 필터링을 쉽게 할 수 있다.

```kotlin
data class SiteVisit(val path: String, val duration: Double, val os: OS)
enum class OS { WINDOWS, LINUX, MAC, IOS, ANDROID }

val log = listOf(
    SiteVisit("/", 34.0, OS.WINDOWS),
    SiteVisit("/", 22.0, OS.MAC),
    SiteVisit("/login", 12.0, OS.WINDOWS),
    SiteVisit("/signup", 8.0, OS.IOS),
    SiteVisit("/", 16.3, OS.ANDROID)
)

fun List<SiteVisit>.averageDurationFor(predicate: (SiteVisit) -> Boolean): Double {
    return filter(predicate).map(SiteVisit::duration).average()
}

fun main() {
    println(log.averageDurationFor { it.os in setOf(OS.ANDROID, OS.IOS))}) // 12.15
    println(log.averageDurationFor { it.os == OS.IOS })     // 8.0
}

```

- `averageDurationFor`에 원하는 조건을 람다로 전달하면 중복 없이 필터링

---

## 10.2 인라인 함수를 사용해 람다의 부가 비용 없애기

### **람다 표현식의 부가 비용**

- 코틀린에서 **람다**를 함수 인자로 넘기는 구문은 일반적인 제어 구조(`if`, `for`)와 매우 유사하게 보인다.
- 하지만 실제로 람다를 사용하면 **익명 클래스(Anonymous Class)**가 생성된다.
- 이로 인해, **람다식이 실행될 때마다 객체가 생성**되며, **메모리 사용과 GC 비용**이 발생한다

### 인라이닝이 작동하는 방식

- 코틀린은 람다의 부가 비용을 줄이기 위해 **inline** 키워드를 제공한다.
- **inline**으로 선언된 함수는 **실제 호출되는 부분에 함수의 본문이 그대로 대체된다**.
- 즉, 호출하지 않고 **복사 붙여넣기**처럼 동작하게 된다.

**인라인 함수 정의하기**

```kotlin
import java.util.concurrent.locks.Lock
import java.util.concurrent.locks.ReentrantLock

// inline 키워드를 사용하여 인라인 함수 선언
inline fun <T> synchronized(lock: Lock, action: () -> T): T {
    lock.lock()
    try {
        // 전달받은 람다 실행
        return action()
    } finally {
        lock.unlock()
    }
}

fun main() {
    val l = ReentrantLock()
    synchronized(1) {
        //...
    }
}
```

**인라이닝된 바이트코드의 동작**

- 인라인 함수로 선언된 `synchronized`는 함수 호출이 아닌 **코드 복사**가 발생한다.
- 즉, 컴파일 후 바이트코드는 다음과 비슷하게 번역된다.

```kotlin
fun foo(l: Lock) {
	println("Before sync")
	synchronized(l) {
		println("Action!")
	}
	println("After sync")
}

fun __foo__(l: Lock) {
	println("Before sync")
	l.lock()
	try {
		println("Action!")
	} finally {
		l.unlock()
	}
	println("After sync")
}
```

- `synchronized` 함수 호출이 사라지고, 그 본문이 **직접 코드로 복사된다.**

### 인라인 함수의 제약

- 모든 함수가 인라이닝될 수 있는 것은 아니다.
- 만약 함수의 파라미터로 받은 람다를 **다른 변수에 저장**하거나 **다른 함수로 전달**하면 인라이닝할 수 없다.

**인라인 함수에서 변수에 저장 시 오류 발생**

```kotlin
class FunctionStorage {
    var myStoredFunction: ((Int) -> Unit)? = null

    // inline 함수에서 람다를 저장하면 컴파일 에러 발생
    inline fun storeFunction(f: (Int) -> Unit) {
        myStoredFunction = f // 오류 발생: 인라인 함수는 람다를 변수에 저장할 수 없습니다.
    }
}
```

- `Illegal usage of inline-parameter` 오류가 발생한다.
- 인라인 함수는 **변수에 저장할 수 없고** **그 자리에서 실행만 가능하다.**

**noinline 키워드를 사용하여 예외 처리**

- 만약 일부 람다만 인라이닝하지 않으려면 **noinline** 키워드를 사용할 수 있다.

```kotlin
inline fun fun(inlined: () -> Unit, noinline notInlined: () -> Unit) {
	// ...    
}
```

### 컬렉션 연산 인라이닝

- 코틀린의 표준 라이브러리에서 제공하는 컬렉션 함수(`filter`, `map`)는 **인라인 함수**이다.
- 따라서 람다 표현식이 별도의 객체를 생성하지 않고 직접 호출된다

```kotlin
data class Person(val name: String, val age: Int)

val people = listOf(Person("Alice", 29), Person("Bob", 31))

fun main() {
    // filter는 인라인 함수이므로 객체 생성 없이 바로 실행
    println(people.filter { it.age < 30 })
    // [Person(name=Alice, age=29)]
}
```

- filter는 **인라인 함수**로 동작하여 성능 저하 없이 조건에 맞는 값을 추출한다.

### 언제 함수를 인라인으로 선언할지 결정

- 모든 함수에 `inline`을 붙이면 오히려 코드 크기만 커진다.
- 특히 **람다를 인자로 받는 함수**에서 성능 향상이 두드러진다.
- 그러나 단순한 연산 함수는 JVM이 자체적으로 최적화를 진행하므로 인라인이 필요하지 않다.

**인라인이 적합한 경우**

1. **람다를 반복적으로 호출**하는 경우
2. **고차 함수**에서 성능 최적화가 필요한 경우
3. **익명 클래스 생성**을 피하고 싶은 경우

**인라인이 적합하지 않은 경우**

1. 람다를 **변수에 저장**해야 하는 경우
2. 람다를 **다른 함수로 전달**해야 하는 경우
3. 단순한 로직이지만 **호출이 많은 경우** (코드 중복만 발생)

### withLock, use, useLines로 자원 관리

- 코틀린 표준 라이브러리는 **자원 관리**를 위한 인라인 함수들을 제공합니다.
- `withLock`: Lock을 잠그고 해제하는 코드
- `use`: 파일, DB와 같은 자원 해제 처리
- `useLines`: 파일을 한 줄씩 읽고 처리

**파일 읽기와 자원 해제**

```kotlin
import java.io.BufferedReader
import java.io.FileReader

fun readFirstLineFromFile(fileName: String): String {
    BufferedReader(FileReader(fileName)).use { br ->
        return br.readLine()
    }
}
```

- `use` 함수가 인라인으로 처리되므로, 별도의 객체 생성 없이 **파일이 안전하게 닫힌다.**

---

## 10.3 람다에서 반환

### **람다에서의 `return`: 고차 함수에서의 흐름 제어**

- 일반적인 `for` 루프에서는 `return`이 메서드 전체를 종료시키는 역할을 한다.
- 그러나, 람다식 안에서의 `return`은 좀 다르게 동작한다.
- 람다식이 **인라인 함수**에 전달된 경우, `return`은 **람다를 감싸고 있는 함수 전체를 종료**시킨다.

### 람다 안의 `return`: 비지역 반환 (Non-local Return)

**일반적인 for문에서의 return**

```kotlin
data class Person(val name: String, val age: Int)

val people = listOf(Person("Alice", 29), Person("Bob", 31))

fun lookForAlice(people: List<Person>) {
    for (person in people) {
        if (person.name == "Alice") {
            println("Found!")
            return // 함수 전체가 종료됨
        }
    }
    println("Alice is not found")
}

fun main() {
    lookForAlice(people)
    // Found!
}

```

- `return`은 **함수 전체를 종료**한다.
- Alice가 발견되면 `lookForAlice` 함수 자체가 종료되며, 이후 코드는 실행되지 않는다.

**`forEach`에서의 return**

```kotlin
fun lookForAlice(people: List<Person>) {
    people.forEach {
        if (it.name == "Alice") {
            println("Found!")
            return // 여기서도 함수 전체가 종료됨
        }
    }
    println("Alice is not found")
}
```

- `forEach`는 **인라인 함수**이기 때문에, 람다 안의 `return`이 바깥 함수(lookForAlice)를 종료한다.
- 이것을 비지역 반환(Non-local Return)이라고 한다.

### 람다로부터 반환: 레이블을 사용한 `return`

- 람다식에서 **로컬 반환(Local Return)**을 하기 위해서는 **레이블(label)**이 필요하다.
- 레이블을 사용하면, **람다 블록만 빠져나오고** 함수는 계속 실행된다.

**레이블을 사용한 return**

```kotlin
fun lookForAlice(people: List<Person>) {
    people.forEach label@{
        if (it.name != "Alice") {
            return@label // label을 사용하면 forEach 블록만 빠져나옴
        }
        println("Found Alice!")
    }
}

fun main() {
    lookForAlice(people)
    // Found Alice!
}
```

- `label@`을 사용하여 **람다 블록만 종료**하도록 지정한다.
- `return@label`을 사용하면 **해당 람다만 종료**되기 때문에, `println`이 정상 실행된다.

**레이블을 람다 이름으로 사용할 수 있음**

- 람다 이름이 명시적일 경우, 레이블 대신 **람다 이름**을 사용할 수 있다.

```kotlin
fun lookForAlice(people: List<Person>) {
    people.forEach {
        if (it.name != "Alice") return@forEach // 람다 이름을 레이블로 사용
        println("Found Alice!")
    }
}
```

- `@forEach`를 사용해 **람다 블록만 종료한다.**
- 함수 전체가 종료되지 않고, 그 이후의 코드가 계속 실행된다.

### 익명 함수(Anonymous Function): 기본적으로 로컬 반환

- 람다 대신 **익명 함수**를 사용하면, 별도의 레이블 없이 **로컬 반환**이 가능하다.
- 익명 함수의 `return`은 **자기 자신만 종료**하고, 바깥 함수는 계속 실행된다.

**익명 함수로 로컬 반환 처리**

```kotlin
fun lookForAlice(people: List<Person>) {
    people.forEach(fun(person) {
        if (person.name == "Alice") {
            println("Found Alice!")
            return // 여기서는 익명 함수만 종료됨
        }
        println("${person.name} is not Alice")
    })
}

fun main() {
    lookForAlice(people) // Bob is not Alice.
}
```

- 익명 함수(`fun`)는 **자기 자신만 종료**한다.
- 람다와는 다르게, **바깥 함수**에는 영향을 주지 않는다.

---

## 요약

- 함수 타입을 사용해 함수에 대한 참조를 담는 변수나 파라미터나 반환값을 만들 수 있다.
- 고차 함수는 다른 함수를 인자로 받거나 함수를 반환한다. 함수의 파라미터 타입이나 반환 타입으로 함수 타입을 사용하면 고차 함수를 선언할 수 있다.
- 인라인 함수를 컴파일할 때 컴파일러는 그 함수의 본문과 그 함수에게 전달된 람다의 본문을 컴파일한 바이트코드를 모든 함수 호출 지점에 직접 넣어준다. 이렇게 만들어지는 바이트코드는 람다를 활용한 인라인 할수 코드를 풀어서 직접 쓴 경우와 비교할 때 아무 부가 비용이 들지 않는다.
- 고차 함수를 사용하면 컴포넌트를 이루는 각 부분의 코드를 더 잘 재사용할 수 있다. 또한 고차 함수를 활용해 강력한 제네릭 범용 라이브러리를 만들 수 있다.
- 인라인 함수에서는 람다 안에 있는 return 문이 바깥쪽 함수를 반환시키는 바로컬 return을 사용할 수 있다.
- 익명 함수는 람다식을 대신할 수 있으며 return 식을 처리하는 규칙이 일반 람다식과는 다르다. 반환 지점에 여럿 있는 코드 블록을 만들어야 한다면 람다 대신 익명 함수를 쓸 수 있다.