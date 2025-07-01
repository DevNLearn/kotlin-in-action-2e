# Chapter13. DSL 만들기

## 13장에서 다루는 내용

- 도메인 특화 언어 만들기
- 수신 객체 지정 람다 사용
- invoke 관례 사용
- 기존 코틀린 DSL 예제

수신 객체 지정 람다를 사용하면 코드 블록에서 이름(변수)이 가리키는 대상을 결정하는 방식을 변경해서 DSL 구조를 더 쉽게 만들 수 있다.

invoke 관례를 사용하면 DSL 코드 안에서 람다와 프로퍼티 대입을 더 유연하게 조합할 수 있다.

## API에서 DSL로: 표현력이 좋은 커스텀 코드 구조 만들기

---

- 코틀린의 궁극적인 목표는 코드의 가독성과 유지 보수성을 가장 좋게 유지하는 것이다.
- 이 목표를 달성하려면 개별 클래스에 집중하는 것만으로는 충분치 않다.
- 클래스에 있는 코드 중 대부분은 다른 클래스와 상호작용한다.
- 따라서 상호작용이 일어나는 연결 지점을 살펴봐야 한다.
- 즉 클래스의 API를 살펴봐야한다.
- 라이브러리뿐만 아니라 애플리케이션 안의 모든 클래스의 상호작용도 이해하기 쉽고 명확하게 표현할 수 있어야 프로젝트 유지보수성이 좋다.

### API가 깔끔하다.

- 코드를 읽는 독자가 어떤 일이 벌어질지 명확하게 이해할 수 있어야 한다.
    - 이름과 개념을 잘 선택하면 이런 목적을 달성할 수 있다.
    - 언어와 관계 없이 중요하다.
- 코드에 불필요한 구문이나 번잡한 준비 코드가 가능한 한 적어야 한다.
    - 이번 장에서 주로 초점을 맞추는 부분
    - 깔끔한 API는 언어에 내장된 기능과 거의 구분할 수 없다.
- 이런 깔끔한 api를 돕는 코틀린 기능에는 확장 함수, 중위 함수 호출, 람다에 사용할 수 있는 it등의 문법적 편의, 연산자 오버로딩 등이 있다.

| 일반 구문 | 간결한 구문 | 사용한 언어 특성 |
| --- | --- | --- |
| StringUtil.capitalize(s) | s.capitalize() | 확장 함수 |
| 1.to("one") | 1 to "one" | 중위 호출 |
| set.add(2) | set += 2 | 연산자 오버로딩 |
| map.get("key") | map["key"] | get 메소드에 대한 관례 |
| file.use({ f -> f.read() }) | file.use { it.read() } | 람다를 괄호 밖으로 빼내는 관례 |
| sb.append("yes") sb.append("no") | with(sb) { 
  append("yes") 
  append("no") 
} | 수신 객체 지정 람다 |
| val m = mutableListOf<Int>()
m.add(1)
m.add(2)
return m.toList() | return buildList {
  add(1)
  add(2)
} | 람다를 받는 빌더 함수 |
- 더 나아가 DSL 구축을 도와주는 코틀린 기능을 살펴보자
- 코틀린 DSL은 문법적 특성과 여러 메서드 호출에서 구조를 만들어내는 능력 위에 구축된다.
    - 그 결과로 DSL은 메서드 호출만을 제공하는 API에 비해 더 표현력이 풍부해지고 사용하기 편해진다.
- 코틀린 DSL도 온전히 컴파일 시점에 타입이 정해진다.
    - 따라서 컴파일 시점 오류 감지, IDE 지원 등 모든 정적  타입 지정 언어의 장점을 누릴 수 있다.

### 도메인 특화 언어

- DSL이라는 개념은 오래된 개념이다.
- 범용 프로그래밍 언어(general-purpose programming language)와 특정 과업 또는 영역(도메인)에 초점을 맞추고 그 영역에 필요하지 않은 기능을 없앤 도메인 특화 언어를 구분해왔다.
- 가장 익숙한 DSL 예시
    - SQL과 정규식
