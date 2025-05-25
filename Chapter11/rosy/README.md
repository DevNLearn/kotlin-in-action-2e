# 11장 제네릭스

Kotlin에서도 Java처럼 타입에 유연함을 주기 위해 제네릭스를 사용한다.   
이전까지는 자주 쓰였지만 자세히 설명하지 않았던 제네릭의 개념을 이 장에서 깊이 있게 다루며, 
Kotlin만의 특징적인 개념인 선언 지점 변성, 실체화된 타입 파라미터 등을 함께 설명한다.

---

## 11.1 타입 인자를 받는 타입 만들기: 제네릭 타입 파라미터

- 제네릭 타입을 사용하면 타입 파라미터를 받는 타입을 정의할 수 있다.
- 이렇게 하면 리스트 같은 컨테이너가 어떤 타입의 값을 담는지 명확하게 표현할 수 있다.

```kotlin
val authors = listOf("Dmitry", "Svetlana")
```

- `listOf` 함수에 문자열을 전달했기 때문에, 컴파일러는 이 리스트가 `List<String>`이라는 것을 추론할 수 있다. IDE에서는 이런 추론된 타입을 인레이 힌트로 보여준다.

    ```kotlin
    val authors: List<String> = listOf("Sveta", "Seb", "Dima", "Roman")
    ```

- 하지만 **빈 리스트를 만들 경우**에는 타입을 추론할 근거가 없기 때문에 **직접 타입 인자를 명시해야** 한다.
  다음은 두 가지 방식 모두를 보여주는 예제다.

    ```kotlin
    val readers: MutableList<String> = mutableListOf()
    val readers = mutableListOf<String>()
    ```

    - 두 선언은 완전히 동일하게 동작한다.
      컬렉션 생성 함수는 이미 앞에서 다뤘지만, 여기서는 타입 인자와의 관계를 다시 강조하고 있다.

> **Kotlin에는 Raw 타입이 없다.**
>
>
> Java에서는 제네릭이 도입되기 이전과의 호환성을 위해 타입 인자를 생략할 수 있었고, 이를 Raw 타입이라고 불렀다.
>
> ```java
> ArrayList aList = new ArrayList(); // Java에서는 허용
> ```
>
> 하지만 Kotlin은 처음부터 제네릭을 도입했기 때문에 타입 인자는 항상 명시되거나 추론되어야 한다. 타입 인자를 생략한 Java의 Raw 타입을 Kotlin에서는 `Any!`로 처리한다. 여기서 `Any!`는 플랫폼 타입을 의미한다.
>

### 제네릭 타입과 함께 동작하는 함수와 프로퍼티

- 제네릭 리스트를 다룰 때는 특정 타입의 리스트뿐 아니라 모든 타입의 리스트를 처리할 수 있는 함수가 필요하다. 이를 위해 제네릭 함수를 작성한다.
- 제네릭 함수는 함수 자체가 타입 파라미터를 받는다. 예를 들어, 컬렉션 라이브러리 함수 중 `slice`는 다음처럼 정의되어 있다.

    ```kotlin
    fun <T> List<T>.slice(indices: IntRange): List<T>
    ```

    - 여기서 `T`는 타입 파라미터이며, 이 함수는 어떤 타입의 리스트든 받아서 같은 타입의 리스트를 반환한다.

```kotlin
fun main() {
    val letters = ('a'..'z').toList()
    println(letters.slice<Char>(0..2)) // [a, b, c]
    println(letters.slice(10..13))     // [k, l, m, n]
}
```

- 타입 파라미터 `T`는 생략해도 Kotlin 컴파일러가 추론할 수 있으므로 명시하지 않아도 된다.
- `filter` 함수는 조건을 만족하는 원소만 골라내는 함수로, 다음과 같이 정의된다.

    ```kotlin
    fun <T> List<T>.filter(predicate: (T) -> Boolean): List<T>
    ```


```kotlin
fun main() {
    val authors = listOf("Sveta", "Seb", "Roman", "Dima")
    val readers = mutableListOf("Seb", "Hadi")
    
    println(readers.filter { it !in authors }) // [Hadi]
}
```

- 람다에서 사용된 `it`의 타입은 `String`이다. 컴파일러는 `filter`가 `List<T>`에 정의되어 있고, 현재 리스트의 타입이 `List<String>`임을 알고 있으므로, `T`는 `String`이라고 추론할 수 있다.

### 제네릭 확장 프로퍼티

- 함수뿐 아니라, 확장 프로퍼티도 제네릭하게 정의할 수 있다.
- 예를 들어 리스트의 끝에서 두 번째 요소를 반환하는 `penultimate` 프로퍼티는 다음과 같이 정의할 수 있다.

    ```kotlin
    val <T> List<T>.penultimate: T
        get() = this[size - 2]
    
    fun main() {
        println(listOf(1, 2, 3, 4).penultimate) // 3
    }
    ```

- 이처럼 확장 프로퍼티만 제네릭으로 정의할 수 있다.
- 일반 프로퍼티는 제네릭 타입 파라미터를 사용할 수 없다. 예를 들어 다음 코드는 오류가 발생한다.

    ```kotlin
    val <T> x: T = TODO()
    // ERROR: type parameter of a property must be used in its receiver type
    ```


