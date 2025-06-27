# 14장 코루틴

## **14.1 동시성과 병렬성**

| **개념** | **설명** |
| --- | --- |
| **동시성(Concurrency)** | 여러 작업을 **순차적으로 빠르게 전환**하며 실행하는 것.한 번에 하나씩 실행되지만 마치 동시에 실행되는 것처럼 보임. |
| **병렬성(Parallelism)** | 여러 작업을 **동시에 물리적으로 실행**하는 것.여러 CPU 코어를 사용하여 진짜 동시에 처리함. |

→  Kotlin의 코루틴은 동시성에 특화되어 있으며, 가볍고 효율적인 비동기 프로그래밍이 가능하다.

---

## **14.2 코틀린의 동시성 처리 방법: 일시 중단 함수와 코루틴**

- Kotlin에서 동시성을 구현하는 기존 방식들
    1. Thread 직접 사용
        - 무거움. 수천 개 생성에 부적합.
    2. ExecutorService + Future
        - submit, get 방식 → 결과 대기 중 블로킹 발생.
    3. 비동기 콜백
    - 함수에 콜백을 전달하여 비동기 실행. → 콜백 지옥(callback hell) 문제가 생길 수 있음.

  → 이 문제들을 해결하기 위해 Kotlin에서는 **코루틴(Coroutine)** 이라는 새로운 동시성 기법을 언어 차원에서 지원함.


---

## **14.3 스레드와 코루틴 비교**

### 스레드란 무엇인가?

- JVM에서 동시성과 병렬성을 다루는 전통적인 방식은 스레드이다.
- 코틀린도 자바와 100% 호환되므로, 자바처럼 스레드를 사용할 수 있다.
- 예를 들어, 코틀린에서는 kotlin.concurrent.thread 함수를 사용해 간단하게 스레드를 만들 수 있다.

    ```kotlin
    import kotlin.concurrent.thread
    
    fun main() {
        println("I'm on ${Thread.currentThread().name}")
        thread {
            println("And I'm on ${Thread.currentThread().name}")
        }
    }
    ```

    - 실행 결과

        ```kotlin
        I'm on main
        And I'm on Thread-0
        ```

        - `main` 스레드는 기본 실행 스레드
        - `Thread-0`은 우리가 새로 만든 스레드

### 스레드의 한계

- 스레드는 강력하지만 **비용이 크고, 제약이 많다.**
    - JVM의 스레드는 **운영체제에서 관리**하는 **무거운 단위**
    - **스레드 하나당 몇 MB의 메모리**를 필요로 하며, 수천 개 이상을 띄우는 건 어렵다.
    - 스레드 전환은 운영체제 커널 수준의 작업 → 오버헤드 큼
    - 네트워크 요청처럼 기다리는 작업에서는 **스레드가 블로킹됨** (자원 낭비)
    - 스레드에는 **계층적 구조가 없음** → 작업의 취소, 예외 관리가 어렵습니다.
    - 구조화된 동시성의 부재 → 코드가 지저분해지고 자원 누수 발생 가능

### 코루틴의 등장

- **코루틴(coroutine)**은 코틀린이 제안하는 **경량 동시성 처리 방식이다.**
- 코루틴의 장점 정리

    | 항목 | 설명 |
    | --- | --- |
    | 초경량 | 한 프로세스에서 수백만 개의 코루틴 실행 가능 |
    | 스레드 대체 | 대부분의 스레드 작업을 코루틴으로 대체 가능 |
    | 일시 중단 | `delay()` 같은 함수를 통해 실행을 **중단**하고 **재개** 가능 |
    | 구조화된 동시성 | 부모-자식 관계로 코루틴 계층 구성 → **취소와 예외 처리 간편** |
    | 비용 절감 | 생성/소멸이 빠르고, 메모리도 거의 안 듦 |
    - 예를 들어, 코루틴에서는 `delay()`처럼 비동기 처리 중에도 **스레드를 점유하지 않는다.**
- 내부적으로는 스레드를 사용한다.
    - 코루틴도 **JVM 스레드 위에서 실행된다.**
    - 하지만 **운영체제 스레드의 한계에 영향을 받지 않음**.
    - 즉, 코루틴은 **스레드의 효율성을 살리되, 제약은 벗어난 구조**이다.

    ```kotlin
    한 프로세스
     ├─ 수천 개 스레드
     │   └─ 각각 수백~수만 개 코루틴
    ```