- DSL은 범용 프로그래밍 언어와 달리 더 선언적이라는 점이 중요하다.
    - 범용 프로그래밍 언어는 보통 명령적
        - 명령적 언어는 어떤 연산을 완수하기 위해 필요한 각 단계를 순서대로 정확히 기술한다.
    - 선언적 언어는 원하는 결과를 기술하기만 하고 그 결과를 달성하기 위해 필요한 세부 실행은 언어를 실행하는 엔진에 맡긴다.
        - 실행 엔진이 결과를 얻는 과정을 전체적으로 한꺼번에 최적화 하기 때문에 선언적 언어가 더 효율적인 경우가 자주 있다.
        - eg)SQL
- DSL의 단점
    - 범용 언어로 만든 호스트 애플리케이션과 DSL을 함꼐 조합하기가 어렵다는 것
    - DSL은 자체 문법이 있기 때문에 다른 언어의 프로그램 안에 직접 포함시킬 수가 없다.
        - 따라서 DSL로 작성한 프로그램을 다른 언어에서 호출하려면 DSL 프로그램을 별도의 파일이나 문자열 리터럴로 저장해야 한다.
        - 하지만 이런 식으로 DSL을 저장하면 호스트 프로그램과 DSL의 상호작용을 컴파일 시점에 제대로 검증하거나, DSL 프로그램을 디버깅하거나, DSL 코드 작성을 돕는 IDE 기능을 제공하기 어려워지는 문제점이 있다.
- 이런 문제를 해결하면서 DSL의 다른 이점을 살리는 방법으로 코틀린에서는 내부(internal) DSL을 만들 수 있게 해준다.

### 내부 DSL은 프로그램의 나머지 부분과 매끄럽게 통합된다.

```kotlin
(Country innerJoin Customer)
	.slice(Country.name, Count(Customer.id))
	.selectAll()
	.groupBy(Country.name)
	.orderBy(Count(Customer.id), order = SortOder.DESC)
	.limit(1)
```

- 독립적인 문법 구조를 갖는 외부(external) DSL 과는 반대로 내부 DSL은 범용 언어로 작성된 프로그램의 일부며, 범용 언어와 동일한 문법을 사용한다.
- 따라서 내부 DSL은 완전히 다른 언어가 아니라 DSL의 핵심 장점을 유지하면서 주 언어를 별도의 문법으로 사용하는 것
    - eg) exposed
- 코드는 어떤 구체적인 과업을 달성(sql 질의를 만듦)하기 위한 것이지만 범용 언어(코틀린)의 라이브러리로 구현된다.

### DSL의 구조

- DSL과 일반 API 사이에 잘 정의된 일반적인 경계는 없다.
- DSL은 중위 호출이나 연산자 오버로딩 같인 다른 문맥에서도 널리 쓰이는 언어 기능에 의존하기도 한다.
- 하지만 다른 API에는 존재하지 않지만 DSL에만 존재하는 특징이 한가지 있다.
    - 구조 또는 문법
- 전형적인 라이브러리는 여러 메서드로 이뤄지며 클라이언트는 그런 메서드를 한번에 하나씩 호출함으로써 라이브러리를 사용한다.
    - 함수 호출 시퀀스에는 내포나 그룹 같은 아무런 구조가 없으며 한 호출과 다른 호출 사이에는 맥락이 유지되지 않는다.
    - 그런 API를 때로 명령-질의(command-query) API라고 부른다.
- 반대로 DSL 메서드 호출은 DSL 문법에 의해 정해지는 더 커다란 구조에 속한다.
    - 코틀린 DSL에서는 보통 람다를 내포시키거나 메서드 호출을 연쇄시키는 방식으로 구조를 만든다.
    - 질의를 실행하려면 필요한 결과 집합의 여러 측면을 기술하는 메서드 호출을 조합해야 한다.
    - 그렇게 메서드를 조합해서 만든 질의는 질의에 필요한 인자를 메서드 호출 하나에 모두 다 넘기는 것보다 훨씬 더 읽기 쉽다.
    - 이런 문법이 있기 때문에 내부 DSL을 언어라고 부를 수 있다.
- DSL에서는 여러 함수 호출을 조합해서 연산을 만들며 타입 검사기는 여러 함수 호출이 바르게 조합됐는지를 검사한다.
    - 결과적으로 함수 이름은 보통 동사 역할을 하고 함수 인자는 명사 역할을 한다.
