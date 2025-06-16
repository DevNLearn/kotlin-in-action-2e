## 13장

### DSL 은
- 도메인 특화 언어 (`Domain Specific Language`)
- 개발자는 API 를 깔끔하고 가독성 좋게 만들어야한다.
- 깔끔한 API 는...
  - 언어와 관계없이 코드를 읽는 독자가 어떤 일이 벌어질지 명확하게 이해해야한다
  - 코드에 불필요한 구문이나 번잡한 준비 코드가 적어야 한다.
- 간결한 구문을 지원하기 위해 코틀린은 다음과 같은 방법들이 있다.

| 일반 구문                                                                       | 간결한 구문                                              | 사용한 언어 특성         |
|-----------------------------------------------------------------------------|-----------------------------------------------------|-------------------|
| StringUtil.capitalize(s)                                                    | s.capitalize()                                      | 확장 함수             |
| 1.to("one")                                                                 | 1 to "one                                           | 중위 호출             |
| set.add(2)                                                                  | set += 2                                            | 연산자 오버로딩          |
| map.get("key")                                                              | map["key"]                                          | get 메서드에 대한 관례    |
| file.use({ f -> f.read() })                                                 | file.use { it.read() }                              | 람다를 괄호 밖으로 빼내는 관례 | 
| sb.append("yes")<br/>             sb.append("no")                           | with (sb) {<br/>append("yes")<br/>append("no")<br/>} | 수신 객체 지정 람다       | 
| val m = mutableListOf<Int>()<br/>m.add(1)<br/>m.add(2)<br/>return m.toList() | return buildList {<br/> add(1)<br/>add(2)<br/>}     | 람다를 받는 빌더 함수      | 


- 코틀린 DSL 도 온전히 컴파일 시점에 타입이 정해진다.
  - 컴파일 시점 오류 감지
  - IDE 지원

---

### 도메인 특화 언어
- 특정 도메인이나 문제 영역에 특화된 언어. SQL 과 정규식 등이 대표적인 DSL
- SQL -> 데이터베이스, 정규식 -> 문자열
- 제공하는 기능을 제한하여, 해당 기능에 더 특화되게 사용한다
- 일반 프로그래밍 언어는 명령적인데에 반해, DSL 은 선언적이다.
- 원하는 결과를 기술만 하고, 각 단계별 세부 실행은 엔진에 맡긴다.
- 하지만 범용 언어로 만든 어플리케이션과 DSL 을 조합하기가 힘든 단점도 존재
  - 문자열로 리터럴 저장
  - 컴파일 검증이 힘듬
  - IDE 기능 사용이 힘듬
  - 배워야하는게 두배

---

### 내부 DSL, 외부 DSL
- 외부 DSL 은 아예 독립적인 문법 구조를 갖게된다
- 내부 DSL 은 동일한 언어로 작성된 일부며, 주 언어의 별도의 문법으로 사용

- SQL
- 외부 DSL
```sql
select Country.name, COUNT(Customer.id)
    from Country
    INNER JOIN Customer
        ON Country.id = Customer.country_id
    GROUP BY Country.name
    ORDER BY COUNT(Customer.id) DESC
    LIMIT 1
```

---
- Kotlin + exposed 
- 코드는 외부 DSL 을 만들기 위한 것이지만, 범용 언어 라이브러리로 구현 된다.


```kotlin
(Country innerJoin Customer)
    .slice(Country.name, Count(Customer.id))
    .selectAll()
    .groupBy(Country.name)
    .orderBy(count(Customer.id), order = SortOrder.DESC)
    .limit(1)
```


---

### DSL 구조

