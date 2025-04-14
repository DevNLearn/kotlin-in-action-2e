# 3. 함수 정의와 호출

코틀린의 함수 정의 방식, 확장 함수, 인자 처리, 컬렉션, 문자열 등

## 3.1 코틀린에서 컬렉션 만들기

- 리스트와 맵을 아래와 같은 방법으로 만들 수 있음.
- 코틀린은 표준 자바 컬렉션 클래스를 사용

```kotlin
fun main() {
	val set = setOf(1, 7, 53)
	val list = listOf(1, 7, 53)
	val map = mapOf(1 to "one", 7 to "seven", 53 to "fifty-three")
	
	println(set.javaClass)
	// class java.util.LinkedHashSet
	println(list.javaClass)
	// class java.util.Arrays$ArrayList
	println(map.javaClass)
	// class java.util.LinkedHashMap
}
```

- `to`는 중위 함수이며 `Pair` 객체를 생성함 → `1 to "one"` 은 `Pair(1, "one")`과 같음.
- `listOf(Pair(1, "one"), Pair(2, "two"))` 대신 `listOf(1 to "one", 2 to "two")`처럼 사용 가능.

> 📖 코틀린 컬렉션은 기본적으로 읽기 전용입니다. 이에 대응하는 가변 인터페이스는 8장에서 자세히 다룹니다.
>

---

## 3.2 함수를 호출하기 쉽게 만들기

- **이름 붙은 인자**로 가독성 향상

    ```kotlin
    // before
    joinToString(collection, " ", " ", ".")
    // after
    joinToString(collection, separator = " ", prefix = " ", postfix = ".")
    ```

- **기본 인자값(default argument)** 으로 함수 호출을 유연하게 구성할 수 있음.

    ```kotlin
    fun <T> joinToString(
        collection: Collection<T>,
        separator: String = ", ",
        prefix: String = "",
        postfix: String = ""
    ): String {
        val result = StringBuilder(prefix)
        for ((index, element) in collection.withIndex()) {
            if (index > 0) result.append(separator)
            result.append(element)
        }
        result.append(postfix)
        return result.toString()
    }
    ```

    - 아래처럼 인자를 지정하면 생략할 수 있음.

        ```kotlin
        fun main() {
        	joinToString(list, postifx = ";", prefix = "# ")
        	// # 1, 2, 3;
        }
        ```

    - 자바에서는 오버로딩으로 처리했던 걸 코틀린은 기본 인자값으로 해결함.
    - `@JvmOverloads` 어노테이션으로 자바에서 호출할 수 있는 오버로드 함수 생성.

        ```kotlin
        @JvmOverloads
        fun <T> joinToString(
            collection: Collection<String>,
            separator: String = ", ",
            prefix: String = "",
            postfix: String = ""
        ): String {/*...*/}
        
        // 자바
        String joinToString(Collection<T> collection, String separator, 
        		String prefix, String postfix);
        
        String joinToString(Collection<T> collection, String separator, 
        		String prefix);
        
        String joinToString(Collection<T> collection, String separator);
        
        String joinToString(Collection<T> collection);
        ```

- 최상위 함수는 파일 이름 기준으로 클래스(`파일명 + Kt`)로 컴파일됨.
  → `Join.kt` → `JoinKt`
- 변경하고 싶다면 `@file:JvmName("CustomName")` 사용.
- **최상위 프로퍼티**도 가능.

    ```kotlin
    var opCount = 0
    
    fun performOperation() {
    	opCount++;
    }
    ```

- 상수 정의 시 `const val` 사용 → 자바의 `public static final`과 동일.

    ```kotlin
    // Kotiln
    const val UNIX_LINE_SEPARATOR = "\n"
    
    //Java
    public static final String UNIX_LINE_SEPARATOR = "\n"
    ```


---

## 3.3 확장 함수와 확장 프로퍼티