### 코루틴과 프로젝트 룸

- Project Loom이란?
    - JVM이 도입하려는 **가상 스레드 기반의 동시성 모델**
    - 목표: **무거운 OS 스레드 없이도** 가볍게 동시 작업을 수행
- 코루틴 vs 프로젝트 룸의 차이점

    | 항목 | 코틀린 코루틴 | Project Loom |
    | --- | --- | --- |
    | 도입 시기 | 2016년부터 성숙하게 운영 중 | 현재 진행 중 |
    | 플랫폼 | Kotlin/Native, JS 등 범용 | JVM 전용 |
    | 일시 중단 | `suspend` 키워드로 명확히 구분 | 없음 (함수 차원 구분 불가능) |
    | 동시성 구조화 | 언어 수준에서 제공 | 실험 중 |
    | 비용 | 매우 저렴, 구조적 동시성 내장 | 기존 코드 호환성에 초점, 약간 무거움 |
- 즉, **코루틴은 애초에 구조화된 동시성과 저비용 작업 실행을 고려한 설계**이다.

---

## 14.4 잠시 멈출 수 있는 함수: 일시 중단 함수

- 코틀린의 코루틴이 기존의 **스레드, 리액티브 스트림, 백프레셔 기반 동시성** 기법과 다른 점은, 코드를 동기식처럼 순차적으로 작성할 수 있으면서도, 내부적으로는 비동기로 처리된다는 것이다.

  그 중심에 **일시 중단 함수 (`suspend` function)** 가 있다!


### 일시 중단 함수를 사용한 코드는 순차적으로 보인다.

- 일시 중단 함수가 왜 중요한지를 이해하기 위해, 먼저 **블로킹 방식**의 코드를 살펴보겠다.
- 블로킹 코드

    ```kotlin
    fun login(credentials: Credentials): UserID
    fun loadUserData(userID: UserID): UserData
    fun showData(data: UserData)
    
    fun showUserInfo(credentials: Credentials) {
        val userID = login(credentials)
        val userData = loadUserData(userID)
        showData(userData)
    }
    ```

    - `login`, `loadUserData`는 네트워크 요청처럼 시간이 오래 걸리는 작업이다.
    - 위 코드는 **순차적**이지만, 각 함수가 **네트워크 응답을 기다리는 동안 스레드를 블로킹**한다.
- 블로킹의 문제점
    - 블로킹 중에는 **기저 스레드가 아무 작업도 하지 않고 대기**한다.
    - 네트워크 요청을 대량 처리하거나, UI 스레드에서 사용하면 성능이 급격히 저하된다.
    - 예를 들어 UI 애플리케이션에서 이 함수가 실행되면, UI가 멈춰버릴 수 있다.

### 일시 중단 함수 (`suspend`)로 개선

- 일시 중단 함수를 이용한 비블로킹 코드

    ```kotlin
    suspend fun login(credentials: Credentials): UserID
    suspend fun loadUserData(userID: UserID): UserData
    fun showData(data: UserData)
    
    suspend fun showUserInfo(credentials: Credentials) {
        val userID = login(credentials)
        val userData = loadUserData(userID)
        showData(userData)
    }
    ```

    - 변화한 점
        - `suspend` 키워드가 추가되었을 뿐, **코드 구조는 그대로!**
        - 여전히 위에서 아래로, **순차적인 코드처럼 보인다.**
- 일시 중단 함수란?
    - `suspend` 키워드를 붙인 함수는 **일시적으로 실행을 중단할 수 있는 함수**
    - 예를 들어 네트워크 응답을 기다리는 동안 **실행을 일시 중단(suspend)** 하고,

      → **나중에 중단된 지점부터 다시 실행(resume)** 된다.


> **중요한 점**: 이 모든 과정에서 **실행 중인 스레드는 블로킹되지 않는다!**
>

### 일시 중단 함수의 장점

| 기존 방식 (블로킹) | 일시 중단 함수 (비블로킹) |
| --- | --- |
| 스레드를 점유한 채 대기 | 스레드를 점유하지 않고 대기 |
| 대기 중 다른 작업 불가 | 대기 중 다른 코드를 실행 가능 |
| UI 멈춤, 처리량 저하 | UI 부드럽게 동작, 처리량 향상 |
| 동시 작업 수 제한 많음 | 수천 개의 작업도 무리 없음 |

