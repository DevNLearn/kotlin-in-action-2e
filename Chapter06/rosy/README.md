# 6.컬렉션과 시퀀스

Kotlin의 컬렉션 API는 자바의 컬렉션을 기반으로 하면서도, 람다를 활용한 함수형 스타일을 적극적으로 지원한다. 이 챕터에서는 컬렉션을 다루는 다양한 함수형 도구들과, 성능 향상을 위한 `Sequence` 사용법, null 처리 및 읽기 전용 컬렉션 등 **효율적이고 타입 안전한 컬렉션 처리 방법**을 익힐 수 있다.

---

## 6.1 함수형 스타일의 컬렉션 API

Kotlin은 컬렉션을 간결하게 다룰 수 있도록 **고차 함수(map, filter 등)** 를 제공합니다. 이는 컬렉션의 각 요소를 연산하고 새로운 컬렉션을 반환하는 방식이다.

### map, filter

```kotlin
val list = listOf(1, 2, 3, 4)
val doubled = list.map { it * 2 }       // [2, 4, 6, 8]
val evens = list.filter { it % 2 == 0 } // [2, 4]

```

### 조합 사용 예

```kotlin
val people = listOf(Person("Alice", 29), Person("Bob", 31))
val namesOfAdults = people
    .filter { it.age > 30 }
    .map { it.name } // Bob

```

> 컬렉션 연산은 단계마다 새 리스트를 만들므로 중간 결과가 많아지면 성능에 영향을 줄 수 있다.
>

### reduce vs fold

- `reduce`: 첫 번째 요소를 초기값으로 삼아 누적 계산
- `fold`: 사용자가 초기값을 지정해 누적 계산

```kotlin
val nums = listOf(1, 2, 3, 4)
val sum = nums.reduce { acc, i -> acc + i }       // 10
val folded = nums.fold(10) { acc, i -> acc + i }  // 20

```

> reduce는 리스트가 비어있으면 예외가 발생하지만, fold는 초기값을 제공하므로 빈 리스트에도 안전하게 동작한다.
>

### all, any, none, count, find

컬렉션의 모든 원소가 어떤 조건을 만족하는지 판단하는 연산이다.

- `all`: 모든 원소가 만족하는지 검사
- `any`: 만족하는 원소가 하나라도 있는지 검사
- `count`: 조건을 만족하는 원소 개수 반환
- `find` / `firstOrNull`: 조건을 만족하는 첫 원소 반환

```kotlin
val canBeInClub27 = { p: Person -> p.age <= 27 }
val people = listOf(Person("Alice", 27), Person("Bob", 31))

println(people.all(canBeInClub27)) // false
println(people.any(canBeInClub27)) // true
println(people.count(canBeInClub27)) // 1
println(people.find(canBeInClub27)) // Person("Alice", 27)
```

### partition

조건을 기준으로 컬렉션을 둘로 나눈다. 결과는 Pair로 반환되며, 첫 번째는 조건을 만족하는 요소들의 리스트, 두 번째는 나머지다.

```kotlin
val (comeIn, stayOut) = people.partition(canBeInClub27)

```

### groupBy

조건을 기준으로 컬렉션을 그룹화합니다. 반환값은 Map이며, 키는 조건, 값은 리스트이다.

```kotlin
val grouped = listOf("apple", "apricot", "banana", "cantaloupe").groupBy { it.first() }

```

> groupBy는 키를 기준으로 여러 값을 그룹화하며, 다대일 관계를 생성한다.
>

### associate, associateBy, associateWith

Map을 생성할 때 사용한다.

- `associate`: Pair(key, value) 형태로 직접 지정
- `associateBy`: key는 지정, value는 원소 전체
- `associateWith`: value는 지정, key는 원소 전체

```kotlin
val people = listOf(Person("Joe", 22), Person("Mary", 31))
val nameToAge = people.associate { it.name to it.age }
val ageToPerson = people.associateBy { it.age }
val personToAge = people.associateWith { it.age }
```

### replaceAll, fill

- `replaceAll`: 기존 리스트의 값을 새로운 값으로 교체
- `fill`: 리스트의 모든 값을 동일한 값으로 채움

```kotlin
val names = mutableListOf("Martin", "Samuel")
names.replaceAll { it.uppercase() }
names.fill("(redacted)")

```

### ifEmpty, ifBlank

- `ifEmpty`: 문자열이 비어있을 때 기본값 제공
- `ifBlank`: 문자열이 비어있거나 공백뿐일 때 기본값 제공

```kotlin
val blank = " "
val name = "J. Doe"
println(blank.ifEmpty { "(unnamed)" }) // " "
println(blank.ifBlank { "(unnamed)" }) // "(unnamed)"
println(name.ifBlank { "(unnamed)" }) // "J. Doe"

```

> ifBlank는 공백 문자열까지 처리하며, 텍스트 입력 검증 시 유용하다.
>

### chunked vs windowed

