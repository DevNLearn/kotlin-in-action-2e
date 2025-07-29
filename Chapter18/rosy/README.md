# 18장 오류 처리와 테스트
- 지금까지는 플로우와 코루틴으로 **동시성 코드**를 효율적으로 다루는 방법을 배웠다면, 이제는 그 코드가 **문제가 생겼을 때** 어떻게 **안정적으로** 동작할 수 있는지 알아본다.

---

## 18.1 코루틴 내부에서 던져진 오류 처리

- launch 안에서 발생한 예외는 외부 try-catch에서 잡히지 않는다.

    ```kotlin
    import kotlinx.coroutines.*
    
    fun main(): Unit = runBlocking {
        try {
            launch {
                throw UnsupportedOperationException("Ouch!")
            }
        } catch (e: UnsupportedOperationException) {
            println("Handled $e")
        }
    }
    
    // Exception in thread "main" java.lang.UnsupportedOperationException: Ouch!
    // at MyExampleKt$main$1$1.invokeSuspend(MyExample.kt:6)
    // ..
    ```

  - 이 코드는 `catch`가 실행되지 않고 예외가 **전파됨**.
  - 이유: `launch {}`는 **새로운 코루틴을 생성**하기 때문에, **다른 스레드처럼 예외가 외부로 전파되지 않음.**
- launch 내부에 try-catch를 직접 써야 한다.

    ```kotlin
    import kotlinx.coroutines.*
    
    fun main(): Unit = runBlocking {
        launch {
            try {
                throw UnsupportedOperationException("Ouch!")
            } catch (e: UnsupportedOperationException) {
                println("Handled $e")
            }
        }
    }
    // Handled java.lang.UnsupportedOperationException: Ouch!
    ```

  - 이렇게 하면 예외는 **코루틴 내부에서 직접 처리** 가능.
- async에서 발생한 예외는 await 시점에 다시 발생

    ```kotlin
    import kotlinx.coroutines.*
    
    fun main(): Unit = runBlocking {
        val myDeferredInt: Deferred<Int> = async {
            throw UnsupportedOperationException("Ouch!")
        }
    
        try {
            val i: Int = myDeferredInt.await()
            println(i)
        } catch (u: UnsupportedOperationException) {
            println("Handled: $u")
        }
    }
    ```

  - `await()`는 실제 값을 얻는 시점이기 때문에 여기서 예외가 **다시 발생**함.
  - 콘솔에는 예외 메시지가 **두 번** 찍힘
    - `try-catch`에서 한 번
    - 코루틴의 예외 처리자에서 한 번 (예외가 부모 코루틴에 전파되기 때문)

---

## 18.2 코틀린 코루틴에서의 오류 전파

- 구조적 동시성과 오류 전파
  - **자식 코루틴의 실패가 부모에게 전파되는 방식**은 코루틴의 구조적 관계에 따라 달라진다.
  - 크게 2가지 상황
    - **공동 작업 수행 중 실패**: 하나의 자식 실패 → 부모 실패 → 다른 자식 취소
    - **감독자 역할 수행 중 실패**: 자식이 실패해도 부모와 다른 자식은 영향을 받지 않음

### 자식이 실패하면 모든 자식을 취소하는 코루틴

- 기본동작
  - `Job` 기반으로 생성된 코루틴은 자식 예외 발생 시:
    1. 부모에게 예외 전파
    2. 다른 자식 모두 **취소**
    3. 상위 부모로 **예외 전파**
- 예제 – 기본 오류 전파

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
    
        launch {
            delay(1000)
            throw UnsupportedOperationException("Ow!")
        }
    }
    ```

  - 출력 결과

      ```kotlin
      Heartbeat!
      Heartbeat!
      Heartbeat terminated: kotlinx.coroutines.JobCancellationException: ...
      Exception in thread "main" java.lang.UnsupportedOperationException: Ow!
      ```

  - 두 번째 launch의 예외 발생 → 첫 번째 코루틴까지 취소됨

### 구조적 동시성은 코루 스코프를 넘는 예외에만 영향을 미친다

- **try-catch로 내부에서 직접 예외를 잡으면**, 구조적 동시성 전파를 막을 수 있음

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
            } catch (e: UnsupportedOperationException) {
                println("Caught: $e")
            }
        }
    }
    ```

  - 출력 결과

      ```kotlin
      Heartbeat!
      Heartbeat!
      Caught: java.lang.UnsupportedOperationException: Ow!
      Heartbeat!
      Heartbeat!
      ...
      ```

  - 내부에서 예외를 catch 하면 **형제 코루틴은 계속 작동**함

### 슈퍼바이저는 부모와 형제가 취소되지 않게 한다

