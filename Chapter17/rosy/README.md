# 17장 플로우 연산자

## 17.1 플로우 연산자로 플로우 조작

- Kotlin의 **Flow**는 컬렉션(`List`, `Set`)이나 시퀀스(`Sequence`)처럼 **다양한 연산자(operator)**를 제공한다.
- 연산자를 통해 값을 **변환**, **필터링**, **누적**, **수집**할 수 있다.

| 구분 | 설명 |
| --- | --- |
| 중간 연산자 | 연산 결과로 **새로운 플로우**를 반환 (지연 실행됨) |
| 최종 연산자 | **플로우를 수집하며 실행**됨. 값을 반환하거나 side effect 발생 |
- 중간과 최종 연산자 비교

    ```kotlin
    val flow = flowOf(1, 2, 3) // 콜드 플로우 생성
    
    val mappedFlow = flow.map { it * 2 } // 중간 연산자: 실행되지 않음
    
    runBlocking {
        mappedFlow.collect { println(it) } // 최종 연산자: 여기서 실행됨
    }
    ```

    - 결과

        ```kotlin
        2
        4
        6
        ```

- `map`, `filter`, `take` 등은 **중간 연산자**
- `collect`, `toList`, `first` 등은 **최종 연산자**
- **중간 연산자만 사용하면 실행되지 않음**
- **최종 연산자가 호출될 때** Flow가 실행됨

---

## 17.2 중간 연산자는 업스트림 플로우에 적용되고 다운 스트림 플로우를 반환한다.

| 용어 | 의미 |
| --- | --- |
| 업스트림(Upstream) | 현재 연산자가 **입력받는 플로우** |
| 다운스트림(Downstream) | 현재 연산자가 **반환하는 플로우** |
- 중간 연산자는 항상
    - **업스트림 플로우**에서 값을 받음
    - **가공 후 다운스트림 플로우**로 넘김
- 여러 중간 연산자 연결

    ```kotlin
    val original = flowOf(1, 2, 3, 4, 5)
    
    val result = original
        .filter { it % 2 == 1 }    // 업스트림: original → 다운스트림: 홀수만
        .map { it * 10 }           // 업스트림: 홀수만 → 다운스트림: 10배
    
    ```

    - → 이 시점에서는 **아직 아무것도 실행되지 않음**

        ```kotlin
        runBlocking {
            result.collect { println(it) } // collect를 호출해야 실행됨
        }
        ```

    - 결과

        ```kotlin
        10
        30
        50
        ```

- 플로우는 **콜드(cold)** 하기 때문에 **최종 연산자**가 실행되기 전까지는 아무 작업도 하지 않음.
- 즉, `flow { ... }` 안의 코드나 `map`, `filter`는 **collect를 만나야만 실행됨**.

### 업스트림 원소별로 임의의 값을 배출: transform 함수

- `map`은 1개의 입력 → 1개의 출력으로 단순 변환한다.
- 하지만 여러 개의 출력을 만들고 싶다면? `transform`을 사용해야 합니다.
    - `map` - 각 요소를 다른 형태로 변환하는 데 사용된다.

        ```kotlin
        import kotlinx.coroutines.*
        import kotlinx.coroutines.flow.*
        
        fun main() {
        	val names = flow {
        		emit("Jo")
        		emit("May")
        		emit("Sue")
        	}
        	
        	val uppercasedNames = names.map {
        		it.uppercase()
        	}
        	runBlocking {
        		uppercasedNames.collect{ print("$it") }
        	}
        	
        	// JO MAY SUE
        }
        ```

        - `map`은 일반 컬렉션에서 사용되는 것과 똑같이 동작한다.
        - 각각의 요소에 함수를 적용한 결과를 다시 플로우로 만든다.
    - `transform` – 더 복잡한 변환을 할 수 있다.

        ```kotlin
        fun main() {
        	val names = flow {
        		emit("Jo")
        		emit("May")
        		emit("Sue")
        	}
        	
        	val upperAndLowercasedNames = names.map {
        		emit(it.uppercase())
        		emit(it.lowercase())
        	}
        	runBlocking {
        		upperAndLowercasedNames.collect{ print("$it") }
        	}
        	
        	// JO jo MAY may SUE sue
        }
        ```

        - `transform`은 단순히 하나의 값을 하나로 매핑하는 것이 아니라, 하나의 값을 여러 개로 바꾸거나 조건을 걸어 선택적으로 배출하는 등 더 유연한 처리가 가능하다.

### `take`나 관련 연산자는 플로우를 취소할 수 있다

