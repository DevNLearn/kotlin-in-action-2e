# 고차 함수: 람다를 파라미터와 반환값으로

**다루는 내용**

- 함수 타입
- 고차 함수와 코드를 구조화할 때 고차 함수를 사용하는 방법
- 인라인 함수
- 비로컬 return과 레이블
- 익명 함수

## 고차 함수

**고차 함수란?**

- 람다나 함수를 인자로 받을 수 있거나 반환하는 함수
    - 물론 함수를 인자로 받는 동시에 함수를 반환하는 함수도 고차 함수
- ex) `filter()`는 `Boolean`을 반환하는 `Predicate` 함수를 인자로 받으므로 고차 함수이다

### 고차 함수의 타입은 람다의 파라미터 타입과 반환 타입을 지정한다

**함수 타입 선언 방법**

- 람다 타입을 정의하려면 함수 파라미터의 타입을 괄호로 감싸고 그 뒤에 화살표를 추가한 다음 함수의 반환 타입을 지정하면 된다
    - ex) `val sum: (Int, Int) → Int = { x, y → x + y }`
        - 함수 타입 안에 파라미터 타입을 지정했기 때문에 람다식 안의 x, y는 타입을 생략해도 된다
    - ex) `val action: () → Unit = { println(42) }`
        - 함수 타입 선언 시에는 Unit 생략이 불가능하다
    - ex) `var canReturnNull: (Int, Int) -> Int? = {x, y -> null}`
        - 함수 타입에서도 반환 타입을 nullable하게 할 수 있다
    - ex) `var funOrNull: ((Int, Int) → Int)? = null`
        - 변수의 타입 자체는 nullable하게 할 수 있다
        - 이 때는 위와 다르게 호출하려면 null체크부터 해야한다
        - null이 아닌 경우엔 반환값이 non-null함이 보장된다

### 인자로 받은 함수 호출

```kotlin
fun twoAndThree(operation: (opA: Int, opB: Int) -> Int) {
    val result = operation(2, 3)
    println("Result: $result")
}

fun main() {
    twoAndThree { opA, opB -> opA + opB }   // Output: Result: 5
    twoAndThree { a, b -> a + b }           // Output: Result: 5
}
```

- 인자로 받은 함수를 호출하는 구문은 일반 함수를 호출하는 구문과 같다
- 함수 이름 뒤에 괄호를 붙이고 괄호 안에 원하는 인자를 콤마로 구분해 넣으면 된다
- 함수 타입에서 파라미터 이름을 지정할 수도 있다
    - 파라미터 이름은 타입 검사 시 무시돼서 꼭 일치하지 않아도 되지만 가독성을 위해 지키는 것이 좋다

**filter 간단 버전 구현해보기**

```kotlin
fun String.filter(predicate: (Char) -> Boolean): String {
    return buildString { 
        for (char in this@filter) {
            if (char in this@filter) {
                if (predicate(char)) append(char)
            }
        }
    }
}

fun main() {
    println("ab1c".filter { it in 'a'..'z' }) // Output: abc
}
```

- `filter`는 `Predicate` 함수를 인자로 받고 그 함수의 인자로 받은 `Char`가 살아 있기를 바랄 때 `true`, 죽기를 바라면 `false`를 반환하면 된다
- 구현 자체는 간단하게 문자열의 각 문자가 `Predicate`를 만족하는지 검사한 뒤 `buildString`으로 결과를 조합한 다음 반환해주면 된다

### 자바에서 코틀린 함수 타입 사용

자동 SAM 변환으로 코틀린 람다를 함수형 인터페이스를 요구하는 자바에게 넘겨 자바의 고차 함수를 사용할 수 있었는데 함수 타입도 마찬가지로 자바에서 사용 가능하다

```kotlin
// 코틀린 선언
fun processTheAnswer(f: (Int) -> Int) {
    println(42)
}

// 자바에서 사용
processTheAnswer(number -> number+1) // 43
```

- 자동으로 자바 람다가 코틀린 함수 타입으로 변환된다

**자바에서 코틀린 표준 라이브러리 확장 함수 사용하는 예제**

```kotlin
List<String> strings = new ArrayList<>();
strings.add("42")
CollectionsKt.forEach(strings, s -> {
    System.out.println(s);
    return Unit.INSTANCE;
});
```

