# 13. DSL 만들기

도메인 특화 언어 (Domain Specific Language, DSL)는 특정 도메인에 특화된 언어로, 해당 도메인에서의 문제 해결을 위해 설계된다. <br /> 
DSL은 일반적으로 특정 작업을 더 쉽게 수행할 수 있도록 도와주며, 코드의 가독성을 높이고 유지보수를 용이하게 한다.

**코틀린 DSL의 2가지 특성**
- 수신 객체 지정 : 수신 객체를 지정하여 해당 객체의 메서드와 속성에 접근할 수 있다. 이를 통해 코드의 가독성을 높이고, 특정 객체에 대한 작업을 더 직관적으로 표현할 수 있다.
- invoke 관례 : invoke 관례를 사용하면 DSL 코드 안에서 람다와 프로퍼티 대입을 더 유연하게 조합할 수 있다.

### 표현력이 좋은 커스텀 코드 구조 만들기

DSL을 사용하는 궁극적인 목표는 코드의 가동성과 유지 보수성을 좋게 만드는 것이다.<br />
이 목표를 달성하기 위해서는 클래스 간의 상호작용이 일어나는 API가 깔끔해야 한다. <br />

**API가 깔끔하는건 어떤 뜻일까?**
- 코드를 읽는 독자가 어떤 일이 벌어질지 명확하게 이해할 수 있어야 한다. (이름과 개념을 잘 선택해야함.)
- 코드에 불필요한 구문이나 번잡한 준비 코드가 가능한 한 적어야 한다.

<코틀린이 간결한 구문을 지원하는 방법>

| 일반 구문                                                                           | 간결한 구문                                                     | 사용한 언어 특성        |
|---------------------------------------------------------------------------------|------------------------------------------------------------|------------------|
| StringUtil.capitalize(s)                                                        | s.capitalize()                                             | 확장 함수            |
| 1.to("one")                                                                     | 1 to "one"                                                 | 중위 호출            |
| set.add(2)                                                                      | set += 2                                                   | 연산자 오버로딩         |
| map.get("key")                                                                  | map["key"]                                                 | get 메서드에 대한 관례   |
| file.use({ f -> f.read()})                                                      | file.use {it.read() }                                      | 람다를 괄호 밖으로 빼는 관례 |
| sb.append("yes")<br /> sb.append("no")                                          | with (sb) { <br />append("yes")<br /> append("no") <br />} | 수신 객체 지정 람다      |
| val m = mutableListOf<Int>()<br />m.add(1)<br />m.add(2)<br />return m.toList() | return buildList {<br/>add(1)<br/>add(2)<br/>}             | 람다를 받는 빌더 함수     |


코틀린 언어 의 다른 특성과 마찬가지로 코틀린 DSL도 컴파일 시점에 타입이 정해진다.<br/>
따라서, 컴파일 시점 오류 감지, IDE 지원 등 모든 정적 타입 지정 언어의 장점을 가진다.

### 도메인 특화 언어 (Domain Specific Language, DSL)

**DSL 정의**
- 특정 도메인이나 문제 영역에 특화된 언어
- 범용 프로그래밍 언어와 달리 특정 작업에 최적화

**DSL 종류**
- **외부 DSL**: 별도 파서가 필요하다. (SQL, 정규식)
- **내부 DSL**: 호스트 언어 문법 활용 (코틀린 DSL)

**DSL 장점**
- 도메인 전문가도 읽고 쓸 수 있음
- 코드 가독성 향상
- 반복 코드 제거

**코틀린 DSL 특징**
- 타입 안전성 보장
- IDE 지원 (자동완성, 리팩토링)
- 컴파일 타임 검증

### DSL의 구조

**DSL 구조 특징**
- **중첩 구조**: 계층적 데이터 표현에 적합
- **메서드 호출 체이닝**: 순차적 작업 표현

**중첩 구조 DSL**
```kotlin
// DSL
dependencies {
    testImplementation(kotlin("test"))
    implementation("org.jetbrains.exposed:exposed-core:0.40.1")
    implementation("org.jetbrains.exposed:exposed-dao:0.40.1")
}

// 명령-질의 API
project.dependencies.add("testImplementation", kotlin("test"))
project.dependencies.add("implementation", "org.jetbrains.exposed:exposed-core:0.40.1")
project.dependencies.add("implementation", "org.jetbrains.exposed:exposed-dao:0.40.1")
```
- 블록 내부에서 특정 메서드들만 사용 가능
- 컨텍스트에 따라 사용 가능한 연산 제한

**메서드 체이닝 DSL** (중위 함수 활용)
```kotlin
// DSL
str should startWith("kot")

// 명령-질의 API
assertTrue(str.startsWith("kot"))
```

### 내부 DSL로 HTML 만들기

HTML 텍스트를 직접 작성하지 않고, 코틀린 코드로 HTML을 생성할 수 있다.

**DSL로 변환한 구조의 장점**
- HTML 구조를 명확하게 표현
- 코드 가독성 향상
- 타입 안전성 보장 (ex. td를 tr 안에서만 사용할 수 있다. 그렇지 않으면 컴파일 오류 발생)
- 동적으로 생성 가능

