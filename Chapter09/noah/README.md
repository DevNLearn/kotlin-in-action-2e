# 연산자 오버로딩과 다른 관례
- 연산자 오버로딩
- 관례: 여러 연산을 지원하기 위한 특별한 이름이 붙은 메서드
- 위임 프로퍼티

## 1. 산술 연산자를 오버로드해서 임의의 클래스에 대한 연산을 더 편리하게 만들기

### plus, times, divide 등: 이항 산술 연산 오버로딩

`operator` 키워드를 붙임으로써 어떤 함수가 관례를 따르는 함수인지 명확히 할 수 있고, 실수로 관례에서 사용하는 함수 이름을 사용하는 경우 막아준다.

```kotlin
data class Point(val x: Int, val y: Int) {
    operator fun puls(other: Point): Point {
        return Point(x + other.x, y + other.y)
    }
}
```

연산자를 멤버 함수로 만드는 대신 확장 함수로 정의할 수도 있다.

```kotlin
operator fun Point.plus(other: Point): Point {
    return Point(x + other.x, y + other.y)
}
```

연산자를 정의할 때 두 피연산자의 타입이 같을 필요는 없다.

```kotlin
operator fun Point.times(scale: Double): Point {
    return Point(x * scale).toInt(), (y * scale).toInt())
}

fun main() {
    val p = Point(10, 20)
    println(p * 1.5) //Point(x=15, y=30)
}
```

반환 타입이 두 피연산자의 타입과 같을 필요도 없다.

```kotlin
operator fun Char.times(count: Int): String {
    return toString().repeat(count)
}

fun main() {
    println('a' * 3) //aaa
}
```

### 오버로딩 가능한 이항 산술 연산자

| 식 | 함수 이름 |
| --- | --- |
| a * b | times |
| a / b | div |
| a % b | mod |
| a + b | plus |
| a - b  | minus |

### 연산을 적용한 다음에 그 결과를 바로 대입: 복합 대입 연산자 오버로딩

코틀린은 `+=` , `-=` 등의 복합 대인 연산자도 지원한다.

```kotlin
fun main() {
    var point = Point(1, 2)
    point += Point(3, 4)
    println(point) // Point(x=4, y=6)
}
```

기존 `plus` 의 경우 객체에 대한 참조를 새로운 객체를 참조하도록 바꾸는 반면, 기존 객체의 내부 값을 변경하고 싶은 경우 `plusAssign` 를 사용하면 된다.

반환 타입이 `Unit`인 `plusAssign` 함수를 정의하면서 operator로 표시하면 코틀린은 += 연산자에 그 함수를 사용한다.

```kotlin
data class Point(var x: Int, var y: Int) {
    operator fun plusAssign(other: Point) {
        x += other.x
        y += other.y
    }
}
```

### 피연산자가 1개뿐인 연산자: 단항 연산자 오버로딩

단항 연산자도 operator를 표시해서 사용할 수 있다.

```kotlin
operator fun Point.unaryMinus(): Point {
    return Point(-x, -y)
}

fun main() {
    val p = Point(10, 20)
    println(-p) // Point(x=-10, y=-20)
}
```

| 식 | 함수 이름 |
| --- | --- |
| +a | unaryPlus |
| -a | unaryMinus |
| !a | not |
| ++a, a++ | inc |
| —a,a— | dec |

## 2. 비교 연산자를 오버로딩해서 객체를 사이의 관계를 쉽게 검사

코틀린에서는 산술 연산자와 마찬가지로 기본 타입 값뿐 아니라 모든 객체에 비교 연산을 수행할 수 있다. 코틀린에서는 equals나 compareTo 호출해야되는 자바와 달리 `==` 비교 연산자를 직접 사용할 수 있어 간결하며 이해하기 쉽다.

### 동등성 연산자: equals

`==`, `!=` 연산자 호출은 `equals` 메서드 호출로 컴파일된다.

