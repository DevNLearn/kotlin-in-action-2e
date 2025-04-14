# 4. 클래스, 객체, 인터페이스
코틀린의 함수 정의 방식, 컬렉션 처리, 확장 함수, 문자열과 정규식, 클래스 계층 구조, 생성자 초기화 방식, equals/hashCode 오버라이딩, 싱글턴과 동반 객체 정의, 인라인 클래스

## 4.1 클래스 계층 정의

- 코틀린은 클래스 상속과 인터페이스 구현에서 `:` 기호를 사용하며, 클래스는 단일 상속만 가능하고 인터페이스는 다중 구현이 가능함.

### 인터페이스

- 코틀린 인터페이스는 추상 메서드와 디폴트 구현 메서드를 모두 포함할 수 있음.

```kotlin
interface Clickable {
	fun click()                                <- 일반 메서드 선언
	fun showOff() = println("I'm clickable!")  <- 디폴트 구현이 있는 메서드
}

class Button : Clickable {
	override fun click() = println("I was clicked")
}
```

- `override`는 반드시 명시해야 하며, 인터페이스의 디폴트 메서드는 메서드 본문으로 제공됨.
- 동일 시그니처의 디폴트 메서드를 여러 인터페이스로부터 상속받으면, 반드시 명시적으로 오버라이딩하고 `super<인터페이스>.메서드()` 형태로 호출해야 함.
- 같은 이름의 디폴트 메서드를 여러 인터페이스로부터 상속받으면 **컴파일 오류가 발생함** → 해결하려면 명시적으로 오버라이딩 필요.

    ```kotlin
    interface Focusable {
        fun showOff() = println("I'm focusable!")
    }
    
    class Button : Clickable, Focusable {
        override fun click() = println("I was clicked")
        override fun showOff() {
            super<Clickable>.showOff()
            super<Focusable>.showOff()
        }
    }
    
    fun main(args: Array<String>) {
    	val button = Button()
    	button.showOff()
    	// I'm clickable!
    	// I'm focusible!
    	button.click()
    	// I was Clicked.
    }
    ```

- 자바에서는 코틀린 인터페이스의 디폴트 메서드를 자동으로 상속하지 않음 → 직접 구현해야 함.

    ```kotlin
    public class JavaButton implements Clickable {
        @Override
        public void click() {
            System.out.println("I was clicked");
        }
    
        @Override
        public void showOff() {
            System.out.println("I'm showing off");
        }
    }
    ```


### **클래스 상속과 open/final**

- 코틀린 클래스는 기본적으로 `final`. 상속하려면 `open`을 명시해야 함.
- `override`는 기본적으로 `open`, 하지만 `final`로 다시 닫을 수 있음.

    ```kotlin
    open class RichButton: Clickable { <- 이 클래스는 열려 있음. 다른 클래스가 상속 가능
    	fun disable() {/* ... */}  <- 이 함수는 파이널, 하위 클래스가 override 할 수 없음.
    	open fun animate() {/* ... */}  <- 이 함수는 열려 있다. 하위 클래스에서 이 메서드를 override 해도됨.
    	override fun click() {/* ... */} <- 상위 클래스에서 열려있어서 override 함.
    }
    ```

  > “상속을 위한 설계와 문서를 갖춰라. 그럴 수 없다면 상속을 금지하라.”
    - 이펙티브 자바/ Joshua Block-
  >

### 스마트 캐스트와 final

- `val` + `final` + 기본 getter 프로퍼티는 스마트 캐스트에 활용 가능.
- 프로퍼티가 open이거나 custom getter가 있다면 스마트 캐스트 불가.
- final 프로퍼티는 타입 검사 이후 값이 바뀌지 않는다는 보장을 제공함

### **추상 클래스와 멤버**

- `abstract class`: 인스턴스화 불가, 일부 멤버는 추상으로 선언 가능.
- 추상 멤버는 항상 open 상태이며, 구현이 없어야 함.

