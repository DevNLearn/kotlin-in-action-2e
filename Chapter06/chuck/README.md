# 6. 컬렉션과 시퀀스

- 함수형 프로그래밍 스타일은 컬렉션을 다룰때 여러가지 장점 제공
- 컬렉션을 직접 순회하면서 데이터를 하나로 만드는 방법과 비교할 때 함수형 방식을 사용하면, 일반적인 연산을 일관성 있게 표현할 수 있음

### 원소 제거와 변환: filter, map

```kotlin
fun main() {
    val list = listOf(1, 2, 3, 4)
    println(list.filter { it % 2 == 0 }) // [2, 4]
}
```

- `filter`는 컬렉션의 원소를 필터링하는 함수
- 원소들을 필터링만 하고, 그 과정에서 원소를 변환하지는 않는다.
- 자바와 다르게 `stream`을 사용하지 않고도 컬렉션을 필터링할 수 있음

```kotlin
fun main() {
    val list = listOf(1, 2, 3, 4)
    println(list.map { it * it }) // [1, 4, 9, 16]
}
```
- `map`은 컬렉션의 원소를 변환하는 함수
- 원소들을 변환만 하고, 그 과정에서 원소를 필터링하지는 않는다.

```kotlin
fun main() {
    // 인덱스가 짝수면서 3보다 큰 원소만 선택하는 예제
    val numbers = listOf(1, 2, 3, 4, 5, 6, 7)
    val filtered = numbers.filterIndexed { index, element ->
        index % 2 == 0 && element > 3
    }
    println(filtered) // [5, 7]

    // 인덱스와 원소의 합계
    val mapped = numbers.mapIndexed { index, element -> index + element }
    println(mapped) // [1, 3, 5, 7, 9, 11, 13]
}
```

### 컬렉션 값 누적: reduce, fold

- 컬렉션 정보를 종합하는데 사용된다.
- 원소로 이뤄진 컬렉션을 받아서 하나의 값으로 축약하는 함수
- 누적기(accumulator)와 원소를 인자로 받아서 새로운 값을 반환하는 람다식을 사용한다.
- `reduce`는 초기값을 제공하지 않고, `fold`는 초기값을 제공한다.

```kotlin
// reduce 예제
fun main() {
    val list = listOf(1, 2, 3, 4)
    println(list.reduce { acc, element -> acc + element }) // 10
}
```
- `reduce`는 컬렉션의 첫 번째 원소를 누적기로 사용하기 때문에, `reduce` 내부의 람다는 3번 호출된다.
- 처음에는 `1과 2를 더한 3이 누적기로 사용`되고, 그 다음에는 `3과 3을 더한 6이 누적기로 사용`되고, 마지막으로 `6과 4를 더한 10이 반환`된다.

```kotlin
// fold 예제
class Person(val name: String, val age: Int)

fun main() {
    val people = listOf(
        Person("Alice", 30),
        Person("Bob", 25),
        Person("Charlie", 35)
    )

    val folded = people.fold("") { acc, person -> acc + person.name + " " }
    println(folded) // Alice Bob Charlie
}
```
- 위 예제에서는 초기값을 `빈문자열("")`로 시작했기 때문에, 첫 실행에서 acc는 빈문자열이 들어온다.
- `fold`는 초기값을 제공하기 때문에, `fold` 내부의 람다는 4번 호출된다.

```kotlin
class Person(val name: String, val age: Int)

fun main() {
    val list = listOf(1, 2, 3, 4)
    println(list.runningReduce { acc, element -> acc + element }) // [1, 3, 6, 10]

    val people = listOf(
        Person("Alice", 30),
        Person("Bob", 25),
        Person("Charlie", 35)
    )
    println(people.runningFold("") { acc, person -> acc + person.name + " " }) // [, Alice , Alice Bob , Alice Bob Charlie ]
}
```

- running이 붙은 함수들을 사용하면 모든 누적 값들을 뽑아낼 수 있다.

### 모든 원소가 참인지 확인하기: all

```kotlin
class Person(val name: String, val age: Int)

fun main() {
    val people = listOf(Person("Alice", 30), Person("Bob", 25), Person("Charlie", 35))
    println(people.all { it.age > 20 }) // true
}
```

### 원소 중 참인게 하나라도 있는지 확인하기: any

