
## 컬렉션과 시퀀스

### 컬렉션을 Predicate 기준으로 필터링하고 싶다면? `filter`
- Predicate? 조건을 만족하는지 여부를 판단하는 함수형 인터페이스
  - 자바의 함수형 인터페이스 `Predicate<T>`와 유사하게 Boolean 값이 결과인 함수를 뜻한다
  - `(T) -> Boolean` 형태
- `filter`는 컬렉션의 각 요소에 Predicate를 적용하여 조건을 만족하는 요소만 포함된 새로운 컬렉션을 생성한다
  - 그 과정에서 원소를 변환하지는 않고 추출만 하는 역할
```kotlin
val numbers = listOf(1, 2, 3, 4, 5)
val evenNumbers = numbers.filter { it % 2 == 0 }  // [2, 4]
```

`filter` 함수를 실제 까보면 `Iterable` 컬렉션의 확장 함수로 정의되어 있고 List 형태로 반환한다
```kotlin
public inline fun <T> kotlin.collections.Iterable<T>.filter(predicate: (T) -> kotlin.Boolean): kotlin.collections.List<T> { /* compiled code */ }
```

**인덱스도 필터링에 영향이 간다면? `filterIndexed`**
```kotlin
val numbers = listOf(1, 2, 3, 4, 5)
val evenNumbers = numbers.filterIndexed { index, value -> 
    index % 2 == 0 && value > 3 
} // [5]
```

### 컬렉션을 다른 형태로 변환하고 싶다면? `map`
- `map`은 컬렉션의 각 요소에 주어진 함수를 적용하여 새로운 컬렉션을 생성한다
  - 자바의 함수형 인터페이스 `Function<T, R>`와 유사하게 T 타입의 요소를 R 타입으로 변환하는 함수를 넘겨준다.
  - `(T) -> R` 형태
```kotlin
data class Person(val name: String, val age: Int)

fun main() {
    val people = listOf(Person("Alice", 30), Person("Bob", 25))
    
    // 1. 
    println (people.map { it.name }) // ["Alice", "Bob"]
  
    // 2. 멤버 참조 사용 가능 
    println (people.map(Person::name)) // ["Alice", "Bob"]
}
```

`map` 함수를 실제 까보면 `Iterable` 컬렉션의 확장 함수로 정의되어 있고 List 형태로 반환한다
```kotlin
public inline fun <T, R> kotlin.collections.Iterable<T>.map(transform: (T) -> R): kotlin.collections.List<R> { /* compiled code */ }
```

**인덱스도 변환에 영향이 간다면? `mapIndexed`**
```kotlin
val numbers = listOf(1, 2, 3, 4, 5)
val indexedNumbers = numbers.mapIndexed { index, value -> 
    "Index: $index, Value: $value" 
} // ["Index: 0, Value: 1", "Index: 1, Value: 2", ...]
```

### 컬렉션을 누적해서 하나의 값으로 반환하고 싶다면? `reduce`
- 컬렉션의 첫 값을 누적기에 넣은 뒤 전달한 람다가 호출되면서 누적 값과 2번째 원소가 인자로 전달되는 원리
  - 첫 값을 누적기에 넣어야 하기 때문에 빈 리스트에 대해서는 사용할 수 없다
  - 자바의 함수형 인터페이스 `BiFunction<T, U, R>`와 유사하게 T 타입의 요소를 U 타입으로 변환하는 함수를 안자로 넘겨준다
  - `(S, T) -> S` 형태
```kotlin
fun main() {
    val list = listOf(1, 2, 3, 4, 5)
    println(list.reduce { acc, element -> acc + element }) // 15
    println(list.reduce { acc, element -> acc * element }) // 120
}
```

```kotlin
public inline fun <S, T : S> kotlin.collections.Iterable<T>.reduce(operation: (S, T) -> S): S { /* compiled code */ }
```
- 여기서 누적기가 `S`, 요소가 `T`, 결과는 다시 `S`가 된다 -> 반환타입도 `S`
- 즉 첫 값을 `S`에 넣고 다음 값을 `T`에 넣어 람다를 호출하고, 그 결과를 다시 `S`에 넣는 방식
  - 'T'는 'S'의 하위 타입이므로 그냥 'S'라고 생각하면 될 거 같다