- DSL 구조의 장점은 같은 맥락을 매 함수 호출 시마다 반복하지 않고도 재사용을 할 수 있다는 점이다.
    - eg) 그레이들 빌드 스크립트에서 의존관계 정의
        
        ```kotlin
        dependencies { // 람다 내포를 통해 구조를 만든다.
            implementation(kotlin("test"))
            implementation("org.junit.jupiter:junit-jupiter:5.8.2")
            implementation("org.jetbrains.kotlin:kotlin-reflect:1.7.0")
            implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.8.1-Beta")
        }
        ```
        
- 메서드 호출 연쇄는 DSL 구조를 만드는 또 다른 방법이다.
    - 테스트 프레임워크에서 단언문을 여러 메서드 호출로 나눠 작성하는 경우가 많다.
    - 그런 단언문은 훨씬 더 읽기 쉽다.
    - 특히 중위 호출 구문을 사용하면 가독성이 더 좋아진다.
    - eg) kotest
        - `str should startWith(”kot”)`
    - junit
        - `assertTrue(str.startWith(”kot”))`

### 내부 DSL로 html 만들기

```kotlin
import kotlinx.html.stream.createHTML
import kotlinx.html.*

fun createSimpleTable() = createHtml().
	table {
		tr {
			td { + "cell"}
		}
	}
```

- kotlinx.html 라이브러리
- 코틀린 코드로 html 만드는 이유
    - 타입 안전성
    - 코틀린 코드를 원하는 대로 사용 가능
- 이제 DSL 문법을 만들 때 가장 중요한 역할을 하는 수신 객체 지정 람다를 더 자세히 살펴보자

## 구조화된 API 구축: DSL에서 수신 객체 지정 람다 사용

---

수신 객체 지정 람다는 구조화된 API를 만들 때 도움외 되는 강력한 코틀린 기능이다.

구조가 있다는 점은 일반 API와 DSL을 구분하는 중요한 특성이다. buildString 함수를 예제로 이들을 어떻게 구현하는지 살펴보자

### 수신 객체 지정 람다와 확장 함수 타입

```kotlin
fun buildString(
    builderAction: (StringBuilder) -> Unit, // 함수 타입인 파라미터
): String {
    val sb = StringBuilder()
    builderAction(sb) // 람다 인자로 StringBuilder 인스턴스를 넘긴다.
    return sb.toString()
}

fun main() {
    val s = buildString {
        it.append("Hello, ") // it은 StringBuilder 인스턴스를 가리킨다.
        it.append("World!")
    }
    println(s)
    // Hello, World!
}

```

- 일반 람다를 받는 buildString()
- 이해하기 쉽지만 편하지 않다.
    - 람다 본문에서 매번 it을 사용해 참조해야 한다.
- 람다의 목적이 StringBuilder를 텍스트로 채우는 것이므로 it을 접두사로 넣지 않고 더 간단하게 호출하기를 바란다.
    - 그렇게 하려면 람다를 수신 객체 지정 람다로 바꿔야 한다.
    - 람다의 인자 중 하나에 수신 객체라는 상태를 부여하면 이름과 마침표를 명시하지 않아도 그 인자의 멤버를 바로 사용할 수 있다.

```kotlin
fun buildString(
    builderAction: StringBuilder.() -> Unit, // 수신 객체가 지정된 함수 타입의 파라미터
): String {
    val sb = StringBuilder()
    sb.builderAction() // StringBuilder 인스턴스를 람다의 수신 객체로 넘긴다.
    return sb.toString()
}

fun main() {
    val s = buildString {
        this.append("Hello, ") // this는 StringBuilder 인스턴스를 가리킨다.
        append("World!") // this 생략 가능
    }
    println(s)
    // Hello, World!
}
```

- 수신 객체 지정 람다를 파라미터로 받는 buildString()
- it을 사용하지 않아도 된다.
- this는 모호성을 해결해야 할 때만 쓴다.
- 파라미터 타입을 선언할 때 일반 함수 타입 대신 확장 함수 타입을 사용한다.
    - 확장 함수 타입 선언은 람다의 파라미터 목록에 있던 수신 객체 타입을 파라미터 목록을 여는 괄호 앞으로 빼내 중간에 마침표를 붙인 형태다.
    - 이런 타입을 수신 객체 타입이라 부르며, 람다에 전달되는 그런 타입의 객체를 수신 객체라고 부른다.