```kotlin
fun createSimpleTable() = createHTML()
    .table {
        tr {
            td { +"cell" }
        }
    }

fun main() {
    println(createSimpleTable())
}
/*
<table>
  <tr>
    <td>cell</td>
  </tr>
</table>
 */
```

## 구조화된 API 구축: DSL에서 수신 객체 지정 람다 사용

### 수신 객체 지정 람다와 확장 함수 타입

수신 객체 지정 람다의 대표적인 예는 `StringBuilder`의 `buildString` 함수이다. <br />
아래처럼 쉽게 구현가능

```kotlin
fun buildString(
    builderAction: (StringBuilder) -> Unit,
): String {
    val sb = StringBuilder()
    builderAction(sb)
    return sb.toString()
}

fun main() {
    val s = buildString {
        it.append("Hello, ")
        it.append("World!")
    }
    println(s)
}
```

`it.append` 처럼 `it` 접두사를 넣지 않고, `append` 처럼 더 간단하게 호출하게 만들기 
- 타입선언 일반함수 -> 확장함수(수신 객체 타입)
- builderAction 으로 들어온 로직은 확장함수로 만들어지고, 코드 내부에서 실행됨.
- `with`, `apply` 함수도 수신 객체를 갖고 확장 함수 타입의 람다를 호출하는 식으로 구현되어 있음.

```kotlin
fun buildString(
    builderAction: StringBuilder.() -> Unit, // 수신 객체가 지정된 함수 타입의 파라미터를 선언
): String {
    val sb = StringBuilder()
    sb.builderAction()
    return sb.toString()
}

fun main() {
    val s = buildString {
        append("Hello, ")
        append("World!")
    }
    println(s)
}
```

### 수신 객체 지정 람다를 HTML 빌더 안에서 사용

- `table`, `tr`, `td` 등은 모두 평범한 함수로 구성
- 각 함수는 수신 객체 지정 람다를 인자로 받음
- HTML 규칙에 따라 클래스가 만들어졌고, 그 규칙에 따르면, `table` 람다 영역에서는 `td`는 호출 불가능, `tr`은 호출 가능
- 따라서, HTML 언어의 문법을 따르는 코드만 작성할 수 있다.

```kotlin
fun createSimpleTable() = createHTML()
    .table {
        tr {
            td { +"cell" }
        }
    }

class TABLE : Tag {
    fun tr(init: TR.() -> Unit)
}

class TR : Tag {
    fun td(init: TD.() -> Unit)
}

class TD : Tag
```

**외부 람다의 수신 객체에 접근하지 못하게 제안하는 방법 @DslMarker**
- `@DslMarker` 어노테이션이 붙은 영역 안에서는 암시적 수신 객체가 2개가 될 수 없다.

```kotlin
@DslMarker
annotation class HtmlTagMarker

@HtmlTagMarker
open class Tag(val name: String) {
    private val children = mutableListOf<Tag>()

    protected fun <T : Tag> doInit(child: T, init: T.() -> Unit) {
        child.init()
        children.add(child)
    }

    override fun toString(): String {
        return "<$name>${children.joinToString("")}</$name>"
    }
}


fun table(init: TABLE.() -> Unit) = TABLE().apply(init)


class TABLE : Tag("table") {
    fun tr(init: TR.() -> Unit) = doInit(TR(), init)
}

class TR : Tag("tr") {
    fun td(init: TD.() -> Unit) = doInit(TD(), init)
}

class TD : Tag("td")

fun createTable() =
    table {
        tr {
            td {
                // @DslMarker가 없다면, tr { } 이 들어갈 수 있다.
            }
        }
    }

fun main() {
    println(createTable())
    // <table><tr><td></td></tr></table>
}
```

### 코틀린 빌더: 추상화와 재사용을 가능하게 해준다.

반복되는 코드를 새로운 함수로 묶어서 이해하기 쉬운 이름을 붙일 수 있다. **(SQL이나 HTML을 별도로 분리해 이름을 부여하긴 어려움.)**<br />
코틀린 내부 DSL은 일반 코드와 마찬가지로 반복되는 부분을 새 함수로 묶어 재사용 가능!

```html
<!--HTML 코드-->
<body>
    <ul>
        <li><a href="#0">The Three-Body Problem</a></li>
        <li><a href="#1">The Dark Forest</a></li>
        <li><a href="#2">Death's End</a></li>
    </ul>
    <h2 id="0">The Three-Body Problem</h2>
    <p>The first book tackles...</p>
    <h2 id="1">The Dark Forest</h2>
    <p>The second book starts with...</p>
    <h2 id="2">Death's End</h2>
    <p>The third book contains...</p>
</body>
```

```kotlin
// 코틀린 DSL 코드
fun buildBookList() = createHTML().body {
        ul {
            li { a("#1") { +"The Three-Body Problem" } }
            li { a("#2") { +"The Dark Forest" } }
            li { a("#3") { +"Death's End" } }
        }
        h2 { id = "1"; +"The Three-Body Problem" }
        p { +"The first book tackles..." }

        h2 { id = "2"; +"The Dark Forest" }
        p { +"The second book starts with..." }

        h2 { id = "3"; +"Death's End" }
        p { +"The third book contains..." }
    }

// 도우미 함수 사용
fun buildBookList() = createHTML().body {
    listWithToc {
        item("The Three-Body Problem", "The first book tackles...")
        item("The Dark Forest", "The second book starts with...")
        item("Death's End", "The third book contains...")
    }
}
```

