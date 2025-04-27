# 5. 람다를 사용한 프로그래밍

### 람다식

- 람다는 다른 함수에 넘길 수 있는 작은 코드 조각을 의미한다.
- 람다를 사용하면 공통 코드 구조를 라이브러리 함수로 쉽게 뽑아낼 수 있다.
- 함수를 값처럼 사용할 수 있다.

### object 선언으로 리스너 구현

```kotlin
button.setOnClickListener(object : onClickListener {
    override fun onClick(v: View) {
        println("I was clicked!")
    }
})
```

코틀린에서 이 코드를 아래와 같이 람다로 바꿀 수 있다.

```kotlin
button.setOnClickListener {
    println("I was clicked!")
}
```

---

### 람다와 컬렉션 (기본구현)

```kotlin
data class Person(val name: String, val age: Int)

fun findTheOldest(people: List<Person>) {
    var maxAge = 0
    var theOldest: Person? = null
    for (person in people) {
        if (person.age > maxAge) {
            maxAge = person.age
            theOldest = person
        }
    }
    println(theOldest)
}

fun main() {
    val people = listOf(Person("Alice", 29), Person("Bob", 31))
    findTheOldest(people)
}

/*
Person(name=Bob, age=31)
 */
```

- 컬렉션을 for문으로 순회하면서 가장 나이가 많은 사람을 찾는 코드이다.
- 이러한 방법은 가독성이 떨어지고, 코드가 길어지며, 유지보수가 힘들다.

```kotlin
fun main() {
    val people = listOf(Person("Alice", 29), Person("Bob", 31))
    println(people.maxByOrNull { it.age })
}

/*
Person(name=Bob, age=31)
 */
```

```java
// 자바에서는 아래와 같이 작성할 수 있다.
List<Person> people = List.of(new Person("Alice", 29), new Person("Bob", 31));
System.out.println(people.stream().maxByOrNull(it -> it.getAge()));
```

- 코틀린에서는 `maxByOrNull` 함수를 사용하여 가장 나이가 많은 사람을 찾을 수 있다.
- 중괄호로 둘러쌓인 부분에서는 인자로 받아 비교에 사용할 값을 반환해야한다.
- 인자가 하나일 경우에는 `it`이라는 암시적 이름을 사용한다.
- 이 예제에서는 컬렉션의 원소가 `Person` 객체이므로 `it`은 `Person` 객체가 된다.

### 람다식의 문법

```kotlin
{ x: Int, y: Int -> x + y}
```

- 중괄호로 둘러싸인 부분이 람다식이다.
- 화살표 (->) 가 인자 목록과 람다 본문을 구분해준다.
- 자바에서는 인자가 2개 이상일 경우, 괄호`(`와 `)`로 감싸야 한다.
- 코틀린에서는 인자가 0개일 경우에도 괄호를 생략할 수 있다.

```kotlin
fun main() {
    val sum = { x: Int, y: Int -> x + y }
    println(sum(1, 2))
}
```

- 위와 같이 람다식은 변수로 저장될 수 있고, 호출할 수 있다.
- 자바에서 위 람다식을 표현하려면 `BiFunction` 같은 `Functional Interface`를 사용해야 한다.

```kotlin
// #1
fun main() {
    val people = listOf(Person("Alice", 29), Person("Bob", 31))
    val names = people.joinToString(
        separator = " ",
        transform = { p: Person -> p.name },
    )
    println(names)
}

/*
Alice Bob
 */

// #2
fun main() {
    val people = listOf(Person("Alice", 29), Person("Bob", 31))
    val names = people.joinToString(separator = " ") { p: Person -> p.name }
    println(names)
}

/*
Alice Bob
 */
```

- 함수 호출 인자로 람다식을 넘길때, 람다식를 넣어줄 수 있지만,
- 함수 호출 인자의 마지막이 람다식인 경우, 블록 형태로 넘겨줄 수 있다.

```kotlin
// 모두 같은 코드
people.maxByOrNull({ p: Person -> p.age })
people.maxByOrNull() { p: Person -> p.age }
people.maxByOrNull { p: Person -> p.age }
people.maxByOrNull { p -> p.age }
people.maxByOrNull { it.age }
people.maxByOrNull(Person::age)
```

- 람다식의 인자도 마찬가지로 타입 추론이 되므로, 파라미터 타입을 명시할 필요가 없다.
- it을 사용하면, 코드를 아주 간단하게 만들어주지만, 남용하면 안 된다.
- 람다가 내포 되는 경우, 직접 명시하여 사용하는게 좋다.

```kotlin
val getAge = { p: Person -> p.age }
people.maxByOrNull(getAge)
```

