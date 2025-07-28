# 18장 코루틴 오류 처리와 테스트

**다루는 내용**

- 오류와 예외 발생 시 코드 동작 제어
- 오류 처리를 구조적 동시성 개념과 연결하는 방법
- 시스템 일부가 실패해도 정상적으로 작동하는 코드 작성법
- 동시성 코드를 위한 단위 테스트 작성법
- 테스트 실행 속도를 높이고 세밀한 동시성 제약 조건을 테스트하는 방법
- 터빈 라이브러리를 사용한 플로우 테스트

## 18.1 코루틴 내부에서 던져진 오류 처리

**코루틴 빌더는 새로운 코루틴을 생성하기 때문에 try-catch를 사용해도 효과가 없다**

- 새로 생성된 스레드의 예외가 잡히지 않는 것과 동일한 원리

**예외를 처리하는 방법 중 하나는 코루틴 빌더에 전달되는 람다 블록 안에 try-catch를 넣는 것이다**

- 예외가 코루틴 경계를 넘지 않기 때문

**async로 생성된 코루틴의 예외를 처리하고 싶다면?**

```kotlin
fun main(): Unit = runBlocking {
    val myDeferredInt: Deferred<Int> = async {
        throw UnsupportedOperationException("Ouch!")
    }
    try {
        val i: Int = myDeferredInt.await()
        println(i)
    } catch (u: UnsupportedOperationException) {
        println("Handled ${u.message}")
    }
}

[출력]
Handled Ouch!
Exception in thread "main" java.lang.UnsupportedOperationException: Ouch
```

- 코루틴 안에서 이미 예외가 발생했지만 `await`를 호출할 때 그 예외가 다시 발생!
    - 원래의 예외는 부모 코루틴으로 전파되어 그대로 예외가 터진 것
    - `await` 호출 시 예외 catch도 된 모습(원래의 예외와 같은 예외지만 다른 인스턴스)
- `await`로 기대하는 의미있는 값을 줄 수 없기에 발생시키는 것
- 따라서 `await`를 try-catch로 감싸면 예외 처리 가능하다

## 18.2 코루틴에서의 오류 전파

**구조적 동시성의 핵심: 부모-자식 코루틴 간의 생명주기와 예외를 체계적으로 관리하는 것**

자식에게 작업을 나누는 2가지 방식에 따라 자식의 오류를 처리하는 방식이 나뉜다

방식 1) 부모 코루틴 컨텍스트의 `Job`으로 작업을 나눈 경우

- 자식이 실패하면 모든 자식을 취소하는 코루틴

방식 2) 부모 코루틴 컨텍스트의 `SupervisorJob`으로 작업을 나눈 경우

- 하나의 자식이 실패해도 전체 실패로는 이어지지 않는 코루틴

### 18.2.1 부모 코루틴 컨텍스트의 `Job`으로 작업을 나눈 경우

**자식 코루틴에서 발생한 잡히지 않은 예외는 부모 코루틴에 전파되고 부모는 다음 3가지를 수행한다**

1. 불필요한 작업을 막기 위해 다른 모든 자식을 취소한다
2. 같은 예외를 발생시키면서 자신의 실행을 완료시킨다
3. 자신의 상위 계층으로 예외를 전파한다

```kotlin
fun main(): Unit = runBlocking {
    launch {
        try {
            while (true) {
                println("Heartbeat!")
                delay(500.milliseconds)
            }
        } catch (e: Exception) {
            println("Heartbeat stopped: ${e.message}")
            throw e
        }
    }
    launch {
        delay(1.seconds)
        throw UnsupportedOperationException("Ow!")
    }
}

[출력]
Heartbeat!
Heartbeat!
Heartbeat stopped: Parent job is Cancelling  // 자식 취소
Exception in thread "main" java.lang.UnsupportedOperationException: Ow! // 같은 예외 발생 (부모)
```

**구조적 동시성은 코루틴 스코프를 넘는 예외에만 영향을 미친다**

- 예외가 상위로 전파되려면 스코프를 넘는 처리되지 않은 예외여야 한다
- 따라서 전파를 막으려면 처음부터 스코프를 넘는 예외를 안던지면 된다
    - 코루틴 경계를 넘기 전 해당 코루틴에서 예외를 잡아버리면 된다

```kotlin
fun main(): Unit = runBlocking {
    launch {
        try {
            while (true) {
                println("Heartbeat!")
                delay(500.milliseconds)
            }
        } catch (e: Exception) {
            println("Heartbeat stopped: ${e.message}")
            throw e
        }
    }
    launch {
        try {
            delay(1.seconds)
            throw UnsupportedOperationException("Ow!")
        } catch (u: UnsupportedOperationException) {
            println("Caught exception: ${u.message}")
        }

    }
}

[출력]
Heartbeat!
Heartbeat!
Caught exception: Ow!
Heartbeat!
Heartbeat!
...
```

