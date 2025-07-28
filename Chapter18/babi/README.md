## 오류 처리와 테스트

- 코루틴을 활용한 동시성 코드 작성을 이것저것 살펴봤다.
- 어플리케이션은 우리가 제어할수 없는 외부 시스템과 상호작용할 수 있으며, 외부 요소에 영향받지않고 적절한 오류처리를 할수 있어야한다.

---

### 코루틴 내부에서의 오류 처리
- 코루틴 빌더 안에 작성한 코드가 예외가 발생할 수 있다.
- 하지만 `launch`, `async` 는 새로운 코루틴을 생성하는 코루틴 빌더 함수이기 때문에, `try-catch` 로 통째로 감싸더라도 예외 처리할 수 없다.
  - 새로운 스레드를 생성하여 예외가 발생했는데, 메인스레드에서 잡히지 않는것과 동일하다

```kotlin
fun main(): Unit = runBlocking {
    try {
        launch {
            throw UnsupportedOperationException("Ouch!")
        }
    } catch (e: UnsupportedOperationException) {    // launch 를 통째로 try-catch 로 감싸더라도 예외가 처리되지 않음
        println("Handled $e")
    }
}
```


---

- `launch`에 전달되는 람다 블록 내부에서 `try-catch` 블록을 넣는다면, 동일한 코루틴 안에서 예외를 처리할 수 있다

```kotlin
fun main(): Unit = runBlocking {
    launch {
        try {   // 내부에서 try-catch 
            throw UnsupportedOperationException("Ouch!")
        } catch (e: UnsupportedOperationException) {
            println("Handled $e")
        }
    }
}
```

---

- `async` 으로 생성된 코루틴이 예외를 던지면, 결과값을 `await` 해올 때 예외가 발생한다.
- `await` 가 원하는 의미있는 값을 돌려줄 수 없기 때문에, `await` 를  `try-catch` 로 감싸서 예외를 처리할 수 있다. 

```kotlin
fun main(): Unit = runBlocking {
    val myDeferredInt: Deferred<Int> = async {
        throw UnsupportedOperationException("Ouch!")
    }
    try {
        val i: Int = myDeferredInt.await()  // 결과값 호출시 에러 발생
        println(i)
    } catch (u: UnsupportedOperationException) {
        println("Handled: $u")
    }
}
```

- 위 예외에서 `async` 에서 예외를 부모 코루틴인 `runBlocking` 에 전파하고 프로그램은 종료된다.
- 자식 코루틴에서 발생한 예외는 항상 부모 코루틴에 전파하게되는데, 해당 예외를 처리하는 다양한 방법들이 있다.
---

### 코틀린-코루틴 오류 전파
- 자식 코루틴에서 발생한 예외를 처리하는 방식을 어떻게 처리하느냐에 따라 다음 두가지로 나눌 수 있다.
  - 모든 자식 코루틴이 성공해야만 한다 -> 부모 코루틴에도 예외로 완료, 다른 자식 코루틴 실행 취소 
  - 자식 코루틴이 실패하더라도 서비스에 영향이 없어야 한다 -> 부모, 다른 자식 코루틴에 영향이 없음

---

### 자식이 실패하면, 모든 자식을 취소하는 코루틴

- 부모-자식간 계층이 `Job` 객체로 구축되었을때, 자식 코루틴에서 처리하지 않은 예외는 부모 코루틴을 예외로 완료하는 방식으로 처리된다
  - 다른 모든 자식 취소
  - 예외 발생하며 자신의 실행 완료
  - 상위 계층으로 예외 전파
- 해당 코루틴의 예외 처리 방식은, 같은 스코프 안에 동시성 계산을 함께 수행하고 공통의 결과를 반환하는 코루틴 그룹에서 아주 유용하다.
- 다음 예제를 보면, 형제 코루틴에서 예외를 던지면 무한루프인 하트비트 코루틴도 취소되는걸 볼 수 있다.

```kotlin
fun main(): Unit = runBlocking {
    launch {
        try {
            while (true) {
                println("Heartbeat!")
                delay(500)
            }
        } catch (e: Exception) {
            println("Heartbeat terminated: $e")
            throw e
        }
    }

    launch {        // 예외 던지는 형제 코루틴
        delay(1000)
        throw UnsupportedOperationException("Ow!")
    }
}

output> Heartbeat!
Heartbeat!
Heartbeat terminated ~~~
~~~: Ow!        // 이놈은 어디서 찍히게 되는건지 정확히 모르겠슴
```

