# 7장 널이 될 수 있는 값
Kotlin은 **null 안정성**을 타입 시스템에 통합하여, NullPointerException(NPE)을 컴파일 타임에 방지할 수 있도록 한다.
이 장에서는 nullable 타입을 안전하게 다루는 다양한 기법과 연산자들에 대해 배운다.

---

## 7.1 NullPointerException을 피하고 값이 없는 경우 처리: 널 가능성

- 코틀린은 null 관련 문제를 컴파일 시점으로 옮겨, 런타임 오류를 줄인다.
- 코틀린은 null을 허용하는 타입(`String?`)과 허용하지 않는 타입(`String`)을 구분한다.
  이로 인해 null 안전성을 컴파일 시점에 검사할 수 있다.
- Null 가능성은 프로그램의 안정성을 크게 높여준다.

---

## 7.2 널이 될 수 있는 타입으로 널이 될 수 있는 변수 명시

- 코틀린과 자바의 가장 중요한 차이는 코틀린 타입 시스템이 널이 될 수 있는 타입을 명시적으로 지원한다.

    ```java
    /* 자바 */
    int strLen(String s) {
    	return s.length();
    }
    ```

  위의 함수에서 null을 넘기면 NullPointerException이 발생

    ```kotlin
    fun strLen(s: String) = s.length
    
    fun main() {
    	strLen(null)
    	// ERROR: Null can not be a value of a non-null type String
    } 
    ```

  위의 코틀린 코드에서는 null 인자를 넘기면 컴파일 시 오류 발생

- 코틀린은 타입 이름 뒤에 `?`를 붙이면 **널이 될 수 있는 타입**이 된다.

    ```kotlin
    fun strLenSafe(s: String?): Int =
        if (s != null) s.length else 0
    ```

- 널이 될 수 있는 타입(`String?`)의 값은 널이 될 수 없는 타입(`String`)에 바로 대입할 수 없다.

    ```kotlin
    val x: String? = null
    var y: String = x
    // ERROR: Type mismatch:
    // inferred type is String? but String was expected
    ```

- if 검사를 통해 null 값을 다루면 코드가 컴파일 된다.

    ```kotlin
    fun strLenSafe(s: String?) :Int = 
    	if (s != null) s.length else 0
    ```


---

## 7.3 타입의 의미 자세히 살펴보기

- 자바에서는 `@Nullable`, `@NotNull` 어노테이션을 사용해 널 가능성을 표시할 수 있지만, 컴파일러가 강제하지 않기 때문에 널 관련 오류를 완벽히 막을 수 없다.
- 코틀린은 타입 시스템에 널 가능성(`?`)을 아예 통합해, 컴파일 시점에 널 안전성을 강제한다.
- 자바에서는 `Optional<T>`를 사용해 널 대신 의미를 명시적으로 표현할 수 있지만, 코틀린은 언어 차원에서 이를 지원해 별도의 래퍼 타입 없이 자연스럽게 널 처리를 할 수 있다.

---

## 7.4 안전한 호출 연산자로 null 검사와 메서드 호출 합치기: ?.

- null이 아닐 때만 메서드나 프로퍼티를 호출하는 안전한 연산자이다.

    ```kotlin
    fun printAllCaps(str: String?) {
    	val allCaps: String? = str?.uppercase()
    	println(allCaps)
    }
    
    fun main() {
    	printAllCaps("abc") // ABC
    	printAllCaps(null)  // null
    }
    ```

- 안전한 호출 연산자를 여러 개 연쇄해 사용할 수 있다.

    ```kotlin
    class Address(val streetAddress: String, val zipCode: Int,
    							val city: String, val country: String)
    							
    class Company(val name: String, val address: Address?)
    
    class Person(val name: String, val company: Company?)
    
    fun Person.countryName(): String {
    	val country = this.company?.address?.country
    	return if (country != null) country else "Unknown"
    }
    
    fun main() {
    	val person = Person("Dmity", null)
    	println(person.countryName()) // Unknown
    }
    ```

    - 중간에 어떤 값이 null이어도 안전하게 넘어간다.

---

## 7.5 앨비스 연산자로 null에 대한 기본값 제공: ?:

- 값이 null이면 기본값을 지정할 수 있다.

    ```kotlin
    val recipient: String = name ?: "unnamed"
    ```

- 함수의 전제조건을 검사하는 경우 유용하다.

---

## 7.6 예외를 발생시키지 않고 안전하게 타입을 캐스트하기: as?

- 타입을 변환하려고 할 때, 실패하면 예외를 던지는 대신 null을 반환한다.

    ```kotlin
    val x = "abc"
    val y = x as? Int
    println(y) // null
    ```