- 람다를 변수에 저장할 때에는 파라미터 타입을 추론할 문맥이 존재하지 않기 때문에, 파라미터 타입을 명시해야한다.

### 변수 캡처

```kotlin
fun printMessagesWithPrefix(messages: Collection<String>, prefix: String) {
    messages.forEach {
        println("$prefix $it")
    }
}

fun main() {
    val errors = listOf("403 Forbidden", "404 Not Found")
    printMessagesWithPrefix(errors, prefix = "Error:")
}

/*
Error: 403 Forbidden
Error: 404 Not Found
*/
```

- 람다식은 자신이 선언된 위치의 변수를 캡처할 수 있다.
- 자바와 다른점은 코틀린 람다 안에서 파이널 변수가 아닌 변수에 접근할 수 있다는 점이다.
- 따라서, 람다식 안에서 변수를 변경할 수 있다.

```kotlin
fun printProblemCounts(responses: Collection<String>) {
    var clientErrors = 0
    var serverErrors = 0

    responses.forEach {
        if (it.startsWith("4")) {
            clientErrors++
        } else if (it.startsWith("5")) {
            serverErrors++
        }
    }

    println("$clientErrors client errors, $serverErrors server errors")
}

fun main() {
    val responses = listOf("200 OK", "418 I'm a teapot", "500 Internal Server Error")
    printProblemCounts(responses)
}

/*
1 client errors, 1 server errors
 */
```
- 코드에서 볼 수 있듯이 람다식에서 람다 밖 함수에 있는 파이널이 아닌 변수 clientErrors와 serverErrors를 캡처하여 사용하고 있다.
- 캡처한 변수가 존재하는 람다를 저장해서 함수가 끝난 뒤에 실행해도 람다의 본문 코드는 여전히 캡처한 변수를 읽거나 쓸 수 있다.

```kotlin
class Ref<T>(var value: T)

fun main() {
    val counter = Ref(0)
    val inc = { counter.value++ }
}
```

- 파이널 변수를 캡처한 경우에는 람다 코드를 변수 값과 함께 저장하지만,
- 파이널이 아닌 변수를 캡처한 경우에는 변수를 특별한 래퍼로 감싸서 나중에 변경하거나 읽을 수 있게 한다음, 래퍼에 대한 참조를 람다 코드와 함께 저장한다.

### 멤버 참조

```kotlin
people.maxByOrNull(Person::age)
people.maxByOrNull { person: Person -> person.age }
```

- 함수로 넘기려는 코드가 이미 함수로 선언된 경우, 멤버참조를 이용해 값을 넣어줄 수 있다.
- 자바 8과 마찬가지로 이중 콜론`(::)`을 사용한다.
- 같은 역할을 하는 람다식을 더 간략하게 표현해준다.

### 단일 추상 메서드 (Single Abstract Method, SAM)

```java
/* 자바 8 이전 */
button.setOnClickListener(new OnClickListener() {
    @Override
    public void onClick(View v) {
        System.out.println("I was clicked!");
    }
});

/* 자바 8 이후 */
button.setOnClickListener(v -> System.out.println("I was clicked!"));
```
```kotlin
/* 코틀린 */
button.setOnClickListener { v: View -> println("I was clicked!") }
```

- 자바 8부터는 인터페이스를 람다식으로 표현할 수 있다.
- 코틀린에서는 단순히 람다를 전달하면 된다.
- `OnClickListener` 인터페이스는 인터페이스 안에 추상 메서드가 단 하나뿐인<br>
단일 추상 메서드 인터페이스이기 때문에 람다로 표현할 수 있다.
- 이런 인터페이스를 `Fuctional Interface`나 `단일 추상 메서드 (SAM)`이라고 한다.

```kotlin
postponeComputation(1000, object : Runnable {
    override fun run() {
        println(42)
    }
})

postponeComputation(1000) { println(42) }
```

- 위 두 코드는 같은 구현을 나타내지만, 내부적으로는 차이가 있다.
- 첫번째 코드는 명시적으로 객체를 선언했기 때문에 매번 호출할 때마다 새 인스턴스가 생긴다.
- 하지만, 두번째 코드는 람다가 자신이 정의된 함수의 변수에 접근하지 않는다면, 함수가 호출될 때마다 람다에 해당하는 익명 객체가 재사용된다.

```kotlin
fun handleComputation(id: String) {
    postponeComputation(1000) { 
        println(id) 
    }
}
```
- 자신이 정의된 함수의 변수 `id`를 캡처하기 때문에 `handleComputation` 호출마다 새 람다 객체가 생성된다.