### 임의의 시작 값으로부터 컬렉션을 누적해서 하나의 값으로 반환하고 싶다면? `fold`
- `fold`는 `reduce`와 비슷하지만 누적기의 초깃값과 타입을 지정할 수 있다
  - 초기값이 있기에 빈 리스트에 대해서도 사용할 수 있어서 좀 더 안전하다
- 누적기를 초깃값으로 초기화한 뒤 점차 누적해가며 최종 결과를 만드는 방식

```kotlin
data class Item(val name: String, val price: Int)

fun main() {
    val cart = listOf(
        Item("Apple", 1000),
        Item("Banana", 2000),
        Item("Cherry", 3000)
    )
  
    // 부가세 10% 포함 총 가격 계산
    val total = cart.fold(0) { acc, item -> 
        acc + (item.price * 1.1).toInt()
    }
  
    // 가장 싼 아이템과 가장 비싼 아이템
    val (cheapest, mostExpensive) = cart.fold(
        Pair(Item("None", Int.MAX_VALUE), Item("None", Int.MIN_VALUE))
    ) { acc, item ->
        val (cheapest, mostExpensive) = acc
        val newCheapest = if (item.price < cheapest.price) item else cheapest
        val newMostExpensive = if (item.price > mostExpensive.price) item else mostExpensive
        Pair(newCheapest, newMostExpensive)
    }
}
```

### 컬렉션에 Predicate 적용
- 컬렉션의 모든 원소가 어떤 조건을 만족하는지 판단하는 연산

| 함수       | 설명                                              | 반환 값         | 컬렉션을 끝까지 순회할 필요 여부 |
|------------|-------------------------------------------------|-----------------|-----------------------------|
| `all`      | **모든 원소가 조건을 만족하는지** 확인                         | `Boolean`       | 전체 컬렉션을 순회할 필요 있음   |
| `any`      | **하나 이상의 원소가 조건을 만족하는지** 확인                     | `Boolean`       | 조건을 만족하는 원소를 찾으면 순회 종료  |
| `none`     | **모든 원소가 조건을 만족하지 않는지** 확인                      | `Boolean`       | 전체 컬렉션을 순회할 필요 있음   |
| `count`    | **조건을 만족하는 원소의 개수** 반환                          | `Int`           | 전체 컬렉션을 순회할 필요 있음   |
| `find`     | **조건을 만족하는 첫 번째 원소**를 반환  (`firstOrNull()`과 동일) | `T?` (null 가능) | 조건을 만족하는 원소를 찾으면 순회 종료  |


**빈 컬렉션에 Predicate를 적용하면?**
- `any`는 람다를 만족하는 원소가 없는 것이므로 `false`를 반환
- `none`은 `any`를 반전시킨 것으로 람다를 만족하는 원소가 없으므로 `true`를 반환
- `all`은 람다가 무엇이든 관계없이 빈 컬렉션에 대해 항상 `true`를 반환
```kotlin
fun main() {
    val emptyList = listOf<Int>()

    println(emptyList.any { it > 0 }) // false
    println(emptyList.none { it > 0 }) // true
    println(emptyList.all { it > 0 }) // true
}
```

**`count` vs `size`**
- `count`는 조건을 만족하는 원소의 개수를 반환하는 함수이고 `size`는 컬렉션의 크기를 나타내는 속성
```kotlin
data class Person(val name: String, val age: Int)

fun main() {
    val people = listOf(Person("Alice", 30), Person("Bob", 25), Person("Charlie", 35))
    
    // 1. 바로 카운트만 세서 반환하므로 효율적
    println(people.count { it.age > 30 }) // 2
    
    // 2. 중간 리스트가 생성되기에 비효율
    println(people.filter { it.age > 30 }.size) // 2
}
```

### 리스트를 분할해 Pair로 만들고 싶다면? partition
- `partition`은 컬렉션을 두 개의 리스트로 나누어 Pair로 반환한다
  - 첫 번째 리스트는 Predicate를 만족하는 그룹, 두 번째 리스트는 Predicate를 만족하지 않는 그룹이 된다