### 제네릭 클래스 선언하기

- 클래스나 인터페이스 이름 뒤에 `<T>`처럼 타입 파라미터를 명시하면, 그 클래스나 인터페이스는 제네릭하게 동작한다.

    ```kotlin
    interface List<T> {
        operator fun get(index: Int): T
    }
    ```

    - 이 인터페이스는 `T`라는 타입 파라미터를 선언하고, 이 타입은 클래스 본문 안에서 일반 타입처럼 사용할 수 있다.
- 제네릭 클래스를 상속하거나 구현할 때는 타입 인자를 지정해야 한다

    ```kotlin
    class StringList : List<String> {
        override fun get(index: Int): String = TODO()
    }
    
    class ArrayList<T> : List<T> {
        override fun get(index: Int): T = TODO()
    }
    ```

    - `StringList`는 `List<String>`을 구현하기 때문에 `get` 함수는 `String`을 반환해야 한다.
    - `ArrayList<T>`는 자신의 타입 파라미터 `T`를 그대로 기반 클래스에 넘긴다.
      두 `T`는 이름만 같고 서로 다른 타입 파라미터다.
- 클래스가 자기 자신을 타입 인자로 사용할 수도 있다.
  `Comparable<T>` 인터페이스를 구현하는 방식이 그렇다.

    ```kotlin
    interface Comparable<T> {
        fun compareTo(other: T): Int
    }
    
    class String : Comparable<String> {
        override fun compareTo(other: String): Int = TODO()
    }
    ```

    - 이 구조는 같은 타입의 값끼리 비교 가능하게 만들고자 할 때 자주 쓰인다.

### 제네릭 클래스나 함수가 사용할 수 있는 타입 제한: 타입 파라미터 제약

- 함수나 클래스에 사용되는 타입 파라미터는 특별한 제약이 없는 한 어떤 타입이든 사용할 수 있다.
  하지만 때로는 타입 파라미터로 특정 조건을 만족하는 타입만 사용하도록 **제한**하고 싶을 때가 있다.
  이럴 때는 **타입 파라미터 제약**을 사용할 수 있다.
- 예를 들어 숫자 타입의 리스트 원소를 모두 더하는 `sum` 함수를 만들고 싶다고 하자.
  이 함수는 `List<Int>`나 `List<Double>` 등에는 적용될 수 있지만, `List<String>`에는 적용되면 안 된다.

    ```kotlin
    fun <T : Number> List<T>.sum(): T
    ```

    - 여기서 `T : Number`는 T는 Number의 하위 타입이어야 한다는 의미다. 따라서 `Int`, `Float`, `Double` 같은 타입만 허용된다.

    ```kotlin
    fun main() {
        println(listOf(1, 2, 3).sum()) // 6
    }
    ```

    - 타입 파라미터 T가 Number를 확장한다고 지정했기 때문에, `toDouble()` 같은 Number 클래스의 메서드를 호출할 수 있다.

### 값을 비교할 수 있는 타입으로 제약하기

- 다음은 두 값을 받아 더 큰 값을 반환하는 `max` 함수를 작성하는 예제다.
  이 함수는 두 값이 서로 비교 가능해야 하므로, 타입 파라미터 `T`가 `Comparable<T>`를 구현한다고 제한해야 한다.

    ```kotlin
    fun <T : Comparable<T>> max(first: T, second: T): T {
        return if (first > second) first else second
    }
    
    fun main() {
        println(max("kotlin", "java")) // kotlin
    }
    ```

    - `first > second`는 사실 내부적으로 `first.compareTo(second) > 0`으로 변환된다. 타입 `T`가 `Comparable<T>`를 구현한다고 선언했기 때문에 이 비교가 가능하다.
    - 만약 비교 불가능한 타입을 넣으려고 하면 컴파일 오류가 발생한다.

        ```kotlin
        // println(max("kotlin", 42)) // ❌ 오류 발생: 서로 비교 불가능한 타입
        ```


### 타입 파라미터에 여러 제약을 가하기

- 타입 파라미터에는 두 개 이상의 제약을 동시에 걸 수 있다.
- 예를 들어, `CharSequence`의 끝에 마침표를 붙이는 함수를 정의하려면 문자열 내용을 확인하고 수정할 수 있어야 한다. 따라서 `CharSequence`와 `Appendable` 두 인터페이스를 모두 구현한 타입이 필요하다.

    ```kotlin
    fun <T> ensureTrailingPeriod(seq: T)
    		where T : CharSequence, T : Appendable {
      if (!seq.endsWith('.')) {
            seq.append('.')
      }
    }
    
    fun main() {
        val sb = StringBuilder("Hello World")
        ensureTrailingPeriod(sb)
        println(sb) // Hello World.
    }
    ```

    - 여기서 사용된 `where` 절은 다중 제약을 명시할 때 사용하는 문법이다.
    - `StringBuilder`는 `CharSequence`와 `Appendable`을 모두 구현하므로 이 함수에 적합한 인자다.

