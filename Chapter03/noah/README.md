# 함수 정의와 호출
## 1. 코틀린에서 컬렉션

코틀린은 표준 자바 컬렉션 클래스를 사용한다. 코틀린 컬렉션 인터페이스가 디폴트로 읽기 전용이다.

```kotlin
val set = setOf(1, 7, 53) //집합
val list = listOf(1, 7, 53) //리스트
val map = listOf(1 to "one", 7 to "seven", 53 to "fifty-three") //맵
```

> `to` 는 키워드가 아니라 일반 함수이다.
>

코틀린 컬렉션은자바 컬렉션과 똑같은 클래스이긴 하지만 더 많은 기능을 쓸 수 있다

```kotlin
fun main() {
    val strings = listOf("first", "second", "fourteenth")
    string.last() // 마지막 원소 가져오기
    strings.shuffled() // 운소 섞기
    val numbers = setOf(1, 14 ,2)
    numbers.sum() // 컬렉션 합계 계산
}
```

## 2. 함수 호출

```kotlin
fun <T> joinToString(
    collection: Collection<T>,
    separator: String,
    prefix: String,
    postfix: String
): String {
}
```

### 이름 붙인 인자

코틀린으로 작성한 함수를 호출할 때는 함수에 전달하는 인자의 이름을 명시할 수 있다. 전달하는 모든 인자의 이름을 지정하는 경우 인자 순서를 변경할 수도 있다.

```kotlin
joinToString(
    postfix = ".",
    separator = " ",
    collection = collection,
    prefix = " "
)
```

### 디폴트 파라미터 값

코틀린에서는 파라미터에 기본 값을 지정하여 함수를 호출할 때 모든 인자를 쓸 수 도 있고, 일부를 생략할 수도 있다.

```kotlin
fun <T> joinToString(
    collection: Collection<T>,
    separator: String = ", ",
    prefix: String = "",
    postfix: String = ""
): String {
}

fun main() {
    joinToString(list, ", ", "", "")
    joinToString(list)
    joinToString(list, "; ")
}
```

### 최상위 함수와 프로퍼티

함수를 직접 소스 파일의 최상위 수준, 모든 다른 클래스의 밖에 위치하여 `최상위 함수`를 선언할 수있다.

> Java의 static 멤버과 비슷하다.
>

```kotlin
package strings
fun joinToString(/* ... */ ): String{}
```

`join.kt` 라는 파일에 위와 같이 선언 시 JVM언에서 컴파일된다면 아래와 같이 변환한다.

```kotlin
package strings
public class JoinKt { // 파일 이름에 해당하는 클래스 생성
    public static String joinToString(/* ... */) {}
}
```

함수와 마찬가지로 프로퍼티도 파일 최상위 수준에 둬서 `최상위 프로퍼티`를 선언할 수 있다.

```kotlin
var opCount = 0 // 최상위 프로퍼티 선언
fun performOperation() {
    opCount++ // 최상위 프로퍼티 값 변경
    ...
}
fun reportOperationCount() {
    println("$opCount"). // 최상위 프로퍼티의 값을 읽는다.
}
```

상수 선언으로 사용하고 싶으면 `const` 변경자를 추가하면된다.

```kotlin
const val UNIX_LINE_SEPARATOR = "\n"
```

## 3. 확장 함수와 확장 프로퍼티

`확장 함수`는 클래스 밖에 선언된 함수로 메서드를 다른 클래스에 추가한 함수이다. `확장 함수`는 클래스의 일부가 아니기 때문에 오버라이드를 할 수 없다.

- 수신 객체 타입: 추가 함수 이름 앞에  그 함수가 확장할 클래스의 이름
- 수신 객체: 확장 함수 호출 시 호출 하는 대상 값

```kotlin
package strings

// String : 수신 객체 타입, this: 수신 객체
fun String.lastChar(): Char = this.get(this.length - 1)

// this는 생략 가능 (수신 객체 멤버를 this 없이 접근 가능)
fun String.lastChar(): Char = get(length - 1)
```

`확장 함수` 내부에서는 수신 객체의 메서드나 프로퍼티를 바로 사용할 수 있다.

### 임포트와 확장 함수

`확장 함수`를 사용하려면 다른 클래스나 함수와 마찬가지로 임포트를 해야한다.

```kotlin
import strings.lastChar
val c = "Kotiln".lastChar()
```

`as` 키워드를 사용하여 다른 이름으로 사용하여 이름 충동을 막을 수 있다.

