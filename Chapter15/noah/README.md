# 구조화된 동시성
구조회된 동시성(Structured Concurrency)를 통해 애플리케이션 안에서 코루틴과 생애 주기의 계층을 고나리하고 추적한다.

- 코루틴을 사용할 때 많은 코루틴을 관리
- 여러 동시 작업을 할 때, 개별 작업을 추적, 취소를 통해 오류를 제대로 처리
- 코루틴 추적을 통해 리소스 누수와 불필요한 작업을 방지

## 1. 코루틴 스코프가 코루틴 간의 구조를 확립한다.

구조화된 동시성을 통해 각 코루틴은 `코루틴 스코프`에 속한다. 코루틴 스코프는 코루틴 간의 `부모-자식` 관계를 확립하는데 도움을 준다.

`launch`와 `async` 코루틴 빌더 함수는 `CoroutineScope` 인터페이스의 확장 함수로 새로운 코루틴을 만들면 자동으로 해당 코루틴의 자식이 된다.

```kotlin
fun main() {
    runBlocking {
        launch {
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
/*
20 [main @coroutine#1] Parent done!
539 [main @coroutine#3] Child 2 done!
1039 [main @coroutine#2] Child 1 done
1293 [main @coroutine#4] Grandchild done
*/
```

`runBlocking`  함수 본문은 모든 자식 코루틴이 완료될 때까지 프로그램이 종료되지 않는다.

### 코루틴 스코프 생성: coroutineScope 함수

`coroutineScope`은 새로운 코루틴을 만들지 않고 코루틴 스코프를 그룹화 하는 경우 사용하는 함수이다.

- 작업을 동시성으로 실행하기 위해 분해할 때 사용
- 새로운 코루틴 스코프를 생성하고 해당 영역의 모든 자식 코루틴이 완료될 때까지 기다리기 때문에 일시 중단 함수
- coroutineScope 은 값을 반환 가능
- 사용 사례:  동시적 작업 분해(concurrent decomposition of work)에서 여러 코루틴을 활용해 계산을 수행

```kotlin
suspend fun generateValue(): Int {
    delay(500.milliseconds)
    return Random.nextInt(0, 10)
}

suspend fun computeSum() {
    log("Computing a sum...")
    val sum = coroutineScope {
        val a = async { generateValue() }
        val b = async { generateValue() }
        a.await() + b.await()
    }
    log("Sum is $sum")
}

fun main() = runBlocking {
    computeSum()
}

/*
0 [main @coroutine#1] computing a sum...
532 [main @coroutine#1] Sum is 10
*/
```

### 코루틴 스코프를 컴포넌트와 연관시키기: CoroutineScope

`CoroutineScope`은 코루틴 빌더를 사용해 새로운 코루틴을 만들면 자체적인 CoroutineScope 생성한다.

- 실행을 일시 중단하지 않음
- 구체적 생명주기를 정의하고 동시 처리나 코루틴의 시작과 종료를 관리하는 클래스를 만들고 싶을때 사용
- 코루틴 스코프와 연관된 코루틴 컨텍스트를 파라미터로 받음
- 반환된 코루틴 스코프를 나중에 취소 가능
- 기본적으로 `디스패처`만으로 Job이 자동으로 생성되지만, 실무에서는 `CoroutineScope`과 `SupervisorJob` 을 함께 사용 권장

> SupervisorJob은 동일한 영역과 관련된 다른 코루틴을 취소하지 않고, 처리되지 않은 예외를 전파하지 않게 해주는 특수한 Job
>