```kotlin
a == b -> a?.equals(b) ?: (b == null)
```

`equals` 를 구현한다면 다음과 비슷한 코드가 된다. (hashCode 구현은 생략)

```kotlin
class Point(val x: Int, val y: Int) {
    override fun equals(obj: Any?): Boolean {
        if (obj == this) return true
        if (obj !is Point) return false
        return obj.x == x && obj.y == y
    }
}

fun main() {
    println(Point(10, 20) == Point(10, 20)) // true
    println(Point(10, 20) != Point(5, 5)) // true
    println(null == Point(1, 2)) // false
}
```

`equals` 함수에는 `override`가 붙어있다. 다른 연산자 오버로딩 관례와 달리 `equals` 는 `Any` 에 정의된 메서드이므로 `override`가 필요하다.

Any의 equals에는 operator가 붙어있지만, 그 메서드를 오버라이드하는 메서드 앞에는 operator 변경자를 붙이지 않아도 자동으로 상위 클래스의 operator 지정이 적용된다.

또한, Any에서 상속받은 equals가 확장 함수보다 우선순위가 높기 때문에 equals 를 확장함수로 정의할 수 없다.

### 순서 연산자: compareTo (<, >, <=, >=)

코틀린도 자바와 같은 `Comparable` 인터페이스를 지원한다. `Comparable` 인터페이스 안에 있는 `compareTo` 메서드를 호출하는 관례를 제공하고, 비교 연산자를 사용하는 코드를 `compareTo` 호출로 컴파일한다. `compareTo` 가 반환하는 값은 `Int`다.

```kotlin
class Person(
    val firstName: String, val lastName: String
) : Comparable<Person> {
    override fun compareTo(other: Person): Int {
        return compareValuesBy(this, other, Person::lastName, Person::firstName)
    }
}

fun main() {
    val p1 = Person("Alice", "Smith")
    val p2 = Person("Bob", "Johnson")
    println(p1 < p2)
}
```

`Comparable` 의 `compareTo` 에도 `operator` 변경자가 붙어있으므로 하위 클래스에서 오버라이드할 때 함수 앞에 `operator`를 붙일 필요가 없다.

코틀린 표준 라이브러리의 `compareValuesBy` 함수를 사용해 `compareTo`를  쉽고 간결하게 정의할 수 있다. `compareValuesBy` 는 두 객체와 여러 비교 함수를 인자로 받는다. 첫 번째 비교 함수에 두 객체를 넘겨 두 객체가 같지 않다는 결과(0이 아닌 값)가 나오면 그 결과값을 즉시 반환하고, 두 객체가 같다는 결과(0)가 나오면 두 번째 비교 함수를 통해 두 객체를 비교한다.

`compareValluesBy` 는 두 객체의 대소를 알려주는 0이 아닌 값이 처음 나올 때까지 인자로 받은 함수를 차례로 호출해 두 값을 비교하고, 모든 함수가 0을 반환하면 0을 반환한다.

## 3. 컬렉션과 범위에 대해 쓸 수 있는 관례

### 인덱스로 원소 접근: get과 set

```kotlin
operator fun Point.get(index: Int): Int {
    return when(index) {
        0 -> x,
        1 -> y
        else -> throw IndexOutOfBoundsException("Invalid coordinate $index")
    }
}

fun main() {
    val p = Point(10, 20)
    println(p[1]) // 20
}
```

```kotlin
data clsass MutablePoint(var x: Int, var y: Int)

operator fun MutablePoint.set(index: Int, value: Int) {
    when(index) {
        0 -> x = value
        1 -> y = value
        else -> throw IndexOutOfBoundsException("Invalid coordinate $index")
    }
}

fun main() {
    val p = MutablePoint(10, 20)
    p[1] = 42
    println(p) // MutablePoint(x=10, y=42)
}
```

### 어떤 객체가 컬렉션에 들어있는지 검사: in 관례