- `take(n)`은 n개의 값만 받고, 그 이후의 플로우는 **자동으로 취소**됩니다.

    ```kotlin
    import kotlinx.coroutines.flow.*
    
    //getTemperatures 함수는 16장에서
    
    fun main() {
    	val temps = getTemperatures()
    	temps
    		.take(5)
    		.collect{
    			log(it)
    		}
    }
    ```

- 결과

    ```kotlin
    37 [main @coroutine#1] 7
    568 [main @coroutine#1] 9
    1123 [main @coroutine#1] 2
    1640 [main @coroutine#1] -6
    2148 [main @coroutine#1] 7
    ```

- `take(n)`은 n개의 값만 받고, 그 뒤 플로우를 자동 취소
- 내부적으로 `collect`를 취소하는 것이 아닌, **업스트림 전체를 취소**

### 플로우의 각 단계 후킹

| 연산자 | 실행 시점 |
| --- | --- |
| `onStart` | 첫 값 배출 전에 실행됨 |
| `onEach` | 각 값 배출 직후 실행됨 |
| `onCompletion` | 정상 종료/예외 발생 후 실행됨 |
| `onEmpty` | 아무 값도 배출되지 않을 경우 실행됨 |
- `onCompletion` 으로 완료 여부 확인
    - 플로우가 정상 종료되거나, 취소되거나, 예외로 종료된 후 호출되는 람다를 지정할 수 있게 해준다.

    ```kotlin
    fun main() = runBlocking {
    	val = temps = getTemperatures()
    	temps
    		.take(5)
    		.onCompletion { cause ->
    			if (cause != null) {
    				println("An error occurred! $cause")
    			} else {
    				println("Completed!")
    			}
    		}
    		.collect {
    			println(it)
    		}
    }
    ```

    - 플로우 생명주기의 특정 단계에서 작업을 수행할 수 있는 중간 연산자에 속한다.
- 전체 생명주기 훅킹

    ```kotlin
    fun main() {
    	//flow 변수 정의는 생략
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
    ```

    ```kotlin
    fun main() {
    	runBlocking {
    		process(flowOf(1, 2, 3))
    		// Starting!
    		// On 1!
    		// On 2!
    		// On 3!
    		// Done!
    		process(flowOf())
    		// Starting!
    		// Nothing - emitting default value!
    		// On 0!
    		// Done!
    	}
    }
    ```

    - `onEmpty`는 **업스트림에서 아무 것도 emit되지 않았을 때**만 실행됨
    - `onEach`는 `onEmpty`에서 emit한 값을 받을 수 **없다** (왜냐하면 `onEmpty`는 더 아래 단계이기 때문)

### 다운스트림 연산자와 수집자를 위한 원소 버퍼링: buffer 연산자

- 실제 애플리케이션에서는 다음과 같은 작업이 **플로우 내에서 자주 발생**한다
    - `collect`, `onEach` 같은 연산자 내부에서 **시간이 오래 걸리는 일시 중단 함수 호출**
    - 예를 들어, 데이터베이스 접근, 네트워크 호출 등이 대표적
- 그럴 경우, **값을 방출하는 생산자**는 **이전 원소 처리가 끝날 때까지 대기**하게 된다. 이를 해결하기 위해 **`buffer` 연산자**를 사용한다.
- 예제 1: 버퍼 없이 작동하는 플로우 (느린 처리)

    ```kotlin
    fun getAllUserIds(): Flow<Int> = flow {
        repeat(3) {
            delay(200.milliseconds) // DB 접근 대기 시간
            log("Emitting!")
            emit(it)
        }
    }
    
    suspend fun getProfileFromNetwork(id: Int): String {
        delay(2.seconds) // 네트워크 지연
        return "Profile[$id]"
    }
    
    fun main() = runBlocking {
    		val ids = getAllUserIds();
    		runBlocking {
    			ids
    				.map { getProfileFromNetwork(it) }
            .collect { log("Got $it") }
    		}
    }
    ```

    - 결과

        ```kotlin
        310 [main] Emitting!
        2402 [main] Got Profile[0]
        
        2661 [main] Emitting!
        4732 [main] Got Profile[1]
        
        5007 [main] Emitting!
        7048 [main] Got Profile[2]
        ```

    - `getAllUserIds()`는 200ms마다 ID를 방출
    - `getProfileFromNetwork()`는 각 ID당 2초씩 걸림
    - 따라서 **다음 ID 방출은 이전 프로필이 처리될 때까지 대기**
    - 결과적으로 **전체 실행 시간은 약 7초**
    - 이 방식은 **생산자(emit)와 소비자(collect)가 강하게 연결**되어 있어서 비효율적