**중요) 코루틴에서 예외를 잡을 때 취소 예외 타입을 주의해야 한다**

- 취소 예외(`CancellationException`)는 코루틴 생명주기의 일부이기 때문에 잡아버리면 안된다
- 잡더라도 반드시 다시 예외를 던져주자

### 18.2.2 부모 코루틴 컨텍스트의 `SupervisorJob`으로 작업을 나눈 경우

**부모가 자식의 `Supervisor`(감독관)이 되기에 자식이 실패하더라도 생존한다**

- Job이랑 똑같은 역할이지만 예외를 전파하지 않고 다른 자식이 실패해도 취소되지 않는다
- 따라서 슈퍼바이저 코루틴으로 예외 전파의 경계를 정의할 수 있다

```kotlin
fun main(): Unit = runBlocking {
    supervisorScope {
        launch {
            try {
                while (true) {
                    println("Heartbeat!")
                    delay(500.milliseconds)
                }
            } catch (e: Exception) {
                println("Heartbeat stopped: ${e.message}")
                throw e
            }
        }
    }
    launch {
        delay(1.seconds)
        throw UnsupportedOperationException("Ow!")
    }
}

[출력]
Heartbeat!
Heartbeat!
Exception in thread "main" java.lang.UnsupportedOperationException: Ow!
Heartbeat!
Heartbeat!
...
```

**ktor는 슈퍼바이저 역할을 하는 Application 스코프를 지원한다**

- 다른 코루틴의 예외에도 전체 애플리케이션을 중단시키지 않는다
- 개별 요청 핸들러의 수명보다 더 오래 실행되는 코루틴을 시작할 때 쓴다

**Ktor는 각 요청의 생명주기에 맞춘 코루틴 관리를 위해 PipelineContext를 제공한다**

- 특정 요청 핸들러와 같은 수명을 가진 코루틴을 책임진다
    - 여기서 PipelineContext 내의 여러 코루틴이 함께 작업해 결과를 계산한다는 가정이 필요하다
- 즉, Job과 유사하게 요청이 완료되거나 취소되면 해당 코루틴들을 자동으로 정리해준다

## 18.3 CoroutineExceptionHandler: 예외 처리를 위한 마지막 수단

**예외가 슈퍼바이저 혹은 최상위 루트 코루틴에 도달하면 예외는 더 이상 전파되지 않는다.**

- 더 이상 전파될 수 없는 예외는 `CoroutineExceptionHandler`에서 예외를 잡아서 후처리한다
    - `CoroutineExceptionHandle`는 `CoroutineContext.Element`를 상속하고 있기에 코루틴 컨텍스트로 등록할 수 있다
    - 코루틴 컨텍스트에 예외 핸들러가 등록되지 않았다면 시스템 전역 예외 핸들러로 전달된다
    - 시스템 전역 예외 핸들러는 JVM의 경우 `UncaughtExceptionHandler`와 마찬가지로 핸들러가 예외 스택 트레이스를 오류 콘솔에 출력한다

**`CoroutineExceptionHandler` 예외 처리하는 방법**

1. `CoroutineExceptionHandler` 구현
    - 람다의 파라미터로 코루틴 컨텍스트와 처리되지 않은 예외 정보 전달
2. CoroutineScope 구성 시 구현한 `CoroutineExceptionHandler` 원소로 추가

**예시**

```kotlin
class ComponentWithScope(
    dispatcher: CoroutineDispatcher = Dispatchers.Default
) {
    // 1. 사용자 정의 예외 핸들러 구현
    private val exceptionHandler = CoroutineExceptionHandler { _, exception ->
        println("[ERROR] ${exception.message}")
    }
    
    // 2. SupervisorJob으로 스코프를 슈퍼바이저로 설정 및 디스패처와 예외 핸들러로 스코프 구성
    private val scope = CoroutineScope(
        SupervisorJob() + dispatcher + exceptionHandler
    )
    fun action() = scope.launch {
        throw UnsupportedOperationException("Ouch!")
    }
}

fun main() = runBlocking {
    val supervisor = ComponentWithScope()
    supervisor.action()
    delay(1.seconds)
}

[출력]
[ERROR] Ouch!
```