- `SupervisorJob`을 사용하면 자식 예외가 부모와 형제에게 영향을 미치지 않음
- 대표적인 사용 예: UI, 애플리케이션 루트 코루틴 스코프
- 예제 – supervisorScope로 예외 격리

    ```kotlin
    fun main(): Unit = runBlocking {
        supervisorScope {
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
    
    ```

  - 출력 결과

      ```kotlin
      Heartbeat!
      Heartbeat!
      Exception in thread "main" java.lang.UnsupportedOperationException: Ow!
      ...
      Heartbeat!
      Heartbeat!
      ...
      ```

  - 예외 발생해도 **Heartbeat는 계속 실행됨**
  - `SupervisorJob`은 예외 전파를 막아 안정성을 확보함
- 실제 활용 예: Ktor
  - `Application` 스코프: Supervisor 역할 → 앱 전체 중단 없음
  - `PipelineContext`: 구조적 동시성 → 한 요청에서 발생한 예외가 전체 취소 유발

---

## 18.3 CoroutineExceptionHandler: 예외 처리의 마지막 수단

- 처리되지 않은 예외의 흐름
  - 자식 코루틴에서 **처리되지 않은 예외**는 부모 코루틴으로 전파된다.
  - 최상위 루트 코루틴에 도달하거나, `SupervisorJob`에 도달한 경우 더 이상 예외를 전파하지 않음.
  - 이 시점에 도달한 예외는 `CoroutineExceptionHandler`에 의해 처리된다.
- 시스템 전역 예외 핸들러
  - JVM: `UncaughtExceptionHandler`처럼 스택 트레이스를 콘솔에 출력.
  - Android: 앱이 **즉시 종료**됨.
- CoroutineExceptionHandler 정의 및 사용

    ```kotlin
    class ComponentWithScope(dispatcher: CoroutineDispatcher = Dispatchers.Default) {
    	private val exceptionHandler = CoroutineExceptionHandler { _, e ->
        println("[ERROR] ${e.message}")
    	}
    	
    	private val scope = CoroutineScope(
    		SupervisorJob() + dispatcher + exceptionHandler
    	)
    	
    	fun action() = scope.launch {
        throw UnsupportedOperationException("Ouch!") // 예외 발생
    	}
    }
    
    fun main() = runBlocking{
    	val supervisor = ComponentWithScope()
    	supervisor.action()
    	delay(1.seconds)
    }
    ```

- 출력 결과

    ```kotlin
    [ERROR] Ouch!
    ```

- 핵심 개념
  - `CoroutineExceptionHandler`는 **루트 코루틴**(`launch`)에만 적용된다.
  - **중간 launch**에 설정된 핸들러는 무시된다.
  - 예외는 **계층을 따라 위로만 전파**되며, 중간 계층에서는 인터셉트되지 않는다.

### CoroutineExceptionHandler를 launch와 async에 적용할 때의 차이점

- 핵심 차이점
  - `launch`로 시작된 루트 코루틴 → `CoroutineExceptionHandler` 적용 ✅
  - `async`로 시작된 루트 코루틴 → `CoroutineExceptionHandler` 적용 ❌ (예외를 처리하려면 반드시 `await()`에서 `try-catch` 해야 함)
- launch 예시: 핸들러가 예외를 처리함

    ```kotlin
    val scope = CoroutineScope(SupervisorJob() + Dispatchers.Default + exceptionHandler)
    
    scope.launch {
        async {
            throw UnsupportedOperationException("Ouch!") // 핸들러에서 처리됨
        }
    }
    ```

- async 예시: 예외가 무시됨

    ```kotlin
    val scope = CoroutineScope(SupervisorJob() + Dispatchers.Default + exceptionHandler)
    
    scope.async {
        launch {
            throw UnsupportedOperationException("Ouch!") // 아무것도 출력되지 않음
        }
    }
    ```

  - `async`는 예외를 `await()` 하는 쪽이 직접 처리해야 한다.
- 코루틴에서 예외를 안정적으로 처리하고 싶다면:
  - **최상위 루트 코루틴**은 `launch`로 시작하고,
  - `CoroutineExceptionHandler`를 설정하고,
  - `SupervisorJob`과 함께 사용하자.

---

## 18.4 플로우에서 예외 처리

- 플로우도 예외를 던질 수 있다
  - 일반 코틀린 함수나 일시 중단 함수처럼, 플로우도 예외를 던질 수 있다.
  - 예: 아래는 0~4까지 emit한 후 `UnhappyFlowException`을 던지는 예제

      ```kotlin
      val exceptionalFlow = flow {
          repeat(5) { number ->
              emit(number)
          }
          throw UnhappyFlowException()
      }
      ```

