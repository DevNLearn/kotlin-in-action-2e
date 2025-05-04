# 널이 될 수 있는 값
- 널이 될 수 있는 타입
- 널이 될 가능성이 있는 값을 다루는 구문의 문법
- 널이 될 수 있는 타입과 널이 될 수 없는 타입의 반환
- 코틀린의 널 가능성 개념과 자바 코드 사이의 상호운영성

코틀린에서는 널이 될 수 있는 타입(`nullable type`)을 제공하여 `NullPointerException` 을 피하고 코드 가독성을 살려준다.

## NullPointerException을 피하고 값이 없는 경우 처리: 널 가능성

널 가능성(nullability)은 NullPointerException 오류를 피할 수 있게 해주는 코틀린 타입 시스템의 특성이다.

코틀린에서 타입 이름 뒤에 `?` 키워드를 붙여 타입의 변수나 프로퍼티에 `null` 참조를 저장할 수 있다

```kotlin
Type? = Type 또는 null
```

모든 타입은 기본적으로 `null` 이 아닌 타입이며 명시적으로 `?` 가 붙어야 `null`이 될 수 있다.

```kotlin
fun strLenSafe(s: String) = s.length()
// ERROR: only safe (?.) or non-null asserted (!!.) calls are allowed
// on a nullable receiver of type kotlin.String?
```

널이 될 수 없는 값을 널이 될 수 없는 타입의 변수에 대입할 수 없다.

```kotlin
fun main() {
    val x: String? = null
    var y: String = x
    // ERROR: Type mismatch:
    // inferred type is String? but String was expected
}
```

널이 될 수 있는 타입의 값을 널이 아닌 타입의 파라미터를 받는 함수에 전달 할 수 없다.

```kotlin
fun strLen(s: String) = s.length
fun main() {
    val x: String? = null
    strLen(x)
    // ERROR: Type mismatch:
    // inferred type is String? but String was expected
}
```

코틀린에서 널과 비교하고 나면 컴파일러는 그 사실을 기억하고 널이 아님이 확실한 영역에서는 해당 값을 널이 아닌 타입의 값처럼 사용할 수 있다.

```kotlin
fun strLenSafe(s: String?): Int = 
    if (s != null) s.length else 0 // null 검사를 추가하면 코드가 컴파일된다.
    
fun main() {
    val x: String? = null
    println(strLenSafe(x)) // 0
    println(strLenSafe("abc")) // 3
}
```

## 1. 안전한 호출 연산로 null 검사와 메서드 호출 합치기: ?.

`?.` 연산자는 널 검사와 메서드 호출을 한 연산으로 수행해준다.

```kotlin
str?.uppercase() // if(str != null) str.uppercase() else null 와 같다
```

프로퍼티를 읽거나 쓸 때도 안전한 호출을 사용할 수 있다.

```kotlin
class Employee(var name: String, val manager: Employee?)
fun managerName(employee: Employee): String? = employee.manager?.name
fun main() {
    val ceo = Employee("Da Boss", null)
    val developer = Employee("Bob Smith", ceo)
    println(managerName(developer)) // Da Boss
    println(managerName(ceo)) // null
}
```

객체 그래프에서 널이 될 수 있는 중간 객체가 여럿 있다면 안전한 호출을 연쇄해서 사용할 수 있다.

```kotlin
class Address(val streetAddress: String, val zipCode: Int,
              val city: String, val country: String)
class Company(val name: String, val address: Address?)
class Person(val name: String, val company: Company?)
fun person.countryName(): String {
    val country = this.company?.address?.country
    return if (country != null) country else "Unknown"
}
fun main() {
    val person = Person("Dmitry", null)
    println(person.countryName()) // Unknown
}
```

## 2. 엘비스 연산자로 null에 대한 기본값 제공: ?:

코틀린에서 `엘비스 연산자(?:)` 를 통해 널 대신 사용할 기본값을 지정할 수 있다.

```kotlin
fun greet(name: String?) {
    val recipient: String = name ?: "unnamed"
    println("Hello, $recipient!")
}
```

코틀린에서는 `return` 과 `throw` 등도 식이기 때문에 엘비스 연산자의 오른쪽에 넣을 수 있다.

> 엘비스 연산자의 왼쪽 값이 null이 되면 함수가 즉시 어떤 값을 반환하거나 예외를 던지는 패턴은 함수의 전제조건을 검사하는 경우 유용하다.
>

```kotlin
fun printShippingLable(person: Person) {
    val address = person.company?.address ?: throw IllegalArgumentException("No Address")
    with(address) {
        println(streetAddress)
        println("$zipCode $city, $country")
    }
}
```

## 3. 예외를 발생시키지 않고 안전하게 타입을 캐스트하기: as?

