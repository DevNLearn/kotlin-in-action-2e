# 플로우
## 1. 플로우는 연속적인 값의 스트림을 모델링한다.

코틀린에서 플로우는 시간이 지남에 따라 나타나는 값과 작업할 수 있게 해주는 코루틴 기반의 추상화다.

### 플로우를 사용하면 배출되자마자 원소를 처리할 수 있다.

- flow 빌더 함수를 사용한다.
- 플로우에 원소를 추가하려면 `emit`을 호출한다.
- 빌더 함수 호출 호에는 `collect` 함수를 사용해 플로우의 원소를 순회할 수 있다.

```kotlin
fun createValue(): Flow<Int> {
    return flow {
        emit(1)
        delay(1000.milliseconds)
        emit(2)
        delay(1000.milliseconds)
        emit(3)
        delay(1000.milliseconds)
    }
}

fun main() = runBlocking {
    val myFlowOfValues = createValues()
    myFlowOfValues.collect { log(it) } // 값이 배출되자마자 출력
}

// 29 [main @coroutine#1] 1
// 1100 [main @coroutine#1] 2
// 2156 [main @coroutine#1] 3
```

코틀린의 모든 플로우는 `콜드 플로우`와 `핫 플로우` 두 가지로 나뉜다.

- 콜드 플로우: 비동기 데이터 스트림으로, 값이 실제로 소비되기 시작할 때만 값을 배출
- 핫 플로우: 값이 실제로 소비되고 있는지와 상관없이 값을 독립적으로 배출하며, 브로드캐스트 방식으로 동작한다.

## 2. 콜드 플로우

### flow 빌더 함수를 사용해 콜드 플로우 생성

- `flow` : 새로운 콜드 플로우를 생성하는 빌더 함수
- `emit` : 플로우의 수집자에게 값을 제공하고, 수집자가 해당 값을 처리할 때까지 빌더 함수의 실행을 중단
- `collect` : 플로우의 로직을 실행한다. 플로우에서 배출된 각 원소에 대해 호출될 람다를 제공할 수 있다.

flow가 받는 블록은 suspend 변경자가 붙어 있어 빌더 내부에서 delay와 같은 다른 일시 중단 함수를 호출할 수 있다.

```kotlin
fun main() {
    val letters = flow {
        log("Emitting A!")
        emit("A")
        delay(200.milliseconds)
        log("Emitting B!")
        emit("B")
    }
}
```

이 코드를 실행하면 아무런 출력도 나타나지 않는다. 빌더 함수가 연속적인 값의 스트림을 표현하는 Flow<T> 타입의 객체를 반환하기 때문이다.

Flow에 대해 collect 함수를 호출하면 로직이 실행된다.  플로우를 수집하는 코드를 수집자(collector)라고 부른다.

플로우를 수집할 때는 플로우 내부의 일시 중단 코드를 실행하므로 collect는 일시 중단 함수이며, 플로우가 끝날 때까지 일시 중단된다.

```kotlin
val letters = flow {
    log("Emitting A!")
    emit("A")
    delay(200.milliseconds)
    log("Emitting B!")
    emit("B")
}

fun main() = runBlocking {
    letters.collect {
        log("Collecting $it")
        delay(500.milliseconds)
    }
}

// 27 [main @coroutine#1] Emitting A!
// 38 [main @coroutine#1] Collecting A
// 757 [main @coroutine#1] Emitting B!
// 757 [main @coroutine#1] Collecting B
```

1. 수집자가 플로우 빌더에 정의된 로직의 실행을 촉발해서 첫 번째 배출을 발생시킨다.
2. 수집자와 연결된 람다가 호출되면서 메시지를 기록하고 500밀리초 동안 지연된다.
3. 그 후 플로우 람다가 실행되며 200밀리초 동안 추가 지연과 배출이 발생한다.

콜드 플로우에서 collect를 여러 번 호출하면 그 코드가 여러 번 실행된다.