`in` 에 대응하는 함수는 `contain` 이다.

```kotlin
data class Rectangle(val upperLeft: Point, val lowerRight: Point) {
    operator fun Rectangle.conatins(p: Point): Boolean {
        return p.x in upperLeft.x..<lowerRight.x && p.y in upperLeft.y..<lowerRight.y
    }
}

fun main() {
    val rect = Rectangle(Point(10, 20), Point(50, 50))
    println(Point(20, 30) in rect) // true
    println(Point(5, 5) in rect) // false
}
```

### 객체로부터 범위 만들기: rangeTo와 rangeUntil 관례

`..` 연산자는 `rangeTo` 함수 호출을 간략하게 표현하는 방법이다.

```kotlin
start..end -> start.rangeTo(end)
```

`rangeTo` 함수는 범위를 반환한다. 하지만, 어떤 클래스가 `Comparable`  인터페이스를 구현하면 `rangeTo` 를 정의할 필요가 없다. 코틀린 표준라이브러리에는 모든 `Comparable` 객체에 대해 적용 가능한 `rangeTo` 함수가 들어있다.

```kotlin
import java.time.LocalDate

fun main() {
    val now = LocalDat.now()
    val vacation = now..now.plusDays(10)
    println(now.plusWeeks(1) in vacation) // true
}
```

0..n.forEach {} 와 같은 식은 컴파일할 수 없다. 범위 연산자는 우선순위가 낮아 범위의 메서드를 호출하려면 범위를 괄호로 둘러싸야한다.

```kotlin
fun main() {
    val n = 9
    (0..n).forEach { print(it) } // 0123456789
}
```

`rangeTo` 연산자와 비슷하게 `rangeUntil` 연산자는 열린 범위를 만든다.

```kotlin
fun main() {
    (0..<9).forEach { print(it) } // 012345678
}
```

### 자신의 타입에 대해 루프 수행: iterator 관례

반복문은 `list.iterator()` 를 호출해서 이터레이터를 얻은 다음 `hasNext` 와 `next` 호출을 반복하는 식으로 변환된다. 코틀린에서 iterator 메서드를 확장 함수로 정의할수 있어 일반 자바 문자열에 대한 for 루프가 가능하다. 코틀린 표준 라이브러리는 String의 상위 클래스인 `CharSequence` 에 대한 iterator 확장 함수를 제공한다.

```kotlin
operator fun CharSequence.iterator(): CharIterator // 이 라비으러리 함수는 문자열을 이터레이션할 수 있게 해준다.

fun main() {
    for (c in "abc") { }
}
```

```kotlin
import java.time.LocalDate

operator fun ClosedRange<LocalDate>.iterator(): Iterator<LocalDate> =
    object: Iterator<LocalDate> {
        val current = start
        override fun hasNext() = current <= endInclusive
        override fun next(): LocalDate {
            val thisDate = current
            current = current.plushDays(1)
            return thisDate
        }
}

fun main() {
    val newYear = LocalDate.ofYearDay(2042, 1)
    val daysOff = newYear.minusDays(1)..newYear
    for (dayOff in daysOff) { println(dayOff) }
    // 2041-12-31
    // 2042-01-01
}
```

## 4. Comzonent 함수를 사용해 구조 분해 선언 제공

내부에서 구조 분해 선언은 다시 관례를 사용한다. 구조 분해 선은의 각 변수를 초기화하고자 `componentN` 이라는 함수를 호출한다. `N` 은 구조 분해 선언에 있는 변수 위치에 따라 붙는 번호이다.

```kotlin
val (a, b) = p
val a = p.component1()
val b = p.component2()
```

데이터 클래스의 주 생성자에 들어있는 프로퍼티에 대해서는 컴파일러가 자동으로 `componentN` 함수를 만들어준다.

> componentN 함수는 무한정 사용할 수 없고 맨 앞의 다섯 원소에 대해서 제공한다.
>

