# 16장 플로우

플로우를 사용하면 시간이 지남에 따라 나타나는 여러 값을 다루는 상황에서 코틀린의 동시성 매커니즘을 활용할 수 있다. 플로우의 여러 유형과 이를 생성, 변환 소비하는 방법을 다룬다.

---

## 16.1 플로우는 연속적인 값의 스트림을 모델링한다

### 일시 중단 함수는 “한번에” 값을 반환한다

- 기존의 `suspend fun`은 **여러 값을 순차적으로 생성할 수 있지만,** 그 결과를 **한꺼번에** 반환한다.

    ```kotlin
    import kotlinx.coroutines.delay
    import kotlinx.coroutines.runBlocking
    import kotlin.time.Duration.Companion.seconds
    
    suspend fun createValues(): List<Int> {
        return buildList {
            add(1)
            delay(1.seconds)
            add(2)
            delay(1.seconds)
            add(3)
            delay(1.seconds)
        }
    }
    
    fun main() = runBlocking {
        val list = createValues()
        list.forEach { log(it) }
    }
    ```

    - `createValues()`는 리스트를 반환하지만 **모든 값이 준비될 때까지 기다렸다가 한꺼번에 반환**한다.

        ```kotlin
        [3초 후]
        [main #coroutine#1] 1
        [main #coroutine#1] 2
        [main #coroutine#1] 3
        ```

    - 첫 번째 값이 1초 후에 준비되더라도 우리는 **3초 전체를 기다린 후**에야 사용할 수 있다.


### 플로우(Flow): 값을 **하나씩, 지연되며**, **스트림처럼** 흘려보낼 수 있다

- 코루틴 기반의 추상화인 **Flow**를 사용하면 값이 준비되는 즉시 하나씩 "emit"해서 소비할 수 있다.
- `emit()`으로 값을 보내고, `collect()`로 값을 받는다.

    ```kotlin
    import kotlinx.coroutines.delay
    import kotlinx.coroutines.runBlocking
    import kotlinx.coroutines.flow.*
    import kotlin.time.Duration.Companion.milliseconds
    
    fun createValues(): Flow<Int> {
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
        myFlowOfValues.collect { log(it) }
    }
    
    ```

    - 값이 배출되자마자 출력된다.

        ```kotlin
        [main #coroutine#1] 1
        [main #coroutine#1] 2
        [main #coroutine#1] 3
        ```

- 즉, **계산된 값이 준비되자마자 바로 처리**할 수 있다.
- UI 업데이트, 네트워크 응답 처리 등에 아주 유용하다.

### Flow는 어떻게 동작할까?

- **flow { }** 안에서 `emit()`으로 값을 내보낸다.
- `collect { }` 로 그 값을 받아 처리한다.
- 기본적으로 Flow는 **콜드(Cold)** 하다.
    - → 수집이 시작되어야 `emit()`이 작동한다.
    - → `collect`가 호출되지 않으면 아무 일도 일어나지 않는다.

### ❄️ 콜드 vs 🔥 핫 플로우

| 구분 | 콜드 플로우 (Cold Flow) | 핫 플로우 (Hot Flow) |
| --- | --- | --- |
| 동작 시점 | `collect()` 호출 시 시작 | 생성되자마자 동작 |
| 수집자 | 각 수집자는 독립적 | 여러 수집자가 동시에 같은 값을 받음 |
| 예시 | `flow {}` | `StateFlow`, `SharedFlow` |

---

## 16.2 콜드 플로우

### 콜드 플로우란?

- 플로우는 **“시간에 따라 계산되는 값들의 스트림”**을 표현하는 추상화다.
- 그 중 콜드 플로우는 **수집(collect)**이 시작되기 전까지는 아무 일도 하지 않는 비활성 상태의 흐름이다.
- 마치 Sequence처럼 필요할 때 값을 계산하는 **지연 계산(lazy evaluation)**의 성격을 갖는다.

### `flow` 빌더로 콜드 플로우 생성하기

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*
import kotlin.time.Duration.Companion.milliseconds

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