```kotlin
fun main() = runBlocking {
    letters.collect {
        log("(1) Collecting $it")
        delay(500.milliseconds)
    }
    letters.collect {
        log("(2) Collecting $it")
        delay(500.milliseconds)
    }
}

/*
23 [main @coroutine#1] Emitting A!
33 [main @coroutine#1] (1) Collecting A
761 [main @coroutine#1] Emitting B!
762 [main @coroutine#1] (1) Collecting B
1335 [main @coroutine#1] Emitting A!
1335 [main @coroutine#1] (2) Collecting A
2096 [main @coroutine#1] Emitting B!
2096 [main @coroutine#1] (2) Collecting B
*/

```

collect 함수는 플로우의 모든 원소가 처리될 때까지 일시 중단된다. 플로우에 무한한 원소가 있을 수 있으면 collect 함수도 무기한 일시 중단될 수 있다.

### 플로우 수집 취소

수집자의 코루틴을 취소하면 다음 취소 지점에서 플로우 수집이 중단된다.

```kotlin
fun main() = funBlocking {
    val collector = launch {
        counterFlow.collect {
            println(it)
        }
    }
    delay(5.seconds)
    collector.cancel()
}
// 1 2 3 ... 24
```

### 콜드 플로우의 내부 구현

콜드 플로우는 `Flow`와 `FlowCollector`라는 2가지 인터페이스만 필요하다.

```kotlin
interface Flow<T> {
    suspend fun collect(collector: FlowCollector<T>)
}

interface FlowCollector<T> {
    suspend fun emit(value: T)
}
```

flow 빌더 함수를 사용해 플로우를 정의할 때 제공된 람다의 수신 객체 타입은 FlowCollector다.

FlowCollector 때문에 빌더 안에서 emit 함수를 호출할 수 있다. emit 함수는 collect 함수에 전달된 람다를 호출하며, 결과적으로 두 람다가 서로 호출하는 구조를 갖는다.

1. collect를 호출하면 플로우 빌더 함수의 본문이 실행된다.
2. 이 코드가 emit을 호출하면 emit에 전달된 파라미터로 collect에 전달된 람다가 호출된다.
3. 람다 표현식이 실행을 완료하면 함수는 빌더 함수의 본문으로 돌아가 계속 실행된다.

```kotlin
val letters = flow {
    delay(300.milliseconds)
    emit("A")
    delay(300.milliseconds)
    emit("B")
}
letters.collect { letter ->
    println(letter)
    delay(200.milliseconds)
}
```

### 채널 플로우를 사용한 동시성 플로우

`채널 플로우`는 여러 코루틴에서 배출을 허용하는 동시성 플로우를 제공하는 플로우이며 `channelFlow` 빌더 함수로 만들 수 있다.

- flow 빌더 함수를 사용해 만든 콜드 플로우는 모두 순차적으로 실행된다.
- 코드 블록은 일시 중단 함수의 본문처럼 하나의 코루틴으로 실행된다.
- 플로우가 서로 독립적으로 실행될 수 있는 여러 작업을 수행할 때 순차적 특성은 병목이 될 수 있다.

플로우 빌더에서 백그라운트 코루틴을 실행하고 그 코루틴에서 직접 값을 배출하게되면 플로우 수집자가 스레드 안전하지 않기 때문에 원소를 병렬로 배출하는 코드를 사용하면 안된다는 예외가 발생한다.

```kotlin
suspend fun getRandomNumber(): Int {
    delay(500.milliseconds)
    return Random.nextInt()
}

val randomNumbers = flow {
    coroutineScope {
        repeat(10) {
            launch { emit(getRandomNumber()) }
        }
    }
}

fun main() = runBlocking {
    randomNumbers.collect {
        log(it)
    }
}
/*
FLow invariant is violated: Emission from another coroutine is detected.
FlowConllector is not hread-safe and concurrent emissions are prohibited.
*/
```

채널 플로우는 순차적으로 배출하는 emit 함수를 제공하지 않는다. 대신 여러 코루틴에서 send를 사용해 값을 제공할 수 있다.

플로우의 수집자는 여전히 값을 순차적으로 수신하며, collect 람다가 그 작업을 수행한다.