---

### 구조적 동시성은 코루틴 스코프를 넘는 예외에만 영향을 미친다. 

- 형재 코루틴을 취소하고, 예외를 코루틴 계층 상위로 전파하는 동작은 코루틴 스코프를 넘는 처리되지 않은 예외에만 영향에만 미친다.
- 그러니깐 해당 코루틴 스코프에서 처리만 해준다면, 해당 에러가 전파가 안된다.
```kotlin
fun main(): Unit = runBlocking {
    launch {
        try {
            while (true) {
                println("Heartbeat!")
                delay(500)
            }
        } catch (e: Exception) {
            println("Heartbeat terminated: $e")
        }
    }

    launch {
        try {
            delay(1000)
            throw UnsupportedOperationException("Ow!")
        } catch (e: UnsupportedOperationException) {    // 코루틴 스코프 내에서 예외 처리를 했음
            println("Caught: $e")
        }
    }
}

ouptut> Heartbeat!
Heartbeat!
Caught: ~~~
Heartbeat!
...
```

### 슈퍼바이저는 부모와 형제가 취소되지 않게 한다
- `SupervisorJob` 는 자식이 실패하더라도 상관없다. 슈퍼바이저라고 부른다.
- 슈퍼바이저는 자식이 실패하더라도 다른 상위로 전파하지 않고, 다른 자식 작업이 실패해도 취소하지 않게 한다.

```kotlin
fun main(): Unit = runBlocking {
    supervisorScope {       // launch 호출을 supervisorScope 으로 감싼다
        launch {
            try {
                while (true) {
                    println("Heartbeat!")
                    delay(500)
                }
            } catch (e: Exception) {
                println("Heartbeat terminated: $e")
                throw e
            }
        }

        launch {
            delay(1000)
            throw UnsupportedOperationException("Ow!")
        }
    }
}

output> Heartbeat!
Heartbeat!
Heartbeat!
~~~: Ow!
Heartbeat!
Heartbeat!
...
```

- `SupervisorJob` 이 `launch` 빌더로 시작된 자식 코루틴에 `CoroutineExceptionHandler` 를 호출하기 때문에, 형제, 부모 코루틴이 멈추지 않게 된다.
- 슈퍼바이저는 코루틴 계층의 위쪽에 위치하는 경우가 많다. 어플리케이션 전체 수명, 창, 뷰 등 오랫동안 실행되어야하는부분은 코루틴에 연관시키지 않기 위함이다.
- 미세한 작업 함수들은 슈퍼바이저를 사용하지 않는데, 불필요한 작업이 취소되는게 바람직한 특성이기 때문

---


### CoroutineExceptionHandler: 예외 처리를 위한 마지막 수단
- 자식 코루틴은 처리되지 않은 예외를 부모 코루틴에 전파하는데, 슈퍼바이저까지 도달하거나, 루트 코루틴까지 도달해야 예외전파가 끝나게 된다.
- 만일, 그때까지 처리되지 않은 예외는 `CoroutineExceptionHandler` 라는 핸들러에 전달된다.
- 해당 예외를 처리하는 동작을 커스텀화 할수 있다. ktor 는 해당 핸들러를 사용해 처리되지 않은 예외의 문자열 표현을 로그 프로바이더에 보내지만, 안드로이드 ViewModel 은 해당 핸들러를 지정하지 않기 때문에, 코루틴에서 `launch` 에 처리되지 않은 예외가 발생하면 앱이 종료된다.


```kotlin
class ComponentWithScope(dispatcher: CoroutineDispatcher = Dispatchers.Default) {
    private val exceptionHandler = CoroutineExceptionHandler { _, e ->  
        println("[ERROR] ${e.message}")
    }

    private val scope = CoroutineScope(
        SupervisorJob() + dispatcher + exceptionHandler
    )

    fun action() = scope.launch {
        throw UnsupportedOperationException("Ouch!")
    }
}

fun main() = runBlocking{
    val supervisor = ComponentWithScope()
    supervisor.action()
    delay(1.seconds)
}

output> [ERROR] Ouch!   // 커스텀 예외 핸들러로 [ERROR] 접두사가 붙어서 나왔다.
```

- 만약 핸들러가 각 코루틴마다 존재한다고 하더라도, 결국 예외는 부모 코루틴에게 전파되기 때문에, 중간에 있는 핸들러는 사용되지 않는다.

