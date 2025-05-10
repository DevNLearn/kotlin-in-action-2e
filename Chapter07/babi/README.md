## 7장

### Null 네이놈

- `NullPointerException` 
- 런타임에서 발견할수 있었던 오류를 컴파일시 발견할수 있도록 null 을 다룬다.
- 개인적으로 자바에서 해당 에러가 발생하지 않도록 많은 노력을 했었다.
```java
import java.util.Objects;

Objects.equals(a, b);
Objects.isNull(a);
```
- 그럼에도 불구하고 간헐적으로 발생...

---

#### Null 이 될 수 있는 변수

- 자바에서 `Wrapper Type` 데이터타입으로는 `null`이 들어갈 수 있다.
- 예제로 다음 함수에서 들어온 변수에 대해 널 체크를 하지 않는다면 `NullPointerException` 이 발생한다.
```java
int strLen(String s){
    return s.length();
}
```
- `NotNull`을 명시하더라도 `warning` 표시만 날뿐, 못쓰도록 강제화되지 않는다.
```java
int strLen(@NotNUll String s){
    return s.length();
}

void getLength(){
    // Passing 'null' argument to parameter annotated as @NotNull 주의 발생은 하는데 강제화하여 막지는 않는다
    strLen(null);
}
```
- 코틀린에서는 `?` 기호를 데이터타입 뒤에 붙임으로써, 해당 변수의 `null` 가능성을 표시한다.
```kotlin
// str 변수는 null 가능성이 없다.
fun strLen(str: String){
    // Do it
}

// str 변수는 null 일수도 있다.
fun strLen(str: String?){
    // Do it
}
```
- 그리고 자바와 다르게 `?` 기호가 없는 변수에 `null` 을 넣는다면 컴파일시점에서 부터 에러가 발생한다.
```kotlin
// str 변수는 null 가능성이 없다.
fun strLen(str: String){
    // Do it
}

fun getTest(){
    // 에러발생 : Null can not be a value of a non-null type String
    strLen(null)
}
```
- 또한 `null` 가능성이 있는 타입은 바로 쓸수없는데, `String` 과 `String?` 은 다른 변수로 취급이된다.
```kotlin
fun main() {
    val x: String? = null
    // Type mismatch. Required: String, Found: String?
    strLen(x)
}

fun strLen(str: String){
    // Do it
}
```
---
### null 을 다루기 위한 도구들
- 코틀린에서 스마트 캐스트가 잘된다는걸 앞전에 봤었는데, `null` 관련된 작업들도 한번 비교하면 컴파일러가 `null` 이 아님을 기억한다
- 그리고 null 을 다루기 위한 도구들로는 다음처럼 있다.
#### if
```kotlin
fun strLenSafe(s: String?): Int {
    // 가능
    if (s != null) s.length else 0
}
```

---
#### ?.
- 호출연산자로써, `null` 검사와 메서드 호출을 하나로 수행한다.
- 예로, `str?.uppercase()` 은 `str` 변수가 `null` 인지 검사를 하고, `null` 이 아니라면 `uppercase()` 메서드까지 실행한다.
  - `str != null` -> `str.uppercase()`
  - `str == null` -> `null`
- `str` 이 `null` 이 아니라면 `String` 이 반환되고, `str` 이 `null` 이라면 `null` 이 반환되기 때문에, 해당 식의 데이터타입은 `String?` 이다.
- 연속해서 다음처럼 사용할 수도 있다.
```kotlin
class Address(val company: Company?)
class Company(val person: Person?)
class Person(val name: String?)

fun getName(address: Address){
    // 연속해서 사용할 수 있긴하지만 좋은코드인지는 음...
    val name = address?.company?.person?.name
}
```
---
#### ?:
- 엘비스 연산자라고 불린다.
- `nullable` 인 변수를 `nullable` 하지 않게 만들어 준다.
- 예로, `nullable` 한 변수인 `val name: String?` 을 엘버스 연산자를 사용하여 `val notNullName = name ?: "unnamed"` 이라고 한다면 결과는 다음과 같다
  - `name != null` -> name
  - `name == null` -> "unnamed"
- `str` 이 `null` 인 경우에도 항상 `String` 데이터타입으로 반환되기 때문에 `nullable` 하지 않아진다.
- 엘버스 연산자 뒤로 동일한 데이터타입만 들어가야하는건 아니다. `throw`, `return` 등도 식이기 때문에 들어갈 수 있다

---

#### as?
- 타입 캐스트 연산자인 `as` 에 `?` 를 붙여 안전하게 변환할 수 있다.
- `other as? Person`
  - `other is Person` -> `other as Person`
  - `other !is Person` -> `null`
