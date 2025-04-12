# 함수 정의와 호출

## 컬렉션

**Set(집합)**
- 원소의 순서와 관계없이 중복을 허용하지 않는 컬렉션
- 포함된 원소들이 같으면 순서와 관계없이 두 집합은 같다
- `val set = setOf(1, 2, 3)`

**List**
- 순서를 유지해 데이터를 저장하는 컬렉션
- 인덱스를 이용해 원소 접근 가능


**Map**
- 키와 값의 쌍으로 이루어진 컬렉션으로 키는 중복을 허용하지 않지만 값은 중복을 허용
- `val map = mapOf("key1" to "value1", "key2" to "value2")`
    - 여기서 `to`는 코틀린이 제공하는 키워드가 아닌 일반 확장 함수

**코틀린은 표준 자바 컬렉션 클래스를 사용**
```kotlin
fun main() {
    val set = setOf(1, 2, 3)
    val list = listOf(1, 2, 3) 
    val map = mapOf("key1" to "value1", "key2" to "value2")
  
    println(set.javaClass)      // java.util.LinkedHashSet
    println(list.javaClass)     // java.util.Arrays$ArrayList
    println(map.javaClass)      // java.util.LinkedHashMap
}
```
- 자바에서 코틀린, 코틀린에서 자바 호출 시 컬렉션을 서로 변환할 필요가 없다
- 자바와 다른 점은 코틀린 컬렉션 인터페이스가 디폴트로 읽기 전용(불변)이라는 점
- 자바보다 더 많은 기능을 제공
  - ex) `last()`: 리스트의 마지막 원소 획득
  - ex) `first()`: 리스트의 첫 번째 원소 획득
  - ex) `shuffled()`: 리스트의 원소를 랜덤하게 섞은 새로운 리스트 반환
  - ex) `sorted()`: 리스트의 원소를 정렬한 새로운 리스트 반환
  - ex) `sum()`: 리스트의 원소를 모두 더한 값 반환
  - ...

## 컬렉션의 커스텀 출력 함수 `joinTostring()` 리팩토링하기
**요구사항**
- 자바 컬렉션의 디폴트 `toString()`은 출력 형식이 고정되어 있는데 커스텀한 형식으로 출력하고 싶다
- 구분자, 접두사, 접미사를 지정할 수 있어야 한다
- 제네릭을 사용해 어떤 타입의 컬렉션도 지원해야 한다

> 사실 이 역할의 함수를 코틀린 표준 라이브러리에서 제공하고 있지만 직접 구현해보자.

> 자바의 경우 구아바나 아파치 커먼즈같은 라이브러리를 사용해야 한다

### 초기 `joinString()`
```kotlin
fun <T> joinToString(
    collection: Collection<T>,
    separator: String,
    prefix: String,
    postfix: String,
): String {
    val result = StringBuilder(prefix)
    for ((index, element) in collection.withIndex()) {
        if (index > 0) result.append(separator) // 첫 원소 앞에는 구분자 X
        result.append(element)
    }
    result.append(postfix)
    return result.toString()    
}
```

### step 1) 이름 붙인 인자로 함수 호출 시 가독성 향상시키기
- 함수의 시그니처를 확인해야 각 인자가 어떤 역할을 하는지 알 수 있다
- 특히 Boolean 플래그 값을 전달해야 하는 경우 가독성이 떨어진다
- 코틀린은 함수에 전달하는 인자의 이름을 명시할 수 있다.
  - 모든 인자의 이름을 지정하는 경우 인자 순서를 변경할 수 있다
  ```kotlin 
  fun main() {
  
    val list = listOf(1, 2, 3)
    println(
      joinToString(
        collection = list,
        separator = ",",
        prefix = "[",
        postfix = "]"
      )
    )
  }
  ```

### step 2) 디폴트 파라미터 값을 사용해 인자 기본값 지정해주기
- 자바의 경우 메서드 오버로딩을 통해 인자의 기본값을 지정하는데 경우에 따라 매우 많은 메서드가 필요할 수 있다
  - 많은 오버로드 함수들 중 어떤 함수가 불릴지 모호한 경우가 생긴다
- 코틀린은 디폴트 파라미터로 인자의 기본값을 지정해 오버로딩 없이 처리할 수 있다
  - 함수 호출 시 인자를 생략하면 디폴트 파라미터가 사용된다

