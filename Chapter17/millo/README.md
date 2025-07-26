# 17장 플로우 연산자

**다루는 내용**

- 플로우를 변형하고 다루기 위한 연산자
- 중간 연산자와 최종 연산자
- 커스텀 플로우 연산자 만들기

## 17.1 플로우 연산자로 플로우 조작

- 시퀀스와 마찬가지로 플로우도 중간 연산자와 최종 연산자를 구분한다
- 중간 연산자는 코드를 실행하지 않고 변경된 플로우만 반환한다
- 최종 연산자는 계산된 값을 반환하거나 아무 값도 반환하지 않으면서 플로우를 수집하고 실제 코드를 실행한다

## 17.2 중간 연산자는 플로우에 적용되고 새로운 플로우를 반환한다

**플로우 2종류**

- 업스트림 플로우
    - 중간 연산자가 적용되는 플로우
- 다운스트림 플로우
    - 중간 연산자가 반환하는 플로우
    - 또 다른 연산자의 업스트림 플로우로 작용할 수 있다

**중간 연산자가 호출되더라도 플로우 코드가 실제로 실행되지는 않는다**

- 중간 연산자에 의해 반환된 다운스트림 플로우는 콜드 상태이다

**그렇다면 핫 플로우에 중간 연산자를 적용한다면?**

- 실제로 핫 플로우에 연산자를 적용해도 `collect`와 같은 최종 연산자가 호출돼 핫 플로우가 구독이 될 때까지 정의한 동작이 실행되지 않는다

### 17.2.1 transform 연산자는 원하는 원소들을 배출할 수 있다

- `map` 함수는 업스트림 플로우를 받아 원소를 변환한 후 다운스트림 플로우에 그 원소를 배출한다
- 그런데 만약 하나 이상의 원소를 배출하고 싶다면 `transform`을 사용하면 된다
- `transform`은 업스트림 플로우의 각 원소에 대해 원하는 만큼의 원소를 다운스트림 플로우에 배출할 수 있다

```kotlin
fun main() {
    val names = flowOf("Jo", "May", "Sam")
    val upperAndLowercaseNames = names.transform {
        emit(it.uppercase())
        emit(it.lowercase())
    }

    runBlocking {
        upperAndLowercaseNames.collect { print("$it ") }
    }
}

[출력]
JO jo MAY may SAM sam 
```

### 17.2.2 take 연산자는 플로우를 취소할 수 있다

- `take` 연산자는 지정한 조건이 더 이상 유효하지 않을 때 업스트림 플로우가 취소하고 더 이상 원소를 배출하지 않는다

```kotlin
fun main() {
    val temps = getTemperatures()
    runBlocking {
        temps
            .take(5)     // 5번만 배출 후에 플로우 취소
            .collect {
                log(it)
            }
    }
}

[출력]
27) [main] 21
541) [main] 1
1046) [main] 2
1552) [main] 8
2054) [main] 7
```

### 17.2.3 플로우의 각 단계를 후킹하는 연산자

**원소 수집 후 종료를 확인하는 `onCompletion` 중간 연산자**

```kotlin
fun main() {
    val temps = getTemperatures()
    runBlocking {
        temps
            .take(5)
            .onCompletion { cause ->
                if (cause != null) {
                    println("An error occurred! ${cause.message}")
                } else {
                    println("Completed successfully!")
                }
            }
            .collect { 
                log("$it ")
            }
    }
}
```

**플로우 생명주기의 특정 단계에서 쓰이는 다양한 중간 연산자**

| 연산자 | 실행 시점 | 주요 기능 |
| --- | --- | --- |
| **onStart** | 플로우 수집 시작 후 첫 배출 전 딱 한 번 실행 | 초기화 작업 수행 |
| **onEach** | 각 원소 배출 시마다 실행 | 원소별 작업 수행 후 다운스트림 전달 |
| **onEmpty** | 플로우가 원소 없이 종료 시에만 실행 | 빈 플로우 처리 또는 기본값 제공 |

```kotlin
fun process(flow: Flow<Int>) {
    runBlocking {
        flow
            .onEmpty {
                println("Nothing - emitting default value!")
                emit(0)
            }
            .onStart {
                println("Starting!")
            }
            .onEach {
                println("On $it!")
            }
            .onCompletion {
                println("Done!")
            }
            .collect()
    }
}

fun main() {
    runBlocking {
        process(flowOf(1, 2, 3))
        println("======================")
        process(flowOf())
    }
}

[출력]
Starting!
On 1!
On 2!
On 3!
Done!
======================
Starting!
Nothing - emitting default value!
On 0!
Done!
```