- `SupervisorJob`로 해당 컴포넌트의 스코프를 슈퍼바이저로 만들어 예외 전파 막음
- 예외 핸들러 코루틴 컨텍스트의 요소로 지정해 처리되지 않은 예외 처리
- 따라서 예외가 발생해도 애플리케이션 자체가 터지는 게 아니라 정의된 커스텀 핸들러에서 예외 처리

**중간에 있는 `CoroutineExceptionHandler`라는 것은 존재하지 않는다**

- 계층의 최상위까지 예외가 전파되기에 루트 코루틴이 아닌 코루틴의 컨텍스트에 설치된 핸들러는 절대 사용되지 않는다

```kotlin
private val topLevelHandler = CoroutineExceptionHandler { _, exception ->
    println("[TOP] ${exception.message}")
}

private val intermediateHandler = CoroutineExceptionHandler { _, exception ->
    println("[INTERMEDIATE] ${exception.message}")
}

@OptIn(DelicateCoroutinesApi::class)
fun main() = runBlocking {
    GlobalScope.launch(topLevelHandler) {
        launch(intermediateHandler) {
            throw UnsupportedOperationException("Ouch!")
        }
    }
    Thread.sleep(1000)
}

[출력]
[TOP] Ouch!
```

- 최상위 예외 핸들러만 작동하는 모습..!
- 중간에 예외가 터지고 그 컨텍스트에 예외 핸들러가 구현되어 있더라도 그 예외가 더 전파될 곳이 남아있기에 작동하지 않는다
- 최상위 코루틴인 `GlobalScope.launch`의 예외 핸들러만 호출된다

### 18.3.1 CoroutineExceptionHandler를 launch와 async에 적용할 때의 차이점

**`CoroutineExceptionHandler`는 최상위 코루틴이 `launch`로 생성된 경우에만 호출된다**

```kotlin
class ComponentWithScope(
    dispatcher: CoroutineDispatcher = Dispatchers.Default
) {
    private val exceptionHandler = CoroutineExceptionHandler { _, exception ->
        println("[ERROR] ${exception.message}")
    }
    private val scope = CoroutineScope(
        SupervisorJob() + dispatcher + exceptionHandler
    )
    fun action() = scope.launch {
        async {
            throw UnsupportedOperationException("Ouch!")
        }
    }
}

fun main() = runBlocking {
    val supervisor = ComponentWithScope()
    supervisor.action()
    delay(1000)
}

[출력]
[ERROR] Ouch!
```

- 최상위 코루틴이 `async`로 시작된 경우

```kotlin
class ComponentWithScope(
    dispatcher: CoroutineDispatcher = Dispatchers.Default
) {
    private val exceptionHandler = CoroutineExceptionHandler { _, exception ->
        println("[ERROR] ${exception.message}")
    }
    private val scope = CoroutineScope(
        SupervisorJob() + dispatcher + exceptionHandler
    )
    fun action() = scope.async {
        launch {
            throw UnsupportedOperationException("Ouch!")
        }
    }
}

fun main() = runBlocking {
    val supervisor = ComponentWithScope()
    supervisor.action()
    delay(1000)
}

// 출력 X
```

**launch와 async의 예외 처리 방식의 차이 때문!**

- `launch`는 즉시 예외를 최상위까지 전파하고 `CoroutineExceptionHandler`가 작동된다
- `async`는 결과를 기대하는 방식으로 지연된 예외 전파가 이루어진다
    - `await()`가 호출될 때까지 예외를 내부에 저장해두고 대기하기 때문에 호출하지 않으면 핸들러가 작동하지 않는다.
    - 즉, 예외를 처리하는 책임은 `await()`를 호출하는 `Deferred`의 소비자에게 있다 !
        - 코루틴 예외 핸들러는 이 예외를 무시할 수 있다
        - 소비자는 `await` 호출을 `try-catch`로 감싸 예외를 처리한다

하지만! `await`를 `try-catch`를 한다고 해서 원래의 예외가 잡히는 것은 아니라서 코루틴 취소에 영향을 줄 수 없다.

- 현 예제는 `SupervisorJob`으로 해둬서 다른 코루틴까지 퍼지진 않았겠지만 그냥 `Job`이었으면 처리되지 않은 예외가 스코프의 다른 코루틴들을 모두 취소시켰을 것이다

## 18.4 플로우 예외 처리

**플로우가 생성되거나 변환되거나 수집되는 중에 예외가 발생하면 collect에서 예외가 던져진다**

- `collect` 호출을 `try-catch`로 감싸면 예외를 처리할 수 있다
    - 역시나 `CancellationException`과 관련된 주의점은 지켜야 한다 (잡을거면 다시 던지기)