- 기존 클래스에 함수를 추가하는 것처럼 보이지만 실제로는 정적 메서드로 컴파일됨.
- 수신 객체 타입을 지정하고, 해당 타입 내부처럼 `this` 생략하고 접근 가능.

    ```kotlin
    fun String.lastChar():  
       Char = this.get(this.length - 1)
       
    // this 생략
    fun String.lastChar(): Char = get(length - 1)
       
    println("Kotlin".lastChar()) // → 'n'
    ```

- 확장 함수는 멤버 함수보다 **우선순위가 낮음.**
- 확장 함수는 정적으로 바인딩되므로 **오버라이딩 불가.**

    ```kotlin
    open class View {
    	open fun click() = println("View clicked")
    }
    
    class Button: View() {
    	override fun click() = println("Button clicked")
    }
    
    fun main() {
    	val view: View = Button()
    	view.click()
    	// Button clicked
    }
    ```


- **확장 프로퍼티**도 선언 가능

    ```kotlin
    val String.lastChar: Char
        get() = get(length - 1)
        
    var StringBuilder.lastChar: Char
    	get() = get(length - 1)
    	set(value: Char) {
    		this.setCharAt(length - 1, value)
    	}
    ```


---

## 3.4 컬렉션 처리와 다양한 문법

- 코틀린은 자바 컬렉션 클래스를 그대로 사용하며, 그 위에 다양한 확장 함수를 제공함
    - 대표적인 예: `last` , `max`, ..
- **가변 길이 인자 vararg**
    - 여러 개의 인자를 배열처럼 넘길 수 있음

    ```kotlin
    val list = listOf(2, 3, 5, 7, 11)
    
    fun <T> listOf(vararg elements: T): List<T> { /* 구현 */}
    ```

    - 다른 배열을 전달할 땐 `*` 스프레드 연산자 사용

    ```kotlin
    fun main(args: Array<Strings>) {
    	val list = listOf("args: ", *args)
    	println(list)
    }
    ```

- **중위 호출 infix**
    - 중위 호출로 `1 to "one"` 같은 표현 가능 → `infix fun` 으로 선언된 함수만 가능

    ```kotlin
    infix fun Any.to(other: Any) = Pair(this, other)
    val (number, name) = 1 to "One"
    ```

- 구조 분해 선언
    - `Pair`, `Map`, `withIndex()` 등에서 값을 분해해 변수에 담을 수 있음

    ```kotlin
    for((index, element) in collection.withIndex()){
    	println("$index: $element")
    }
    ```

  > 📖 구조 분해 선언의 내부 동작과 일반 규칙은 9장에서 자세히 다룹니다.
>

---

## 3.5 문자열과 정규식 다루기

- 코틀린 문자열은 자바의 `String`과 동일.
- 자바처럼 `split()`을 사용할 수 있고, 확장 함수로 여러 구분자를 직접 넘길 수 있음.

    ```kotlin
    println("12.345-6.A".split("\\.|-".toRegex())) 
    // [12, 345, 6, A]
    
    println("12.345-6.A".split('.', '-')) 
    // [12, 345, 6, A]
    ```

- 특정 문자 기준으로 분해 가능

    ```kotlin
    fun parsePath(path: String) {
    	val directory = path.substringBeforeLast("/")
    	val fullName = path.substringAfterLast("/")
    	val fileName = fullName.substringBeforeLast(".")
    	val extension = fullName.substringAfterLast(".")
    	
    	println("Dir: $directory, name: $fileName, ext: $extension")
    }
    
    fun main() {
    	parsePath("/Users/yole/kotiln-book/chapter.adoc")
    	// Dir: /Users/yole/kotlin-book, name: chapter, ext: adoc
    }
    ```


