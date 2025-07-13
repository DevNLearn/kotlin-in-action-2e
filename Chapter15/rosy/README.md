# 15장 구조화된 동시성

실제 애플리케이션에서는 여러 코루틴을 동시에 실행하게 되는데, 이들을 일일이 추적하고 종료하는 것은 쉽지 않습니다. 예를 들어 사용자가 화면을 이동했는데, 이전 화면에서 실행된 네트워크 요청 코루틴이 계속 실행된다면 리소스 낭비가 발생할 수 있습니다.

이를 해결하기 위해 코틀린은 **구조화된 동시성**을 제공합니다. 각 코루틴을 명확한 **스코프** 안에서 실행하고, 부모-자식 관계를 통해 생명주기를 자동으로 관리합니다. 덕분에 불필요한 코루틴을 방지하고, 안정적인 코루틴 관리가 가능해집니다.

이번 장에서는 구조화된 동시성의 원리와 이를 활용해 코루틴을 안전하게 관리하는 방법을 다룹니다.

---

## 15.1 코루틴 스코프가 코루틴 간의 구조를 확립한다.

### 구조화된 동시성이란?

- 동시성을 안전하고 예측 가능하게 다루기 위한 **설계 철학**
- **코루틴은 반드시 어떤 "스코프(scope)" 안에서 실행되어야 하며,
  그 스코프가 코루틴의 생명주기를 책임진다.**

### 코루틴 스코프이란?

- 코루틴을 실행할 수 있는 **"환경"이자 "컨텍스트"**
- `CoroutineScope`는 내부에 `Job`과 `CoroutineContext`를 가지고 있어

  해당 스코프에서 실행된 **모든 코루틴의 생명주기를 추적**하고 **관리**할 수 있다.

| 역할 | 설명 |
| --- | --- |
| 부모-자식 관계 생성 | 스코프 안에서 실행된 모든 코루틴은 **부모-자식** 구조를 가진다 |
| 일괄 취소 | 부모 스코프가 취소되면 모든 자식 코루틴도 **자동으로 취소**됨 |
| 예외 전파 | 자식에서 발생한 예외는 부모로 **전파**되어 처리 가능 |
| 종료 대기 | 부모 코루틴은 자식들이 **모두 종료될 때까지 기다린다** |

### 코루틴의 계층 구조

```kotlin
fun main() {
 runBlocking { // this: CoroutineScope
    launch {   // this: CoroutineScope
        delay(1.seconds)
        launch {
            delay(250.milliseconds)
            log("Grandchild done")
        }
        log("Child 1 done!")
    }

    launch {
        delay(500.milliseconds)
        log("Child 2 done!")
    }

    log("Parent done!")
	}
}
```

- 실행 결과 (예시)

    ```kotlin
    [main @coroutine#1] Parent done!
    [main @coroutine#3] Child 2 done!
    [main @coroutine#2] Child 1 done!
    [main @coroutine#4] Grandchild done
    ```

- `runBlocking`은 **부모 코루틴**
- 그 안에서 생성한 `launch`는 **자식 코루틴**
- 자식 코루틴 내부의 `launch`는 **손자(자식의 자식)**

→ 부모(runBlocking)는 **모든 자식 코루틴이 끝날 때까지** 기다린다

→ `await()`나 `join()`을 직접 호출하지 않아도 자동으로 기다려줌

→ 이게 바로 **구조화된 동시성** !

### 코루틴 스코프 생성: coroutineScope 함수

- 전형적인 사용 사례는 동시적 작업 분해(여러 코루틴을 활용해 계산을 수행)
- 새로운 코루틴을 만들지 않고도 코루틴 스코프를 생성할 수 있음
    - 이럴 때 `coroutineScope { ... }` 사용!

```kotlin
suspend fun computeSum() {
    log("Computing a sum...")
    val sum = coroutineScope {
        val a = async { generateValue() }
        val b = async { generateValue() }
        a.await() + b.await()
    }
    log("Sum is $sum")
}

suspend fun generateValue(): Int {
    delay(500)
    return Random.nextInt(0, 10)    
}

fun main() = runBlocking {
    computeSum()
}
```

- `coroutineScope { ... }` 안의 자식 코루틴들이 모두 끝날 때까지 **자동으로 기다림**
- `async`로 비동기 병렬 실행 → 결과를 `await()`로 받아 합산

### 코루틴 스코프를 컴포넌트와 연관시키기: CoroutineScope

