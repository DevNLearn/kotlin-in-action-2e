# 코루틴
코루틴은 비동기적으로 실행되는 넌블로킹 동시성 코드를 작성할 수 있게 해준다. 스레드와 같은 전통적 방법과 비교하면 코루틴이 훨씬 더 가볍게 작동한다.

> 동시성: 여러 작업을 동시에 실행하는 것
병렬성: 여러 작업을 여러 CPU 코어에서 물리적으로 동시에 실행하는 것
>

## 1. 코틀린의 동시성 처리 방법: 일시 중단 함수와 코루틴

일시 중단 함수는 스레드를 블록시키는 단점 없이 순차적 코드처럼 보이는 동시성 코드를 작성할 수 있게 해준다.

## 2. 스레드와 코루틴 비교

| 항목 | **스레드 (Thread)** | **코루틴 (Coroutine)** |
| --- | --- | --- |
| **개념** | OS 수준의 무거운 동시성 단위 | JVM 수준의 가벼운 동시성 단위 |
| **비용** | 생성 및 전환 비용이 큼 | 매우 저렴, 수천 개 생성 가능 |
| **실행 방식** | 병렬(멀티코어 활용 가능), 독립적 실행 | 동시성(논리적 병렬성), 협력적 실행 |
| **블로킹 여부** | 블로킹(예: I/O 대기 시 스레드 중단) | 논블로킹(중단 후 재개 가능) |
| **관리 주체** | 운영체제 | 사용자 코드 / 런타임 라이브러리(Kotlin의 경우 Dispatcher 등) |
| **에러/취소 관리** | 복잡함, 수동 제어 필요 | 구조화된 동시성으로 체계적 관리 가능 |
| **사용 예시** | CPU 집중형 작업, 멀티코어 활용 | I/O 중심 비동기 작업, 네트워크 처리 등 |
| **병렬성 활용** | O (멀티코어에 직접 할당) | O (멀티스레드 Dispatchers 사용 시) |

### 코루틴

- 코루팀은 초경량 추상화다.
- 코루틴은 생성하고 관리하는 비용이 저렴하여 세밀한 작업이나 짧은 시간 동안만 실행하는 작업에도 넓게 사용 가능
- 코루틴은 시스템 자원을 블록시키지 않고 실행을 일시 중단 할 수 있어 나중에 중단된 지점에서 재실행 가능하다.
- 코루틴이 네트워크 요청이나 입출력 작업 같은 비동기 작업을 처리할 때 블로킹 스레드보다 훨씬 효율적이다.
- 코루틴은 구조화된 동시성이라는 개념을 통해 동시 작업의 구조와 계층을 확리하며, 취소 및 오류 처리를 위한 메커니즘을 제공한다.
- 동시 계산의 일부가 실패하거나 더 이상 필요하지 않게 됐을 때 구조화된 동시성은 자식으로 시작된 다른 코루틴들도 함께 취소되도록 보장한다.
- 내부적으로 코루틴은 하나 이상의 JVM 스레드에서 실행된다.
- 코루틴을 사용해 작성한 코드도 여전히 기본 스레드 모델이 제공하는 병렬성을 활용할 수 있지만, 운영체제가 부과하는 스레드의 한계에 얽매이지 않는다는 것을 의미한다.

## 3. 잠시 멈출 수 있는 함수: 일시 중단 함수

코틀린의 코루틴의 동시성 접근 방식으로 스레드, 반응형 스트림, 콜백 뿐만 아니라 일시 중단 함수를 중심으로  동시성에 접근한다.

### 일시 중단 함수를 사용한 코드는 순차적으로 보인다.

1. 어떤 형태의 동시성도 사용하지 않고 단순히 하나의 함수가 완료된 후 다음 함수를 호출하며, 마지막으로 네트워크 요청에 대한 응답이 오면 값을 반한

