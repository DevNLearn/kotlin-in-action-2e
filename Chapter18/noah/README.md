# 오류 처리와 테스트
## 1. 코루틴 내부에서 던져진 오류 처리

코루틴 빌더는 새로운 코루틴에서 발생한 예외는 `catch` 블록에 의해 잡히지 않는다.

```kotlin
fun main(): Unit = runBlocking {
    try {
        launch {
            throw UnsupportedOperationException("Ouch!")
        }
    } catch(u: UnsupportedOperationException) {
        println("Handled $u")  // 실행되지 않음
    }
}

// Exception in thread "main" java.lang.UnsupprotedOperationException: Ouch!
```

이 예외를 올바르게 처리하기 위해서는 `launch`에 전달되는 람다 블록 안에 `try-catch` 블록을 넣는것이다.

예외가 코루틴 경계를 넘지 않기 때문에 코루틴이 없을때 처럼 실행된다.

```kotlin
fun main(): Unit = runBlocking {
    launch {
        try {   
            throw UnsupportedOperationException("Ouch!")
        } catch(u: UnsupportedOperationException) {
            println("Handled $u")
        }
    }
}

// Handled java.lang.UnsupportedOperationException: Ouch!
```

`async`로 생성된 코루틴이 예외를 던진다면 `await`를 호출할때 예외가 발생한다.

```kotlin
fun main(): Unit = runBlocking {
    val myDeferredInt: Deferrerd<Int> = async {
        throw UnsupportedOperationException("Ouch!")
    }
    try {
        val i: Int = myDeferredInt.await()
        println(i)
    } catch (u: UnsupprotedOperationException) {
        println("Handled: $u")
    }
}
// Handled: java.lang.UnsupprotedOperationException: Ouch!
// Exception in thread "main" java.lang.UnsupprotedOperationException: Ouch!
// ...
```

`await` 를 감싼 `try-catch` 에서 예외를 잡지만, 동시에 오류 콘솔에도 예외가 출력된다. `await` 가 예외를 다시 던지지만 원래의 예외도 여전히 관철되기 때문이다.

## 2. 코틀린 코루틴에서의 오류 전파

구조적 동시성 패러다임은 자식 코루틴에서 발생한 잡히지 않은 예외가 부모 코루틴에 영향을 준다.

자식에게 작업을 나누는 방식에 따라 자식의 오류를 처리하는 방식도 달라진다.

**코루틴이 작업을 동시적으로 분해해 처리하는 경우 - Job**

자식 중 하나의 실패는 더 이상 최종 결과를 얻을 수 없음으로 한 자식의 실패는 부모의 실패로 이어진다.

- 부모 코루틴도 예외로 완료돼야 한다.
- 여전히 작업 중인 다른 자식은 더이상 필요가 없기 때문에 취소한다.

**하나의 자식이 실패해도 전체 실패로는 이어지지 않는 경우 - SupervisorJob**

자식의 실패로 인해 시스템 전체가 실패하면서 멈추치 않아야 하기 때문에 부모가 자식들에게 벌어진 실패를 처리해야 하며 자식이 부모의 실행을 감독한다고 한다.

- 감독 코루틴은 일반적으로 코루틴 계층의 최상위에 위치한다.
- ex) 데이터를 가져오는 작업이 실패하더라도 UI 구성 요소는 살아 있어야 한다.

### 자식이 실패하면 모든 자식을 취소하는 코루틴

코루틴 간의 부모-자식 계층이 Job 객체를 통해 구축되어 코루틴이 SuperVisorJob 없이 생성된 경우 자식 코루틴에서 발생한 잡히지 않은 예외는 부모 코루틴을 예외로 완료시키는 방식으로 처리된다.

실패한 자식 코루틴은 자신의 실패를 부모에게 전파하며 부모는 다음을 수핸한다.

- 불필요한 작업을 막기 위해 다른 모든 자식을 취소
- 같은 예외를 발생시키면서 자신의 실행을 완료
- 자신의 상위 계층으로 예외를 전파

