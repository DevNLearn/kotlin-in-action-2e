# 클래스, 객체, 인터페이스
## 1. 인터페이스

코틀린 인터페이스 안에는 추상 메서드와 구현이 있는 메서드도 정의 할 수 있다. 다만 인터페이스에는 아무런 상태도 들어갈 수 없다.

```kotlin
// 인터페이스 선언과 구현
interface Clickable {
    fun click()
    fun showOff() = println("I;m clickable!")
}

class Button: Clickable {
    override fun click() = println("I was clicked")
}
```

코틀린에서 상속(inheritance)이나 구성(composition)에서 모두 클래스 이름 뒤에 콜론을 붙인다.

클래스는 인터페이스를 개수 제한 없이 구현 가능하지만 확장은 하나만 가능하다.

이름과 시그니처가 같은 멤버 메서드에 대해 둘이상의 디폴트 구현이 있는 인터페이스를 구현하는 경우, 명시적으로 새로운 구현을 제공해야 한다.

```kotlin
interface Focusable {
    fun setFocus(b: Boolean) = println("... focus")
    fun showOff() = println("I'm focusable!")
}

class Button: Clickable, Focusable {
    override fun click() = println("I was clicked")
    override fun showOff() {
        super<Clickable>.showOff()
        super<Focusable>.showOff()
    }
}
```

## 2. 제어자

코틀린에서는 `상속/오버라이딩 제어자`와 `가시성 제어자` 를 제공한다.

| 구분 | 키워드 | 설명 |
| --- | --- | --- |
| 상속/오버라이딩 제어자 | open, final, abstract | 외부 접근 가능한지 제어 |
| 가시성 제어자 | public, private, protected, internal | 클래스/멤버를 확장 가능한지 제어 |

### 상속/오버라이딩 관련 제어자

코틀린에서는 `open`, `final`, `abstract` 세 가지  `상속/오버라이딩 관련 제어자` 를 제공한다. 코틀린에서 모든 클래스와 메서드는 기본적으로 `final` 이다.

```kotlin
open class RichButton: Clickable { // 상속 가능
    fun disable() {} // final 함수로, 오버라이드 불가능
    open fun animate() {} // 오버라이드 가능
    override fun clieck() {} // 하위 클래스에서도 오버라이드 가능
}
```

기반 클래스나 인터페이스의 멤버를 오버라이드한 경우 기본적으로 `open`으로 간주된다. 오버라이드를 제한하려면 `final` 선언이 필요하다.

```kotlin
open class RichButton: Clickable {
    final override fun click() {}
}
```

> 기본 상태를 `final` 로 함으로써 얻을 수 있는 이점
1. 스마트 캐스트가 가능하다.
2. 취약한 기반 클래스 문제 방지
>

| 변경자 | 오버라이드 여부 | 설명 |
| --- | --- | --- |
| final | X | 클래스 멤버의 기본 변경자 |
| open | O | 반드시 open을 명시해야 오버라이드 가능 |
| abstract | 반드시 오버라이드 필요 | 추상 클래스의 멤버에만 이 변경자 설정 가능하며 추상 멤버에는 구현이 있으면 안됨 |
| override | 상위 클래스나 인스턴스의 멤버를 오버라이드하는 중 | 오버라이드 멤버는 기본적으로 열려있음 |

### 가시성 변경자

가시성 변경자는 클래스 외부 접근을 제어한다.

| 변경자 | 클래스 멤버 | 최상위 선언 |
| --- | --- | --- |
| public (기본 설정) | 모든 곳에서 볼 수 있음 | 모든 곳에서 볼 수 있음 |
| internal | 같은 모듈 안에서만 볼 수 있음 | 같은 모듈 안에서만 볼 수 있음 |
| protected | 하위 클래스 안에서만 볼 수 있음 | 적용 불가능 |
| private | 같은 클래스 안에서만 볼 수 있음 | 같은 파일 안에서만 볼 수 있음 |

타입의 가시성은 선언 위치보다 같거나 더 넓어야 한다.

## 3. 내부 클래스와 내포된 클래스

클래스 안에 다른 클래스를 선언하면 도우미 클래스를 캡슐화하거나 코드 정의를 그 코드를 사용하는 곳 가까이에 두고 싶을 때 유용하다. 코틀린은 기본적으로 내포 클래스로 제공한다.

### 내포 클래스

내포 클래스(nested class)는 명시적으로 요청하지 않는 한 바깥쪽 클래스 인스턴스에 대한 접근 권한이 없다.

```kotlin
interface State: Serializable
interface View {
    fun getCurrentState(): State
    fun restoreState(state: State) {}
}

class Button: View {
    override fun getCurrentState(): State = ButtonState()
    override fun restoreState(state: State) {}
    class ButtonState: State {}
}
```

클래스 계층을 만들되 그 계층에 속한 클래스의 수를 제한하고 싶은 경우 내포 클래스를 쓰면 편리하다.

### 내부 클래스

내부 클래스로 변경해서 바깥쪽 클래스에 대한 참조를 포함하게 만들고 싶다면 `inner` 변경자를 붙여야한다. 내부 클래스에서 바깥쪽 클래스의 참조에 접근하려면 `this@클래스명` 의 형식으로 사용한다.