코틀린 타입 캐스트 연산자인 `as` 로 지정한 타입으로 바꿀 수 없으면 `ClassCastException` 이 발생한다. `as` 를 사용할 때마다 `is` 를 통해 미리 `as` 로 변환 가능한 타입인지 검사 해볼 수 있지만, `as?` 연산자는 어떤 값을 지정한 타입으로 변환하고 변환할 수 없다면 null을 반환한다.

안전한 캐스트를 사용할 때 일반적인 패턴은 캐스트를 수행한 뒤에 엘비스 연산자를 사용하는 것이다.

```kotlin
class Person(val firstName: String, val lastName: String) {
    override fun equals(o: Any?): Boolean {
        val otherPerson = o as? Person ?: return false
        
        return otherPerson.firstName == firstName && 
               otherPerson.lastName == lastName // 안전 캐스트를 하고 마년 스마트 캐스트가 된다.
    }
}
```

## 4. 널 아님 단언: !!

코틀린에서 `널 아님 단언(!!)` null 처리 지원을 쓰는 대신, 직접 컴파일러에게 어떤 값이 실제로는 null 아니라는걸 알려 줄 수 있다.

```kotlin
fun ignoreNulls(str: String?) {
    val strNotNull: String = str!!
    println(strNoNull.length)
}
```

하지만, 널 아님 단언을 사용해도 실제로 null 값이 오면 예외가 발생한다. 예외가 발생하는 시점은 함수를 사용하는 부분이 아닌 널 아님 단언 선언부에서 예외가 발생한다.

```kotlin
fun main() {
    ignoreNulls(null)
    // Exception in thread "main" kotlin.KolinNullPointerException
}
```

`!!` 은 null에 대해 사용해서 발생하는 예외의 스택 트레이스에는 어떤 파일의 몇 번째 줄인지에 대한 정보는 들어 있지만 어떤 식에서 예외가 발생했는지 정보가 없어 한 줄에 함께 쓰는 일을 피하는게 좋다.

```kotlin
person.company!!.address!!.country // 한 줄에 사용하면 좋지 않다.
```

## 5. let 함수

`let` 함수를 사용하면 널이 될 수 있는 식을 더 쉽게 다룰 수 있다. `let` 함수를 `안전한 호출 연산자` 와 함께 사용하면 원하는 식을 평가해서 null인지 검사한 다음 결과를 변수에 넣는 작업을 간단한 식을 사용해 한번에 처리할 수 있다.

```kotlin
fun sendEmailTo(email: String) { /*...*/ }
```

sendEmailTo 함수에는 널이 될 수 잇는 타입을 값으로 넘길 수 없어 인자를 넘기기 전에 주어진 값이 null 인지 검사해야한다.

```kotlin

fun main() {
    val email: String? = "foo@bar.com"
    if (email != null) sendEmailTo(email)
}
```

안전한 호출 구문을 사용해 let 함수로 자신의 수신 객체를 인자로 넘길 수 있다.

```kotlin
email?.let { sendEmailTo(it) }
```

## 6. 직접 초기화하지 않는 널이 아닌 타입: 지연 초기화 프로퍼티

객체를 일단 생성한 다음에 나중에 전용 메서드를 통해 초기화하는 프레임워크가 많다. 하지만, 코틀린에서는 일반적으로 생성자에서 모든 프로퍼티를 초기화해야 한다.

특히 프로퍼티 타입이 널이 될 수 없는 타입이라면 널이 아닌 값으로 그 프로퍼티를 초기화 해야한다. 초기화 값을 제공할 수 없으면 널이 될 수 잇는 타입을 사용할 수 밖에 없다. 하지만, 널이 될 수 있는 타입을 사용하면 모든 프로퍼티 접근에 null 검사를 넣거나 !! 연산자를 써야한다.

```kotlin
class MyService {
    fun performAction(): String = "Action Done!"
}

@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class MyTest {
    private var myService: MyService? = null // null로 초기화하기 위해 널이 될수 있는 타입
    
    @BeforeAll fun setUp() {
        myService = MyService() // 진짜 초기값을 지정
    }
    
    @Test fun testAction() {
        assertEquals("Action Done!", myService!!.performAction()) // 널 가능성으로 !!나 ?를 사용해야한다.
    }
}
```

이 코드는 프로퍼티를 여러 번 사용해야 하며 좋지 못한 코드이다. 이를 해결하기 위해 `lateinit`  변경자를 붙여 `지연 초기화(late initialize)` 할 수 있다.

```kotlin
class MyService {
    fun performAction(): String = "Action Done!"
}

@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class MyTest {
    private lateinit var myService: MyService // 초기화하지 않고 널이 아닌 프로퍼티 선언
    
    @BeforeAll fun setUp() {
        myService = MyService() // 초기값을 지정
    }
    
    @Test fun testAction() {
        assertEquals("Action Done!", myService.performAction()) // 널 검사를 수행하지 않고 프로퍼티 사용
    }
}
```

