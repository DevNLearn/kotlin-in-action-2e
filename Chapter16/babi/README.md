## 플로우-연속적인 값의 스트림 모델링

- 일시 중단 함수는 원시 타입, 객체, 객체의 컬렉션과 같은 단일값만 반환 가능하다.

```kotlin
suspend fun create(): List<Int> {
    return buildlist {
        add(1)
        delay(1.second)
        add(2)
        delay(1.second)
        add(3)
        delay(1.second)
    }
}

fun main() = runBlocking {
    val list = create()
    list.forEach {
        log(it)
    }
}

output>
// 3초 이후
1
2
3
```

- 하지만 `create()` 함수의 실제 구현을 보면 첫번째 원소는 즉시 사용가능하고, 두번째 원소는 1초 delay 이후 바로 사용가능한것으로 보인다.
- 특정 함수가 여러 값을 시간이 지남에 따라 계산할때, 전체 함수가 끝날때까지 기다리지 않고 값을 사용할때 **플로우**가 유용하다.
- 코루틴 기반의 추상화

---

- 플로우 사용 예시

```kotlin

fun create(): Flow<Int> {
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
    val values = create()
    values.collect {
        log(it)
    }
}

output>
1
// 1초 이후
2
// 1초 이후
3
```

- 플로우는 `emit`으로 데이터 전달을 하며, 빌더 함수 호출 이후는 `collect` 함수를 사용해 플로우의 원소를 순회한다
- 함수가 모두 끝날때까지 기다리지 않고, 값이 계산되자마자 사용 가능한 추상화가 플로우의 핵심 개념이다

## 플로우 유형

간단하게 다음과 같이 이해하는데 자세히 살펴보자

- 콜드 플로우 : 값이 실제로 소비되기 시작할때만 값을 배출
- 핫 플로우 : 값이 소비되는것과 상관없이 독립적으로 배출

### 콜드 플로우

- 플로우를 생성하는 빌더 함수로 만든 예제를 한번 보자

```kotlin
fun main() {
    val letters = flow { // 플로우 생성하는 빌더 함수
        log("Emit A")
        emit("A")
        delay(200.milliseconds)
        log("Emit B")
        emit("B")
    }
}

output>
```

- 위 코드를 실행하더라도 아무 출력이 나타나지 않는다.
- 빌더 함수는 연속적인 스트림을 표현하는 `Flow<T>` 타입의 객체를 반환하고, 호출되지 않아 아직은 **비활성 상태**이다
- 최종 연산자(collect, collectLatest, single, reduce, toList 등등)이 호출되어야 해당 빌더에서 계산이 시작된다.
- flow 빌더 함수만으로는 (호출을 하지 않는다면) 실제 작업이 시작되지 않기 때문에 다음과 같은 무한 플로우를 정의해도 상관없긴하다

```kotlin

val infinityFlow = flow {
    var x = 0
    while (true) {
        emit(x++)
        delay(200.milliseconds)
    }
}

output>
```

---

### 콜드 플로우는 수집되기 전까지 작업을 수행하지 않는다.

먼저 예제부터 확인하자

```kotlin
val letters = flow {
    log("Emit A")
    emit("A")
    delay(200.milliseconds)
    log("Emit B")
    emit("B")
}

fun main() = runBlocking {
    letters.collect {
        log("Collectiong ${it}")
        delay(500.milliseconds)
    }
}

output>
Emit A
Collecting A
// 약 700 milliseconds 이후
Emit B
Collecting B
```

- `main()` 함수에서 `collect()` 최종연산자가 호출되면서, flow 빌더 함수가 실행된다
- `emit("A")` 으로 배출한 값을 `collect()` 최종연산자에서 받아서 실행한다.
- `collect()` 최종연산자가 끝나면 배출한 `emit("A")` 에서부터 다시 실행

### 콜드 플로우에서 여러번 호출하기

- 콜드 플로우에서 최종연산자를 여러번 호출하면, 해당 flow 빌더함수가 여러번 실행된다.

```kotlin
val letters = flow {
    log("Emit A")
    emit("A")
    delay(200.milliseconds)
    log("Emit B")
    emit("B")
}

fun main() = runBlocking {
    letters.collect {
        log("(1) Collecting ${it}")
        delay(500.milliseconds)
    }
    letters.collect {
        log("(2) Collecting ${it}")
        delay(500.milliseconds)
    }
}

output>
Emit A
(1) Collecting A
// 약 700 milliseconds 이후
Emit B
(1) Collecting B
// 약 700 milliseconds 이후
Emit A
(2) Collecting A
// 약 700 milliseconds 이후
Emit B
(2) Collecting B
```