- `filter`와 `filterNot`을 조합한 것과 동일한 동작이지만 전체 컬렉션을 한번만 순회해서 쌍을 내놓기 때문에 더 효율적
```kotlin
data class User(val userNo: Long, val role: String)

fun main() {
    val users = listOf(
        User(1, "admin"),
        User(2, "user"),
        User(3, "admin"),
        User(4, "guest")
    )
  
    val (admins, others) = users.partition { it.role == "admin" }
  
    println(admins) // [User(userNo=1, role=admin), User(userNo=3, role=admin)]
    println(others) // [User(userNo=2, role=user), User(userNo=4, role=guest)]
}
```

### 리스트를 여러 그룹으로 이뤄진 맵으로 만들고 싶다면? groupBy
- 참/거짓 그룹으로 나누는 `partition`과 달리 `groupBy`는 어떤 특성에 따라 여러 그룹으로 나누어 맵으로 반환한다
- 맵의 키는 그룹화 기준이 되는 속성이고 값은 해당 속성을 가진 원소들의 리스트가 된다
```kotlin
data class User(val userNo: Long, val role: String)

fun main() {
    val users = listOf(
        User(1, "admin"),
        User(2, "user"),
        User(3, "admin"),
        User(4, "guest")
    )
  
    val groupedByRole: Map<String, List<User>> = users.groupBy { it.role }
  
    println(groupedByRole)
    // {admin=[User(userNo=1, role=admin), User(userNo=3, role=admin)], 
    //  user=[User(userNo=2, role=user)], 
    //  guest=[User(userNo=4, role=guest)]}
}
```

멤버 참조를 활용해서 그룹핑할 수도 있다
- 멤버가 아닌 확장함수여도 멤버 참조를 사용해 확장함수에 접근할 수 있다
```kotlin
data class Person(val name: String, val age: Int)

fun main() {
  val people = listOf(
    Person("Alice", 20),
    Person("Bob", 25),
    Person("Ally", 30),
  )

  val grouped = people
      .map(Person::name)    // 멤버 참조로 name 접근
      .groupBy(String::first) // 멤버 참조로 확장함수 first() 접근
  
  println(grouped) // {A=[Alice, Ally], B=[Bob]}
}
```

### 컬렉션을 그룹화 없이 맵으로 변환하고 싶다면? associate
- `associate`는 입력 컬렉션의 각 원소를 키-값 쌍으로 변환하여 맵으로 반환한다
```kotlin
data class Person(val name: String, val age: Int)

fun main() {
    val people = listOf(
        Person("Alice", 30),
        Person("Bob", 25),
        Person("Charlie", 35)
    )
  
    val nameToAge = people.associate { it.name to it.age }
    println(nameToAge) // {Alice=30, Bob=25, Charlie=35}
}
```

**컬렉션의 원소와 다른 어떤 값 사이의 연관을 만들고 싶을 때에는?**
1. 컬렉션의 원래 원소를 키로 사용하고 람다의 결과를 값으로 사용해 맵으로 변환하는 `associateWith`
2. 컬렉션의 원소를 값으로 사용하고 람다의 결과를 키로 사용해 맵으로 변환하는 `associateBy`
```kotlin
data class Person(val name: String, val age: Int)

fun main() {
    val people = listOf(
        Person("Alice", 22),
        Person("Bob", 33),
        Person("Charlie", 22)
    )
  
    val personToAge = people.associateWith { it.age }
    println(personToAge)
    // {Person(name=Alice, age=22)=22, Person(name=Bob, age=33)=33, Person(name=Charlie, age=22)=22}
  
    val ageToPerson = people.associateBy { it.age }
    println(ageToPerson)
    // {22=Person(name=Charlie, age=22), 33=Person(name=Bob, age=33)}
    // 같은 22살인 Alice는 Charlie에 의해 덮어씌워진 모습..!
}
```
- 맵에서 키는 유일해야 하기 때문에 변환 함수가 키가 같은 값을 여러 번 추가하게 되면 마지막 결과가 그 전ㅇ늬 결과를 덮어쓰게 된다