### 주의할 점

- 일시 중단 함수의 효과를 제대로 누리려면, **호출되는 함수들 (`login`, `loadUserData`)도 suspend로 작성**되어야 한다.
- 즉, 내부 동작이 실제로 **비동기**로 이루어져야 한다.
- 이를 위해 많은 코틀린 라이브러리들이 **코루틴 친화적 API**를 제공
- 대표적인 코루틴 친화 라이브러리
    - **Ktor HTTP Client**
    - **Retrofit + OkHttp (코루틴 어댑터)**
    - **Room (Android DB ORM)**

---

## 14.5 코루틴을 다른 접근 방법과 비교

- 많은 프로그래밍 언어에서 동시성을 처리하는 다양한 방식이 존재한다.
- 이 절에서는 **코루틴이 콜백, 퓨처, 반응형 스트림(RxJava)** 과 어떻게 다른지 비교하며, **코루틴의 장점**을 살펴본다.

### 콜백 기반 접근 방식

```kotlin
fun loginAsync(credentials: Credentials, callback: (UserID) -> Unit)
fun loadUserDataAsync(userID: UserID, callback: (UserData) -> Unit)
fun showData(data: UserData)

fun showUserInfo(credentials: Credentials) {
    loginAsync(credentials) { userID ->
        loadUserDataAsync(userID) { userData ->
            showData(userData)
        }
    }
}

```

- 문제점
    - **콜백 안에 또 콜백**이 들어가는 구조 → "콜백 지옥(callback hell)"
    - 코드 흐름 파악이 어렵고, 예외 처리나 에러 디버깅이 힘듦
    - 로직이 복잡해질수록 **가독성 및 유지보수성**이 급격히 떨어짐

### 퓨처(CompletableFuture) 기반 접근 방식

- 콜백 지옥을 피하기 위해 **Java의 `CompletableFuture`** 같은 Future 기반 API를 사용할 수 있다.
- CompletableFuture 사용

    ```kotlin
    fun loginAsync(credentials: Credentials): CompletableFuture<UserID>
    fun loadUserDataAsync(userID: UserID): CompletableFuture<UserData>
    fun showData(data: UserData)
    
    fun showUserInfo(credentials: Credentials) {
        loginAsync(credentials)
            .thenCompose { loadUserDataAsync(it) }
            .thenAccept { showData(it) }
    }
    ```

- 단점
    - `thenCompose`, `thenAccept` 등 **새로운 연산자**를 익혀야 함
    - 함수의 반환 타입이 `CompletableFuture<T>`로 **복잡해짐**
    - 예외 처리 및 흐름 제어가 여전히 직관적이지 않음

### 반응형 스트림(RxJava) 기반 접근 방식

- RxJava는 함수형 연산자 조합을 통해 콜백 지옥을 피할 수 있지만, 여전히 **코드가 추상화에 의존**하게 된다.
- RxJava 사용

    ```kotlin
    fun login(credentials: Credentials): Single<UserID>
    fun loadUserData(userID: UserID): Single<UserData>
    fun showData(data: UserData)
    
    fun showUserInfo(credentials: Credentials) {
        login(credentials)
            .flatMap { loadUserData(it) }
            .doOnSuccess { showData(it) }
            .subscribe()
    }
    ```

- 단점
    - `flatMap`, `doOnSuccess`, `subscribe` 등 **다양한 연산자 학습 필요**
    - 반환 타입도 `Single<T>` 등 Rx 타입으로 변경
    - 흐름이 함수형 체이닝 구조라서 **직관적인 순서 이해가 어려움**

### 코루틴 기반 접근 방식의 장점

- `suspend` 키워드만 추가하면 비동기 동작을 구현할 수 있음
- 기존 함수 호출 구조를 **변경하지 않아도 됨**
- 코드가 **순차적으로 읽히고**, **가독성이 좋음**
- 스레드를 블로킹하지 않으면서도 **동기 코드처럼 동작**

