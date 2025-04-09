# 2챕터 질문 정리

## 질문 정리

---

1. 자바처럼 스프링 애플리케이션 파일을 만들어보니 실행버튼이 뜨질 않습니다. 아래처럼 작성해야 실행버튼이 생기는데 그 이유가 뭘까요?
    - 질문 관련 코드

        ```kotlin
        @SpringBootApplication
        class KopringApplication {
            fun main(args: Array<String>) {
              runApplication<KopringApplication>(*args)
            }
        }
        
        @SpringBootApplication
        class KopringApplication
          
        fun main(args: Array<String>) {
            runApplication<KopringApplication>(*args)
        }
        ```

    - 내 답변
        - IDE에서 클래스 내부의 **`main`** 메서드가 아닌, 최상위 레벨의 **`main`** 함수를 진입점으로 인식하여 실행 버튼을 표시하기 때문인 것 같다.
2. 코틀린은 기본 가시성이 `public`이라고 합니다.
    - 질문 관련 코드

        ```kotlin
        class Person(
          val name: String,   
          var isStudent: Boolean
        )
        ```

    - 답변
        - 코틀린에서는 프로퍼티를 언어기능으로 제공하기 때문에 필요한 접근자를 제공하기 때문에 캡슐화 측면에서 기본적으로 `private 필드`로 만들어주는 것 같다.
            - 하지만 `private 프로퍼티`는 아니다.
            - 접근자가 존재하기때문
        - 프로퍼티가 private할려면 `class Person(private val name: String)`
3. 커스텀 Getter와 멤버함수 중 많이 사용되는 방식은?
    - 답변

      상태를 표현하기 위해서라면 커스텀 접근자, 행위를 표현하려면 멤버함수로 사용할 것 같다.

4. data class는 간편하게 사용이 가능해보이는데 어떤 경우에는 사용하지 않는 게 더 좋을까요? 기준이 있을까요?
    - 답변
        - 사용하지 않는게 더 좋아보이는 기준
            - 오브젝트의 불변성이 필요하지 않는 경우(상태 변경 로직이 많은 경우)
            - 오브젝트의 정체성이 식별성이 중요한 엔티티인 경우
            - 비용이 큰 경우: copy, 해시코드 계산이 비싼경우?
5. Kotlin 에서의 Int 와 Java의 원시타입 int 의 차이점은?
    - 답변
        - 객체 타입과 원시타입의 차이성
6. 코틀린에서의 for loop가 자바와 다른 점 두가지 설명해주세요.
    - 답변
        - 루프 문법과 형식
        - 컬렉션에 대한 구조분해 순회 지원
7. 책에서 나온 예시를 보면, `print("Kotlin" in setOf("Java", "Scala")`의 결과는 `false`가 나왔다.그렇다면, 아래의 결과는 어떻게 될까요?
    - 질문 관련 코드

        ```kotlin
        fun main() {
          val map = mapOf(
            "Java" to "자바",
            "Kotlin" to "코틀린",
            "Python" to "파이썬",
            "Cpp" to "씨쁠쁠",
          )
        
          when ("Java" to "자바") {
            in map -> println("in map")
            else -> throw IllegalArgumentException("not in map")
          }
        }
        
        // 참고로 for-each 구문을 사용하면 아래와 같이 사용할 수 있다.
        for ((key, value) in map) {
            println("$key = $value")
        }
        ```

    - 답변
        - `"Java" to "자바"` 는 Pair<String, String> 타입이므로 컴파일 에러가 난다.
        - in map은 기본적으로 `map.contains`로 변환되기 때문에 key 검사만 가능
        - key-value entry로 검사하려면 `map.entries.contains("Java" to "자바")`
8. 추가) **Kotlin `data class` vs Java `record class`**
    - 답변
        - record 클래스는 상속 불가능
            - 추상클래스의 구현이기때문에
        - record 클래스 copy() 메서드가 없음, 구조분해 불가능
9. 추가) 코틀린의 static
    - 답변
        - 좀 더 자세히 예정
