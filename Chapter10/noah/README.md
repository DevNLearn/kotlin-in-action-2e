# 고차 함수: 람다를 파라미터와 반환값으로 사용
- 함수 타입
- 고차 함수와 코드를 구조화할 때 고차 함수를 사용하는 방법
- 인라인 함수
- 비로컬 return과 레이블
- 익명함수

람다를 인자로 받거나 반환하는 함수인 `고차 함수` 를 만드는 방법과 람다를 사용할 때 발생하는 성능상 부가 비용을 없애고 유연하게 제어할 수 있는 `인라임 함수` 를 설명한다.

## 다른 함수를 인자로 받거나 반환하는 함수 정의: 고차 함수

### 함수 타입은 람다의 파라미터 타입과 반환 타입을 지정한다.

코틀린의 타입 추론으로 변수 타입을 지정하지 않아도 람다를 변수에 대입할 수 있다.

```kotlin
val sum = { x: Int, y: Int -> x + y }
val sum: (Int, Int) -> Int = { x, y -> x + y } //구체적인 타입 선언도 가능
```

널이 될 수 있는 타입으로도 지정 할 수 있다.

```kotlin
var canReturnNull: (Int, Int) -> Int? = { x, y -> null }
var funOrNull: ((Int, Int) -> Int)? = null
```

### 인자로 전달 받은 함수 호출

람다를 함수 인자로 전달 할 수 있다.

```kotlin
fun twoAndThree(operation: (Int, Int) -> Int) {
    val result = operation(2, 3)
    println("Thre reuslt is $result")
}

fun main() {
    twoAndThree { a, b -> a + b } // Thre reulst is 5
    twoAndThree { a, b -> a * b } // Thre reulst is 6
}
```

filter 함수 내부 구현할 수 있다.

```kotlin
fun String.filter(predicate: (Char) -> Boolean): String {
    return buildString {
        for (char in this@filter) {
            if (predicate(char)) append(char)
        }
    }
}

fun main() {
    println("ab1c".filter { it in 'a'..'z'}) //abc
}
```

### 자바에서 코틀린 함수 타입 사용

SAM 변환을 통해 코틀린 람다를 함수형 인터페이스를 요구하는 자바 메서드에 넘길 수 있다.

```kotlin
/* 코틀린 선언 */
fun processTheAnswer(f: (Int) -> Int) {
    println(f(42))
}

/* 자바 호출 */
processTheAnswer(number -> number + 1); // 43
```

Unit을 반환하는 함수나 람다를 자바로 작성할 수 있지만, 코틀린 Unit 타입에는 값이 존재하므로 자바에서는 그 값을 명시적으로 반환해줘야 한다. 파라미터 위치에 void를 반환하는 자바 람다를 넘길 수는 없다.

```kotlin
import kotlin.collections.CollectionsKt;

public static void main(String[] args) {
    List<String> strings = new ArrayList();
    strings.add("42")
    CollectionsKt.forEach(strings, s -> {
        System.out.println(s);
        return Unit.INSTANCE; // Unit 값을 명시적으로 반환해야한다.
    })
}
```

### 함수 타입의 파라미터에 대한 기본값을 지정할 수 있고, 널이 될 수도 있다.

함수 타입의 파라미터에 대한 기본값으로 람다식을 넣을 수 있다.

```kotlin
fun <T> Collection<t>.joinToString(
    separator: String = ", ",
    prefix: String = "",
    postfix: String = "",
    transform: (T) -> String = { it.toString() }
): String {
    val result = StringBuilder(prefix)
    
    for ((index, element) in this.withIndex()) {
        if (index > 0) result.append(separator)
        result.pappend
    }
    
    result.append(postfix)
    return result.toString()
}
```

널이 될 수 있는 함수 타입을 사용할 수도 있다. 널이 될 수 있는 함수 타입으로 함수를 받으면 그 함수를 직접 호출할 수 없다. 안전한 호출과 함수를 직접 실행해주는 invoke() 함수를 사용할 수 있다.

```kotlin
fun <T> Collection<t>.joinToString(
    separator: String = ", ",
    prefix: String = "",
    postfix: String = "",
    transform: ((T) -> String)? = null
): String {
    val result = StringBuilder(prefix)
    
    for ((index, element) in this.withIndex()) {
        if (index > 0) result.append(separator)
        val str = transform?.invoke(element) // 안전한 호출을 사용해 함수를 호출
            ?: element.toString() // 엘비스 연산자를 사용해 람다를 지정하지 않은 경우 처리
        result.pappend
    }
    
    result.append(postfix)
    return result.toString()
}
```

### 함수를 함수에서 반환

함수를 반환할 수도 있다.