| 함수 | 설명 | 예시 |
| --- | --- | --- |
| `chunked(n)` | n개 단위로 잘라 리스트 생성 | "KotlinTest".chunked(3) → ["Kot", "lin", "Tes", "t"] |
| `windowed(n)` | 슬라이딩 윈도우 (중첩 포함 가능) | "abcd".windowed(2) → ["ab", "bc", "cd"] |

### zip

두 리스트를 Pair로 묶어주는 함수로, 연관된 데이터를 함께 처리할 때 사용한다.

```kotlin
val names = listOf("Joe", "Mary", "Jamie")
val ages = listOf(22, 31, 31)
val zipped = names.zip(ages) // [(Joe, 22), (Mary, 31), (Jamie, 31)]

```

### flatten과 flatMap

- `flatten`: 중첩 리스트를 1차원으로 펼침
- `flatMap`: 변환과 펼침을 동시에 처리

```kotlin
val nested = listOf(listOf(1, 2), listOf(3, 4))
nested.flatten() // [1, 2, 3, 4]

val strings = listOf("abc", "def")
strings.flatMap { it.toList() } // [a, b, c, d, e, f]

```

> 🔍 flatMap은 map().flatten()과 동일한 기능을 하나로 처리한다.
>

---

## 6.2 지연 컬렉션 연산: Sequence

Kotlin의 `Sequence`는 **지연 연산(lazy evaluation)** 을 지원하여, 각 단계의 결과를 중간 컬렉션으로 저장하지 않고 하나씩 처리할 수 있는 컬렉션 처리 방식이다. 특히 **대규모 컬렉션**, **여러 중간 연산이 연결된 경우**, **연산 순서가 중요한 경우**에 성능상 이점을 가진다.

### 일반 컬렉션 처리 vs Sequence 처리

```kotlin
val numbers = listOf(1, 2, 3, 4)
val result = numbers.asSequence()
    .map { println("map($it)"); it * 2 }
    .filter { println("filter($it)"); it % 2 == 0 }
    .toList()
```

- 일반 컬렉션 처리: 전체 map → 전체 filter → 결과 생성
- Sequence 처리: 요소 하나씩 map → filter → toList 로 전달됨

> asSequence() 를 호출하지 않으면 각 연산은 중간 컬렉션을 생성하여 메모리를 더 많이 사용한다.
>

### 시퀀스는 언제 사용할까?

- 요소 개수가 매우 많거나
- map, filter 같은 중간 연산이 많을 때
- 요소별로 순차적으로 처리하고 싶을 때

### 시퀀스 구성 흐름

1. `asSequence()`로 변환 (혹은 `sequence { ... }` 사용 가능)
2. 중간 연산 연결: map, filter, take 등 (지연됨)
3. 최종 연산 호출: `toList()`, `count()`, `find()` 등 → 연산 실행됨

### 시퀀스와 루프 비교 예시

```kotlin
val list = listOf(1, 2, 3, 4, 5, 6)

// 일반 컬렉션
val result1 = list.filter { it > 2 }.map { it * 2 }.take(2)

// 시퀀스
val result2 = list.asSequence()
    .filter { it > 2 }
    .map { it * 2 }
    .take(2)
    .toList()
```

> take, first, find 같은 연산은 시퀀스에서 특히 효율적으로 작동함. 필요한 수만큼만 처리하고 종료하므로 불필요한 연산을 줄일 수 있다.
>

### 시퀀스 생성 방식

- `asSequence()`: 기존 컬렉션을 지연 처리용으로 변환
- `generateSequence(seed) { next }`: 시드값으로부터 무한 시퀀스 생성
- `sequence { yield(...) }`: 사용자 정의 시퀀스 빌더

```kotlin
val naturalNumbers = generateSequence(0) { it + 1 }
val evens = naturalNumbers.filter { it % 2 == 0 }
    .take(10)
    .toList()
```

> 무한 시퀀스를 다룰 때는 take(n) 같은 종료 조건을 반드시 사용해야 한다.
>

---

## 요약

- Kotlin은 반복문 없이도 컬렉션을 다루는 다양한 **표준 라이브러리 함수**를 제공한다.
- `filter`, `map`으로 컬렉션을 **조건에 맞게 걸러내거나 변환**할 수 있다.
- `reduce`, `fold`를 사용하면 **하나의 결과값으로 누적**할 수 있다.
- `associate`, `groupBy`를 통해 리스트를 **Map 형태로 재구조화**할 수 있다.
- `chunked`, `windowed`, `zip` 등은 컬렉션을 **하위 그룹이나 쌍으로** 처리하는 데 유용하다.
- `all`, `any`, `none` 같은 **Boolean 반환 함수**로 조건 만족 여부를 쉽게 판단할 수 있다.
- 중첩된 컬렉션은 `flatten`, `flatMap`으로 **한 층 펼치거나 동시에 변환**할 수 있다.
- `Sequence`를 사용하면 **중간 컬렉션 없이 지연 계산**이 가능하여 **성능이 향상**된다.