### 명시적으로 타입 파라미터를 널이 될 수 없는 타입으로 제한하기

- 기본적으로 타입 파라미터에는 널이 될 수 있는 타입도 사용할 수 있다.

    ```kotlin
    class Processor<T> {
        fun process(value: T) {
            value?.hashCode()
        }
    }
    ```

    - 이 코드는 잘 작동하며 다음처럼 사용할 수 있다.

        ```kotlin
        val nullableStringProcessor = Processor<String?>()
        nullableStringProcessor.process(null)
        ```

- 하지만 어떤 경우에는 널이 될 수 없는 타입만 허용하고 싶은 때도 있다. 이럴 때는 `T : Any`라고 제한을 걸 수 있다.

    ```kotlin
    class Processor<T : Any> {
        fun process(value: T) {
            value.hashCode()
        }
    }
    ```

    - 이제 다음과 같은 코드는 컴파일 오류가 발생한다.

        ```kotlin
        // val nullableProcessor = Processor<String?>() // ❌ 오류: String?은 Any의 하위 타입이 아님
        ```

    - 이처럼 `T : Any`를 선언하면, 널이 될 수 없는 타입만 타입 인자로 받을 수 있다.
      `Any?`는 모든 타입(널 포함)의 최상위 타입이고, `Any`는 널을 제외한 타입의 최상위 타입이기 때문이다.

### 자바 코드와의 상호 운용에서 널 가능성 제약 표현하기

- 자바 코드와 Kotlin을 함께 사용할 경우, 제네릭 타입에서 **널 가능성**을 어떻게 해석해야 할지 애매할 때가 있다.
- 예를 들어, 자바에서는 다음과 같은 제네릭 인터페이스를 정의할 수 있다.

    ```java
    public interface JBox<T> {
        void put(@NotNull T t);
        void putIfNotNull(T t);
    }
    ```

    - `put()`은 널이 될 수 없는 값을 받는다.
    - `putIfNotNull()`은 널 가능성이 있는 값을 받을 수 있다.
- 이 인터페이스를 Kotlin에서 구현하려고 하면, 다음과 같이 문제가 생긴다.

    ```kotlin
    class KBox<T : Any> : JBox<T> {
        override fun put(t: T) { /* OK */ }
        override fun putIfNotNull(t: T) { /* ❌ null 불가인데? */ }
    }
    ```

    - `T : Any`로 타입 파라미터를 제한했기 때문에 `putIfNotNull`에서 널 값을 받을 수 없다. 하지만 자바에서는 `putIfNotNull`은 널 값을 받을 수 있어야 한다.
- 이런 문제를 해결하기 위해 Kotlin에서는 타입 사용 지점에서 널이 될 수 없음을 명시하는 `T & Any` 문법을 제공한다.

    ```kotlin
    class KBox<T> : JBox<T> {
        override fun put(t: T & Any) { /* t는 절대로 null 아님 */ }
        override fun putIfNotNull(t: T) { /* t는 null일 수 있음 */ }
    }
    ```

    - 이처럼 타입 파라미터 자체는 널 가능성을 허용하면서도, 특정 메서드에서는 명확하게 널 불가로 지정할 수 있게 해주는 것이 Kotlin의 `T & Any` 문법이다.

---

## 11.2 실행 시점 제네릭스 동작: 소거된 타입 파라미터와 실체화된 타입 파라미터

- Kotlin에서 제네릭은 컴파일 타임에는 타입 정보를 가지고 있지만, 실행 시점에는 그 정보가 사라지는 타입 소거(type erasure) 방식으로 구현되어 있다.
- 이 말은 즉, `List<String>` 이든 `List<Int>` 이든 런타임에는 그냥 `List` 로 인식된다는 뜻이다.

    ```kotlin
    val list1: List<String> = listOf("a", "b")
    val list2: List<Int> = listOf(1, 2, 3)
    ```

    - 이 두 리스트는 컴파일 타임에는 서로 다른 타입이지만, 런타임에는 모두 `List`로 보인다.
    - `list1`이 문자열 리스트인지, `list2`가 정수 리스트인지 is 검사나 캐스팅으로 구별할 수 없다.

```kotlin
fun printList(l: List<Any>) {
    when (l) {
        is List<String> -> println("Strings: $l")
        is List<Int> -> println("Integers: $l")
    }
}
```

- 이 코드는 컴파일 오류가 난다.
- `is List<String>`이나 `is List<Int>` 같은 검사는 런타임에 불가능하기 때문이다.
  타입 정보는 이미 지워져서 확인할 수 없다.
- 이런 상황에서 유일하게 확인할 수 있는 것은 **스타 프로젝션 구문**인데  `value is List<*>` 같은 형태다.
- 이 경우 `value`가 리스트라는 것까진 알 수 있지만, 그 안의 원소가 어떤 타입인지는 알 수 없다.

### 캐스팅은 가능한가?

- kotlin에서 `as` 나 `as?` 를 사용해서 **제네릭 타입으로의 캐스팅**은 가능하다.
- 다만 이때는 컴파일러가 “Unchecked cast” 경고를 출력한다.
  실행 시 타입 정보가 없기 때문에 성공 여부를 보장할 수 없기 때문이다.