```kotlin
class Outer {
    inner class Inner {
        fun getOuterReference(): Outer = this@Outer
    }
}
```

## 4. 봉인된 클래스

코틀린에서는 `봉인된 클래스` 를 통해서 확장이 제한된 클래스 계층을 정의할 수 있다.

```kotlin
sealed class Expr
class Num(val value: Int): Expr()
class Sum(val left: Expr, val right: Expr): Expr()

fun eval(e: Expr): Int = 
    when (e) {
        is Expr.Num -> e.value
        is Expr.Sum -> eval(e.right) + eval(e.left)
    }
```

`when` 식에서 `sealed` 클래스의 모든 하위 클래스를 처리한다면 디폴트 분기(else 분기)가 필요없다. 컴파일러가 모든 분기를 처리하는지 확인해주기 때문이다. `sealed` 변경자는 클래스가 추상 클래스임을 명시하며 `abstract`를 붙일 필요가 없다.

## 5. 생성자

코틀린은 `주 생성자` 와 `부 생성자` 를 구분하고 `초기화 블록` 을 통해 초기화 로직을 추가할 수 있다.

### 클래스 초기화: 주 생성자와 초기화 블록

- 주 생성자 : 클래스 이름 뒤에 오는 괄호로 둘러싸인 코드

주 생성자는 생성자 파라미터를 지정하고 그 생성자 파라미터에 의해 초기화되는 프로퍼티를 정의하는 2가지 목적으로 쓰인다.

```kotlin
class User(val nickname: String) 

// 명시적 선언 시
class User constructor(_nickname: String) { // 파라미터가 하나만 있는 주 생성자
    val nickname: String
    
    init { // 초기화 블록
        nickname = _nickname
    }
}
```

생성자에 기본값을 사용할 수 있고, 생성자 인자에 이름을 지정할 수도 있다.

```kotlin
class User(val nickname: String, val isSubscribed: Boolean = true)

fun main(){
    val dave = User(nickname = "Dave", isSubscribed = true)
}
```

클래스 외부에서 인스턴스화하지 못하게 막고 싶다면 생성자를 `private`로 만들수 있다.

```kotlin
class Secretive private constructor(private val agentName: String){}
```

### 부 생성자: 상위 클래스를 다른 방식으로 초기화

주 생성자를 선언하지 않고 부 생성자만 여러가지 선언할 수 있다. 부 생성자는 `constructor` 키워드로 시작한다.

```kotlin
open class Downloader {
    constructor(url: String?){}
    constructor(uri: URI?){}
}
```

상위 클래스 생성자에 위임할 수도 있다.

```kotlin
class MyDownloader : Downloader {
    constructor(url: String?) : super(url) {} // 상위 클래스의 생성자를 호출
    constructor(uri: URI?): super(uri) {} // 상위 클래스의 생성자를 호출
}
```

같은 클래스의 다른 생성자에 위임할 수도 있다.

```kotlin
class MyDownloader : Downloader {
    constructor(url: String?) : this(URI(url)) {}
    constructor(uri: URI?): super(uri) {} 
}
```

### 인터페이스에 선언된 프로퍼티 구현 방법

```kotlin
interface User {
    val nickname: String
}

class PrivateUser(override val nickname: String) : User // 주 생성자에 있는 프로퍼티
class SubscribingUser(val email: String): User {
    override val nickname: String
        get() = email.substringBefore('@') // 커스텀 게터
}
class SocialUser(val accountId: Int) : User {
    override val nickname = getNameFromSocialNetwork(accountId) // 프로퍼티 초기화 식
}
fun getNameFromSocialNetwork(accountId: Int) = "kodee$accountId"
```

### 접근자의 가시성 변경

`get` 이나 `set` 앞에 가시성 변경자를 추가해서 접근자의 가시성을 변경할 수 있다.

```kotlin
class LengthCounter {
    var counter: Int = 0
        private set
}
```

## 6. 데이터 클래스와 클래스 위임

데이터 클래스는 `equals`, `hashCode`, `toString` 을 기본적으로 제공한다. 데이터 클래스 앞에 `data` 키워드를 선언한다.

> 코틀린에서 == 연산자는 참조 동일성을 검사하지 않고 객체의 동등성을 검사한다.
>

```kotlin
data class Customer(val name: String, val postalCode: Int)
```

또한, 몇가지 유용한 메서드를 제공한다.

### Copy() 메서드

데이터 클래스의 모든 프로퍼티를 읽기 전용으로 만들어 데이터 클래스를 `불변 클래스`로 만들도록 권장한다.

`copy` 메서드를 통해 일부 프로퍼티 값을 바꾸거나 복사본을 만들 수 있다

```kotlin
fun main() {
    val lee = Customer("하이", 4122)
    lee.copy(postalCode = 4000)
}
```

### 클래스 위임: by 키워드

코틀린에서 기본적으로 클래스를 final로 취급하고 상위 클래스의 소스코드를 변경할 때는 open 변경자를 보고 해당 클래스를 다른 클래스가 상속할 수 있을지 예상 할 수 있다.