```kotlin
suspend fun login(credentials: Credentials): UserID
suspend fun loadUserData(userID: UserID): UserData
fun showData(data: UserData)

suspend fun showUserInfo(credentials: Credentials) {
    val userID = login(credentials)
    val userData = loadUserData(userID)
    showData(userData)
}

```

> "여전히 순차적이지만, 블로킹은 없다!”
>

### 일시 중단 함수 호출

- `suspend` 함수는 아무 데서나 호출할 수 있을까?
    - 아니요. 일시 중단 함수는 일시 중단 가능한 문맥 안에서만 호출할 수 있다.
- 즉, 다음과 같이 **일반 함수**에서는 `suspend` 함수 호출이 불가능!

    ```kotlin
    suspend fun mySuspendingFunction() {}
    
    fun main() {
        mySuspendingFunction() // 컴파일 오류 발생
    }
    ```

    - 오류 메시지

        ```kotlin
        Suspend function 'mySuspendingFunction' should be called only from a coroutine or another suspend function.
        ```


### 일시 중단 함수를 호출하는 방법

1. `main` 함수를 suspend로 변경

    ```kotlin
    suspend fun main() {
        showUserInfo(credentials)
    }
    ```

    - 작고 단순한 유틸리티 프로그램에서 사용 가능
    - 안드로이드 등에서는 `main()` 시그니처를 수정하기 어려움
2. 코루틴 빌더 사용 (`runBlocking`, `launch`, `async` 등)

    ```kotlin
    fun main() = runBlocking {
        showUserInfo(credentials)
    }
    ```

    - `runBlocking`: 현재 스레드에서 코루틴을 시작 (테스트나 main 함수용)
    - `launch`, `async`: 실제 비동기 작업을 정의할 때 사용

| 접근 방식 | 특징 | 단점 |
| --- | --- | --- |
| 콜백 | 직관적이나 중첩으로 인한 콜백 지옥 발생 | 코드 가독성 및 유지보수성 떨어짐 |
| 퓨처 | 중첩은 피하나, then 연산자 체이닝 필요 | 추상화 복잡도 증가, 예외 처리 까다로움 |
| RxJava | 함수형 연산자 조합으로 비동기 흐름 처리 | 연산자 학습 필요, 타입 추적 어려움 |
| **코루틴** | `suspend`만 붙이면 순차 코드 그대로 유지 | 초기엔 코루틴 문법에 익숙해져야 함 |

---

## 14.6 코루틴의 세계로 들어가기: 코루틴 빌더

- 코루틴이란?
    - **코루틴**은 일시 중단 가능한 계산의 인스턴스
    - 스레드처럼 여러 작업을 동시에 실행할 수 있지만, 더 가볍고 효율적
    - **일시 중단 함수 (`suspend`)는 다른 일시 중단 함수나 코루틴 내에서만 호출**할 수 있다.
- 코루틴을 생성하려면 **코루틴 빌더 함수**를 사용해야 합니다. 주요 코루틴 빌더는 다음과 같다.

| 코루틴 빌더 | 설명 |
| --- | --- |
| `runBlocking` | 일반 블로킹 코드와 코루틴 세계를 연결합니다. 테스트나 `main`에서 사용 |
| `launch` | 값을 반환하지 않고 새로운 코루틴을 시작합니다 |
| `async` | 값을 비동기적으로 계산하고 `Deferred`로 반환합니다 |

### `runBlocking`: 일반 코드에서 코루틴 시작

- `runBlocking`은 일반 함수(`main`) 안에서 코루틴을 실행할 수 있도록 해주는 특별한 함수이다.
- 코루틴이 완료될 때까지 현재 스레드를 **블로킹**한다.

    ```kotlin
    import kotlinx.coroutines.*
    import kotlin.time.Duration.Companion.milliseconds
    
    suspend fun doSomethingSlowly() {
        delay(500.milliseconds)
        println("I'm done")
    }
    
    fun main() = runBlocking {
        doSomethingSlowly()
    }
    
    ```

    - `doSomethingSlowly()`는 일시 중단 함수
    - `main` 함수는 `runBlocking`을 통해 코루틴으로 감쌈

> 테스트나 간단한 스크립트 환경에서는 메인 스레드 블로킹이 문제되지 않음.
대신 코루틴 안에서 **자식 코루틴을 실행하면 스레드를 블로킹하지 않고 협력적으로 실행 가능**.
>