- 일반적으로 엘비스 연산자와 함께 사용하여 `null` 일때 특정 작업을 하면 유용하게 사용할 수 있다
```kotlin
class Person(val firstName: String, val lastName: String) {
    override fun equals(other: Any?): Boolean {
        val otherPerson = o as? Person ?: return false // 타입이 서로 맞지 않다면 바로 return false
        // equals 연산
    }
}
```
---

#### !!
- 널 아님 단언 코드로, `?` 연산자가 사용되어 `null` 이 될수있는 변수이지만 절대 `null` 이 들어오지 않으니 널이 아닌 타입으로 강제로 바꾸는 연산자.
```kotlin
fun strLength(str: String?){
    str!!.length // String? 을 String 으로 강제 캐스트
}
```

- 하지만 `!!` 연산자를 통해 강제 캐스트를 했는데, 실제로 `null` 이 들어온다면, `NullPointerException`이 발생하게 된다.
- `!!` 연산자를 사용하지 않으려면 다음과 같이 사용할 수도 있다 `val value = list.selectedValue ?: return`
  - `list.selectedValue` 의 값에 따라 조기종료가 되어 이후 로직에서는 `null` 이 아님을 보장 받는다.
- 혹시 사용을 한다면 `!!` 연산자를 한 줄에 함께 쓰는 일을 피하는것이 좋다. **스택 트레이스**에서 어떤 파일의 몇 번째 줄인지는 나오지만, 어떤 식에서 발생했는지 나오지 않는다.
  - `person!!.company!!.address!!.country // 추천하지 않는다.`

---

#### let
- `null` 이 될 수 있는 식을 더 쉽게 다룰 수 있다. 널이 될 수 있는 값을 검사한 다음 그 결과에 변수를 넣는 작업을 한번에 처리할 수 있다.
- 흔한 용례로, 함수의 파라미터로 널이 아닌 값을 보내야할때, 널이될수 있는 변수를 가지고 해당 함수를 호출하는 경우가 있다.
```kotlin
// null 이 아닌 값을 파라미터로 받는 함수
fun sendEmailTo(email: String) { }

// 1. 바로 호출하면 에러 발생
fun main() {
  val email: String? = "mail"
  // Error: Type mismatch 
  sendEmailTo(email)
}

// 2. 조건문으로 null 여부 검사 후 호출
fun main() {
  val email: String? = "mail"
  // 정상동작 
  if(email != null) sendEmailTo(email)
}

// 3. let 을 사용하여 null 여부 검사 후 호출
fun main() {
  val email: String? = "mail"
  // 정상동작 
  email?.let { sendEmailTo(email) }
}
```
- `email?.let { sendEmailTo(email) }` 은 다음과 같이 분기되어 실행된다
  - `email != null` -> let 구문에서 `it`은 널이 아니다
  - `email == null` -> let 구문 안타고 종료

---

#### also

- 객체를 수정하거나 로깅등을 수행할 때 사용한다.
- also 블록 안에서는 it 키워드를 통해 주체 객체에 접근할 수 있다.
- 주체 객체를 그대로 반환하기 때문에, 체이닝을 이어갈 수 있다.
```kotlin
fun main() {
  val list = mutableListOf("a", "b")
  .also { println("초기 리스트: $it") }
  .also { it.add("c") }
  .also { println("변경된 리스트: $it") }
}
```

---

#### apply
- 객체를 생성하고 객체의 속성을 설정할 때 사용한다.
- apply 블록 안에서는 this 키워드를 통해 객체 멤버에 접근할 수 있으며, this는 생략 가능하다.
- 주체 객체를 그대로 반환한다.

```kotlin
fun main() {
  val user = User().apply {
  name = "민수"
  age = 20
  email = "minsu@example.com"
  }
}
```

---
#### run
- 객체에 대해 작업을 수행하고, 최종 결과를 얻고 싶을 때 사용한다.
- run 블록 안에서는 this 키워드를 통해 객체 멤버에 접근할 수 있다.
- run 블록의 마지막 표현식이 최종 반환된다.
```kotlin
fun main() {
  val length = "Hello Kotlin".run {
    println("문자열 출력: $this")
    length  // 마지막 표현식이 반환된다
  }
  println(length) // 12
}
```

---
#### with
- 주어진 객체에 대해 여러 작업을 수행할 때 사용한다.
- with는 확장 함수가 아니라 일반 함수이며, 첫 번째 파라미터로 객체를 받고 블록 내부에서 this를 통해 접근한다.
- 블록 마지막 표현식이 반환된다.


```kotlin

fun main() {
  val result = with(StringBuilder()) {
    append("Hello, ")
    append("World!")
    toString()
  }
  println(result) // Hello, World!
}
```