```kotlin
class ComponentWithScope(dispatcher: CoroutineDispatcher = Dispatchers.Default) {
    private val scope = CoroutineScope(dispatcher + SupervisorJob())
    
    fun start() {
        log("Starting!")
        scope.launch {
            while(true) {
                delay(500.milliseconds)
                log("Component working!")
            }
        }
        scope.launch {
            log("Doing a one-off task...")
            delay(500.milliseconds)
            log("Task done!")
        }
    }
    
    fun stop() {
        log("Stopping!")
        scope.cancle()
    }
}

fun main() {
    val c = ComponentWithScope()
    c.start()
    Thread.sleep(2000)
    c.stop()
}

/*
22 [main] Starting!
37 [DefaultDispatcher-worker-2 @coroutine#2] Doing a one-off task
544 [DefaultDispatcher-worker-1 @coroutine#2] Task done!
544 [DefaultDispatcher-worker-2 @coroutine#1] Component working!
1050 [DefaultDispatcher-worker-1 @coroutine#1] Component working!
1555 [DefaultDispatcher-worker-1 @coroutine#1] Component working!
*/
```

### GlobalScope의 위험성

`GlobalScope`은 전역 수준에 존재하는 코루틴 스코프다.

- GlobalScope를 사용하면 구조화된 동시성이 제공하는 이점을 사용할 수 없다.
- 전역 범위에서 시작된 코루틴은 자동으로 취소되지 않고, 생명주기에 대한 개념이 없다
- 리소스 누수가 발생하거나 불필요한 작업을 계속 사용하며 자원을 낭비하게 될 가능성이 크다
- GlobalScope를 선택해야 하는 경우는 매우 드물며, 보통은 코루틴 빌더나 coroutineScope 함수를 사용해 더 적합한 영역을 찾아 시작하는것 이 좋다.

```kotlin
fun main() {
    runBlocking {
        GlobalScope.launch {
            delay(1000.milliseconds)
            launch {
                delay(250.millseconds)
                log("Grandchild done")
            }
            log("Child 1 done!")
        }
        GlobalScope.launch {
            delay(500.milliseconds)
            log("Child 2 done!")
        }
        log("Parent done!")
    }
}

// 28 [main @coroutine#1] Parent done!
```

GlobalScope을 사용함으로써 구조화된 동시성에서 자동으로 설정되는 계층 구조가 깨저 즉시 종료된다.

coroutine#2 ~ coroutine#4는 runBlocking과 연관된 coroutine#1과의 부모 관계에서 벗어나 있어, 부모 코루틴이 없으므로 프로그램은 자식들이 완료되기 전에 종료된다.

이러한 이유로 GlobalScope은 `@DelicateCoroutinesApi` 주석과 함께 선언된다.

### 코루틴 콘텍스트와 구조화된 동시성

`코루틴 콘텍스트` 는 코루틴이 실행될 때 사용되는 정보로 Job 객체, 디스패처 등의 정보를 가지고 있다.

**새로운 코루틴을 시작할 때 코루틴 콘텍스트의 흐름**

1. 부모의 콘텍스트를 상속
2. 부모-자식 관계를 설정하는 역할을 하는 새 Job 객체 생성
3. 코루틴 콘텍스트에 전달된 인자 적용 (해당 인자들은 상속받은 값을 덮어 쓸 수 있음)

```kotlin
fun main() {
    runBlocking(Dispatchers.Default) {
        log(coroutineContext)
        launch {
            log(corooutineContext)
            launch(Dispatchers.IO + CoroutineName("mine") {
                log(coroutineContext)
            }
        }
    }
}

/**
0 [DefaultDispatcher-worker-1 @coroutine#1] [CoroutineId(1), 
"coroutine#1": BlockingCoroutine{Active}]@12323, Dispatchers.Default]
1 [DefaultDispatcher-worker-2 @coroutine#2] [CoroutineId(2), 
"coroutine#2": StandaloneCoroutine{Active}]@52312, Dispatchers.Default]
2 [DefaultDispatcher-worker-3 @mime#3] [CoroutineId(3), 
"coroutine#3": StandaloneCoroutine{Active}]@43535, Dispatchers.IO]
**/
```

코루틴 간의 부모-자식 관계 뿐만 아니라, 코루틴과 연관된 Job 간의 관계도 확인이 가능하다.