```kotlin
// 여러 함수를 호출하는 블로킹 코드
fun login(credentials: Credentials): UserID
fun loadUserData(userID: UserID): UesrData
fun showData(data: UserData)

fun showUserInfo(credentials: Credentials) {
    val userID = login(credentials)
    val userData = loadUserData(userID)
    showData(userData)
}
```

1. 코루틴을 사용해 넌블로킹 방식으로 구현하여 함수가 블록되어 발생하는 자원이 낭비되지 않도록 한다.

```kotlin
// 일시 중단 함수를 사용
suspend fun login(credentials: Credentials): UserID
suspend fun loadUserData(userID: UserID): UesrData
fun showData(data: UserData)

suspend fun showUserInfo(credentials: Credentials) {
    val userID = login(credentials)
    val userData = loadUserData(userID)
    showData(userData)
}
```

## 4. 코루틴을 다른 접근 방법과 비교

동시성 접근 방식으로 일반적으로 `콜백`, `반은형 스트림(RxJava)`, `퓨처(Future)` 3가지가 있다.

### 콜백

```kotlin
fun loginAsync(credentials: Credentials, callback: (UserID -> Unit)
fun loadUserDataAsync(userID: UserID, callback: (UserData) -> Unit)
fun showData(data: UserData)

fun showUserInfo(credentials: Credentials) {
    loginAsync(credentials) { userID ->
        loadUserDataAsync(UserID) { userData ->
            showData(userData)
        }
    }
}
```

콜백을 사용해 구현하려면 함수의 시그니처를 변경해 콜백 파라미터를 제공해야 한다. 하지만, 로직이 커지면 콜백이 중첩된 복잡한 코드가 되는 `콜백 지옥` 이 발생하여 가독성이 급격히 떨어진다.

### 퓨처

`CompletableFutre`를 사용하면 콜백 중첩을 피할 수 있지만, `thenCompose`와 `thenAccept` 같은 새로운 연산자의 의미를 배워야 한다. 또한, 함수의 반환 타입을 `CompletableFuture` 로 감싸야한다.

```kotlin
fun loginAsync(credentials: Credentials): CompletableFuture<UserID>
fun loadUserDataAsync(userID: UserID): CompletableFuture<UesrData>
fun showData(data: UserData)

fun showUserInfo(credentials: Credentials) {
    loginAsync(credentials)
        .thenCompose { loadUserDataAsync(it) }
        .thenAccept { showData(it) }
}
```

### 반응형 스트림(RxJava)

반응형 스트림을 사용해도 콜백 중첩을 피할 수 있지만, 함수 시그니처를 변경해야 하며 반환값을 `Single` 로 감싸야하고, `flatMap`, `doOnSuccess`, `subscribe` 같은 연산자를 사용해야 한다.

```kotlin
fun login(credentials: Credentials): Single<UserID>
fun loadUserData(userID: UserID): Single<UesrData>
fun showData(data: UserData)

fun showUserInfo(credentials: Credentials) {
    login(credentials)
        .flatMap { loadUserData(userID) }
        .doOnSuccess { showData(userData) }
        .subscribe()
}
```

`퓨처`, `반응형 스트림` 방식 모두 인지적 부가 비용이 있고, 함수를 선언하거나 사용할 때 새로운 연산자를 코드에 추가해야 한다. 코틀린 코루틴을 사용하는 방식에서는 함수에 `suspend` 변경자만 추가하면 된다.

### 일시 중단 함수 호출

일시 중단 함수는 실행을 일시 중단할 수 있는 코드 블록 안에서만 호출할 수 있다.

```kotlin
suspend fun login(credentials: Credentials): UserID
suspend fun loadUserData(userID: UserID): UesrData
fun showData(data: UserData)

suspend fun showUserInfo(credentials: Credentials) {
    val userID = login(credentials)
    val userData = loadUserData(userID)
    showData(userData)
}
```

일반적인 임시 중단 코드가 아닌 코드에서 일시 중단 함수를 호출하려고 하면 오류가 발생한다.

