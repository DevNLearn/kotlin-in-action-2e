# Module Graph

# 1장. 코틀린이란 무엇이며, 왜 필요한가?

- 트레일링 콤마(trailing comma)를 사용한다.

새로운 속성을 추가할 때, 마지막 줄에 트레일링 콤마가 있다면 <br>
그 줄을 수정 없이 그대로 복사해 쓸 수 있다.

```kotlin
Person(
    "chuck",
    25,
)

// Person에 핸드폰 번호가 추가되는 경우,
Person(
    "chuck",
    25, // 여기에 변화가 없음
    "010-0000-0000", 
)
```

- 코틀린은 정적 타입 지정 언어. **(객체지향 언어 + 함수형 언어)**
  - 실행시점이 아니라 컴파일 시점에 오류를 잡아낼 수 있다.
  - 성능, 신뢰성, 유지 보수성 상승
  - 타입추론 제공 `ex. val x = 1`
  - 널이 될 수 있는 타입 지원 `ex. var x: String? = null`
  - 함수형 프로그래밍 지원 (람다식)
    - 람다함수를 인라이닝해서 새로운 객체가 만들어지지 않아 객체 증가로 인한 가비지 컬렉션으로 프로그램이 더 자주 잠시 멈추는 것을 겪지 않아도됨.

### ex.1_1

```kotlin
data class Person(
    val name: String,
    val age: Int? = null
)

fun main() {
    val persons = listOf(
        Person("영희", age = 29),
        Person("철수"),
    )

    val oldest = persons.maxBy {
        it.age ?: 0
    }

    println("가장 나이가 많은 사람: $oldest")
}

/*
가장 나이가 많은 사람: Person(name=영희, age=29)
 */
```

### ex.1_2

```kotlin
@SpringBootApplication
class DemoApplication

fun main(args: Array<String>) {
    runApplication<DemoApplication>(*args)
}

@RestController
class GreetingResource {

    @GetMapping
    fun index(): List<Greeting> = listOf(
        Greeting(1, "Hello"),
        Greeting(2, "Bonjour!"),
        Greeting(3, "Guten Tag!"),
    )
}

data class Greeting(val id: Int, val text: String)
```