- 안전한 사용자를 사용해 equals를 구현할 수 있다.

    ```kotlin
    class Person(val firstName: String, val lastName: String) {
    	override fun equalso: Any?): Boolean {
    		val otherPerson = o as? Person ?: return false
    		
    		return otherPerson. firstName == firstName &&
    				 otherPerson. lastName == lastName
    	 }
    
    	override fun hashCode(): Int =
    					firstName.hashCode() * 37 + lastName. hashCode()
    }
    
    fun main () {
    	val p1 = Person ("Dmitry", "Jemerov" )
    	val p2 = Person ("Dmitry", "Jemerov")
    	println(p1 == null)
    	// false
    	println(p1 == p2)
    	// true
    	println(p1.equals(42))
    	// false
    }
    ```

    - as?를 사용하면 타입 캐스팅 실패시 null로 안전하게 처리 가능!

---

## 7.7 널 아님 단언: !!

- 강제로 null이 아님을 단언하지만, 실패하면 즉시 예외 발생한다.

    ```kotlin
    fun ignoreNulls(str: String?) {
    	val strNotNull: String = str !!
    	println(strNotNull.length)
    }
    
    fun main() {
    	ignoreNulls(null)
    	// Exception in thread "main" kotlin.kotlinNullPointerException
    	// . at <...>.ignoreNulls(07_NotnullAssertions.kt:2)
    }
    ```

    - `!!` 는 정말 null이 아닐 때만 사용해야 하고, 남용하면 위험하다.

---

## 7.8 let 함수

- 안전한 호출 후 널이 아닐 때만 특정 작업을 수행할 수 있다.
- `let` 은 수신 객체를 `it` 으로 전달한다.

```kotlin
fun sendEmailTo(email: String) {
    println("Sending email to $email")
}

fun main() {
    val email: String? = "yole@example.com"
    email?.let { sendEmailTo(it) } // Sending email To yole@example.com
    email = null
    email?.let { sendEmailTo(it) }
}
```

- 코틀린에는 다양한 영역 함수가 있다.

    | 함수 | 특징 | 수신 객체 이름 | 리턴 값 |
    | --- | --- | --- | --- |
    | `let` | 계산 결과를 반환하거나, 체이닝에 사용 | `it` | 마지막 표현식 |
    | `also` | 부수 효과(로깅, 디버깅 등) 발생시킬 때 사용 | `it` | 객체 자체 |
    | `apply` | 객체를 구성할 때 사용 (초기화 목적) | `this` | 객체 자체 |
    | `run` | 수신 객체에서 여러 작업 수행 후 결과 반환 | `this` | 마지막 표현식 |
    | `with` | 객체를 인자로 받아 블록 안에서 작업 수행 | `this` | 마지막 표현식 |

---

## 7.9 직접 초기화하지 않는 널이 아닌 타입: 지연 초기화 프로퍼티

- 코틀린은 생성자에서 모든 프로퍼티를 초기화해야한다.
- 코틀린은 널이 될 수 없는 타입은 반드시 초기화되어야 한다는 규칙이 있다.
- 하지만 어떤 경우에는 나중에 초기화 하고 싶을 때가 있음.

    ```kotlin
    class MyService {
        fun performAction(): String = "Action Done!"
    }
    
    @TestInstance(TestInstance.Lifecycle.PER_CLASS)
    class MyTest {
        private lateinit var myService: MyService
    		
    		@BeforeAll fun setUp() {
            myService = MyService()
        }
    
        @Test fun testAction() {
            assertEquals("Action Done!", myService.performAction())
        }
    }
    ```

- `lateinit`을 사용하면 초기화가 setUp() 같은 메서드 안에서 이루어질 수 있다.
- `lateinit`은 **var**(가변 변수)에만 쓸 수 있고, 기본 타입(Int, Boolean 등)에는 사용할 수 없다.
- 사용 전에 초기화가 되어있지 않으면 `UninitializedPropertyAccessException`이 발생할 수 있다.

---

## 7.10 안전한 호출 연산자 없이 타입 확장: 널이 될 수 있는 타입에 대한 확장

- 코틀린에서는 널이 될 수 있는 타입에 대해 **확장 함수**를 만들 수 있다.
- 이런 확장을 이용하면 매번 안전한 호출(`?.`) 없이도 깔끔하게 null 처리를 할 수 있다.

    ```kotlin
    fun String?.isNullOrBlank(): Boolean =
        this == null || this.isBlank()
        
    fun verifyUserInput(input: String?) {
        if (input.isNullOrBlank()) {
            println("Please fill in the required fields")
        }
    }
    
    fun main() {
        verifyUserInput(" ")   // "Please fill in the required fields"
        verifyUserInput(null)  // "Please fill in the required fields"
    }
    ```

    - 이런 식으로 null-safe 기능을 추가하는 확장 함수를 만들어두면 코드가 훨씬 깔끔해진다.
- 자주 쓰이는 표준 확장 함수
    - `isNullOrEmpty()`
    - `isNullOrBlank()`

---

## 7.11 타입 파라미터의 널 가능성

- **제네릭 타입**(타입 파라미터)을 사용할 때는 **널이 될 수 있는지 여부**가 명확하지 않을 수 있다.
- 기본적으로 타입 파라미터는 널이 될 수 있다.

    ```kotlin
    fun <T> printHashCode(t: T) {
        println(t?.hashCode())
    }
    
    fun main() {
        printHashCode(null)
    }
    ```

    - 여기서 `t` 는 null일 수도 있기 때문에 안전한 호출(?.)을 해야 한다.