```kotlin
class Person(val name: String, val age: Int)

fun main() {
    val people = listOf(Person("Alice", 30), Person("Bob", 25), Person("Charlie", 35))
    println(people.any { it.age < 20 }) // false
}
```

### 원소 중 참인게 하나라도 없는지 확인하기 (모두 거짓): none

```kotlin
class Person(val name: String, val age: Int)

fun main() {
    val people = listOf(Person("Alice", 30), Person("Bob", 25), Person("Charlie", 35))
    println(people.none { it.age < 20 }) // true (모두가 20살 미만이 아니므로 참!)
}
```

### 원소의 개수 세기: count

```kotlin
class Person(val name: String, val age: Int)

fun main() {
    val people = listOf(Person("Alice", 30), Person("Bob", 25), Person("Charlie", 35))
    println(people.count { it.age <= 30 }) // 2 (30살 이하인 사람의 수)
}
```

### 처음으로 만족하는 원소 찾기: find

```kotlin
data class Person(val name: String, val age: Int)

fun main() {
    val people = listOf(Person("Alice", 30), Person("Bob", 25), Person("Charlie", 35))
    println(people.find { it.age == 35 }) // Person(name=Charlie, age=35)
}
```

### 리스트를 분할해 리스트의 쌍으로 만들기 (참인 그룹과 거짓인 그룹으로 나누는 함수): partition

```kotlin
data class Person(val name: String, val age: Int)

fun main() {
    val people = listOf(Person("Alice", 30), Person("Bob", 25), Person("Charlie", 35))
    println(people.partition { it.age >= 30 }) 
    // ([Person(name=Alice, age=30), Person(name=Charlie, age=35)], [Person(name=Bob, age=25)])
}

// 30살 이상인 사람들: [Person(name=Alice, age=30), Person(name=Charlie, age=35)]
// 30살 미만인 사람들: [Person(name=Bob, age=25)]
```

### 리스트를 여러 그룹으로 이뤄진 맵으로 바꾸기: groupBy

- 맵으로 리턴돼서 좋은 것 같다. (ex. DB로 조회한 유저 정보 `office_no`끼리 그룹화)

```kotlin
data class Person(val name: String, val age: Int)

fun main() {
    val people = listOf(
        Person("Alice", 30),
        Person("Bob", 25),
        Person("Charlie", 35),
        Person("Chuck", 35),
    )
    println(people.groupBy { it.age })
    // {30=[Person(name=Alice, age=30)], 25=[Person(name=Bob, age=25)], 35=[Person(name=Charlie, age=35), Person(name=Chuck, age=35)]}
}
```

### 원소를 그룹화하지 않으면서 컬렉션으로부터 맵을 만들기: associate

- `key`가 중복될 경우, 마지막 원소의 값으로 덮어씌워진다.

```kotlin
data class Person(val name: String, val age: Int)

fun main() {
    val people = listOf(Person("Alice", 29), Person("Bob", 31), Person("Alice", 39))
    val nameToAge = people.associate { it.name to it.age }
    println(nameToAge) // {Alice=39, Bob=31}
    println(nameToAge["Alice"]) // 39
}
```

### 컬렉션의 원소와 다른 어떤 값 사이의 연관을 만들어내기: associateWith, associateBy

- `associateWith`는 원소를 키로 사용하고, `associateBy`는 원소를 값으로 사용한다.

```kotlin
data class Person(val name: String, val age: Int)

fun main() {
    val people = listOf(Person("Alice", 29), Person("Bob", 31), Person("Alice", 39))
    println(people.associateWith { it.age }) 
    // {Person(name=Alice, age=29)=29, Person(name=Bob, age=31)=31, Person(name=Alice, age=39)=39}
    println(people.associateBy { it.age })
    // {29=Person(name=Alice, age=29), 31=Person(name=Bob, age=31), 39=Person(name=Alice, age=39)}
}
```

### 가변 컬렉션의 원소 변경: replaceAll, fill

- `replaceAll`은 원소를 변환하는 함수이고, `fill`은 원소를 대체하는 함수이다.
- 아래의 예제에서는 `replaceAll`은 원소를 대문자로 변환하고, `fill`은 원소를 `(redacted)` 문자열로 대체한다.
- `List` 에서는 호출 불가능