```kotlin
  String     .     (Int, Int)     →    Unit

-수신객체타입-        -파라미터 타입-       -반환타입-
```

- 확장 함수 타입 선언
- 왜 확장 함수 타입일까?
    - 외부 타입의 멤버를 아무런 수식자 없이 사용한다는 개념 → 확장 함수
    - 확장 함수나 수신 객체 지정 람다에서는 모두 함수(람다)를 호출할 때 수신 객체를 지정해야만 하고, 함수(람다) 본문 안에서는 그 수신 객체를 특별한 수식자 없이 사용할 수 있다.
    - 결과적으로 확장 함수 타입은 확장 함수처럼 호출될 수 있는 코드 블록을 표시한다.

```kotlin
buildString { this.append("!") }
            // { this.append("!") } <> builderAction
            // this <> sb

fun buildString(
    builderAction: StringBuilder.() -> Unit,
): String {
    val sb = StringBuilder()
    sb.builderAction()
    return sb.toString()
}

```

- 일반 함수 타입의 변수를 호출할 때와 확장 함수 타입의 변수를 호출할 때는 호출 방법도 다르다.
    - 객체를 인자로 넘기는 대신, 람다 변수를 마치 확장 함수처럼 호출해야 한다.
        - `buildAction(sb)` → `sb.buildAction()`
    - `sb.buildAction()` 에서 `buildAction` 은 StringBuilder에 정의된 메서드가 아니며, StringBuilder 인스턴스인 sb는 확장 함수를 호출할 때와 동일한 구문으로 호출할 수 있는 함수 타입(따라서 확장 함수 타입)의 인자일 뿐이다.
- buildString 함수(수신 객체 지정 람다)의 인자는 확장 함수 타입의 파라미터(builderAction)와 대응한다.
- 호출된 람다 본문 안에서 수신객체(sb)가 암시적 수신 객체(this)가 된다.

```kotlin
val appendExcl: StringBuilder.() -> Unit =
    { this.append("!") } // 확장 함수 타입의 값

fun main() {
    val stringBuilder = StringBuilder("Hi")
    stringBuilder.appendExcl() // 확장 함수처럼 호출할 수 있다.
    println(stringBuilder)
    // Hi!
    println(buildString(appendExcl)) // 인자로 넘길 수 있다.
    // !
}

```

- 확장 함수 타입의 변수를 정의할 수도 있다.
- 정의된 확장 함수 타입 변수를 마치 확장 함수처럼 호출하거나 수신 객체 지정 람다를 요구하는 함수에 인자로 넘길 수 있다.
- 소스코드에서 수신 객체 지정 람다는 일반 람다와 똑같아 보인다는 점을 유의하자.
    - 람다에 수신 객체가 있는지 알아보려면 그 람다가 전달되는 함수를 살펴봐야 한다.
    - 함수 시그니처를 보면 람다에 수신 객체가 있는지와 람다가 어떤 타입의 수신 객체를 요구하는지를 알 수 있다.

```kotlin
public inline fun <T> T.apply(block: T.() -> Unit): T {
    block()
    return this
}

public inline fun <T, R> with(receiver: T, block: T.() -> R): R {
    return receiver.block()
}
```

```kotlin
fun main() {
    val map = mutableMapOf(1 to "one")
    map.apply { this[2] = "two" }
    with(map) { this[3] = "three" }
    println(map)
    // {1=one, 2=two, 3=three}
}
```

- 두 함수 모두 자신이 제공받은 수신 객체를 갖고 확장 함수 타입의 람다를 호출한다.
- apply는 수신 객체 타입에 대한 확장 함수로 선언됐기 때문에 수신 객체의 메서드처럼 호출되며 수신 객체를 암시적 인자(this)로 받는다.
- with은 수신 객체를 첫 번째 파라미터로 받는다.
- apply는 수신 객체를 다시 반환하지만 with은 람다를 호출해 얻은 결과를 반환한다.
- 따라서 결과를 받아서 쓸 필요가 없다면 두 함수를 서로 바꿔 쓸수 있다.
- 이런 개념을 DSL에서 어떻게 사용하는지 살펴보자