### `launch`: 값을 반환하지 않는 코루틴 생성

- `launch`는 새로운 코루틴을 시작하지만, **결과를 기다리지 않는 "발사 후 망각"** 스타일이다.
- 결과값보다는 **부수 효과가 필요한 작업**에 적합!
- `launch`를 통해 여러 코루틴을 실행 예제

    ```kotlin
    private var zeroTime = System.currentTimeMillis()
    fun log(msg: Any?) {
        println("${System.currentTimeMillis() - zeroTime} [${Thread.currentThread().name}] $msg")
    }
    
    fun main() = runBlocking {
        log("The first, parent, coroutine starts")
    
        launch {
            log("The second coroutine starts and is ready to be suspended")
            delay(100)
            log("The second coroutine is resumed")
        }
    
        launch {
            log("The third coroutine can run in the meantime")
        }
    
        log("The first coroutine has launched two more coroutines")
    }
    ```

    - 실행 결과

        ```kotlin
        36 [main @coroutine#1] The first, parent, coroutine starts  
        40 [main @coroutine#1] The first coroutine has launched two more coroutines  
        42 [main @coroutine#2] The second coroutine starts and is ready to be suspended  
        47 [main @coroutine#3] The third coroutine can run in the meantime  
        149 [main @coroutine#2] The second coroutine is resumed  
        ```

- 코루틴은 모두 **main 스레드에서 순차적으로 실행**
- `delay`로 일시 중단되면 해당 코루틴은 **스케줄러에 의해 잠시 대기**하고, 다른 코루틴에게 **스레드가 양도**됨

### `async`: 값을 반환하는 비동기 코루틴

- `async`는 값을 비동기적으로 계산하는 코루틴을 시작하며, 결과를 나중에 `await()`로 받을 수 있다. 반환 타입은 `Deferred<T>` 이다.

    ```kotlin
    suspend fun slowlyAddNumbers(a: Int, b: Int): Int {
        log("Waiting a bit before calculating $a + $b")
        delay(100L * a)
        return a + b
    }
    
    fun main() = runBlocking {
        log("Starting the async computation")
    
        val myFirstDeferred = async { slowlyAddNumbers(2, 2) }
        val mySecondDeferred = async { slowlyAddNumbers(4, 4) }
    
        log("Waiting for the deferred value to be available")
        log("The first result: ${myFirstDeferred.await()}")
        log("The second result: ${mySecondDeferred.await()}")
    }
    ```

    - 실행 결과

        ```kotlin
        0 [main @coroutine#1] Starting the async computation  
        4 [main @coroutine#1] Waiting for the deferred value to be available  
        8 [main @coroutine#2] Waiting a bit before calculating 2 + 2  
        9 [main @coroutine#3] Waiting a bit before calculating 4 + 4  
        213 [main @coroutine#1] The first result: 4  
        415 [main @coroutine#1] The second result: 8 
        ```

- `async`로 병렬 실행 가능
- 각 `Deferred`는 비동기적으로 실행되며 `await()` 호출 시점까지 기다림
- `await()`를 사용해야 실제 결과를 사용할 수 있음

| 코루틴 빌더 | 사용 목적 | 반환값 | 특징 |
| --- | --- | --- | --- |
| `runBlocking` | 블로킹 환경에서 일시 중단 함수 호출 | `Unit` | 일반 함수(main)에서 코루틴 시작 |
| `launch` | 결과를 기다리지 않는 작업 | `Job` | 부수 효과 중심의 작업 (예: 로그, 저장) |
| `async` | 비동기 계산, 결과 필요 | `Deferred<T>` | `await()`로 결과 수신 |

---

## 14.7 어디서 코드를 실행할지 정하기: 디스패처

- 코루틴은 특정 스레드에 묶여 있지 않다.
  실행 중 일시 중단되었다가, 다시 **다른 스레드**에서 실행을 재개할 수도 있다.
  코루틴을 어떤 **스레드**에서 실행할지를 결정해주는 것이 바로 **디스패처(Dispatcher)이다.**

> 스레드 풀이란?
스레드 풀(Thread pool)은 여러 스레드를 미리 만들어두고, 작업이 생기면 스레드를 재활용해 실행하는 구조
>
> - 새 스레드를 만드는 비용이 크기 때문에 효율적
> - 코루틴도 이러한 스레드 풀에서 실행