### 가벤 컬렉션일 때 지정한 람다의 결과로 모든 원소를 변경하고 싶다면? replaceAll
```kotlin
fun main() {
    val inputs = mutableListOf("   Alice ", "   BOB  ", "charlie  ")
    inputs.replaceAll { it.trim().lowercase() }

    println(inputs) // [alice, bob, charlie]
}
```

### 가변 리스트의 모든 원소를 똑같은 값으로 바꾸고 싶다면? fill
- 가변 리스트 초기화 시 사용
```kotlin
fun main() {
  val scores = mutableListOf(100, 80, 90, 95, 75)

  // 점수 0으로 초기화
  scores.fill(0)
  println("After reset: $scores") // [0, 0, 0, ..., 0]
}
```

### 컬렉션이 비어있을 경우 기본값을 주고 싶다면? ifEmpty
- `ifEmpty`는 컬렉션이 비어있을 경우 기본값을 생성하는 람다를 제공할 수 있다
```kotlin
fun main() {
    // 사용자 검색 기록 (처음이라 기록 없음)
    val recentSearches = listOf<String>() 

    // 검색 기록 없으면 인기 검색어로 
    val displaySearches = recentSearches.ifEmpty {
        listOf("인기검색어: 코틀린", "ChatGPT 사용법", "면접 질문 정리")
    }

    println("최근 검색어 또는 추천 검색어:")
    displaySearches.forEach { println("- $it") }
}
```

**문자열이 공백인 경우에도 기본값을 주고 싶다면? ifBlank**
- `ifBlank`는 문자열이 비어있거나 공백으로만 이루어진 경우 기본값을 생성하는 람다를 제공할 수 있다
```kotlin
fun main() {
  val userInput = "   "
  val message = userInput
      .ifBlank { "공백 입력입니다." }
      .ifEmpty { "빈 문자열입니다." }

  println(message)    // "공백 입력입니다."
}
```

### 컬렉션의 데이터를 슬라이딩 윈도우를 적용시켜 연속적인 값들로 처리하고 싶을 때? windowed
- `windowed`는 컬렉션을 슬라이딩 윈도우 방식으로 나누어 연속적인 값들로 처리할 수 있게 해준다
  - 주어진 크기만큼의 연속된 원소를 포함하는 리스트를 생성한다

**활용 예제) 사용자 클릭 분석**
```kotlin
data class NetworkPacket(val id: Int, val timestamp: Long, val isLost: Boolean)

fun main() {
    // 네트워크 패킷 기록 (패킷 id, 전송 시간, 손실 여부)
    val packets = listOf(
        NetworkPacket(1, 1000, false),
        NetworkPacket(2, 1005, false),
        NetworkPacket(3, 1010, true), // 손실된 패킷
        NetworkPacket(4, 1015, false),
        NetworkPacket(5, 1020, true), // 손실된 패킷
        NetworkPacket(6, 1025, true), // 손실된 패킷
        NetworkPacket(7, 1030, false)
    )
  
    // 최근 3개의 패킷에서 연속된 손실 패턴을 찾아 알림을 보냄
    val consecutiveLosses = packets
        .windowed(size = 3, step = 1) { window ->
            // 3개의 패킷 중 2개 이상이 손실된 경우
            window.count { it.isLost } >= 2
        }
        .count { it }

    if (consecutiveLosses > 0) {
        println("네트워크 이상 징후 감지: 손실 패킷이 3개 중 2개 이상인 구간이 ${consecutiveLosses}건 존재합니다.")
    } else {
        println("네트워크 상태가 정상입니다.")
    }
  
    // 출력: 경고! 최근 3개의 패킷 중 2개 이상이 손실된 윈도우가 3개 있습니다.
}
```

### 컬렉션의 원소를 특정 기준으로 겹치지 않게 묶고 싶다면? chunked
- `chunked`는 컬렉션을 주어진 크기만큼의 청크로 나누어 리스트로 반환한다
  - 각 청크는 원소를 포함하는 리스트가 되고 마지막 청크는 주어진 크기보다 작을 수 있다