- try-catch로 예외 처리
  - 일반적인 함수와 마찬가지로 `collect` 블록을 `try-catch`로 감싸 예외 처리 가능

      ```kotlin
      val transformedFlow = exceptionalFlow.map { it * 2 }
      try {
          transformedFlow.collect { 
              print("$it ") 
          }
      } catch (e: UnhappyFlowException) {
          println("\nHandled: $e")
      }
      // 출력: 0 2 4 6 8
      // Handled: UnhappyFlowException
      ```


### `catch` 연산자: 업스트림 예외를 중간에서 처리

- `catch`는 **업스트림 예외만 처리**함 (자신보다 앞쪽 연산자의 예외만 잡음)
- `emit()`을 통해 기본 값 제공 가능
- `CancellationException`은 자동으로 무시됨 (catch 블록 호출 안 됨)

```kotlin
exceptionalFlow
    .catch { cause ->
        println("Handled: $cause")
        emit(-1)
    }
    .collect { print("$it ") }

// 출력: 0 1 2 3 4
// Handled: UnhappyFlowException
// -1
```

- 주의: `catch`는 업스트림 예외만 처리한다

    ```kotlin
    exceptionalFlow
        .map { it + 1 }
        .catch { cause -> println("Handled: $cause") }
        .onEach { throw UnhappyFlowException() }
        .collect()
    
    // 위 코드에서 예외는 catch에서 처리되지 않음
    ```

  - `catch`는 `onEach` 이후에 있어 예외를 잡지 못함
  - → 해결: 예외가 발생 가능한 지점 앞에 catch를 두거나, `collect`를 try-catch로 감싸야 함

### 술어가 참일 때 플로우의 수집 재시도: retry 연산자

- `catch`처럼 **업스트림 예외만 처리**
- 람다에서 `true`를 반환하면 재시도 수행
- `flow`의 처음부터 다시 수집됨 → 부수 효과 있는 작업은 주의

```kotlin
val unreliableFlow = flow {
    println("starting the flow!")
    repeat(10) { number ->
        if (Random.nextDouble() < 0.1) throw CommunicationException()
        emit(number)
    }
}

unreliableFlow
    .retry(5) { cause ->
        println("Handled: $cause")
        cause is CommunicationException
    }
    .collect { number -> print("$number ") }

// 실행 시: 재시도하며 0~9 모두 수집될 수 있음
```

- 주의
  - `retry`는 `flow`가 처음부터 다시 실행됨
  - → **부수 효과(idempotency)** 고려해야 함

---

## 18.5 코루틴과 플로우 테스트

- 테스트 기본 원칙
  - 코루틴이 포함된 코드도 결국 일반 테스트와 원칙은 같음
  - 단, 지연(delay)이 많은 코드는 `runBlocking`을 쓰면 **실제 시간만큼 기다려야 해서 느림**
  - 이를 해결하기 위해 **`runTest`와 가상 시간 가속** 사용

### 코루틴을 사용하는데 테스트를 빠르게 만들기: 가상 시간과 테스트 디스패처

- `runTest`란?
  - 코루틴 테스트 전용 빌더
  - 가상 시간(virtual time) 기반으로 delay 등을 빠르게 진행
  - 테스트 시간 단축, 더 자주 테스트 가능
- 예제: delay 포함 테스트

    ```kotlin
    @Test
    fun testDelay() = runTest {
        val startTime = System.currentTimeMillis()
        delay(20.seconds)
        println(System.currentTimeMillis() - startTime)
    }
    ```

  - 결과: `delay(20.seconds)`지만 실제 실행 시간은 거의 0초
- 가상 시간 제어 메서드

    | 메서드 | 설명 |
    | --- | --- |
    | `runCurrent()` | 현재 예약된 코루틴만 실행 |
    | `advanceUntilIdle()` | 모든 예약된 코루틴 실행 |
    | `currentTime` | 현재 가상 시간 확인 |
