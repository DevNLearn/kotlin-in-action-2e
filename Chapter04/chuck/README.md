# 4. 클래스, 객체, 인터페이스

### 인터페이스 

```kotlin
interface Clickable {
  fun click()
}

class Button : Clickable {
  override fun click() {
      println("Click!")
  }
}
```

- 코틀린에서 인터페이스 정의는 위와 같이 `interface` 키워드를 사용하여 정의한다.
- 인터페이스를 구현할때에는 `@Override` 어노테이션을 붙이는 자바와 다르게 코틀린에서는 `override` 키워드를 꼭 붙여야한다.
- 코틀린에서는 상속이나 구현 모두 `:` 기호를 사용한다.
- 자바와 동일하게 상속은 최대 1개까지 가능하고, 인터페이스는 여러개 구현할 수 있다.

```kotlin
interface Clickable {
  fun click()
  fun showOff() = println("I'm clickable!")
}
```

- `interface`에 `default` 구현을 정의할 수 있다.

### open, final, abstract

기본적으로 코틀린에서 모든 클래스는 `final`이다. 즉, 상속이 불가능하다.<br>
상속을 허용하고 싶다면 `open` 키워드를 사용해야한다.

```kotlin
open class RichButton : Clickable {
  fun disable() {}
  open fun animate() {}
  override fun click() {}
  final override fun glick() {}
}
```

- `open` 키워드를 사용하여 상속을 허용할 수 있다.
- 메서드도 마찬가지로 `open` 키워드를 통해 허용한 메서드만 재정의가 가능하다.
- override 메서드도 기본적으로 재정의가 가능하지만, `final` 키워드가 붙은 경우에는 재정의가 불가능하다.

```kotlin
abstract class Animated {
    abstract val animationSpeed: Double
    abstract fun animate()
}
```
- `abstract` 키워드를 사용하여 추상 클래스를 만들 수 있다.
- 추상 메서드와 추상 프로퍼티를 정의할 수 있다. (추상 프로퍼티의 경우, 값이나 접근자를 지정해야한다.)

### 가시성 변경자

자바에서는 아무 변경자도 없는 경우 package private 이지만, 코틀린에서는 `public`이다.<br>

- `public` : 어디서든 접근 가능
- `protected` : 하위 클래스에서만 접근 가능
- `private` : 같은 클래스에서만 접근 가능
- `internal` : 같은 모듈에서만 접근 가능

### 봉인된 클래스

코틀린의 `sealed class`는 제한된 클래스 계층 구조를 정의할 때 사용하는 특별한 종류의 클래스이다.

- **제한된 하위 클래스**: `sealed` 클래스의 하위 클래스는 반드시 같은 파일 내에 선언되어야 한다.
- **열거형(enum)의 확장**: enum보다 더 유연한 방식으로 제한된 타입 집합을 표현할 수 있다.
- **타입 안전성**: 컴파일러가 모든 가능한 하위 타입을 알고 있어서 when 표현식에서 모든 경우를 처리했는지 확인할 수 있습니다.
- **패턴 매칭**: `when` 표현식과 함께 사용하면 강력한 패턴 매칭이 가능합니다.

```kotlin
sealed class Result {
    data class Success(val data: String) : Result()
    data class Error(val message: String, val code: Int) : Result()
    object Loading : Result()
    object Empty : Result()
}

fun handleResult(result: Result) = when(result) {
    is Result.Success -> println("성공: ${result.data}")
    is Result.Error -> println("오류: ${result.message}, 코드: ${result.code}")
    is Result.Loading -> println("로딩 중...")
    is Result.Empty -> println("데이터 없음")
    // 모든 케이스가 처리되었으므로 else 분기가 필요 없음
}
```

- `when` 에서 모든 케이스가 처리되었으므로 `else` 분기가 필요하지 않다.
- 만약 `Result`의 하위 클래스가 추가되면, `when` 절이 컴파일 오류가 나서 모든 경우를 처리했는지 확인할 수 있다.


### 주 생성자와 부 생성자 (+ 초기화 블록)

```kotlin
class User(val nickname: String)
```

- 코틀린에서는 클래스 이름 뒤에 오는 `(val nickname: String)` 영역이 주 생성자

<br>

```kotlin
class User constructor(_nickname: String) {
    val nickname: String

    init {
        println("calling init")
        nickname = _nickname
    }
}

fun main() {
    User("chuck")
    User("millo")
}
/*
calling init
calling init
 */
```

- `constructor` 키워드는 주 생성자나 부 생성자 정의를 시작할때 사용된다.
- 초기화 블록은 `init {}` 형태로 작성되고, 인스턴스가 생성되면서 실행된다.

