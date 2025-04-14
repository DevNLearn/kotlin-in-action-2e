# 3. 함수 정의와 호출

### 컬렉션 만들기

```kotlin
val set = setOf(1, 7, 53)
val list = listOf(1, 7, 53)
val map = mapOf(1 to "one", 7 to "seven", 53 to "fifty-three")
```

- 코틀린의 컬렉션은 기본적으로 `불변(immutable)`이다.
- 자바와 다르게 여러 편의 메서드들이 확장함수로 구현되어 있다.

### 이름 붙인 인자

- 코틀린은 이름 붙인 인자를 지원한다.
- 메서드에 전달할 인자가 많아 지는 경우, 어떤 값을 넣어야하는지 파악하기 굉장히 힘들어지는데, <br>
    이름 붙인 인자를 사용하면 어떤 인자에 어떤 값을 전달하는지 명확하게 알 수 있다.


```kotlin
// 사용전
joinToString(collection, sperator, prefix, postfix)

// 사용후
joinToString(
    collection = collection,
    separator = separator,
    prefix = prefix,
    postfix = postfix
)
```

### 디폴트 파라미터 값

- 자바에서는 메서드나 클래스에 파라미터 값이 지정되지 않은 경우, `null` 체크 등의 로직을 추가해 `default` 값을 지정해준다.
- 하지만, 코틀린에서는 디폴트 파라미터 값을 지정할 수 있다.

```kotlin
fun <T> joinToString(
    collection: Collection<T>,
    sparator: String = ", ",
    prefix: String = "",
    postfix: String = ""
): String
```

- 위와 같이 메서드에 디폴트 파라미터 값을 지정하면, 메서드를 호출할 때 인자를 생략할 수 있다.
- 생략된 경우 디폴트 파라미터 값이 들어가게 된다.

```kotlin
fun main() {
    joinToString(
        collection = listOf(1, 2, 3),
        postfix = ";",
        prefix = "# "
    )
    // sparator 는 지정되지 않았으므로, 기본값인 ", "이 들어간다.
}
```

### 최상위 함수와 프로퍼티

- 코틀린에서는 최상위 함수와 프로퍼티를 지원한다.
- 자바에서는 클래스 안에 메서드와 프로퍼티를 정의해야 했지만, 코틀린에서는 클래스 밖에서도 정의할 수 있다.
- 이렇게 정의된 최상위 함수와 프로퍼티는 자바에서 static 메서드와 static 프로퍼티로 사용된다.

```kotlin
/* 코틀린 */
// join.kt
fun joinToString(): String { ... }

/* 자바 */
// join.kt
public class JoinKt {
    public static String joinToString() { ... }
}
```

- 위와 같이 최상위 함수를 정의하면, 자바에서는 `joinKt.joinToString()`으로 사용된다.


```kotlin
const val PI = 3.14

fun main() {
    val radius = 2.5
    println(radius * radius * PI)
}
```
- 최상위 프로퍼티는 상수를 정의할 때 유용하다.
- `const` 키워드를 사용하면 컴파일 타임에 상수로 변환된다.
- `const` 키워드는 `val`과 함께 사용해야 한다.

### 확장 함수

- 코틀린에서는 기존 클래스에 새로운 메서드를 추가할 수 있다.
- 이렇게 추가된 메서드는 기존 클래스의 메서드처럼 사용할 수 있다.
- 확장 함수는 정적 바인딩(static binding)으로 동작한다.
- 즉, 확장 함수는 컴파일 타임에 바인딩된다.

```kotlin
fun String.lastChar(): Char = this.get(this.length - 1)
```

- 위와 같이 확장 함수를 정의하면, 기존 클래스에 새로운 메서드를 추가할 수 있다.
- 확장 함수는 기존 클래스의 메서드처럼 사용할 수 있다.

```kotlin
fun main() {
    val str = "Kotlin"
    println(str.lastChar()) // n
}
```

- 확장 함수가 어떻게 구현되는지 자바코드로 변환해보면, 아래와 같이 변환된다.