- 코틀린에서 확장함수를 사용하는 것만큼 맛있진 않지만 사용 가능하다
- 수신 객체를(`CollectionKt`) 명시적으로 전달해야 하고 반환할 값이 없는 경우 `Unit.INSTANCE`를 명시해서 반환해줘야 한다
    - 즉, `void`를 반환하는 자바 람다를 넘길 수는 없다

**코틀린 함수 타입 내부 구조**

- 코틀린 함수 타입은 사실 실제로는 일반 인터페이스이다
    - `(Int) -> Int`와 같은 함수 타입이 내부적으로 `Function1<Int, Int>` 인터페이스를 구현한 객체로 취급된다는 것
    - `FunctionN`은 코틀린 표준 라이브러리에서 제공하는 함수 인터페이스 시리즈
- `FunctionN` 인터페이스는 함수 파라미터 개수에 따라 달라진다


    | 함수 형태 | 대응되는 인터페이스 |
    | --- | --- |
    | `() -> R` | `Function0<R>` |
    | `(P) -> R` | `Function1<P, R>` |
    | `(P1, P2) -> R` | `Function2<P1, P2, R>` |
    | ... | ... 최대 `Function22`까지 제공 |

```kotlin
// 함수 타입
val double: (Int) -> Int = { x -> x*2 }

// 내부적으론 사실 다음과 유사
val double = object : Function1<Int, Int> {
    override fun invoke(x: Int): Int = x * 2
}
```

- 즉, 코틀린 컴파일러는 작성한 람다식을 해당하는 `FunctionN` 인터페이스를 구현하는 익명 객체로 변환한다
- 인터페이스에는 `Invoke`라는 유일한 메서드가 있는데 객체를 함수처럼 호출할 수 있게 해준다
    - ex) `double(3)`이 사실 `double.invoke(3)`으로 컴파일된다

### 함수 타입의 파라미터에 기본값을 지정할 수 있다

- 다른 디폴트 파라미터 값과 마찬가지로 함수 타입에 대한 기본값 선언도 `=` 뒤에 람다를 넣어주면 된다

```kotlin
fun <T> Collection<T>.joinToString(
    separator: String = ", ",
    prefix: String = "",
    postfix: String = "",
    transform: (T) -> String = { it.toString() },    // 기본값 지정
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
    val list = listOf("Apple", "Banana")
    println(list.joinToString())    // Apple, Banana
    println(list.joinToString { it.uppercase() })   // APPLE, BANANA
    println(list.joinToString { it.lowercase() })   // apple, banana
}
```

### 널이 될 수 있는 함수 타입을 사용할 수 있다

- nullable한 함수 타입으로 함수를 받으면 그 함수를 직접 호출할 수 없음을 기억하자
    - NPE 발생 위험으로 컴파일 에러가 발생한다
    - null 여부를 명확하게 검사한 뒤에 호출하거나 세이프콜 + 엘비스 연산자 조합으로 사용한다

```kotlin
fun <T> Collection<T>.joinToString(
    separator: String = ", ",
    prefix: String = "",
    postfix: String = "",
    transform: ((T) -> String)? = null,   // nullable하지만 그 반환 타입은 non-null
): String {
    val result = StringBuilder(prefix)

    for ((index, element) in this.withIndex()) {
        if (index > 0) result.append(separator)
        val str = transform?.invoke(element)    // 세이프 콜 
            ?: element.toString()               // 엘비스 연산자로 대비
    }

    result.append(postfix)
    return result.toString()
}
```

### 함수를 함수에서 반환할 수 있다

프로그램의 상태나 다른 조건에 의해 달라질 수 있는 로직인 경우 적절함 로직의 함수를 반환하는 함수를 정의해 사용할 수 있다

- ex) 사용자가 선택한 배송 수단에 따라 배송비 계산 방법이 달라지는 경우

```kotlin
enum class Delivery { STANDARD, EXPEDITED }

class Order(val itemCount: Int)

fun getShippingCostCalculator(
    delivery: Delivery
): (Order) -> Double {  // 함수를 반환
    if (delivery == Delivery.EXPEDITED)
        return { order -> 6 + 2.1 * order.itemCount }    // 함수 반환
    else
        return { order -> 1.2 * order.itemCount }    // 함수 반환
}

fun main() {
    val calculator = getShippingCostCalculator(Delivery.EXPEDITED)
    println("배송비: ${calculator(Order(3))}") // 배송비: 12.3
}
```