```kotlin
abstract class Animated { 
	abstract val animationSpeed: Double <- 추상 프로퍼티는 값이 없으며 
																					하위 클래스에서 반드시 값이나 접근자를 제공할 필요가 있다.
	val keyframes: Int = 20  <- 추상클래스의 (추상이 아닌) 프로퍼티는 기본적으로 열려있지 않음.
	open val frames: Int = 60    하지만 open으로 지정도 가능.
	
	abstract fun animate()  <- 추상 함수는 구현이 없고, 하위 클래스는 이 함수를 반드시 Override
	open fun stopAnimating() { /* ... */}  <- 추상 클래스의 (추상이 아닌) 함수는 기본적으로는
	fun animateTwice() { /* ... */}           열려있지 않지만 open으로 지정도 가능.
}
```

| 변경자 | 의미 | 설명 |
| --- | --- | --- |
| `final` | 오버라이드 할 수 없음 | 클래스 멤버의 기본 변경자 |
| `open` | 오버라디으할 수 있음 | 반드시 open을 명시해야 오버라이드할 수 있다. |
| `abstract` | 반드시 오버라이드 해야함 | 추상 클래스의 멤버에만 이 변경자를 붙일 수 있다. 추상 멤버에는 구현이 있으면 안된다. |
| `override` | 상위 클래스나 인스턴스의 멤버를 오버라이드하는 중 | 오버라이드하는 멤버는 기본적으로 열려있다.
하위 클래스의 오버라이드를 금지하려면 final을 명시해야한다. |

### **가시성 변경자**

| 변경자 | 클래스 멤버 | 최상위 선언 |
| --- | --- | --- |
| `public`
(기본 가시성) | 모든 곳에서 볼 수 있다. | 모든 곳에서 볼 수 있다. |
| `internal` | 같은 모듈 안에서만 볼 수 있다. | 같은 모듈 안에서만 볼 수 있다. |
| `protected` | 하위 클래스 안에서만 볼 수 있다.
(최상위 선언에 적용할 수 없음.) | - |
| `private` | 같은 클래스 안에서만 볼 수 있다. | 같은 파일 안에서만 볼 수 있다. |
- public 함수에서는 더 낮은 가시성의 타입을 반환하거나 참조할 수 없음.

    ```kotlin
    internal open class TalkativeButton {
        private fun yell() = println("Hey!")
        protected fun whisper() = println("Let's talk!")
    }
    
    fun TalkativeButton.giveSpeech() {
        // 오류 발생
        // yell() → private
        // whisper() → protected
    }
    ```


> 코틀린의 public, protected, private 가시성은 자바 바이트코드에도 그대로 유지되며, private 클래스만 예외적으로 패키지 전용 클래스로 컴파일된다.
internal 변경자는 자바에서 정확히 대응되는 가시성이 없기 때문에 바이트코드에서는 public으로 컴파일되지만, 컴파일러가 이름을 망글링하여 실질적으로 외부에서 사용하기 어렵게 만든다.
>

### 내부 클래스와 내포된 클래스

- **코틀린 클래스 안에 정의된 클래스는 기본적으로 중첩 클래스(nested class)이며 static**이다.
- 바깥 클래스 인스턴스를 참조하려면 `inner` 변경자를 붙여야 함

    ```kotlin
    class Outer{
    	inner class Inner {
    		fun getOuterReference(): Outer = this@Outer
    	}
    }
    ```

- **직렬화와 클래스 구조 예시**

    ```kotlin
    interface State: Serializable
    
    interface View {
    	fun getCurrentState(): State
    	fun restoreState(state: State) {/* ... */}
    }
    
    class Button : View {
        override fun getCurrentState(): State = ButtonState()
        override fun restoreState(state: State) {}
        class ButtonState : State {}
    }
    ```

    - 코틀린에서는 `class`로 선언한 중첩 클래스는 자바의 `static class`처럼 바깥 인스턴스를 참조하지 않음. → 직렬화 가능
    - 자바에서는 `static class`가 아닌 내부 클래스는 바깥 인스턴스에 대한 참조를 가져 직렬화 시 문제가 생김.

  > 💡 자바에서 직렬화 오류 방지하려면 반드시 static 내부 클래스를 사용해야 하며,
  코틀린은 기본적으로 중첩 클래스가 static이기 때문에 문제 발생하지 않음