```kotlin
enum class Delivery { STANDARD, EXPEDITED }
class Order(val itemCount: Int)
fun getShippingCostCalculator(delivery: Deilvery): (Order) -> Double {
    if (delivery == Delivery.EXPEDITED) {
        return { order -> 6 + 2.1 * order.itemCount }
    }
    return { order -> 1.2 * order.itemCount }
}

fun main() {
    val calculator = getShippingCostCalculator(Delivery.EXPEDITED)
    println("Shipping costs ${calculator(Order(3))}")
    // Shipping costs 12.3
}
```

### 람다를 활용해 중복을 줄여 코드 재사용성 높이기

함수 타입과 람다식은 재사용하기 좋은 코드를 만들 때 쓸 수 있는 훌륭한 도구이다.

```kotlin
data class SiteVisit(
    val path: String,
    val duration: Double,
    val os: OS
)

enum class OS { WINDOWS, LINUX, MAC< IOS, ANDROID }
val log = listOf(
    SiteVisit("/", 34.0, OS.WINDOWS),
    SiteVisit("/", 22.0, OS.MAC),
    SiteVisit("/login", 12.0, OS.WINDOWS),
    SiteVisit("/signup", 8.0, OS.IOS),
    SiteVisit("/", 16.3, OS.ANDROID)
)
```

```kotlin
val averageWindowsDuration = log
    .filter { it.os == OS.WINDOWS }
    .map(SiteVisit::duration)
    .average()
fun main() {
    println(averageWindowsDuration) //23
}
```

일반 함수를 통해 중복 제거할 수 있다.

```kotlin
fun List<SiteVisit>.averageDurationFOr(os: OS) =
    filter { it.os == os }.map(SiteVisit::duration).average()

fun main() {
    println(log.averageDurationFor(OS.WINDOWS)) // 23.0
    println(log.averageDurationFor(OS.MAC)) // 22.0
}    
```

## 2. 인라인 함수를 사용해 람다의 부가 비용 없애기

람다식마다 새로운 클래스가 생기고 람다가 변수를 캡처한 경우 람다 정의가 포함된 코드를 호출하는 시점마다 새로운 객체가 생기며 부가 비용이 든다.

inline 선언을 통해 컴파일러는 함수가 쓰이는 위치에 함수 호출을 생성하는 대신에 함수를 구현하는 코드로 바꿔쳐준다.

### 인라이닝이 작동하는 방식

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
    synchronized(1) {
        // ...
    }
}
```

synchronized 함수를 inline으로 선언했으므로 함수 본문뿐 아니라 synchronized에 전달된 람다의 본문도 함께 인라이닝된다.

```kotlin
fun foo(l: Lock) {
    println("Before sync")
    synchronized(l) {
        println("Action")
    }
    printLnt("After sync")
}
```

인라인 함수를 호출하면서 람다를 넘기는 대신에 함수 타입의 변수를 넘길 수도 있다.

```kotlin
class LockOwner(val lock: Lock) {
    fun runUnderLock(body: () -> Unit) {
        synchronized(lock, body)
    }
}
```

함수 타입의 변수를 넘기는 경우 인라인 함수를 호출하는 코드 위치에서는 변수에 저장된 람다의 코드를 알 수 없어 람다 본문은 인라이닝 되지 않는다. synchronized 함수의 본문만 인라이닝된다.

```kotlin
class LockOwner(val lock: Lock) {
    fun __runUnerLock__(body: () -> Unit) {
        lock.lcok()
        try {
            body()
        } finally {
            lock.unlock()
        }
    }
}
```

- 하나의 인라인 함수를 두 곳에서 각각 다른 람다를 사용해 호출한다면 그 두 호출은 각각 따로 인라이닝된다.
- 인라인 함수의 본문 코드가 호출 지점에 복사되고 각 람다의 본문이 인라인 함수의 본문 코드에서 람다를 사용하는 위치에 복사된다.
- 함수, 프로퍼티 접근자에도 inline을 붙일 수 있다.
    - 코틀린을 실체화한 제네릭에 유용

### 인라인 함수의 제약

람다를 사용하는 모든 함수를 인라이닝할 수 없다. 함수가 인라이닝될 때 인자로 전달된 람다식 본문은 결과 코드로 직접 들어갈 수 있다. 하지만, 람다가 본문에 직접 펼쳐지기 때문에 함수가 파라미터로 전달받은 람다를 보눔에 사용하는 방식이 한정 된다.

파라미터로 받은 람다를 다른 변수에 저장하고 그 변수를 사용한다면 람다를 표현하는 객체가 어딘가는 존재해야 하기 때문에 람다를 인라이닝할 수 없다.

```kotlin
class FunctionStorage {
    var myStoredFunction: ((Int) -> Unit)? = null
    inline fun storeFunction(f: (Int) -> Unit) {
        myStoredFunction = f // 전달된 파라미터를 저장한다. 컴파일러는 모든 호출 지점에서 이 코드를
                             // 대치할 수 없어 인라인 파라미터 f를 잘못 사용한다고 오류 발생
    }
}
```

일밙거으로 인라인 함수의 본문에서 람다식을 바로 호출하거나 다른 인라인 함수의 인자로 전달하는 경우 인라이닝 할 수 있다.

```kotlin
fun <T,R> Sequence<T>.map(trasnform: (T) -> R): Sequence<R> {
    return TransformingSequence(this, transform)
}
```

map 함수는 transform 파라미터로 전달받은 함수 값을 호출하지 않는 대신, TransformingSequence라는 클래스의 생성자에게 함수 값을 넘긴다. TransformingSequence 생성자는 전달 받은 람다를 프로퍼티로 저장한다.

이런 기능을 지원하려면 map에 전달되는 transform 인자를 일반적인 함수 표현, 즉 함수 인터페이스를 구현하는 익명 클래스 인스턴스로 만들 수 밖에 없다.

둘 이상의 람다를 인자로 받는 함수에서 일부 람다만 인라이닝하고 싶을때 noinline 변경자를 붙여 인라이닝을 금지할 수 있다.

```kotlin
inline fun foo(inlined: () -> Unit, noinline notInlined: () -> Unit) {
    // ...
}
```

### 컬렉션 연산 인라이닝

코틀린 표준라이브러리의 컬렉션 함수는 대부분 람다를 인자로 받는다. 표준 라이브러리 함수를 사용하지 않고 직접 연산을 구현하는 것과 비교한다.

```kotlin
// filter 표준라이브러리 사용 o
data class Person(val name: String, val age: Int)