### 코틀린에서의 SAM 인터페이스 정의: fun interface
```kotlin
fun interface IntCondition {
    fun check(i: Int): Boolean // 추상 메서드
    fun checkString(s: String) = check(s.toInt()) // 비추상 메서드
    fun checkChar(c: Char) = check(c.digitToInt()) // 비추상 메서드
}

fun main() {
    val isOdd = IntCondition { it % 2 != 0 }
    println(isOdd.check(1))
    println(isOdd.checkString("2"))
    println(isOdd.checkChar('3'))
}

/*
true
false
true
 */
```
- 코틀린에서는 SAM 인터페이스를 정의할 때 `fun interface` 키워드를 사용한다.
- 추상 메서드는 한개만 존재 해야하지만, 비추상 메서드는 여러개 존재해도 된다.
- 
```kotlin
fun checkCondition(i: Int, condition: IntCondition): Boolean {
    return condition.check(i)
}

fun main() {
    println(checkCondition(1) { it % 2 != 0})

    val isOdd: (Int) -> Boolean = { it % 2 != 0}
    println(checkCondition(1, isOdd))
}

/*
true
true
 */
```

- 람다를 직접 구현할 수 있고, 구현된 람다를 인자로 넣어줄 수도 있다.

### 수신객체 지정 람다: with, apply, also

- 코틀린에서는 수신객체 지정 람다를 사용하여 객체를 생성하고 초기화하는 코드를 간결하게 작성할 수 있다.
- 수신객체를 명시하지 않고 람다의 본문 안에서 다른 객체의 메서드를 호출할 수 있게 하는 람다를 `수신 객체 지정 람다`라고 부른다.

### with

- `with`는 수신객체를 인자로 받아서 람다를 실행하는 함수이다.

```kotlin
// StringBuilder를 사용하여 문자열을 생성하는 예제
fun alphabet(): String {
    val result = StringBuilder()
    for (letter in 'A'..'Z') {
        result.append(letter)
    }
    result.append("\nNow I know the alphabet!")
    return result.toString()
}

fun main() {
    println(alphabet())
}

/*
ABCDEFGHIJKLMNOPQRSTUVWXYZ
Now I know the alphabet!
 */

// with를 사용해서 리팩토링
fun alphabet(): String {
    val result = StringBuilder()
    return with(result) {
        for (letter in 'A'..'Z') {
            this.append(letter)
        }
        this.append("\nNow I know the alphabet!")
        this.toString()
    }
}

// 불필요한 StringBuilder 변수 제거
fun alphabet(): String = with(StringBuilder()) {
    for (letter in 'A'..'Z') {
        append(letter)
    }
    append("\nNow I know the alphabet!")
    toString()
}
```

- `with` 함수는 첫 번째 인자로 받은 객체를 두 번째 인자로 받은 람다의 수신 객체로 만든다.
- `this.`을 없애고 매서드나 프로퍼티 이름만 사용해 접근할 수도 있다.

### apply

- `apply`는 수신객체를 인자로 받아서 람다를 실행하고, 수신객체를 반환하는 함수이다.
- alphabet() 함수를 `apply`를 사용하여 리팩토링하면 아래와 같다.

```kotlin
fun alphabet(): String = StringBuilder().apply {
    for (letter in 'A'..'Z') {
        append(letter)
    }
    append("\nNow I know the alphabet!")
}.toString()
```

- `StringBuilder()`를 통해 `apply` 함수를 호출했기 때문에 `apply`의 반환값은 수신객체인 `StringBuilder` 객체이다.
- 인스턴스를 만들면서 즉시 프로퍼티 중 일부를 초기화해야 하는 경우 `apply`가 유용하다.

### also

- `also`도 `apply`와 마찬가지로 수신 객체를 받으며, 그 수신 객체에 대해 어떤 동작을 수행한 후 수신 객체를 반환한다.
- 주된 차이는 a`lso`의 람다 안에서는 수신 객체를 인자로 참조한다는 점이다.
- 이런 특징으로 인해, 수신 객체를 인자로 받는 동작을 실행할 때 `also`가 유용하다.

```kotlin
fun main() {
    val fruits = listOf("Apple", "Banana", "Cherry")
    val uppercaseFruits = mutableListOf<String>()

    val reversedLongFruits = fruits
        .map { it.uppercase() }
        .also { uppercaseFruits.addAll(it) } 
        .filter { it.length > 5 }
        .also { println(it) } // [BANANA, CHERRY]
        .reversed()

    println(uppercaseFruits) // [APPLE, BANANA, CHERRY]
    println(reversedLongFruits) // [CHERRY, BANANA]
}
```
