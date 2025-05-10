# 컬렉션과 시퀀스
## 컬렉션에 대한 함수형 API
### 원소 제거와 변환: filter, map
* filter : 원소 거르기
* map : 변환
```kotlin
data class Person(val name: String, val age: Int)

// 나이가 30살 이상인 사람들 거르기
fun main() {
    val people = listOf(Person("Alice", 29), Person("Bob", 31))
    println(people.filter {it.age >= 30})
    // [Person(name=Bob, age=31)]
}

fun main() {
    val people = listOf(Person("Alice", 29), Person("Bob", 31))
    //사람들의 이름의 리스트를 출력
    println(people.map{it.name})
    println(people.map(Person::name)) //멤버참조
    // [Alice, Bob]

    //30살 이상인 사람의 이름 출력
    println(people.filter {it.age >= 30}.map(Person::name))
    // [Bob]
    
    //가장 나이가 많은 사람의 이름
    people.filter { //사람수대로 비교하게 됨
        val oldest = people.maxByOrNull(Person::age)
        it.age == oldest?.age
    }
    
    val maxAge = people.maxByOrNull(Person::age)?.age
    people.filter {it.age == maxAge} //한 번만 비교
}
```
원소의 값 뿐 아니라 인덱스에 따라서도 달라진다면, **filterIndexed**, **mapIndexed를** 사용하면 됨
```kotlin
fun main() {
    val numbers = listOf(1, 2, 3, 4, 5, 6, 7)
    val filtered = numbers.filterIndexed{index, element -> 
        index % 2 == 0 && element > 3
    }
    println(filtered)
    // [5, 7]
    val mapped = numbers.mapIndexed{index, element -> 
        index + element
    }
    println(mapped)
    // [1, 3, 5, 7, 9, 11, 13]
}
```
filter와 map을 **맵**에 적용할 수도 있음
```kotlin
fun main() {
    val numbers = mapOf(0 to "zero", 1 to "one")
    println(numbers.mapValues { it.value.uppercase() })
    // {0=ZERO, 1=ONE}
}
```
맵의 경우 **filterKeys**, **mapKeys와** **filterValues**, **mapValues** 존재함

### 컬렉션 값 누적 : reduce, fold
* reduce : 컬렉션의 첫 번째 값을 누적기에 넣고, 람다가 호출되며 누적 값과 2번째 원소가 인자로 전달됨
* fold : 개념적으로 reduce와 비슷하지만, 컬렉션 첫 번째 값을 누적 값으로 시작하는 대신, 임의의 시작 값을 선택할 수 있음
```kotlin
//reduce
fun main() {
    val list = listOf(1, 2, 3, 4)
    println(list.reduce {acc, elem -> acc + elem}) // 1+2 3+3 6+4
    // 10
    println(list.reduce {acc, elem -> acc * elem}) // 2, 6, 24
    // 24
}

//fold
fun main() {
    val people = listOf(
        Person("Alex", 29),
        Person("Natalia", 28)
    )
    val folded = people.fold("") {acc, elem -> acc + person.name} // ""가 누적기 시작값
    println(folded)
    // AlexNatalia
}
```
중간 단계의 모든 누적값을 뽑고 싶으면 **runningReduce**, **runningFold** 사용 가능  
얘네는 **리스트** 반환, reduce, fold는 하나의 결과만 반환  
```kotlin
fun main() {
    val list = listOf(1, 2, 3, 4)
    val summed = list.runningReduce {acc, elem -> acc + elem}
    println(summed)
    // [1, 3, 6, 10]
    val multiplied = list.runningReduce {acc, elem -> acc * elem}
    println(summed)
    // [1, 2, 6, 24]
    val people = listOf(
        Person("Alex", 29),
        Person("Natalia", 28)
    )
    println(people.runningFold(""){acc, person -> acc + person.name})
    // [, Alex, AlexNatalia]
}
```
### 컬렉션에 술어 적용 : all, any, none, count, find
* all, any, none : 어떤 조건을 만족하는지 판단하는 연산
* count : 조건을 만족하는 원소의 개수 반환
* find : 조건을 만족하는 첫 번째 원소 반환
```kotlin
val canBeInClub27 = {p: Person -> p.age <= 27} // 27살 이하인지 판단하는
fun main() {
    val people = listOf(Person("Alice", 27), Person("Bob", 31))
    println(people.all(canBeInClub27)) //모든 원소에 대해
    // false
    println(people.any(canBeInClub27)) //하나라도
    // true
    pritnln(people.count(canBeInClub27)) //개수
    // 1
    println(people.find(canBeInClub27)) //만족하는 원소 하나 찾고 싶으면 (제일 처음 맞는 것 반환, 없으면 null - firstOrNull과 같음)
    // Person(name=Alice, age=27)
}
```
> !all == any, !any == none
> * 빈 리스트에 any : false
> * 빈 리스트에 none : true
> * 빈 리스트에 all : true