아무 접두사나 접미사 없이 콤마로 원소를 구분해 출력하도록 디폴트 파라미터를 지정한다
```kotlin
fun <T> joinToString(
    collection: Collection<T>,
    separator: String = ", ",
    prefix: String = "",
    postfix: String = ""
): String {
    val result = StringBuilder(prefix)
    for ((index, element) in collection.withIndex()) {
        if (index > 0) result.append(separator) // 첫 원소 앞에는 구분자 X
        result.append(element)
    }
    result.append(postfix)
    return result.toString()
}
```
- 디폴트 파라미터를 설정하고 이름 붙인 인자로 커스텀하고 싶은 인자의 값만 지정해주면 된다
```kotlin
fun main() {
    joinToString(list, postfix = ";", prefix = "# ")
  // # 1, 2, 3;
}
```

코틀린에서 함수의 디폴트 파라미터 값은 함수 선언 쪽에 직접 포함되며 함수 시그니처와 함께 컴파일된다.
- 함수 호출동안 지정된 인자가 없으면 컴파일러가 해당 인자의 기본값을 자동으로 사용하도록 한다.
- 만약 함수의 디폴트 파라미터를 변경한 뒤 그 클래스가 포함된 파일을 재컴파일하면 그 함수를 호출하는 코드 중에 값을 지정하지 않은 모든 인자는 자동으로 바뀐 기본값을 적용받는다

자바는 디폴트 파라미터 개념이 없어 코틀린 함수를 호출할 때 모든 인자를 명시해야 한다.
- `@JvmOverloads`을 함수에 추가해두면 코틀린 컴파일러가 자동으로 맨 마지막 파라미터로부터 파라미터를 하나씩 생략한 오버로딩한 자바 메서드를 추가해준다.

```kotlin
@JvmOverloads
fun <T> joinToString(
    collection: Collection<T>,
    separator: String = ", ",
    prefix: String = "",
    postfix: String = ""
): String {
    val result = StringBuilder(prefix)
    for ((index, element) in collection.withIndex()) {
        if (index > 0) result.append(separator) // 첫 원소 앞에는 구분자 X
        result.append(element)
    }
    result.append(postfix)
    return result.toString()
}
```
- 디컴파일 시 `@JvmOverloads`가 붙은 메서드에 대해 오버로딩된 메서드가 추가된 것을 확인할 수 있다
```java
   @JvmOverloads
   @NotNull
   public static final String joinToString(
           @NotNull Collection collection, 
           @NotNull String separator, 
           @NotNull String prefix, 
           @NotNull String postfix
   ) { }

   @JvmOverloads
   @NotNull
   public static final String joinToString(
           @NotNull Collection collection, 
           @NotNull String separator, 
           @NotNull String prefix
   ) { }

   @JvmOverloads
   @NotNull
   public static final String joinToString(
           @NotNull Collection collection, 
           @NotNull String separator
   ) { }

   @JvmOverloads
   @NotNull
   public static final String joinToString(
           @NotNull Collection collection
   ) { }
```

### step 3) 정적인 유틸리티 클래스 없애기: 최상위 함수와 프로퍼티
- 자바는 모든 코드를 클래스의 메서드로 작성해야만 하기 때문에 특별한 상태나 인스턴스 메서드가 없는 클래스가 생겨난다
  - ex) JDK의 Collections 클래스
  - ex) Util 관련 클래스
- 코틀린은 이런 무의미한 클래스를 만들 필요 없이 함수를 직접 소스 파일의 최상위 수준에 작성해 모든 다른 클래스의 밖에 두고 사용할 수 있다.
  - 패키지의 멤버 함수로 사용할 때 그 패키지를 임포트하면 된다
  - 불필요한 내포 단계(클래스 이름)이 사라진다

```kotlin
package strings
fun joinToString( /* ... */ ): String { /* ... */ }
```
- JVM은 클래스 안에 들어있는 코드만을 실행할 수 있는데 어떻게 실행될 수 있는걸까?
- 코틀린 컴파일러가 이 파일을 컴파일할 때 최상위 함수가 들어있던 코틀린의 파일의 이름으로 클래스를 생성하고 그 클래스 안에 `joinToString()` 함수를 넣어준다
  - 자바의 다른 정적 메서드처럼 호출될 수 있는 이유가 여기 있다

실제 디컴파일 해보면 파일명 클래스 안에 정적 메서드로 정의된 것을 확인할 수 있다
```java
package strings;

public final class JoinKt {
    @NotNull
    public static final String joinToString(/* .. */) { /* ... */ }
}
```