```kotlin
fun main() = runBlocking(CoroutineName("")) {
    log("A's job: ${coroutineCOntext.job}")
    launch(CoroutineName("B")) {
        log("B's job: ${coroutineContext.job}")
        log("B's parent: ${coroutineCOntext.job.parent}")
        log("A's children: ${coroutineContext.job.children.toList()}")
    }
}

/*
0 [main @A#1] A's job: "A#1": BlockingCoroutine{Active}@41
10 [main @A#1] A's children: ["B#2": StandaloneCoroutine{Active}@24]
11 [main @B#2] B's Job: "B#2": BlockingCoroutine{Active}@24
11 [main @B#1] B's parent: "A#1": BlockingCoroutine{Active}@41
*/
```

launch와 async 같은 빌더 함수로 시작된 코루틴과 만찬가지로, coroutineScope 함수도 자체 Job 객체를 갖고 무모-자식 계층 구조에 참여하기 때문에, coroutineScope의 coroutineContext.job 속성을 통해 확인 가능하다.

```kotlin
fun main() = runBlocking<Unit> {
    log("A's job: ${coroutineContext.job}")
    coroutineScope {
        log("B's parent: ${coroutineCOntext.job.parent}")
        log("B's job: ${coroutineCOntext.job}")
        launch {
            log("C's parent: ${coroutineCOntext.job.parent}")
        }
    }
}

/*
0 [main @coroutine#1] A's job: "coroutine#1":BlockingCoroutine{Active}@41
2 [main @coroutine#1] B's parent: "coroutine#1":BlockingCoroutine{Active}@41
2 [main @coroutine#1]] B's Job: "coroutine#1":ScopeCoroutine{Active}@56
4 [main @coroutine#2]] C's parent: "coroutine#1":ScopeCoroutine{Active}@56
*/
```

## 2. 코루틴 취소

### 취소 촉발

코루틴 빌더 함수의 반환값을 취소를 촉발하는 핸들로 사용할 수 있다.

코루틴 빌더는 `job`, async는 `Deffered` 를 반환한다. 각 코루틴 스코프의 코루틴 콘텍스트에도 `job`이 포함되어 있으면 이를 사용해 영역을 취소할 수 있다.

### 시간제한이 초과된 후 자동으로 취소 홏출

코틀린 코루틴 라이브러리는 코루틴의 취소를 자동으로 촉발 할 수 이쓴 몇가지 함수를 제공한다.

- withTimeout : 타임아웃이 되면 `TimeoutCancellationException` 예외를 발생시킨다.
- withTimeoutOrNull : 타임아웃이 발생하면 `null`을 반환한다.

```kotlin
suspend fun calculateSomthing(): Int {
    delay(3.seconds)
    return 2 + 2
}

fun main() = runBlocking {
    val quickResult = withTimeoutOrNull(500.milliseconds) {
        calculateSomthing()
    }
    println(quickResult) // null
    val slowResult = withTimeoutOrNull(5.seconds) {
        calculateSomething()
    }
    println(slowResult) // 4
}
```

### 취소는 모든 자식 코루틴에게 전파된다.

코루틴을 취소하면 해당 코루틴의 모든 자식 코루틴도 자동으로 취소된다.

코루틴이 중첩돼 있는 경우에도 가장 바깥쪽 코루틴을 취소하면 손자 코루틴까지도 모두 취소된다.

```kotlin
fun main() = runBlocking {
    val job = launch {
        launch {
            launch {
                launch {
                    log("I'm started")
                    delay(500.milliseconds)
                    log("I'm done!")
                }
            }
        }
    }
    delay(200.milliseconds)
    job.cancel()
}
// 0 [main @coroutine#5] I'm started
```

### 취소된 코루틴은 특별한 지점에서 CancellationException을 던진다.

취소된 코루틴은 일시 중단 지점에서 `CancellationException`을 던진다.

```kotlin
coroutineScope {
    log("A")
    delay(500.milliseconds) // 취소 지점
    log("B")
    log("C")
}
// A 또는 ABC 가 출력된다.
```

