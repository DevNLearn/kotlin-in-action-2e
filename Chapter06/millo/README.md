
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