데이터 타입이 아닌 클래스에서는 구현하는 방법은 아래와 같다.

```kotlin
class Point(val x, Int, val y: Int) {
    operator fun component1() = x
    operator fun component2() = y
}
```

### 구조 분해 선언과 루프

함수 본문 내의 선언뿐만 아니라 변수 선언이 들어갈 수 있는 장소라면 구조 분해 선언을 사용할 수 있다.

특히 맵의 원소에 대해 이터레이션할 때 구조 분해 선언이 유용하다.

```kotlin
fun printEntries(map: Map<String, String>) {
    for ((key, value) in map) {
        println("$key -> $value")
    }
}

fun main() {
    val ap = mapOf("Oracle" to "Java", "JetBrains" to "Kotlin")
    printEntries(map)
    // Oracle -> Java
    // JetBrains -> Kotlin
}
```

forEach 에서도 사용이 가능하다.

```kotlin
map.forEach { (key, value) ->
    println("$key -> $value")
}
```

### _ 문자를 사용해 구조 분해 값 무시

컴포넌트가 여러개 있는 객체에 대해 구조 분해 선언을 사용할 때 필요없는 변수에 대해 `_` 문자를 사용한다.

```kotlin
data class Person(
    val firstName: String,
    val lastName: String,
    val age: Int,
    val city: String
)

```

구조 분해 선언에서 뒤쪽의 구조 분해 선언을 제거할 수 있다.

```kotlin
val (firstName, lastName, age) = p
```

필요없는 변수에 대해 `_` 문자를 사용한다.

```kotlin
val (firstName, _, age) = p
```

## 5. 프로퍼티 접근자 로직 재활용: 위임 프로퍼티

위임 프로퍼티를 사용하면 값을 뒷받침하는 필드에 단순히 저장하는 것보다 더 복잡한 방식으로 작동하는 프로퍼티를 접근자 로직을 매번 재구현할 필요 없이 쉽게 구현할 수 있다.

위임은 객체가 직접 작업을 수행하지 않고 다른 도우미 객체가 그 작업을 처리하도록 맡기는 디자인 패턴이다.

패턴을 프로퍼티에 적용해서 접근자 기능을 도우미 객체가 수행하도록 위임한다.

### 위임 프로퍼티의 기본 문법과 내부 동작

위임 프로퍼티 일반적인 문법은 다음과 같다.

```kotlin
var p: Type by Delegate()
```

컴파일러는 숨겨진 도우미 프로퍼티를 만들고 그 프로퍼티를 위임 객체의 인스턴스로 초기화한다. p 프로퍼티는 바로 그 위임 객체에게 자신의 작업을 위임한다.

```kotlin
class Foo {
    private val delegate = Deletate() // 컴파일러가 생성한 도우미 프로퍼티
    
    var p: Type
        set(value: Type) = delegate.setVale(/* ... */, value)
        get() = delegate.getValue(/* ... */)
}
```

프로퍼티 위임 관례에 따라 Delegate 클래스는 다음과 같은 메서드를 제공한다.

- getValue: 게터를 구현하는 로직을 담는다. (필수)
- setValue: 세터를 구현하는 로직을 담는다. (필수)
- provideDelegate: 위임 객체를 생성하거나 제공하는 로직을 담는다. (옵션)

`provideDelegate` 함수는 최초 생성 시 검증 로직을 수행하거나 위임이 인스턴스화되는 방식을 변경할 수 있다. 함수들은 멤버로 구현할 수도, 확장 함수로 구현할 수도 있다.