지연 초기화 프로퍼티는 널이 될 수 없는 타입이라 해도 생성자 안에서 초기화할 필요가 없다. 하지만 프로퍼티를 초기화하기 전에 프로퍼티에 접근하면 오류가 발생한다.

## 7. 안전한 호출 연산자 없이 타입 확장: 널이 될 수 있는 타입에 대한 확장

널이 될 수 있는 타입에 대한 확장 함수를 정의하면 널 값을 다루는 강력한 도구로 활용할 수 있다. 메서드를 호출하기 전에 수신 객체 역할을 하는 변수가 널이 될수 없다 보장하는 대신, 메서드 호출이 null을 수신 객체로 받고 내부에서 null을 처리하게 할 수 있다.

```kotlin
fun String?.isNullOrBlank(): Boolean = this == null || this.isBlank()
```

확장 함수에서 수신 객체가 null 될 수 있다.

> let은 this가 null인지 검사하지 않는다.
>

## 8. 타입 파라미터의 널 가능성

코틀린에서 함수나 클래스의 모든 타입 파라미터는 기본적으로 null이 될 수 있다. 널이 될 수 있는 타입을 포함하는 어떤 타입이라도 타입 파라미터를 대신할 수 있다. 따라서 타입 파라미터 `T` 를 클래스나 함수 안에서 타입 이름으로 사용하면 이름 끝에 물음표가 없더라도 `T` 가 널이 될 수 있다.

```kotlin
fun <T> printHashCode(t: T) {
    println(t?.hashCode()) // t가 널이 될 수 있어 안전한 호출을 사용해야 한다.
}

fun main() {
    printHashCode(null) // T의 타입은 Any?로 추론된다.
}
```

타입 파라미터가 널이 아님을 확실히 하려면 널이 될 수 없는 타입 상계를 지정해야한다.

```kotlin
fun <T: Any> printHashCode(t: T) {} // T는 널이 될 수 없는 타입이다.
```

## 9. 널 가능성과 자바

자바 코드에도 어노테이션으로 표신된 널 가능성 정보가 있다. 이런 정보가 코드에 있으면 코틀린도 그 정보를 활욘한다.

```kotlin
// 자바 = 코틀린
@Nullable + Type = Type?
@NotNull + Type = Type
```

이러한 널 가능성 어노테이션이 없는 경우 자바의 타입은 코틀린의 플랫폼 타입이 된다.

### 플랫폼 타입

`플랫폼 타입` 은 코틀린이 널 관련 정보를 알 수 없는 타입을 말한다. 플랫폼 타입은 널이 될 수 있는 타입으로 처리해도 되고 널이 될 수 없는 타입으로 처리해도 된다.

플랫폼 타입에 대한 수행하는 모든 연산에 대한 책임은 개발자에게 있다. 컴파일러는 모든 연산을 허용한다.

코틀린은 보통 널이 될 수 없는 타입의 값에 대해 널 안정성을 검사하는 연산을 수행하면 경고를 표시하지만 플랫폼 타입의 값에 대해 널 안전성 검사를 중복 수행해도 경고를 표시하지 않는다.

```kotlin
// 자바 코드
pubilc class Person {
    private final String name;
    
    public Person(String name) {
        this.name = name;
    }
    
    public String getName() {
        return name;
    }
}
```

널 검사 없이 자바 클래스를 사용할 수 있지만, `NullPointerException` 이 발생할 수도 있다.

```kotlin
// null 검사 없이 자바 클래스 접근하기
fun yellAt(person: Person) {
    println(person.name.uppercase() + "!!!")
}

fun main() {
    yellAt(Person(null)) // java.lang.NullPointerException
}
```

```kotlin
// null 검사를 통해 자바 클래스 접근
fun yellAtSafe(person: Person) {
    println((person.name ?: "Anyone").uppercase() + "!!!")
}

fun main() {
    yellAtSafe(Person(null)) // Anyone!!!
}
```

> 모든 자바 타입을 널이 될 수 있는 타입으로 두지 않고 플랫폼 타입을 도입한 이유는 널 가능성을 판단하지 못하므로 널이 될 수 없는 값에 대해 모두 불필요한 null 검사가 수행되어 비용이 훨씬 커지기 때문이다.
>

### 상속

코틀린에서 자바 메서드를 오버라이드할 때 메서드의 파라미터와 반환 타입을 널이 될 수 있는 타입으로 선언할지 널이 될 수 없는 타입으로 선언할지 결정해야한다.

```kotlin
// 자바 코드
interface StringProcessor {
    void process(String value);
}
```

```kotlin
// 두가지 구현 모두 가능
class StringPrinter : StringProcessor {
    override fun process(value: String) {
        println(value)
    }
}

class NullableStringPrinter : StringProcessor {
    override fun process(value: String?) {
        if (value != null) {
            println(value)
        }
    }
}
```