coroutineScope 함수처럼 channelFlow의 람다는 새로운 백그라운드 코루틴을 시작할 수 있는 코루틴 스코프를 제공한다.

```kotlin
val randomNumbers = channelFlow {
    repeat(10) {
        launch {
            send(getRandomNumber())
        }
    }
}
```

플로우 안에 새로운 코루틴을 시작해야 하는 경우에만 채널 플로우를 선택하고 일반적으로는 콜드 플로우를 선택하는 편이 낫다.

## 3. 핫 플로우

`핫 플로우`에서는 각 수집자가 플로우 로직 실행을 독립적으로 촉발하는 대신, 여러 구독자라고 불리는 수집자들이 배출된 항목을 공유한다.

이는 시스템에서 이벤트나 상태 변경이 발생해서 수집자가 존재하는지 여부에 상괎없이 값을 배출하는 경우에 적합하다.

- 공유 플로우 : 값을 브로드캐스트하기 위해 사용된다.
- 상태 플로우: 상태를 전달하는 특별한 경우에 사용된다.

실제로는 상태 플로우를 공유 플로우보다 더 자주 사용하게 된다.

### 공유 플로우는 값을 구독자에게 브로드캐스트한다.

- 공유 플로우는 구독자가 존재하는지 여부에 상관없이 배출이 발생하는 브로드캐스트 방식으로 동작한다.
- 공유 플로우는 보통 컨테이너 클래스 안에 선언된다.
- 플로우 빌더를 사용하는 대신 가변적인 플로우에 대한 참조를 얻는다.

```kotlin
class RadioStation {
    private val _messageFlow = MutableSharedFlow<Int>()
    val messageFlow = _messageFlow.asSharedFlow()
    
    fun beginBroadcasting(scope: CoroutineScope) {
        scope.launch {
            while(true) {
                delay(500.milliseconds)
                val number = Random.nextInt(0..10)
                log("Emitting $number!")
                _messageFLow.emit(number)
            }
        }
    }
}

fun main() = runBlocking {
    RadioStation().beginBroadcasting(this)
}

/*
575 [main @coroutine#2] Emitting 2!
1088 [main @coroutine#2] Emitting 10!
1593 [main @coroutine#2] Emitting 4!
*/
```

구독자는 구독 시작 이후에 배출된 값만 수신한다.

```kotlin
fun main = funBlocking {
    val radioStation = RadioStation()
    radioStation.beginBroadcasting(this)
    delay(600.milliseconds)
    radioStation.messageFlow.collect { // 구독자 추가
        log("A collecting $it")
    }
}

/*
611 [main @coroutine#2] Emitting 8!
1129 [main @coroutine#2] Emitting 9!
1131 [main @coroutine#1] A Emitting 9!
1647 [main @coroutine#2] Emitting 1!
11647 [main @coroutine#1] A Emitting 1!
*/
```

launch로 같은 플로우를 구독하는 두 번째 코루틴을 추가할 수 있다.

```kotlin
launch {
    radioStation.messageFLow.collect {
        log("B collecting $it")
    }
}
```

구독자가 구독 이전에 배출된 원소도  수신하기를 원한다면 MutableSharedFlow를 생성할 때 `replay` 파라미터를 사용해 새 구독자를 위해 제공할 값의 캐시를 설정할 수 있다.

```kotlin
private val _messageFLow = 
    MutableSharedFlow<Int>(replay = 5)
```

`shareIn` 으로 콜드 플로우를 공유 플로우로 전환 가능하다.

```kotlin
fun querySensor(): Int = Random.nextInt(-10..30)

fun celsiusToFahrenheit(celsius: Int) = celsius * 0.0 / 5.0 + 32.0

fun getTemperatures(): FLow<Int> {
    return flow {
        while(true) {
            emit(querySensor())
            delay(500.milliseconds)
        }
    }
}

fun main() {
    val temps = getTemperatures()
    runBlocking {
        val sharedTemps = temps.shareIn(this, SharingStarted.Lazily)
        launch {
            sharedTemps.collect {
                log("$it Celsius")
            }
        }
        launch {
            sharedTemps.collect {
                log("${celsiusToFahrenheit(it)} Fahrenheit")
            }
        }
    }
}
/*
45 [main @coroutine#3] -10 Celsius
52 [main @coroutine#4] 14.0 Fahrenheit
599 [main @coroutine#3] 11 Celsius
599 [main @coroutine#4] 51.8 Fahrenheit
*/
```