```kotlin
suspend fun mySuspendingFunction() {}
fun main() {
    mySuspendingFunction() 
}

// Error: Suspend function MySuspendingFunction should be
// called only from a coroutine or another suspend function
```

## 5. 코루틴의 세계로 들어가기: 코루틴 빌더

코루틴은 일시 중단 가능한 계산의 인스턴스이다. 다른 코루틴들과 동시에 실행될 수 있는 코드 블록으로 생각 할 수 있다.

코루틴을 생성할 때는 코루틴 빌더 함수 중 하나를 사용한다.

| 빌더 | 반환 값 | 설명 |
| --- | --- | --- |
| runBlocking | 람다가 계산한 값 | 블로킹 코드와 일시 중단 함수를 연결할 때 사용 |
| launch | Job | 값을 반환하지 않는 새로운 코루틴을 시작할 때 사용 |
| async | Deferred<T> | 비동기적으로 값을 계산할 때 사용 |

### 일반 코드에서 코루틴의 세계로: runBlocking 함수

일반 블로킹 코드를 일시 중단 함수의 세계로 연결하려면 runBlocking 코루틴 빌더 함수에게 코루틴 본문을 구성하는 코드 블록을 전달할 수 있다.

```kotlin
import kotlinx.coroutines.*
import kotlin.time.Duration.Companion.milliseconds

suspend fun doSomethingSlowly() {
    delay(500.milliseconds)
    println("I'm done")
}

fun main() = runBlocking {
    doSomethingsSlowly()
}
```

runBlocking을 사용 할 때는 하나의 스레드를 블로킹하지만, 해당 코루틴 안에서는 추가적인 자식 코루틴을 얼마든지 시작할 수 있고, 자식 코루틴 안에서는 추가적인 자식 코루틴을 얼마든지 시작할 수 있다. 자식 코루틴들은 다른 스레드를 더 이상 블록시키지 않는다.

### 발사 후 망각 코루틴 생성: launch 함수

`launch` 함수는 새로운 자식 코루틴을 시작하는 데 쓰인다. 코드를 실행하되 그 결과값을 기다리지 않는 경우에 적합하다.

```kotlin
private var zeroTime = System.currentTimeMillis()
fun log(message: Any?) =
    println("${System.currentTimeMillis() - zeroTime} " +
        "[${Thread.currentThread().name}] $message")
        
fun main() = runBlocking {
    log("The first, parent, coroutine starts")
    launch {
        log("The second coroutine starts and is ready to be suspended")
        delay(100.milliseconds)
        log("The second coroutine is resumed")
    }
    launch {
        log("The third coroutine can run in the meantime")
    }
    log("The first coroutine has has launched two more coroutines")
}

/*
36 [main @coroutine#1] The first, parent, coroutine starts
40 [main @coroutine#1] The first coroutine has has launched two more coroutines
42 [main @coroutine#2] The second coroutine starts and is ready to be suspended
47 [main @coroutine#3] The third coroutine can run in the meantime
149 [main @coroutine#2] The second coroutine is resumed
*/
```

> -Dkotlinx.coroutines.debug JVM 옵션 또는 코틀린 플레이그라운드에서 실행하면 스레드 이름 옆에 코루틴 이름에 대한 추가 정보를 얻을 수 있다.
>

이 코드에서는 3개의 코루틴이 시작된다.