- 플로우에 중간 연산자가 적용되어 있는지 여부와는 관계없이 모든 예외가 수집 지점으로 전파된다

```kotlin
val exceptionalFlow = flow {
    repeat(5) { number ->
        emit(number)
    }
    throw UnhappyFlowException()
}

fun main() = runBlocking {
    val transformedFlow = exceptionalFlow.map { it * 2 }
    try {
        transformedFlow.collect { print("$it ") }
    } catch (u: UnhappyFlowException) {
        println("\nHandled: $u")
    }
}

[출력]
0 2 4 6 8 
Handled: UnhappyFlowException
```

### 18.4.1 catch 연산자로 업스트림 예외 처리

**플로우에서 발생한 예외를 처리하는 중간 연산자 catch**

- 오직 업스트림 예외만 처리한다
    - 플로우처리 파이프라인의 앞쪽에서 발생한 예외들만 잡아낸다
- `catch`에 연결된 람다 안에서 플로우에 발생한 예외에 접근 가능하다
    - 예외가 람다의 파라미터로 전달되어서 예외 종류별 분기 처리도 가능
- `catch`는 취소 예외(`CancellationException`)를 자동으로 인식한다
    - 즉, 취소가 발생하면 catch 블록이 실행되지 않고 취소가 정상적으로 전파
- 직접 값을 방출할 수 있어서 예외를 오류값으로 변환 가능하다
    - 다운스트림 플로우에서 정상적인 값처럼 소비도 가능

```kotlin
fun main() = runBlocking {
    exceptionalFlow
        .catch { cause ->
            println("\nHandled: $cause")  // 로그로 기록하고
            emit(-1)                      // 오류 값으로 -1 방출
        }
        .collect {
            print("$it ")                 // 방출된 오류값 -1 소비
        }
}

[출력]
0 1 2 3 4 
Handled: UnhappyFlowException
-1 
```

```kotlin
fun main() = runBlocking {
    exceptionalFlow
        .map { it + 1 }
        .catch { cause ->
            println("\nHandled: $cause") 
            emit(-1)   // 오류 값으로 -1 방출
        }
        .onEach {
            throw UnhappyFlowException()  // catch 이후의 다운 스트림이므로 예외 캐치 X
        }
        .collect {
            print("$it ")
        }
}

[예외 발생]
Exception in thread "main" UnhappyFlowException
```

### 18.4.2 retry 연산자로 플로우 수집 재시도

**플로우 처리 중 예외 발생 시 재시도를 지원하는 retry 중간 연산자**

- catch와 마찬가지로 오직 업스트림에 대해서만 동작한다
    - 업스트림에서 예외 발생 시 플로우를 처음부터 다시 시작한다
    - 지정된 최대 재시도 횟수만큼 자동으로 재시도를 수행한다
        - 지정된 재시도에도 실패하면 마지막 예외를 다운스트림으로 전파한다
- 조건부 재시도를 지원한다
    - 예외를 처리하고 `Boolean` 값을 반환하는 람다를 사용해 람다가 `true`를 반환하면 재시도가 시작된다
    - 특정 예외 타입에 대해서만 재시도하도록 조건을 설정할 수 있다
- 재시도 동안은 업스트림의 플로우가 처음부터 다시 수집되면서 모든 중간 연산이 다시 실행된다
    - 처음부터 다시 수집하기에 멱등성을 갖거나 반복 실행해도 괜찮은지 확인이 필요하다
- `CancellationException`를 자동으로 인식해 코루틴 취소 시 즉시 중단되고 취소를 전파한다

```kotlin
val unreliableFlow = flow {
    println("Starting the flow!") // 재시도마다 출력
    repeat(10) { number ->
        // 10% 확률로 CommunicationException 발생
        if (Random.nextDouble() < 0.1) throw CommunicationException()
        emit(number)
    }
}

fun main() = runBlocking {
    unreliableFlow
        .retry(5) { cause ->
            println("\nHandled: $cause")
            cause is CommunicationException // true일 때만 즉, CommunicationException 일때만 재시도
        }
        .collect { number ->
            print("$number ")
        }
}

[출력]
Starting the flow!
0 1 2 3 4 5 
Handled: CommunicationException: Communication failed!
Starting the flow!
0 1 
Handled: CommunicationException: Communication failed!
Starting the flow!
0 1 2 3 4 5 6 7 8 9 
```

## 18.5 코루틴과 플로우 테스트

**테스트에서 runBlocking이 비효율적인 이유**