**외부 API로 대량의 ID를 전달해야 하는 상황**
```kotlin
import kotlin.random.Random

fun sendToRemoteApi(batch: List<Int>) {
    val success = Random.nextDouble() > 0.5 // 50% 확률로 성공
    if (!success) throw RuntimeException("알 수 없는 에러 발생!")
    println("Sent batch: $batch")
}

fun sendWithRetry(
    batch: List<Int>,
    maxRetries: Int = 3,
    delayMillis: Long = 1000
): Result<Unit> {
    var attempt = 0
    while (attempt < maxRetries) {
        runCatching {
            sendToRemoteApi(batch)  
        }.onSuccess {
            return Result.success(Unit)
        }.onFailure { e ->
            println("실패 (시도 ${attempt + 1}): ${e.message}")
        }
    
        attempt++
        if (attempt < maxRetries) Thread.sleep(delayMillis)
    }
    return Result.failure(RuntimeException("최대 재시도 초과"))
}


fun main() {
    val allUserIds = (1..50).toList()
    val failedBatches = mutableListOf<List<Int>>()
  
    allUserIds.chunked(10).forEach { batch ->
        val result = sendWithRetry(batch)
        if (result.isFailure) {
            println("요청 최종 실패 청크: $batch")
            failedBatches.add(batch)
        }
    }
  
    println("\n처리 완료. 실패한 배치 수: ${failedBatches.size}")
}
```

- 출력 예시
```text
Sent batch: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
Sent batch: [11, 12, 13, 14, 15, 16, 17, 18, 19, 20]
실패 (시도 1): 알 수 없는 에러 발생!
실패 (시도 2): 알 수 없는 에러 발생!
Sent batch: [21, 22, 23, 24, 25, 26, 27, 28, 29, 30]
Sent batch: [31, 32, 33, 34, 35, 36, 37, 38, 39, 40]
실패 (시도 1): 알 수 없는 에러 발생!
Sent batch: [41, 42, 43, 44, 45, 46, 47, 48, 49, 50]

처리 완료. 실패한 배치 수: 0
```

### 연관이 있는 두 컬렉션을 합치고 싶다면? zip
- `zip`은 두 개의 컬렉션을 결합하여 Pair로 묶은 리스트를 생성한다
  - 두 컬렉션의 크기가 다를 경우 짧은 쪽에 맞춰서 결과가 생성된다
  - 람다를 전달해 출력을 변환할 수 있다
- 중위 표기법으로도 쓸 수 있는데 이 때에는 람다를 전달할 수 없다

**데이터 마이그레이션 시 필드별 변경 내역 추적 예제**
```kotlin
data class Product(val id: Int, val name: String, val price: Double)

fun trackDataChanges(
    oldProducts: List<Product>,
    newProducts: List<Product>
): List<String> {
    return oldProducts
        .zip(newProducts)
        .mapIndexedNotNull { _, (old, new) ->
            val changedFields = buildList {
                if (old.name != new.name) add("name: '${old.name}' → '${new.name}'")
                if (old.price != new.price) add("price: ${old.price} → ${new.price}")
            }
      
            if (changedFields.isNotEmpty()) "Product ID ${old.id}: ${changedFields.joinToString(", ")}"
            else null
        }
}

fun main() {
    val oldProducts = listOf(
        Product(1, "Apple", 100.0),
        Product(2, "Banana", 50.0),
        Product(3, "Cherry", 150.0)
    )
  
    val newProducts = listOf(
        Product(1, "Apple", 110.0),
        Product(2, "Banana", 50.0),
        Product(3, "Cherry", 140.0)
    )
  
    // 데이터 변경 내역 추적
    val changes = trackDataChanges(oldProducts, newProducts)
  
    if (changes.isNotEmpty()) {
        println("변경된 항목들:")
        changes.forEach { println(it) }
    } else {
        println("변경된 데이터가 없습니다.")
    }
}
```

- 출력 예시
```text
변경된 항목들:
Product ID 1: price: 100.0 → 110.0
Product ID 3: price: 150.0 → 140.0
```