```java
public final class StringKt {
    public static final char lastChar(@NotNull String str) {
        Intrinsics.checkNotNullParameter(str, "<this>");
        return str.charAt(str.length() - 1);
    }
}
```

- 즉, 확장 함수는 기존 클래스의 메서드를 호출하는 형태로 구현된다.
- 그렇기 때문에, 확장 함수는 기존 클래스의 `private` 메서드에 접근할 수 없다.
- 또한, 확장 함수는 기존 클래스의 메서드를 오버라이드할 수 없다.


### 확장 프로퍼티

- 코틀린에서는 기존 클래스에 새로운 프로퍼티를 추가할 수 있다.
- 이렇게 추가된 프로퍼티는 기존 클래스의 프로퍼티처럼 사용할 수 있다.
- 확장 프로퍼티도 마찬가지로 정적 바인딩(static binding)으로 동작한다.

```kotlin
val String.lastChar: Char
    get() = this.get(this.length - 1)
```

- 확장 프로퍼티도 마찬가지로 자바코드로 변환해보면, 아래와 같이 변환된다.

```java
public final class StringKt {
   public static final char getLastChar(@NotNull String str) {
      Intrinsics.checkNotNullParameter(str, "<this>");
      return str.charAt(str.length() - 1);
   }
}
```

- 확장 프로퍼티는 클래스 내부에 프로퍼티가 실제로 추가되는 것이 아니다.
- `getLastChar()` 메서드가 추가되는 것이다.

< 궁금해서 해본 거 >

```kotlin
val String.lastChar: Char
    get() = this.get(this.length - 1)

fun String.lastChar(): Char {
    return this.get(this.length - 1)
}
```

이걸 변환하면 어떻게될까??

```java
public final class StringKt {
   public static final char getLastChar(@NotNull String str) {
      Intrinsics.checkNotNullParameter(str, "<this>");
      return str.charAt(str.length() - 1);
   }

   public static final char lastChar(@NotNull String str) {
      Intrinsics.checkNotNullParameter(str, "<this>");
      return str.charAt(str.length() - 1);
   }
}
```

---

### 가변 인자 함수

- 코틀린에서도 가변 인자 함수를 지원한다.
- 자바에서는 `String...` 형태로 가변 인자 함수를 정의했지만, 코틀린에서는 `vararg` 키워드를 사용한다.
- 스프레드 연산자 `*`를 사용하면, 배열을 가변 인자로 전달할 수 있다.

```kotlin
/*
fun listOf<T>(vararg values: T): List<T> {}
*/

fun main() {
    val list: Array<Int> = arrayOf(1, 2, 3, 4, 5)
    println(listOf(6, 7, *list))
}
// [6, 7, 1, 2, 3, 4, 5]

```

### 3중 따옴표 문자열

- 코틀린에서는 3중 따옴표 문자열을 지원한다.
- 3중 따옴표 문자열은 여러 줄에 걸쳐서 문자열을 작성할 수 있다.

```kotlin
val text = """
    |Tell me and I forget.
    |Teach me and I remember.
    |Involve me and I learn.
""".trimIndent()

fun main() {
    println(text)
}
/*
|Tell me and I forget.
|Teach me and I remember.
|Involve me and I learn.
 */
```

### 로컬 함수


```kotlin
class User(val id: Int, val name: String, val address: String)

fun saveUser(user: User) {

    fun validate(user: User, value: String, fieldName: String) {
        if (value.isEmpty()) {
            throw IllegalArgumentException("$fieldName must not be empty")
        }
    }

    validate(user, user.name, "Name")
    validate(user, user.address, "Address")
}

fun main() {
    saveUser(user = User(1, "", "New York"))
}

// Exception in thread "main" java.lang.IllegalArgumentException: Name must not be empty
```

- 로컬 함수는 메서드 안에 정의된 함수를 의미한다.
- 메서드 안에서만 사용할 수 있고, 가독성을 높이는 데 유용하다.

> 이런 형태가 괜찮은 구조인지 개인적으로 궁금하다. 🤔

