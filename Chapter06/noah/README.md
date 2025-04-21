# 컬렉션과 시퀀스
- 함수형 스타일로 컬렉션 다루기
- 시퀀스: 컬렉션 연산을 지연시켜 수행하기

컬렉션 접근 패턴을 표준 라이브러리 함수와 람다를 조합해서 코드를 더 간결하게 만들 수 있다

시퀀스를 통해 컬렉션 연산의 부가 비용을 줄일 수 있다.

## 1. 컬렉션에 대한 함수형 API

### filter와 map

`filter`는 컬렉션을 순회 하면서 주어진 람다가 true를 반환하는 원소들만 모은다.

```kotlin
fun main() {
    val list = listOf(1, 2, 3, 4)
    println(list.filter { it % 2 == 0 })
    // [2, 4]
}
```

`map`은 입력 컬렉션의 원소를 변환할 수 있게 해준다.

```kotlin
fun main() {
    val people = listOf(Person("Alice", 29), Person("Bob", 31))
    println(people.map { it.name })
    // [Alice, Bob]
}
```

멤버 참조를 사용할 수도 있다

```kotlin
people.map(Person::name)
```

원소의 값 뿐 아니라 인덱스도 사용하고 싶으면 `filterIndexed`와 `mapIndexed` 를 사용하면 된다.

```kotlin
fun main() {
    val numbers = listOf(1, 2, 3, 4, 5, 6, 7)
    val filtered = numbers.filterIndexed { index, element ->
        index % 2 == 0 && element > 3
     }
    println(flitered) // [5, 7]
    
    val mapped = numbers.mapIndexed { index, element -> 
        index + element
    }
    println(mapped) // [1, 3, 5, 7, 9, 11, 13]
}
```

### 컬렉션 값 누적: reduce와 fold

`reduce`와 `fold`는 컬렉션의 정보를 종합하는 데 사용된다.

`reduce`를 사용하면 컬렉션의 첫 번째 값을 누적기(accumulator)에 넣은 후 그 후 람다가 호출되면서 누적 값과 2번째 원소가 인자로 전달된다.

```kotlin
fun main() {
    val list = listOf(1, 2, 3, 4)
    println(list.reduce { acc, element -> 
        acc + element
    }) // 10
    println(list.reduce { acc, element -> 
        acc * element
    }) // 24
}
```

`fold` 함수는 첫 번째 원소를 누적 값으로 시작하는 대신, 임의의 시작 값을 선택할 수 있다.

```kotlin
fun main() {
    val people = listOf(
        Person("Alex", 29)
        Person("Natalia", 28)
    )
    val folded = people.fold("") { acc, person ->
        acc + person.name
    }
    println(folded) // AlexNatalia
}
```

중간 단계의 모든 누적 값을 뽑아내고 싶다면 `runningReduce`와 `runningFold` 를 사용할 수 있다.

```kotlin
fun main() {
    val list = listOf(1, 2, 3, 4)
    val summed = list.runnigReduce { acc, element ->
        acc + element
    }
    println(summed) // [1, 3, 6, 10]
    
    val multiplied = list.runningReduce { acc, element ->
        acc * element
    }
    println(multiplied) // [1, 2, 6, 24]
    val people = listOf(
        Person("Alex", 29)
        Person("Natalia", 28)
    )
    println(people.runningFold("") { acc, person ->
        acc + person.name
    }) //[, Alex, AlexNatalia]
    
}
```

### 컬렉션에 술어 적용: all, any, none, count, find

`all` , `any`, `none` 는 컬랙션의 모든 원소가 어떤 조건을 만족하는 판단하는 연산이다.

`all`은 모든 원소가 조건을 만족하는지 판단할 때 사용한다.

```kotlin
val canBeInClub27 = { p: Person -> p.age <= 27 }
fun main() {
    val people = listOf(Person("Alice", 27), Person("Bob", 31))
    println(people.all(canBeInClub27)) //false
}
```

`any`는 만족하는 원소가 하나라도 있는지 판단할 때 사용한다.

```kotlin
fun main() {
    println(people.any(canBeInClub27)) //true
}
```

`none`은 만족하는 원소가 하나도 없는지 판단할 때 사용한다.

```kotlin
fun main() {
    val list = listOf(1, 2, 3)
    println(list.none { it == 4 }) //true
}
```

`count`는 만족하는 원소의 개수를 알고 싶을 때 사용한다.