```kotlin
fun printSum(c: Collection<*>) {
    val intList = c as? List<Int>
        ?: throw IllegalArgumentException("List is expected")
    println(intList.sum())
}
```

- 이 코드는 문제없이 컴파일되지만, 잘못된 타입이 들어오면 런타임 오류가 발생할 수 있다.

```kotlin
fun main() {
    printSum(listOf(1, 2, 3))          // 6 출력
    printSum(setOf(1, 2, 3))           // 예외 발생: List가 아님
    printSum(listOf("a", "b", "c"))    // 예외 발생: String을 Number로 변환 불가
}
```

- 문자열 리스트를 넘겼을 때는 캐스팅 자체는 성공하지만, `sum()`을 호출하는 시점에 `ClassCastException`이 발생한다.

### 컴파일 타임에 타입을 알면 안전하게 검사할 수 있다.

- 컴파일러가 타입을 알고 있으면 다음처럼 안전한 타입 검사가 가능하다.

```kotlin
fun printSum(c: Collection<Int>) {
    when (c) {
        is List<Int> -> println("List sum: ${c.sum()}")
        is Set<Int> -> println("Set sum: ${c.sum()}")
    }
}
```

- `c`의 타입이 `Collection<Int>`로 명확히 지정되어 있기 때문에 `is List<Int>`같은 검사가 허용된다.즉, 타입 소거는 런타임에 문제지만, 컴파일 타임에 타입이 명확하면 안전하게 사용할 수 있다.

### 일반 함수에서는 타입 인자를 사용할 수 없다.

- 타입 인자 `T`를 사용하는 함수 내부에서 `value is T` 같은 코드를 작성하면 오류가 발생한다.

```kotlin
fun <T> isA(value: Any) = value is T
// Error: Cannot check for instance of erased type: T
```

- 이유는 똑같다. 타입 정보가 **런타임에 지워지기 때문에 검사할 수 없기 때문이다.**

### inline + reified로 타입 인자를 실행 시점에 유지할 수 있다.

- Kotlin은 **inline 함수**안에서 **reified 키워드**를 사용하면 이 한계를 극복할 수 있다.
  `reified`는 “타입이 실체화되었다”는 뜻으로, 런타임에도 타입 정보를 유지한다.

    ```kotlin
    inline fun <reified T> isA(value: Any) = value is T
    
    fun main() {
        println(isA<String>("abc")) // true
        println(isA<String>(123))   // false
    }
    ```

- 이제 `is T` 검사도 가능해졌고, 코드도 자연스럽게 동작한다.

### 실체화된 타입 파라미터를 활용한 대표 함수: filterIsInstance

- Kotlin 표준 라이브러리에 있는 `filterIsInstance` 함수는 내부적으로 `reified`를 사용한 대표적인 함수다.

    ```kotlin
    fun main() {
        val items = listOf("one", 2, "three")
        println(items.filterIsInstance<String>()) // [one, three]
    }
    ```

    - 이 함수는 리스트의 각 원소에 대해 `is T`로 검사하고, 타입이 맞는 원소만 골라서 새로운 리스트를 만든다.

```kotlin
inline fun <reified T> Iterable<*>.filterIsInstance(): List<T> {
    val destination = mutableListOf<T>()
    for (element in this) {
        if (element is T) {
            destination.add(element)
        }
    }
    return destination
}
```

- 함수 본문을 보면 알 수 있듯, 이 코드는 컴파일 시점에 `element is String`처럼 구체적인 타입으로 변환된다.

### 클래스 참조를 더 간단히 하기 – ::class.java 대신 실체화된 타입 쓰기

- Java의 많은 API는 `java.lang.Class<T>` 객체를 요구한다. 예를 들어 `ServiceLoader.load`는 다음처럼 쓴다.

    ```kotlin
    val serviceImpl = ServiceLoader.load(Service::class.java)
    ```

- Kotlin에서는 다음처럼 줄일 수 있다.

    ```kotlin
    inline fun <reified T> loadService(): T {
        return ServiceLoader.load(T::class.java)
    }
    
    val service = loadService<Service>()
    
    ```

    - 타입 인자를 함수 호출에서 지정하면, `::class.java` 같은 문법을 쓰지 않고도 클래스 참조를 직접 사용할 수 있다.

### 안드로이드에서도 유용하게 쓰인다.

- 안드로이드에서 `startActivity` 같은 함수도 다음처럼 더 간결하게 만들 수 있다.

```kotlin
inline fun <reified T : Activity> Context.startActivity() {
    val intent = Intent(this, T::class.java)
    startActivity(intent)
}

// 호출 시
startActivity<DetailActivity>()
```

### 실체화된 타입을 프로퍼티에서 사용하기

- 프로퍼티에서도 `inline val`과 `reified`를 함께 쓰면 타입 정보를 사용할 수 있다.

```kotlin
inline val <reified T> T.canonical: String
    get() = T::class.java.canonicalName

fun main() {
    println(listOf(1, 2, 3).canonical) // java.util.List
    println(1.canonical)              // java.lang.Integer
}
```