>

### **봉인된 클래스**

- `sealed class`는 상속 가능한 하위 클래스의 범위를 제한.
- 같은 패키지 내에서 정의된 하위 클래스만 허용됨.
- `when` 식에서 모든 하위 클래스가 처리되면 `else` 분기 생략 가능.

    ```kotlin
    sealed class Expr
    class Num(val value: Int) : Expr()
    class Sum(val left: Expr, val right: Expr) : Expr()
    
    fun eval(e: Expr): Int = when(e) {
        is Num -> e.value
        is Sum -> eval(e.left) + eval(e.right)
    }
    ```


**sealed class와 when**

- `sealed` 클래스의 하위 클래스는 반드시 같은 파일 안에 있어야 함.
- 새로운 하위 클래스 추가 시, `when` 분기에서 오류 발생 가능 (컴파일러가 모든 경우 체크하기 때문)

```kotlin
class Mul(val left: Expr, val right: Expr) : Expr()
// eval 함수에 'is Mul ->' 분기 없으면 컴파일 오류 발생
```

**sealed interface**

- `sealed interface`도 가능하며, 구현 클래스가 모두 같은 패키지 안에 있어야 함.

```kotlin
sealed interface Toggleable {
    fun toggle()
}

class LightSwitch : Toggleable {
    override fun toggle() = println("Lights!")
}

class Camera : Toggleable {
    override fun toggle() = println("Camera!")
}
```

- sealed 인터페이스 구현체도 when 분기에서 `else` 생략 가능.

## 4.2 뻔하지 않은 생성자나 프로퍼티를 갖는 클래스 선언

- 코틀린 클래스는 주 생성자(primary constructor)와 보조 생성자(secondary constructor)를 구분하며, 생성자와 초기화 로직은 매우 직관적으로 정의 가능함.
- 클래스 선언에서 `constructor` 키워드는 생략 가능하며, 프로퍼티 선언과 동시에 초기화를 수행할 수 있음.

### **클래스 초기화**

- 코틀린에서는 생성자에서 받은 인자를 바로 프로퍼티로 선언 가능

    ```kotlin
    class User(val nickname: String)
    ```

- 클래스 본문 없이도 객체 생성 및 초기화가 가능함.
- 생성자 파라미터에 기본값을 지정하면 다양한 방식으로 인스턴스를 생성할 수 있음.
    - 기본값은 함수 오버로딩 없이 다양한 인스턴스를 생성할 수 있도록 해줌

    ```kotlin
    class User(val nickname: String, val isSubscribed: Boolean = true)
    
    fun main() {
    	val alice = User("Alice")
    	println(alice.isSubscribed)
    	// true
    	
    	val bob = User("Bob", false)
    	println(bob.isSubscribed)
    	// false
    	
    	/* 생성자 인자 중 일부에 대해 이름을 지정할 수 있음. */
    	val carol = User("Carol", isSubscribed = false)
    	println(carol.isSubscribed)
    	// false
    	
    	/* 모든 생성자 인자에 대해 이름을 지정할 수 있음. */
    	val dave = User(nickname = "Dave", isSubscribed = true)
    	println(dave.isSubscribed)
    	// true
    }
    ```

  > 💡 `@JvmOverloads` 어노테이션을 사용하면 자바 호출 시 여러 생성자 버전을 자동 생성해줌
>
- 클래스의 **인스턴스를 외부에서 만들지 못하도록 막고 싶을 때** 비공개 생성자 사용.
    - 보통 싱글턴이나 생성 제한 목적으로 사용함, 코틀린에서는 `object` 키워드로 싱글턴 구현 가능.

    ```kotlin
    class Secretive private constructor(private val agentName: String) {}
    ```