**여기서 만약 `onEmpty { }` 로직은 더 다운스트림으로(`colllect` 바로 앞) 내린다면?**

- 위처럼 기본값 출력 및 방출이 일어나지 않는다
- 이유는 `onEmpty`는 직전 업스트림이 비어있는지 확인하는 연산자인데 더 업스트림에 있는 `onEach`가 아무것도 반환하지 않는 새로운 플로우를 만들어서 반환하기 때문에다.
- `onEmpty` 입장에서는 `onEach`에 의해 생성된 플로우는 빈 플로우가 아니라고 판단하기 때문에 작동하지 않는다.
- 따라서 onEmpty를 쓰고 싶으면 반드시 원소를 배출할 가능성이 있는 지점에서 가장 가까운 위치에 둬야 한다

### 17.2.4 buffer 연산자는 다운 스트림 연산자와 수집자를 위해 원소 버퍼링을 제공할 수 있다

플로우 내에서 시간이 걸리는 임시 중단 함수를 호출하는 경우 값 생산자는 수집자가 이전 원소를 처리할 때까지 작업을 중단한다

- 원소가 배출되면 다운스트림 플로우가 해당 원소를 처리할 때까지 생산자 코드는 계속되지 않는다

```kotlin
fun getAllUserIds(): Flow<Int> {
    return flow {
        repeat(3) {
            delay(200.milliseconds) // DB 지연 시간
            log("Emitting!")
            emit(it)
        }
    }
}

suspend fun getProfileFromNetwork(id: Int): String {
    delay(2.seconds) // 네트워크 지연 시간
    return "Profile[$id]"
}

fun main() {
    val ids = getAllUserIds()
    runBlocking {
        ids
            .map { getProfileFromNetwork(it) }
            .collect { log("Got $it") }
    }
}

[출력]
236) [main] Emitting!
2244) [main] Got Profile[0]
2447) [main] Emitting!
4453) [main] Got Profile[1]
4658) [main] Emitting!
6663) [main] Got Profile[2]
```

- 방출되면 즉시 2초간 네트워크 작업하고 다시 방출하면 … 반복되는 모습

`buffer` 연산자는 버퍼를 추가해서 다운스트림 플로우가 이미 배출된 원소를 처리하느라 바쁜 동안에도 업스트림 플로우가 원소를 배출할 수 있게 해준다

- 즉, 수집자가 원소를 처리할 때까지 생산자가 기다리지 않고 원소를 생성한다
- 이렇게 되면 플로우에서 연산자 사슬의 연결을 분리하는 효과가 있어서 생산자는 계속 생산하면서 버퍼에 넣고 별개로 수집자는 시간이 걸리는 일시 중단 함수를 수행할 수 있다
    - 사슬의 연결을 분리한다? 업스트림 플로우의 실행을 다운 스트림 플로우로부터 분리하는 것

```kotlin
fun main() {
    val ids = getAllUserIds()
    runBlocking {
        ids
            .buffer(3)
            .map { getProfileFromNetwork(it) }
            .collect { log("Got $it") }
    }
}

[출력]
238) [main] Emitting!
445) [main] Emitting!
650) [main] Emitting!
2248) [main] Got Profile[0]
4255) [main] Got Profile[1]
6261) [main] Got Profile[2]
```

- `getAllUserIds`에서 반환된 플로우는 수집자가 작업하는 동안 원소를 버퍼에 배출할 수 있기에 더 빠르게 작업이 끝난 모습

**onBufferOverflow 파라미터로 버퍼 용량 초과 시 동작 방식을 결정할 수 있다**

| onBufferOverflow 값 | 동작 방식 | 설명 |
| --- | --- | --- |
| **SUSPEND** (기본값) | 일시 중단 | 버퍼가 가득 차면 새 원소 배출을 중단하고 대기 |
| **DROP_OLDEST** | 가장 오래된 원소 제거 | 버퍼가 가득 차면 가장 오래된 원소를 제거하고 새 원소 추가 |
| **DROP_LATEST** | 가장 최신 원소 제거 | 버퍼가 가득 차면 새로 들어오는 원소를 무시 |

### 17.2.5 conflate 연산자로 중간값을 버릴 수 있다

- `conflate` 연산자를 사용하면 수집자가 바쁜동안 배출된 원소를 그냥 버리는 것이 가능하다
    - 느린 수집자가 최신 배출된 원소만 처리하게 되기 때문에 성능을 유지할 수 있다
- `buffer`와 마찬가지로 업스트림 플로우의 실행을 다운스트림 연산자의 실행과 분리할 수 있다