```kotlin
fun main(): Unit = runBlocking {
    launch {
        try {
            while (true) {
                println("Heartbeat!")
                delay(500.milliseconds)
            }
        } catch (e: Exception) {
            println("Heartbeat terminated: $e")
            throw e
        }
    }
    launch {
        delay(1.seconds)
        throw UnsupportedOperationException("Ow!")
    }
}

/*
Heartbeat!
Heartbeat!
Heartbeat terminated: kotlinx.coroutines.JobCancellationException: Parent job 
is Cancelling; job=BlockingCoroutine{Cancelling}@1517365b
Exception in thread "main" java.lang.UnsupprotedOperationException: Ow!
*/
```

### 구조적 동시성은 코루틴 스코프를 넘는 예외에만 영향을 미친다.

형제 코루틴을 취소하고 예외를 코루틴 계층 상위로 전파하는 동작은 `코루틴 스코프`를 넘는 처리되지 않은 예외에만 영향을 미친다.

```kotlin
fun main(): Unit = runBlocking {
    launch {
        try {
            while (true){
                println("Heartbeat!")
                delay(500.milliseconds)
            }
        } catch (e: Exception) {
            println("Heartbeat Terminated: $e")
            throw e
        }
    }
    launch {
         try {
             delay(1.seconds)
             throw UnsupportedOperationException("Ow!")
         } catch(u: UnsupprotedOperationException) {
             println("Caught $u")
         }
    }
}

/*
Heartbeat!
Heartbeat!
Caught java.lang.UnsupprotedOperationException: Ow!
Heartbeat!
Heartbeat!
*/
```

### 슈퍼바이저는 부모와 형제가 취소되지 않게 한다.

`슈퍼바이저`는 자식이 실패하더라도 생존한다. 슈퍼바이저는 다른 자식 코루틴을 취소하지 않으며 예외를 구조적 동시성 계층 상위로 전파하지 않는다. 이 때문에 종종 코루틴 계층의 최상위 코루틴으로서 슈퍼바이저가 쓰인다.

코루틴이 자식 코루틴의 슈퍼바이저가 되려면 그 코루틴에 연관된 `Job`이 일반적으로 `SupervisorJob`이어야 한다.

- SupervisorJob은 예외를 부모에게 전파하지 않는다.
- SupervisorJob은 다른 자식 잡업이 실패해도 취소하지 않게 한다.
- SupervisorJob도 구조적 동시성에 참여해 취소될 수 있고 취소 예외를 올바르게 전파할 수 있다.

슈퍼바이저의 동작을 직접 확인하려면 supervisorScope 함수를 사용해 스코프를 만들 수 있다.

```kotlin
fun main(): Unit = runBlocking {
    supervisorScope {
        launch {
            try {
                while(ture) {
                    println("Heartbeat!")
                    delay(500.milliseconds)
                }
            } catch (e: Exception) {
                println("Heartbeat terminated: $e")
                throw e
            }
        }
        launch {
            delay(1.secondes)
            throw UnsupprotedOperationException("Ow!")
        }
    }
}
/*
Heartbeat!
Heartbeat!
Exception in thread "main" java.lang.UnsupprotedOperationException: Ow!
...
Heartbeat!
Heartbeat!
...
*/
```

슈퍼바이저가 자식 코루틴이 부모 코루틴을 취소하지 못하게 막은 것이다. `SupervisorJob`이 `launch` 빌도로 시작된 자식 코루틴에 대해 `CoroutineExceptionHandler`을 호출하기 때문이다.

## 3. CorutineExceptionHandler: 예외 처리를 위한 마지막 수단

- 자식 코루틴은 처리되지 않은 예외를 부모 코루틴에 전파한다.
- 예외가 슈퍼바이저에 도달하거나 계층의 최상위로 가서 부모가 없는 루트 코루틴에 도달하면 예외는 더 이상 전파되지 않는다.
- 이 시점에서 처리되지 않은 예외는 `CoroutineExceptionHandler`라는 특별한 핸들러에 전달되며 이 핸들러는 코루틴 콘텍스트의 일부이다.
- 코루틴 콘텍스트에 예외 핸들러가 없다면 처리되지 않은 예외는 시스템 전역 예외 핸들러로 인동한다.
- CoroutineExceptionHandler를 코루틴 콘텍스트에 제공하면 처리되지 않은 예외를 처리하는 동작을 커스텀화 할 수 있다.
- CoroutineExceptionHandler를 코루틴 콘텍스트의 원소로 추가할 수 있다.