```kotlin
class Delegate {
    operator fun getValue(/* ... */) {/* ... */}
    operator fun setValue(/* ... */, value: Type) {/* ... */}
    operator fun provideDelegate(/* ... */): Delegate {/* ... */}
}

class Foo {
    var p: Type by Delegate()
}

fun main() {
    val foo = Foo() // 위임 프로퍼티가 있는 타입의 객체를 생성할 때, 위임 객체에 provideDelegate가 잇으면 그 함수를 호출해서 위임 객체 생성
    val oldValue = foo.p // foo.p 프로퍼티에 접근하면 내부에서 delegate.getValue() 호출한다.
    foo.p = newValue // 프로퍼티 값을 변경하는 문장은 내부에서 delegate.setValue()를 호출한다.
}
```

### 위임 프로퍼티 사용: by lazy()를  사용한 지연 초기화

`지연 초기화`는 객체 일부분을 초기화하지 않고 실제로 값이 필요할 경우 초기화할 때 쓰이는 패턴이다. 초기화 과정에 자원을 많이 사용하거나 객체를 사용할 때마다 꼭 초기화하지 않아도 되는 프로퍼티에 대해 지연 초기화 패턴을 사용할 수 있다.

```kotlin
class Email {/*...*/}
fun loadEmails(person: Person): List<Email> {
    println("${person.name}의 이메이을 가져옴")
    return listOf(/*...*/)
}

class Person(val name: String) {
    private var _emails: List<Email>? = null
    
    val emails: List<Email>
        get() {
            if (_emails == null) {
                _emails = loadEmails(this)
            }
            return _emails!!
        }
    }
    
}

fun main() {
    val p = Person("Alice")
    p.emails // 최초로 emails를 읽을 때 단 한 번만 이메일을 가져온다.
    // Load emails for Alice
    p.emails
}
```

`_emails` 는 `emails(공개 프로퍼티)` 의 `뒷받침하는 프로퍼티(Backing Property)` 이다. emails를 호출하면 `loadEmails()` 메서드를 통해 초기화를 하며 지연초기화를 구현하고 있다.

위임 프로퍼티를 사용하면 간단하게 지연 초기화를 구현할 수 있다.

```kotlin
class Person(val name: String) {
    val emails by lazy { loadEmails(this) }
}
```

`lazy` 함수는 `getValue` 메서드가 있는 객체를 반환하며 기본적으로 스레드 안전하다. 필요하면 동기화에 사용할 락을 `lazy`  함수에 전달할 수 있다.

다중 스레드 환경에서 사용하지 않을 프로퍼티를 위해 `lazy` 함수가 동기화를 생략하게 할 수도 있다.

### 위임 프로퍼티 구현

`옵저버블(Observable)`은 프로그래밍에서 **데이터의 변화나 이벤트를 감지하고, 그에 반응하는 구조**를 만들기 위한 **패턴** 또는 **개념이다. 코틀린에서 위임프로퍼티를 통해 `옵저버블` 을 구현할 수 있다.**

우선 위임프로퍼티 없이 옵저버블을 구현하고 위임 프로퍼티를 사용하도록 리팩토링을 진행한다.

```kotlin
// 위임 프로퍼티 없이 구현
fun inerface Observer {
    fun onChage(name: String, oldValue: Any?, newValue: Any?)
}

open class Observable {
    val observers = mutableListOf<Observer>()
    fun notifyObjservers(propName: String, oldValue: Any?, newValue: Any?) {
        for (obs in observers) {
            obs.onChange(propName, oldValue, newValue)
        }
    }
}

class Person(val name: String, age: Int, salary: Int): Observable() {
    var age: Int = age
        set(newValue) {
            val oldValue = field
            field = newValue
            notifyObservers(
                "age", oldValue, newValue
            )
        }
    var salary: Int = salary
        set(newValue) {
            val oldValue = field
            field = newValue
            notifyObservers(
                "salary", oldValue, newValue
            )
        }
}

fun main() {
     val p = Person("Seb", 28, 1000)
     p.observers += Observer { propName, oldValue, newValue ->
         println(
             """
             Property $propName changed from $oldValue to $newValue!
             """.trimIndent()
         )
     }
     p.age = 29 // Property age Changed from 28 to 29
     p.salary = 1500 // Property salary Change from 1000 to 1500!
}
```