- ex) GUI 연락처 관리 앱에서 사용자가 UI 입력창에 입력한 문자열로 시작하는 연락처만 표시해야 할 때
    - 연락처 목록을 필터링하는 Predicate를 만드는 함수를 만들고 이를 filter의 인자로 넘긴다
        - prefix 검사 및 연락처 검사 등

```kotlin
data class Person(
    val firstName: String,
    val lastName: String,
    val phoneNumber: String?,
)

class ContactListFilters {
    var prefix: String = ""
    var onlyWithPhoneNumber: Boolean = false

    fun getPredicate(): (Person) -> Boolean {
        val startsWithPrefix = { p: Person ->
            p.firstName.startsWith(prefix) || p.lastName.startsWith(prefix)
        }
        if (!onlyWithPhoneNumber) return startsWithPrefix
        return { startsWithPrefix(it) && it.phoneNumber != null }
    }
}

fun main() {
    val contacts = listOf(
        Person("John", "Doe", "123-4567"),
        Person("Jane", "Smith", null),
    )
    val contactListFilters = ContactListFilters()
    with(contactListFilters) {
        prefix = "J"
        onlyWithPhoneNumber = true
    }

    println(
        contacts.filter(contactListFilters.getPredicate())
    )
    // Output: [Person(firstName=John, lastName=Doe, phoneNumber=123-4567)]
}
```

### 람다를 활용해 중복을 줄여 코드 재사용성 높이기

**ex) 웹 사이트 방문 기록을 분석하는 예제**

- 방문한 사이트의 경로, 사이트에 머문 시간, 사용자의 운영체제 정보가 들어있는 클래스가 존재

    ```kotlin
    data class SiteVisit(
        val path: String,
        val duration: Double,
        val os: OS,
    )
    
    enum class OS { WINDOWS, MAC, LINUX }
    
    val log = listOf(
        SiteVisit("/", 34.0, OS.WINDOWS),
        SiteVisit("/home", 22.0, OS.MAC),
        SiteVisit("/home", 15.0, OS.WINDOWS),
    )
    ```

- os별 평균 방문 시간을 출력하고 싶다면?

    ```kotlin
    fun List<SiteVisit>.averageDurationFor(os: OS) =
        filter { it.os == os }.map { it.duration }.average()
    
    fun main() {
        val windowsAverage = log.averageDurationFor(OS.WINDOWS)
        val macAverage = log.averageDurationFor(OS.MAC)
    
        println("Windows Average: $windowsAverage") // 24.5
        println("Mac Average: $macAverage") // 22.0
    }
    ```


**고차 함수를 사용해 리팩토링**

- 요구사항이 더 복잡해질 경우 간단한 파라미터로는 처리하기 어렵다
- 이 때 함수 타입을 사용하면 필요한 조건을 파라미터로 뽑아내기 좋다

```kotlin
fun List<SiteVisit>.averageDurationFor(predicate: (SiteVisit) -> Boolean) =
    filter(predicate).map { it.duration }.average()

fun main() {
    // 복잡한 요구사항들 ..
    val averageDuration = log.averageDurationFor { it.path == "/home" }
    println("Average duration for path '/home': $averageDuration")  // Output: 18.5

    val averageDurationForWindows = log.averageDurationFor { it.os == OS.WINDOWS && it.path == "/" }
    println("Average duration for path '/' on Windows: $averageDurationForWindows") // Output: 34.0
}
```

## 인라인 함수: 람다 부가 비용 없애기

코틀린은 람다를 매우 편리하게 지원하지만 사실 매번 익명 클래스로 컴파일 되기에 성능 저하를 유발할 수 있다

- 특히 변수를 캡처하는 경우 매 호출마다 새로운 객체가 생성되어 GC 부담도 생긴다
- 코틀린은 `inline` 키워드를 통해 함수가 호출 시점에 함수 본문을 그대로 삽입(inline)해 함수 호출 비용을 제거하고 람다의 객체 생성을 방지하도록 지원한다

### **`inline` 동작 방식**

`inline`을 함수에 붙이면, 해당 함수의 본문이 호출된 곳에 직접 삽입된다.