```kotlin
class ComponentWithScope(dispatcher: CoroutineDispatcher = Dispatchers.Default) {
    private val exceptionHandler = CoroutineExceptionHandler { _, e -> 
        println("[ERROR] ${e.message}")
    }
    private val scope = CoroutineScope(
        SupervisorJob() + dispatcher + exceptionHandler
    )
    fun action() = scope.launch {
        thorw UnsupprotedOperationException("Ouch!")
    }
}

fun main() = runBlocking {
    val supervisor = COmponentWithScope()
    supervisor.action()
    delay(1.seconds)
}
// [ERROR] Ouch!
```

출력 결과는 예외가 커스텀 예외 핸들러에 처리된다.

예외는 무모에게 위임하며 계층의 최상위에 이를 때까지 이런 위임이 계속된다. 따라서 `중간에 있는` CroutineExceptionHandler라는 것은 존재 하지 않는다.

`GlobalScope.launch API` 를 사용해 루트 코루틴을 생성하고 커스텀 코루틴 예외 핸들러를 그 콘텍스트의 일부로 제공한다. 또한 중간 예외 핸들러를 launch 코루틴에게 제공했을때 계층의 최상위에 있는 코루틴 예외 핸들러만 실행되며 중간에 있는 헨들러는 사용되지 않는다.

```kotlin
private val topLevelHandler = CoroutineExceptionHandler { _, e ->
    println("[TOP] ${e.message}")
}

private val intermediateHandler = CoroutineExceptionHandler { _, e ->
    println("[INTERMEDIATE] ${e.messsage}")
}

@OptIn(DelicateCorutinesApi::class)
fun main() {
    GlobalScope.launch(topLevelHandler) {
        launch(intermediateHandler) {
            throw UnsupprotedOperationException("Ouch!")
        }
    }
    Thread.sleep(1000)
}

// [TOP] Ouch!
```

### CoroutineExceptionHandler를 launch와 async에 적용할 때의 차이점

`CoroutineExceptionhandler` 는 최상위 코루틴이 `launch`로 생성된 경우에만 호출된다. 최상위 코루틴이 `async` 로 생성된 경우에는 `CoroutineExceptionhandler`가 호출되지 않는다.

```kotlin
class ComponentWithScope(dispatcher: CoroutineDispatcher = Dispatchers.Default) {
    private val exceptionHandler = CoroutineExceptionHandler { _, e ->
        println("[ERROR] ${e.message}")
    }
    private val scope = CoroutineScope(SupervisorJob() + dispatcher + excpetionHandler)
    
    fun action() = scope.launch {
        async {
            throw UnsupprotedOperationException("Ouch!")
        }
    }
}

fun main() = runBlocking {
    val supervisor = ComponentWithScope()
    supervisor.action()
    delay(1.secondes)
}
// [ERROR] Ouch!
```

만약 바깥의 코루틴을 `async`로 구현된다면 예외 핸들러가 호출되지 않는다.

```kotlin
class ComponentWithScope(dispatcher: CoroutineDispatcher = Dispatchers.Default) {
    private val exceptionHandler = CoroutineExceptionHandler { _, e ->
        println("[ERROR] ${e.message}")
    }
    private val scope = CoroutineScope(SupervisorJob() + dispatcher + excpetionHandler)
    
    fun action() = scope.async {
        launch {
            throw UnsupprotedOperationException("Ouch!")
        }
    }
}

fun main() = runBlocking {
    val supervisor = ComponentWithScope()
    supervisor.action()
    delay(1.secondes)
}

// 아무것도 출력되지 않음
```

최상위 코루틴이 `async` 로 시작되면 이 예외를 처리하는 책임은 `await`를 호출하는 `Deferred`의 소비자에게 있다. 따라서 코루틴 예외 핸들러는 이 예외를 무시할 수 있으며 `try-catch` 블록으로 감싸는 방식으로 예외를 처리할 수 있다.

## 4. 플로우에서 예외 처리

일반적으로 플로우의 일부분에서 예외가 발생하면 `collect`에서 예외가 던져진다. 이는 `collect` 호출을 `try-catch` 블록으로 감싸면 예상대로 동작한다. 이때 플로우에 중간 연산자가 적용됐는지 여부와는 관계가 없다.