```kotlin
fun main() {
    val people = listOf(Person("Alice", 27), Person("Bob", 31))
    println(people.count(canBeInClub27)) // 1
}
```

`find` 는 만족하는 원소를 하나 찾고 싶을 때 사용한다. `find`는 원소가 전혀 없는 경우 `null`을 반환한다.

```kotlin
fun main() {
    val people = listOf(Person("Alice", 27), Person("Bob", 31))
    println(people.find(canBeInClub27))
    // Person(name=Alice, age=27)
}
```

### 리스트를 분할해 리스트의 쌍으로 만들기: partition

`partition`은 조건을 만족하는 그룹과 그렇지 않은 그룹으로 나누어 준다.

```kotlin
fun main() {
    val people = listOf(
        Person("Alice", 26),
        Person("Bob", 29),
        Person("Carol", 31)
    )
    val (comeIn, stayOut) = people.partition(canBeInClub27)
    println(comeIn) // [Person(name=ALice, age=26)]
    println(stayOut) // [Person(name=Bob, age=29), Person(name=Carol, age=31)]
}
```

### 리스트를 여러 그룹으로 이뤄진 맵으로 바꾸기: groupBy

`groupBy`는 특정 조건에 따라 분류할 수 있다.

```kotlin
fun main() {
    val people = listOf(
        Person("Alice", 31),
        Person("Bob", 29),
        Person("Carol", 31)
    )
    println(people.groupBy { it.age })
    // {31=[Person(name=Alice, age=31), Person(name=Carol, age=31)],
    //  29=[Person(name=Bob, age=29)]}
}
```

### 컬렉션을 맵으로 변환: associate, associateWith, associateBy

`asscociate`는 원소를 그룹화하지 않으면서 컬렉션으로 부터 맵을 만들어 낸다.

```kotlin
fun main() {
    val people = listOf(Person("Joe", 22), Person("Mary", 31))
    val nameToAge = people.associate { it.name to it.age }
    println(nameToAge) // {Joe=22, May=31}
    println(nameToAge["Joe"]) //22
}
```

컬렉션의 원소와 다른 어떤 값 사이의 연관을 만들어낼때 `associateWith`와 `asccociateBy` 를 사용한다.

`associateWith`는 컬랙션의 원소를 키로 사용하고 `associateBy`는 컬렉션의 원소를 맵의 키로 사용한다.

```kotlin
fun main() {
    val people = listOf(
        Person("Joe", 22),
        Person("Mary", 31),
        Person("Jamie", 22)
    )
    
    val personToAge = people.asscociateWith { it.age}
    println(personToAge)
    // {Person(name=Joe, age=22)=22, Person(name=Mary, age=31)=31,
    // Person(name=Jamie, age=22)=22}
    
    val ageToPerson = people.associateBy{ it.age }
    println(ageToPerson)
    // {22=Person(name=Joe, age=22), 31=Person(name=Mary, age=31),
    // 22=Person(name=Jamie, age=22)}
}
```

### 가변 컬렉션의 원소 변경: replcaeAll, fill

`replaceAll` 함수를 `MutableList` 에 적용하면 지정한 람다로 모든 원소를 변경하고 `fill` 함수를 사용하면 가변 리스트의 모든 원소를 똑같은 값으로 바꾼다.

```kotlin
fun main() {
    val names = mutableListOf("Martin", "Samuel")
    println(names)
    
    names.replaceAll { it.uppercase() }
    println(names) // [MARTIN, SAMUEL]
    names.fill("(redacted)")
    println(names) // [(redacted), (redacted)]
}
```

### 컬렉션의 특별한 경우 처리: ifEmpty

`ifEmpty` 함수는 아무 원소가 없는지 검증하고 이를 통해 기본값을 생성하는 람다를 제공할 수 있다.

```kotlin
fun main() {
    val empty = emptyList<String>()
    val full = listOf("apple", "orange", "banna")
    println(empty.ifEmpty { listOf("no", "values", "here")})
    // [no, vallues, here]
}
```

### 컬렉션 나누기: chunked와 windowed

컬레션의 데이터가 어떤 계열 정보를 표현할 때 데이터를 연속적인 시간의 값 들로 처리할 수 있다.

`windowed`함수를 통해 슬라잉 윈도우를 생성할 수 있다.

```kotlin
val temperatures = listOf(27.7, 29.8, 22.0, 35.5, 19.1)
fun main() {
    println(temperatures.windowed(3))
    // [[27.7, 29.8, 22.0], [29.8, 22.0, 35.5], [22.0, 35.5, 19.1]]
}
```