### 실체화된 타입 파라미터의 제약

- `reified`는 강력하지만 몇 가지 제약이 있다.
- **할 수 있는 일**
    - `is`, `as` 검사 (타입 검사와 캐스팅)
    - `::class`, `::class.java` 사용
    - 다른 함수 호출에 타입 인자로 넘기기
- **할 수 없는 일**
    - `T()`처럼 타입 인자로 받은 클래스를 직접 인스턴스화
    - `T.Companion.someMethod()` 호출
    - `reified`가 필요한 함수를 일반 타입 파라미터로 호출
    - `inline`이 아닌 함수에서 `reified` 사용
      (`reified`는 inline 함수에서만 사용 가능하다.)
- 람다를 함께 전달해야 하는데 인라이닝을 원치 않는 경우엔 `noinline` 키워드로 인라이닝을 막을 수 있다.

---

## 11.3 변성은 제네릭과 타입 인자 사이의 하위 타입 관계를 기술

- 제네릭 타입에서 `List<String>`과 `List<Any>` 와 같이 기저 타입이 같고 타입 인자가 다른 여러 타입이 서로 어떤 관계가 있는지 설명할 때 **변성(variance)**이라는 개념을 사용한다.
- 예를 들어 `String`은 `Any`의 하위 타입이지만, 그렇다고 해서 `List<String>`이 `List<Any>`의 하위 타입인 것은 아니다.
- 왜냐하면 `List`에 **타입 파라미터를 어떻게 쓰느냐에 따라** 다르기 때문이다.

### 함수에 List<Any>를 받고 List<String>을 넘기면 괜찮을까?

```kotlin
fun printContents(list: List<Any>) {
    println(list.joinToString())
}

fun main() {
    val strings = listOf("abc", "def")
    printContents(strings) // OK?
}
```

- 이 코드는 잘 작동한다. `String`은 `Any`의 하위 타입이기 때문에 문자열 리스트도 `List<Any>`로 동작할 수 있다.

```kotlin
fun addAnswer(list: MutableList<Any>) {
    list.add(42)
}

fun main() {
    val strings = mutableListOf("abc", "def")
    addAnswer(strings) // 컴파일 오류
}
```

- `MutableList<String>`에 `Int`를 추가하는 코드다. 이건 **타입 안전하지 않기 때문에** Kotlin 컴파일러가 막는다.

- 읽기 전용인 `List<T>`는 안전하게 `List<String>` → `List<Any>`로 넘길 수 있다.
- 쓰기가 가능한 `MutableList<T>`는 타입 불일치로 인해 위험하므로 허용되지 않는다.
- 이처럼 제네릭 타입에서 타입 인자 간의 관계를 어떻게 유지할지 지정하는 것이 바로 변성이다.

### 타입과 클래스는 다르다.

- `String`은 클래스이자 타입이다.
- `String?`도 같은 클래스지만 다른 타입이다.
- 제네릭 클래스 `List<T>`는 클래스지만 타입은 아니다. `List<Int>`, `List<String>`은 타입이다.
  즉, 하나의 제네릭 클래스는 수많은 타입을 만들어낼 수 있다.

### 하위 타입이란?

- 어떤 타입 B가 타입 A가 필요한 모든 자리에 들어갈 수 있다면, B는 A의 **하위 타입(subtype)**이다.

```kotlin
fun test(i: Int) {
	val n: Number = i      // OK
	
	fun f(s: String) {/*...*/}
	f(n)                    // Int가 String 하위 타입이 아니어서 컴파일 되지 않음!
 }
```

- `Int`는 `Number`의 하위 타입이므로 `Number` 자리에 들어갈 수 있지만,
- `Int`는 `String`의 하위 타입이 아니므로 `String` 자리에 들어갈 수 없다.

### 널 가능성과 하위 타입

- `String`은 `String?`의 하위 타입이다.
- 반대로 `String?`은 `String`의 하위 타입이 아니다.

```kotlin
val s: String = "abc"
val t: String? = s       // OK
val u: String = t        // 에러 발생!
```

- 따라서 `T`와 `T?`는 같은 클래스지만 타입의 상하위 관계는 다르다.

### 제네릭 타입의 하위 타입 관계

- `MutableList<Any>`와 `MutableList<String>`은 아무 관계 없다.
- `MutableList<T>`에서 `T`는 읽기도 하고 쓰기도 하기 때문에 **공변(out)**도 **반공변(in)**도 될 수 없다.
  즉 **무공변(invariant)**이다.
- 반면, `List<T>`는 읽기만 가능하므로 **공변(out)**으로 지정해도 된다.

### 공변성(out)은 읽기 전용일 때 안전하다.

- 공변성은 **타입 파라미터의 하위 타입 관계가 그대로 유지되는 것**을 말한다.
- 예를 들어 `Cat`은 `Animal`의 하위 타입이고, `List<Cat>`을 `List<Animal>`처럼 사용하고 싶을 때가 있다.
  Kotlin에서는 이런 공변성을 **`out` 키워드로 명시**한다.

    ```kotlin
    interface Producer<out T> {
        fun produce(): T
    }
    ```