```kotlin
import strings.lastChar as last
val c = "Kotlin".last()
```

### 확장 함수로 유틸리티 함수 정의

```kotlin
fun <T> Collection<T>.joinToString( //확장 함수 선언
    separator: String = ", ",
    prefix: String = "",
    postfix: String = ""
): String {
...
}

fun Collection<String>.join(
    separator: String = ", ",
    prefix: String = "",
    postfix: String = ""
) = joinToString(separator, prefix, postfix)
```

### 확장 프로퍼티

프로퍼티라는 이름으로 불리지만 상태를 저장할 수 없어 확장 프로퍼티는 커스텀 접급자를 정의한다.

```kotlin
val String.lastChar: Char
    get() = get(length - 1)
    set(value: Char) {
        this.setCharAt(length - 1, value)
    } 
```

## 4. 컬렉션 처리

### 가변인자

`vararg` 키워드를 사용하여 호출 시 인자 개수가 달라질 수 있는 함수를 정의할 수 있다.

```kotlin
fun listOf<T>(vararg values: T): List<T> {}
```

`스프레드 연산자` 를 통해 배열을 펼쳐 listOf 함수의 파라미터로 사용할 수 있다.

```kotlin
fun main(args: Array<String>) {
    val list = listOf("args: ", *args)
}
```

### 중위 호출

맵을 만들려면 mapOf 함수를  사용한다.

```kotlin
val map = mapOf(1 to "one", 7 to "seven", 53 to "fifty-three")
```

여기서 `to` 는 `중위 호출` 이라는 방식으로 `to` 라는 일반 메서드를 호출한 것이다. 중위 호출 시에는 수신 객체 뒤에 메서드 이름을 위치시키고 메서드 인자를 넣는다.

- 인자가 하나뿐인 일반 메서드나 확장 함수에만 중위 호출 사용 가능
- 함수를 중위 호출에 사용하려면 `infix` 변경자를 함수 앞에 선언

```kotlin
infix fun Any.to(other: Any) = Pair(this, other)

1.to("one")
1 to "one"
```

### 구조 분해 선언(Destructuring Declaration)

구조 분해 선언을 통해 객체의 프로퍼티를 여러 변수로 대입할 수 있다.

```kotlin
val (number, name) = 1 to "one"

for ((index, element) in collection.withIndex()) {
}
```

## 5. 문자열 다루기

### 문자열 나누기

`split` 메서드를 통해 문자열을 분리할 수 있다. 코틀린의 `split` 메서드에서 정규식은 Regex 타입으로 받는다.

```kotlin
fun main() {
    val text = "12.345-6.A"
    println(text.split('.')) // [12, 345-6, A]
    println(text.split("\\.|-".toRegex())) // [12, 345, 6, A]
}
```

### 정규식과 3중 따옴표로 묶은 문자열

```kotlin
fun parserPathRegex(path: String) {
   val regex = """(.+)/(.+)\.(.+)""".toRegex()
   val matchResult = regex.matchEntrie(path)
   val (direcctory, filename, extension) = matchResult.destructured
}
```

3중 따옴표를 사용하면 줄 바꿈을 포함한 문자열을 사용할 수 있다.

```kotlin
fun main() {
    val text = """
         가나다
         라마사
    """.trimIndent()
}
```

## 6. 로컬 함수와 확장

```kotlin
class User(val id: Int, val name: String, val address: String)

fun saveUser(user: User) {
    if (user.name.isEmpty()){
        throw IllegalArgumentException(...)
    }
    if (user.address.isEmpty()) {
        throw IllegalArgumentException(...)
    }
}
```

### 로컬 함수를 사용해 코드 중복 줄이기

로컬 함수는 자신이 속한 바깥 함수의 모든 파라미터와 변수를 사용할 수 있다.

```kotlin
class User(val id: Int, val name: String, val address: String)
fun saveUser(user: User) {
    fun validate(value: String, fieldName: String) {
        if(value.isEmpty()) {
            throw IllegalArgumentException(...)
        }
    }
    
    validate(user.name, "Name")
    validate(user.address, "Address")
}
```

### 확장함수로 추출

```kotlin
class User(val id: Int, val name: String, val address: String)
fun User.validateBeforeSave() {
    fun validate(value: String, fieldName: String) {
        if(value.isEmpty()) {
            throw IllegalArgumentException(...)
        }
    }
}

fun saveUser(user: User) {
    user.validateBeforeSave()
}
```