```kotlin
fun main() = runBlocking {
    val transformedFlow = exceptionalFlow.map {
        it * 2
    }
    try {
        transformedFlow.collect {
            print("$it ")
        }
    } catch(u: UnhappyFlowException) {
        println("\nHandled: $u")
    }
    // 0 2 4 6 8
    // Handled: UnhappyFlowException
}
```

하지만, 더 복잡하고 긴 플로우 파이프라인을 구축할 때는 `catch` 연산자를 사용하는 쪽이 편리하다.

### catch 연산자로 업스트림 예외 처리

`catch`는 플로우에서 발생한 예외를 처리할 수 있는 중간 연산자다.

- 이 함수에 연결된 람다 안에서 플로우에 발생한 예외에 접근할 수 있다. 이때 예외는 람다의 파라미터로 전달된다.
- 이 연산자는 취소 예외를 자동으로 인식하기 때문에 취소가 발생한 경우에는 catch 블록이 호출되지 않는다.
- catch는 스스로 값을 방출할 수도 있다.

```kotlin
fun main() = funBlocking {
    exceptionalFlow
        .catch { cause ->
            println("\nHandled: $cause")
            emit(-1)
        }
        .collect {
            print("$it ")
        }
}
// 0 1 2 3 4
// Handled: UnhappyFlowException
// -1
```

catch 연산자가 오직 업스트림에 대해서만 작동하며, 플로우 처리 파이프라인의 앞쪽에서 발생한 예외들만 잡아낸다. catch 호출 다음에 위치한 onEach 람다에서 발생한 예외는 잡히지 않는다.

```kotlin
fun main() = funBlocking {
    exceptionalFlow
        .map {
            it + 1
        }
        .catch { cause ->
            println("\nHandled: $cause")
            emit(-1)
        }
        .onEach {
            throw UnhappyFlowException()
        }
        .collect()
}
// Exception in thread "main" UnhappyFlowException
```

### 술어가 참일 때 플로우의 수집 재시도: retry 연산자

`retry` 연산자는 작업을 재시도 할 수 있게 해준다. catch와 마찬가지로 retry는 업스트림의 예외를 잡는다. 재시도할 때는 업스트림 연산자가 모두 다시 실행된다.

이 경우 작업이 멱등성을 갖거나 반복 실행이 다른 방식으로 올바르게 처리되는지 확인해야 한다.

```kotlin
class CommunicationException : Exception("Communication failed!")

val unreliableFLow = flow {
    println("Starting the flow!")
    repeat(10) { number ->
        if (Random.nextDouble() < 0.1) throw CommunicationException()
        emit(number)
    }
}

fun main() = runBlocking {
    unreliableFLow
        .retry(5) { cause ->
            println("\nHandled: $cause")
            cause is CommunicationException
        }
        .collect { number ->
            print("$number")
        }
}
/*
Starting the flow!
0 1 2 3 4
Handled: CommunicationException: Communication failed!
Starting the flow!
0 1 2 3
Handled: CommunicationException: Communication failed!
Starting the flow!
0 1 2 3 4 5 6 7 8 9
*/
```

## 5. 코루틴과 플로우 테스트

코틀린 코루틴을 사용하는 코드를 위한 테스트도 일반적인 테스트와 마찬가지로 동작한다. 테스트 메서드에서 코루틴을 사용하려면 `runTest`  코루틴 빌더를 사용하면 된다.

### runBlocking 빌더 함수 테스트 단점

- 코루틴, 플로우를 사용하는 코드를 테스트할 때도 쓸 수 있다.
- 하지만 runBlocking을 사용하면 테스트가 실시간으로 실행되어 코드에 delay가 지정된 경우에 시간 지연이 전부 실행된다.
- 테스트 스위트가 커지면 불필요한 긴 실행 시간이 누적되면서 전체 애플리케이션 테스트 속도가 느련진다.

### 코루틴을 사용하는 테스트를 빠르게 만들기: 가상 시간과 테스트 디스패처

코틀린 코루틴 가상 시간을 사용해 테스트 실행을 빠르게 진행할 수 있게 해줘 가상 시간을 사용할 때는 지연을 자동으로 빠르게 진행될 수 있다.