즉, 함수를 호출하는 바이트코드 대신 함수 본문을 번역한 바이트코드로 컴파일된다

**synchronized 예제**

```kotlin
inline fun <T> synchronized(lock: Lock, action: () -> T): T {
    lock.lock()
    return try {
        action()
    } finally {
        lock.unlock()
    }
}

fun main() {
    val l = ReentrantLock()
    synchronized(l) { ... }
}
```

- Lock 클래스의 인스턴스를 요구한다는 점을 제외하면 자바의 synchronized 문과 거의 동일하다
    - 명시적인 락을 사용해 더 신뢰할 수 있다
- inline으로 선언해주었기 때문에 호출하는 코드는 모두 자바의 synchronized 문과 같아진다

위에서 inline으로 정의한 synchronized를 아래처럼 사용했을 때

```kotlin
fun foo(l: Lock) {
    println("Before critical section")
    synchronized(l) {
        println("Critical section")
    }
    println("After critical section")
}
```

실제로 컴파일된 바이트코드는 다음과 같은 형식이다

```kotlin
fun __foo__(l: Lock) {
    println("Before critical section")
    lock.lock()
    return try {
        println("Critical section")
    } finally {
        lock.unlock()
    }
    println("Outside critical section")
}
```

- synchronized 함수의 본문뿐 아니라 전달된 람다의 본문도 함께 인라이닝되는 모습..
- 람다의 본문에 의해 만들어지는 바이트코드는 그 람다를 호출하는 코드(synchronized) 정의의 일부분으로 간주되기 대문에 코틀린 컴파일러는 그 람다를 함수 인터페이스를 구현하는 익명 클래스로 감싸지 않아서 성능 저하를 막을 수 있다

**인라인 함수를 호출하면서 람다 대신 함수 타입의 변수를 넘길 경우**

```kotlin
class LockOwner(val lock: Lock) {
    fun runUnderLock(body: () -> Unit) {
        synchronized(lock, body)
    }
}
```

- 인라인 함수 synchronized를 호출하는 코드 위치에서는 전달받은 변수 body에 저장된 람다의 코드를 알 수 없기 때문에 인라이닝되지 않는다
    - 대신 synchronized 함수의 본문만 인라이닝된다

실제로 컴파일된 바이트코드는 다음과 같은 형식이다

```kotlin
class LockOwner(val lock: Lock) {
    fun __runUnderLock__(body: () -> Unit) {
		    lock.lock()
		    try {
		        body()   // 인라이닝되지 않고 그대로 들어간다
		    } finally {
		        lock.unlock()
		    }
	  }
}
```

**하나의 인라인 함수를 두 곳에서 각각 다른 람다를 사용해 호출할 경우**

- 두 호출은 각각 따로 인라인된다
- 인라인 함수의 본문 코드가 호출 지점에 복사되고 각 람다의 본문이 인라인 함수의 본문 코드에서 람다를 사용하는 위치에 복사된다

```kotlin
inline fun runTwice(action: () -> Unit) {
    action()
}

// 실제 코드
fun main() {
    runTwice { println("Hello") }
    runTwice { println("Bye") }
}

// 바이트코드
fun main() {
    // runTwice { println("Hello") } 
    println("Hello")

    // runTwice { println("Bye") }
    println("Bye")
}

```

### 인라인 함수의 제약

인라이닝에 의해 람다가 본문에 직접 펼쳐지기 때문에 함수가 파라미터로 전달받은 람다를 본문에 사용하는 방식이 한정된다

- 파라미터로 받은 람다를 다른 변수에 저장하고 나중에 그 변수를 사용할 때에는 람다를 표현하는 객체가 어딘가는 존재해야 해서 람다를 인라이닝할 수 없다

```kotlin
class FunctionStorage {
    var myStoredFunction: ((Int) -> Unit)? = null
    inline fun storeFunction(f: (Int) -> Unit) {
        // 전달된 파라미터(람다) 저장
        myStoredFunction = f  // 컴파일가 호출 시 이 코드를 대치할 수 없어서 컴파일 에러 발생
    }
}
```

- `Illegal usage of inline parameter` 인라인을 컴파일러단에서 금지시킨다

**시퀀스는 람다를 받아 모든 시퀀스 원소에 적용해 새 시퀀스를 반환하는 함수가 많다**

