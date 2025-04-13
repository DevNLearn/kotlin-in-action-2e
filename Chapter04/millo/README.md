
# 클래스, 객체, 인터페이스
## 클래스 계층 정의
### 코틀린 인터페이스
- 인터페이스 안에는 아무런 상태도 들어갈 수 없지만 추상 메서드, 구현이 있는 메서드가 정의될 수 있다
- 인터페이스의 추상 메서드는 하위 클래스(비추상 클래스)에서 반드시 `override` 키워드로 구현해야 한다
- 상속이나 구성 시 클래스 이름 뒤에 `:`를 붙이고 인터페이스 이름을 적어주면 된다
  - 인터페이스는 클래스와 다르게 원하는 만큼 개수 제한 없이 구현할 수 있다
  - 클래스는 하나만 확장 가능하다

**인터페이스 프로퍼티**
- 프로퍼티를 선언할 수는 있지만(추상 프로퍼티) 이를 위한 백킹 필드는 가질 수 없다
- 프로퍼티에 대한 커스텀 게터를 제공해(확장 프로퍼티) 사용할 수 있다
```kotlin
interface MyInterface {
    val prop: String // 추상 프로퍼티
    
    val computedProperty: String // 커스텀 게터가 있는 확장 프로퍼티
        get() = "default implementation"
}
```

**override 변경자**
- 인터페이스의 메서드나 프로퍼티를 구현할 때 `override` 키워드를 사용해야 한다
- 실수로 상위 클래스의 메서드를 오버라이드하는 경우를 방지하기 위해 필수로 사용해야 한다
- 상위 클래스와 동일한 시그니처를 가진 메서드를 하위 클래스에서 선언하는 경우 컴파일이 안 되기 때문에 `override` 변경자를 붙여 그 상위 클래스의 메서드를 재정의하던가 시그니처를 바꿔야 한다 

**다중 구현 충돌**
- 동일한 시그니처의 메서드를 여러 인터페이스에서 상속받았을 때, 어떤 구현을 사용할지 명시해야 한다 
  - `super<인터페이스이름>.메서드()` 형태로 선택할 수 있다
```kotlin
interface Clickable {
    fun showOff() = println("I'm clickable!")
}

interface Focusable {
    fun showOff() = println("I'm focusable!")
}

class Button : Clickable, Focusable {
    override fun showOff() {
        super<Clickable>.showOff() // Clickable 인터페이스의 showOff() 메서드 호출
        super<Focusable>.showOff() // Focusable 인터페이스의 showOff() 메서드 호출
    }
}
```

**디폴트 구현**
- 인터페이스가 메서드에 대한 구체적인 디폴트 구현을 제공할 수 있다
- 하위 클래스는 이 디폴트를 사용해도 되고 오버라이드해서 사용해도 된다

### open, final, abstract 변경자
- 기본적으로 모든 클래스와 메서드가 `final`이고 `open` 키워드를 붙여야 상속이 가능하다
  - 프로퍼티도 `open` 키워드를 붙여 오버라이드 가능하도록 할 수 있다
  - 자바는 모든 클래스를 다른 클래스가 상속할 수 있어 편리하지만 기반 클래스의 변경으로 인해 예측하기 어려운 상황이 발생할 수 있는 "취약한 기반 클래스" 라는 문제가 있다
```kotlin
interface Clickable {   // 인터페이스는 기본적으로 전부 open
    fun click()
    fun showOff() = println("I'm clickable!")
}

open class RichButton : Clickable { 
    fun disable() {} // final로 상속 불가
    open fun animate() { /* ... */ }
    override fun click() { /* ... */ }
}

class ThemedButton : RichButton() {
    override fun animate() { /* ... */ } 
    override fun click() { /* ... */ } 
    override fun showOff() { /* ... */ } // 상위 클래스인 RichButton이 오버라이드하진 않았지만 가능
}
```