---

### 콜드 플로우 내부 구현

- 콜드 플로우는 일시 중단 함수와 수신 객체 지정 람다를 결합했다.
- 콜드 플로우는 다음 두가지 인터페이스만으로 구현되었다

```kotlin
interface Flow<T> {
    suspend fun collect(collector: FlowCollector<T>)
}

interface FlowCollector<T> {
    suspend fun emit(value: T)
}
```

- `collect()` 를 호출하면 플로우 빌더 함수의 본문이 실행된다.
- 플로우 빌더 함수의 본문에서 `emit()`을 호출하면 파라미터로 전달된 값으로 `collect()` 가 실행된다
- `collect()` - `emit()` 함수가 서로를 실행시킨다

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

output>
// 약 300 milliseconds 이후
A
// 약 500 milliseconds 이후
B
```

### 채널 플로우를 사용한 동시성 플로우

- 다음 예제를 보면, 난수 하나를 생성하는데 약 5초이상이 소요되어, 병목이 일어날수 있는 코드가 있다

```kotlin
suspend fun getRandomNumber(): Int {
    delay(500.milliseconds)
    return Random.nextInt()
}

val randomNumbers = flow {
    repeat(10) {
        emit(getRandomNumber())
    }
}

fun main() = runBlocking {
    randomNumbers.collect {
        log(it)
    }
}

output>
// 약 5초 이후
난수
// 약 5초 이후
...
// 약 5초 이후
난수
```

- 플로우는 순차적으로 실행되고, 계산은 동일한 코루틴에서 실행되어 결과를 받는데까지 오랜 시간이 걸렸다.
- 난수를 생성하는 부분을 병렬로 빼기위해, 플로우 빌더 블록 내부에서 코루틴을 실행하게 되면 `FlowCollector is not thread-safe and concurrent emissions are prohibited` 오류메세지가 나오는데, 말그대로 `FlowCollector`는 thread-safe 하지않기때문에, 병렬로 `emit()` 함수를 호출하는건 허용하지 않는다.
- 기본적으로 콜드 플로우는 같은 코루틴에서 동기적으로 실행되도록 보장되어야 한다(같은말)

- 대신 이와같은 구현을 최적화하기 위해 채널 플로우(`channelFlow`)를 사용한다.

```kotlin
val randomNumbers = channelFlow {
    repeat(10) {
        launch {
            send(getRandomNumber())
        }
    }
}

ouptut>
// 약 5초 이후
난수
난수
난수
...
```

---

### 콜드 플로우와 채널 플로우의 선택?

콜드 플로우

- 간단하다
- 성능이 좋다
- 순차적으로 실행된다
- 관리 포인트가 적다
- 오버헤드가 없다

채널 플로우

- 동시작업에 특화
- 동시작업을 위한 채널관리로 생성에 약간의 비용이 추가
- 코루틴 간 통신을 위한 저수준의 추상화
- **플로우 내에서 새로운 코루틴을 열어야할때만 채널 플로우를 선택하자**

---

## 핫 플로우

- 배출-수집 메커니즘은 동일하지만, 몇가지 차이점이 있다.
- 항상 활성 상태라 구독자의 유무에 관계없이 배출이 발생할 수 있다
- 기본 핫 플로우 구현이 다음과 같이 제공된다
  - 공유 플로우 - 값을 브로드캐스트 하기위함
  - 상태 플로우 - 상태를 전달하는 특별한 경우

---

### 공유 플로우

- 구독자의 존재 여부에 상관없이 배출이 발생한다.
- 가변 공유 플로우는 `private` 으로 감싸서 캡슐화시키고, 구독자에게 배출하는 읽기 전용 버전은 `public` 속성으로 노출한다.

```kotlin
class RadioStation {
    private val _messageFlow = MutableSharedFlow<Int>() // 가변 공유 플로우는 private
    val messageFlow = _messageFlow.asSharedFlow()   // 읽기 공유 플로우는 public

    fun beginBroadcasting(scope: CoroutineScope) {
        scope.launch {
            while(true) {
                delay(500.milliseconds)
                val number = Random.nextInt(0..10)
                log("Emitting ${number}")
                _messageFlow.emit(number)
            }
        }
    }
}
```

- 그리고 해당 클래스 인스턴스를 생성 후, `beginBroadcasting` 함수를 호출하면 구독자가 없어도 즉시 실행된다.

```kotlin
fun main() = runBlocking {
    RadioStation().beginBroacasting(this)
}