- ex) `Sequence.map`

```kotlin
public fun <T, R> Sequence<T>.map(transform: (T) -> R): Sequence<R> {
    return TransformingSequence(this, transform)
}

internal class TransformingSequence<T, R>
constructor(private val sequence: Sequence<T>, private val transformer: (T) -> R) : Sequence<R> {
    override fun iterator(): Iterator<R> = object : Iterator<R> {
        val iterator = sequence.iterator()
        override fun next(): R {
            return transformer(iterator.next())
        }

        override fun hasNext(): Boolean {
            return iterator.hasNext()
        }
    }

    internal fun <E> flatten(iterator: (R) -> Iterator<E>): Sequence<E> {
        return FlatteningSequence<T, R, E>(sequence, transformer, iterator)
    }
}
```

- `transform` 파라미터로 전달받은 람다를 호출하지 않는 대신 `TransformingSequence` 클래스의 생성자로 넘기면 생성자에서 전달받은 람다를 프로퍼티로 저장한다.
    - 바로 호출하지 않고 클래스로 넘기는 이유는 map()이 호출되는 시점에 아무일도 일어나지 않도록 하기 위함!
    - 실제 람다 실행은 `TransformingSequence.iterator()`가 호출되고 내부 익명 `Iterator`의 `next()`가 호출될 대 `transform()`이 실행된다 → 최종 연산 실행할 때
- 따라서 이 기능을 사용하려면 map에 전달되는 `transform` 인자를 일반적인 함수 표현, 즉 함수 인터페이스를 구현하는 익명 클래스 인스턴스로 만들 수밖에 없다
    - 이것이 시퀀스 오버헤드 발생 이유이며 작은 크기의 시퀀스에 하면 오히려 객체 생성 오버헤드로 일반 컬렉션보다 느릴 수 있는 이유

**둘 이상의 람다를 인자로 받는 함수에서 일부 람다만 인라이닝 하고 싶을 때**

- ex) 어떤 람다에 너무 많은 코드가 들어가거나 어떤 람다에 인라이닝하면 안 되는 코드가 들어가는 경우
- `noinline` 키워드를 파라미터 이름 앞에 붙여 인라이닝을 금지할 수 있다

**자바에서 인라인 함수를 호출한 경우**

- 컴파일러는 인라인 함수를 인라이닝하지 않고 일반 함수 호출로 컴파일한다

### 컬렉션 연산 인라이닝

코틀린 표준 라이브러리 컬렉션 함수는 대부분 람다를 인자로 받는데 인라인 함수로 되어 있어서 여러번 체이닝해서 사용해도 성능 저하가 없다

- 직접 함수 없이 구현한 것과 거의 동일한 코드이므로 코틀린답게 함수 인라이닝을 믿고 안전하게 사용하면 된다

**filter와 map을 연쇄해서 사용할 경우**

```kotlin
people.filter { it.age > 30 }
      .map(Person::name)
```

- filter는 람다, map은 멤버 참조를 사용하는데 두 함수 모두 인라인 함수라서 본문이 인라이닝되어 추가 객체나 클래스 생성은 없다
- 그러나 리스트를 걸러낸 결과를 저장하는 중간 리스트를 만드는 것은 어쩔 수 없다
    - 원소가 많을 때에는 asSequence로 중간 리스트 비용을 줄이는 게 좋다 (다만 이 때의 람다는 인라이닝되지 않고 다 익명 클래스로 컴파일된다)
    - Sequence의 최종 연산은 중간 시퀀스에 있는 여러 람다를 연쇄 호출하기 때문에 무조건 사용하기보단 컬렉션 크기가 클 때 사용해야 한다

**언제 함수를 인라인 해야할까?**

일반 함수 호출의 경우 JVM은 이미 바이트코드 → 기계어 번역 과정(JIT)에서 가장 효율적인 방향으로 호출을 인라이닝해준다.

- 바이트코드에서는 각 함수의 구현이 정확히 한 번만 있으면 되기 때문
- 코틀린 인라인의 경우 바이트코드에서 각 함수 호출 지점을 함수 본문으로 대치하기 때문에 코드 중복이 생긴다

그러나! 인라이닝의 이점이 훨씬 더 많다