> size 보다 count가 나은 이유는 size는 중간 컬렉션이 생기지만 count는 개수만 추적하기 때문

### 리스트를 분할해 리스트의 쌍으로 만들기 : partition
어떤 술어를 만족하는 그룹, 만족하지 않는 그룹 두개로 나누려고 할 때 filter와 filterNot을 사용할 수 있을텐데  
partition이 더 간단하게 할 수 있음
```kotlin
fun main() {
    val people = listOf(
        Person("Alice", 26),
        Person("Bob", 29),
        Person("Carol", 31)
    )
    val (comeIn, stayOut) = people.partition(canBeInClub27)
    println(comeIn)
    // [Person(name=Alice, age=26)]
    println(stayOut)
    // [Person(name=Bob, age=29), Person(name=Carol, age=31)]
}
```

### 리스트를 여러 그룹으로 이루어진 맵으로 바꾸기 : groupBy
```kotlin
fun main() {
    val people = listOf(
        Person("Alice", 31),
        Person("Bob", 29),
        Person("Carol", 31)
    )
    println(people.groupBy{it.age})
    // {31=[Person(name=Alice, age=31), Person(name=Carol, age=31)],
    // 29=[Person(name=Bob, age=29)]}
    // 각 그룹의 결과 타입은 Map<Int, List<Person>>
}
```

멤버 참조를 활용해 문자열을 첫번째 글자에 따라 분류하는 코드
```kotlin
fun main() {
    val list = listOf("apple", "apricot", "banana", "carrot")
    println(list.groupBy(String::first))
    // {a=[apple, apricot], b=[banana], c=[carrot]}
}
```

### 컬렉션을 맵으로 변환 : associate, associateWith, associateBy
그룹화 하지 않으면서 맵으로 변환하고 싶을 때
```kotlin
fun main() {
    val people = listOf(Person("Joe", 22), Person("Mary", 31))
    val nameToAge = people.associate {it.name to it.age}
    println(nameToAge)
    // {Joe=22, Mary=31}
    println(nameToAge["Joe"])
    // 22
}
```
* associateWith : 컬렉션의 원래 원소를 키로 사용
* associateBy : 컬렉션의 원래 원소를 값으로 사용, 람다가 만드는 값을 키로
```kotlin
fun main() {
    val people = listOf(Person("Joe", 22), Person("Mary", 31), Person("Jamie", 22))
    val personToAge = people.associateWith {it.age}
    println(personToAge)
    // {Person(name=Joe, age=22)=22, Person(name=Mary, age=31)=31, Person(name=Jamie, age=22)=22}
    val ageToPerson = people.associateBy {it.age}
    println(ageToPerson)
    // {22=Person(name=Jamie, age=22), 31=Person(name=Mary, age=31)}
}
```

### 가변 컬렉션의 원소 변경 : replaceAll, fill
* replaceAll을 mutableList에 적용하면 지정한 람다로 얻은 결과로 모든 원소를 변경
```kotlin
fun main() {
    val names = mutableListOf("Martin", "Samuel")
    println(names)
    // [Martin, Samuel]
    names.replaceAll{it.uppercase()}
    println(names)
    // [MARTIN, SAMUEL]
    names.fill("(redacted)")
    println(names)
    // [(redacted), (redacted)]
}
```

### 컬렉션의 특별한 경우 처리 : ifEmpty
* ifEmpty : 아무 원소도 없을 때 기본 값 생성하는 람다 제공
* ifBlank : 빈 문자열 대체
```kotlin
fun main() {
    val empty = emptyList<String>()
    val full = listOf("apple", "banana", "orage")
    println(empty.ifEmpty { listOf("no", "values", "here") })
    // [no, values, here]
    println(full.ifEmpty { listOf("no", "values", "here") })
    // [apple, banana, orange]
}
```