만약 하위 클래스가 상위 클래스의 메서드를 오버라이드하지 못하도록 막으려면 'final' 키워드를 사용하면 된다
- `override` 키워드가 붙은 오버라이드 메서드일 때에도 기본적으로 `open`이므로 상속을 막으려면 `final` 키워드를 붙여야 한다
```kotlin
open class RichButton : Clickable { 
    fun disable() {} // final로 상속 불가
    open fun animate() { /* ... */ }
    final override fun click() { /* ... */ }  // final로 상속 불가
}
```

**open 클래스와 스마트 캐스트**
- 스마트 캐스트는 `val`로 선언된 불변 변수나 타입 검사(is) 이후 값이 변경되지 않는 변수에만 자동으로 적용
- 기본적인 상속 가능 상태를 `final`로 하면 기본적으로 대부분 스마트 캐스트가 가능하다는 장점이 있다
- `open`인 프로퍼티는 상속받은 클래스에서 상위 클래스의 프로퍼티를 오버라이드하면 변경될 가능성이 있어 스마트 캐스트가 불가능하다

**`abstract` 추상 클래스**
- 구현 없이 하위 클래스에서 오버라이드해야만 하는 추상 멤버만 존재해 인스턴스화 할 수 없는 클래스
- 항상 `open`이므로 `open` 키워드를 붙일 필요가 없다
```kotlin
abstract class Animated {   // 인스턴스화 불가능
    abstract val animationSpeed: Double // 추상 프로퍼티로 반드시 하위 클래스가 값이나 접근자 구현
    val keyframes: Int = 20 // 추상이 아닌 프로퍼티는 기본적으로 닫혀있고, `open`으로 열어줄 수 있다
    open val frames: Int = 60
  
    abstract fun animate() // 추상 메서드로 반드시 하위 클래스가 구현
    open fun stopAnimating() { /* ... */ } // 추상이 아닌 함수는 기본적으로 닫혀있고, `open`으로 열어줄 수 있다
    fun animateTwice() { /* ... */ }
}
```

**인터페이스의 경우는?**
- 인터페이스 멤버의 경우 `final`, `open`, `abstract` 키워드를 사용하지 않는다.
- 항상 `open`이고 멤버에 본문이 없으면 자동으로 `abstract` 추상 멤버가 되기 때문에 키워드들을 명시할 필요가 없는 것

### 가시성 변경자
- 가시성 변경자는 클래스, 메서드, 프로퍼티 등 선언에 대한 클래스 외부 접근을 제어한다
- 클래스의 내부 구현을 변경하더라도 그 클래스를 의존하는 외부 코드를 깨지 않고 유지할 수 있다

**Java의 접근 제어자**

| public | 모든 곳에서 접근 가능 |
| --- | --- |
| protected | 같은 패키지 또는 하위 클래스에서만 접근 가능 |
| default | 같은 패키지에서만 접근 가능 |
| private | 선언된 클래스 내에서만 접근 가능 |

**Kotlin의 접근 제어자**

| public | Java와 동일 |
| --- | --- |
| protected | **선언된 클래스 또는 하위 클래스에서만 접근 가능** |
| internal | 같은 모듈에서만 접근 가능 |
| private | Java와 동일 |
- Kotlin에서는 기본적으로 패키지라는 개념을 namespace를 관리하기 위한 용도로만 사용하고 가시성 제어를 위해서는 사용하지 않는다
- Java에서는 아무것도 안 적어주면 기본 접근 제어자로 `default`가 설정되었다면 Kotlin은 기본 접근 제어자로 `public`을 설정한다 
- `internal`은 같은 모듈에서만 접근 가능하다는 뜻으로, 모듈은 Gradle, Maven, Ant 등으로 빌드할 때의 단위로 생각하면 된다
  - 즉, 같은 모듈에 속한 클래스는 서로 접근할 수 있지만 다른 모듈에 속한 클래스는 접근할 수 없다
  - 하위 모듈에 internal이 적용된 것들은 상위 모듈에서 가져다 쓰지 못한다