----

### 영역 함수 비교

| 구분    | 대상 (수신 객체) | 반환값           | 블록 내 키워드     | 주요 용도                       | 주 사용 시점               |
|-------|------------|---------------|--------------|-----------------------------|-----------------------|
| let   | it         | 마지막 식         | it           | null 체크, 안전 호출              | 값이 null이 아닐 때 코드 실행   |
 | also  | it         | 수신 객체 (자기 자신) | it           | 부수 효과(side effect), 로깅, 디버깅 | 객체를 그대로 반환하며 중간 작업 삽입 |
| apply | this       | 수신 객체 (자기 자신) | this (생략 가능) | 객체 초기화, 빌더 패턴               | 객체 프로퍼티를 설정하고 반환할 때   |
 | run   | this       | 마지막 식         | this (생략 가능) | 결과 생성, 계산 후 결과 반환           | 객체 조작 후 결과가 필요할 때     |
| with  | this       | 마지막 식         | this (생략 가능) | 객체에 대해 여러 작업 수행             | 여러 메서드를 호출해야 할 때      |


---

### 지연 초기화 프로퍼티
- 객체를 생성한 이후 나중에 전용 메서드를 통해 초기화하는 프레임워크가 많다. 당장 테스트에서 사용하는 `JUnit` 에서도 `@BeforeAll`, `@BeforeEach` 어노테이션 메서드 안에서 초기화 로직을 수행한다.
- 코틀린에서는 클래스 안의 널이 아닌 프로퍼티를 생성자 안에서 초기화하지 않고 특별한 메서드 안에서 초기화할 수 없다. 일반적으로 생성자에서 모든 프로퍼티를 초기화해야한다.
- 초기화 값을 제공할 수 없으면 널이 될 수 있는 타입을 사용해야하고, 해당 타입은 `null` 검사 혹은 `!!` 연산자를 사용해야한다.
- 다음은 못생긴 코드의 예시다
```kotlin
class MyService{
    fun action(): String = "Actions"
}

@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class MyTest {
    private var myService: MyService? = null
  
    @BeforeAll
    fun setUp() {
        myService = MyService()
    }
  
    // 못생긴 코드들
    @Test
    fun test(){
      // `!!` 사용
      assertEquals("Actions", myService!!.action())
      // null 검사
      if(mySevice != null) 
          assertEquals("Actions", myService.action())
    }
}
```

---
- 위와같은 상황을 해결하기 위해 지연 초기화를 할 수 있다.
- `lateinit` 변경자를 붙임으로써 프로퍼티를 나중에 초기화할 수 있다.
```kotlin

class MyService{
    fun action(): String = "Actions"
}

@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class MyTest {
    // lateinit 사용을 통해 지연 초기화
    private lateinit var myService: MyService
  
    @BeforeAll
    fun setUp() {
        myService = MyService()
    }
  
    // 못생긴 코드들
    @Test
    fun test(){
      // null 검사를 하지 않아도 상관없다!
      assertEquals("Actions", myService.action())
    }
}
```
---
- `val` 프로퍼티는 파이널 필드로 컴파일되고, 반드시 생성자 안에서 초기화돼야 하기 때문에 지연 초기화 프로퍼티는 항상 **`var`** 여야 한다
- 그리고 프로퍼티를 초기화하기 전에 접근하면 `UniitializedPropertyAccessException`이 발생한다.
- `lateinit` 프로퍼티는 반드시 클래스의 멤버가 아니어도 된다. 함수 본문 안의 지역변수, 최상위 프로퍼티도 가능하다.

---

### 타입 파라미터의 널
- 코틀린 함수, 클래스의 모든 타입 파라미터는 `null` 이 될 수 있다.
- 파라미터 `T`를 클래스나 함수 안에서 타입 이름으로 사용하면 이름 끝에 물음표가 없더라도 `T`가 `null`이 될 수 있는 타입이다.

```kotlin
fun <T> printHashCode(t: T) {
    println(t?.hashCode()) // t가 null이 될 수 있으므로, 안전한 호출 사용
}

fun main() {
  printHashCode(null) // T 의 타입은 `Any?` 로 추론
}
```
- 타입이름 `T` 에는 물음표가 붙어있지 않지만, t 는 `null`을 받을 수 있다.
- 타입 파라미터가 null 이 아님을 확실히 하려면 타입 상계를 지정해야한다.
```kotlin
fun <T: Any> printHashCode(t: T) { // t는 더이상 null이 될 수 없다
  println(t.hashCode()) 
}

fun main() {
  printHashCode(null) // Error
}
```