종종 상속을 허용하지 않는 클래스에게 새로운 동작을 추가해야 할 때 일반적인 방법이 `데코레이터 패턴`이다. 데코레이터 패턴의 경우 준비 코드가 많이 필요하지만, 코틀린에서는 클래스 위임을 일급 시민 기능으로 지원한다.

```kotlin
class CountingSet<T>(
    private val innerSet: MutableCollection<T> = hashSetOf<T>()
): MutableCollection<T> by innerSet { //MutableCollection의 구현을 innserSet에게 위임

    var ojbectsAdded = 0
    
    override fun add(element: T): Boolean {
        objectsAdded++
        return innserSet.add(element)
    }
    
    ovveride fun addAll(elements: Collection<T>): Boolean {
        objectsAdded += elements.size
        return innerSet.addAll(elements)
    }
}
```

## 7. Object 키워드

코틀린에서 `object` 키워드를 통해 클래스 선언과 인스턴스 생성을 한번에 할 수 있다. `object` 키워드를 사용하는 상황으로 객체 선언, 동반 객체, 객체 식 등이 있다

### 객체 선언

객체 선언은 클래스를 정의하고 그 클래스의 인스턴스를 만들어 변수에 저장하는 작업을 처리한다. 객체 선언에서는 생성자를 쓸 수 없다.

인스턴스가 하나만 필요한 클래스를 만들기 위해 싱글턴 패턴을 이용하여 구현한다. 코틀린은 객체 선언 기능을 통해 싱글턴을 기본 제공한다.

```kotlin
object Payroll {
    val allEmployees = arrayListOf<PerSon>()
}

Payroll.allEmployees.add(Person()) // 메서드나 프로퍼티에 접근 가능
```

클래스나 인스턴스를 상속 할 수도 있다.

```kotlin
object CaseInsensitiveFileCompartor: Comparator<File> {
    override fun compare(file1: File, file2: File): Int {
        return file.path.compareTo(file2.path, ignoreCase = true)
    }
}
```

클래스 안에서 객체 선언을 할 수 있다.

```kotlin
data class Person(val name: String) {
    object NameComparator: Comparator<Person> {
        override fun compare(p1: Person, p2: Person): Int = 
            p1.name.compareTo(p2.name)
    }
}
```

### 동반 객체

코틀린 클래스 안에는 정적인(static) 멤버가 없다. 그 대신 코틀린에서는 패키지 수준의 최상위 함수와 객체 선언을 활용한다. `companion`  키워드를 사용한다

```kotlin
class MyClass {
    companion object {
        fun callMe() {
           ...
        }
    }
}

fun main() {
    MyClass.callMe()
}
```

클래스의 인스턴스는 동반 객체의 멤버에 접근할 수 없다는 점에서 자바의 정적 멤버와 다르다.

```kotlin
fun main() {
   val myObject = MyClass()
   myObject.callMe() // Error
}
```

동반 객체는 팩토리 패턴을 구현하기 유용하다.

```kotlin
class User private constructor(val nickname: String) {
    companion object {
        fun newSubscribingUser(email: String) = 
            User(email.substringBefore('@'))
        fun newSocialUser(accountId: Int) = 
            User(getNameFromSocialNetwork(accountId))
    }
}
```

동반 객체를 일반 객체처럼 사용할 수도 있다. 다른 객체 선언처럼 이름을 붙이거나, 인터페이스를 상속하거나, 확장 함수와 프로퍼티를 정의할 수 있다.

```kotlin
class Person(val name: String) {
    companion object Loader {
      fun fromJson(jsonText: String): Person = /*..*/
    }
}

fun main() {
    // 두가지 모두 호출 가능
    val person = Person.Loader.fromJson("""{"name" : "test"}""")
    val person = Person.fromJson("""{"name" : "test2"}""")
}
```

동반 객체에서 인터페이스 구현하기

```kotlin
interface JSONFactory<T> {
    fun fromJSON(jsonText: String): T
}

class Person(val name: String) {
    companion object : JSONFactory<Person> {
        override fun fromJSON(jsonText: String): Person = /*..*/
    }
}
```

동반 객체에 대한 확장 함수 정의

```kotlin
class Person(val firstName: String, val lastName: String) {
    companion object {} //비어 있는 동반 객체 선언
}

fun Pserson.Companion.fromJSON(json: String): Person { //확장 함수 선언
    /*..*/
}
```

### 객체 식

익명 객체를 정의할 때도 `object` 키워드를 쓴다.

```kotlin
interface MouseListener {
    fun onEnter()
    fun onClick()
}
val listener = ojbect : MouseListener {
    override fun onEnter() {...}
    override fun onClieck() {...}
}
```

## 8. 인라인 클래스

인라인 클래스로 사용하려면 앞에 `value` 키워드를 사용하고 `@JvmInline` 어노테이션을 붙여야한다.

```kotlin
interface PrettyPrintable {
    fun prettyPrint()
}

@JvmInline
value class UsdCent(val amount: Int): PrettyPrintable {
    val salesTax get() = amount * 0.06
    override fun prettyPrint() = println("$amount")
}
```