val people = listOf(Person("Alice", 29), Person("Bob", 31))

fun main() {
    println(people.filter { it.age < 30})
    //[Person(name=Alice, age=29)]
}
```



```kotlin
// 표준라이브러리 사용 x
fun main() {
    val result = mutableListOf<Person>()
    for(person in people) {
        if (person.age < 30) result.add(person)
    }
    println(result)
    // [Person(name=Alice, age=29]
}
```

코틀린에서 fliter 함수는 인라인 함수다. filter 함수에 전달된 람다 본문은 filter 호출한 위치에 인라이닝된다. 그 결과 앞 예제에서 filter를 써서 생긴 바이트코드와 뒤 예제에서 생긴 바이트코드는 거의 같다.

표준라이브러리를 통해 코틀린다운 연산을 안전하게 사용할 수 있고, 코틀린이 제공하는 함수 인라이닝을 믿고 성능에 신경 쓰지 않아도 된다.

### 언제 함수를 인라인으로 선언할지 결정

JVM은 코드를 실행을 분석ㅐ서 가장 이익이 되는 방향으로 호출을 인라이닝한다. 코틀린 인라인 함수는 바이트코드에서 각 함수 호출 지점을 함수 본문으로 대치하기 때문에 코드 중복이 생긴다.

람다를 인자로 받는 함수를 인라이닝하면 생기는 이익

- 인라이닝을 통해 없앨 수 있는 부가 비용이 크다
    - 함수 호출 비용
    - 람다를 표현하는 클래스와 람다 인스턴스에 해당하는 객체를 만들 필요가 없어진다.
- 현재의 JVM은 함수 호출과 람다를 인라이닝해줄 정도로 똑똑하지 못하다.
- 일반 람다에서는 사용할 없는 몇가지 기능을 사용할 수 있다. (비로컬 리턴)

하지만 inline 변경자를 함수에 붙일 때는 코드 크기에 주의를 기울여야 한다. 인라이닝하는 함수가 큰 경우 함수의 본문에 해당하는 바이트코드를 모든 호출 지점에 복사해 넣으면 바이트코드가 전체적으로 커질 수 있다.

### withLock, use, useLines로 자원 관리를 위해 인라인 람다 사용

람다로 중복을 업앨 수 있는 일반적인 패턴 중 한가지는 작업을 하기 전에 자원을 획득하고 작업을 마친 후 자원을 해제하는 자원 관리이다.

자원 관리 패턴을 만들 때 보통 try/finally 문을 사용한다. 코틀린 라이브러리에는 좀더 코틀린다운 API를 구현할 수 있다.

```kotlin
val l: Lock = ReentrantLock()
l.withLock {
}
```

`withLock` 코드는 다음과 같다.

```kotlin
fun <T> Lock.withLock(action: () -> T): T {
    lock()
    try {
        return action()
    } finally {
        unlock()
    }
}
```

`use` 함수는 닫을 수 잇는 자원(Closeable 인터페이스를 구현한 객체)에 대해 호출하는 확장 함수다.

이 함수는 람다를 호출하고 사용 후 자원이 확실히 닫히게 한다. `use` 함수는 인라인 함수이기 때문에 성능에는 영향이 없다.

```kotlin
import java.io.BufferedReader
import java.io.FileReader