1. 인라이닝을 통해 함수 호출 비용을 줄일 수 있을 뿐 아니라 람다를 표현하는 클래스와 람다 인스턴스에 해당하는 객체를 만들 필요도 없어져 부가 비용을 상당히 없앨 수 있다
2. 현재의 JVM은 함수 호출과 람다를 인라이닝해줄 정도로 똑똑하지 않다
3. 인라이닝 사용 시 일반 람다에서는 사용할 수 없는 몇 가지 기능을 지원한다
    - non-loacl return 등

그래도 언제나 inline 키워드 사용할 때에는 신중해야 한다

- 인라이닝하는 함수가 큰 경우 본문에 해당하는 바이트코드를 모든 호출 지점에 복사해 넣어서 전체적으로 비대해질 수 있다
    - 이런 경우 람다 인자와 무관한 코드를 별도의 비인라인 함수로 빼낼 수도 있다
    - 코틀린 표준 라이브러리가 제공하는 인라인 함수들을 보면 다 크기가 작은걸 볼 수 있다

### 자원 관리를 위해 인라인된 람다 사용하기

**자원 관리?**

- 파일, 락, 데이터베이스 트랜잭션 등 자원을 획득하고 작업을 마친 후 자원을 해제
- 보통 try 블록 시작 전에 자원을 획득하고 finally에서 해제하는 것이 일반적

**락 관리 - withLock**

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

- `Lock` 인터페이스의 확장 함수로 락을 획득한 후 람다를 받는 숙어를 별도의 함수로 분리
- `unlock()` 자원 해제를 못하는 실수를 방지할 수 있고, `inline`이라서 성능 저하도 없다
- `Mutex`도 `Lock`과 유사하게 비슷한 방식의 `withLock`을 제공

**use**

```kotlin
@InlineOnly
public inline fun <T : Closeable?, R> T.use(block: (T) -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    var exception: Throwable? = null
    try {
        return block(this)
    } catch (e: Throwable) {
        exception = e
        throw e
    } finally {
        when {
            apiVersionIsAtLeast(1, 1, 0) -> this.closeFinally(exception)
            this == null -> {}
            exception == null -> close()
            else ->
                try {
                    close()
                } catch (closeException: Throwable) {
                    // cause.addSuppressed(closeException) // ignored here
                }
        }
    }
}
```

- 닫을 수 있는 자원(`Closable` 인터페이스를 구현한 객체)에 대해 호출하는 인라인 확장 확장 함수
- 람다를 호출하고 사용 후 자원이 확실히 닫히게 보장
    - 람다가 정상적으로 끝났던 예외가 터졌던 상관없이 무조건 해제

**ex) use를 활용해 파일 자원 관리**

```kotlin
fun readFirstLineFromFile(fileName: String): String {
    BufferedReader(FileReader(fileName)).use { reader ->
        return reader.readLine()
    }
}
```

**useLines 확장 함수**

- 파일을 라인 단위로 읽으면서 자동으로 자원 해제를 보장
- `BufferedReader`로부터 한 줄씩 읽는 `Sequence<String>`을 제공하면서 자원은 자동으로 닫아준다

```kotlin
public inline fun <T> Reader.useLines(block: (Sequence<String>) -> T): T =
    buffered().use { block(it.lineSequence()) }
```

- Reader를 확장하면서 buffered()로 BufferedReader로 감싸서 use 사용
- BufferedReader의 lineSequence로 lazy하게 한 줄씩 읽어 `Sequence<String>` 반환

## 람다에서 반환

### 람다 안의 `return`문

```kotlin
fun lookForAlice(people: List<Person>) {
    people.forEach { 
        if (it.name == "Alice") {
            println("Found Alice, age ${it.age}")
            return
        }
    }
    println("Alice not found.")
}
```

- 람다 안에서 `return`을 사용하면 람다에서만 반환되는 것이 아니라 그 람다를 호출하는 함수가 실행을 끝내고 반환된다
    - 이렇게 자신을 둘러싼 블록(여기선 람다)보다 더 바깥에 있는 다른 블록을 반환하게 만드는 `return`문을 비로컬 `return`이라고 한다
    - 마치 자바의 for문 안에서나 synchronized 블록 안에서 return 사용 시 블록을 끝내지 않고 메서드를 반환시키는 것과 동일하다