- 예제: `advanceUntilIdle`

    ```kotlin
    @Test
    fun testAdvanceUntilIdle() = runTest {
        var x = 0
        launch { 
    	    x++ 
    	    launch { 
    		    x++ 
    	    }
        }
        
        assertEquals(2, x)
    
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

- 주의: 일시 중단이 없으면 동시 실행 안됨

    ```kotlin
    @Test
    fun testFailing() = runTest {
        var x = 0
        launch { x++ }
        launch { x++ }
        assertEquals(2, x) // 실패 가능
    }
    ```

  - 해결: `delay`, `yield()` 등 일시 중단 지점 추가

### 터빈(Turbine)을 이용한 플로우 테스트

- 터빈이란?
  - JetBrains가 아닌 서드파티 라이브러리
  - 플로우 테스트를 간결하고 확실하게 만들어줌
  - https://github.com/cashapp/turbine
- 기능 요약

    | 기능 | 설명 |
    | --- | --- |
    | `awaitItem()` | 방출된 아이템 기다림 |
    | `awaitComplete()` | 완료 신호 기다림 |
    | `awaitError()` | 예외 기다림 |
- 예제: 기본 테스트

    ```kotlin
    val myFlow = flow {
        emit(1)
        emit(2)
        emit(3)
    }
    
    @Test
    fun doTest() = runTest {
        val results = myFlow.toList()
        assertEquals(3, results.size)
    }
    ```


---

## 요약

- 한 코루틴에만 국한된 예외는 코루틴이 아닌 일반적인 코드와 마찬가지로 처리할 수 있다. 코루틴 경계를 넘는 예외는 좀 더 주의를 기올려야 한다.
- 기본적으로 코루틴에서 처리되지 않은 예외가 발생하면 부모 코루틴과 모든 형제 코루틴이 취소된다. 이를 통해 구조적 동시성 개념이 강제로 적용된다.
- supervisorScope나 SupervisonJob을 사용하는 다른 코루틴 스코프에서 사용되는 슈퍼바이저는 자식 코루틴 중 하나가 실패해도 다른 자식 코루틴을 취소하지 않는다. 또한 처리하지 않은 예외를 코루틴 계층의 위로 전파하지도 않는다.
- await는 async 코루틴에서 발생한 예외를 다시 던진다.
- 슈퍼바이저는 애플리케이션에서 오랫동안 실행되는 부분에 자주 사용된다.
  종종 케이토의 Application처럼 프레임워크에 내장된 부품으로 제공되는 경우도 있다.
- 처리되지 않은 예외는 슈퍼바이저를 만나거나 코루틴 계층의 최상단에 도달할 때까지 전파된다. 이 시점에서 처리되지 않은 예외는 코루틴 콘텍스트의 일부인 CoroutineExceptionHandler에게 전달된다. 콘텍스트에 코루틴 예외 핸들러가 없으면 시스템의 전역 예외 핸들러에 전달된다.
- JVM과 안드로이드에서 기본 시스템 예외 핸들러가 다르다. JVM에서는 스택 트레이스를 오류 콘솔에 기록하고, 안드로이드에서는 오류를 발생시키면서 애플리케이션을 중단시킨다.
- CoroutineExceptionHander는 예외를 처리하는 마지막 수단으로 예의를 잡을 수는 없지만, 예외가 기록되는 방식을 사용자 정의할 수 있다. CoroutineExceptionHander는 계층의 최상단에 있는 루트 코루틴의 콘텍스트에 위치한다.
- 최상단 코루틴을 launch 빌더로 시작한 경우에만 CoroutineExceptionHander가 호출된다. async 빌더로 시작한 경우에는 이 핸들러가 호출되지 않으며, Deferred를 기다리는 코드가 예외를 처리해야 한다.
- 플로우에서 오류 처리는 Collect를 try-catch 문으로 감싸거나 전용 catch 연산자를 사용한다.
- catch 연산자는 업스트림에서 발생한 예외만 처리하며, 다운스트림의 예외는 무시한다. 심지어 예외를 다시 던져 다운스트림에서 처리하게 하기 위해 catch를 사용할 수도 있다.
- retry를 사용해 예외가 발생했을 때 플로우 수집을 처음부터 다시 시작할 수 있다. 이를 통해 코드가 오류를 복구할 수 있는 기회를 가질 수 있다.
- runTest의 가상 시간을 활용하면 코루틴 코드 테스트 속도를 높일 수 있다.
  모든 지연 시간이 자동으로 빠르게 진행된다.
- TestCoroutineScheduler는 runTest가 노출시키는 TestScope의 일부로, 현재 가상 시간을 추적하며 runCurrent와 advanceUntilIdle 같은 함수로 테스트 실행을 세밀하게 제어할 수 있다.
- 테스트 디스패처는 단일 스레드로 작동한다. 이에 따라 테스트 단언문을 호출하기 전에 새로 시작한 코루틴들이 실행될 수 있는 시간을 수동으로 (yield나 다른 일시 중단 지점을 호출하는 방식으로) 보장해줘야만 한다.
- 터빈 라이브러리는 플로우 기반의 코드를 간편하게 테스트하게 해준다. 이 라이브러리의 핵심 API는 test 확장 함수로, 플로우에서 원소를 수집하고 awaitItem과 같은 함수를 사용해 테스트 중인 플로우의 원소 배출을 확인할 수 있다.