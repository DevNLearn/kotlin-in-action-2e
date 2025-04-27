# 7. 널이 될 수 있는 값

- 널 가능성은 NPE (NullPointerException) 오류를 피할 수 있게 돕는 코틀린 타입 시스템의 특성
- 코틀린은 널이 될 수 있는지 여부를 타입 시스템에 추가함으로써 컴파일 타임에 이를 감지해, 런타임 시점에 발생할 수 있는 예외의 가능성을 줄이는데 목적이 있다.
- 기본적인 타입은 널이 될 수 없다.
- 널이 될 수 있는 값은 `?`를 붙여서 표현한다.

```java
int strLen(String s) {
    return s.length();
}
```

```kotlin
fun strLen(s: String) = s.length
```

- 위 두 코드는 동일한 기능을 하지만, 코틀린은 `String` 타입이 널이 될 수 없다는 것을 명시적으로 표현하고 있다. 자바에서는 `String`이 널이 될 수 있는지 여부를 알 수 없다.
- 따라서, `strLen`을 사용할때 자바에서는 자신도 모르게 `null` 될 수 있는 값을 넣을 수 있다.

```kotlin
fim strLenSafe(s: String?) = ...
```

- `String?`은 널이 될 수 있는 `String` 타입을 의미한다.
- `String` 혹은 `null`이 들어올 수 있다.

```kotlin
fun strLenSafe(s: String?) = s.length
/*
Only safe (?.) or non-null asserted (!!.) calls are allowed on a nullable receiver of type String?
 */
```
- `String?` 타입의 `s`는 널이 될 수 있으므로, `String` 타입만 지원하는 `length` 함수를 호출할 수 없다.

```kotlin
fun strLenSafe(s: String?): Int =
    if (s != null) s.length else 0

fun main() {
    println(strLenSafe("abc")) // 3
    println(strLenSafe(null)) // 0
}
```

- `null`을 핸들링 해주게 되면, 컴파일러는 이를 인식해 `null`이 아닌 타입의 값처럼 사용할 수 있다.

### null 검사와 메서드 호출 합치기: `?.`

- 위에서 널을 검사하기 위해 `if` 문을 사용했지만, `?.` 연산자를 사용하면 이를 간단하게 표현할 수 있다.

```kotlin
if (str != null) str.uppercase() else null
str?.uppercase()
```

- 위 두가지 식은 동일한 기능을 수행한다.
- `str`이 널이 아닐경우 `uppercase()`를 호출하고, 널일 경우 `null`을 반환한다.
- 주의할 점은 `str?.uppercase()`의 리턴 값은 `String`이 아닌, `String?` 이라는 점이다.

```kotlin
class Employee(val name: String, val manager: Employee?)

fun managerName(employee: Employee): String? = employee.manager?.name

fun main() {
    val ceo = Employee("Da Boss", null)
    val developer = Employee("Bob smith", ceo)
    println(managerName(developer)) // Da Boss
    println(managerName(ceo)) // null
}
```
- `managerName` 함수는 `Employee` 객체를 받아서, 그 객체의 `manager` 속성이 널이 아닐 경우에만 `name` 속성을 반환하기 때문에
- `ceo`의 `managerName`을 호출하는 경우에는 `ceo.manager`는 `null`이기 때문에 `null`을 리턴한다.

```kotlin
class Address(val streetAddress: String, val zipCode: Int,
    val city: String, val country: String)

class Company(val name: String, val address: Address?)
class Person(val name: String, val company: Company?)

fun Person.countryName(): String {
    val country: String? = company?.address?.country
    return if (country != null) country else "Unknown"
}

fun main() {
    val person = Person("Dmitry", null)
    println(person.countryName()) // Unknown
}
```
- `Person` 클래스의 `countryName` 확장 함수는 `company` 속성이 널이 아닐 경우에만 `address` 속성을 사용하고, 그 속성이 널이 아닐 경우에만 `country` 속성을 사용한다.
- Java의 `Optional` 에서 `map` 함수로 체이닝을 하는 것과 비슷하다.

### null 검사와 메서드 호출 합치기: `?:`

- `?:` 연산자는 `엘비스 연산자`라고 불리며, 널이 아닐 경우에는 그 값을 사용하고, 널일 경우에는 다른 값을 사용하도록 하는 연산자이다.

```kotlin
fun Person.countryName(): String {
    return company?.address?.country ?: "Unknown"
}
```

- 위 코드는 `company` 속성이 널이 아닐 경우에는 `address` 속성의 `country` 값을 사용하고, 널일 경우에는 `"Unknown"`을 사용한다.

```kotlin
class Address(val streetAddress: String, val zipCode: Int,
    val city: String, val country: String)

class Company(val name: String, val address: Address?)
class Person(val name: String, val company: Company?)

fun printShippingLabel(person: Person) {
    val address = person.company?.address
        ?: throw IllegalArgumentException("No Address")

    with (address) {
        println(streetAddress)
        println("$zipCode $city, $country")
    }
}

fun main() {
    val address = Address("Elsestr. 47", 80687, "Munich", "Germany")
    val jetbrains = Company("JetBrains", address)
    val person = Person("Dmitry", jetbrains)

    printShippingLabel(person)

    printShippingLabel(Person("Alexey", null))
}
/*
Elsestr. 47
80687 Munich, Germany
Exception in thread "main" java.lang.IllegalArgumentException: No Address
 */
```