코루틴은 예외를 사용해 코루틴 계층에서 취소를 전파하기 때문에 예외 핸들링이 주의해야한다.

```kotlin
suspend fun doWork() {
    delay(500.milliseconds)
    throw UnsupportedOperationException("Didn't work!")
}
fun main() {
    runBlocking {
        withTimeoutOrNull(2.seconds) {
            while(true) {
                try {
                    doWork()
                } catch (e: Exception) {
                    println("Oops: ${e.message}")
                }
            }
        }
    }
}
```

1. 2초 후 withTimeoutOrNull 함수는 자식 코루틴 스코프의 취소를 요청
2. delay 호출이 CancellationException을 던짐
3. catch 구문에서 모든 종류의 예외를 잡아 코드는 무한 반복

### 취소는 협력적이다.

코틀린 코루틴에 기본적으로 포함된 모든 함수는 이미 취소 가능하다.

케이토와 같은 라이브러리에서 제공하는 일시 중단 API를 사용할 때도 해당 라이브러리의 일시 중단 함수는 내부적으로 취소 가능하다.

하지만 직접 작성한 코드는 직접 코루틴을 취소 가능하게 만들어야 한다.

```kotlin
suspend fun doCpuHeavyWork(): Int {
    log("I'm doing work!")
    var counter = 0
    val startTime = System.currentTImeMills()
    while (System.currentTimeMillis() < startTime + 500) {
        counter++
        delay(100.milliseconds) // 취소 가능 시점
    }
    return counter
}

fun main() {
    runBlocking {
        val myJob = launch {
            repeat(5) {
                doCpuHeavyWork()
            }
        }
        delay(600.milliseconds)
        myJob.cancel()
    }
}
```

### 코루틴이 취소됐는지 확인

코틀린 코루틴에는 코드를 취소 가능하게 만드는 유틸리티 함수들이 있다.

| 함수/프로퍼티 |  |
| --- | --- |
| isActive | CoroutineScope의 속성으로 `false` 이면 활성 상태가 아니며 현재 작업을 완료하고 획득한 리소스를 닫은 후 반환할 수 있다. |
| ensureActive | 코틀린 코루틴의 편의 함수로 활성 상태가 아니면 `CancellationException`을 던진다. |
| yield() | 코드 안에서 취소 가능 지점을 제공할 뿐만 아니라 현재 점유된 디스패처에서 다른 코루틴이 작업할 수 있게 해준다. |

```kotlin
val myJob = launch {
    repeat(5) {
        doCpuHeavyWork()
        if(!isActive) return@launch
    }
}

val myJob = launch {
    repeat(5) {
        doCpuHeavyWork()
        ensureActive()
    }
}
```

### 다른 코루틴에게 기회를 주기: yield 함수

```kotlin
fun doCpuHeavyWork(): Int {
    var counter = 0
    val startTime = System.currentTimeMillis()
    while (System.currentTImeMillis() < startTIme + 500) {
        counter++
        yield()
    }
    return counter
}
fun main() {
    runBlocking {
        launch {
            repeat(3) {
                doCpuHeavyWork()
            }
        }
        launch {
            repeat(3) {
                doCpuHeavyWork()
            }
        }
    }
}
/*
29 [main @coroutine#2] I'm doing work!
533 [main @coroutine#3] I'm doing work!
1036 [main @coroutine#2] I'm doing work!
1537 [main @coroutine#3] I'm doing work!
2042 [main @coroutine#2] I'm doing work!
2543 [main @coroutine#3] I'm doing work!
*/
```

### 리소스를 얻을 때 취소를 염두에 두기

데이터베이스, IO 등과 같은 리소스를 사용해 작업해야 하는 경우 이를 명시적으로 닫아줘야한다.

```kotlin
class DatabaseConnection : AutoCloseble {
    fun write(s: String) = println("writing $s!")
    override fun close() {
        println("Closing!")
    }
}
```

1. finally 블록을 사용해 리소스 닫기