### 내포 없이 내포된 컬렉션을 계산하고 싶다면? flatMap
- `flatMap`은 먼저 컬렉션의 각 원소를 파라미터로 주어진 함수를 사용해 변환한 뒤 그 결과를 하나의 리스트로 합친다
- 원소들의 컬렉션의 컬렉션일 때 변환 과정 후 하나의 리스트로 합치고 싶으면 사용하면 된다
  - 단순히 하나의 리스트로 합치고만 싶을 때에는 `flatten`을 사용하면 된다
```kotlin
data class Subject(val name: String, val score: Int)
data class Student(val name: String, val subjects: List<Subject>)

fun main() {
    val students = listOf(
        Student("Alice", listOf(
            Subject("Math", 85),
            Subject("English", 90),
            Subject("Science", 88)
        )),
        Student("Bob", listOf(
            Subject("Math", 92),
            Subject("English", 89),
            Subject("Science", 93)
        )),
        Student("Charlie", listOf(
            Subject("Math", 78),
            Subject("English", 80),
            Subject("Science", 82)
        ))
    )
  
    // 과목별 평균 점수 계산
    val subjectAverage = students
        .flatMap { it.subjects }
        .groupBy { it.name }
        .mapValues { (_, subjects) ->
            subjects.map { it.score }.average()
        }
  
    // 과목별 평균 출력
    subjectAverage.forEach { (name, average) ->
        println("Subject: $name, Average Score: ${"%.2f".format(average)}")
    }
}
```

## 지연 계산 컬렉션 연산: 시퀀스
- 컬렉션 함수를 연쇄적으로 사용하면 매 단계마다 새로운 컬렉션이 생성된다
  - 이로 인해 메모리 사용량이 증가하고 성능이 저하될 수 있다
```kotlin
val numbers = (1..1_000_000)
  .filter { it % 2 == 0 }   // 새로운 컬렉션 생성
  .map { it * 2 }           // 새로운 컬렉션 생성
```

- 이런 문제를 해결하기 위해 코틀린은 시퀀스(Sequence)라는 지연 계산 컬렉션을 제공한다
  - 중간 저장 컬렉션이 생기지 않기 때문에 원소가 많은 경우 성능이 향상된다
```kotlin
val numbers = (1..1_000_000)
  .asSequence()             // 시퀀스로 변환
  .filter { it % 2 == 0 }   // 중간 저장 컬렉션이 생기지 않는다
  .map { it * 2 }           // 중간 저장 컬렉션이 생기지 않는다
  .toList()                 // 최종 결과를 리스트로 변환
```

**Sequence 인터페이스**
```kotlin
public interface Sequence<out T> {
    public abstract operator fun iterator(): kotlin.collections.Iterator<T>
}
```
- 단지 한 번에 하나씩 열거될 수 있는 원소의 시퀀스로 `iterator()` 단 하나의 메서드만 존재한다
- Sequence의 원소는 필요할 때 Lazy하게 지연 계산되기 때문에 중간 저장 컬렉션 없이 연쇄적으로 연산을 수행할 수 있다
  - 위의 예제에서 `map`, `filter` 전부 이 시퀀스에 대해 연산을 수행한 것

### 시퀀스 연산 실행
- 시퀀스 연산은 크게 두 가지로 나뉜다
  - 중간 연산: 다른 시퀀스를 변환하는 연산으로 항상 Lazy하게 실행된다
  - 최종 연산: 시퀀스를 소비하는 연산으로 결과를 반환한다
- 최종 연산이 실행될 때까지 중간 연산은 실행되지 않는다
  - 최종 연산이 호출되면 그때까지의 중간 연산을 모두 실행한다
```kotlin
fun main() {
    val numbers = listOf(1, 2, 3)

    val sequence = numbers.asSequence()
        .map {
            println("Mapping $it")
            it * 2
        }
        .filter {
            println("Filtering $it")
            it > 5
        }
    println("중간 연산만 했을 때는 아무 일도 일어나지 않음")

    val result = sequence.toList() // 최종 연산 실행
    println("결과: $result")
}
```