- `printShippingLabel` 함수는 `Person` 객체를 받아서, 그 객체의 `company` 속성이 널이 아닐 경우에만 `address` 속성을 사용하고, 널일 경우에는 예외를 발생시킨다.

### 예외를 발생시키지 않고 안전하게 타입을 캐스트하기: `as?`

- 자바 타입 캐스트와 마찬가지로 대상 값을 `as`로 지정한 타입으로 바꿀 수 없는 경우에는 `ClassCastException` 예외가 발생한다.
- `as?` 연산자는 널이 아닐 경우에는 그 값을 사용하고, 널일 경우에는 `null`을 반환하는 연산자이다.

```kotlin
class Person(val firstName: String, val lastName: String) {
    override fun equals(other: Any?): Boolean {
        val otherPerson = other as? Person ?: return false
        return otherPerson.firstName == firstName &&
                otherPerson.lastName == lastName
    }

    override fun hashCode(): Int =
        firstName.hashCode() * 37 + lastName.hashCode()
}

fun main() {
    val p1 = Person("Dmitry", "Jemerov")
    val p2 = Person("Dmitry", "Jemerov")
    println(p1 == null) // false
    println(p1 == p2) // true
    println(p1.equals(42)) // false
}
```

- 이 패턴을 사용하면, 파라미터로 받은 값이 원하는 타입인지 쉽게 검사하고 캐스트 할 수 있고,
- 타입에 맞지 않는 경우 쉽게 `false`를 반환할 수 있다.

### 널이 될 수 있는 타입을 강제로 널이 아닌 타입으로 변환하기: `!!`

- `!!` 연산자는 널이 될 수 있는 타입을 강제로 널이 아닌 타입으로 변환하는 연산자이다.
- 널이 아닐 경우에는 그 값을 사용하고, 널일 경우에는 `NullPointerException` 예외를 발생시킨다.

```kotlin
fun ignoreNulls(str: String?) {
    val strNotNull: String = str!!
    println(strNotNull.length)
}

fun main() {
    ignoreNulls(null)
    // Exception in thread "main" java.lang.NullPointerException
}
```

### 널이 될 수 있는 값을 널이 아닌 값만 받는 함수에 넘기기: `let`

- `let` 함수는 널이 될 수 있는 값을 널이 아닌 값만 받는 함수에 넘길 수 있도록 해준다.

```kotlin
fun sendEmailTo(email: String) { /* ... */ }

fun main() {
    val email: String? = "foo@bar.com"
    sendEmailTo(email) // Type mismatch Error!
    if (email != null) sendEmailTo(email)
}
```

- 위 코드에서 컴파일 에러를 해결하기 위해선, `if (email != null) sendEmailTo(email)` 처럼 널 검사를 해줘야 한다.
- 하지만, `let` 함수를 사용하면 이를 간단하게 표현할 수 있다.

```kotlin
fun main() {
    val email: String? = "foo@bar.com"
    email?.let { sendEmailTo(it) } // Type mismatch Error!
}
```

### 직접 초기화하지 않고 나중에 초기화하기: `lateinit`

- `lateinit`은 `var` 프로퍼티에만 사용할 수 있으며, `null`이 아닌 타입을 지정할 수 있다.
- `lateinit` 프로퍼티는 초기화 되기 전까지는 사용할 수 없으며, 초기화 되지 않은 상태에서 사용하려고 하면 `UninitializedPropertyAccessException` 예외가 발생한다.

```kotlin
class MyService {
    fun performAction(): String = "Action Done!"
}

// 개선전
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class MyTest {
    private var myService: MyService? = null
    
    @BeforeAll
    fun setUp() {
        myService = MyService()
    }
    
    @Test
    fun testAction() {
        assertEquals("Action Done!", myService!!.performAction())
    }
}

// 개선후
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class MyTest {
    private lateinit var myService: MyService

    @BeforeAll
    fun setUp() {
        myService = MyService()
    }

    @Test
    fun testAction() {
        assertEquals("Action Done!", myService.performAction())
    }
}
```

### 타입 파라미터의 널 가능성
- 제네릭 타입 파라미터는 기본적으로 널이 될 수 있는 타입이다.
- 따라서, `T`는 널이 될 수 있는 타입으로 선언된다. **(T는 Any?로 추론)**

```kotlin
fun <T> printHashCode(t: T) {
    println(t?.hashCode())
}

fun main() {
    printHashCode(null) // null
}
```

```kotlin
fun <T: Any> printHashCode(t: T) {
    println(t.hashCode())
}

fun main() {
    printHashCode("Hello") // 69609650
}
```
- 타입 파라미터가 널이 아님을 확실히 하려면, 널이 될 수 없는 타입 상계**(upper bound)**를 지정해야한다.