- 동시 처리나 코루틴의 시작과 종료를 관리하는 클래스를 만들고 싶을 때
- `CoroutineScope` 생성자 함수를 이용해 클래스와 스코프를 연결할 수 있음

```kotlin
class ComponentWithScope(dispatcher: CoroutineDispatcher = Dispatchers.Default) {
    private val scope = CoroutineScope(dispatcher + SupervisorJob())

    fun start() {
        log("Starting!")
        scope.launch {
            while (true) {
                delay(500.miliseconds)
                log("Component working!")
            }
        }

        scope.launch {
            log("Doing a one-off task...")
            delay(500.miliseconds)
            log("Task done!")
        }
    }

    fun stop() {
        log("Stopping!")
        scope.cancel()
    }
}

fun main() {
    val component = ComponentWithScope()
    component.start()
    Thread.sleep(2000)
    component.stop()
}
```

- `CoroutineScope`는 클래스 내부에서 사용할 고유한 코루틴 스코프를 만들어줌
- `SupervisorJob()`은 오류가 다른 코루틴에 **전파되지 않도록 막아줌**
- `stop()`을 호출하면 scope 내 코루틴이 모두 **취소됨**

- `coroutineScope` vs `CoroutineScope` 차이

    | 구분 | coroutineScope | CoroutineScope |
    | --- | --- | --- |
    | 타입 | 일시 중단 함수 | 생성자 함수 |
    | 용도 | 동시 작업을 그룹으로 묶기 | 클래스의 생명주기와 스코프 연동 |
    | 특징 | 모든 자식 코루틴이 끝날 때까지 기다림 | 단순히 코루틴 시작할 수 있는 스코프 생성 |
    | 예시 사용 | `suspend fun` 안에서 사용 | 클래스 내부에서 scope 생성 시 사용 |


### `GlobalScope`는 위험성

```kotlin
fun main() {
	runBlocking {
    GlobalScope.launch {
        delay(1000.miliseconds)
        log("Global scope coroutine done!")
    }

    log("Parent done!")
	}
}
```

- `GlobalScope`는 부모가 없음 → 구조화된 동시성의 계층 밖임
- 부모 코루틴이 끝나도 자식 코루틴은 **계속 실행됨**
- 프로그램이 종료되면 자식 코루틴이 완료되지 못할 수도 있음
    - 특수한 주석(`@DelicateCoroutineApi`)과 함께 선언된다.

**결론**: 일반 애플리케이션에서 GlobalScope는 되도록 사용 금지!

→ 꼭 써야 한다면 전체 생명주기 동안 살아있는 **백그라운드 작업**에만 한정

### 코루틴 콘텍스트와 구조화된 동시성

```kotlin
fun main() = runBlocking(CoroutineName("A")) {
    log("A's job: ${coroutineContext.job}")
    launch(CoroutineName("B")) {
        log("B's job: ${coroutineContext.job}")
        log("B's parent: ${coroutineContext.job.parent}")
    }
    log("A's children: ${coroutineContext.job.children.toList()}")
}
```

- 새로운 코루틴은 부모의 **CoroutineContext**를 상속받는다
- 코루틴 간에는 **Job 객체**를 통해 부모-자식 관계를 형성
- 이름(`CoroutineName`), 디스패처(`Dispatchers`), Job 등은 Context에 포함
- 부모가 취소되면 자식도 자동으로 취소됨 (→ 15.2절에서 자세히 다룸)

---

## 15.2 취소

- 코루틴에서 취소가 왜 중요한가?
    - **불필요한 작업 방지**

      → 예: 사용자가 화면을 나가도 네트워크 요청은 계속됨 → 낭비

    - **리소스 누수 방지**

      → 취소할 수 없으면 DB 연결, 메모리 누수가 발생할 수 있음

    - **오류 처리와 연결됨**

      → 여러 작업 중 하나라도 실패하면 나머지는 취소해야 합리적


### 취소 촉발

- launch → `Job` 반환
- async → `Deferred` 반환
- 둘 다 `cancel()`로 취소 가능!

```kotlin
fun main() { 
	runBlocking {
    val lanchedJob = launch {
        log("I'm launched!")
        delay(1000.miliseconds)
        log("I'm done!")
    }

    val asyncDeferred = async {
        log("I'm async!")
        delay(1000.miliseconds)
        log("I'm done!")
    }

    delay(200.miliseconds)
    lanchedJob.cancel()
    asyncDeferred.cancel()
	}
}
```

