### 코틀린 예제

```kotlin
data class Person(
	val name : String,
	val age : Int? = null
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

// 가장 나이가 많은 사람: Person(name=영희, age=29)
```

### 람다식

```kotlin
val oldest = persons.maxBy {
		it.age ?: 0
}
```

- 파라미터가 하나일 때, 디폴트 이름을  `it` 으로 쓸 수 있음
- 엘비스 연산자 `?:` 은 age가 null이면 0을 리턴함 → 철수는 age가 null이지만 엘비스 연산자가 0을 리턴해서 비교 가능

### 코틀린의 특징

1. 다중 패러다임 언어
    1. 정적 타입 지정 언어 - 런타임 X 컴파일 시점에 오류를 많이 잡을 수 있음
        1. 성능 : 어떤 메소드를 호출하는 지 런타임에 찾아보지 않아서 더 빠름
        2. 신뢰 : 컴파일러가 타입을 사용해 일관성 검증, 런타임에 실패할 가능성 줄어듦
        3. 유지보수 : 타입이 이미 정해져있어서 낯선 코드를 봐도 쉽게 다룰 수 있음
        4. 도구 지원: IDE 기능 정확도를 높일 수 있음 (컴파일 시에 검증 가능)
        5. 타입 추론 가능
    2. 객체지향 + 함수형 = 추상화
        1. 일급시민 함수 (first-class 함수) : 함수를 일반 값처럼 다룰 수 있음 (변수에 저장, 인자로 다른 함수에 전달, 새로운 함수 만들어 반환)
        2. 불변성 : 내부 상태가 바뀌지 않는 불변 객체를 사용해 프로그램 작성 (멀티스레드에서 안전)
        3. 부수효과 없음 : 입력이 같으면 항상 같은 출력을 내놓음, 다른 객체의 상태를 변경하지 않고 함수 외부나 다른 바깥 환경과 상호작용하지 않는 순수 함수 사용

        ```kotlin
        messages
        	.filter{it.body.isNotBlank() && !it.isRead}
        	.map(Message::sender) // 람다 & 멤버참조
        	.distinct()
        	.sortedBy(Sender::name)
        ```

    3. 비동기 코드 → 코루틴 (일시 중단 가능한 계산)
        1. 코드가 자신의 실행을 잠시 중단시키고, 나중에 중단한 지점부터 작업을 계속 수행할 수 있음

        ```kotlin
        suspend fun processUser(credentials: Credentials) {
        	val user = authenticate(credentials)
        	val data = loadUserData(user)
        	val profilePicture = loadImage(data.imageID)
        	// ...
        }
        
        suspend fun authenticate(c: Credentials): User {}
        suspend fun loadUserData(u: User): Data {}
        suspend fun loadImage(id: Int): Image {}
        ```



```kotlin
        suspend fun loadAndOverlay(first: String, second: String): Image = 
        	continueScope {
        		val firstDeferred = async { loadImage(first) } // 새로운 코루틴으로 이미지 적재
        		val secondDeferred = async { loadImage(second) } // 또 다른 코루틴으로 이미지 적재
        		combineImages(firstDeferred.await(), secondDeferred.await()) // 두 이미지가 적재되면 두 이미지를 겹친 결과를 반환
        	}
        	
        	// firstDeferred, secondDeferred 둘 중 한쪽이 실패하면 두번째는 자동으로 취소
```

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
            Greeting(2, "Hi")
        )
    }

    data class Greeting(val id: Int, val text: String)
```