- 일반 API 와 DSL 은 구분하기 쉽지않다.
- 하지만 DSL 에서 존재하는 특징으로 구조/문법 이 있다.
- 질의를 실행하려면 결과 집합의 여러 측면을 기술하는 메서드 호출을 조합해야하며, 메서드를 조합해서 만든 질의는 질의의질의에 필요한 인자를 메서드 호출 하나에 모두 다 넘기는 것보다 읽기 쉽다.
- 여러 함수 호출을 조합하여 호출이 바르게 되었는지 검사하는데, 함수 이름은 보통 동사 역할, 함수 인자는 명사 역할을 한다
- 함수 호출 시마다 반복하지 않고 재사용 가능하다

- 코틀린을 위한 서드파티 테스트 프레임워크 코테스트는 다음과 같이 간결하게 사용할 수 있다

```kotlin
// koTest
str should startWith("kot")

// 일반 JUnit 스타일 API
assertTrue(str.startwith("kot"))

```

- 코틀린에서 HTML 페이지를 생성하는 DSL 이 존재하고, 표를 만드는 간단한 예시는 다음과 같다
```kotlin
fun createSimpleTable() = createHTML().
    table {
        tr {
            td {"cell"}
        }
    }
```

- 위 코틀린 코드는 다음과 같은 HTML 을 반환한다
```html
<table>
    <tr>
        <td>cell</td>
    </tr>
</table>
```

- 직접 HTML 작성하는것에 비해, 타입 안정성을 보장한다.
- 코틀린 코드기 때문에 동적으로 표의 셀을 생성할 수 있다

---

### 수신 객체 지정 람다와 확장 함수 타입

- 일반 람다를 받는 `buildString` 함수가 다음처럼 있다고 가정하자
```kotlin
fun buildString(
    builderAction: (StringBuilder) -> Unit
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
    // Hello, World!
    println(s)
}
```

- 람다 본문에서 매번 it을 사용하여 `StringBuilder` 인스턴스를 참조해야 한다.
- 람다의 인자 중 하나의 수신객체 상태를 부여하면, 인자의 멤버를 바로 사용할 수 있는데,

```kotlin
fun buildString(
    builderAction: StringBuilder.() -> Unit // 수신 객체가 지정된 함수의 파라미터 선언
): String {
    val sb = StringBuilder()
    sb.builderAction()      // StringBuilder 인스턴스를 람다의 수신 객체로 넘긴다
    return sb.toString()
}

fun main() {
    val s = buildString {
        this.append("Hello, ")  // this 는 StringBuilder 인스턴스를 가르킨다
        append("World!")        // this 생략해도 StringBuilder 인스턴스가 수신 객체로 취급
    }
    // Hello, World!
    println(s)
}
```

- 주요 차이점으로 `(StringBuilder) -> Unit` 을 `StringBuilder.() -> Unit` 으로 바꿨고, 해당 타입을 `수신 객체 타입` 이라고 부른다. 
- 람다에 전달되는 타입의 객체를 `수신 객체`라고 한다.

---

- 표준 라이브러리의 `buildString` 구현은, `builderAction` 을 명시적으로 호출하는 대신, `apply` 함수에게 인자로 넘긴다. 그러면 한 줄로 줄일 수 있다

```kotlin
fun buildString(builderAction: StringBuilder.() -> Unit): String =
    StringBuilder().apply(builderAction).toString()
```

- `apply` 함수는 인자로 받은 람다나 함수를 호출하며 자신의 수신 객체를 람다나 함수의 암시적 수신 객체로 사용한다
- `with` 은 수신 객체를 첫 번째 파라미터로 받고, 람다를 호출 해 얻은 결과를 반환한다
```kotlin
inline fun <T> T.apply(block: T.() -> Unit): T {
    block()
    return this // 수신 객체 반환
}

inline fun <T, R> with(receiver: T, block: T.() -> R): R =
    receiver.block()    // 람다를 호출해 결과를 반환
```

- 결과를 받아서 쓸 필요가 없다면 두 함수를 서로 바꿔 쓸 수 있다

---

