## 6장

### 컬렉션

- 함수형 프로그래밍 스타일은 컬렉션을 다룰때 여러 장점을 제공한다.
- 대부분의 작업에 대해 표준 라이브러리가 제공하는 함수를 활용하고 람다를 전달하여 함수를 커스텀화할 수 있다.

---

#### filter
- 자바에서도 사용하듯, 컬렉션을 순회하며 `true` 인 객체들만 반환한다.
- 객체에서 주어진 조건에 맞는 원소들로 이루어진 새 컬렉션을 만들수 있지만, 원소를 변환하지는 않는다.
```kotlin
fun main() {
    val list = listof(1, 2, 3, 4)
    // 2, 4
    println(list.filter {it % 2 === 0 })
}
```

#### map
- 컬렉션을 순회하며 각 원소에 주어진 함수값을 적용하고, 결과값을 새로운 컬렉션에 모은다
```kotlin
fun main() {
    val list = listOf(1, 2, 3, 4)
    // [1, 4, 9, 16]
    println(list.map { it * it })
}
```

- 멤버 참조를 사용할 수도 있다
```kotlin
val people = listOf(
        Person(name = "babi", age = 20),
        Person(name = "bob", age = 25)
)
// ["babi", "bob"]
people.map(Person::name)
```

- `filterKeys`, `filtersValues` 들은 `Map` 에서 사용하게되는 함수들이며, 생긴것 그대로 각각 `Key`에서 필터링, `Value`에서 필터링 하냐가 다르다
```kotlin
val map = mapOf("a" to 1, "bb" to 2, "ccc" to 3)
val byKey   = map.filterKeys   { it.length > 1 }   // {"bb"=2, "ccc"=3}
val byValue = map.filterValues { it % 2 == 0 }     // {"bb"=2}
```

---

#### reduce

- 첫 번째 원소를 누적값으로 삼고 컬렉션의 나머지를 순차 계산한다.
- 빈 컬렉션에서는 호출 시 에러 발생
- 만약 하나밖에 존재하지 않는다면, 해당 원소가 그대로 반환
```kotlin
listOf(1, 2, 3, 4).reduce { acc, e -> acc + e }  // 10
listOf(1, 2, 3, 4).reduce { acc, e -> acc * e }  // 24
```
---

#### fold
- 사용자가 명시적으로 초기 누적값을 지정할 수 있다.
- 빈 컬렉션도 초기값을 반환
```kotlin
listOf("A", "B", "C")
  .fold("Letters:") { acc, s -> "$acc $s" }    // "Letters: A B C"
```

---

#### runningReduce, runningFold
- 최종결과값을 포함한 중간 계산값들을 리스트형태로 반환한다
```kotlin
listOf(1, 2, 3).runningReduce { acc,e -> acc+e }  // [1, 3, 6]
listOf(1, 2, 3).runningFold(0) { acc,e -> acc+e } // [0, 1, 3, 6]
```

---

### 컬렉션의 술어

#### all
- 모든 원소가 해당 술어를 만족하는지 여부
- 빈 컬렉션은 `true`
```kotlin
val above27 = { p: Person -> p.age <= 27 }

fun main() {
    val people = listOf(
            Person(name = "Alice", age = 30),
            Person(name = "Babi", age = 25)
    )
    // false
    println(people.all(above27))
}
```
---

#### any
- 해당 술어를 하나라도 만족하는지 여부
- 빈 컬렉션은 `false`
```kotlin
fun main() {
    // true
    println(people.any(above27))
}
```

---

#### none
- 자바에서도 많이 사용했듯, 특정 조건 앞에 `!` 을 붙여 부정문을 만드는건 아주 가독성이 떨어진다.
- 그래서 `any` 의 반대인 `none` 이 존재한다
- 조건을 하나도 만족하지 않는지 여부
```kotlin
fun main() {
    val list = listOf(1, 2, 3)
    println(!list.any {it == 4})    // hmmmmm...
    
    println(list.none {it == 4})    // good!
}
```

---
#### count
- 해당 조건을 만족하는 원소의 개수 반환
```kotlin
fun main() {
    val people = listOf(
            Person(name = "Alice", age = 30),
            Person(name = "Babi", age = 25)
    )
    // 1
    println(people.count(above27))
}
```
- `size` 와 `count` 둘다 동일한 결과값을 반환하지만, `size` 에서는 조건에 맞는 모든 원소가 들어가는 중간 컬렉션이 생긴다
- 하지만 `count` 는 단순 원소의 갯수만 저장하기때문에 단순히 갯수만 있어야한다면 `count`가 훨씬 효율적이다.


---

#### partition
- 컬렉션을 조건에 따라 두개의 그룹으로 나눈다.
- 첫번째그룹은 조건을 만족하는 그룹, 두번째 그룹은 조건을 만족하지 않는 그룹
```kotlin
val (above27Group, under27Group) = people.partition(above27)
```

---

#### groupBy
- 특정 조건에 따라 여러 그룹으로 그룹화
```kotlin
val people = listOf(
        Person(name = "A", age = 31), 
        Person(name = "B", age = 29), 
        Person(name = "C", age = 31)
)
// {31=[A,C], 29=[B]}
val byAge = people.groupBy { it.age }
```