- `flow {}` 안에서 `emit(value)`로 값을 배출한다.
- 이 코드 **자체로는 실행되지 않음**. 왜냐면 `flow {}`는 단순히 **설명서**일 뿐이고, collect가 호출돼야 실행된다.
- 즉, **수집자가 있어야** 이 flow가 일을 시작한다.
- 무한 스트림 만들기

    ```kotlin
    val counterFlow = flow {
        var x = 0
        while (true) {
            emit(x++)
            delay(200.milliseconds)
        }
    }
    ```

    - 이런 무한 스트림도, 실제로 수집하지 않으면 아무 일도 안 일어난다.

### 플로우는 수집되기 전까지 동작하지 않는다

- `collect()`를 호출하면 flow가 비로소 동작을 시작한다.

    ```kotlin
    import kotlinx.coroutines.*
    import kotlinx.coroutines.flow.*
    import kotlin.time.Duration.Companion.milliseconds
    
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
    ```

    - 실행 결과

        ```kotlin
        [main @coroutine#1] Emitting A!
        [main @coroutine#1] Collecting A
        [main @coroutine#1] Emitting B!
        [main @coroutine#1] Collecting B
        ```

    - `emit` → `collect` → `delay` → 다시 `emit` 순서로 실행됨.
    - **모두 같은 코루틴 안**에서 실행되기 때문에 **완전한 순차 실행**임.
    - `emit()`은 내부적으로 다음 수집자 작업이 끝날 때까지 **중단(suspend)** 된다.
    - 정리 흐름

        ```kotlin
        [flow] emit("A")  -->  [collector] println("A") + delay
                ↓                         ↑
        [flow] emit("B")  <--  [collector] println("B") + delay
        ```


### 플로우 수집 취소