- 예제 2:  `buffer(3)` 추가 (효율적 처리)

    ```kotlin
    fun main() = runBlocking {
        val ids = getAllUserIds();
    		runBlocking {
    			ids
            .buffer(3)
            .map { getProfileFromNetwork(it) }
            .collect { log("Got $it") }
        }
    }
    ```

    - 결과

        ```kotlin
        304 [main] Emitting!
        525 [main] Emitting!
        796 [main] Emitting!
        
        2373 [main] Got Profile[0]
        4388 [main] Got Profile[1]
        6461 [main] Got Profile[2]
        ```

    - `buffer(3)`로 인해 최대 3개의 값을 **버퍼에 미리 저장**
    - 생산자는 0, 1, 2번 ID를 **연속으로 빠르게 emit**
    - 소비자는 그 후 **각각의 네트워크 작업을 순차적으로 처리**
    - 전체 실행 시간이 **약 2초 이상 단축됨** (7초 → 6초 이하)
- `onBufferOverflow`
    - 파라리터를 통해 버퍼 용량이 초과될 때 어떤 일이 발생할지 지정할 수 있다.
    - `buffer()`는 **버퍼 초과 시 행동**도 설정 가능:
        - `BufferOverflow.SUSPEND`: (기본값) 버퍼가 가득 차면 emit을 일시 중단
        - `BufferOverflow.DROP_OLDEST`: 오래된 값 제거
        - `BufferOverflow.DROP_LATEST`: 새 값 무시

        ```kotlin
        buffer(capacity = 3, onBufferOverflow = BufferOverflow.DROP_OLDEST)
        ```


### 중간값을 버리는 연산자: conflate 연산자

- 다운스트림(수집자)이 느릴 때, 최신 값만 처리하고 중간값은 버린다.

    ```kotlin
    import kotlinx.coroutine.flow.*
    
    fun main() {
    	runBlocking {
    		val temps = getTemperatures()
        temps 
            .onEach { log("Read $it from sensor") }
            .conflate()
            .collect {
                log("Collected $it")
                delay(1000) // 느린 수집자
    	  }      
      }
    }
    
    ```

    - 결과

        ```kotlin
        43 [main @coroutine#2] Read 20 from sensor
        51 [main @coroutine#1] Collected 20
        558 [main @coroutine#2] Read -10 from sensor
        1078 [main @coroutine#2] Read 3 from sensor
        1294 [main @coroutine#1] Collected 3
        1579 [main @coroutine#2] Read 13 from sensor
        2153 [main @coroutine#2] Read 26 from sensor
        2556 [main @coroutine#1] Collected 26
        
        ```

    - `conflate()`는 수집자가 느릴 경우 중간 값을 건너뛰고 최신 값만 전달한다.
    - sensor는 지속적으로 데이터를 emit하지만, 수집자는 1초 delay로 느리다.
    - 중간에 있던 값들(`10`, `13`)은 건너뛰고 최신값만 수집된다.
    - 최신 상태가 중요할 때(예: UI 갱신) 유용한다.

### **일정 시간 동안 값을 필터링하는 연산자: `debounce`**

- 업스트림에서 원소가 배출되지 않은 상태로 정해진 타임아웃 시간이 지나야만 항목을 다운 스트림 플로우로 배출한다.

    ```kotlin
    val searchQuery = flow {
        emit("K")
        delay(100.milliseconds)
        emit("Ko")
        delay(200.milliseconds)
        emit("Kotl")
        delay(500.milliseconds)
        emit("Kotlin")
    }
    
    fun main() = runBlocking {
        searchQuery
            .debounce(250)
            .collect { log("Searching for $it") }
    }
    ```

    - 결과

        ```kotlin
        644 [main] Searching for Kotl
        1876 [main] Searching for Kotlin
        ```

    - `debounce(250)`은 입력이 일정 시간 멈춘 경우에만 최신 값을 emit한다.
    - 사용자 입력이 빠르게 발생하면 그 사이 값들은 무시되고 마지막 안정된 값만 처리된다.
    - 검색창 자동완성처럼, "입력 완료 후 검색 시작"에 적합하다.