- `Producer<Cat>`은 `Producer<Animal>`의 하위 타입이 된다.
- 왜냐하면 `Producer`는 `T`를 반환만 하고, 소비(입력)하지 않기 때문!

### 공변 클래스를 직접 만들어보자!

```kotlin
open class Animal { fun feed() { /*...*/ } }

class Cat : Animal() { fun cleanLitter() { /*...*/ } }

class Herd<out T : Animal>(private val animals: List<T>) {
    val size: Int get() = animals.size
    operator fun get(i: Int): T = animals[i]
}
```

- `Herd<out T>`로 지정했기 때문에 다음과 같이 사용할 수 있다.

```kotlin
fun feedAll(animals: Herd<Animal>) {
    for (i in 0 until animals.size) {
        animals[i].feed()
    }
}

val cats = Herd(listOf(Cat(), Cat()))
feedAll(cats) // OK! 공변성 덕분
```

- `Herd<Cat>`은 `Herd<Animal>`의 하위 타입으로 취급된다.
- 타입 파라미터가 출력(out) 위치에서만 쓰였기 때문에 안전하다.

### 아웃 위치와 인 위치란?

- **아웃 위치**: 반환 타입에서 쓰이는 타입 파라미터 → 안전 (공변 가능)
- **인 위치**: 함수 인자 타입에 쓰이는 타입 파라미터 → 위험 (공변 불가)

```kotlin
interface Box<out T> {
    fun get(): T     // ✅ 아웃 위치
    // fun set(t: T) // ❌ 인 위치 → 공변 불가
}
```

- `out T`로 선언했으면 반환 타입에서는 사용 가능, 파라미터에서는 불가

### 왜 `MutableList<T>`는 공변이 아닐까?

- `MutableList<T>`는 값을 읽기도 하고 쓰기도 하기 때문에 T가 **인/아웃 위치 둘 다에서 사용된다.**
- 그래서 `out` 도 `in` 도 붙일 수 없는 **무공변 타입이다.**

```kotlin
interface MutableList<T> {
    fun get(index: Int): T     // 아웃 위치
    fun add(element: T)        // 인 위치
}
```

- 그래서 다음 코드는 컴파일되지 않는다.

```kotlin
val list: MutableList<out Number> = mutableListOf()
list.add(42) // ❌ Error: out-projected 타입이라 add() 호출 불가
```

- `out`으로 선언하면 인 위치인 `add`를 막아야 하므로, 사용할 수 없게 된다.

### 반공변성(in)은 소비 전용일 때 안전하다.

- 이번에는 `Comparator<T>`처럼 **타입 파라미터를 인자로만 사용하는 경우**다.
- 이럴 때는 타입 파라미터를 **in**으로 표시해서 **반공변성**을 선언할 수 있다.

```kotlin
interface Comparator<in T> {
    fun compare(e1: T, e2: T): Int { /*...*/ }
}
```

- `Comparator<Any>`는 `Comparator<String>`의 하위 타입이다.
- 즉, 더 넓은 범위의 타입을 비교할 수 있는 Comparator는 **더 구체적인 타입에도 사용할 수 있다.**
- `Comparator<Any>`는 `Comparator<String>`보다 범용적이다.

    ```kotlin
    val comp: Comparator<Any> = Comparator { a, b -> a.hashCode() - b.hashCode() }
    
    val strings = listOf("a", "b", "c")
    println(strings.sortedWith(comp)) 
    ```

    - `Comparator<Any>`는 `Comparator<String>`처럼 사용 가능하다.
    - 문자열을 비교하려면 문자열보다 넓은 범위인 `Any`를 받아도 괜찮다.

### 공변성, 반공변성, 무공변성 요약 비교

- Kotlin의 제네릭 변성은 크게 세 가지로 나뉘며, 각각 `out`, `in`, 아무 것도 없는 형태로 선언된다.
- 각 변성은 제네릭 타입의 **타입 인자(T)**가 **하위 타입 관계를 어떻게 유지하는가**에 따라 달라집니다.

| 개념 | 공변성 | 반공변성 | 무공변성 |
| --- | --- | --- | --- |
| 문법 | out T | int T | T(변성 없음) |
| 설명 | 타입 인자의 하위 타입 관계가 데네릭 타입에서도 유지된다. | 타입 인자의 하위 타입 관계가 제네릭 타입에서 뒤집힌다. | 하위 타입 관계가 성립하지 않는다. |
| 예시 타입 | Producer<out T> | Consumer<in T> | MutableList<T> |
| 예시 관계 | Producer<Cat>은 Producer<Animal>의 하위 타입이다. | Consumer<Animal>은 Consumer<Cat>의 하위 타입이다. |  |
| T 사용 위치 | T를 아웃 위치에서만 사용할 수 있다. | T를 인 위치에서만 사용할 수 있다. | T를 아무 위치에서나 사용할 수 있다. |

### 선언 지점 변성과 사용 지점 변성