파일에 대하는 클래스의 이름을 `@file:JvmName` 어노테이션으로 변경할 수 있다.
- 파일의 맨 앞, 패키지 이름 선언 전에 어노테이션을 추가하면 된다
```kotlin
@file:JvmName("StringFunctions")
package strings
fun joinToString( /* ... */ ): String { /* ... */ }
```

**최상위 프로퍼티에 `const` 변경자를 추가해 자바 코드에 상수 노출시키기**
- 파일 최상위 수준에 프로퍼티를 두면 이 프로퍼티의 값은 정적 필드로 저장된다.
- 접근자 메서드를 통해 자바 코드에 노출되는데 자바의 `public static final` 상수처럼 노출하고 싶다면 `const` 변경자를 추가해주면 된다

```kotlin
val UNIX_LINE_SEPARATOR = "\n"
const val WINDOWS_LINE_SEPARATOR = "\r\n"
```
```java
// 디컴파일 결과
public final class JoinKt {
   @NotNull
   private static final String UNIX_LINE_SEPARATOR = "\n";
   @NotNull
   public static final String WINDOWS_LINE_SEPARATOR = "\r\n";

   @NotNull
   public static final String getUNIX_LINE_SEPARATOR() {
      return UNIX_LINE_SEPARATOR;
   }
}
```

## 확장 함수
- 코틀린은 확장 함수와 함께 수신 객체 개념을 사용해 클래스 외부에 정의된 함수도 클래스 내부의 함수인 것처럼 사용할 수 있다.
  - 자바 API를 재정의하지 않고도 기존 클래스에 새로운 기능을 추가할 수 있다
  - 어떤 클래스의 멤버 메서드인 것처럼 호출할 수 있지만 실제로는 그 클래스의 밖에 정의된 함수다
- 이름 충돌을 막기 위해 사용하고 싶은 함수를 개별 임포트해 사용한다

**수신 객체?**
- 확장 함수로 기능을 확장하고 싶은 대상 객체를 수신 객체라고 한다
- 확장 함수 작성 시 수신 객체의 타입을 덧붙여 작성하고 수신 객체에 `this` 키워드로 접근할 수 있다
```kotlin
// 수신 객체 타입: String
// 수신 객체: this (String 객체)
fun String.lastChar() = this.get(this.length - 1)
fun String.lastChar() = get(length - 1) // this 생략 가능

// 사용 예시
fun main() {
    println("Kotlin".lastChar()) // n
}
```
- 마치 소유하고 있지도 않은 String 클래스에 직접 메서드를 추가하는 것과 같은..!
- 다른 JVM 언어로 작성된 클래스도 확장 가능하고, final로 상속을 막은 클래스도 확장 가능하다
  - 클래스의 메서드를 재정의하는 것이 아니라 새로운 메서드를 추가하는 것이기 때문

**수신 객체의 메서드나 프로퍼티를 바로 사용할 수 있지만 캡슐화를 깨지는 않는다!**
- 확장 함수는 수신 객체의 private 메서드나 프로퍼티에 접근할 수 없다 (컴파일 에러)
- 호출하는 쪽에서 확장 함수와 멤버 메서드를 구분할 수 없고, 구분을 할 필요도 없다

**자바에서 확장 함수 호출하기**
- 확장 함수는 수신 객체를 첫 번째로 인자로 받는 정적 메서드로 자바 코드처럼 사용하면 된다

**확장 함수는 단지 정적 메서드 호출에 대한 문법적인 편이일 뿐**
- 클래스가 아닌 더 구체적인 타입을 수신 객체 타입으로 지정할 수 있다
- ex) 문자열 컬렉션만 호출할 수 있는 join 함수
```kotlin
fun Collection<String>.join(
    separator: String = ", ",
    prefix: String = "",
    postfix: String = ""
) = joinToString(separator, prefix, postfix) 
```

### 확장함수는 정적 메서드와 마찬가지로 하위 클래스에서 오버라이드할 수 없다

**일반적인 오버라이드**
- 코틀린에서 상위 클래스의 하위 클래스를 생성하려면 상위 클래스가 `open` 키워드로 허용해주어야 한다
  - 메서드도 마찬가지로 구현을 `open` 키워드로 허용해주어야 한다
```kotlin
open class Shape {
    open fun draw() { println("Shape") }
}

class Circle: Shape() {
    override fun draw() { println("Circle") }
}
```
- Circle이 Shape의 하위 타입이기 때문에 Shape 타입 변수에 Circle 객체를 대입할 수 있다
- 이 Shape 타입 변수로 `draw()`를 호출하면 Circle의 `draw()`가 호출된다