- **멀티라인 문자열**
    - 코틀린에서는 `"""`(삼중 따옴표)로 감싸서 **여러 줄짜리 문자열을 그대로 작성할 수 있음.**
    - 자바처럼 줄 끝에 `\n`을 붙이거나, 줄바꿈을 이스케이프하지 않아도 됨.
    - 들여쓰기를 정리할 때는 `trimIndent()`를 사용하면, **가장 왼쪽의 공백 기준으로 줄 들여쓰기를 제거**해줌.

        ```kotlin
        val kotlinLogo =
        	"""
        	| //
        	| //
        	|/\
        	""".trimIndent()
        
        fun main() {
        	println(kotilnLogo)
        	// | //
        	// | //
        	// |/\
        }
        ```

    - HTML, XML, JSON 등 **포맷을 유지한 텍스트 작성에 유리함.**

---

## 3.6 코드 깔끔하게 다듬기: 로컬 함수와 확장

- 자바에서는 DRY 원칙(반복하지 말라)을 따르기 쉽지 않음.
  → 보통 메서드 추출로 리팩터링
- 코틀린에서는 로컬 함수로 중복을 제거할 수 있음.
    - 예를 들어 아래와 같이 빈 값 검사를 반복하는 코드가 있다고 가정해보자.

        ```kotlin
        class User(val id: Int, val name: String, val address: String)
        
        fun saveUser(user: User) {
            if (user.name.isEmpty()) {
                throw IllegalArgumentException(
                    "Can't save user ${user.id}: empty Name")
            }
            if (user.address.isEmpty()) {
                throw IllegalArgumentException(
                    "Can't save user ${user.id}: empty Address")
            }
        
            // 저장 로직...
        }
        ```

    - 함수안에 함수 정의 가능, 바깥 함수의 파라미터나 변수에 접근 가능.

        ```kotlin
        fun saveUser(user: User) {
            fun validate(value: String, fieldName: String) {
                if (value.isEmpty()) {
                    throw IllegalArgumentException(
                        "Can't save user ${user.id}: empty $fieldName")
                }
            }
        
            validate(user.name, "Name")
            validate(user.address, "Address")
        
            // 저장 로직...
        }
        
        ```

- 확장 함수로 분리하여 재사용성 향상.
    - 검증 로직을 `User` 클래스에 대한 **확장 함수로 분리**할 수 있음.
    - `saveUser` 함수가 더 읽기 쉬워지고, 검증 로직도 재사용 가능.

    ```kotlin
    class User(val id: Int, val name: String, val address: String)
    
    fun User.validateBeforeSave() {
        fun validate(value: String, fieldName: String) {
            if (value.isEmpty()) {
                throw IllegalArgumentException(
                    "Can't save user $id: empty $fieldName")
            }
        }
    
        validate(name, "Name")
        validate(address, "Address")
    }
    
    fun saveUser(user: User) {
        user.validateBeforeSave()
        // 저장 로직...
    }
    
    ```


---

##  요약 정리

- 코틀린은 자체 컬렉션 클래스를 정의하지 않지만 자바 클래스를 확장해서 더 풍부한 API를 제공한다.
- 함수 파라미터의 기본값을 정의하면 오버로딩한 함수를 정의할 필요성이 줄어든다. 이름붙인 인자를 사용하면 함수의 인자가 많을 때 함수 호출의 가독성을 더 향상시킬 수 있다.
- 코틀린 파일에서 클래스 멤버가 아닌 최상위 함수와 프로퍼티를 직접 선언할 수 있다. 이를 통해 코드 구조를 더 유연하게 만들 수 있다.
- 확장 함수와 프로퍼티를 사용하면 외부 라이브러리에 정의된 클래스를 포함해 모든 클래스의 API를 그 클래스의 소스코드를 바꿀 필요 없이 확장할 수 있다.
- 중위 호출을 통해 인자가 하나밖에 없는 메서드나 확장 함수를 더 깔끔한 구문으로 호출할 수 있다.
- 코틀린은 정규식과 일반 문자열을 처리할 때 유용한 다양한 문자열 처리 함수를 제공한다.
- 자바 문자열로 표현하려면 수많은 이스케이프가 필요한 문자열의 경우 3중 따옴표 문자열을 사용하면 더 깔끔하게 표현할 수 있다.
- 로컬 함수를 써서 코드를 더 깔끔하게 유지하면서 중복을 제거할 수 있다.