- `collect()`는 보통 flow가 끝날 때까지 suspend 된다. 하지만 중간에 취소할 수도 있다.

    ```kotlin
    import kotlinx.coroutines.*
    import kotlinx.coroutines.flow.*
    import kotlin.time.Duration.Companion.seconds
    
    fun main() = runBlocking {
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

- `emit()`은 suspend 함수이기 때문에 중간에 **취소 포인트로 작동**한다.
- 따라서 `delay` 또는 `emit` 중간에 취소되면 흐름이 멈춘다.

### 콜드 플로우의 내부 구조

- 콜드 플로우는 사실 매우 간단한 두 인터페이스만으로 구현된다.

    ```kotlin
    interface Flow<T> {
        suspend fun collect(collector: FlowCollector<T>)
    }
    
    interface FlowCollector<T> {
        suspend fun emit(value: T)
    }
    ```

    - 즉, 플로우의 본질은 **emit으로 값을 전달하고**, collect로 값을 소비하는 구조이다.

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

    - `emit()`은 `collect()` 안에서 받은 람다를 호출한다.
    - 둘은 서로 **번갈아가며** 실행되는 구조다.

### 병렬 처리를 위한 `channelFlow`

- 문제 상황

    ```kotlin
    import kotlinx.coroutines.*
    import kotlinx.coroutines.flow.*
    import kotlin.random.Random
    import kotlin.time.Duration.Companion.milliseconds
    
    suspend fun getRandomNumber() : Int {
    	delay(500.milliseconds)
    	return Random.nextInt();
    }
    
    val randomNumbers = flow {
        repeat(10) {
            emit(getRandomNumber())  // 500ms씩 지연
        }
    }
    
    fun main() = runBlocking {
    	randomNumbers.collects {
    		log(it)
    	}
    }
    ```

    - 총 10개의 숫자를 1개씩 순차적으로 생성 → 5초 소요
- 해결 방법: `channelFlow`

    ```kotlin
    val randomNumbers = channelFlow {
        repeat(10) {
            launch {
                send(getRandomNumber())  // 병렬로 수행
            }
        }
    }
    ```

    - `channelFlow`는 여러 코루틴에서 `send()`로 값을 보낼 수 있다.
    - 내부적으로 **채널(Channel)** 을 사용해 수집자에게 값을 전달한다.
    - **collect는 여전히 순차적이지만, emit은 병렬로 가능하다.**
- 실행 결과는 이렇게 빨라진다!

    ```kotlin
    [main] -19273893727
    [main] 22239348939
    ...
    [main] 45489899999
    
    → 전체 수행 시간 500ms 내외
    ```


- 일반 `flow` vs `channelFlow` 비교

    | 항목 | `flow` | `channelFlow` |
    | --- | --- | --- |
    | 실행 방식 | 순차 실행 | 병렬 실행 가능 (코루틴에서 send) |
    | emit 가능 위치 | 동일한 코루틴 내에서만 | 여러 코루틴에서 가능 |
    | 성능/오버헤드 | 가볍고 단순 | 약간 무겁고 내부에 채널 구조 포함 |
    | 사용 목적 | 일반적인 비동기 스트림 | 동시적인 데이터 생성 필요 시 |
    - 따라서 가능하면 `flow`를 쓰고, **여러 코루틴에서 값을 동시에 배출해야 할 때만** `channelFlow`를 쓰자!

---

## 16.3 핫 플로우

- 콜드 플로우와 배출과 수집이라는 전체적 구조는 동일하지만 다른 여러 속성을 가지고 있다.
- 각 수집자가 플로우 로직 실행을 독립적으로 촉발하는 대신, 여러 구독자라고 불리는 수집자들이 배출된 항목을 공유한다.
- 핫 플로우의 2가지 구현체
    - shared flow(공유 플로우)
        - 값을 브로드캐스트하기 위해 사용된다.
        - 여러 수집자에게 값을 전파할 수 있다.
    - state flow(상태 플로우)
        - 상태를 전달하는 특별한 경우에 사용된다.
        - 현재 상태(state)를 기억하고, 최신 값만 보존한다.

### 공유 플로우는 값을 구독자에게 브로드캐스트한다

- 공유 플로우(SharedFlow)는 일반 `Flow` 와 다르게, **값을 브로드캐스트(여러 구독자에게 동시에 전송)**하는 방식으로 동작한다.
- 즉, **emit() 호출 시점에 존재하는 모든 구독자에게 같은 데이터를 전달**하는 구조이다.
- 이는 라디오 방송국과 비슷함.
  방송국은 누가 듣고 있든 상관없이 계속 방송하고, 라디오를 켠 사람(구독자)은 그때부터 방송을 수신한다.

    ```kotlin
    import kotlinx.coroutines.*
    import kotlinx.coroutines.flow.*
    import kotlin.random.*
    import kotlin.time.Duration.Companion,milliseconds
    
    class RadioStation {
    	private val _messageFlow = MutableSharedFlow<Int>()
    	val messageFlow = _messageFlow.asSharedFlow()
    	
    	fun beginBroadcasting(scope: CoroutineScope) {
    		scope.launch {
    			while(true) {
    				delay(500.milliseconds)
    				val number = Random.nextInt(0..10)
    				log("Emitting $number!")
    				_messageFlow.emit(number)
    			}
    		}
    	}
    }
    ```

    - `_messageFlow`: 내부에서만 사용할 수 있는 가변 공유 플로우
    - `messageFlow`: 외부에는 읽기 전용으로 노출
    - `beginBroadcasting()`: 주어진 스코프에서 값을 계속 배출하는 브로드캐스트 코루틴 실행
    - SharedFlow는 Flow 빌더로 만들지 않고, `MutableSharedFlow`를 직접 인스턴스로 만듬!
    - `emit()`으로 값을 내보내면, 모든 활성화된 수집자에게 전달됨
    - 콜드 플로우와 달리 *“*수집자가 없어도 emit은 계속됨*”*

- 네이밍 컨벤션

    ```kotlin
    private val _messageFlow = ...
    val messageFlow = _messageFlow.asSharedFlow()
    ```

    - 이런 식으로 `_`를 붙여 private 값을 구분하고, 외부에 노출할 땐 `_`를 제거하는 패턴
    - 코틀린 1.x에서는 같은 프로퍼티에 대해 `private`과 `public` 타입을 다르게 지정할 수 없음.
      그래서 타입 보호(정보 은닉)를 위해 이런 관례를 따름.
    - → 향후 코틀린 2.x에서는 이걸 언어 차원에서 지원할 예정!

- 실행 예제: 구독자가 없어도 방송됨

    ```kotlin
    fun main() = runBlocking {
        RadioStation().beginBroadcasting(this)
    }
    ```

    - 출력

        ```kotlin
        575 [main @coroutine#2] Emitting 2!
        1068 [main @coroutine#2] Emitting 10!
        1593 [main @coroutine#2] Emitting 4!
        ```

        - 즉, **구독자가 없어도 브로드캐스트는 계속된다.**

- 실행 예제: 늦게 구독한다면?

    ```kotlin
    fun main() = runBlocking {
        val radioStation = RadioStation()
        radioStation.beginBroadcasting(this)
        delay(600.milliseconds)
    
        radioStation.messageFlow.collect {
            log("A collecting $it!")
        }
    }
    ```

    - 출력

        ```kotlin
        611 [main @coroutine#2] Emitting 8!
        1129 [main @coroutine#2] Emitting 9!
        1131 [main @coroutine#1] A collecting 9!
        1647 [main @coroutine#2] Emitting 1!
        1647 [main @coroutine#1] A collecting 1!
        ```

        - 구독자는 **구독한 이후의 값만** 받을 수 있다.

- 다수 구독자는?

    ```kotlin
    launch {
        radioStation.messageFlow.collect {
            log("B collecting $it!")
        }
    }
    ```

    - 여러 구독자는 **동일한 값을 동시에 수신**할 수 있다.
    - 즉, 하나의 방송을 여러 명이 함께 듣는 것과 같음.

- 구독자를 위한 값 재생
    - replay 파라미터를 사용해 새 구독자를 위해 제공할 값의 캐시를 설정할 수 있다.

        ```kotlin
        private val messageFlow = MutableSharedFlow<Int>(replay = 5)
        ```

        - 위처럼 설정하면

            ```kotlin
            560 [main] Emitting 6!
            635 [main] A collecting 6!
            ```

            - 구독 직전에 emit된 값을 최대 5개까지 되돌려서 받을 수 있음!

- shareIn으로 콜드 플로우를 공유 플로우로 전환
    - `shareIn` 함수를 사용하면 주어진 콜드 플로우를 한 플로우인 공유 플로우로 변환할 수 있다.
    - 콜드 플로우 → 핫 플로우 변환 예시

        ```kotlin
        import kotlinx.coroutines.*
        import kotlinx.coroutines.flow.*
        import kotlin.random.*
        import kotlin.time.Duration.Companion.milliseconds
        
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

        - 이걸 두 번 collect하면 센서에 2번 요청 → 비효율!
    - 해결법: shareIn 사용

        ```kotlin
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
        ```

        - 공유된 단일 소스에서 두 collect가 동일한 값을 수신하게 된다.
    - SharingStarted 설정 값
        - `Eagerly`: 즉시 수집 시작
        - `Lazily`: 첫 구독자부터 시작
        - `WhileSubscribed`: 구독자가 없으면 정지

### 시스템 상태 추적: 상태 플로우

- 동시성 시스템에서는 시간이 지나며 변화하는 값을 추적해야 하는 상황이 자주 발생한다.
- 이런 값을 안정적으로 관리하려면 단순한 `Flow`보다 더 특화된 도구가 필요하다.
  그래서 코틀린은 `StateFlow` 라는 특수한 공유 플로우를 제공한다.
- 상태 플로우란?
    - `StateFlow`는 **값을 하나만 유지하면서** 그 값을 구독자에게 **실시간으로 전달**하는 공유 플로우
    - 내부적으로 **초깃값**을 반드시 가지며, 값이 **실제로 달라질 때만 emit(배출)**된다.
- 핵심 주제 4가지
    - 상태 플로우를 생성하고 구독자에게 노출하는 방법
    - 상태 플로우의 값을 안전하게 갱신하는 방법
    - 값이 실제로 변경될 때만 배출하게 하는 “동등성 기반 통합”
    - 콜드 플로우를 상태 플로우로 변환하는 방법

- 예제: 간단한 뷰 카운터

    ```kotlin
    import kotlinx.coroutines.*
    import kotlinx.coroutines.flow.*
    
    class ViewCounter {
        private val _counter = MutableStateFlow(0)
        val counter = _counter.asStateFlow()
    
        fun increment() {
            _counter.update { it + 1 }
        }
    }
    
    fun main() {
        val vc = ViewCounter()
        vc.increment()
        println(vc.counter.value) // 1
    }
    ```

    - `_counter`: 내부에서 사용하는 `MutableStateFlow`
    - `counter`: 외부에 읽기 전용으로 노출 (`asStateFlow`)
    - `increment()`에서는 `update()`를 사용해 값을 1 증가시킴
    - `StateFlow.value`를 통해 값을 **일시 중단 없이 동기적으로 읽을 수 있음**
    - `update()`는 안전한 갱신을 위해 사용하는 고차 함수

- UPDATE 함수로 안전하게 상태 플로우에 쓰기

    ```kotlin
    fun increment() {
        _counter.value++
    }
    ```

    - 겉보기에 단순하지만 **이 연산은 원자적이지 않음** (스레드 안전하지 않음!)
    - 병렬 환경에서 잘못된 결과

        ```kotlin
        fun main() {
            val vc = ViewCounter()
            runBlocking {
                repeat(10_000) {
                    launch { vc.increment() }
                }
            }
            println(vc.counter.value) // 기대값: 10000 / 실제값: 훨씬 낮음
        }
        ```

        - `_counter.value++`는 3단계 작업:
            1. 현재 값 읽기
            2. +1 계산
            3. 새 값 쓰기
        - 여러 스레드에서 동시에 실행되면 race condition 발생!
          한쪽의 연산이 덮어씌워져서 값 손실됨.
    - 해결책: `update()` 사용

        ```kotlin
        fun increment() {
            _counter.update { it + 1 }
        }
        ```

        - `update()`는 내부적으로 값을 원자적으로 읽고-계산하고-반영해줌
        - 병렬 환경에서도 정확한 결과 보장!

- 상태 플로우는 값이 실제로 달라졌을 때만 값을 배출한다: 동등성 기반 통합
    - 상태 플로우도 `collect` 함수를 호출해 시간에 따라 값을 구독할 수 있다.
    - 예제: 방향 스위치

        ```kotlin
        import kotlinx.coroutines.*
        import kotlinx.coroutines.flow.*
        
        enum class Direction { LEFT, RIGHT }
        
        class DirectionSelector {
            private val _direction = MutableStateFlow(Direction.LEFT)
            val direction = _direction.asStateFlow()
        
            fun turn(d: Direction) {
                _direction.update { d }
            }
        }
        ```

    - 구독자 코드

        ```kotlin
        fun main() = runBlocking {
            val switch = DirectionSelector()
        
            launch {
                switch.direction.collect {
                    println("Direction now $it")
                }
            }
        
            delay(200)
            switch.turn(Direction.RIGHT)
        
            delay(200)
            switch.turn(Direction.LEFT)
        
            delay(200)
            switch.turn(Direction.LEFT) // 중복 값
        }
        ```

        - 결과 값 출력

            ```kotlin
            37 [main @coroutine#2] Direction now LEFT
            240 [main @coroutine#2] Direction now RIGHT
            445 [main @coroutine#2] Direction now LEFT
            ```

            - **마지막 `LEFT`는 emit되지 않음!**
    - 이유: **동등성 기반 통합(equality-based conflation)**
        - `StateFlow`는 **값이 달라졌을 때만 emit**
        - 같은 값을 설정하면 emit 생략됨

- stateIn으로 콜드 플로우를 상태 플로우로 변환하기
    - `stateIn()`을 사용하면 콜드 플로우를 상태 플로우로 바꿀 수 있다.

        ```kotlin
        val temps = getTemperatures()
        
        val tempState = temps.stateIn(this)
        ```

        - → 여기서 `getTemperatures()`는 콜드 플로우라고 가정
    - 예제: `stateIn` 변환

        ```kotlin
        import kotlinx.coroutines.*
        import kotlinx.coroutines.flow.*
        import kotlin.time.Duration.Companinon.milliseconds
        
        fun main() {
        	val temps = getTemperatures()
        	
        	runBlocking {
        		val tempState = temps.stateIn(this)
        		println(tempState.value)
        		delay(800.milliseconds)
        		println(tempState.value)
        		// 18
        		// -1
        	}
        }
        ```

    - `stateIn()`은 `shareIn()`처럼 **공유된 플로우 생성**
    - 항상 **가장 최신 값**을 `value`로 제공
    - `stateIn()`은 내부적으로 **코루틴 스코프에서 즉시 실행**되고, **스코프가 취소될 때까지 유지**

### 상태 플로우와 공유 플로우의 비교

| 플로우 종류 | 특징 |
| --- | --- |
| **SharedFlow** | 이벤트 중심 (구독 중일 때만 배출됨) |
| **StateFlow** | 상태 중심 (항상 최신 값을 유지) |
- 둘 다 구독자 유무와 관계없이 값을 배출할 수 있지만, 사용 방식은 다르다.
- SharedFlow vs StateFlow 비교

    | 항목 | SharedFlow | StateFlow |
    | --- | --- | --- |
    | 배출 방식 | 구독자가 있어야 배출됨 | 값이 **변경**되면만 배출됨 |
    | 값 유지 | X (기록 안 함) | O (항상 최신 값 유지) |
    | 복잡성 | 조금 복잡 (재생 캐시 등 고려 필요) | 상대적으로 단순함 |
    | 적합한 상황 | 실시간 이벤트, 알림 등 | 상태 추적 (예: 로그인 상태 등) |

- 예제 1: SharedFlow의 단점

    ```kotlin
    import kotlinx.coroutines.flow.*
    import kotlinx.coroutines.*
    import kotlin.time.Duration.Companion.milliseconds
    
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
        delay(200)
        broadcaster.messages.collect {
            println("Message: $it")
        }
    }
    // 출력 없음!
    ```

    - 왜 출력이 없을까?
        - `emit()`이 먼저 실행됨 → **구독자가 없었기 때문**에 아무도 받지 못함
        - 이건 SharedFlow의 기본 동작임!
- 해결책: StateFlow로 메시지 기록 저장

    ```kotlin
    import kotlinx.coroutines.flow.*
    import kotlinx.coroutines.*
    import kotlin.time.Duration.Companion.milliseconds
    
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
    // 출력: [Hello!, Hi!, Hola!]
    ```

    - 모든 메시지를 리스트에 저장 → **이후 구독자도 전체 기록을 확인 가능**
    - 더 간단하고 예측 가능함

### 언제 어떤 플로우를 사용할까?

- 플로우 종류 요약

    | 항목 | 콜드 플로우 | 핫 플로우 |
    | --- | --- | --- |
    | 기본 상태 | 비활성 (수집 시 실행) | 항상 활성 |
    | 수집자 수 | 1명 | 여러 명 |
    | 완료 여부 | 완료됨 | 일반적으로 완료되지 않음 |
    | 값 배출 시작 | 수집자 생긴 후 | 언제든 배출 가능 |
    | 값 배출 주체 | 하나의 코루틴 | 여러 코루틴 가능 |
- 사용 지침

    | 사용 상황 | 추천 플로우 |
    | --- | --- |
    | DB 조회, 네트워크 요청 | **Cold Flow** |
    | 버튼 클릭, 알림 이벤트 | **SharedFlow** |
    | 로그인 상태, UI 상태 추적 | **StateFlow** |
- 반응형 라이브러리와의 연동
    - Kotlin Flow는 **RxJava**, **Reactor**, **Reactive Streams**와 상호운용 가능
    - 변환 함수가 이미 내장되어 있음
        - Rx → Flow
        - Flow → Rx 등
    - 공식 문서 참고: https://kotlinlang.org/api/kotlinx.coroutines

---

## 요약

- 코틀린 플로우는 시간이 지남에 따라 발생하는 값을 처리할 수 있는 코루틴 기반의 추상화다.
- 플로우에는 핫 플로우와 콜드 플로우라는 2가지 유형이 있다.
- 콜드 플로우는 기본적으로 비활성 상태이며, 하나의 수집자와 연결된다.
  `flow` 빌더 함수로 콜드 플로우를 생성하며, `emit` 함수로 비동기적으로 값을 제공한다.
- 채널 플로우는 콜드 플로우의 특수 유형으로, 여러 코루틴에서 `send` 함수를 통해 값을 배출할 수 있다.
- 핫 플로우는 항상 활성 상태이며, 여러 구독자와 연결된다. 공유 플로우와 상태 플로우는 핫 플로우의 예다.
- 코루틴 간에 값을 브로드캐스트 방식으로 전달하는 데 공유 플로우를 사용 할 수 있다.
- 공유 플로우의 구독자는 구독을 시작한 시점부터 배출된 값을 받으며, 재생된 값도 수신할 수 있다.
- 동시성 시스템에서 상태를 관리할 때 상태 플로우를 사용할 수 있다.
- 상태 플로우는 동등성 기반 통합을 수행한다. 이는 값이 실제로 변경된 경우에만 배출이 발생하고, 같은 값이 여러 번 대입되면 배출이 발생하지 않는다는 뜻이다.
- `shareIn`이나 `stateIn` 함수를 통해 콜트 플로우를 핫 풀로우로 전환할 수있다.