---

#### associate
- Pair를 반환하는 람다로 Map을 생성
```kotlin
fun main() {
    val people = listOf(
            Person(name = "Joe", age = 22),
            Person(name = "Mary", age = 31)
    )
    val nameToAge = people.associate { it.name to it.age }
    // {Joe=22, Mary=31}
    println(nameToAge)
}
```

---

#### associateBy, associateWith
- 컬렉션 원소를 값으로 두고, 키는 사용자가 지정하는 `associateBy`
- 원소를 키로두고, 람다 결과를 값으로 두는 `associateWith`

```kotlin
val list = listOf("one", "three", "five")
val byFirst = list.associateBy { it.first() }
// {o="one", t="three", f="five"}
val lengthMap = list.associateWith { it.length }
// {"one"=3, "three"=5, "five"=4}
```

---

#### replaceAll, fill

- 가변컬렉션인 `MutableList` 에 람다로 얻은 결과로 컬렉션의 모든 원소를 변경하는 `replaceAll`
- 가변리스트에 모든 원소를 똑같은 값으로 바꾸는 `fill`

```kotlin
fun main() {
    val names = mutableMapOf("Hi", "Babi")
    println(names) // [Hi, Babi]
    names.replaceAll { it.uppercase() }
    println(names) // [HI, BABI]
    names.fill("dot")
    println(names) // [dot, dot]
}
```

---

#### ifBlank, ifEmpty
- **컬렉션**과 **문자열**에서 비어있는지 여부를 판별하고, 비어있다면 특정 처리를 하는 `ifEmpty`
- 문자열에서 공백으로만 이루어져있는지 여부를 판별하고, 비어있다면 특정 처리를 하는 `ifBlank`
```kotlin
fun main() {
    val blankName = "         "
    val name = "Babi"
    println(blankName.ifEmpty { "noname" })  // 
    println(blankName.ifBlank { "noname" })  // noname
    println(name.ifBlank { "noname" })       //  Babi
} 
```

---

#### chunked, windowed, zip
- 크기가 `n` 씩 겹치지 않는 청크로 분할하는 `chunked`
  - 지정한 청크 사이즈만큼 자르고, 남아서 모자라면 모자란대로 마지막에 삽입
- 크기가 `n` 인 슬라이딩 윈도우 생성하는 `windowed`
- 두개의 컬렉션을 동일한 인덱스끼리 쌍으로 묶는 `zip`

```kotlin
listOf(1, 2, 3, 4).windowed(3) // [[1,2,3], [2,3,4]]
listOf(1, 2, 3, 4, 5).chunked(2) // [[1,2], [3,4], [5]]
listOf("a", "b").zip(listOf(1 ,2 ,3)) // [(a,1), (b,2)]
```


---
#### flatMap, flatten

- 컬렉션의 각 원소를 파라미터로 주어진 함수를 사용해 변환하고, 변환한 결과를 하나의 리스트로 합치는 `flatMap`
- 변환과정 없이 단순히, 리스트의 리스트를 하나의 리스트로 바꾸고싶을때 사용하는 `flatten`



> **더 많은 표준 라이브러리는 IDE 또는 표준라이브러리 참조 문서를 보자**
>  - https://kotlinlang.org/api/latest/jvm/stdlib/

---
### 시퀀스
#### 왜 시퀀스?
- 컬렉션 API는 `map->filter` 연쇄 시 **매 단계마다** 임시 리스트를 만든다
- 시퀀스(Sequence) 사용 시 중간 결과를 저장하지 않고, 필요할 때만 하나씩 처리한다(Lazy 계산)
- 대용량 데이터·무한 수열 처리에 유리하다
```kotlin
people
        .asSequence()       // 원본 컬렉션을 시퀀스로 변환
        .map(Person::name)
        .filter { it.startWith("A") }
        .toList()           // 시퀀스를 다시 리스트로 변환
```

---

#### 시퀀스 연산 실행
- 시퀀스에서 최종연산(결과를 반환하는것) 전까지의 연산은 항상 지연 계산된다.
```kotlin
fun main() {
    println(
            listOf(1, 2, 3)
                    .asSequence()
                    .map { 
                        print("map($it) ")
                        it * it
                    }.filter {
                        print("filter($it) ")
                        it % 2 == 0
                    }
    )
}
```
- 위 코드는 아무 내용도 출력되지 않는다.
- 최종연산이 아직 호출되기 전이라 결과를 얻을 필요가 없기 때문.
- 컬렉션에서는 `map` -> `filter` 순서라면 즉시계산이 되어 모든 결과값이 임시 컬렉션에 저장을 하며 각 단계가 완료된 이후 다음 단계로 넘어가고, 시퀀스는 원소 단위로 순차적으로 진행한다.


---

#### 시퀀스 생성
- `generateSequence()` 시퀀스를 생성한 뒤, 최종연산 `sum()` 을 수행한다.
- 최종연산이 수행하기 전까지는 시퀀스는 계산되지 않는다.
```kotlin
fun main() {
    val naturalNumbers = generateSequence(0) { it + 1 }
    val numbersTo100 = naturalNumbers.takeWhile { it <= 100 }
    // 5050
    println(numbersTo100.sum())
}
```