<br>

```kotlin
class User(
    val nickname: String,
    val isSubscribed: Boolean = true
)
```

- 주 생성자에서 기본값을 지정할 수 있다. 위 예시에서는 `isSubscirbed`의 값을 별도로 지정하지 않은 경우 `true`로 초기화된다.

<br>

```kotlin
fun main() {
    val alice = User("Alice")
}
```

- 인스턴스를 생성할때에는 자바에서는 `new` 키워드를 사용했지만, 코틀린에서는 `new` 키워드 없이 인스턴스를 생성할 수 있다.

<br>

```kotlin
open class User(val nickname: String)
class SocialUser(nickname: String): User(nickname)
```

- 만약 상속하고자 하는 클래스가 생성자로 인자를 받아야한다면, 이름 뒤에 괄호를 치고 생성자 인자를 넣어주면 된다.

<br>

```kotlin
open class Button
class MyButton: Button()
```

- 그동안의 예제에서 상속하는 하위 클래스에 괄호를 넣어줬던 것도 생성자를 호출해야했기 때문이다.
- 반대로, 인터페이스의 경우에는 생성자 호출이 필요없었어서 괄호가 필요 없었던 것이다.

<br>

```kotlin
import java.net.URI

open class Downloader {
    constructor(url: String?) {
        println("Downloader")
    }

    constructor(uri: URI?) {
        println("Downloader with URI")
    }
}

// #1
class MyDownloader : Downloader {
    constructor(url: String?) : super(url) {
        println("MyDownloader")
    }

    constructor(uri: URI?) : super(uri) {
        println("MyDownloader with URI")
    }
}

// #2
class MyDownloader : Downloader {
    constructor(url: String?) : this(URI(url)) {
        println("MyDownloader")
    }

    constructor(uri: URI?) : super(uri) {
        println("MyDownloader with URI")
    }
}
```

- 상위 클래스의 초기화를 부 생성자를 통해 할 수 있다.

### 데이터 클래스


```kotlin
data class Customer(val name: String, val postalCode: Int)

fun main() {
    val c1 = Customer("Sam", 11521)
    val c2 = Customer("Mart", 15500)
    val c3 = Customer("Sam", 11521)

    println(c1)
    println(c1 == c2)
    println(c1.hashCode())
    println(c3.hashCode())
}

/*
Customer(name=Sam, postalCode=11521)
false
2580770
2580770
 */
```
- `toString`, `equals`, `hashCode` 메서드를 자동으로 생성해준다.
- 데이터 클래스는 불변 클래스로 만드는게 권장된다.
  - 불변성을 가지게 되는 경우, HashMap 등의 컬렉션에 넣을때 내부 데이터가 변경되지 않아 안전하다.
  - 다중 스레드 환경에서 데이터가 다른 스레드에 의해 변경되는 것을 방지할 수 있다.

<br>

```kotlin
data class Customer(val name: String, val postalCode: Int)

fun main() {
    val lee = Customer("이계영", 4122)
    println(lee.copy(postalCode = 4000))
}

/*
Customer(name=이계영, postalCode=4000)
 */
```

- 데이터 클래스 인스턴스를 불변 객체로 더 쉽게 활용할 수 있도록 `copy` 메서드를 제공한다.
- `copy` 메서드는 인스턴스를 복사하면서 일부 속성의 값을 변경할 수 있다.

### 클래스 위임: by 키워드

```kotlin
class CountingSet<T>(
    private val innerSet: MutableCollection<T> = hashSetOf()
) : MutableCollection<T> by innerSet {
    var objectsAdded = 0

    override fun add(element: T): Boolean {
        objectsAdded++
        return innerSet.add(element)
    }

    override fun addAll(elements: Collection<T>): Boolean {
        objectsAdded += elements.size
        return innerSet.addAll(elements)
    }
}

fun main() {
    val cset = CountingSet<Int>()
    cset.addAll(listOf(1, 1, 2))
    println("Added ${cset.objectsAdded} objects, ${cset.size} uniques.")
}
```

- `by` 키워드를 사용하여 위임을 할 수 있다.
- 다른 클래스의 기능을 그대로 가져오되, 상속으로 인한 문제(취약한 기반 클래스 문제 등)를 피하고 싶을 때 사용
- `MutableCollection<T> by innerSet`는 `MutableCollection` 인터페이스의 모든 메서드를 `innerSet`에 위임한다.

### object 키워드