- 타입 파라미터에 non-null제한을 걸고 싶으면 `T:Any` 처럼 상한을 설정할 수 있다.

    ```kotlin
    fun <T : Any> printHashCode(t: T) {
        println(t.hashCode())
    }
    
    fun main() {
        printHashCode(null) // 컴파일 오류!
    }
    ```

    - 이렇게 하면 T는 널이 될 수 없는 타입이 되어서 컴파일러가 더 강하게 체크할 수 있다.

> 타입 파라미터는 기본적으로 null이 가능하지만, 명시적으로 상한을 설정해서 더 안전하게 만들 수 있다!
>

---

## 7.12 널 가능성과 자바

- 코틀린에서는 **널 가능성(nullability)** 을 타입 시스템에 포함해서 컴파일 시점에 검증하지만, **자바**는 널 가능성 정보를 타입에 명시하지 않음.
- 그래서 코틀린에서 **자바 코드를 사용할 때**는 주의해야 한다.

### 플랫폼 타입 (Platform Type)

- **플랫폼 타입**은 자바에서 저장되어 있는 파일을 여러번 통화할 때 확인할 수 없는 널 가능성 정보를 가지는 타입을 말한다.
- 자바에서는 보통 타입에 `@Nullable` 또는 `@NotNull` 같은 표시를 하지 않기 때문에 코틀린 입장에서는 “이게 널이 될까?”를 알 수 없다.
- 그래서 플랫폼 타입을 사용할 때는 널 검사(null check) 를 해야하고, 널 아님 단언(!!)이나 안전한 호출(?.)을 써야한다.

    ```kotlin
    val str: String = person.name  
    // person.name의 타입이 플랫폼 타입이면 null이 될 수 있어서 위험
    ```

    - 만약 `person.name`이 자바에서 가져온 **플랫폼 타입**이라면, 실제로는 null이 될 수도 있기 때문에, 이렇게 바로 대입하면 위험할 수 있다.
- 코틀린 컴파일러는 경고를 줄 수도 있고, 런타임에서 NPE가 발생할 수 있습니다.

**플랫폼 타입 정리**

| 자바 타입 | 코틀린 해석 | 설명 |
| --- | --- | --- |
| `@Nullable String` | `String?` | 널이 될 수 있음 |
| `@NotNull String` | `String` | 널이 될 수 없음 |
| `String` (어노테이션 없음) | 플랫폼 타입 (`String!`) | 널 가능성을 알 수 없음 |

> 자바 타입에 어노테이션이 없으면, 코틀린은 플랫폼 타입(String!)으로 간주한다!
>

### 상속

- 자바 클래스를 코틀린에서 상속(extends/implements)하거나 오버라이드할 때, 널 가능성(nullability)을 명시적으로 지정해야한다.
- 자바 메서드의 반환 타입이 플랫폼 타입이면 개발자가 직접 선택해야 한다.
    - 코틀린에서 널이 될 수 없는 타입(String)으로 선언할지
    - 널이 될 수 있는 타입(String?)으로 선언할지

```java
/* Java */
public class Person {
    private final String name;

    public Person(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```

이를 Kotlin에서 상속할 경우

```kotlin
class KotlinPerson(name: String) : Person(name)
```

- 여기서 `getName()`은 널이 안 된다고 (`String`) 명시했다.
- 만약 자바 쪽에서 `null`을 리턴할 가능성이 있으면 위험해질 수 있다.
- 이럴 때는 코틀린 쪽에서도 **`String?`로 오버라이드** 해야 안전하다.

```kotlin
override fun getName(): String? {
    return super.getName()
}
```

- 자바 메서드를 오버라이드할 때는 널 가능성을 정확히 판단해서 코드를 작성해야 한다.

---

## 요약

- 코틀린은 **널이 될 수 있는 타입**을 지원해 **컴파일 시점에 NullPointerException(NPE)** 오류를 예방할 수 있다.
- **일반 타입**은 기본적으로 널이 될 수 없고, **물음표(?)가 붙은 타입**만 널이 될 수 있다.
- 널 가능성 처리를 위해 다양한 도구를 제공한다:
    - **안전한 호출 연산자(?.)** : 널일 수 있는 객체의 메서드나 프로퍼티에 접근할 때 사용.
    - **엘비스 연산자(?:)** : 널이면 기본값을 지정하거나 예외를 던지게 할 수 있다.
    - **널 아님 단언(!!)** : 강제로 null이 아니라고 선언할 수 있으나, 실패 시 NPE가 발생한다.
    - **let 영역 함수**: 안전한 호출과 함께 사용하여 널이 아닌 경우에만 코드를 실행할 수 있다.
    - **as? 연산자**: 타입을 안전하게 변환하며, 변환에 실패하면 null을 반환한다
- 코틀린은 널 가능성을 코드에 명시적으로 반영하여, 보다 안전하고 명확한 프로그래밍을 가능하게 만든다.