```kotlin
class PlaygroundTest {
    @Test
    fun testDelay() = runTest {
        val startTime = System.currentTimeMillis()
        delay(20.seconds)
        println(System.currentTimeMillis() - startTime)
        // 11
    }
}
```

runTest

- 속도를 높이기 위해 특별한 테스트 디스패처와 스케줄러를 사용
- 실제로 코루틴의 지연 시간을 기다리지 않고 빠르게 진행
- runBlocking과 마찬가지로 runTest의 디스패처는 단일 스레드다.
- 기본적으로 모든 자식 코루틴은 동시에 실행되며 테스트 코드와 병렬로 실행되지 않는다.
- 단일 스레드 디스패처를 공유하는 경우 다른 코루틴이 코드를 실행하려면 코드가 일시 중단 지점을 제공해야 한다.
- runTest 본문에 일시 중단 지점이 없으면 테스트의 단언문은 실패한다.

```kotlin
@Test
fun testDelay() = runtest {
    var x = 0
    launch {
        x++
    }
    launch {
        x++
    }
    assertEquals(2, x)
}
```

테스트 디스패처에서는 `TestCoroutineScheduler`를 통해 가상 시간을 더 세밀하게 제어할 수 있다. 이 스케줄러는 코루틴 콘텍스트의 일부다. runTest 빌더 함수의 블록 안에서는 `TestScope`라는 특수한 스코프에 접근 할 수 있으며, 이 스코프는 `TestCoroutineScheduler` 기능을 사용할 수 있게 해준다. 이 스케줄러의 핵심 함수는 다음과 같다.

- runCurrent: 현재 실행하게 예약된 모든 코루틴을 실행한다.
- advanceUntilIdle: 예약된 모든 코루틴을 실행한다.

가상 디스패처의 현재 시간이 궁금하면 `currentTIme` 속성을 사용할 수 있다.

```kotlin
@OptIn(ExperimentalCoroutinesApi::class)
@Test
fun testDelay() = runTest {
    var x = 0
    launch {
        delay(500.milliseconds)
        x++
    }
    launch {
        delay(1.second)
        x++
    }
    println(currentTime) // 0
    
    delay(600.milliseconds)
    assertEquals(1, x)
    println(currentTime) // 600
    
    delay(500.milliseconds)
    assertEquals(2, x)
    println(currentTime) // 1100
}
```

미래 시점에 실행하도록 예약된 코루틴까지 실행하려면 `advanceUntilIdle` 함수를 사용할 수 있다.

```kotlin
@OptIn(ExperimentalCoroutinesApi::class)
@Test
fun testDelay() = runTest {
    var x = 0
    launch {
        delay(500.milliseconds)
        x++
    }
    launch {
        delay(1.second)
        x++
    }

    runCurrent()
    assertEquals(2, x)
    advanceUntilIdel()
    delay(600.milliseconds)
    assertEquals(3, x)
}
```

### 터빈으로 플로우 테스트

플로우를 사용하는 코드를 테스트하는 것도 runTest를 사용하는 다른 일시 중단 코드 테스트와 본질적으로 다르지 않다.

예를 들어 toList를 호출해서 유한한 플로우의 모든 원소를 먼저 컬랙션에 수집한 다음, 기대한 모든 원소가 실제로 결과 컬렉션에 있는지 확인 할 수 있다.

```kotlin
val myFlow = flow {
    emit(1)
    emit(2)
    emit(3)
}

@Test
fun doTest() = runTest {
    val results = myFLow.toList()
    assertEquals(3. results.size)
}
```

`터빈(Turbine)` 라이브러리를 사용하면 더 복잡하며 무한한 플로우나 더 까다로운 불변성을  다루는 테스트 작성에 도움을 준다.

- 터빈의 핵심 기능은 플로우의 확장 함수인 `test 함수`다.
- test 함수는 새 코루틴을 실행하며 내부적으로 플로우를 수집한다.
- test의 람다에서 `awaitItem`, `awaitComplete`, `awaitError` 함수를 테스트 프레임워크의 일반 단언문과 함께 사용해서 플로우에 대한 불변 조건을 지정하고 검증할 수 있다.
- 플로우가 방출한 모든 원소가 테스트에 의해 적절히 소비되도록 보장한다.