확장 함수가 그 원래의 클래스의 `private`나 `protected` 멤버에 접근할 수 없다고 했었다
- `protected`는 선언된 클래스나 그 하위 클래스에서만 접근 가능하기 때문

**Kotlin 파일에서의 접근 제어자**

| public | 기본값. 어디에서든 접근 가능 |
| --- | --- |
| internal | 같은 모듈에서만 접근 가능 |
| private | 같은 파일 내에서만 접근 가능 |
- protected는 선언된 클래스와 하위 클래스에서 사용할 수 있도록 하는 접근 제어자인데 애초에 클래스가 아닌 파일이기 때문에 사용 불가능하다


### Java의 내부(`inner`) 클래스
아래의 자바 코드를 직렬화해보면 `NotSerializableException: Button` 이 발생하는데 왜 그럴까?
```java
interface State implements Serializable {}
interface View {
  State getCurrentState();
  void restoreState(State state);
}

public class Button implements View {
    @Override
    public State getCurrentState() {
        return new ButtonState();
    }

    @Override
    public void restoreState(State state) { /* ... */ }

    // 내부 클래스로 Button 클래스의 상태를 저장하는 클래스 선언
    public class ButtonState implements State { /* ... */
    }
}
```
그 이유는 내부 클래스가 암묵적으로 외부 클래스의 인스턴스에 대한 참조를 가지기 때문이다.
- 클래스를 직렬화할 때는 해당 클래스의 모든 멤버가 직렬화 가능해야 한다
  - ButtonState가 State를 구현했고, State는 Serializable을 구현한다
  - 하지만 내부 클래스 구조로 ButtonState는 Button의 인스턴스를 참조하게 되고, Button은 Serializable을 구현하지 않았기 때문에 직렬화할 수 없는 것이다

**해결방법**
내부 클래스를 `static`으로 선언하면 정적 내포 클래스로 변경되어 외부 클래스의 인스턴스에 대한 참조가 사라지기 때문에 직렬화가 가능해진다
```java
public static class ButtonState implements State { /* ... */ }
```

### Kotlin의 내포 클래스
코틀린에서 내포된 클래스에 아무 변경자도 없으면 자바의 static 내포 클래스와 동일하게 동작한다
- 즉, 외부 클래스의 인스턴스에 대한 참조를 가지지 않는다
```kotlin
class Button: View {
    override fun getCurrentState(): State { /* ... */ }
    override fun restoreState(state: State) { /* ... */ }
    class ButtonState: State { /* ... */ }  // 내포 클래스
}
```
- 내부 클래스로 만드려면 명시적으로 `inner` 키워드를 적어줘야 한다
  - 이 때 외부 클래스 참조에 접근하려면 `this@OuterClassName` 형태로 접근할 수 있다

### Sealed 클래스: 봉인된 클래스
- 상위 클래스에 `sealed` 변경자를 붙이면 그 하위 클래스의 가능성을 제한할 수 있다
  1. 반드시 컴파일 시점에 알려져야 한다
  2. 반드시 상위의 sealed 클래스가 정의된 패키지와 같은 패키지에 속해야 한다
     - (1.1 버전 이전에는 반드시 같은 파일에 있어야 했었다)
  3. 반드시 하위 클래스들이 같은 모듈 안에 위치해야 한다
- 추상 클래스와 비교했을 때 더 엄격하게 클래스 계층의 확장을 관리할 수 있다
- sealed 클래스는 `when` 표현식과 함께 사용하면 유용하다
  - `when` 표현식에서 `else` 분기를 작성할 필요가 없다
  - 모든 하위 클래스에 대해 분기 처리를 해주기 때문에 컴파일러가 모든 경우를 처리했는지 체크해준다
```kotlin
sealed class Expr // 봉인된 클래스
class Num(val value: Int) : Expr() // 하위 클래스
class Sum(val left: Expr, val right: Expr) : Expr() // 하위 클래스
class Mul(val left: Expr, val right: Expr) : Expr() // 하위 클래스

fun eval(e: Expr): Int = when (e) {
    is Num -> e.value
    is Sum -> eval(e.left) + eval(e.right)
    is Mul -> eval(e.left) * eval(e.right)
  }
```