```kotlin
val dbTask = launch {
    val db = DatabaseConnection()
    try {
        delay(500.milliseconds)
        db.write("I love coroutines!"0
    } finally {
        db.close()
    }
}
```

1. AuthCloseable 인터페이스 구현 후 `use` 함수를 사용

```kotlin
val dbTask = launch {
    val db = DatabaseConnection().use {
        delay(500.milliseconds)
        db.write("I love coroutines!"0
    }
}
```

### 프레임워크가 여러분 대신 취소를 할 수 있다.

많은 애플리케이션 프레임워크가 코루틴 스코프를 제공하고 취소를 자동으로 처리한다. 이런 경우 사용자는 적절한 코루틴 스코프를 선택하고, 취소될 수 있도록 설계해야 한다.

`안드로이드`는 ViewModel 클래스가 viewModelScope를 제공한다.

1. 사용자가 viewModel이 표시된 화면을 벗어날 때
2. viewModelScope가 취소되며
3.  이 스코프 안에서 실행된 모든 코루틴도 함께 취소한

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

`케이토`는 각 요청 핸들러는 CoroutineScope를 상속한 PipelineContext 객체를 암시적 수신자로 갖는데, 클라이언트가 연결을 끊으면 이 코루틴 스코프가 취소된다.

```kotlin
routing {
    get("/") {
        launch {
            println("I'm doing some background work!")
            delay(5000.milliseconds)
            println("I'm done")
        }
    }
}
```

클라이언트와 상관없이 비동기적으로 계속 작업을 수행해야 하는 경우에는 다른 스코프를 선택해야 한다.

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

## 3. 요약

- 구조화된 동시성은 코루틴의 작업을 제어할 수  있게 해주며, 코루틴이 취소되지 않고 계속 실행되는 것을 방지한다.
- 일시 중단 함수인 coroutineScope 도우미 함수와 CoroutineScope 생성자 함수를 사용해 새로운 코루틴 스코프를 생성할 수 있다. 이름은 비슷하지만 목적은 다른다
    - coroutineScope는 작업을 병렬로 분해하기 위한 함수로, 여러 코루틴을 시작하고 겨로가를 계산한 후 그 결과를 반환한다.
    - CoroutineScope는 클래스의 생명주기와 코루틴을 연관시키는 스코프를 생성하며, 일반적으로 SupervisorJob과 함께 사용된다.
- GlobalScope는 특별한 코루틴 스코프로, 예제 코드에서 자주 볼 수 있지만 구조화된 동시성을 깨뜨리기 때문에 애플리케이션 코드에서는 사용하지 말아야 한다.
- 코루틴 콘텍스트는 개별 코루틴이 어떻게 실행되는 관리하며, 코루틴 계층을 따라 상속된다.
- 코루틴과 코루틴 스코프 간의 부모-자식 계층 구조는 코루틴 콘텍스트에 있는 Job 객체를 통해 설정된다.
- 일시 중단 지점은 코루틴이 일시 중단될 수 있고, 다른 코루틴이 작업을 시작할 수 있는 지점이다.
- 취소는 일시 중단 지점에서 CancellationException을 던지는 방식으로 구현된다.
- 취소 예외는 절대 무시돼서는 안된다. 예외를 다시 던지거나 잡지 않는 것이 좋다.
- 취소는 정상적인 상황이므로 코드는 이를 처리할 수 있게 설계해야 한다.
- cancel이나 withTimeoutOrNull 같은 함수를 사용해 직접 취소를 호출할 수 있다. 기존의 여러 프레임워크도 코루틴을 자동으로 취소할 수 있다.
- 함수에 suspend 변경자를 추가하는 것만으로는 취소를 지원할 수 없다. 하지만 코틀린 코루틴은 취소 가능한 일시 중단 함수를 작성하는 데 필요한 매커니즘을 제공한다.
- 프레임워크는 코루틴 스코프를 사용해 코루틴을 애플리케이션의 생명주기와 연결하는 데 도움을 준다.