- Kotlin과 Java는 제네릭 타입의 **변성(variance)**을 다루는 방식이 다르다.
- Kotlin: 선언 지점 변성 (declaration-site variance)
    - Kotlin에서는 클래스를 선언할 때 타입 파라미터가 공변인지(`out`), 반공변인지(`in`)를 딱 한 번 선언한다.
    - 이후 사용할 때는 따로 신경 쓸 필요 없이 간결하게 사용 가능하다.
- Java: 사용 지점 변성 (use-site variance)
    - Java는 클래스 선언에서는 변성을 표시하지 않고, **사용할 때마다** `? extends T`, `? super T` 같은 **와일드카드**로 변성을 지정해야 한다.

### 코틀린 선언 지점 변성과 자바 와일드 카드 비교

- Kotlin은 클래스를 선언할 때 `out` , `in` 을 붙이면, 나중에 사용할 때 따로 변성을 신경 쓸 필요가 없다.
- Java는 사용할 때마다 `? extends` , `? super` 처럼 직접 변성을 지정해야 해서 코드가 길고 복잡해진다.

    ```java
    public interface Stream<T> {
        <R> Stream<R> map(Function<? super T, ? extends R> mapper);
    }
    ```

- 결론적으로, Kotlin의 선언 지점 변성은 Java보다 **간결하고 우아한 코드 작성을 가능하게** 해준다.

### Kotlin의 사용 지점 변성

- Kotlin도 Java처럼 **특정 함수에서만 변성**을 명시할 수 있다.
- 대표적인 경우가 `MutableList<T>`를 읽기 전용이나 쓰기 전용으로 제한하고 싶을 때이다.
- 잘못된 복사 함수 (무공변 타입 사용)

    ```kotlin
    fun <T> copyData(source: MutableList<T>, destination: MutableList<T>) {
        for (item in source) {
            destination.add(item)
        }
    }
    ```

    - `source`는 읽기만, `destination`은 쓰기만 하지만 타입이 같아야 해서 유연하지 않는다.
- 해결: 사용 지점 변성(out/in)으로 표현

    ```kotlin
    fun <T> copyData(source: MutableList<out T>, destination: MutableList<T>) {
        for (item in source) {
            destination.add(item)
        }
    }
    ```

    - `source`는 `out T`: 읽기 전용
    - `destination`은 `T`: 일반 쓰기 가능

> Java의 ? extends T, ? super T 와 완전히 같은 표현!
>

### 타입 프로젝션이란?

- `out T`나 `in T`를 타입 인자 위치에서 사용하는 것을 타입 프로젝션(type projection)이라고 한다.

    ```kotlin
    val list: MutableList<out Number> = mutableListOf()
    list.add(42) // ❌ Error: out-projected 타입이라 add() 사용 불가
    ```

    - `out`으로 타입을 제한했기 때문에, `add()` 같이 값을 쓰는 메서드는 막히는 것!

### 스타 프로젝션 (*)

- 타입 인자를 모를 때는 `*` 기호를 사용해 **모든 타입에 대응하는 제네릭 타입**을 나타낸다.
- 예를 들어 원소 타입이 알려지지 않은 리스트는 List<*>라는 구문이다.
- MutableList<*>는 MutableList<Any?>와 같지 않다.
    - MutableList<Any?>는 모든 타입의 원소를 담을 수 있음을 알 수 있는 리스트
    - MutableList<*>는 어떤 정해진 구체적인 타입의 원소만을 담는 리스트지만 그 원소의 타입을 정확히 모른다.

```kotlin
import kotlin.random.Random

fun main() {
	val list: MutableList<Any?> = mutableListOf('a', 1, 'qwe')
	val chars = mutableListOf('a', 'b', 'c')
	val unknownElements: MutableList<*> =
			if (Random.nextBoolean()) list else chars
	println(unknownElements.first()) // a
	unknownElements.add(42) // Error
}
```

- `MutableList<*>`는 실제 타입을 모르지만 `읽기`는 가능하다.
- `Any?` 타입으로 동작하지만, 컴파일러는 쓰기 금지 처리된다.

### 검증기 예제로 보는 안전한 API 설계

- 문제: 검증기를 잘못된 타입으로 캐스팅할 수 있음

    ```kotlin
    val validators = mutableMapOf<KClass<*>, FieldValidator<*>>()
    validators[String::class] = DefaultStringValidator
    
    val stringValidator = validators[String::class] as FieldValidator<String>
    println(stringValidator.validate("")) // false
    // Warning: unchecked cast 경고 발생
    ```

    - 타입 정보를 잃기 때문에 `ClassCastException`이 발생할 수 있다.
- 해결책: 검증기 등록/조회 로직을 안전하게 감싸기(캡슐화 하기)

    ```kotlin
    object Validators {
        private val validators = mutableMapOf<KClass<*>, FieldValidator<*>>()
    
        fun <T : Any> registerValidator(kClass: KClass<T>, fieldValidator: FieldValidator<T>) {
            validators[kClass] = fieldValidator
        }
    
        @Suppress("UNCHECKED_CAST")
        operator fun <T : Any> get(kClass: KClass<T>): FieldValidator<T> =
            validators[kClass] as? FieldValidator<T>
                ?: throw IllegalArgumentException("No validator for ${kClass.simpleName}")
    }
    
    fun main() {
    	Validators.registerValidator(String::class, DefaultStringValidator)
    	
    	println(Validators[String::class].validate("Kotlin")) // true
    }
    ```

    - 내부에서만 unsafe 캐스팅을 하고, 외부에는 타입 안전한 API만 노출한다.