```kotlin
@Test
fun doTest() = runTest {
    val results = myFLow.test {
        assertEquals(1, awaitItem())
        assertEquals(2, awaitItem())
        assertEquals(3, awaitItem())
        awaitComplete()
    }
}
```

## 요약

- 한 코루틴에만 국한된 예외는 코루틴이 아닌 일반적인 코드와 마찬가지로 처리할 수 있다. 코루틴 경계를 넘는 예외는 좀 더 주의를 기울여야 한다.
- 기본적으로 코루틴에서 처리되지 않은 예외가 발생하면 부모 코루틴과 모든 형제 코루틴이 취소된다. 이를 통해 구조적 동시성 개념이 강제로 적용된다.
- supervisorScope나 SupervisorJob을 사용하는 다른 코루틴 스코프에서 사용되는 슈퍼바이저는 자식 코루틴 중 하나가 실패해도 다른 자식 코루틴을 취소하지 않는다. 또한 처리하지 않은 예외를 코루틴 계층의 위로 전파하지도 않는다.
- await는 async 코루틴에서 발생한 예외를 다시 던진다.
- 슈퍼바이저는 애플리케이션에서 오랫동안 실행되는 부분에 자주 사용된다. 종종 케이토의 Application처럼 프레임워크에 내장된 부품으로 제공되는 경우도 있다.
- 처리되지 않은 예외는 슈퍼바이저를 만나거나 코루틴 계츠으이 최상단에 도달할 때까지 전파된다. 이 시점에서 처리되지 않은 예외는 코루틴 콘테스트의 일부인 CoroutineExceptionHandler에게 전달된다. 콘텍스트에 코루틴 예외 핸들러가 없으면 시스템의 전역 예외 핸들러에 전달된다.
- JVM과 안드로이드에서 기본 시스템 예외 핸들러가 다르다. JVM에서는 스택 트레이스를 오류 콘솔에 기록하고, 안드로이드에서는 오류를 발생시키면서 애플리케이션을 중단시킨다.
- CoroutineExceptionHandler는 예외를 처리하는 마지막 수단으로 예외를 잡을 수는 없지만, 예외가 기록되는 방식을 사용자 정의할 수 있다. CoroutineExceptionHandler는 계츠으이 최상단에 있는 루트 코루틴의 콘텍스트에 위치한다.
- 최상단 코루틴을 launch 빌더로 시작한 경우에만 CoroutinExceptionHanler가 호출된다. async 빌더로 시작한 경우에는 이 핸들러가 호출되지 않으며, Deferred를 기다리는 코드가 예외를 처리해야 한다.
- 플로우에서 오류 처리는 collect를 try-catch 문으로 감싸거나 전용 catch 연산자를 사용한다.
- catch 연산자는 업스트림에서 발생한 예외만 처리하며, 다운스트림의 예외는 무시한다. 심지어 예외를 다시 던져 다운스트림에서 처리하게 하기 위해 catch를 사용할 수도 있다.
- retry를 사용해 예외가 발생햇을 때 플로우 수집을 처음부터 다시 시작할 수 있다. 이를 통해 코드가 오류를 복구할 수 있는 기회를 가질 수 있다.
- runTest의 가상 시간을 활용하면 코루틴 코드 테스트 속도를 높일 수 잇다. 모든 지연 시간이 자동으로 빠르게 진행된다.
- TestCoroutineScheduler는 runTest가 노출시키는 TestScope의 일부로, 현재 가상 시간을 추적하며 runCurrent와 advanceUntilIdle 같은 함수로 테스트 실행을 세밀하게 제어할 수 있다.
- 테스트 디스패처는 단일 스레드로 작동한다. 이에 따라 테스트 단언무을 호출하기 전에 새로 시작한 코루틴들이 실행될 수 있는 시간을 수동으로 보장해줘야한다.
- 터빈 라이브러리는 플로우 기반의 코드를 간편하게 테스트하게 해준다. 이 라이브러리의 핵심 API는 test 확장 함수로, 플로우에서 원소를 수집하고 awaitItem과 같은 함수를 사용해 테스트 중인 플로우의 원소 배출을 확인할 수 있다.