**확장 함수도 오버라이드?**
- 확장 함수는 수신 객체로 지정한 변수의 컴파일 시점의 타입에 의해 결정되어 첫 번째 인자가 수신 객체인 정적 자바 메서드로 컴파일된다
  - Java의 static 함수가 정적으로 결정되는 것과 동일
- 즉, 런타임에 그 변수에 저장된 객체의 타입에 의해 결정되는 것이 아니라서(동적이 아닌 정적) 위의 예시처럼 Shape 타입에 그 하위 타입인 Circle을 대입해도 컴파일 시점의 타입은 Shape이기 때문에 Shape의 확장 함수가 호출된다
- 확장 함수가 클래스 정의의 일부가 아니라 외부에 따로 정의된 독립적인 함수임을 기억하자

## 확장 프로퍼티
- 함수가 아닌 프로퍼티 형식의 구문으로 사용할 수 있는 API를 제공할 수 있다
- 프로퍼티긴 하지만 상태를 저장할 수 없기 때문에 아무 상태를 가질 수 없다
  - 저장할 뒷받침하는 필드가 없기 때문에 초기화 코드를 쓸 수 없다
  - 뒷받침하는 필드가 없어 기본 getter 구현을 제공할 수 없으므로 최소한의 getter는 제공해야 한다
```kotlin
var StringBuilder.lastChar: Char
    get() = get(length - 1) // getter 구현
    set(value) {
        this.setCharAt(length - 1, value) // setter 구현
    }
```

## 인자의 개수가 달라질 수 있는 가변 인자 함수
리스트 생성 시 원하는 만큼의 원소를 전달하면 리스트로 생성해줬다.
```kotlin
public fun <T> listOf(vararg elements: T): List<T> = if (elements.size > 0) elements.asList() else emptyList()
```
- `vararg` 키워드를 사용해 가변 인자 함수를 만들 수 있다
- 가변 인자는 배열로 전달되며 스프레드(`*`) 연산자를 사용해 배열을 펼쳐서 전달할 수 있다
```kotlin
fun main() {
    val list = listOf(1, 2, 3)
    println(list) // [1, 2, 3]
    
    val array = arrayOf(4, 5, 6)
    println(listOf(*array)) // [4, 5, 6]
}
```

## 중위 호출
맵 생성 시 `to` 키워드를 사용해 키와 값을 쌍으로 묶어주었다.
- 코틀린 키워드가 아닌 중위 호출(`infix`)로 일반 메서드를 호출한 것이다
```kotlin
infix fun <A, B> A.to(that: B): Pair<A, B> = Pair(this, that)

fun main() {
    1.to("one") // 일반적인 방식
    1 to "one" // 중위 호출 방식
}
```
- 인자가 하나뿐인 일반 메서드(or 확장 함수)에만 `infix` 변경자를 함수 선언 앞에 추가해 중위 호출을 사용할 수 있다.

## 구조 분해 선언
- 구조 분해 선언을 사용하면 객체의 프로퍼티를 여러 개의 변수에 나눠서 대입할 수 있다
- 코틀린 표준 라이브러리 클래스인 `Pair`와 `Triple`은 구조 분해 선언을 지원한다

활용 예 1)
```kotlin
fun main() {
    val map = mapOf("key1" to "value1", "key2" to "value2")
    for ((key, value) in map) {
        println("$key -> $value")
    }
    // key1 -> value1
    // key2 -> value2
}
```
- mapOf도 `vararg`로 `Pair` 인자를 가변으로 받는 함수로 구현되어 있다
```kotlin
fun <K, V> mapOf(vararg pairs: Pair<K, V>): Map<K, V> { /* .. */ }
```

활용 예 2)
```kotlin
fun main() {
    val list = listOf("one", "two", "three")
    for((index, element) in list.withIndex()) {
        println("$index: $element")
    }
    // 0: one
    // 1: two
    // 2: three
}
```

## 문자열 핸들링
- 자바의 `split()`은 정규식을 구분 문자열로 받기 때문에 점(.)을 사용해 문자열을 분리할 수 없다
  - 정규식에서 점은 모든 문자와 매칭되기 때문
  - 이스케이핑 필요
- 코틀린은 여러 조합의 인자를 받는 확장 함수를 제공하는데 정규식은 String이 아닌 Regex 타입으로 받는다
  - 확실하게 정규식이나 일반 텍스트 중 어느 것으로 문자열을 분리하는지 파악할 수 있다
  ```kotlin
  fun main() {
      val text = "a.b.c"
      println(text.split('.')) // [a, b, c]
      println(text.split(Regex("\\."))) // [a, b, c]
  }
  ```