```kotlin
private val topLevelHandler = CoroutineExceptionHandler { _, e ->
    println("[TOP] ${e.message}")
}

private val intermediateHandler = CoroutineExceptionHandler { _, e ->
    println("[INTERMEDIATE] ${e.messsage}") // 호출되지 않음
}

@OptIn(DelicateCorutinesApi::class)
fun main() {
    GlobalScope.launch(topLevelHandler) {
        launch(intermediateHandler) {   // 중간 launch
            throw UnsupprotedOperationException("Ouch!")
        }
    }
    Thread.sleep(1000)
}

output> [TOP] Ouch!
```

---

### CoroutineExceptionHandler 의 launch, async 차이점

- `CoroutineExceptionHandler` 예외 핸들러는 최상위 코루틴이 `launch` 으로 생성된 경우에만 호출이 된다. 
  - `async` 으로 생성된 경우는 `CoroutineExceptionHandler` 가 호출되지 않는다.

```kotlin
class ComponentWithScope(dispatcher: CoroutineDispatcher = Dispatchers.Default) {
    private val exceptionHandler = CoroutineExceptionHandler { _, e ->
        println("[ERROR] ${e.message}")
    }
    private val scope = CoroutineScope(SupervisorJob() + dispatcher + excpetionHandler)

    fun action() = scope.launch {   // launch 로 시작
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

output> [ERROR] Ouch!
```

- 하지만, 최상위 코루틴을 `async` 으로 시작하게되면 핸들러가 호출되지 않는다. 
```kotlin
class ComponentWithScope(dispatcher: CoroutineDispatcher = Dispatchers.Default) {
    private val exceptionHandler = CoroutineExceptionHandler { _, e ->
        println("[ERROR] ${e.message}")
    }
    private val scope = CoroutineScope(SupervisorJob() + dispatcher + excpetionHandler)

    fun action() = scope.async {   // async 로 시작
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

ouptut> (없음)
```

- 최상위 코루틴이 `async` 으로 시작되면 예외를 처리하는 책임은 `await()` 호출하는 `Deferred` 소비자에게 있다.
- `async` 쪽이 아닌, `await` 호출을 `try-catch` 으로 예외 처리해야한다.
---


### 플로우에서의 예외 처리
- 플로우에서도 예외를 던질 수 있다. 
- 다음 플로우 예제를 보면, 5개의 원소가 배출된 이후 에러가 발생한다.
```kotlin
class UnhappyFlowException: Exception()

val exceptionalFlow = flow { 
    repeat(5) { number -> 
        emit(number) 
    }
    throw UnhappyFlowException()
}
```
- 일반적으로 플로우의 일부분에서 예외가 발생하면 `collect` 에서 예외가 던져진다. `async-await` 에서 `await` 에 `try-catch` 를 하는 느낌으로 `collect` 에 `try-catch` 문을 걸어준다.

```kotlin
fun main() = runBlocking {
    val transformedFlow = exceptionalFlow.map { it * 2 }
    try {
        transformedFlow.collect {   // collect 를 try-catch 에 넣었다.
            print("$it ")
        }
    } catch (e: UnhappyFlowException) {
        println("\nHandled: $e")
    }  
}

output> 0 2 4 6 8
Handled: UnhappyFlowException
```

---

### catch 연산자로 업스트림 예외 처리
- catch 는 플로우엥서 발생한 예외를 처리하는 중간 연산자다.
- 취소 예외를 자동으로 인식하기 때문에, 취소가 발생하면 `catch` 블록이 호출되지 않는다.
- 스스로 값을 방출할 수도 있기 때문에 예외를 오류 값으로 변환해 다운스트림 플로우에서 소비할 수 있다.

```kotlin
fun main() = runBlocking { 
    exceptionalFlow
        .catch { cause -> 
            println("Handled: $cause")
            emit(-1) 
        }
        .collect {
            print("$it ") 
        }
}

output> 0 1 2 3 4 
Handled: UnhappyFlowException
-1
```

- `catch` 연산자는 오직 업스트림에 대해서만 작동하게 된다.
- `catch` 호출 다음에 위치한 `onEach` 람다에서 발생한 예외는 잡지 않는다.
```kotlin
fun main() = runBlocking {
    exceptionalFlow
        .map { it + 1 }
        .catch { cause -> 
            println("Handled: $cause") 
        }
        .onEach { 
            throw UnhappyFlowException() 
        }
        .collect()
}

output> 0 1 2 3 4
Handled: UnhappyFlowException
```