### **주 생성자와 init 블록**

- 주 생성자는 클래스 헤더에 선언되며, `init` 블록에서 초기화 코드 실행 가능.

```kotlin
class User(val nickname: String) {
    init {
        println("User initialized with nickname: $nickname")
    }
}
```

- `init` 블록은 여러 개 작성할 수 있으며, 위에서 아래로 순차 실행됨

### **부 생성자**

- 부 생성자는 `constructor` 키워드로 정의함.

    ```kotlin
    open class Downloader {
    	constructor(url: String?) {
    		// 어떤 코드
    	}
    	constructor(uri: URI?) {
    		// 어떤 코드
    	}
    }
    ```

- 주 생성자가 있으면 보조 생성자는 반드시 `this(...)`를 통해 주 생성자에 위임해야 함

    ```kotlin
    class MyDownloader: Downloader {
    	constructor(url: String?): this(URI(url))
    	constructor(uri: URI?): super(uri)
    }
    ```


### **인터페이스 구현 시 프로퍼티 정의**

- 인터페이스의 프로퍼티도 구현 클래스에서 생성자 인자를 통해 바로 구현 가능.
  
  ```kotlin
  interface User {
      val nickname: String
  }
  
  /* 주 생성자에 있는 프로퍼티 */
  class PrivateUser(override val nickname: String): User 
  
  class SubscribingUser(val email: String): User {
      override val nickname: String
          get() email.substringBefore('@') ) <- 커스텀 게터
  }
  
  class SocialUser(val accountId: Int): User {
      override val nickname: getFacebookName(accountId) <- 프로퍼티 초기화 식
  }
  
  fun getNameFromSocialNetwork(accountId: Int) = "kodee$accountId"
  
  fun main() {
      println(PrivateUser("kodee").nickname)
      // kodee
      println(SubscribingUser("test@kotlinlang.org").nickname)
      // test
      println(SocialUser(123).nickname)
      // kodee123
  }
  ```
  
  > 💡 함수 대신 프로퍼티를 사용하는 경우:
  > - 예외를 던지지 않는다.
  >   - 계산 비용이 적게 든다(또는 최초 실행 후 결과를 캐시해 사용할 수 있다).
  >   - 객체 상태가 바뀌지 않으면 여러 번 호출해도 항상 같은 결과를 돌려준다.
  >

### 게터와 세터에서 뒷받침하는 필드에 접근

- 게터(getter)와 세터(setter)를 통해 프로퍼티 접근을 제어 가능.

    ```kotlin
    class User(val name: String) {
    	var address: String = "unspecified"
    		set(value: String) {
    			println(
    				"""
    				Address was changed for $name:
    				"$field" -> "$value".
    				""".trimIndent())
    			field = value
    		}
    }
    
    fun main() {
    	val user = User("Alice")
    	user.address = "Christoph-Rapparini-Bogen 23"
    	// Address was changed for Alice:
    	// "unspecified" -> "Christoph-Rapparini-Bogen 23".
    }
    ```


### **접근자 가시성 변경**

- `get`이나 `set` 키워드에 접근 제한자를 붙여 가시성을 조절할 수 있음

    ```kotlin
    class LengthCounter {
    	var counter: Int = 0
    	private set // 외부에서 counter 값 변경을 금지
    	
    	fun addWord(work: String) {
    		counter += word.length
    	}
    }
    
    fun main() {
    	val lengthCounter = LengthCounter()
    	lengthCounter.addWord("Hi!")
    	println(lengthCounter.counter)
    	// 3
    	 // lengthCounter.counter = 0 // 오류! setter가 private이라 외부에서 변경 불가
    }
    ```


## 4.3 컴파일러가 생성한 메서드: 데이터 클래스와 클래스 위임

