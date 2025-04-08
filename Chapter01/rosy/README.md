# 1. 코틀린이란 무엇이며, 왜 필요한가?
코틀린은 자바 가상머신 플랫폼과 JVM 외의 다른 플랫폼에서 돌아가는 현대 프로그래밍 언어다.

 
## 1.1 코틀린 맛보기

- data class를 통해 기본 생성자, toString 등 메서드를 자동으로 생성할 수 있다.
- 변수 선언 시 val과 var을 구분하여 불변성과 가변성을 명확히 표현한다.
- 함수 호출 시 이름 붙은 인자(named arguments)를 사용할 수 있다.
- nullable 타입(Int?)과 엘비스 연산자(?:)를 통해 NullPointerException을 예방할 수 있다.
- 람다식과 컬렉션 함수(maxBy 등)를 활용해 간결한 데이터 처리 가능하다.
- 문자열 템플릿을 사용하면 가독성 있는 출력이 가능하다.

```kotlin
data class Person (
    val name: String,
    val age: Int? = null
)
fun main() {
   val persons = listOf(
   		Person("철수", 10),
       	Person("영희")
   )
   
   val oldest = persons.maxBy {
       it.age ?: 0; //null인 경우 0으로 대체
   }
    println("가장 나이가 많은 사람은? -> $oldest") 
    // 가장 나이가 많은 사람은? -> Person(name=철수, age=10)
}
```

---

## 1.2 코틀린의 주요 특성

- 코틀린은 **정적 타입 언어**로, 컴파일 시점에 오류를 방지할 수 있다.
- **타입 추론**을 지원해, 타입 안정성과 코드 간결성을 동시에 추구할 수 있다.
- **객체지향과 함수형 스타일을 함께 지원**한다.
    - 함수는 변수처럼 다룰 수 있고, 람다식 문법이 간결하다.
    - 불변성과 순수 함수 중심의 코드 작성이 가능하다.
- **컬렉션 처리에 특화된 표준 API**가 많아 선언형 코드 작성에 유리하다.
    - `filter`, `map`, `sortedBy` 등을 자주 사용하게 된다.
- **코루틴을 사용한 비동기 처리**가 가능하다.
    - `suspend` 함수로 중단 가능한 비동기 로직을 순차적으로 표현할 수 있다.
    - 구조화된 동시성을 지원해 코루틴 생명주기 관리가 쉬움
- 자바와의 **호환성**이 높아 기존 자바 프로젝트에도 쉽게 도입할 수 있다.
- JVM 기반 외에도 JS, Native 등 다양한 플랫폼을 지원한다.
- **오픈소스 언어**이며, 누구나 참여할 수 있다.

---

## 1.3 코틀린이 자주 쓰이는 분야

- **서버 개발**: 스프링과의 호환성 우수, Ktor 같은 경량 프레임워크 존재
- **안드로이드**: 공식 지원 언어, Jetpack Compose 등 최신 도구는 Kotlin 기반
- **다중 플랫폼** : 공통 코드(`expect/actual`)로 JVM/JS/iOS 등에서 로직 공유 가능

---

## 1.4 코틀린의 철학

- **실용적**: 자바 경험자에게 익숙하고 도구 지원 좋음 (IDE 최적화)
- **간결함**: 타입 추론, data class, string template 등으로 코드 양 감소
- **안전함**: nullable 타입, `?.`, `?:`, 스마트 캐스트로 런타임 오류 줄임
- **상호운용성**: 자바와 자유롭게 혼용 가능, 변환기도 제공

```kotlin
fun handle(value: Any) {
    if (value is String) println(value.uppercase())  // 스마트 캐스트
}
```

---

## 1.5 코틀린 도구 사용

- **Kotlin Playground**
    - 설치 없이 실습 가능 ([https://play.kotlinlang.org](https://play.kotlinlang.org/))
- **IDE 통합**
    - IntelliJ, Android Studio에 기본 포함
    - 자동완성, 리팩토링, 디버깅 모두 지원
- **자바 → 코틀린 변환기**
    - 자바 코드 붙여넣으면 자동 코틀린 변환됨
- **컴파일 대상**
    - JVM, JS, Native, WASM

```
# 컴파일 예시
kotlinc Main.kt -include-runtime -d Main.jar
java -jar Main.jar
```


## 🟢 흥미로웠던 포인트

- **자바 → 코틀린 자동 변환기 제공**   
  → 기존 자바 코드를 코틀린 코드로 쉽게 변환할 수 있어 **자바를 알고 있다면 진입 장벽이 낮음**

- **코틀린과 자바를 같은 프로젝트에서 혼용 가능**  
  → 점진적으로 코틀린을 도입할 수 있어 **기존 자바 프로젝트와의 통합이 유리**

- **보일러플레이트 코드 생략 가능**  
  → `data class`로 **기본 생성자**, `toString()`, `equals()`, `hashCode()` 자동 생성
  → `val`, `var`를 이용한 **간단한 프로퍼티 선언**으로 기본 게터/세터 제공