`chunked`는 컬렉션을 서로 겹치지 않는 부분으로 나눌때 사용한다. 람다를 전달하여, 출력을 변환시킬 수 있다.

```kotlin
fun main() {
    println(temperatures.chunked(2)) // [[27.7, 29.8], [22.0, 35.5], [19.1]]
    println(temperatures.chunked(2) { it.sum() }) // [57.5, 57.5, 19.1]
}
```

### 컬렉션 합치기: zip

`zip` 함수를 사용해 두 컬렉션에서 같은 인덱스에 있는 원소들의 쌍으로 리스트를 만들 수 있다. 대응하는 원소가 없는 경우 해당 원소는 무시된다.

```kotlin
fun main() {
    val names = listOf("Joe", "Mary", "Jamie")
    val age = listOf(22, 31, 31, 44, 0)
    println(names.zip(ages))
    // [(Joe, 22), (Mary, 31), (Jamie, 31)]
    println(names.zip(ages) { name, age -> Person(name, age) })
    // [Person(name=Joe, age22), Person(name=Mary, age=31),
    // Person(name=Jamie, age=31)]
}
```

`zip` 함수도 중위 표기법으로 호출할 수 있지만, 주우이 표기법을 쓸 때는 람다를 사용할 수 없다.

```kotlin
println(names zip ages)
// [(Joe, 22), (Mary, 31), (Jamie, 31)]
```

### 내포된 컬렉션의 원소 처리: flatMap과 flatten

```kotlin
class Book(val title: String, val authors: List<String>)
val libray = listOf(
    Book("Kotlin in Action", listOf("Isakova", "Elizarov", "Aigner", "Jemerov")),
    Book("Atomic Kotlin", listOf("Eckel", "Isakova")),
    Book("Thre Three-Body Problem", listOf("Liu"))
)
```

`flatMap` 함수는 컬렉션의 각 원소를 주어진 함수를 사용해 변환(map) 후 변환한 결과를 하나의 리스트로 합친다.

```kotlin
val authors = library.flayMap { it.authors }
println(authors)
// [Isakova, Elizarov, Aigner, Jemerov, Eckel, Isakova, Liu]
```

## 2. 지연 계산 컬렉션 연산: 시퀀스

`map` 이나 `filter` 컬렉션 함수들을 연쇄적으로 호출하는 경우 결과 컬렉션을 즉시 생성한다.

시퀀스를 사용하면 중간 임시 컬렉션을 사용하지 않고 컬렉션 연산을 연쇄적으로 사용할 수 있다.

```kotlin
people
    .asSequence()
    .map(Person::name)
    .filter { it.startsWith("A") }
    .toList()
```

코틀린 지연 계싼 시퀀스는 `Sequence` 인터페이스에서 시작하며 인터페이스 안에는 `iterator` 라는 하나의 메서드가 들어있다.

`Sequence` 의 장점은 중간 처리 결과를 저장할 컬렉션을 만들지 않고도 연산을 연쇄적으로 적용해서 연쇄적인 연산을 효율적으로 수행할 수 있다. `Sequence`를 리스트로 만들 때는 `toList` 를 사용한다.

### 시퀀스 연산 실행: 중간 연산과 최종 연산

시퀀스에 대한 연산은 `중간(intermediate)`연산과 `최종(terminal)` 연산으로 나뉜다. 중간 연산은 다른 시퀀스를 반환하고 최종 연산은 결과를 반환한다.

![image.png](attachment:2eb09332-1a0f-4f0a-aaa1-5909c2111d50:image.png)

```kotlin
fun main() {
    println(
        listOf(1, 2, 3, 4)
            .asSequence()
            .map {
                print("map($it) ")
            }.filter {
                print("filter($it) ")
                it % 2 == 9
            }.toList()
    )
}
```

최종 연산을 호출하면 연기 됐던 모든 계산이 수행된다.

![image.png](attachment:de90a9df-1e4d-4e6e-a01f-6e01bbaa38de:image.png)

### 시퀀스 만들기

시퀀스를 만든 방법으로 `asSequence()` 를 호출하거나 `generateSequence` 함수를 사용하는 방법이 있다.

```kotlin
fun main() {
    val naturalNumber = generateSequence(0) { it + 1 }
    val numberTo100 = naturalNumbers.takeWhile { it <= 100 }
    println(numberTo100.sum()) // 5050
}
```