### 플로우가 실행되는 코루틴 콘텍스트를 바꾸기: `flowOn`

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
```

- 결과

    ```kotlin
    36 [DefaultDispatcher-worker-3] A
    44 [DefaultDispatcher-worker-1] B
    44 [main] C
    ```

- `flowOn`은 해당 시점 “이전의” 연산자들에만 영향을 준다.
- `flowOn(Dispatchers.Default)`는 `onEach { A }`만 Default 디스패처에서 실행.
- 이후의 `onEach { B }`, `onEach { C }`는 다시 IO → Main 디스패처 순으로 실행된다.
- `flowOn`은 **업스트림에만 영향을 주고 다운스트림은 영향을 받지 않는다.**

---

## 17.3 커스텀 중간 연산자 만들기

- **중간 연산자는 생산자 + 소비자 역할**
    - 업스트림에서 원소를 받아와서 그걸 가공하거나 변형해서 다운스트림에 새로운 원소를 전달하는 역할을 한다.
    - 대표적인 예: `map`, `filter`, `onEach`, `transform` 등
- 직접 만들기: 평균을 계산하는 연산자 만들기
    - 최근 `n`개의 숫자를 기억해서 평균을 내는 연산자를 만들어보자.

    ```kotlin
    fun Flow<Double>.averageOfLast(n: Int): Flow<Double> = flow {
        val numbers = mutableListOf<Double>()
        collect { value ->
            if (numbers.size >= n) {
                numbers.removeFirst() // 오래된 값 제거
            }
            numbers.add(value)
            emit(numbers.average()) // 현재까지의 평균 배출
        }
    }
    ```

    - `flow { ... }` 내부에서 `collect { ... }`로 **업스트림 값 수집**
    - `emit(...)`으로 **다운스트림에 평균 값 전달**

    ```kotlin
    fun main() = runBlocking {
        flowOf(1.0, 2.0, 30.0, 121.0)
            .averageOfLast(3)
            .collect { print("$it ") }
    }
    // 출력: 1.0 1.5 11.0 51.0
    ```

    - 1개 → 평균 1.0
    - 1,2 → 평균 1.5
    - 1,2,30 → 평균 11.0
    - 2,30,121 → 평균 51.0
- `collect { ... }` → 업스트림 수집
- `emit(...)` → 다운스트림 전달
- 연산자 안에서도 **콜드 플로우** 특성 유지됨 (실제 실행은 `collect()`로 트리거됨)

---

## 17.4 최종 연산자는 업스트림 플로우를 실행하고 값을 계산한다

- 중간 연산자는 코드 실행 X, 최종 연산자가 실행을 트리거
    - `map`, `filter` 같은 중간 연산자들은 **연산만 정의**할 뿐 실행은 하지 않는다.
    - **`collect`, `first`, `toList`, `count`** 등의 **최종 연산자**가 호출되어야 실행이 시작된다.
- 대표적인 최종 연산자: `collect`

    ```kotlin
    fun main() = runBlocking {
        getTemperatures()
            .onEach { println(it) }
            .collect()
    }
    ```

    - `onEach`는 중간 연산자라 실행 안 됨
    - `collect()`가 실제로 실행을 발생시킴
- 다른 최종 연산자들
    - `first()`: 첫 값만 받고 종료
    - `firstOrNull()`: 값 없으면 null 반환
    - `toList()`: 전체 값을 리스트로 모음
- 모든 최종 연산자는 **일시 중단 함수(suspend)**
    - 플로우를 실행하려면 항상 **일시 중단 함수**를 사용해야 함.
    - 왜냐면, 내부적으로 `collect()`가 코루틴에서 실행되어야 하기 때문!

### 프레임워크는 커스텀 연산자를 제공한다

- Jetpack Compose 예시
    - Jetpack Compose 같은 프레임워크는 플로우를 UI 상태에 연결하기 위해 **커스텀 연산자**를 제공한다.

        ```kotlin
        @Composable
        fun TemperatureDisplay(temps: Flow<Int>) {
            val temperature = temps.collectAsState(null)
        
            Box {
                temperature.value?.let {
                    Text("The current temperature is $it!")
                }
            }
        }
        ```

    - `collectAsState(null)`는 Flow를 Compose의 상태(state)로 변환해줌
    - 값이 null이면 표시하지 않고, 값이 생기면 UI 갱신!

---

## 요약

- 중간 연산자는 플로우를 다른 플로우로 변환한다. 중간 연산자는 업스트림 플로우에 대해 작동하며 다운스트림 플로우를 반환한다. 중간 연산자는 콜드 상태이며 최종 연산자가 호출될 때까지 실행되지 않는다.
- 시퀀스에 사용할 수 있는 중간 연산자 상당수를 플로우에도 직접 사용할 수 있다. 플로우에는 변환을 수행하거나(transfom), 플로우가 실행되는 콘텍스트를 관리하거나(flowOn), 특정 단계에서 코드를 실행(onStart, onCompletion 등)하는 다른 중간 연산자도 추가 제공한다.
- collect와 같은 최종 연산자는 플로우의 코드를 실행한다. 핫 플로우의 경우 collect는 플로우에 대한 구독을 처리한다.
- 플로우 빌더 안에서 플로우를 수집하고 변환된 원소를 배출하는 방식으로 자신만의 중간 연산자를 만들 수 있다.
- 젯팩 컴포즈나 컴포즈 다중 플랫폼 같은 일부 외부 프레임워크는 코틀린 플로우와의 직접적인 통합을 제공한다.