### 시간제한이 초과된 후 자동으로 취소 호출

- 타임아웃이 지나면 코루틴을 자동으로 취소해주는 함수 `withTimeout`, `withTimeoutOrNull`

```kotlin
suspend fun calculate(): Int {
    delay(3000.miliseconds)
    return 2 + 2
}

fun main() = runBlocking {
    val quickResult = withTimeoutOrNull(500.miliseconds) {
        calculateSomething()
    }
    println(quickResult) // null

    val slowResult = withTimeoutOrNull(5.seconds) {
        calculate()
    }
    println(slowResult) // 4
}

```

- `withTimeout()`은 예외(`TimeoutCancellationException`)를 던집니다.
- `withTimeoutOrNull()`은 예외 대신 `null`을 반환합니다.
- 예외 처리가 귀찮으면 `withTimeoutOrNull()`이 더 간편하다.

### 취소는 모든 자식 코루틴에게 전파된다

- 부모 코루틴이 취소되면 자식 코루틴도 모두 취소됨 → 구조화된 동시성의 핵심

```kotlin
fun main() = runBlocking {
    val job = launch {
        launch {
	        launch {
		        launch {    // <- 이 코루틴은 취소된 잡의 고손자
		            log("I'm started!")
			          delay(500.miliseconds)
		            log("I'm done!")
		        }
	        }
        }
    }
    delay(200.miliseconds)
    job.cancel() // 모든 자식도 함께 취소됨
}
```

### 취소된 코루틴은 특별한 지점에서 CancellationException을 던진다.

- 코루틴 내부에서 취소되면 `CancellationException`이 발생함 → `delay()` 같은 일시 중단 지점에서 감지된다.
    - `catch (e: Exception)`처럼 넓은 예외 처리를 하면 이 취소 예외도 삼켜서 무한 루프 빠질 수 있음!

```kotlin
suspend fun doWork() {
	delay(500.milliseconds)  // 여기서 CancellationException을 던지지만
	throw UnsupportedOperationException("Didn't work!") 
}

fun main() {
	runBlocking {
		withTimeoutOrNull(2.seconds)
			while (true) {
				try {
					doWork() 
				} catch (e: Exception) {  // 여기서 예외를 삼켜버려서 취소를 막는다
					println("Oops: ${e.message}")
				}
			}
	}
}
```

### 취소는 협력적이다
- 코루틴이 스스로 취소 상태를 확인하고 협력해야만 취소가 가능하다.
- 직접 만든 suspend 함수 안에 **일시 중단 지점(delay, yield 등)** 이 없으면 취소가 감지되지 않음!

```kotlin
suspend fun doCpuHeavyWork() {
    val start = System.currentTimeMillis()
    while (System.currentTimeMillis() < start + 500) {
        // 취소가 감지되지 않음!
    }
}
```

- 이럴 땐 `isActive`, `ensureActive()` 등을 사용해 **취소 가능 지점**을 만들어야 함.

```kotlin
suspend fun doCpuHeavyWork() {
    val start = System.currentTimeMillis()
    while (System.currentTimeMillis() < start + 500) {
        ensureActive()
    }
}
```

### 코루틴이 취소됐는지 확인

```kotlin
val myJob = lanch {
	repeat(5) {
		doCpuHeavyWork()
		if(!isActive) return@lanch
	}
}

val myJob = lanch {
	repeat(5) {
		doCpuHeavyWork()
		ensureActive()
	}
}
```

- `isActive` 를 통해 코루틴이 취소되었는지 확인할 수 있다.
- `ensureActive` 코틀린 코루틴 편의 함수

    ```kotlin
    public fun Job.ensureActive(): Unit {
        if (!isActive) throw getCancellationException()
    }
    ```

### 다른 코루틴에게 기회를 주기: yield 함수

```kotlin
suspend fun doCpuHeavyWork(): Int {
    var counter = 0
    val startTime = System.currentTimeMillis()
    while (System.currentTimeMillis() < start + 500) {
        counter++
        yield() // 다른 코루틴에게 기회를 줌
    }
    return counter
}

```

- `yield()`는 다른 코루틴이 실행되도록 **양보**하는 역할을 함.

| 메서드 | 설명 |
| --- | --- |
| `isActive` | 코루틴이 아직 살아있는지 확인 |
| `ensureActive()` | 살아있지 않다면 `CancellationException` 던짐 |
| `yield()` | 취소 지점 + 다른 코루틴에게 실행 기회를 넘겨줌 |