### 수신 객체 지정 람다를 HTML 빌더 안에서 사용

```kotlin
import kotlinx.html.stream.createHTML
import kotlinx.html.*

fun createSimpleTable() = createHtml().
	table {
		tr {
			td { + "cell"}
		}
	}
```

- 일반 코틀린 코드이다.
- table, tr, tb 모두 평범한 함수
- 각 함수는 고차 함수로, 수신 객체 지정 람다를 인자로 받는다.
- `+”cell”` 이라고 쓸 때, 이 코드는 단순히 함수 호출이다.
    - unaryPlus 연산자 오버로딩해서 자신을 둘러싼 td 셀 안에 문자열의 내용을 추가
- 중요한점은 각 수신 객체 지정 람다가 이름 결정 규칙을 바꾼다는 점
    - 각 람다는 자신의 본문에서 호출될 수 있는 함수들을 새로(이름을 찾는 영역) 추가한다.
    - table 함수에 넘겨진 람다에서는 tr 함수를 사용할 수 있지만, 이 람다의 밖에서는 tr 이라는 이름의 함수를 찾을 수 없다.

```kotlin
open class Tag

class TABLE : Tag {
    fun tr(init: TR.() -> Unit)
}

class TR : Tag {
    fun td(init: TD.() -> Unit)
}

class TD : Tag
```

```kotlin
import kotlinx.html.stream.createHTML
import kotlinx.html.*

fun createSimpleTable() = createHtml().
	table { 
		this@table.tr {
			(this@tr).td { + "cell"}
		}
	}
```

- 각 블록의 이름 결정 규칙은 각 람다의 수신 객체에 의해 결정된다.
    - table에 전달된 수신 객체는 TABLE이라는 특별한 타입이며, 그 안에 tr 메서드 정의가 있다.
    - 마찬가지로 tr 함수는 TR 객체에 대한 확장 함수 타입의 람다를 받는다.
- 유틸리티 클래스라 모두 대문자로 정의해서 구분
- 수신 객체 지정 람다가 아닌 일반 람다였다면 난잡해질 것이다.
    - 매번 it 붙이거나 람다에 이름 정의
    - 수신 객체를 암시적으로 정하고 this 참조를 쓰지 않아도 되면 문법이 간단해진다.

```kotlin
@DslMarker
annotation class HtmlTagMarker

@HtmlTagMarker
open class Tag(val name: String)
```

- 영역 안에 여러 수신 객체가 있으면 혼동이 올 수 있다.
    - 수신 객체 지정 람다가 다른 수신 객체 지정 람다 안에 들어가면 안쪽 람다에서 외부 람다에 정의된 수신객체를 사용할 수 있기 때문
    - 내포 깊이가 깊은 구조에서는 어떤 식의 수신 객체가 무엇인지 분명하지 않아서 혼동이 올 수 있다.
- 이를 막기 위해 코틀린은 @DslMaker 어노테이션을 사용해 내포된 람다에서 외부 람다의 수신 객체에 접근하지 못하게 제한할 수 있다.
    - 메타 어노테이션

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

    override fun toString() =
        "<$name>${children.joinToString("")}</$name>"
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
            }
        }
    }