- 람다 블록 안에서 바깥쪽 함수를 return으로 반환시킬 수 있으려면 람다를 인자로 받는 함수가 인라인 함수여야 한다
    - 인라이닝되지 않는 함수는 변수에 저장될 수 있고, 그런 경우 함수가 반환된 다음에 나중에 람다를 실행할 수도 있기 때문에 원래 함수를 반환시키기 늦은 시점이 되어버린다

### 레이블을 사용한 `return`

람다식에서도 로컬 return을 사용할 수 있는데 마치 for 루프의 break와 비슷하게 로컬 return은 람다의 실행을 끝내고 람다를 호출한 코드의 실행을 이어간다

- 이 때 비로컬 return과 구분하기 위해 return 뒤에 레이블을 달아줘야 한다

**방법 1) 람다에 레이블 지정해주고, return 시 명시**

```kotlin
fun lookForAlice(people: List<Person>) {
    people.forEach label@{  // 람다 레이블 지정
        if (it.name == "Alice") return@label    // 람다 레이블 명시해서 리턴 
        println("Found ${it.name}!")
    }
}
```

**방법 2) 람다를 인자로 받는 인라인 함수의 이름을 레이블로 사용**

```kotlin
fun lookForAlice(people: List<Person>) {
    people.forEach { 
        if (it.name != "Alice") return@forEach   // 인라인 함수를 레이블로 사용
        println("Found ${it.name}!")
    }
}
```

**레이블이 붙은 `this` 식**

수신 객체 지정 람다의 본문에는 this 참조를 사용해 수신 객체를 가리킬 수 있는데 이 때 수신 객체 지정 람다 앞에 레이블을 붙여주면 this 뒤에 그 레이블을 명시해 암시적인 수신 객체를 지정할 수 있다

```kotlin
fun main() {
    println(StringBuilder().apply sb@{  // 람다에 레이블 붙여주고
        listOf(1, 2, 3).apply {
            this@sb.append(this.toString())  // this@레이블 키워드로 
        }
    })
}
```

- 람다가 2중으로 적용되어 있는데 내부 람다에서 레이블을 사용해 외부 수신 객체를 지정한 것
    - 내부 `this`가 `List<Int>`고, 외부 `this`가 `StringBuilder`인데 그냥 `this` 썼으면 `List<Int>`가 수신 객체로 지정됐겠지만 레이블로 외부 `this`로 명시해준 것

### 익명함수: 기본적으로 로컬 return

**익명함수?**

- 이름이 없는 일반 함수처럼 생긴 함수로 `fun` 키워드를 사용하고 함수의 이름을 생략해 정의한다
    - 파라미터 타입도 컴파일러가 추론할 수 있어 생략한다
    - 일반함수와 마찬가지로 본문이 블록이 아닌 식일 경우 추론할 수 있기에 반환 타입도 생략할 수 있다
        - ex) `val filter = fun(person) = person.age < 30`
- 주로 고차 함수 인자로 전달할 때 사용한다

**람다 vs 익명함수 return 차이**

- 람다는 return이 기본적으로 비로컬 return이라서 람다를 둘러싼(호출한) 바깥 함수를 종료시킨다.
- 반면 익명 함수 안에서 레이블이 붙지 않은 return 식은 익명 함수 자체를 반환시킬 뿐 둘러싼 외부 함수를 반환시키지 않는다.
    - 단순히 `return`은 `fun` 키워드로 정의된 가장 안쪽 함수만 반환시킨다

```kotlin
fun lookForAlice(people: List<Person>) {
    people.forEach(
        fun(person) {
            if (person.name == "Alice") return  // fun 정의된 내부 함수만 종료
            println("${person.name} is not Alice")
        }

        // fun return 이후 forEach는 그대로 진행..
    )
}
```

**익명 함수 사용처**

- 중첩된 람다에서 return 사용 시 레이블을 많이 붙여야 할때 익명 함수로 바꾸면 레이블 없이 깔끔해진다
- 조용히 빠져나가기 좋은 도구라고 한다
    - 더 헷갈리는데요?

```kotlin
people.forEach {
    if (it.name == "Alice") return@forEach // 바깥 함수 종료 방지하려면 레이블 필요
}

people.forEach(fun(person) {
    if (person.name == "Alice") return // 안전하게 해당 forEach만 탈출
})
```