### 디스패처 선택

- 코루틴은 기본적으로 **부모 코루틴의 디스패처를 상속**
- 하지만 직접 지정할 수도 있으며, 코루틴의 용도에 따라 적절한 디스패처를 선택하는 것이 중요

| 디스패처 | 스레드 수 | 용도 |
| --- | --- | --- |
| `Dispatchers.Default` | CPU 코어 수 | CPU 작업 (일반 계산, 무거운 연산 등) |
| `Dispatchers.IO` | 64개 + 코어 수 (최대 64개 병렬) | **IO 작업** (파일, DB, 네트워크 등) |
| `Dispatchers.Main` | 1 | UI 업데이트 (Android, JavaFX 등) |
| `Dispatchers.Unconfined` | 제한 없음 | 특수한 즉시 실행 상황 (일반적 사용 X) |
| `newSingleThreadContext("MyThread")` | 1 | 직접 지정한 단일 스레드에서 실행 |

### 코루틴 빌더에 디스패처 전달하기

```kotlin
fun main() = runBlocking {
	    log("Doing some work")

    launch(Dispatchers.Default) {
        log("Doing some background work")
    }
}
```

- 출력

    ```kotlin
    [main @coroutine#1] Doing some work - 시작
    [DefaultDispatcher-worker-1 @coroutine#2] Doing some backgroup work - 백그라운드 작업
    ```


### `withContext`로 디스패처 변경하기

- 특정 블록에서만 디스패처를 바꾸고 싶다면 `withContext()`를 사용한다.
- UI 작업에서 자주 사용된다.

```kotlin
launch(Dispatchers.Default) {
    val result = performBackgroundOperation()

    withContext(Dispatchers.Main) {
        updateUI(result)
    }
}
```

### 코루틴은 스레드 안전 문제를 자동 해결해주지 않음

- 코루틴이 순차적으로 실행되긴 하지만, **여러 코루틴이 동시에 공유 자원에 접근**하면 여전히 스레드 안전 문제가 발생한다.
- 안전한 예 (코루틴 1개만 접근)

    ```kotlin
    fun main() {
    	 runBlocking {
    		 launch(Dispatchers.Default) {
    			 var x = 0
    			 repeat(10_000) {
                x++
            }
            println(x) // 항상 10_000
        }
    	}      
    }
    ```

- 문제가 있는 예 (10,000개의 코루틴이 동시에 x 증가)

    ```kotlin
    fun main() {
    	runBlocking {
        var x = 0
        repeat(10_000) {
            launch(Dispatchers.Default) {
                x++
            }
        }
        delay(1000)
        println(x) // 10,000보다 작을 수 있음
      }
    }
    ```