**`Sealed` 인터페이스도 가능하다**
- 마찬가지로 `when` 처리 시 모든 구현을 분기해주면 `else` 분기를 작성할 필요가 없다
```kotlin
sealed interface Toggleable {
    fun toggle()
}

class Switch: Toggleable {
    override fun toggle() = println("Switch toggled")
}

class CheckBox: Toggleable {
    override fun toggle() = println("CheckBox toggled")
}
```

## 클래스 선언
### 클래스 초기화: 주 생성자
- 클래스 이름 뒤에 오는 괄호로 둘러싸인 부분으로 2가지 목적이 있다
  1. 생성자 파라미터 지정
  2. 생성자 파라미터에 의해 초기화되는 프로퍼티 정의

```kotlin
class User constructor(_nickname: String) {
  val nickname: String
  init { nickname = _nickname }
}
```
- `constructor` 키워드로 생성자 정의를 시작하고 `init` 블록으로 초기화 코드를 작성한다
- nickname 프로퍼티를 초기화하는 코드를 nickname 프로퍼티 선언에 포함시킬 수 있어 초기화 블록이 필요없고, 별다른 어노테이션이나 가시성 변경자가 없으므로 `constructor` 키워드도 생략할 수 있게 된다
```kotlin
class User(_nickname: String) {
  val nickname = _nickname  // 생성자 파라미터를 프로퍼티로 초기화
}
```

이 때 주 생성자의 파라미터 이름 앞에 `val`이나 `var`를 붙이면 자동으로 프로퍼티가 생성되므로 프로퍼티 정의와 초기화를 한 번에 할 수 있는 것이다
```kotlin
class User(val nickname: String)
```

**주 생성자 기본값 지정 가능**
- 함수형 파라미터처럼 기본값을 지정해줄 수 있다
- 모든 생성자 파라미터에 기본값을 지정하면 컴파일러가 자동으로 파라미터가 없는 생성자를 만들어준다

**하위 클래스는 상위 클래스의 생성자를 반드시 호출해야 한다**
- 아무 인자도 없는 상위 클래스라 할지라도 `클래스명()`으로 생성자를 호출해줘야 한다
  - 컴파일러가 아무 인자도 없는 클래스의 디폴트 생성자를 자동으로 만들어준다
- 인터페이스는 생성자가 없기 때문엔 인터페이스명만 적어줬던 점을 생각하자

**인스턴스화를 막고 싶다면 `private constructor`로 주 생성자를 숨기자**
- 주 생성자가 비공개면 외부에서 인스턴스를 만들 수 없다
- 보통 주 생성자를 비공개로 두고 동반 객체를 활용해 인스턴스를 생성하는 방법을 많이 사용한다
- 특히 자바의 경우 유틸리티 클래스나 싱글톤 클래스에서 어쩔 수 없이 `private` 생성자를 사용해 다른 곳에서 인스턴스화 못하도록 막는데 코틀린은 그냥 최상위 함수를 사용하거나 `object` 키워드로 싱글턴을 만들면 된다
```kotlin
class User private constructor(val nickname: String) {
  companion object { // 동반 객체
    fun newSubscribingUser(email: String) = User(email.substringBefore('@'))
  }
}
```

### 클래스 초기화: 부 생성자
대부분 주 생성자의 디폴트 파라미터 값과 이름 붙은 인자로 해결 가능하지만 가끔 다양한 생성자를 지원해야 하는 경우 부 생성자를 활용할 수 있다
```kotlin
import java.net.URI

open class DownLoader {
    constructor(url: String?) { /* ... */ }
    constructor(uri: URI?) { /* ... */ }
}

class MyDownloader : DownLoader {
    constructor(url: String?) : this(url?.let { URI(it) })
    constructor(uri: URI?) : super(uri)
}
```
- 부 생성자는 `constructor` 키워드로 시작하고, 주 생성자와 마찬가지로 초기화 블록을 가질 수 있다
- 부 생성자는 다른 생성자를 `this()`로 호출해 위임할 수 있고, `super()`로 상위 클래스의 생성자를 호출해 위임할 수 있다
- 클래스에 주 생성자가 없다면 반드시 모든 부 생생성자는 다른 생성자에게 위임하거나 상위 클래스에 위임해야 하고, 결국 마지막 위임을 따라갔을 때 상위 클래스의 생성자에 도달해야 한다