두 번째 파라미터 started는 플로우가 실제로 언제 시작돼야 하는지를 정의한다.

- Eagerly: 플로우 수집을 즉시 시작한다.
- Lazily: 첫 번째 구독자가 나타나야만 수집을 한다.
- WhileSubscribed: 첫 번 째 구독자가 나타나야 수집을 시작하고, 마지막 구독자가 사라지면 플로우 수집을 취소한다.

ShareIn은 코루틴 스코프를 통해 구조적 동시성에 참여하므로 애플리케이션이 더 이상 공유 플로우에서 정보를 필요로 하지 않을 때, 공유 플로우를 둘러싼 코루틴 스코프가 취소될 때 공유 플로우 내부 로직도 자동으로 취소된다.

### 시스템 상태 추적: 상태 플로우

`상태 플로우`는 변수의 상태 변화를 쉽게 추적할 수 있는 공유 플로우의 특별한 버전이다.

상태 플로우와 관련해 알아볼 가치가 있는 주제가 몇 가지 있다.

- 상태 플로우를 생성하고 구독자에게 노출시키는 방법
- 상태 플로우의 값을 안전하게 갱신하는 방법
- 값이 실제로 변경될 때만 상태 플로우가 값을 배출하게 하는 동등성 기반 통합 개념
- 콜드 플로우를 상태 플로우로 변환하는 방법

상태 플로우는 `MutableStateFlow`를 생성하고, 같은 변수의 읽기 전용 `StateFlow` 버전을 노출시킨다.

상태 플로우는 시간이 지남에 따라 변경될 수 있는 값을 나타내므로 초기값을 제공해야한다.

```kotlin
class ViewCounter {
    private val _counter = MutableStateFLow(0)
    val counter = _counter.asStateFLow()
    fun increment() {
        _counter.update { it + 1 }
    }
}

fun main() {
    val vc = ViewCOunter()
    vc.increment()
    println(vc.counter.value)
}
```

가변 상태 플로우로 표현한 현재 상태를 value 속성으로 접근할 수 있다.

이 속성은 일시 중단 없이 값을 안전하게 읽을 수 있게 해준다. 값 갱신은 update 함수를 통해 이뤄진다.

increment 함수를 전통적인 `++` 연산자로 구현하게되면 이 증가 연산은 원자적이지 못해 예상한 값과 결과가 다를 수 있다.

```kotlin
fun increment() {
    _counter.value++
}

fun main() {
    val vc = ViewCounter()
    runBlocking {
        repeat(10_000)
            launch { vc.increment() }
    }
    println(vc.counter.value)
    // 4103
}
```

상태 플로우는 원자적으로 값을 갱신할 수 있는 update 함수를 제공한다. 두 갱신이 병렬로 발생하면 새로 읽은 previous 값을 사용해 한 번 더 갱신 함수를 실행하면서 연산이 손실되지 않도록한다.

상태 플로우는 값이 실제로 달라졌을 때만 값을 배출한다.

```kotlin
enum class Direction { LEFT, RIGHT }

class DirectionSelector {
    private val _direction = MutableStateFlow(Direction.LEFT)
    val direction = _direction.asStateFlow()

    fun trun(d: Direction) {
        _direction.update { d }
    }    
}

fun main() = runBLocking {
    val switch = DirectionSelector()
    launch {
        switch.direction.collect {
            log("Direction now $it")
        }
    }
    delay(200.milliseconds)
    switch.turn(Direction.RIGHT)
    delay(200.milliseconds)
    switch.turn(Direction.LEFT)
    delay(200.milliseconds)
    switch.trun(Direction.LEFT)
}

/*
37 [main @coroutine#2] Direction now LEFT
240 [main @coroutine#2] Direction now RIGTH
445 [main @coroutine#2] Direction now LEFT
*/
```