- 해결 방법 1: `Mutex`로 잠금 처리

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
    	
    	delay(1000)
      println(x)
    }
    ```

- 해결 방법 2: `AtomicInteger` 같은 스레드 안전 자료구조 사용

---

## 14.8 코루틴은 코루틴 콘텍스트에 추가적인 정보를 담고 있다

- 앞에서 `launch(Dispatchers.Default)`나 `withContext(Dispatchers.IO)`처럼 디스패처를 인자로 넘겼지만, 실제로 이 인자의 타입은 `CoroutineDispatcher`가 아니라 `CoroutineContext` 이다.

> CoroutineContext란?
CoroutineContext는 코루틴이 실행될 환경 정보를 담고 있는 데이터 집합
>
>
> 여러 요소(디스패처, Job, 이름, 예외 핸들러 등)로 구성되며, 각각이 **"Context Element"이다.**
>

- 현재 코루틴의 콘텍스트 확인하기
    - 코루틴 내부에서는 `coroutineContext` 속성을 통해 자신의 콘텍스트를 확인할 수 있다.

        ```kotlin
        import kotlinx.coroutines.*
        import kotlin.coroutines.coroutineContext
        
        suspend fun introspect() {
            println(coroutineContext)
        }
        
        fun main() = runBlocking {
            introspect()
        }
        ```

    - 출력 예시 (환경에 따라 다를 수 있음)

        ```kotlin
        [CoroutineId(1), "coroutine#1":BlockingCoroutine{Active}@... , BlockingEventLoop@...]
        ```


### 콘텍스트 구성 변경하기

- 여러 개의 `CoroutineContext` 요소를 **`+` 연산자**로 조합할 수 있다.

    ```kotlin
    fun main() = runBlocking(Dispatchers.IO + CoroutineName("Coolroutine")) {
        println(coroutineContext)
    }
    ```

    - 출력 예시

        ```kotlin
        [CoroutineName(Coolroutine), CoroutineId(1), ... , Dispatchers.IO]
        ```


| 요소 | 값 |
| --- | --- |
| Dispatcher | Dispatchers.IO (기본 디스패처 대체) |
| CoroutineName | "Coolroutine" |
| Job | BlockingCoroutine |

→ `+` 연산자를 이용하면 기존 콘텍스트를 확장하거나 덮어쓸 수 있다.

### 디스패처도 CoroutineContext의 일부다

```kotlin
launch(Dispatchers.Default)
```

- 이 코드는 **Dispatcher**라는 요소 하나만 전달하는 것이고, 실제로는 **`CoroutineContext` 전체를 전달**하는 것과 동일하다.
- 따라서 아래와 같이 **이름과 디스패처**를 함께 지정할 수도 있다.

```kotlin
launch(Dispatchers.IO + CoroutineName("MyWorker"))
```

| 개념 | 설명 |
| --- | --- |
| `CoroutineContext` | 코루틴에 부가 정보를 담는 환경 객체 (스레드, 이름, 예외 처리 등) |
| 주요 요소 | Dispatcher, Job, Name, ExceptionHandler 등 |
| `coroutineContext` | 현재 코루틴의 Context에 접근할 수 있는 컴파일러 지원 프로퍼티 |
| `+` 연산자 | 여러 Context 요소를 조합해서 전달 가능 |
| Dispatcher도 Context의 일부 | `launch(Dispatchers.IO)`는 Dispatcher가 포함된 Context 전달임 |

---

## 요약

- 동시성은 여러 작업을 동시에 처리하는 것을 의미하며, 여러 작업의 여러 부분이 서로 번갈아 실행되는 방식으로 나타난다. 병렬성은 물리적으로 동시에 실행되면서 현대 멀티코어 시스템을 효과적으로 활용하는 것을 말 한다.
- 코루틴은 스레드 위에서 동시 실행을 위해 동작하는 경량 추상화다.
- 코틀린의 핵심 동시성 기본 요소는 일시 중단 함수로, 실행을 잠시 멈출 수 있는 함수다. 다른 일시 중단 함수나 코루틴 안에서 일시 중단 함수를 호출할 수 있다.
- 반응형 스트림, 끝백, 퓨처 같은 다른 접근 방식과 달리 일시 중단 합수를 쓸 때는 코드의 모양이 달라지지 않는다. 코드는 여전히 순차적으로 보인다.
- 코루틴은 일시 중단 가능한 계산의 인스턴스다.
- 코루틴은 스레드를 블로킹하는 문제를 피한다. 스레드 블로킹이 문제가 되는 이유는 스레드 생성에 비용이 많이 들고, 시스템 자원이 제한적이기 때문이다.
- 코루틴 빌더인 runBlocking, launch, async를 사용해 새로운 코루틴을 생성 할 수 있다.
- 디스패처는 코루틴이 실행될 스레드나 스레드 풀을 결정한다.
- 기본 제공되는 디스패처는 서로 다른 목적을 갖고 있다. Dispatchers. Default 는 일반적인 용도에 쓰이며, Dispatchers.Main은 UI 스레드에서 작업을 실 행할 때 사용되고, Dispatchers. IO는 블로킹되는 IO작업을 호출할 때 사용 된다.
- Dispatchers.Default나 Dispatchers.IO와 같은 대부분의 디스패처는 다중 스레드 디스패처이기 때문에 여러 코루틴이 병렬로 같은 데이터를 변경할 때 주의가 필요하다.
- 코루틴을 생성할 때 디스패처를 지정하거나 withContext를 사용해 디스패처를 변경할 수 있다.
- 코루틴 콘텍스트에는 코루틴과 연관된 추가 정보가 들어있다. 코루틴 디스 패처는 코루틴 콘텍스트의 일부다.