- `split()`은 `vararg`로 여러 개의 구분자를 받을 수 있다
  - 자바는 구분자를 하나만 받을 수 있었다

### 정규식과 3중 따옴표로 묶은 문자열
**경로 파싱 메서드**
```kotlin
// version 1
fun parsePathV1(path: String) {
    val directory = path.substringBeforeLast("/")
    val fullName = path.substringAfterLast("/")
    val fileName = fullName.substringBeforeLast(".")
    val extension = fullName.substringAfterLast(".")
  
    println("Dir: $directory, name: $fileName, ext: $extension")
}

// version 2
fun parsePathV2(path: String) {
    val regex = Regex("""(.+)/(.+)\.(.+)""")
    val matchResult = regex.matchEntire(path)
    if (matchResult != null) {
        val (directory, name, ext) = matchResult.destructured
        println("Dir: $directory, name: $name, ext: $ext")
    } else {
        println("Invalid path")
    }
}

fun main() {
    val path = "/home/user/hello.txt"
    parsePathV1(path)
    // Dir: /home/user, name: hello, ext: txt
    
    parsePathV2(path)
    // Dir: /home/user, name: hello, ext: txt
}
```
- 정규식은 3중 따옴표(`"""`)로 묶어주면 이스케이프 없이 사용할 수 있다
- 정규식의 `matchEntire()` 메서드는 정규식과 일치하는 문자열이 있는지 확인하고 일치하는 문자열을 찾으면 `MatchResult` 객체를 반환한다
- `destructured` 프로퍼티를 사용해 정규식의 그룹을 분해해 변수에 대입할 수 있다


**여러 줄 3중 따옴표 문자열**
- 이스케이프를 피하는 목적 외에도 여러 줄 문자열을 작성할 때도 사용된다
- 줄 바꿈을 포함해 아무 문자열이나 이스케이프 없이 그대로 들어간다
```kotlin
fun main() {
    val textA = """
        |Hello
        |World
    """.trimIndent()

    val textB = """
        |Hello
        |World
    """.trimMargin()

    println(textA)
    // |Hello
    // |World
  
    println(textB)
    // Hello
    // World
}
```
- `trimIndent()` 메서드는 문자열의 각 줄 앞에 있는 공백 중 가장 짧은 공통 공백을 찾아 각 줄의 첫 부분에서 제거하고, 공백만으로 이루어진 첫 번째 줄과 마지막 줄을 제겋해준다
- `trimMargin()` 메서드는 각 줄 앞에 있는 기호를 기준으로 공백을 제거해준다
- 줄 바꿈이 들어가지만 줄 바꿈을 `/n`과 같은 특수문자를 사용해 넣을 수 없다
- 문자열 템플릿을 사용할 수 있다
```kotlin
fun main() {
    val name = "Kotlin"
    val text = """
        |Hello, $name
        |Welcome to $name
    """.trimIndent()
  
    println(text)
    // Hello, Kotlin
    // Welcome to Kotlin
}
```

## 로컬 함수
자바의 경우 메서드를 추출해 긴 메서드를 짧은 메서드로 나누는 리팩토링을 할 수 있다. 이러다보면 작은 메서드들이 클래스 안에 너무 많아지고 각 메서드간 관계를 파악하기 힘들어진다

코틀린은 함수에서 추출한 함수를 로컬 함수를 통해 원래의 함수 내부에 내포시킬 수 있다
```kotlin
fun main() {
    fun isOdd(x: Int) = x % 2 != 0
    fun isEven(x: Int) = x % 2 == 0

    val numbers = listOf(1, 2, 3, 4, 5)
    val oddNumbers = numbers.filter(::isOdd)
    val evenNumbers = numbers.filter(::isEven)

    println(oddNumbers) // [1, 3, 5]
    println(evenNumbers) // [2, 4]
}
```
- 클래스 안의 로컬 함수라면 그 클래스의 프로퍼티에 접근할 수 있다
- 선언된 함수 내부에서만 사용 가능하므로 범위를 제한해 모듈성을 향상시킬 수 있다

**언제 사용할까?**
- 특정 함수 내에서 매우 자주 사용되는 함수가 있을 때
- 외부에 노출할 필요 없이 세부사항을 숨기고자 할 때
- 다른 클래스에서 재사용될 가능성이 전혀 없는 경우 모듈간 경계를 명확하게 하기 위해