- Observable: Observer들의 리스트를 관리
- notifyObservers: 모든 Observer의 onChage 함수를 통해 프로퍼티의 이전 값과 새 값을 전달
- 나이가 급여가 바뀌면 리스너에게 전달

세터 코드를 보면 중복이 많이 보인다. 프로퍼티의 값을 저장하고 필요에 따라 통지를 보내주는 도우미 클래스를 추출한다.

```kotlin
class ObservableProperty(
    val propName: String,
    var propValue: Int,
    val observable: Observable
) {
    fun getValue(): Int = propValue
    fun setValue(newValue: Int) {
        val oldValue = propValue
        propValue = newValue
        observable.notifyObservers(propName, oldValue, newValue)
    }
}

class Person(val name: String, age: Int, salary: Int): Observable() {
    val _age = ObservableProperty("age", age, this)
    val age: Int
        get() = _age.getVallue()
        set(newValue) {
            _age.setValue(newVlaue)
        }
    val _salary: Int
        get() = _salary.getValue()
        set(newValue) {
            _salary.setValue(newValue)
        }
}
```

코드가 위임이 작동하는 방시과 비슷해졌다. 프로퍼티 값을 저장하고 그 값이 바뀌면 자동으로 변경 통지를 전달해주는 클래스를 만들었다. 하지만, 프로퍼티마다 ObservableProperty를 만들고 게터와 세터에서 ObservableProperty에 작업을 위임하는 준비 코드가 필요하다.

`위임 프로퍼티` 를 사용하면 이런 준비 코드를 제거할 수 있다.

```kotlin
import kotlin.reflect.KProperty

class ObservableProperty(var propValue: Int, val observable: Observable) {
    operator fun getValue(thisRef: Any?, prop: KProperty<*>): Int = propValue
    operator fun setValue(thisRef: Any?, prop: KProperty<*>, newValue: Int) {
        val oldValue = propValue
        propValue = newValue
        obserable.notifyObservers(prop.name, oldValue, newValue)
    }
}
```

이전 코드와 차이는 다음과 같다.

- 코틀린 관례에 사용하는 함수와 마찬가지로 getValue와 setValue 함수에도 operator 변경자가 붙는다.
- 두 함수(getValue, setValue)는 파라미터를 2개 받는다.
    - thisRef : 바로 설정하거나 읽을 프로퍼티가 들어있는 인스턴스
    - prop: 프로퍼티를 표현하는 객체
- KProperty.name을 통해 메서드가 처리할 프로퍼티 이름을 알 수 있다.
- KProperty 인자를 통해 프로퍼티 이름을 전달받으므로 주 생성자에서는 name 프로퍼티를 없앤다.

```kotlin
class Person(val name: String, age: Int, salary: Int) : Observable() {
    var age by ObservableProperty(age, this)
    var salary by ObservableProperty(saalary, this)
}
```

코틀린은 위임 객체를 감춰진 프로퍼티에 저장하고 주 객체의 프로퍼티를 읽거나 쓸 때마다 위임 객체의 getValue와 setValue를 호출해준다.

`ObservableProperty` 를 직접 구현하지 않고 표준 라이브러리를 사용할 수 있다.

```kotlin
import kotlin.properties.Delegates

class Person(val name: String, age: Int, salary: Int) : Observable() {
    private val onChange = { 
        property: KProperty<*>, oldValue: Any?, newValue: Any? ->
            notifyObservers(property.name, oldValue, newValue)
    }
    
    var age by Delegates.observable(age, onChange)
    var salary by Delegates.observable(salary, onChange)
}
```

### 위임 프로퍼티는 커스텀 접근자가 있는 감춰진 프로퍼티로 변환된다.

```kotlin
class C {
    var prop: Type by MyDelegate()
}

val c = C()
```