output>
// 약 5초 이후
Emitting 난수
// 약 5초 이후
...
// 약 5초 이후
Emitting 난수
```

---

- 구독자 추가는 콜드 플로우수집과 동일하다
- 배출이 발생할 때 마다 람다가 실행되지만, 구독 시작 이후에 배출된 값만 수신한다.

```kotlin
fun main() = runBlocking {
    val radioStation = RadioStation()
    radioStation.beginBroadcasting(this)
    delay(600.milliseconds)
    radioStation.messageFlow.collect {
        log("A collecting ${it}")
    }
}

output>
// 약 5초 이후
Emitting 1
// 약 5초 이후
Emitting 2
A collecting 2
...
```

- 공유 플로우는 동일한 플로우를 구독하는 또 다른 코루틴을 추가할 수 있다

```kotlin
launch {
    radioStation.messageFlow.collect {
        log("B collecting ${it}")
    }
}
```

- 구독 시작 이후에 배출된 값만 수신한다고 설명했다. 하지만, 구독자가 구독 이전에 배출된 우너소를 수신할 수도 있는데, `MutableSharedFlow` 생성시 `replay` 파라미터로 직전 몇개 값까지 캐싱할것인지 설정할 수 있다.
- 다음은 구독 직전에 발생한 최대 5개 값까지 수신 가능하다는 설정이다

```kotlin
private val _messageFlow = MutableSharedFlow<Int>(replaty = 5)
```

### 콜드 플로우를 공유 플로우로

- 다음과 같이 온도를 알려주는 콜드 플로우가 존재한다고 하자

```kotlin
fun querySensor(): Int = Random.nextInt(-10..30)

fun getTemperatures(): Flow<Int> {
    return flow {
        while (true) {
            emit(querySensor())
            delay(500.milliseconds)
        }
    }
}
```

- 섭씨와 화씨로 각각 노출하려고 한다면, `getTemparatures()` 플로우를 부른 값을 두번 수집해야하고, 수집할때마다 플로우 빌더 구문이 실행되게된다.

```kotlin
fun convert(degree: Int) = degree * 9.0 ...