### 인터페이스에 선언된 프로퍼티 구현
- 인터페이스는 아무 상태도 포함할 수 없기에 추상 프로퍼티는 백킹 필드나 게터 정보가 없다
  - 상태를 저장할 필요가 있으면 하위 클래스에서 상태 저장을 위한 프로퍼티를 만들어야 한다
```kotlin
interface User {
    val nickname: String    // 추상 프로퍼티
}

class PrivateUser(
  override val nickname: String // override 키워드로 인터페이스의 프로퍼티 구현
) : User

class SubscribingUser(
  val email: String
) : User {
    override val nickname: String
        get() = email.substringBefore('@')  // 커스텀 게터로 프로퍼티 구현
}

class SocialUser(
    val accountId: Int
) : User {
    override val nickname: String = accountId.toString() // 프로퍼티 초기화로 구현
}
```
- 방법 1) 인터페이스의 프로퍼티를 `override` 키워드로 구현
- 방법 2) 매번 커스텀 게터로 계산해서 가져오는 방법 (상태 저장 X)
- 방법 3) 프로퍼티 초기화로 상태를 백킹 필드에 저장했다가 불러오는 방법

**함수 대신 확장 프로퍼티를 사용하는 경우**
- 예외를 던지지 않을 때
  - 값을 검색하는 개념이므로 예외 발생을 기대하지 않고 사용해야 한다
  - 프로퍼티의 값이 항상 유효해야 한다
- 계산 비용이 적게 들어 최초 실행 후 결과를 캐시해 사용할 때
- 멱등성을 보장할 수 있을 때 
```kotlin
val String.lastChar: Char
    get() = this[length - 1] // 문자 자체가 변하지 않는 한 멱등성 보장 및 간단한 예외없는 연산
```

### 백킹(뒷받침) 필드
- 코틀린 프로퍼티의 값을 저장하는 숨겨진 필드
- 값을 저장하는 동시에 접근할 수 있게 하려면 접근자 안에서 프로퍼티를 뒷받침하는 백킹 필드에 접근할 수 있어야 한다
- `field` 키워드로 백킹 필드에 접근할 수 있는데 게터는 읽을 수만 있고, 세터는 읽거나 쓸 수 있다
```kotlin
class User {
    var name: String = "default"
        get() = field.uppercase() // 백킹 필드에 접근해 대문자로 변환
        set(value) { field = value.trim() } // 백킹 필드에 접근해 공백 제거 후 저장
}
```

**컴파일러가 백킹 필드를 생성하는 조건**
1. 디폴트 접근자(getter/setter)를 사용하는 경우
2. `field` 키워드를 사용하는 커스텀 접근자(get()/set())가 있는 경우

### 접근자의 가시성 변경
- 접근자의 가시성은 기본적으로 프로퍼티의 가시성과 같지만 더 좁은 범위로 설정할 수 있다
```kotlin
class User {
    var name: String = "default"
        private set // 세터는 private으로 설정
}
```

## 데이터 클래스
### 모든 클래스가 정의해야 하는 메서드
**`toString()`**
- 객체 출력 메서드로 기본적으로 `클래스이름@객체주소` 형태로 출력된다
- 쓸만하게 바꾸려면 객체의 필요한 프로퍼티 값을 문자열로 변환해 출력하도록 오버라이딩 해줘야 한다