### 수신 객체 지정 람다를 HTML 빌더 내에 사용
- 위에서 나왔던 예시중 하나로 각각 평범한 함수다.
- 각 수신 객체 지정 람다가 이름 결정 규칙을 바꾼다.
- table 함수에 넘겨진 람다는 tr 함수를 통해 HTML 태그를 만들지만, 람다 밖에서는 해당 tr 함수를 찾을 수 없다. 
```kotlin
fun createSimpleTable() = createHTML()
    .table {this: TABLE
        tr {this: TR
            td {this: TD 
                + "cell"}
        }
    }
```

- 각 블록의 이름은 람다의 수신 객체에 의해 결정된다.
- 다음은 클래스와 메서드의 정의를 간단하게 정리한 코드다
```kotlin
open class Tag

class TABLE : TAG {
    fun tr(init : TR.() -> Unit)
}

class TR : TAG {
    fun td(init : TD.() -> Unit)
}
```

- 하지만 수신 객체 지정 람다가 다른 수신 객체 지정 람다 안에 들어가면 안쪽 람다에 정의된 수신 객체를 사용할 수 있다
- 영역 내부에 여러 수신 객체가 있으면 혼동이 올 수 있다.
```kotlin
createHTML().body {
    a {
        img {
            href = "http://..."     // img 가 아닌 a 에 전달된 람다의 수신 객체의 href 를 가르킨다
        }
    }
}
```


- 이를 막기위해 코틀린에서 `@DslMaker` 어노테이션으로 외부 람다의 수신 객체에 접근하지 못하게 할 수 있다

```kotlin
@DslMarker
annotation class HtmlTagMarker

@HtmlTagMarker
open class Tag(val name: String) {
    private val children = mutableListOf<Tag>() // 모든 내포 태그를 저장

    protected fun <T: Tag> doInit(child: T, init: T.() -> Unit) {
        child.init()            // 자식 태그를 초기화 한다
        children.add(child)     // 자식 태그에 대한 참조를 저장한다
    }

    override fun toString() = "<$name>${children.joinToString("")}</$name>" // 결과 HTML을 문자로 반환
}

fun table(init: TABLE.() -> Unit) = TABLE().apply(init)

class TABLE : Tag("table") {
    fun tr(init: TR.() -> Unit) = doInit(TR(), init)    // TR 태그 인스턴스를 새로 만들고, TABLE 태그의 자식으로 등록
}

class TR : Tag("tr") {
    fun td(init: TD.() -> Unit) = doInit(TD(), init)    // TD 태그 인스턴스를 새로 만들고, TR 태그의 자식으로 등록
}

class TD : Tag("td")
```

---

### 추상화와 재사용의 가능
- 반복되는 코드를 새로운 함수로 묶는게 좋다고 우리는 안다. 하지만 SQL, HTML 들을 별도의 함수로 분리하여 메서드로 만들기는 쉽지 않다.
- 하지만 내부 DSL 을 사용하여 반복되는 코드를 재사용 할 수 있다.

```kotlin
fun buildBookList() = createHTML().body {
    ul {
        li { a("#1") { + "One-Body" } }
        li { a("#2") { + "Two-Body" } }
        li { a("#3") { + "Three-Body" } }
    }

    h2 { id = "1"; + "One-Body" }
    p { + "일번" }
    h2 { id = "2"; + "Two-Body" }
    p { + "이번" }
    h2 { id = "3"; + "Three-Body" }
    p { + "삼번" }
}
```
- 도우미 함수를 사용하면 다음과 같이 변경될 수 있다
```kotlin
fun buildBookList() = createHTML().body {
    listWithToc {
        item("One-Body", "일번")
        item("Two-Body", "이번")
        item("Three-Body", "삼번")
    }
}
```