코틀린에서는 `equals`, `hashCode`, `toString` 등 대부분의 클래스가 재정의하게 되는 메서드를 명시적으로 구현하거나, `data class`를 사용해 자동 생성할 수 있다.

### toString

- 자바처럼 `toString()`은 객체의 문자열 표현을 반환하는 메서드로, 디버깅이나 로깅에서 자주 사용됨.
- 코틀린의 일반 클래스는 `toString()`을 자동으로 구현하지 않기 때문에 수동 오버라이딩 필요.

```kotlin
class Customer(val name: String, val postalCode: Int) {
	override fun toString() = "Customer(name=$name, postalCode=$postalCode)"
}

fun main() {
	val customer1 = Customer("Alice", 342562)
	println(customer1)
	// Customer(name=Alice, postalCode=342562)
}
```

### **equals**

- `equals()`는 두 객체의 내용이 같은지 비교하며, `==` 연산자에 의해 내부적으로 호출됨.
- 기본 구현은 참조 동일성(`===`)을 비교하므로, 값 기반 비교를 위해선 오버라이딩 필요.

```kotlin
val customer1 = Customer("Alice", 342562)
val customer2 = Customer("Alice", 342562)
println(customer1 == customer2) // false
```

- `===`는 참조 동일성 비교, `==`는 equals 호출.
- equals 오버라이딩 시 타입 체크와 프로퍼티 비교를 포함해야 함.

```kotlin
class Customer(val name: String, val postalCode: Int) {
    override fun equals(other: Any?): Boolean {
        if (other == null || other !is Customer) 
	        return false
        return name == other.name && postalCode == other.postalCode
    }
    override fun toString() = "Customer(name=$name, postalCode=$postalCode)"
}
```

### hashCode

- `equals()`를 재정의하면 `hashCode()`도 반드시 같이 재정의해야 함.
- 같은 값을 가진 객체는 동일한 해시코드를 가져야 해시 기반 자료구조(Set, Map)에서 제대로 작동함.

```kotlin
class Customer(val name: String, val postalCode: Int) {
    /* .. */
    override fun hashCode(): Int = name.hashCode() * 31 + postalCode
}
```

### **데이터 클래스의 자동 구현**

- `data class`로 선언하면 필요한 메서드를 컴파일러가 자동으로 생성.
    - `toString`, **`equals`, `hashCode`**

```kotlin
fun main() {
	val c1 = Customer("Sam", 11521)
	val c2 = Customer("Mart", 15500)
	val c3 = Customer("Sam", 11521)
	
	println(c1) // Customer(name=Sam, postalCode=11521)
	println(c1 == c2) // false
	println(c1 == c3) // true
	println(c1.hashCode()) // 2580770
	println(c3.hashCode()) // 2580770
}
```

- `copy()` 함수는 일부 프로퍼티만 바꿔서 객체를 복사할 때 유용함.

```kotlin
class Customer(val name: String, val postalCode: Int) {
	/* ... */
	fun copy(name: String = this.name,
			postalCode: Int = this.postalCode) = Customer(name, postalCode)
}

fun main() {
	val lee = Customer("이계영", 4122)
	println(lee.copy(postalCode = 4000))
	// Customer(name=이계영, postalCode=4000)

}
```

### **자바의 레코드(Record)와 비교**

- 자바의 `record`는 값 객체 생성에 특화된 클래스
- `equals`, `hashCode`, `toString` 등을 자동 생성
- 코틀린의 `data class`는 자바 `record`보다 더 유연함
    - `copy()`와 구조 분해(componentN) 가능
    - 클래스 본문에 추가 메서드 정의 가능
    - `@JvmRecord`를 붙이면 자바와의 호환도 가능

### 클래스 위임: by 키워드 사용

- 클래스가 인터페이스를 구현하면서 구현을 다른 객체에 위임할 수 있음
- `MutableCollection<T> by innerSet` 구문으로 위임 가능.
- 일부 메서드만 오버라이딩하여 커스터마이징 가능