### 타입 별명(typealias)

- 타입이 너무 길어지거나 의미를 부여하고 싶을 때 `typealias`로 **짧고 의미 있는 이름**을 줄 수 있다.

```kotlin
typealias NameCombiner = (String, String, String, String) -> String

val authorsCombiner: NameCombiner = { a, b, c, d -> "$a et al." }
```

> 주의: `typealias`는 컴파일 시 원래 타입으로 바뀌기 때문에 **타입 안전성은 제공하지 않음.**
>
- 타입 안전성이 필요한 경우에는?

    ```kotlin
    @JvmInline
    value class ValidatedInput(val value: String)
    ```

    - `String`과는 다른 타입으로 인식되므로 컴파일 타임에서 실수 방지 가능하다.

### 인라인 클래스와 타입 별명: 언제 무엇을 사용할까?

- 타입 별명 (`typealias`)
    - 기존 타입을 더 읽기 쉽고 의미 있게 표현할 수 있는 문법적인 도구이다.

    ```kotlin
    typealias ValidatedInput = String
    
    fun save(v: ValidatedInput): Unit = TODO()
    
    fun main() {
    	val rawInput = "needs validating!"
    	save(rawInput)
    }
    ```

    - `ValidatedInput`은 사실상 그냥 `String` 이다.
    - 컴파일러는 구분하지 않는다.
    - 단지 가독성을 높이기 위한 수단이다.
      (**타입 안전성은 전혀 보장되지 않음**!)
- 인라인 클래스 (`@JvmInline value class`)
    - 기존 타입에 의미를 부여한 완전히 새로운 타입이다.
    - 런타임에는 포장 없이 동작하지만, 컴파일러는 다른 타입으로 인식한다.

        ```kotlin
        @JvmInline
        value class ValidatedInput(val s: String)
        
        fun save(v: ValidatedInput): Unit = TODO()
        
        fun main() {
        	val rawInput = "needs validating!"
        	save(rawInput) // ❌ 컴파일 오류!
        }
        ```

        - `String`을 직접 넘기면 에러가 난다.
        - 반드시 `ValidatedInput("...")`처럼 명시적 생성자 호출 필요
        - 컴파일 시 타입이 다르다는 걸 인식해서 실수를 방지할 수 있다.

---

### 요약

- 코틀린 제네릭스는 자바와 매우 유사하다. 제네릭 함수와 클래스를 자바와 비슷하게 선언할 수 있다.
- 자바와 마찬가지로, 제네릭 타입의 타입 인자는 컴파일 시점에만 존재한다.
- 타입 인자는 실행 시점에 지워지므로, 제네릭 타입을 `is` 연산자로 검사할 수 없다.
- 인라인 함수의 타입 파라미터를 `reified`로 표시하면, 실행 시점에 해당 타입을 `is`로 검사하거나 `java.lang.Class` 인스턴스를 얻을 수 있다.
- 변성은 기저 클래스는 같지만 타입 파라미터가 다른 두 제네릭 타입 사이의 상하위 타입 관계가, 타입 인자 간의 상하위 타입 관계에 따라 어떤 영향을 받는지를 명시하는 방법이다.
- 제네릭 클래스의 타입 파라미터가 아웃 위치에서만 사용되는 경우(생산자), `out`으로 표시해 공변적으로 만들 수 있다.
- 공변성의 반대는 반공변성이다. 타입 파라미터가 인 위치에서만 사용되는 경우(소비자), `in`으로 표시해 반공변적으로 만들 수 있다.
- 코틀린의 읽기 전용 `List` 인터페이스는 공변적이다. 따라서 `List<String>`은 `List<Any>`의 하위 타입이다.
- 함수 인터페이스는 첫 번째 타입 파라미터에 대해서는 반공변적이고, 두 번째 타입 파라미터에 대해서는 공변적이다.

  (즉, 함수 타입은 **파라미터 타입에 대해서는 반공변성**, **반환 타입에 대해서는 공변성**을 갖는다.)

  예: `(Animal) -> Int`는 `(Cat) -> Number`의 하위 타입이다.

- 코틀린에서는 제네릭 클래스의 공변성을 선언 지점에서 지정하거나, 구체적인 사용 위치에서 지정할 수 있다.
- 제네릭 클래스의 타입 인자가 어떤 타입인지 알 수 없거나, 그 타입이 중요하지 않을 때는 **스타 프로젝션() 구문**을 사용할 수 있다.
- 타입 별명을 사용하면 긴 제네릭 타입에 대해 더 짧은 이름이나 의미 있는 이름을 부여할 수 있다.

  단, 타입 별명은 컴파일 시점에 원래 타입으로 치환되며, 타입 안전성을 더해주진 않는다.