### 리소스를 얻을 때 취소를 염두에 두기

- 코루틴 기반 코드는 항상 취소시에도 견고하게 작동하도록 설계돼야 한다.
- 리소스를 사용할 때는 반드시 **finally** 또는 **use**를 써서 정리해야 한다.

```kotlin
// finally 블록을 사용해 리소스 닫기
val  dbTask = launch {
    val db = DatabaseConnection()
    try {
        delay(500.milliseconds)
        db.write("I love coroutines...")
    } finally {
        db.close() // 반드시 닫아야 함
    }
}

// use를 사용해 리소스를 자동으로 닫기
val dbTask = lanch {
	DatabaseConnection().use {
		delay(500.meilliseconds)
		it.write("I love coroutines...")
	}
}
```

### 프레임워크가 여러분 대신 취소를 할 수 있다.

- Android → `viewModelScope` (화면 벗어나면 자동 취소)

    ```kotlin
    class MyViewModel: ViewModel() {
    	init {
    		viewModelScope.launch {
    			while (true) {
    				println("Tick!")
    				delay(1000.milliseconds)
    			}
    		}
    	}
    }
    ```

- Ktor → `PipelineContext` (요청 종료 시 자동 취소)

    ```kotlin
    routing {
    	get("/") {
    		launch {
    			println("I'm doing some background work!")
    			deplay(5000.milliseconds)
    			println("I'm done")
    		}
    	}
    }
    ```

- Application 전체 범위에서 실행하고 싶으면 → `application.launch`

    ```kotlin
    routing {
    	get("/") {
    		call.application.launch {
    			println("I'm doing some background work!")
    			delay(5000.milliseconds)
    			println("I'm done")
    		}
    	}
    }
    ```


---

## 요약

- 구조화된 동시성은 코루틴의 작업을 제어할 수 있게 해주며, '제멋대로인' 코루틴이 취소되지 않고 계속 실행되는 것을 방지한다.
- 일시 중단 함수인 coroutineScope 도우미 함수와 CoroutineScope 생성자 함수를 사용해 새로운 코루틴 스코프를 생성할 수 있다. 이름은 비슷하지만 이들의 목적은 다르다.
    - coroutineScope는 작업을 병렬로 분해하기 위한 함수로, 여러 코루틴을 시작 하고 결과를 계산한 후 그 결과를 반환한다.
    - Coroutinescope는 클래스의 생명주기와 코루틴을 연관시키는 스코프를 생성 하며, 일반적으로 SupervisorJob과 함께 사용된다.
- Globalscope는 특별한 코루틴 스코프로, 예제 코드에서 자주 볼 수 있지만 구조화된 동시성을 깨뜨리기 때문에 애플리케이션 코드에서는 사용하지 말아야 한다.
- 코루틴 콘텍스트는 개별 코루틴이 어떻게 실행되는지 관리하며, 코루틴 계층을 따라 상속된다.
- 코루틴과 코루틴 스코프 간의 부모-자식 계층 구조는 코루틴 콘텍스트에 있는  Job객체를 통해 설정된다.
- 일시 중단 지점은 코루틴이 일시 중단될 수 있고, 다른 코루틴이 작업을 시작할 수 있는 지점이다.
- 취소는 일시 중단 지점에서 CancellationException을 던지는 방식으로 구현된다.
- 취소 예외는 절대 무시(잡아내고 처리하지 않음)돼서는 안 된다. 예외를 다시 던지던지 아니면 아예 잡아내지 않는 것이 좋다.
- 취소는 정상적인 상황이므로 코드는 이를 처리할 수 있게 설계해야 한다.
- cancel이나 withTimoutOrNull 같은 함수를 사용해 직접 취소를 호출할 수 있다. 기존의 여러 프레임워크도 코루틴을 자동으로 취소할 수 있다.
- 함수에 suspend 변경자를 추가하는 것만으로는 취소를 지원할 수 없다. 하지만 코틀린 코루틴은 취소 가능한 일시 중단 함수를 작성하는 데 필요한 메커니즘(예: ensureActive나 yield 함수나 isActive 속성)을 제공한다.
- 프레임워크는 코루틴 스코프를 사용해 코루틴을 애플리케이션의 생명주기와 연결하는 데 도움을 준다(예: 화면에 ViewModel이 표시되는 동안이나 요청 핸들러가 실행되는 동안).