**`equals()`**
- 객체 비교 메서드로 기본적으로 객체 주소를 비교하는데 상황에 따라 객체의 프로퍼티 값을 비교하도록 오버라이딩 해줘야 한다
- 자바에서는 `==`를 기본 타입과 참조 타입 비교 시 사용하는데 코틀린은 `==`를 객체 비교에 사용하고 `===`를 참조 타입 비교에 사용한다
  - 내부적으로 `==`는 `equals()` 메서드를 호출하고 `===`는 자바와 동일하게 참조 타입 비교를 한다
  - 그래서 `equals()` 메서드를 요구사항대로 오버라이드하고 `==`을 사용해 비교하면 된다

**`hashCode()`**
- `equals()`만 오버라이드하면 두 객체가 논리적으로 같더라도 서로 다른 해시 코드를 가질 수 있다
- JVM 언어에서는 `equals()`가 true를 반환하는 두 객체는 반드시 같은 해시 코드를 가져야 한다는 규정이 있으므로 `hashCode()`도 오버라이드 해줘야 한다
  - 이 규정을 지키지 않으면 해시 기반 컬렉션에서 문제가 발생할 수 있다
  - 예를 들어 HashSet에서 키로 사용되는 객체의 해시 코드가 다르면 같은 키로 인식하지 못해 값을 찾지 못할 수 있다
    - hashSet은 원소 비교 시 비용을 줄이기 위해 먼저 해시코드를 비교하고, 해시코드가 같을 때만 `equals()` 메서드를 호출한다
  ```kotlin
  fun main() {
      val processed = hashSetOf(User("Millo", 241125))
      println(processed.contains(User("Millo", 241125))) // 다른 해시코드로 false
  }
  ```
- 해시 코드는 `equals()`와 마찬가지로 프로퍼티 값을 기반으로 계산해야 한다

### 이 3가지 필수 메서드를 자동으로 생성해주는 data class
- data class는 이 `toString()`, `equals()`, `hashCode()` 메서드를 컴파일러가 자동으로 생성해준다
- `toString()`은 각 프로퍼티 선언 순서대로 `클래스명(프로퍼티=값)` 형태로 출력된다
- `equals()`와 `hashCode()`는 주 생성자에 나열된 모든 프로퍼티를 기반으로 생성된다
```kotlin
data class Customer(
    val name: String,
    val postalCode: Int,
)

fun main() {
    val c1 = Customer("Sam", 11111)
    val c2 = Customer("Mart", 22222)
    val c3 = Customer("Sam", 11111)
  
    println(c1)     // Customer(name=Sam, postalCode=11111)
    println(c1 == c2) // false
    println(c1 == c3) // true
    println(c1.hashCode()) // 11111
    println(c2.hashCode()) // 22222
}
```

**data class와 불변성: `copy()`**
- data class는 불변성을 보장하기 위해 `val`로 선언된 프로퍼티만 가지도록 권장한다
  - 물론 `var` 프로퍼티도 가능은 하다
- 특히 HashMap 등의 컨테이너에 data class 객체를 담고 이후에 변경이 되어버리면 컨테이너가 잘못될 수 있어 불변성이 필수적이다
- 또한 다중 스레드 환경에서 불변성은 스레드 안전성을 보장하기 때문에 필수적이다
- 이렇게 불변 객체로 유지하면서 더 쉽게 사용할 수 있도록 코틀린 컴파일러는 `copy()` 메서드를 자동으로 생성해준다
  - 객체를 복사하면서 일부 프로퍼티를 바꿀 수 있게 해주는 메서드로 객체를 메모리에서 직접 바꾸는 것이 아니라 복사본을 만들어준다
  - 원본과 다른 생명주기를 가지고, 원본에 영향을 전혀 미치지 않기 때문에 불변성을 보장할 수 있다
  - 실제로로 디컴파일 해보면 `copy()` 메서드는 `data class`의 모든 프로퍼티를 파라미터로 받는 생성자를 호출하는 형태로 구현되어 있다
  ```java
   @NotNull
   public final Customer copy(@NotNull String name, int postalCode) {
      Intrinsics.checkNotNullParameter(name, "name");
      return new Customer(name, postalCode);
   }
  ```