fun readFirstLineFromFile(fileName: String): String {
    BufferedReader(FileReader(fileName)).use { br ->
        return br.readLine()
    }
}
```

`use` 는 Closeable과 함께 쓰이지만 `useLines`는 File과 Path 객체에 대해 정으돼 있고, 람다가 문자열 시퀀스에 접근하게 해준다.

```kotlin
import kotlin.io.path.Path
import kotlin.io.path.useLines

fun readFirstLineFromFile(fileName: String): String {
    Path(fileName).useLines {
     return it.first()
    }
}
```

코틀린에서는 use 같은 함수를 사용해 매끄럽게 처리가 가능하기 때문에 try-with-resource를 권장하지 않는다.

## 람다에서 반환: 고차 함수에서 흐름 제어

### 람다 안의 return 문: 람다를 둘러싼 함수에서 반환

아래 에제는 이름이 Alice인 경우 lookForAlice 함수에서 반환된다.

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
    println("Alice is not found") // people안에 엘리스가 없으면 출력된다.
}

fun main() {
    lookForAlice(people) // Found!
}
```

forEach도 동일하다.

```kotlin
fun lookForAlice(people: List<Person>) {
    people.forEach {
        if (it.name == "Alice") {
            println("Found!")
            return // loockForAlice 함수에 반환
        }
    }
    println("Alice is not found")
}
```

자신을 둘러싸고 있는 블록보다 더 바깥에 있는 다른 블록을 반환하게 만드는 return 문을 `비로컬 return` 이라 부른다.

### 람다로부터 반환: 레이블을 사용한 return

람다식에도 레이블을 사용해서 로컬 return을 사용할 수 있다.

```kotlin
fun lookForAlice(people: List<Person>) {
    people.forEach label@{
        if (it.name != "Alice") return@label
        print("Found Alice!")
    }
}

fun main() {
    lookForAlice(people)
}
```

람다를 인자로 받는 인라인 함수의 이름을 return 뒤에 레이블로 사용할 수 있다.

```kotlin
fun lookForAlice(people: List<Person>) {
    people.forEach {
        if (it.name != "Alice") return@forEach
        print("Found ALice!")
    }
}
```

### 익명 함수: 기본적으로 로컬 return

익명 함수는 람다식을 작성하는 다른 문법적 형태다. 따라서 익명 함수는 다른 함수에 전달할 수 있는 코드 블록을 작성하는 다른 방법이다. 하지만 람다와 익명 함수는 return 식을 쓸 수 있다는 점에서 차이가 있다.

```kotlin
fun lookForAlice(people: List<Person>) {
    people.forEach(fun (person) {
        if (person.name == "Alice") return
        println("${person.name} is not Alice")
    })
}
```

익명 함수와 일반 함수의 차이는 함수 이름을 생략하고 파라미터 타입을 컴파일러가 추론하게 할 수 있다는 점뿐이다.

```kotlin
// filter에 익명 함수 넘기기
people.filter(fun (person): Boolean {
    return person.age < 30
})
```

익명 함수도 일반 함수와 같은 반환 타입 지정 규칙을 따른다. 블록이 본문인 익명 함수는 반환 타입을 명시해야 하지만 식을 본문으로 하는 익명 함수의 반환 타입은 생략할 수 있다.

```kotlin
people.filter(fun (person) = person.age < 30)
```

## 요약

- 함수 타입을 사용해 함수에 대한 참조를 담는 변수나 파라미터나 반환값을 만들 수 있다.
- 고차 함수는 다른 함수를 인자로 받거나 함수를 반환한다.
- 함수의 파라미터 타입이나 반환 타입으로 함수 타입을 사용하면 고차 함수를 선언할 수 있다.
- 인라인 함수를 컴파일할 때 컴파일러는 그 함수의 본문과 그 함수에게 전달된 람다의 본문을 컴파일한 바이트코드를 모든 함수 호출 지점에 직접 넣어준다.
- 만들어진 바이트코드는 람다를 활용한 인라인 함수 코드를 풀어서 직접 쓴 경우와 비교할 때 아무 부가 비용이 들지 않는다.
- 고차 함수를 사용하면 컴포넌트를 이루는 각 부분의 코드를 더 재사용할 수 있다.
- 고차 함수를 활용해 강력한 제네릭 범용 라이브러리를 만들 수 있다.
- 인라인 함수에서는 람다 안에 있는 return 문이 바깥쪽 함수를 반환시키는 비로컬 return을 사용할 수 있다.
- 익명 함수는 람다식을 대신할 수 있으며 return 식을 처리하는 규칙이 일반 람다식과 다르다
- 반환 지점에 여럿 있는 코드 블록을 만들어야 한다면 람다 대신 익명 함수를 쓸 수 있다.