fun main() {
    println(createTable())
    // <table><tr><td></td></tr></table>
}
```

### 코틀린 빌더: 추상화와 재사용을 가능하게 해준다.

```kotlin
fun buildBookList() = createHTML().body {
    ul {
        li { a("#1") { +"Chapter 1" } }
        li { a("#2") { +"Chapter 2" } }
        li { a("#3") { +"Chapter 3" } }
    }
    
    h2 { id = "1"; +"Chapter 1" }
    p { +"챕터 1" }
    h2 { id = "2"; +"Chapter 2" }
    p { +"챕터 2" }
    h2 { id = "3"; +"Chapter 3" }
    p { +"챕터 3" }
}
```

- 반복되는 코드를 함수를 묶어서 이해하기 쉬운 이름을 붙일 수 있다.
    - 하지만 sql이나 html을 별도 함수로 분리해 이름을 부여하기 어렵다.

```kotlin
fun buildBookList2() = createHTML().body { 
    listWithToc {
        item("Chapter 1", "챕터 1")
        item("Chapter 2", "챕터 2")
        item("Chapter 3", "챕터 3")
    }
}
```

```kotlin
@HtmlTagMarker
class LISTWITHTOC {
    val entries = mutableListOf<Pair<String, String>>()
    fun item(headline: String, body: String) {
        entries += headline to body
    }
}
```

```kotlin
fun BODY.listWithToc(block: LISTWITHTOC.() -> Unit) {
    val listWithToc = LISTWITHTOC()
    listWithToc.block()

    ul {
        for ((index, entry) in listWithToc.entries.withIndex()) {
            li { a("#$index") { +entry.first} }
        }
    }
    for ((index, entry) in listWithToc.entries.withIndex()) {
        h2 { id = "$index"; +entry.first }
        p { +entry.second }
    }
}
```

- 코틀린 내부 DSL을 사용하면 일반 코드와 마찬가지로 반복되는 내부 DSL 코드 조각을 새 함수로 묶어 재사용 할 수 있다.
    - listWithToc 함수는 새 LISTWITHTOC 인스턴스를 만들고 파라미터로 받은 block 람다를 호출하되 새 LISTWITHTOC 인스턴스를 수신 객체로 지정해 호출한다.
    - 여기서 `ul`, `h2` 등의 함수를 호출할 수 있는 이유는 `listWithToc`이 `BODY`의 확장함수이기 때문이다

## invoke 관례를 사용해 더 유연하게 블록 내포시키기

---

invoke 관례를 사용하면 어떤 커스텀 타입의 객체를 함수처럼 호출 할 수 있다. 

함수 타입의 객체를 함수라고 부를 수 있다는 사실을 이미 살펴봤다. invoke 관례를 쓰면 함수와 똑같은 구문을 지원하는 자신만의 객체를 정의할 수 있다. DSL 에서는 이 관례가 쓸모 있다.

### invoke 관례를 사용해 더 유연하게 블록 내포시키기

```kotlin
class Greeter(val greeting: String) {
    operator fun invoke(name: String) {
        println("$greeting, $name!")
    }
}

fun main() {
    val bavarianGreeter = Greeter("Servus")
    bavarianGreeter("Dmitry") // 인스턴스를 함수처럼 호출
    // Servus, Dmitry!
}

```

- 각괄호 대신 괄호를 사용한다.
- opeator 변경자가 붙은 invoke 메서드 정의가 들어있는 클래스의 객체를 함수처럼 호출할 수 있다.
- invoke 메서드의 시그니처에 대한 요구 사항은 없다.
    - 원하는 대로 파라미터 개수나 타입 지정 가능
- 일반적인 람다 호출이 실제로는 invoke 관례를 적용한 것
    - 인라인 람다를 제외한 모든 람다는 함수형 인터페이스(Function1 등)를 구현하는 클래스로 컴파일된다.
    - 각 함수형 인터페이스 안에는 그 인터페이스 이름이 가리키는 수만큼의 파라미터를 받는 Invoke 메서드가 들어 있다.
    - 람다를 함수처럼 호출하면 이 관례에 따라 invoke 메서드 호출로 변환된다.

### DSL의 invoke 관례: 그레이들 의존관계 선언

```kotlin
class DependencyHandler {
    fun implementation(coordinate: String) {
        println("Added dependency on $coordinate")
    }

    operator fun invoke(
        body: DependencyHandler.() -> Unit,
    ) {
        body() // this가 함수의 수신 객체가 되므로 this.body()와 동일
    }
}

fun main() {
    val dependencies = DependencyHandler()
    dependencies.implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.8.0")
    // Added dependency on org.jetbrains.kotlinx:kotlinx-coroutines-core:1.8.0
    dependencies { // invoke
        implementation("org.jetbrains.kotlinx:kotlinx-datetime:0.5.0")
    }
    // Added dependency on org.jetbrains.kotlinx:kotlinx-datetime:0.5.0
}

```

- **유연하게 내포된 블록 구조를 허용하는 동시에 평평한 함수 호출 구조도 제공하고 싶은 경우**

## 실전 코틀린 DSL

---

실용적인 DSL 구성 예제를 살펴보자

- 테스팅
- 날짜 리터럴
- 데이터베이스 질의

### 중위 호출 연쇄시키기: kotest should 함수

```kotlin
s should startWith("K")