```kotlin
class CountingSet<T>(
    val innerSet: MutableCollection<T> = HashSet<T>()
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
    val cSet = CountingSet<Int>()
    cSet.addAll(listOf(1, 1, 2))
    println("Added: ${cSet.objectsAdded} elements") // 3
    println("Set contents: $cSet") // [1, 2]
}
```

> 💡 클래스 위임은 데코레이터 패턴을 간편하게 구현하는 데 유용함.
위임 대상 객체는 생성자에서 주입하거나 내부에서 생성할 수 있음.

## 4.4 Object 키워드: 클래스 선언과 인스턴스 생성을 한꺼번에 하기

코틀린의 `object` 키워드는 클래스 선언과 동시에 인스턴스를 생성하는 기능을 제공하며, 대표적으로 다음 세 가지 상황에서 사용된다

- 객체 선언 (싱글턴)
- 동반 객체 (Companion Object)
- 객체 식 (익명 객체)

### 객체 선언

- 자바에서는 생성자를 private로 제한하고 정적인 필드에 그 클래스의 유일한 객체를 저장하는 싱글턴 패턴을 통해 이를 구현함.
- 코틀린은 객체 선언 기능을 통해 싱글턴을 기본 지원함.
- 클래스 이름 앞에 `object`를 붙이면 해당 클래스는 인스턴스를 하나만 갖는 싱글턴이 된다.

    ```kotlin
    object Payroll {
        val allEmployees = arrayListOf<Person>()
        
        fun calculateSalary() {
            for (person in allEmployees) {
    	        /* .. */
            }
        }
    }
    ```

- 객체 선언은 즉시 초기화되며, 생성자를 직접 사용할 수 없다.
- 변수와 마찬가지로 객체 이름 뒤에 마침표(.)를 붙이면 객체에 속한 멤버에 접근 가능함.

    ```kotlin
    Payroll.allEmployees.add(Person(/* .. */))
    Payroll.calculateSalary()
    ```

- 일반 객체를 사용할 수 있는 곳에서는 항상 싱글턴 객체를 사용할 수 있음.

    ```kotlin
    object CaseInsensitiveFileComparator: Comparator<File> {
    	override fun compare(file1: File, file2: File): Int {
    		return file1.path.compareTo(file2.path,
    					ignoreCase = true)
    	}
    }
    
    val files = listOf(File("Z"), File("a"))
    println(files.sortedWith(CaseInsensitiveFileComparator))
    ```


**싱글턴과 의존관계 주입**

- 객체 선언은 싱글턴이지만 대규모 컴포넌트에는 적합하지 않을 수 있으며, 그럴 경우 일반 클래스와 의존성 주입을 활용.

**내포 객체를 사용한 Comparator 구현**

- Comparator도 객체 선언으로 간단히 정의 가능

    ```kotlin
    data class Person(val name: String) {
    	object NameComporator : Comparator<File> {
        override fun compare(p1: File, p2: File): Int =
            p1.name.compareTo(p2.name)
      }
    }
    
    fun main() {
    	val persons = listOf(Person("Bob"), Person("Alice"))
    	println(persons.sortedWith(Person.NameComprator))
    	// [Person(name=Alice), Person(name=Bob)]
    }
    ```


**코틀린 객체를 자바에서 사용하기**

- 객체 선언은 자바에서 `INSTANCE`로 접근함.

    ```java
    CaseInsensitiveFileComparator.INSTANCE.compare(file1,file2)
    ```


### 동반 객체