---
### 술어가 참일 때 플로우 수집 재시도: retry

- 플로우를 처리하다 예외가 발생시, 오류 메세지와 함께 종료 작업을 재시도하고 싶을 수 있는데, 이때 내장된 `retry` 연산자로 관리할 수 있다.
- 재시도 동안 업스트림의 플로우가 처음부터 다시 수집되어 모든 중간 연산이 실행된다.

```kotlin
val unreliableFlow = flow {
    println("starting the flow!")
    repeat(10) { number ->
        if (Random.nextDouble() < 0.1) throw CommunicationException()
        emit(number)
    }
}

fun main() = runBlocking {
    unreliableFlow
        .retry(5) { cause ->    // 5회 재시도
            println("Handled: $cause")
            cause is CommunicationException // true 라면 재시도
        }
        .collect { number -> 
            print("$number ") 
        }
}
```

---

### 코루틴과 플로우 테스트
- 코루틴을 테스트하려면 `runTest` 코루틴 빌더를 사용한다.
- `runBlocking` 코루틴 빌더를 사용하면, 테스트가 실시간으로 실행된다. 만약 코드에 `delay` 가 있다면 결과 계산 전  시간 지연이 전부 실행된다는 뜻이다.

---

### 코루틴을 사용하는 테스트 빠르게 만들기: 가상 시간과 테스트 디스패처
- 코틀린 코루틴은 가상 시간을 사용해 테스트 실행을 빠르게 진행할 수 있도록 해준다. 
- 가상 시간을 사용하면 지연이 자동으로 빠르게 진행되기 때문에, `delay` 등이 갖는 문제를 해결할 수 있다.
- 예로 다음 테스는 20초를 delay 선언이 되어있지만, 몇 밀리초 만에 완료되는걸 볼 수 있다.
```kotlin
@Test
fun testDelay() = runTest {
    val startTime = System.currentTimeMillis()
    delay(20.seconds)
    println(System.currentTimeMillis() - startTime)
}
```

- `runBlocking` 과 마찬가지로 `runTest` 는 단일 스레드다.
- 모든 자식 코루틴은 동시에 실행되고, 테스트 코드와 병렬로 실행되지 않는다.
- 단일 스레드 디스패처를 공유하면, 다른 코루틴이 코드를 실행하려면 일시 중단 지점을 제공해야 한다. `runTest` 본문에 일시 중단 지점이 없기 때문에 다음 테스트는 실패한다.
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
---
- 테스트 디스패처에서는 `TestCorutineScheduler` 을 사용해서 가상 시간을 세밀하게 제어할 수 있으며, 이 스케줄러는 코루틴 콘텍스트의 일부다.
- `runTest` 함수 블록 안에서 `TestScope` 라는 특수한 스코프에 접근 할 수 있고, 이 스코프는 `TestCorutineScheduler` 기능을 사용할 수 있게 한다.
  - `runCurrent`: (지금까지) 현재 실행하게 예약된 모든 코루틴을 실행
  - `advanceUntilIdle`: (미래 어느 시점에)예약된 모든 코루틴을 실행

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

    runCurrent()        // 현재 시점까지만 실행
    assertEquals(2, x)
    advanceUntilIdel()  // 미래 시점 예약까지 실행
    delay(600.milliseconds)
    assertEquals(3, x)
}
```

---

### 터빈으로 플로우 테스트
- 플로우를 테스트 하려고, 다음과 같이 `toList` 호출하여 모든 원소를 수집한 뒤, 실제 예상값과 동일한지 확인할 수 있다.
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
- 하지만 플로우 코드는 더 복잡하고, 무한하거나 까다롭기때문에 위와같은 테스트 방식은 쉽지않다..
- 터빈은 서드파티 라이브러리이지만, 플로우 API 테스트에 웰논하게 사용된다.
- 핵심 기능으로 `test` 라고 하는 플로우의 확장 함수를 제공한다. `test` 함수 람다에서 검증할 수 있는 몇몇 함수를 제공한다.
```kotlin
@Test
fun doTest() = runTest {
    val results = myFLow.test {     // test: 플로우 확장 함수
        assertEquals(1, awaitItem())
        assertEquals(2, awaitItem())
        assertEquals(3, awaitItem())
        awaitComplete()
    }
}
```