## invoke 관례를 사용해 더 유연하게 블록 내포시키기

코틀린 DSL에서 `invoke` 관례를 사용하면, 커스텀 타입의 객체를 함수처럼 호출할 수 있다.

invoke 관례는 괄호를 사용한다. operator 변경자가 붙은 `invoke` 메서드를 구현하면 된다. <br />
내부적으론 `bavarianGreeter.invoke("Dmitry")` 처럼 호출된다. <br />

> 람다를 함수처럼 호출하면 invoke 관례가 적용됨.

```kotlin
class Greeter(val greeting: String) {
    operator fun invoke(name: String) {
        println("$greeting, $name!")
    }
}

fun main() {
    val bavarianGreeter = Greeter("Servus")
    bavarianGreeter("Dmitry")
    // Servus, Dmitry!
}
```

### DSL의 invoke 관례: 그레들 의존관계 선언 

`dependencies` 변수에 대해 `implementation` 메서드를 호출한다.

`dependencies` 변수는 `DependencyHandler` 클래스의 인스턴스이고, `DependencyHandler`는 `invoke` 메서드 정의가 들어있다. <br />
`invoke` 메서드는 수신 객체 지정 람다를 파라미터로 받고, 이 람다의 수신객체는 다시 `DependencyHandler`가 된다. <br />

```kotlin
dependencies {
    testImplementation(kotlin("test"))
    implementation("org.jetbrains.exposed:exposed-core:0.40.1")
    implementation("org.jetbrains.exposed:exposed-dao:0.40.1")
}
```

```kotlin
class DependencyHandler {
    fun implemetation(coordinate: String) {
        println("Added dependency on $coordinate")
    }
    
    operator fun invoke(
        body: DependencyHandler.() -> Unit
    ) {
        body()
    }
}
```

위와 같은 구조 때문에, `dependencies.implemtation("")`, `depedencies { implementation("") }` 둘 다 가능하다. <br />

## 실전 코틀린 DSL

### 중위 호출 연쇄시키기 : 테스트 프레임워크의 should 함수

코틀린 DSL에서 중위 호출을 사용하면, 메서드 체이닝을 더 간결하게 표현할 수 있다. <br />
중위 호출은 `infix` 키워드를 사용하여 정의할 수 있으며, 중위 호출을 사용하면 메서드 이름을 생략하고 더 자연스러운 문법으로 호출할 수 있다. <br />

```kotlin
class PrefixTest {
    @Test
    fun testKPrefix() {
        val s = "kotlin".uppercase()
        s should startWith("K")
    }
}

infix fun <T> T.should(matcher: Matcher<T>) = matcher.test(this)

interface Matcher<T> {
    fun test(value: T)
}

fun startWith(prefix: String): Matcher<String> {
    return object : Matcher<String> {
        override fun test(value: String) {
            if (!value.startsWith(prefix)) {
                throw AssertionError("$value does not start with $prefix")
            }
        }
    }
}
```

### 원시 타입에 대해 확장 함수 정의하기: 날짜 처리

days, hours와 같은 원시 타입에 대해 확장 함수를 정의하면, 코드의 가독성을 높이고, 시간 단위를 더 직관적으로 표현할 수 있다. <br />

```kotlin
import kotlin.time.Duration
import kotlin.time.DurationUnit
import kotlin.time.toDuration

val Int.days: Duration
    get() = this.toDuration(DurationUnit.DAYS)

val Int.hours: Duration
    get() = this.toDuration(DurationUnit.HOURS)
```

# 요약

- 내부 DSL은 여러 메서드 호출로 구성된 구조를 더 쉽게 표현할 수 있게 해주는 API를 설계할 때 사용할 수 있는 설계 패턴이다.
- 수신 객체 지정 람다는 람다 본문 안에서 메서드를 결정하는 방식을 재정의함으로써 여러 요소를 내포시킬 수 있는 구조를 만들 수 있다.
- 수신 객체 지정 람다를 파라미터로 받은 경우 그 람다의 타입은 확장 함수 타입이다. 람다를 파라미터로 받아 사용하는 함수는 람다를 호출하면서 람다에게 수신 객체를 제공한다.
- 외부 템플릿이나 마크업 언어 대신 코틀린 내부 DSL을 사용하면, 코드를 추상화하고 재활용할 수 있다.
- 원시 타입에 대한 확장을 정의하면 기간 등의 여러 종류의 상수를 더 읽기 좋게 만들 수 있다.
- invoke 관례를 사용하면, 임의의 객체를 함수처럼 다룰 수 있다.
- kotlinx.html 라이브러리는 HTML 페이지 생성 내부 DSL을 제공한다. 또한 애플리케이션에 맞게 그 내부 DSL을 확장할 수 있다.