1. MyDelegate 클래스의 인스턴스는 감춰진 프로퍼티에 저장된다.  (이하 <delegate> 로 표현)
2. 컴파일러는 프로퍼티를 표현하기 위해 KProperty 타입의 객체를 사용한다. (이하 <property>로 표현)

```kotlin
// 컴파일러가 생성하 코드
class C {
    private val <delegate> = MyDelegate()
    var prop: Type
        get() = <delegate>.getValue(this. <property>)
        set(value: Type) = <delegate>.setValue(this, <property>, value)
}
```

1. 컴파일러는 모든 프로퍼티 접근자 안에 getValue와 setValue 호출 코드를 생성해준다.

```kotlin
val = c.prop -> val x = <delegate>.getValue(c, <property>)
c.prop = x -> <delegate>.setValue(c, <property>, x)
```

### 맵에 위임해서 동적으로 애트리뷰트 접근

자신의 프로퍼티를 동적으로 정의할 수 있는 객체 (`확장 가능한 객체`)를 만들 때 위임 프로퍼티를 활용할 수 있다.

```kotlin
class Person {
    private val _attributes = mutableMapOf<String, String>()
    
    fun setAttribute(attrName: String, value: String) {
        _attributes[attrName] = value
    }
    var name: String
        get() = _attributes["name"]!!
        set(value) {
            _attributes["name"] = value
        }
}

fun main() {
    val p = Person()
    val data = mapOf("name" to "Seb", "company" to "JetBrains")
    for ((attrName, value) in data)
        p.setAttribute(attrName, value)
    println(p.name) // Seb
    p.name = "Sebastian"
    println(p.name) // Sebastian
}
```

```kotlin
// 위임프로퍼티 적용
class Person {
    private val _attributes = mutableMapOf<String, String>()
    
    fun setAttribute(attrName: String, value: String) {
        _attributes[attrname] = value
    }
    
    var name: String by _attributes
}
```

### 실전 프레임워크가 위임 프로퍼티를 활용하는 방법

위임 프로퍼티를 사용해 데이터베이스 칼럼 접근하기

```kotlin
object Users : IdTable() {
    val name = varchar("name", length = 50).index()
    val age = integer("age")
}

class User(id: EntityID) : Entity(id) {
    var name: String by Users.name
    var age: Int by Users.age
}
```

## 요약

- 코틀린은 수학 연산을 오버로드할 수 있게 해준다. 자신만의 연산자를 정의할 수는 없지만 중위 함수를 더 표현력이 좋은 대안으로 사용할 수 있다.
- 비교 연산자를 모든 객체에 사용할 수 있다. 비교 연산자는 equals와 compareTo 메서드 호출로 변환된다.
- get, set, contains라는 함수를 정의하면서 코틀린 컬렉션과 비슷하게 클래스의 인스턴스에 대해 []와 in 연산을 사용할 수 있다.
- 관례를 따라 범위를 만들거나 컬렉션과 배열의 원소를 이터레이션할 수 있다.
- 구조 분해 선언을 통해 한 객체의 상태를 분해해서 어려 변수에 대입할 수 있다.
- 데이터 클래스가 아닌 클래스에 componentN 함수를 정의하면 구조 분해를 사용할 수 있다.
- 위임 프로퍼티를 통해 프로퍼티 값을 저장하거나 초기화하거나 읽거나 변경할 때 사용하는 로직을 재활용할 수 있다.
- 위임 프로퍼티는 프레임워크를 만들 때 아주 강력한 도구로 쓰인다.
- 표준 라이브러리 함수인 lazy를 통해 지연 초기화 프로퍼티를 쉽게 구현할 수 있다.
- Delegates.observable 함수를 사용하면 프로퍼티 변경을 관찰할 수 있는 옵저버를 쉽게 추가할 수 있다.
- 맵을 위임 객체로 사용하는 위임 프로퍼티를 통해 다양한 속성을 제공하는 객체를 유연하게 다룰 수 있다.