### 컬렉션 나누기 : chunked, windowed
* windowed : 슬라이딩 윈도우
* chunked : 지정한 크기만큼 겹치지 않게 나눔
```kotlin
fun main() {
    val temperatures = listOf(27.7, 29.8, 22.0, 35.5, 19.1)
    println(temperatures.windowed(3))
    // [27.7, 29.8, 22.0], [29.8, 22.0, 35.5], [22.0, 35.5, 19.1]
    println(temperatures.windowed(3) {it.sum() / it.size})
    // [26.5, 29.09999, 25.53333]
    println(temperatures.chunked(2))
    // [27.7, 29.8], [22.0, 35.5], [19.1]
    println(temperatures.chunked(2) {it.sum()})
    // [57.5, 57.5, 19.1]
}
```

### 컬렉션 합치기 : zip
각 리스트의 값들이 서로의 인덱스에 따라 대응되고 있는 걸 안다면, 두 컬렉션에서 같은 인덱스에 있는 원소들의 쌍으로 이뤄진 리스트 만들 수 있음
```kotlin
val names = listOf("Joe", "Mary", "Jamie")
val ages = listOf(22, 31, 31, 44, 0)

fun main() {
    pritnln(names.zip(ages))
    // [(Joe, 22), (Mary, 31), (Jamie, 31)] // 반대편 컬렉션에서 남은 건 무시
    println(names.zip(ages){name, age -> Person(name, age)})
    // [Person(name=Joe, age=22), Person(name=Mary, 31), Person(name=Jamie, 31)]
    println(names zip ages) // 중위 표기 가능하지만 람다 전달 불가능
    // [(Joe, 22), (Mary, 31), (Jamie, 31)] 
}
```
zip을 연쇄 호출하면 `a zip b zip c` 했을 때 ((a, b), c) 이렇게 됨

### 내포된 컬렉션의 원소 처리 : flatMap, flatten
```kotlin
val library = listOf(
    Book("A", listOf("aaa", "bbb")),
    Book("B", listOf("ccc, ddd")),
    Book("C", listOf("aaa"))
)

fun main() {
    val authors = library.flatMap {it.authors}
    println(authors)
    // [aaa, bbb, ccc, ddd, aaa] //중복이 있음
}
```
변환할 게 없으면 flatten 사용하면 된다

## 지연 계산 컬렉션 연산 : 시퀀스
시퀀스는 스트림과 비슷하게 중간 임시 컬렉션을 사용하지 않고 컬렉션 연산을 연쇄하는 방법 제공  
큰 컬렉션에 대한 연산을 연쇄 적용해야 할 때는 시퀀스를 꼭 쓰도록 (원소가 많으면 중간 원소 재배열 비용이 더 큼) 
```kotlin
people.map(Person::name) //리스트 반환1
    .filter{it.startsWith("A")} //리스트 반환2

people.asSequence() //원본 컬렉션을 시퀀스로 변환
    .map(Person::name)
    .filter{it.startsWith("A")}
    .toList() //시퀀스를 리스트로 변환
```

### 시퀀스 연산 실행 : 중간 연산, 최종 연산
* 중간 연산 : 다른 시퀀스 반환
* 최종 연산 : 결과 반환
```kotlin
fun main() {
    println(
        listOf(1, 2, 3, 4)
            .asSequence()
            .map {
                print("map($it) ")
                it * it
            }.filter {
                print("filter($it) ")
                it % 2 == 0
            }
    )
    
    //시퀀스 자체에 대한 출력만 나올 뿐
}

fun main() {
    println(
        listOf(1, 2, 3, 4)
            .asSequence()
            .map {
                print("map($it) ")
                it * it
            }.filter {
                print("filter($it) ")
                it % 2 == 0
            }.toList()  // 모든 계산 수행됨
    )
}

fun main() {
    println(
        listOf(1, 2, 3, 4)
            .asSequence()
            .map { it * it }
            .find { it > 3 }
    )
    // 컬렉션이었으면 1, 4, 9, 16 다음에 1, 4 이렇게 찾지만
    // 시퀀스라서 1, 4 하고 끝남
}
```

### 시퀀스 만들기
* asSequence()
* generateSequence()
```kotlin
fun main() {
    val naturalNums = generateSequence(0) {it + 1}
    val numbersTo100 = naturalNums.takeWhile {it <= 100}
    println(numbersTo100.sum()) // 이때 지연 계산 수행
    // 5050
}
```