```kotlin
fun main() {
    val names = mutableListOf("Martin", "Samuel")
    println(names) // [Martin, Samuel]

    names.replaceAll { it.uppercase() }
    println(names) // [MARTIN, SAMUEL]

    names.fill("(redacted)")
    println(names) // [(redacted), (redacted)]
}
```

### 컬렉션 나누기: chunked

- `chunked`는 컬렉션을 주어진 크기로 나누어 새로운 리스트를 만든다.
- 배치 연산 같이 1000개씩 나눠서 작업을 실행시킬때 좋을듯(?)

```kotlin
fun main() {
    val temperatures = listOf(27.7, 29.8, 22.0, 35.5, 19.1)
    println(temperatures.chunked(3)) // [[27.7, 29.8, 22.0], [35.5, 19.1]]
}
```

### 컬렉션 나누기: windowed

- `windowed`는 window 크기 만큼의 인덱스를 1씩 증가해가며 슬라이딩한다.

```kotlin
fun main() {
    val temperatures = listOf(27.7, 29.8, 22.0, 35.5, 19.1)
    println(temperatures.windowed(3)) // [[27.7, 29.8, 22.0], [29.8, 22.0, 35.5], [22.0, 35.5, 19.1]]
}
```

### 컬렉션 합치기: zip

- `zip`은 두 개의 컬렉션을 합쳐서 새로운 리스트를 만든다.
```kotlin
fun main() {
    val names = listOf("Alice", "Bob", "Charlie")
    val ages =  listOf(22, 31, 31, 44, 0)

    println(names.zip(ages)) // [(Alice, 22), (Bob, 31), (Charlie, 31)]
    println(names.zip(ages) { name, age -> Person(name, age)}) 
    // [Person(name=Alice, age=22), Person(name=Bob, age=31), Person(name=Charlie, age=31)]
}
```

### 내포된 컬렉션 원소 처리: flatMap

- `flatMap`은 컬렉션의 원소를 변환하는 함수이다.

```kotlin
data class Book(val title: String, val authors: List<String>)

val library = listOf(
    Book("Kotlin in Action", listOf("Dmitry Jemerov", "Svetlana Isakova")),
    Book("Effective Java", listOf("Joshua Bloch")),
    Book("Clean Code", listOf("Robert C. Martin")),
    Book("The Pragmatic Programmer", listOf("Andrew Hunt", "David Thomas")),
    Book("Design Patterns", listOf("Erich Gamma", "Richard Helm", "Ralph Johnson", "John Vlissides")),
)

fun main() {
    val authors = library.flatMap { it.authors }
    println(authors)
    // [Dmitry Jemerov, Svetlana Isakova, Joshua Bloch, Robert C. Martin, Andrew Hunt, David Thomas, Erich Gamma, Richard Helm, Ralph Johnson, John Vlissides]
}
```

### 지연 계산 컬렉션 연산: 시퀀스

- 시퀀스는 컬렉션을 지연 계산하는 방법이다.
- 앞서 봤던 함수는 컬렉션을 즉시 생성한다.
- 그렇기 때문에 앞서 본 함수에서는 리스트가 여러개 만들어져 수백만개의 원소를 가진 컬렉션의 경우, 효율성이 떨어진다.
- 자바의 `stream`과 비슷하게 `sequence`는 lazy하게 동작한다.

```kotlin
data class Book(val title: String, val authors: List<String>)

val library = listOf(
    Book("Kotlin in Action", listOf("Dmitry Jemerov", "Svetlana Isakova")),
    Book("Effective Java", listOf("Joshua Bloch")),
    Book("Clean Code", listOf("Robert C. Martin")),
    Book("The Pragmatic Programmer", listOf("Andrew Hunt", "David Thomas")),
    Book("Design Patterns", listOf("Erich Gamma", "Richard Helm", "Ralph Johnson", "John Vlissides")),
)

fun main() {
    val result = library.asSequence()
        .flatMap { it.authors }
        .filter { it.startsWith("D") }
        .toList()

    println(result)
}
```

### 시퀀스 만들기

```kotlin
fun main() {
    val result = generateSequence(0) { it + 1 }
        .takeWhile { it <= 100 }
        .sum()
    println(result) // 5050
}
```