infix fun <T> T.should(matcher: Matcher<T>) = matcher.test(this)

```

```kotlin
interface Matcher<T> {
    fun test(value: T)
}

fun startWith(prefix: String): Matcher<String> {
    return object : Matcher<String> {
        override fun test(value: String) {
            if (!value.startsWith(prefix)) {
                throw AssertionError("Expected '$value' to start with '$prefix'")
            }
        }
    }
}
```

- Matcher: 값에 대한 단언문을 표현하는 제네릭 인터페이스
- 중위 호출과 object로 정의한 싱글턴 객체 인스턴스를 조합하면 DSL에 상당히 복잡한 문법 도입 가능
    - 그럼에도 정적 타입 지정언어로 남는다.

### 원시 타입에 대해 확장 함수 정의하기: 날짜 처리

```kotlin
val Int.days: Duration
    get() = this.toDuration(DurationUnit.DAYS)

val Int.hours: Duration
    get() = this.toDuration(DurationUnit.HOURS)

```

- 여기서 days와 hours는 Int 타입의 확장 프로퍼티다.
- 아무타입이나 확장 함수의 수신 객체 타입이 될 수 있다.
    - 원시 타입에 대한 확장 함수를 정의하고 원시 타입 상수에 대해 그 확장 함수를 호출할 수 있다.

### 멤버 확장 함수: SQL을 위한 내부 DSL

- 멤버 확장
    - 클래스 안에서 확장 함수와 확장 프로퍼티를 선언하는 기법
    - 이런 확장 함수와 확장 프로퍼티는 그들이 선언된 클래스의 멤버인 동시에 그들이 확장하는 다른 타입의 멤버이기도 하다.
- exposed

```kotlin
class Table {
    fun Column<Int>.autoIncrement(): Column<Int>
    // ...
}
```

- autoIncrement 함수는 Table 클래스의 멤버이기는 하지만 여전히 Colum의 확장 함수
- 사용하는 이유는 메서드가 적용되는 범위를 제한하기 위함이다.
    - 테이블이라는 맥락이 없으면 칼럼의 프로퍼티를 정의해도 아무 의미가 없는데, 필요한 메서드를 찾아낼 수 없기 때문이다.
- 또 다른 이유는 확장 함수의 다른 멋진 속성은 수신 객체 타입을 제한하는 기능이다.
    - 테이블 안의 어떤 컬럼이든 기본 키가 될 수 있지만 자동 증가 칼럼이 될 수 있는 칼럼은 정수 타입인 칼럼 뿐이다.
    - Column의 확장 함수로 autoIncrement를 정의하면 이런 관계를 API 코드로 구현할 수 있다.
    - 다른 타입은 컴파일 실패

### 요약

---

- 내부 DSL은 여러 메서드 호출로 구성된 구조를 더 쉽게 표현할 수 있게 해주는 API를 설계할 때 사용할 수 있는 설계 패턴이다.
- 수신 객체 지정 람다는 람다 본문 안에서 메서드를 결정하는 방식을 재정의함으로써 여러 요소를 내포시킬 수 있는 구조를 만들 수 있다.
- 수신 객체 지정 람다를 파라미터로 받은 경우 그 람다의 타입은 확장 함수 타입이다.
    - 람다를 파라미터로 받아 사용하는 함수는 람다를 호출하면서 람다에게 수신 객체를 제공한다.
- 외부 템플릿이나 마크업 언어 대신 코틀린 내부 DSL을 사용하면 코드를 추상화하고 재활용할 수 있다.
- 원시 타입에 대한 확장을 정의하면 기간 등의 여러 종류의 상수를 더 읽기 좋게 만들 수 있다.
- invoke 관례를 사용하면 임의의 객체를 함수처럼 다룰 수 있다.
- kotlinx.html 라이브러리는 html 페이지 생성 내부 dsl을 제공한다.
    - 또한 애플리케이션에 맞게 그 내부 dsl을 확장할 수 있다.
- kotest 라이브러리는 단위 테스트에서 읽기 쉬운 단언문을 지원하는 내부 dsl을 제공한다.
- exposed 라이브러리는 db를 다루기 위한 내부 dsl을 제공한다.