```kotlin
fun main() {
    runBlocking {
        val temps = getTemperatures()
        temps
            .onEach {
                log("Read $it form sensor")
            }
            .conflate()
            .collect {
                log("Collected $it")
                delay(1.seconds)
            }
    }
}

[출력]
548) [main] Read 27 form sensor
551) [main] Collected 27
1056) [main] Read 25 form sensor
1552) [main] Collected 25
1557) [main] Read 22 form sensor
2063) [main] Read 26 form sensor
2555) [main] Collected 26
2565) [main] Read 20 form sensor
3557) [main] Collected 20
```

### 17.2.6 debounce 연산자로 일정 시간 동안 값을 필터링할 수 있다

- `debounce` 연산자는 업스트림에서 원소 배출을 멈춘 후 정해진 타임아웃 시간이 지나면 마지막 원소를 다운스트림으로 배출한다
- 타임아웃 시간 내에 새 원소가 들어오면 기존 대기 중인 원소는 취소되고 새로운 타이머가 시작된다
- 사용자가 타이핑을 멈춘 후 일정 시간 뒤 검색 실행하는 기능을 구현할 때 사용할 수 있다

```kotlin
val searchQuery = flow {
    emit("K"); delay(100.milliseconds)
    emit("Ko"); delay(200.milliseconds)
    emit("Kotl"); delay(500.milliseconds)
    emit("Kotlin"); 
}

@OptIn(FlowPreview::class)
fun main() {
    runBlocking {
        searchQuery
            .debounce(250.milliseconds)
            .collect {
                log("Search query: $it")
            }
    }
}

[출력]
597) [main] Search query: Kotl
854) [main] Search query: Kotlin
```

### 17.2.7 flowOn 연산자로 플로우가 실행되는 코루틴 컨텍스트를 바꿀 수 있다

- 플로우도 일반 코루틴과 마찬가지로 블로킹 I/O를 사용하거나 UI 스레드에서 작업할 때 컨텍스트를 고려해야 한다
    - 코루틴 컨텍스트가 플로우 로직이 실행되는 위치를 결정하기 때문이다.
- 기본적으로는 `collect`가 호출된 컨텍스트에서 실행되지만 `flowOn` 연산자를 사용해 컨텍스트를 바꿀 수 있다
    - 마치 `withContext`와 비슷하게 코루틴 컨텍스트를 조정한다

```kotlin
fun main() {
    runBlocking {
        flowOf(1)
            .onEach { log("A") }
            .flowOn(Dispatchers.Default)
            .onEach { log("B") }
            .flowOn(Dispatchers.IO)
            .onEach { log("C") }
            .collect()
    }
}

[출력]
39) [DefaultDispatcher-worker-3] A
47) [DefaultDispatcher-worker-1] B
47) [main] C
```

- 출력을 보면 `flowOn` 연산자는 업스트림 플로우의 디스패처에 영향을 미치는 걸 볼 수 있다
- 그래서 `flowOn` 호출보다 더 앞에 있는 플로우가 영향을 받기 때문에 “C”는 전혀 영향이 없었던 것이다

## 17.3 커스텀 중간 연산자 만들기

**예시) Double 원소로 이뤄진 플로우에서 마지막 n개 원소의 평균을 계산하는 커스텀 연산자**

```kotlin
fun Flow<Double>.averageOfLast(n: Int): Flow<Double> =
    flow {
        val numbers = mutableListOf<Double>()
        collect {
            if (numbers.size >= n) {
                numbers.removeFirst()
            }
            numbers.add(it)
            emit(numbers.average())
        }
    }

fun main() = runBlocking {
    flowOf(1.0, 2.0, 30.0, 121.0)
        .averageOfLast(3)
        .collect {
            print("$it ")
        }
}

[출력]
1.0 1.5 11.0 51.0 
```

## 17.4 최종 연산자는 업스트림 플로우를 실행하고 값을 계산한다

중간 연산자는 주어진 플로우를 다른 플로우로 변환하기만 하고 실제 실행은 최종 연산자가 담당한다

**최종 연산자 `collect`**

- `collect` 호출 시 각 원소에 대해 실행할 람다를 지정할 수 있다.
- `onEach`로 각 원소에 실행할 코드를 미리 호출한 다음 파라미터 없는 `collect`를 호출하는 것과 동일하다
- 최종 연산자는 업스트림 플로우의 실행을 담당하기 때문에 항상 일시 중단 함수이다

**최종 연산자 first, firstOrNull**

- 원소를 받은 다음에 업스트림 플로우를 취소할 수 있는 최종 연산자