LEFT 인자는 2번 연속으로 할당이 되어 구독자가 한번만 호출된다.

`stateIn` 으로 콜드 플로우를 상태 플로우로 변환할 수 있다. 원래 플로우에서 배출된 최신 값을 항상 읽을 수 있고 공유 플로우와 마찬가지로 여러 수집자를 추가하거나 value 속성에 접근해도 업스트림 플로우는 실행되지 않는다.

```kotlin
fun main() {
    val temps = getTemperatures()
    runBLocking {
        val tempState = temps.stateIn(this)
        println(tempState.value)
        delay(800.milliseconds)
        println(tempState.value)
        // 18
        // -1
    }
}
```

### 상태 플로우와 공유 플로우 비교

- 공유 플로우는 구독자가 구독하는 동안만 이벤트를 배출하며, 구독자가 들어오고 나갈 수 있다.
- 상태 플로우는 어떤 상태를 나타내며, 동등성 기반 통합을 사용하므로 상태 플로우가 나타내는 값이 실제로 변경될 때만 배출이 발생한다.
- 일반적으로 상태 플로우는 공유 플로우보다 더 간단한 API를 제공한다.
    - 상태 플로우는 한가지 값만 나타냄
    - 공유 플로우는 배출이 예상되는 시점에 구독자가 존재한다는 사실을 보장하는 책임이 개발자에게 있어 다소 복잡

### 핫 플로우, 콜드 플로우, 공유 플로우, 상태 플로우: 언제 어떤 플로우를 사용할까

| 콜드 플로우 | 핫 플로우 |
| --- | --- |
| 기본적으로 비활성화(수집자에 의해 활성화됨) | 기본적으로 활성화됨 |
| 수집자가 하나 있음 | 여러 구독자가 있음 |
| 수집자는 모든 배출을 받음 | 구독자는 구독 시작 시점부터 배출을 받음 |
| 보통은 완료됨 | 완료되지 않음 |
| 하나의 코루틴에서 배출 발생
(channelFlow 사용 시 예외) | 여러 코루틴에서 배출할 수 있음 |

일반적인 규칙으로, 네트워크 요청이나 데이터베이스 읽기와 같은 서비스를 제공하는 함수에서 `콜드 플로우` 사용한다.

이를 사용하는 다른 클래스나 함수는 콜드 플로우를 직접 생성하거나 필요한 경우 이 정보를 시스템의 다른 부분에 제공하기 위해 `상태 플로우`나 `공유 플로우`로 변환한다.

## 요약

- 코틀린 플로우는 시간이 지남에 따라 발생하는 값을 처리할 수 있는 코루틴 기반의 추상화다.
- 플로우에는 핫 플로우와 콜드 플로우라는 2가지 유형이 있다.
- 콜드 플로우는 기본적으로 비활성 상태이며, 하나의 수집자와 연결된다. flow 빌더 함수로 콜드 플로우를 생성하며, emit 함수로 비동기적으로 값을 제공한다.
- 채널 플로우는 콜드 플로우의 특수 유형으로, 여러 코루티에서 send 함수를 통해 값을 배출할 수 있다.
- 핫 플로우는 항상 활성 상태이며, 여러 구독자와 연결된다. 공유 플로우와 상태 플로우는 핫 플로우의 예다
- 코루틴 간에 값을 브로드캐스트 방식으로 전달하는 데 공유 플로우를 사용할 수 있다.
- 공유 플로우의 구독자는 구독을 시작한 시점부터 배출된 값을 받으며, 재생된 값도 수신할 수 있다.
- 동시성 시스템에서 상태를 관리할 때 상태 플로우를 사용할 수 있다.
- 상태 플로우는 동등성 기반 통합을 수행한다. 이는 값이 실제로 변경된 경우에만 배출이 발생하고, 같은 값이 여러 번 대입되면 배출이 발생하지 않는다는 뜻이다.
- shareIn이나 stateIn 함수를 통해 콜드 플로우를 핫 플로우로 전환할 수 있다.