```kotlin
object Payroll {
    val allEmployees: MutableList<String> = mutableListOf()
    
    init {
        allEmployees.add("John Doe")
        allEmployees.add("Jane Smith")
        allEmployees.add("Alice Johnson")
    }
}

fun main() {
    println(Payroll.allEmployees)
}

/*
[John Doe, Jane Smith, Alice Johnson]
 */
```

```kotlin
data class Person(val name: String) {
    object NameComparator : Comparator<Person> {
        override fun compare(p1: Person, p2: Person): Int {
            return p1.name.compareTo(p2.name)
        }
    }
}

fun main() {
    val persons = listOf(Person("John"), Person("Smith"))
    println(persons.sortedWith(Person.NameComparator))
}

/*
[Person(name=John), Person(name=Smith)]
 */
```

- 코틀린에서는 `object` 키워드를 사용하여 싱글톤을 만들 수 있다.
- 익명 내부 클래스 를 만들때도 사용된다.
- 객체 선언부에 생성자를 쓸 수 없다.

<br>

```kotlin
class MyClass {
    companion object {
        fun callMe() {
            println("Companion object called")
        }
    }
}

fun main() {
    MyClass.callMe()
}

/*
Companion object called
 */
```

- 코틀린에서는 `static` 키워드가 없기 때문에, `companion object`를 사용하여 `static` 메서드를 만들 수 있다.
- `companion object`는 클래스의 인스턴스가 아닌 클래스 자체에 속하는 객체이다.
- `companion object`는 팩토리 메서드와 같은 역할을 할 수 있다.

<br>

```kotlin
class Person(val name: String) {
  companion object Loader {
    fun fromJSON(jsonText: String): Person = /* */
  }
}

fun main() {
  val person = Person.fromJSON("{\"name\": \"John Doe\"}")
  println(person.name)
}
/*
John Doe
 */
```

- `companion object`를 일반 객체처럼 사용할 수 있다.

<br>

```kotlin
// 비즈니스 로직 모듈
class Person(val firstName: String, val lastName: String) {
    companion object {
        
    }
}

// 클라이언트/서버 통신 모듈
fun Person.Companion.fromJSON(json: String): Person {
    /* ... */
}

val p = Person.fromJSON(json)
```

- 관심사를 더욱 명확하게 분리하고 싶을때, 위와 같이 통신 모듈이 비즈니스 로직 모듈에 의존하지 않게 구현할 수 있다.<br>
-> 개인적으로는 이런식으로 분리하면 코드 추적이 힘들어보이는데 정말 이렇게까지 의존성을 분리하는게 맞는가 싶다.

<br>

```kotlin
interface MouseListener {
  fun onEnter()
  fun onClick()
}

class Button(private val listener: MouseListener) {
    
}
```

- 일반적으로 인터페이스는 클래스의 구현을 통해 구현되는데, 클래스 구현없이 인터페이스를 구현하려는 경우<br>
아래와 같이 작성하면, 구현할 수 있다.

```java
// java
public class Main {
  public static void main(String[] args) {
    MouseListener mouseListener = new MouseListener() {
      @Override
      public void onEnter() {}

      @Override
      public void onClick() {}
    };

    new Button(mouseListener);
  }
}
```

```kotlin
// kotlin
fun main() {
  val mouseListener = object : MouseListener {
    override fun onEnter() {}
    override fun onClick() {}
  }
  
  Button(mouseListener)
}
```

### 인라인 클래스

```kotlin
@JvmInline
value class UsdCent(val amount: Int) {
  fun toDollar(): Double {
    return amount / 100.0
  }
}
```
- 인라인 클래스는 `@JvmInline` 어노테이션을 사용하여 정의할 수 있다.
- 인라인 클래스는 단일 값을 래핑하면서도 런타임에 해당 값으로 대체되어 오버헤드를 제거하는 코틀린의 특별한 클래스다. 
- 주로 기본 타입에 의미를 부여하여 타입 안전성을 높이는 데 사용됩니다. 
- 동일한 기본 타입이라도 의미적으로 다른 값들을 컴파일 시점에 구분할 수 있게 해줍니다.

`Java` 에서는 `UnsignedInt`가 없는데 이런식으로 활용해볼 수도 있을 것 같다.

```kotlin
@JvmInline
value class UnsignedInt(val value: Int) {
  init {
    require(value >= 0) { "UnsignedInt must be non-negative" }
  }
}

fun main() {
  val no = UnsignedInt(-1)
  println(no.value)
}

/*
Exception in thread "main" java.lang.IllegalArgumentException: UnsignedInt must be non-negative
 */
```