- runBlocking에 의해 시작된 부모 코루틴
- launch 호출에 의해 시작된 자식 코루틴(coroutine#2, coroutine#3)

launch 함수는 Job 타입의 객체를 반환하는데, Job 객체를 사용하면 코루틴 실행을 제어할 수 있다.

### 일시 중단된 코루틴은 어디로 가는가?

- 코루틴이 제대로 작동할 수 있게 하는 주요 작업은 컴파일러가 수행한다.
- 컴파일러는 코루틴을 일시 중단하고 재개하며, 스케줄링하는 데 필요한 지원 코드를 생성한다.
- 일시 중단 함수의 코드는 컴파일 시점에 변환되고 실행 시점에 코루틴이 일시 중단될 때 해당 시점의 상태 정보가 메모리에 저장된다.
- 저장된 메모리 정보를 바탕으로 나중에 실행을 복구하고 재개할 수 있다.

### 대기 가능한 연산: async 빌더

비동기 계산을 수행할 때 async 빌더 함수를 쓸 수 있다. async 함수의 반환 타입은 Deferred<T> 인스턴스다. Deferred를 사용해 주로 할 일은 await라는 일시 중단 함수로 결과를 기다리는 것이다.

```kotlin
suspend fun slowlyAddNumbers(a: Int, b: Int): Int {
    log("Waiting a bit before calculating $a + $b")
    delay(100.milliseconds * a)
    return a + b
}
fun main() = runBlocking {
    log("Starting the async computation")
    val myFirstDeferred = async { slowlyAddNumbers(2, 2) }
    val mySecondDeferred = async { slowlyAddNumbers(4, 4) }
    log("Waiting for the deferred value to be avaliable")
    log("The first result: ${myFirstDeferred.await()}")
    log("The second result: ${mySecondDeferred.await()}")
}

/*
0 [main @coroutine#1] Starting the async computation
4 [main @coroutine#1] Waiting for the deferred value to be avaliable
8 [main @coroutine#2] Waiting a bit before calculating 2 + 2
9 [main @coroutine#3] Waiting a bit before calculating 4 + 4
213 [main @coroutine#1] The first result: 4
415 [main @coroutine#1] The first result: 8
*/
```

async을 호출한다고 해서 코루틴이 일시 중단되는 것은 아니다. await을 호출하면 그 Deferred에서 결과값이 사용 가능핼질 때까지 루트 코루틴이 일시 중단된다.

> Deferred 객체는 아직 사용할 수 없는 값을 나타낸다.
>

## 6. 어디서 코드를 실행할지 정하기: 디스패처

코루틴 디스패처는 코루틴을 실행할 스레드를 결정한다. 디스패처를 선택함으로써 코루틴을 특정 스레드로 제한하거나 스레드 풀에 분산시킬 수 있다.

본질적으로 코루틴은 특정 스레드에 고정되지 않는다. 코루틴은 한 스레드에서 실행을 일시 중단하고 디스패처가 지시하는 대로 다른 스레드에서 실행을 재개할 수 잇따.

### 디스패처 선택

코루틴은 기본적으로 부모 코루틴에서 디스패처를 상속받으므로 모든 코루틴에 대해 명시적으로 디스패처를 지정할 필요는 없다.

하지만, 디스패처를 선택하여 명시적 코루틴 실행에 도움을 줄 수 있다.

- 기본 환경에서 실행 할때 (Dispatchers.Default)
- UI 프레임워크와 함께 작업할 때 (Dispatcher.Main)
- 스레드를 블로킹하는 API를 사용할 때 (Dispatchers.IO)

### 다중 스레드를 사용하는 범용 디스패처: Dispatchers.Default

- 가장 일반적인 디스패처로 일반적인 작업에 사용할 수 있다.
- CPU 코어 수만큼의 스레드로 구성된 스레드 풀을 기반으로 한다.
- 기본 디스패처에서 코루틴을 스케줄링하면 여러 스레드에서 코루틴이 분산돼 실행되며 멀티코어 시스템에서는 병렬로 실행될 수 있다.
- 특정 스레드나 스레드 풀에 제한할 필요가 없는 한 기본 디스페처를 사용하는 것이 적합하다.

### UI 스레드에서 실행: Dispatchers.Main

- UI 프레임워크를 사용할 때 특정 작업을 UI 스레드나 메인 스레드라고 불리는 특정 스레드에서 실행해야 될 때 사용

### 블로킹되는 IO 작업 처리: Dispatcher.IO

- 서드파티 라이브러리를 사용할 때 코루틴을 염두에 두고 설계된 API를 선택 할 수 없는 경우 사용
- 이 디스패처에서 실행된 코루틴은 자동으로 확장되는 스레드 풀에서 실행된다.
- CPU 집약적이지 않는 작업에 적합하다.

| 디스패처 | 스레드 개수 | 설명 |
| --- | --- | --- |
| Dispatchers.Default | CPU 코어 수 | 일반적인 연산, CPU 집약적인 작업 |
| Dispatchers.Main | 1 | UI 프레임워크를 사용할 때 사용 |
| Dispatcher.IO | 64 + CPU 코어 개수 (단, 최대 64개만 병렬 실행) | 블로킹 IO 작업, 네트워크 작업, 파일 작업등에서 사용 |
| Dispatchers.Unconfined | … | 즉시 스케줄링해야하는 특별한 경우 (일반적인 용도 x) |
| limitedParallelism(n) | 커스텀(n) | 커스텀 시나리오 |

### 코루틴 빌더에 디스패처 전달

코루틴을 특정 디스패처에서 실행하기 위해 코루틴 빌더 함수에게 디스패처를 인자로 전달할 수 있다. runBlocking, launch, async 같은 모든 코루틴 빌더 함수는 코루틴 디스패처를 명시적으로 지정할 수 있다.

```kotlin
// 코루틴 빌더의 인자로 디스패처 지정
fun main() {
    runBlocking {
        log("Doing some work")
        launch(Dispatcher.Default) {
            log("Doing some background work")
        }
    }
}

/*
26 [main @coroutine#1] Doing some work
33 [DefaultDispatcher-worker-1 @coroutine#2] Doing some background work
*/
```

### withContext를 사용해 코루틴 안에서 디스패처 바꾸기

```kotlin
launch(Dispatchers.Default) {
    val result = performBackgroundOperation()
    withContext(Dispatchcers.Main) {
        updateUI(result)
    }
}
```

### 코루틴과 디스패처는 스레드 안전성 문제에 대한 마법 같은 해결책이 아니다.

`Dispatchers.Default` 와 [`Dispatchers.IO`](http://Dispatchers.IO) 는 다중 스레드를 사용하는 기본 제공 디스패처다. 다중 스레드 디스패처는 코루틴을 여러 스레드에 분산시켜 실행한다.

한 코루틴은 항상 순차적으로 실행되며 어느 단일 코루틴의 어떤 부분도 병렬로 실행되지 않는다. 이는 단일 코루틴에 연관된 데이터가 전형적인 동기화 문제를 일으키지 않는다.

하지만, 여러 코루틴이 동일한 데이터를 읽거나 변경하는 경우에는 문제가 발생할 수 있다.

```kotlin
fun main() {
    runBlocking {
        var x = 0
        repeat(10_000) {
            launch(Dispatcher.Default) {
                x++
            }
        }
        delay(1.seconds)
        println(x)
    }
}
// 9,916
```

여러 코루틴이 같은 데이터를 수정하고 있기 때문에 다중 스레드 디스패처에서 실행되면 일부 증가 작업이 서로의 결과를 덮어쓰는 사황이 발생하여 카운터 값이 예상보다 낮다.

이러한 문제를 해결하기 위한 접근 방식이 있다.

Mutex 잠금을 제공하여 코드 임계 영역이 한번에 하나의 코루틴만 실행되게 보장할 수 있다.

```kotlin
fun main() = runBlocking {
    val mutex = Mutex()
    var x = 0
    repeat(10_000) {
        launch(Dispatchers.Default) {
            mutex.withLock {
                x++
            }
        }
    }
    
    delay(1.seconds)
    println(x)
}
```

`AtomicInteger` 나 `ConcurrentHashMap` 같은 병렬 변경을 위해 설계된 원자적이고 스레드 안전한 데이터 구조를 사옹할 수도 있다.

코루틴을 단일 스레드 디스패처에서 실행하도록 제한하는 방법도 있지만, 성능 특성을 고려해야 한다. 여러 코루틴이 병렬로 동일한 데이터를 변경한다면 스레드와 마찬가지로 동기화나 잠금 처리를 해야 한다.

## 7. 코루틴은 코루틴 콘텍스트에 추가적인 정보를 담고 있다.

withContext 함수에 서로 다른 디스패처를 인자로 전달한다. 파라미터가 실제로는 CoroutineDispatcher가 아니라 CoroutineContext다.

### CoroutineContext

CoroutineContext는 각 코루틴의 추가적인 문맥 정보를 담고 있으며 여러 요소로 이루어진 집합이다.

- 코루틴이 어떤 스레드에서 실행될지를 결정하는 디스패처
- 코루틴의 생명주기
- Job 객체
- CoroutineName
- CoroutineExceptionHandler 등

```kotlin
import kotlin.coroutines.coroutineContext

suspend fun introspect() {
    log(coroutineContext) // coroutineContext 컴파일러 고유 
                          // 기능에는 코루틴에 대한 정보가 들어있다.
}
fun main() {
    runBlocking {
        introspect()
    }
}
```

코루틴 빌더나 withContext 함수에 인자를 전달하면 자식 코루틴의 콘텍스트에서 해당 요소를 덮어쓴다. 여러 파라미터를 한 번에 덮어쓰려면 + 연산자를 사용해 CoroutineContext 객체를 결합할 수 있다.

```kotlin
fun main() {
    runBlocking(Dispatcher.IO + CoroutineName("Coolroutine")) {
        introspect()
    }
}
```

## 요약

- 동시성은 여러 작업을 동시에 처리하는 것을 의미하며, 여러 작업의 여러 부분이 서로 번갈아 실행되는 방식으로 나타난다.
- 병렬성은 물리적으로 동시에 실행되면서 현대 멀티코어 시스템을 효과적으로 활용하는 것을 말한다.
- 코루틴은 스레드 위에서 동시 실행을 위해 동작하는 경량 추상화다.
- 코루틴의 핵심 동시성 기본 요소는 일시 중단 함수로, 실행을 잠시 멈출 수 있는 함수다. 다른 일시 중단 함수나 코루틴 안에서 일시 중단 함수를 호출 할 수 있다.
- 반응형 스트림, 콜백, 퓨처 같은 다른 접근 방식과 달리 일시 중단 함수를 쓸 때는 코드의 모양이 달라지지 않는다. 코드는 여전히 순차적으로 보인다.
- 코루틴은 일시 중단 가능한 계산의 인스턴스다.
- 코루틴은 스레드를 블로킹하는 문제를 피한다. 스레드 블로킹이 문제가 되는 이유는 스레드 생성에 비용이 많이 들고, 시스템 자원이 제한적이기 때문이다.
- 코루틴은 빌더인 runBlocking, launch, async를 사용해 새로운 코루틴을 생성할 수 있다.
- 디스패처는 코루틴이 실행될 스레드나 스레드 풀을 결정한다.
- 기본 제공되는 디스페처는 서로 다른 목적을 갖고 있다. Dispacthers.Default는 일반적인 용도에 쓰이며, Dispatchers.Main은 UI 스레드에서 작업을 실행할 때 사용되고, Dispatchers.IO는 블로킹되는 IO 작업을 호출할 때 사용된다.
- Dispachers.Default나 Dispatchers.IO와 같은 대부분의 디스패처는 다중스레드 디스패처이기 때문에 여러 코루틴이 병렬로 같은 데이터를 변경할 때 주의가 필요하다.
- 코루틴을 생성할 때 디스패처를 지정하거나 withContext를 사용해 디스패처를 변경할 수 있다.
- 코루틴 콘텍스트에는 코루틴과 연관된 추가 정보가 들어있다. 코루틴 디스패처는 코루틴 콘텍스트의 일부다.