**자바 record와의 차이점**
- 둘 다 `toString()`, `equals()`, `hashCode()` 메서드를 자동으로 생성해준다
- 다만 자바 record가 data class보다 더 많은 제약이 있다
  - 모든 프로퍼티가 `private final`로 선언되어야 한다
  - 상위 클래스를 확장할 수 없다
  - 클래스 본문 안에서 다른 프로퍼티를 정의할 수 없다
- data class는 자바 record보다 더 유연하게 사용할 수 있다
  - 프로퍼티를 `var`로 선언할 수 있다
  - 상위 클래스를 확장할 수 있다
  - 클래스 본문 안에서 다른 프로퍼티를 정의할 수 있다
  - `copy()` 메서드를 제공한다

### 클래스 위임: `by` 키워드
- 구현 상속은 상위 클래스의 세부 구현 사항에 의존하게 되면서 예측하지 못한 경우가 생겨 시스템의 취약점이 될 수 있다
- 따라서 코틀린은 기본적으로 클래스를 `final`로 선언해 상속을 막고, 상속이 필요한 경우에는 `open` 키워드를 붙여 상속을 허용한다

**상속을 허용하지 않는 클래스에 새로 로직을 추가하고 싶은 경우** 

방법 1) 데코레이터 패턴 적용
```kotlin
class DelegatingCollection<T> : Collection<T> {
    private val innerList = arrayListOf<T>()
  
    override val size: Int
        get() = innerList.size
  override fun isEmpty(): Boolean = innerList.isEmpty()
  override fun contains(element: T): Boolean = innerList.contains(element)
  override fun iterator(): Iterator<T> = innerList.iterator()
  override fun containsAll(elements: Collection<T>): Boolean = innerList.containsAll(elements)
}
```
- 대신 사용할 수 있는 새로운 클래스(데코레이터) 생성
- 기존 클래스와 같은 인터페이스를 데코레이터가 제공하고 기존 클래스를 테코레이터 내부로 유지한다
- 기존 클래스의 메서드를 오버라이드해 새로운 로직을 추가
  - 기존 기능이 그대로 필요한 부분은 데코레이터의 메서드가 기존 클래스의 메서드에게 요청을 전달

방법 2) 코틀린 언어가 일급 시민 기능으로 지원하는 위임
```kotlin
class DelegatingCollection<T>(
    innerList: Collection<T> = arrayListOf()
) : Collection<T> by innerList { // Collection<T> 인터페이스를 위임
    // 추가적인 로직 작성
}
```
- `by` 키워드를 사용해 위임할 인터페이스를 지정하고, 위임할 객체를 생성자 파라미터로 받는다
- 이렇게 하면 `Collection<T>` 인터페이스의 모든 메서드가 `innerList` 객체에게 위임된다
- 위임되면 컴파일러가 자동으로 `Collection<T>` 인터페이스의 메서드를 오버라이드해준다

**클래스 위임 예제**
- 원소 추가 횟수를 기록하는 컬렉션
- 원소 개수 카운팅만 오버라이드하고 나머지는 내부 컨테이너(innerSet)에게 위임한다

```kotlin
class CountingSet<T>(
  private val innerSet: MutableCollection<T> = hashSetOf(),
) : MutableCollection<T> by innerSet {
  var objectsAdded = 0

  // 위임하지 않고 새로 구현할 메서드는 override
  override fun add(element: T): Boolean {
    objectsAdded++
    return innerSet.add(element)
  }

  override fun remove(element: T): Boolean {
    objectsAdded--
    return innerSet.remove(element)
  }
}

fun main() {
  val cset = CountingSet<Int>()
  cset.addAll(listOf(1, 1, 2))
  println("Added ${cset.objectsAdded} objects, ${cset.size} uniques.")
  // Added 3 objects, 2 uniques.
}
```

### `object` 키워드: 클래스 선언과 인스턴스 언언을 한꺼번에?!