- `LISTWITHOTC` 클래스를 직접 만들수도 있다.
- `@HtmlTagMaker` 어노테이션으로 DSL 영역 규칙을 따르게 할 수 있다
- 목차를 직접 HTML `BODY` 아래로 넣고자 한다면, `listWithToc` 함수를 `BODY` 확장함수로 사용하여 다음처럼 만들 수 있다.
- `block` 내부의 `item` 을 추가하여 `entries` 리스트에 항목을 추가한다.

```kotlin
@HtmlTagMaker
class LISTWITHTOC {
    val entries = mutableListOf<Pair<String, String>>()
    fun item(headline: String, body: String) {
        entries += headline to body
    }
}

fun BODY.listWithToc(block: LISTWITHTOC.() -> Unit) {
    val listWithToc = LISTWITHTOC()
    listWithToc.block()
    ul {
        for ((index, entry) in listWithToc.entries.withIndex()) {
            li { a("#$index") { +entry.first } }
        }
    }
    for ((index, entry) in listWithToc.entries.withIndex()) {
        h2 { id = "$index"; +entry.first }
        p { +entry.second }
    }
}
```
---

### invoke 관례

- 커스텀 타입의 객체를 함수처럼 호출할 수 있다.
- 예전에 배운 관례중 `get` 관례가 있는데, `Foo[bar]` = `foo.get(bar)` 으로 번역이 되었다.
- 비슷하게, `invoke` 는 각괄호 대신 괄호를 사용한다.

```kotlin
class Greeter(val greeting: String) {
    operator fun invoke(name: String) {
        println("$greeting, $name!")
    }
}


fun main() {
    val greeter = Greeter("Babi")
    greeter("Hi")       // 인스턴스를 함수처럼 호출
    // Babi, Hi!
}
```

- 따로 제약사항이 없어서, 원하는 대로 파라미터 갯수 및 타입을 지정할 수 있다.
- 인라인하는 람다를 제외한 모든 람다는 함수형 인터페이스를 구현하는 클래스로 컴파일 된다.

---

#### DSL 과 invoke 관례

- 평평한 함수 호출과, 내포된 블록 구조를 둘다 허용하도록 하고 싶다.
```groovy
// 평평한 함수
dependencies.implementation("core:0.40")

// 내포된 블록 구조
dependencies {
    implementation("core:0.40")
}
```

- 평평한 함수는 `dependencies` 변수에 대해 `implementation `메서드를 정의하면 된다.
- 내포된 블록 구조는 `dependencies` 안에 람다를 받는 `invoke` 메서드를 정의하면 된다.


```kotlin
class DependencyHandler {
    fun implementation(coordinate: String) {    // 일반적인 명령형 API를 정의
        println("Added dependency on $coordinate")
    }

    operator fun invoke(    
        body: DependencyHandler.() -> Unit  // invoke를 정의하여 DSL 스타일의 API 제공
    ) {
        body()  // this가 함수의 수신 객체가 되므로 this.body() 처럼 접근 
    }
}
```

---

### 테스트 프레임워크 should
- 깔끔한 구문은 내부 DSL 의 핵심 특징이다.
- 그리고 kotest 에서 prefix 에 특정 문자열이 존재하는지 검증하려면 다음과 같은 코드로 확인할 수 있다
```kotlin
s should startWithi("K")
```

- 위와같은 문법을 사용하려면 `should` 함수 선언 앞에 `infix` 변경자를 붙여야 한다
```kotlin
infix fun <T> T.should(matcher: Matcher<T>) = matcher.test(this)

```

### 원시 타입에 확장 함수 정의
```kotlin
val now =  Clock.System.now()
val yesterday = now - 1.days
val later = now + 5.hours
```

- 위처럼 사용을 한다면, 구현부는 다음처럼 정리할 수 있다

```kotlin
import java.time.Duration
import kotlin.time.DurationUnit

val Int.days: Duration
    get() = this.toDuration(DurationUnit.DAYS)

val Int.hours: Duration
    get() = this.toDuration(DurationUnit.HOURS)
```

---

### SQL을 위한 내부 DSL
