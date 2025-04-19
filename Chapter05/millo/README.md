
## 람다

### 람다란?
- 다른 함수에 넘길 수 있는 작은 코드 조각을 의미하는 람다 표현식
- 어떤 동작을 변수에 저장하거나 인자로 전달하고 싶을 때 예전 자바에서는 익명 내부 클래스를 사용했다.
  - 상당히 번거롭다
    ```java
    Runnable runnable = new Runnable() {
        @Override
        public void run() {
            System.out.println("Hello");
        }
    };
    ```
- 함수를 값처럼 다루는 람다식을 사용하면 클래스의 인스턴스를 넘기는 방법 대신 직접 함수를 넘겨줄 수 있다
    ```java
    Runnable runnable = () -> System.out.println("Hello");
    ```

**함수형 프로그래밍 정리**
1. 일급 시민 함수
   - 함수를 값으로 다룰 수 있어 변수에 저장하거나 인자로 전달하거나 반환할 수 있다
2. 불변성
   - 객체 생성 이후 내부 상태가 변하지 않는 불변 객체로 설계할 수 있다.
3. 순수 함수 (사이드 이펙트 X)
   - 같은 입력에 대해 항상 같은 출력을 반환하는 참조 투명성을 가지기에 값으로 다룰 수 있는 것.
   - 순수하다고 표현하며 부수 효과가 없다 (동작이 예측이 가능하다)
   - 외부로부터 영향을 받지 않음과 동시에 내부에서 외부에도 영향을 주지 않는다
    ```kotlin
    fun main() {
        val numbers = listOf(1, 2, 3, 4, 5)
        val factor = 2
    
        // 람다
        fun multiply(list: List<Int>, factor: Int) = list.map { it * factor }
    
        val result = multiply(numbers, factor)
    
        // 원본은 변하지 않음(불변성) + 항상 같은 결과를 반환(순수함수)
        println("Original numbers: $numbers") // [1, 2, 3, 4, 5]
        
        // 새로운 객체를 반환
        println("Multiplied numbers: $result") // [2, 4, 6, 8, 10]
    }

**람다 리팩토링 예제**
```kotlin
// 클릭을 처리하는 OnClickListener 인터페이스 인스턴스를 전달
button.setOnClickListener(object: OnclickListener {
    override fun onClick(v: View) { println("Button clicked!") }
})
```

- 인스턴스를 선언하지 않고 람다를 통해 동작 자체를(함수를) 넘겨 리팩토링
```kotlin
button.setOnClickListener { println("Button clicked!") }
```

### 람다와 컬렉션
- 컬렉션을 다룰 때 대부분 패턴화된 작업을 하게 되는데 코틀린은 람다를 적용해 컬렉션을 더 편리하게 다루는 라이브러리를 제공한다

**가장 나이 많은 사람을 찾는 예제 리팩토링**
```kotlin
data class Person(val name: String, val age: Int)

fun findTheOldest(people: List<Person>) {
    var maxAge = 0
    var oldest: Person? = null
    
    for (person in people) {
        if (person.age > maxAge) {
            maxAge = person.age
            oldest = person
        }
    }
    println(oldest)
}

fun main() {
    val people = listOf(Person("Alice", 30), Person("Bob", 25), Person("Charlie", 35))
    findTheOldest(people) // Person(name=Charlie, age=35)
}
```

- 코틀린의 라이브러리 함수 중 `maxByOrNull`을 사용해 리팩토링
```kotlin
data class Person(val name: String, val age: Int)

fun main() {
    val people = listOf(Person("Alice", 30), Person("Bob", 25), Person("Charlie", 35))
    println(people.maxByOrNull { it.age }) // Person(name=Charlie, age=35)
}
```

`maxByOrNull`을 살펴보자
```kotlin
inline fun <T, R : Comparable<R>> kotlin.collections.Iterable<T>.maxByOrNull(selector: (T) -> R): T? { /* compiled code */ }
```
- 제네릭 `T`로 받아 `R`로 반환하는 데 이 때 `R`은 `Comparable`을 상속받은, 즉 비교할 수 있는 타입이어야 한다
- `Iterable<T>`를 확장한 함수로 `T`를 요소로 가지는 반복 가능한 컬렉션에 대해 동작한다 (List, Set 등)
- 유일한 인자인 `selector`는 `T`를 받아 `R`을 반환하는 람다인데 `T`의 요소를 `R`로 변환하는 함수이다
- 리스트가 비어있을 경우 `null`을 반환하도록 반환타입이 `T?`이므로 호출할 때에도 `?.` 세이프 콜을 사용해야 한다
- 전달하는 인자는 람다의 결과값으로 비교할 수 있는 값이어야 한다 (여기서는 나이 `{ it.age }`)
- 실제 동작하는 코드(최댓값찾기)는 라이브러리로 바이너리로 제공되어 소스 코드에서는 볼 수 없네요

### 람다 문법
- 코틀린 람다식은 항상 중괄호(`{}`)로 감싸져 있으며 `->`로 인자 목록과 본문을 구분한다

**`run`을 사용해 인자로 받은 람다 바로 실행해주기**
```kotlin
fun main() {
    // 1.
    val result = run { 1 + 2 }
    println(result) // 3

    // 2.
    println(run { 1 + 2 }) // 3

    // 3.
    run { println(1+2) }  // 3

    // 람다 출력
    run { println {1 + 2} } // () -> kotlin.Int
}
```

**람다 호출 여러 방법**
```kotlin   
data class Person(val name: String, val age: Int)
fun main() {
    val people = listOf(Person("Alice", 30), Person("Bob", 25), Person("Charlie", 35))

    // 1. 정식 호출
    people.maxByOrNull({ p: Person -> p.age })
    
    // 2. 함수 호출 시 맨 뒤에 있는 인자가 람다식이면 그 람다를 괄호 밖으로 분리 가능
    people.maxByOrNull() { p: Person -> p.age }
    
    // 3. 람다가 유일한 인자이면 호출 시 빈 괄호 생략 가능
    people.maxByOrNull { p: Person -> p.age }
    
    // 4. 컴파일러가 추론할 수 있는 람다의 인자 타입 생략 가능 (확장 함수이기에 항상 컬렉션 원소 타입과 동일하므로 추론 가능)
    people.maxByOrNull { p -> p.age }
}
```
- 람다가 유일한 인자일 때에는 괄호 밖으로 빼는 게 좋고, 둘 이상일 때에는 괄호 안에 넣는 게 좋다

**이전 챕터의 `joinToString()` 리팩토링**
```kotlin
data class Person(val name: String, val age: Int)
fun main() {
    val people = listOf(Person("Alice", 30), Person("Bob", 25))
    val name = people.joinToString(
        separator = ", ",
        transform = { p: Person -> p.name }
    )
    println(name) // Alice, Bob
}
```

- `joinToString()`의 `transform` 인자는 람다식이므로 괄호 밖으로 추출 가능
```kotlin
data class Person(val name: String, val age: Int)
fun main() {
    val people = listOf(Person("Alice", 30), Person("Bob", 25))
    val name = people.joinToString(", ") { p: Person -> p.name }
    println(name) // Alice, Bob
}
```