- runBlocking은 일시 중단 함수, 코루틴, 플로우 사용 코드 모두 테스트할 수 있지만 실시간으로 실행되기 때문에 delay와 같은 지연되는 코드가 전부 실행된다
    - 예를 들어 과부하를 막기 위해 질의 사이에 지연을 둔 코드가 있는데 이걸 runBlocking으로 테스트하면 테스트에서도 그 지연이 실행돼서 불필요하게 테스트 속도가 느려진다

### 18.5.1 코루틴을 사용하는 테스트를 빠르게 만들기

**runTest를 사용해 가상 시간으로 테스트를 빠르게 실행하자**

- 가상 시간을 사용하면 지연이 자동으로 빠르게 진행되어 불필요한 시간을 줄일 수 있다

```kotlin
testImplementation("org.jetbrains.kotlinx:kotlinx-coroutines-test:1.7.3")
```

```kotlin
class PlaygroundTest {
    @OptIn(ExperimentalCoroutinesApi::class)
    @Test
    fun testDelay() = runTest {
        val startTime = System.currentTimeMillis()
        delay(100.seconds)
        println(System.currentTimeMillis() - startTime) // 1
        println(currentTime)  // 100000
    }
}
```

- 100초 딜레이를 걸었는데도 가상 시간으로 실행되어 실제 실행 시간은 매우 짧은 모습..!
- 가상 디스패처의 현재 시간이 궁금하면 `currentTime` 속성을 사용하면 된다

**runTest는 runBlocking과 마찬가지로 단일 스레드이다**

- 모든 자식 코루틴의 동시 실행되기 때문에 일시 중단 지점을 제공해야 한다
- 테스트 단언문 작성 시 일시 중단 지점 없으면 실행되게 할 수 있는 방법이 없기 때문에 실패한다

```kotlin
class PlaygroundTest {
    @Test
    fun testDelay() = runTest {
        var x = 0
        launch { x++ }
        launch { x++ }
        assertEquals(2, x) // 실패
    }
}
```

- `delay`나 `yield`같은 일시 중단 함수 호출을 추가해야 테스트가 통과된다

**runTest는 속도를 높이기 위해 특별한 테스트 디스패처와 스케줄러를 사용한다**

- 특별한 테스트 디스패처에서는 `TestCoroutineScheduler` 스케줄러를 통해 가상 시간을 제어할 수 있다
    - 이 스케줄러는 코루틴 컨텍스트의 일부로
- `runTest` 안에선 `TestScope` 스코프에 접근 가능하다
    - 이 스코프 안에서 확장 함수나 `testScheduler` 송성으로 `TestCoroutineScheduler` 스케줄러 기능을 사용할 수 있따

**TestCoroutineScheduler 스케줄러 핵심 함수**

1. `runCurrent`
    - 현재 실행되도록 스케줄되어 있는 모든 코루틴 실행
    - 즉시 실행할 새 코루틴이 예약되면 그 코루틴도 직접 실행된다
2. `advanceUntilIdle`
    - 미래의 어느 시점에 실행되도록 예약된 코루틴까지 실행
    - 예약된 모든 코루틴을 실행한다

```kotlin
		@OptIn(ExperimentalCoroutinesApi::class)
    @Test
    fun testDelay() = runTest {
        var x = 0
        launch {
            x++
            launch { x++ }
        }
        launch {
            delay(200.milliseconds)
            x++
        }
        runCurrent()
        assertEquals(2, x)
        advanceUntilIdle()
        assertEquals(3, x)
    }
```

### 18.5.2 터빈으로 플로우 테스트하기

**터빈 라이브러리 핵심 기능은 확장함수 test**

1. 자동 플로우 수집
    - 새 코루틴을 실행하고 내부적으로 플로우 수집
    - 개발자가 직접 수집 로직 작성하지 않아도 된다
2. 순차적으로 검증할 수 있는 API 제공
    - API를 테스트 단언문과 함께 사용 가능
        - **`awaitItem()`**: 다음 방출값을 기다리고 반환
        - **`awaitComplete()`**: 플로우 정상 완료 확인
        - **`awaitError()`**: 예외 발생 확인
    - 플로우에 대한 불변 조건 지정하고 검증도 가능
3. 플로우가 방출한 모든 원소가 완전히 소비되도록 보장
    - 누락된 원소나 예상치 못한 방출을 자동으로 감지

```kotlin
    testImplementation("app.cash.turbine:turbine:1.0.0")
```

```kotlin
    val myFlow = flowOf(1, 2, 3)

    @Test
    fun doTest() = runTest {
        val results = myFlow.test {
            assertEquals(1, awaitItem())
            assertEquals(2, awaitItem())
            assertEquals(3, awaitItem())
            awaitComplete()
        }
    }
```