- 출력 예시
```text
중간 연산만 했을 때는 아무 일도 일어나지 않음
Mapping 1
Filtering 2
Mapping 2
Filtering 4
Mapping 3
Filtering 6
결과: [6]
```
여기서 컬렉션에 대해 체이닝 걸었을 때와 다르게 시퀀스에 대해서는 각 원소를 파이프라인처럼 하나씩 처리한다

**연산의 순서는 성능에 영향을 미친다!**
- `map`과 `filter`를 조합할 때는 순서에 따라 결과는 같아도 성능이 달라질 수 있다
  - `filter`를 먼저 적용하면 불필요한 원소에 대해 `map`을 수행하지 않게 된다
```kotlin
fun main() {
    val numbers = (1..10_000_000).asSequence()

    // 1. filter 먼저 적용
    val result1 = numbers
        .filter { it % 2 == 0 } // 짝수만 필터링: 총 5,000,000개
        .map { it * 2 }         // 짝수에 대해서만 2배: 총 5,000,000개
        .toList()
  
    // 2. map 먼저 적용
    val result2 = numbers
        .map { it * 2 }         // 모든 원소에 대해 2배: 총 10,000,000개
        .filter { it % 2 == 0 } // 짝수만 필터링: 총 15,000,000개
        .toList()
}
```
항상 연쇄적인 연산에서 더 빨리 원소를 제거하는 순서로 연산을 수행하자 

### 시퀀스 생성
1. asSequence()로 이미 있는 컬렉션을 시퀀스로 만들기
2. generateSequence()로 초기값을 기준으로 다음 값을 계산하는 큐칙을 통해 시퀀스 만들기
```kotlin
fun main() {
    val naturalNumbers = generateSequence(1) { it + 1 } // 1씩 증가하는 무한 시퀀스 (Lazy)
    val numbersTo100 = naturalNumbers.takeWhile { it <= 100 } // 1부터 100까지 제한 (Lazy)
    println(numbersTo100.sum()) // 5050 
}
```
- 최종 연산인 `sum()`이 호출되고 나서야 시퀀스가 실제로 1부터 100까지 생성되며 함산된다

**시퀀스 사용처**
- 일반적인 시퀀스 사용처 중 하나는 객체의 조상들로 이뤄진 시퀀스를 만들어내는 것이다.
- 어떤 객체의 조상이 자신과 같은 타입이고 모든 조상의 시퀀스에서 어떤 특성을 알고 싶을 때 사용한다
  - ex) 사람이나 파일 디렉터리 계층 구조 

- 파일의 상위 디렉터리를 뒤지면서 숨김 속성을 가진 디렉터리가 있는지 검사해 해당 파일이 감춰진 디렉터리 안에 있는지 판단하는 예제
```kotlin
import java.io.File
fun File.isInsideHiddenDirectory() =
  generateSequence(this) { it.parentFile }.any { it.isHidden }

fun main() {
  val file = File("/Users/svtk/.HiddenDir/a.txt")
  if (file.isInsideHiddenDirectory()) println("파일은 숨김 디렉터리 안에 있습니다.")
  else println("파일은 숨김 디렉터리 안에 없습니다.")
}
```

- 상품 분류를 표시하는 예제
```kotlin
data class Category(
    val name: String, 
    val parent: Category? = null
)

fun main() {
    // 상품 카테고리 계층 구조 정의
    val root = Category("전체")
    val fashion = Category("패션", root)
    val men = Category("남성", fashion)
    val outerwear = Category("아우터", men)
    val jackets = Category("자켓", outerwear)
  
    val categoryPath = generateSequence(jackets) { it.parent }
      .map { it.name }
      .toList() // 여기서 중간 연산 map과 generateSequence가 모두 실행됨
      .reversed()   // 이건 시퀀스와 무관한 이미 변환된 컬렉션에 대한 연산
  
    println("카테고리 경로: ${categoryPath.joinToString(" > ")}")
    // 출력: 카테고리 경로: 전체 > 패션 > 남성 > 아우터 > 자켓
}
```