- 코틀린에는 자바의 정적 필드가 없으므로, 정적 멤버를 구현하기 위해 `companion object`를 사용.
- 클래스 내부에 정의되고, 클래스 이름으로 호출.

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
      // Companion object called
    }
    ```

- 인스턴스를 통해 동반 객체에 접근할 수는 없음.

    ```kotlin
    fun main() {
    	val myObject = MyClass()
    	myObject.callMe()
    	// Error: Unresolved reference: callMe
    }
    ```


**생성자 위임과 팩토리 메서드로 동반 객체 사용하기**

- 여러 개의 부 생성자를 정의하는 대신, **동반 객체의 팩토리 메서드**를 사용하는 것이 더 나은 대안이 될 수 있음.

    ```kotlin
    class User{
    	val nickname: String
    	
    	constructor(email: String) { <- 부 생성자
    		nickname = email.substringBefore('@')
    	}
    	
    	constructor(socialAccountId: Int) { <- 부 생성자
    		nickname = getSocialNetworkName(socialAccountId)
    	}
    }
    
    // 부 생성자를 팩토리 메서드로 대신하는 방법
    class User private constructor(val nickname: String) {
    	companion object {
    			fun newSubscribingUser(email: String) = 
    					User(email.substringBefore('@'))
    			fun newSocialUser(accountId: Int) = 
    					User(getNameFromSocialNetwork(accountId))
    	}
    }
    
    fun main() {
    	val subscribingUser = User.newSubscribingUser("bob@gmain.com")
    	val socialUser = User.newSocialUser(4)
    	println(subscribingUser.nickname)
    	// bob
    }
    ```

- `private constructor`는 클래스 외부에서 생성자를 호출하지 못하게 하고, 동반 객체에서 객체 생성을 위임함으로써 **생성자 호출을 제어**할 수 있음.

**동반 객체에서 인터페이스 구현**

- 동반 객체는 클래스와 동일한 이름의 팩토리 역할도 가능.
- 인터페이스 구현을 통해 JSON 역직렬화 팩토리 등 정의 가능.

    ```kotlin
    interface JSONFactory<T> {
        fun fromJSON(json: String): T
    }
    
    class Person(val name: String) {
        companion object : JSONFactory<Person> {
            override fun fromJSON(json: String): Person = /* ... */
        }
    }
    
    val p = Person.fromJSON("Kotlin")
    ```


**자바에서 코틀린 객체 사용하기**

- 동반 객체는 `Companion`이라는 이름으로 접근하거나, `@JvmStatic`으로 바로 접근 가능.

    ```java
    Payroll.Companion.calculate();
    
    class Employee(val name: String) {
        companion object {
            @JvmStatic fun fromJSON(json: String): Employee = Employee(json)
        }
    }
    ```


**동반 객체 확장**

- 클래스 외부에서 동반 객체에 대해 **확장 함수**를 정의.
- 모듈 분리 시에 유용.

    ```java
    //비즈니스 로직 모듈
    class Person(val firstName: String, val lastName: String) {
    	companion object { <- 비어있는 동반 객체를 선언
    	
    	}
    }
    
    //클라이언크/ 서버 통신 모듈
    fun Pserson.Companion.fromJSON(json: String): Person{ <- 확장 함수를 선언
    	/* ... */
    }
    
    val p = Person.fromJSON(json)
    ```
  > 동반 객체에 대한 확장 함수를 작성하려면 원래 클래스에 동반 객체를 꼭 선언해야함.


### 객체 식

- 자바의 익명 내부 클래스처럼 **즉석에서 객체를 생성**할 수 있음.

    ```kotlin
    interface MouseListener {
    	fun onEnter()
    	fun onClick()
    }
    
    class Button(private val listener: MouseListener) { /* ... */ }
    ```

- 이름 없이 객체를 정의하고 바로 인스턴스를 생성할 수 있음.

    ```kotlin
    fun main() {
    	Button(object: MouseListener {
    		override fun onEnter() { /* ... */ }
    		override fun onClick() { /* ... */ }
    	})
    }
    ```

- 또는 직접 인자로 전달할 수도 있음.

    ```kotlin
    val listener = object : MouseListener() {
        override fun onEnter() { /* ... */ }
    		override fun onClick() { /* ... */ }
    }
    ```

- 객체 식은 주변 변수에 접근 가능

    ```kotlin
    fun countClicks(window: Window) {
        var clickCount = 0
        window.addMouseListener(object : MouseAdapter() {
            override fun mouseClicked(e: MouseEvent) {
                clickCount++
            }
        })
    }
    ```
  > 💡 객체 식은 일회성 로직을 구현할 때, 그리고 다중 메서드를 오버라이드할 필요가 있을 때 특히 유용!

## 4.5 부가 비용 없이 타입 안전성 추가: 인라인 클래스

- 함수 인자나 반환 타입이 단순 타입(Int, String 등)일 때, 의미를 명확히 하기 위해 클래스로 감싸는 것이 좋지만 일반 클래스는 성능 손실이 생김.
- **인라인 클래스**는 성능 손실 없이 타입 안전성과 의미 명확성을 제공하기 위한 기능이다.

  ```kotlin
  class UsdCent(val amount: Int)
  
  fun addExpense(expense: UsdCent) {
      // 비용을 미국 달러의 센트 단위로 저장
  }
  
  fun main() {
      addExpense(UsdCent(147))
  }
  ```

- 위처럼 클래스를 사용하면 의미 명확성은 생기지만 성능 저하가 생길 수 있다.
- 인라인 클래스를 사용하면 객체를 감싸지만 **컴파일 시점에 실제 클래스가 사라지고 값만 전달**되므로 런타임 오버헤드가 없다.

  ```kotlin
  @JvmInline
  value class UsdCent(val amount: Int)
  ```

- 프로퍼티는 하나만 가질 수 있으며, 생성자에 바로 정의해야 함.
- 인터페이스 구현도 가능하다.

  ```kotlin
  interface PrettyPrintable {
   fun prettyPrint()
  }
  
  @JvmInline
  value class UsdCent(val amount: Int): PrettyPrintable {
      val salesTax get() = amount * 0.06
      override fun prettyPrint() = println("${amount}")
  }
  
  fun main() {
      val expense = UsdCent(1_99)
      println(expense.salesTax)
      // 11.94
      expense.prettyPrint()
      // 199
  }
  ```
  
  > 💡 인라인 클래스는 단위를 구분하거나 다른 의미를 가진 값을 명확히 표현하고 싶을 때 사용하면 유용하다.

### 인라인 클래스와 프로젝트 발할라

- 현재 인라인 클래스는 코틀린 컴파일러의 특성이다. JVM 차원에서는 정식으로 지원되지 않기 때문에 `@JvmInline`을 사용해야 한다.
- 향후 자바 JDK 개선 프로젝트인 **Project Valhalla**가 도입되면 JVM 자체에서도 인라인 타입을 공식 지원할 예정.
- 그에 따라 `@JvmInline`을 명시하지 않아도 JVM에서 직접 인라인 클래스를 최적화할 수 있게 될 것이다.

## 요약 정리

- 코틀린의 인터페이스는 자바 인터페이스와 유사하지만 디폴트 구현과 프로퍼티도 포함 가능
- 모든 코틀린 선언은 기본적으로 `final`, `public`
- 상속 가능하게 만들려면 `open`을 명시해야 함
- `internal`은 같은 모듈 안에서만 접근 가능
- 중첩 클래스는 기본적으로 static, 바깥 클래스를 참조하려면 `inner`로 명시
- `sealed` 클래스는 하위 클래스가 모두 컴파일 시점에 결정됨 → when 분기에서 else 생략 가능
- `field` 키워드를 통해 프로퍼티의 backing field에 접근 가능
- `data class`는 `equals`, `hashCode`, `toString`, `copy` 자동 생성
- 클래스 위임은 인터페이스를 구현하는 반복 코드를 줄여줌
- `object`는 싱글턴을 선언하는 방법이며, 동반 객체로 팩토리 메서드 구현에도 유용함
- `object expression`은 익명 객체를 선언해 일회성 구현에 활용됨
- **인라인 클래스**는 성능 저하 없이 명확한 타입 구분이 필요한 경우 유용함