fun main() {
    val temps = getTemperatures()
    runBlocking {
        launch {
            temps.collect {     // 플로우 수집
                log("${it} Celsius")
            }
        }
        launch {
            temps.collect {     // 플로우 수집
                log("${convert(it)} Fahrenheit")
            }
        }
    }
}
```

- 위와같이 불필요한 상호작용 및 연산을 피하기 위해 수집을 공유해야 한다.
- `shareIn` 함수를 통해 콜드 플로우를 공유 플로우로 변환할 수 있다
- 플로우 코드가 실행되게 하므로 `shareIn` 코루틴 내에서 호출해야 한다.

```kotlin
fun main() {
    val temps = getTemperatures()
    runBlocking {
        val sharedTemps = temps.shareIn(this, SharingStarted.Lazily)    // temps 플로우 공유
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
```

- 두번째 파라미터로 받은 `SharingStarted` 는 플로우가 언제 시작되는지 정의할 수 있다
  - `Eagerly` : 플로우 수집 즉시 시작
  - `Lazily` : 첫 번째 구독자가 나타나면 수집 시작
  - `WhileSubscribed` 첫 번째 구독자가 나타나면 수집 시작. 마지막 구독자가 사라지면 수집 취소

### 상태 플로우

- 동시 시스템에서 불변하지 않은 값들의 상태 관리가 필요한 경우가 있다.
- 이런 상태를 추적하도록 특수화된 추상화인 상태 플로우를 제공한다.
- 상태 플로우를 생성하는건 공유 플로우와 비슷한데, 가변한 플로우는 `private` 으로 캡슐화하고, 불변한 플로우는 `public` 하게 둔다.
- 상태 플로우는 시간이 지남에 따라 변경될 수 있는 값이므로, 생성자로 초기화 해줘야한다.
- 대신 배출할때는 `emit()` 대신 값을 변경하는 `update()` 함수를 사용한다.
- 플로우 현재 상태를 `value` 속성으로 접근할 수 있는데, 이는 일시 중단 없이 값을 안전하게 읽을 수 있게 한다.

```kotlin
class ViewCounter {
    private val _counter = MutableStateFlow(0)  // 초깃값 설정
    val counter = _counter.asStateFlow()

    fun increment() {
        _counter.update { it + 1 }
    }
}

fun main() {
    val vc = ViewCounter()
    vc.increment()
    println(vc.counter.value)
}

output>
1
```

### 왜 UPDATE 인가

- 위 코드의 `increment()` 함수에서 update 를 쓰는 대신, `value` 속성에 직접 `++` 연산자로 구현할 수 있지 않을까?
- 코루틴 초반에도 나왔었는데, 스레드 세이프 하지 않은 로직이다.
- 각 코루틴들이 여러 스레드에 분산 실행되기 때문에, 비원자적으로 수행된다.
- 상태 플로우는 원자적으로 수행되기 위해 `update` 함수를 제공한다.
- 갱신이 병렬로 발생하면, 새로 읽은 `previous` 값을 한번 더 사용해 연산이 누락되지 않게 한다.

---

### 동등성 기반 통합

- 상태 플로우는 값이 달라졌을때만 값을 배출한다.
- 왼쪽과 오른쪽 두개의 상태를 가지고있는 상태 플로우 코드와 실행하는 main 코드가 있다고 하자.

```kotlin
enum class Direction { LEFT, RIGHT }

class DirectionSelector {
    private val _direction = MutableStateFlow(Direction.LEFT)
    val direction = _direction.asStateFlow()

    fun turn(d: Direction) {
        _direction.update { d }
    }
}

fun main() = runBlocking {
    val switch = DirectionSelector()
    launch {
        switch.direction.collect {
            println("Direction now ${it}")
        }
    }
    delay(200)
    switch.turn(Direction.RIGHT)
    delay(200)
    switch.turn(Direction.LEFT)
    delay(200)
    switch.turn(Direction.LEFT) // 출력안됨
}

output>
Direction now LEFT
Direction now RIGHT
Direction now LEFT
```

- 최초 초기값 `LEFT`
- 두번째 변경값 `RIGHT`
- 세번째 변경값 `LEFT`
- 위와같이 세개만 출력이 되고, 마지막 `LEFT` 는 출력이 안된다. - 값이 동일하다면 새로운 원소가 배출되지 않는다

### 콜드 플로우를 상태 플로우로

```kotlin
fun querySensor(): Int = Random.nextInt(-10..30)

fun getTemperatures(): Flow<Int> {
    return flow {
        while (true) {
            emit(querySensor())
            delay(500.milliseconds)
        }
    }
}

fun main() {
    val temps = getTemperatures()

    runBlocking {
        val tempState = temps.stateIn(this) // stateIn 함수 사용
        println(tempState.value)
        delay(800.milliseconds)
        println(tempState.value)
    }
}
```

- `shareIn` 함수와 다르게 시작 전략이 없다.
- 항상 코루틴 스코프 안에서 플로우가 시작되고, 코루틴 스코프가 취소될 때 까지 value 프로퍼티 전달하기 때문

---

### 상태 플로우와 공유 플로우

- 공유 플로우
  - 구독자가 구독 시작한 시점부터 이벤트 배출
  - 구독/구독취소 가능
- 상태 플로우
  - 상태를 나타냄
  - 플로우가 나타내는 값이 실제로 변경될때만 배출
  - 상대적으로 간단한 API 제공

---

- 공유 플로우는 최초 구독자가 나타나기 전에 메세지를 브로드캐스트 하면, 구독자는 아무런 메시지를 받을 수 없다.

```kotlin
class Broadcaster {
    private val _messages = MutableSharedFlow<String>()
    val messages = _messages.asSharedFlow()

    fun beginBroadcasting(scope: CoroutineScope) {
        scope.launch {
            _messages.emit("Hello!")
            _messages.emit("Hi!")
            _messages.emit("Hola!")
        }
    }
}

fun main(): Unit = runBlocking {
    val broadcaster = Broadcaster()
    broadcaster.beginBroadcasting(this)
    delay(200.milliseconds)
    broadcaster.messages.collect {
        println("Message: ${it}")
    }
}

output>

```

- 상태 플로우는 전체 메시지 기록을 리스트로 저장하면서, 구독자가 구독하기 전 모든 메세지에 쉽게 접근할 수 있다.

```kotlin
class Broadcaster {
    private val _messages = MutableStateFlow<List<String>>(emptyList())
    val messages = _messages.asStateFlow()

    fun beginBroadcasting(scope: CoroutineScope) {
        scope.launch {
            _messages.update { it + "Hello!" }
            _messages.update { it + "Hi!" }
            _messages.update { it + "Hola!" }
        }
    }
}

fun main() = runBlocking {
    val broadcaster = Broadcaster()
    broadcaster.beginBroadcasting(this)
    delay(200)
    println(broadcaster.messages.value)
}

output>
[Hello!, Hi!, Hola!]
```
