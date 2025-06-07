# 12. 어노테이션과 리플렉션

## 어노테이션 선언과 적용

어노테이션을 사용하면 선언에 추가적인 메타데이터를 연관시킬 수 있다. <br />
그 후 어노테이션이 설정된 방식에 따라 메타데이터를 `소스코드`, `컴파일된 클래스 파일`,<br />
`런타임에 대해 작동하는 도구`를 통해 접근할 수 있다. 

### 어노테이션을 적용해 선언에 표지 남기기

코틀린에서는 어노테이션을 적용하려면 @와 어노테이션 이름을 선언 앞에 넣으면 된다.

```kotlin
class MyTest {
    @Test
    fun testTrue() {
        assertTrue(1 + 1 == 2)
    }
}
```

```kotlin
@Deprecated(
    level = DeprecationLevel.WARNING, 
    message = "Use removeAt(index) instead.", 
    replaceWith = ReplaceWith("removeAt(index)")
)
fun remove(index: Int) { /* ... */ }
```

어노테이션 인자를 지정하는 문법은 자바와 약간 다르다.

- 클래스를 어노테이션 인자로 지정: `@MyAnnotation(MyClass::class)` 처럼 클래스 이름 뒤에 `::class`를 붙인다.
- 다른 어노테이션을 인자로 지정: 이름 앞에 @를 붙지지 않는다. `@MyAnnotation(OtherAnnotation)` 처럼 사용한다.<br />
위에서 살펴본 `@Deprecated`의 `ReplaceWith` 도 어노테이션이다.
- 배열을 인자로 지정: `@RequestMapping(path = ["/foo", "/bar"]) `처럼 대괄호로 감싼다. <br />
임의의 프로퍼티를 인자로 사용하려면 `const` 변경자가 붙은 상수만 프로퍼티로 사용할 수 있다.

### 어노테이션이 참조할 수 있는 정확한 선언 지정: 어노테이션 타깃
어노테이션을 붙일때 어떤 요소에 어노테이션을 붙일지 표시하는 기능을 어노테이션 타깃`(annotation target)`이라고 한다. <br />
`사용 지점 타깃` 선언을 통해 어노테이션을 붙일 요소를 정할 수 있다.

```kotlin
// @get이 사용 지점 타깃
// JvmName이 어노테이션 이름
@get:JvmName("obtainCertificate")
```
- `property`: 프로퍼티 전체 (자바에서 선언된 어노테이션으로는 사용 못함)
- `field`: 프로퍼티에 의해 생성되는 필드
- `get`: 프로퍼티 게터
- `set`: 프로퍼티 세터
- `receiver`: 확장 함수나 프로퍼티의 수신 객체 파라미터
- `param`: 생성자 파라미터
- `setparam`: 세터 파라미터
- `delegate`: 위임 프로퍼티의 위임 인스턴스를 담아둔 필드
- `file`: 파일 안에 선언된 최상위 함수와 프로퍼티를 담아두는 클래스

### 어노테이션을 활용해 JSON 직렬화 제어

직렬화는 객체를 저장 장치에 저장하거나 네트워크를 통해 전송할 수 있는 형식으로 변환하는 과정이다. <br />
반대과정인 역직렬화는 저장된 데이터를 다시 객체로 변환하는 과정이다. <br />

```kotlin
import kotlinx.serialization.Serializable
import kotlinx.serialization.json.Json

@Serializable
data class Person(
    val name: String,
    val age: Int,
)

fun main() {
    val person = Person("Alice", 29)
    println(Json.encodeToString(person))
    // {"name":"Alice","age":29}
}
```

```kotlin
@Serializable
data class Person(
    val name: String,
    val age: Int,
)

fun main() {
    val jsonData = """{"name":"Alice","age":29}"""
    println(Json.decodeFromString<Person>(jsonData))
    // Person(name=Alice, age=29)
}
```

### 어노테이션 선언

어노테이션 선언은 `annotation class` 키워드를 사용한다. <br />
```kotlin
annotation class JsonExclude
``` 
처럼 선언한다. 하지만 어노테이션 클래스는 선언이나 식과 관련있는 메타데이터의 구조만 정의하기 때문에 <br />
내부에 아무코드도 들어있을 수 없다. <br />

파라미터가 있는 어노테이션을 정의하려면 일반적인 주 생성자 구문을 사용하면서 모든 파라미터를 `val`로 선언한다. <br />
```kotlin
annotation class JsonName(val name: String)
```

### 메타어노테이션: 어노테이션을 처리하는 방법 제어

자바와 마찬가지로 코틀린 어노테이션 클래스에도 어노테이션을 붙일 수 있다. <br />
어노테이션 클래스에 적용할 수 있는 어노테이션을 `메타어노테이션`이라 부른다. <br />
표준 라이브러리에는 여러 메타어노테이션이 있으며 그런 메타 어노테이션들은<br /> 
컴파일러가 어노테이션을 처리하는 방법을 제어한다.

가장 많이 사용되는 어노테이션은 적용 가능한 타깃을 지정하는 `@Target`이다. 

```kotlin
@Target(AnnotationTarget.PROPERTY)
annotation class JsonExclude
```

`@Target`을 지정하지 않으면, 모든 선언에 적용할 수 있는 어노테이션이 된다. <br />
`@Target`은 `AnnotationTarget` 열거형을 인자로 받는다. <br />

### 어노테이션 파라미터로 클래스 사용

어떤 클래스를 선언 메타데이터로 참조할 수 있는 기능이 필요할 때도 있다. <br />
클래스 참조를 파라미터로 하는 어노테이션 클래스를 선언하면 그런 기능을 사용할 수 있다.

제이키드 라이브러리에 있는 `@DeserializeInterface` 는 인터페이스 타입인 프로퍼티에 대한<br />
역직렬화를 제어할 때 쓰는 어노테이션이다. 인터페이스의 인스턴스를 직접 만들 수는 없다. <br />
따라서, 역직렬화 시 어떤 클래스를 사용해 인터페이스를 구현할지를 지정할 수 있어야한다.

```kotlin
interface Company {
    val name: String
}

data class CompanyImpl(override val name: String) : Company

data class Person(
    val name: String,
    @DeserializeInterface(CompanyImpl::class) val company: Company
)
```

`@DeserializeInterface` 의 구조는 아래와 같이 되어있고, `CompanyImpl::class` 를 넣었으니 `KClass<CompanyImpl>`이 들어간다.<br />
`CompanyImpl`은 `Any`의 하위 클래스이므로 `out` 을 사용해 `Any`의 하위 클래스를 받을 수 있도록 설정했다.

```kotlin
annotation class DeserializeInterface(
    val impl: KClass<out Any>
)
```

## 리플렉션: 실행 시점에 코틀린 객체 내부 관찰

어노테이션에 저장된 데이터에 접근하려면 리플렉션을 사용해야 한다. <br />
리플렉션은 실행 시점에 코틀린 객체의 구조를 관찰하고 조작할 수 있는 기능이다.

타입과 관계없이 객체를 다뤄야 하거나 객체가 제공하는 메서드나 프로퍼티 이름을 실행 시점에만 알 수가 있는 경우가 있다.<br/>
대표적인 예로, 직렬화 라이브러리의 경우 어떤 객체든 JSON으로 직렬화 할 수 있어야하기 때문에 리플렉션을 사용한다.

코틀린에서 리플렉션을 다루기 위해선 리플렉션 API를 다루면 된다. <br />
이 API는 코틀린 표준 라이브러리의 `kotlin.reflect` 패키지에 있다. <br />


### 코틀린 리플렉션 API: `KClass`, `KCallable`, `KFunction`, `KProperty`

- KClass: 코틀린 클래스에 대한 정보를 담고 있는 객체
  - `MyClass::class`로 KClass의 인스턴스를 얻을 수 있다.
  - `myClass::class`로 인스턴스 객체도 KClass를 얻을 수 있다.
  - 익명 객체의 경우 `simpleName`과 같은 함수를 사용하면 `null`을 반환한다.

```kotlin
import kotlin.reflect.full.memberProperties

class Person(val name: String, val age: Int)

fun main() {
    val person = Person("Alice", 29)
    val kClass = person::class

    println(kClass.simpleName) // Person
    kClass.memberProperties.forEach { println(it.name) }
    // age
    // name
}
```

`KClass`에 존재하는 모든 멤버의 컬렉션인 `members`는 `KCallable`의 `Collection`이다. <br />
- `KCallable`: 함수와 프로퍼티를 아우르는 공통 인터페이스로 call 을 사용하면 함수나 프로퍼티의 게터를 호출할 수 있다.
```kotlin
interface KCallable<out R> {
    fun call(vararg args: Any?): R
    // ...
}
```

- `KFunction`: 함수에 대한 정보를 담고 있는 인터페이스로 `KCallable`을 상속받는다.
  - `KFunction`은 함수의 이름, 파라미터, 반환 타입 등을 제공한다.
  - `call` 메서드를 사용해 함수를 호출할 수 있다.

```kotlin
fun foo(x: Int) = println(x)

fun main() {
    val kFunction = ::foo
    kFunction.call(42)
    // 42
}
```

`call`을 호출할때에는 `vararg`이기 때문에 호출하려는 함수에 정의된 파라미터 개수가 맞아 떨어져야한다.<br />
하지만, 함수를 호출하기 위해 좀더 구체적인 메서드를 사용할 수 있다.

`KFunction1, 2`와 같은 인터페이스를 통해 함수의 파라미터가 몇개인지 명시적으로 나타낼 수 있다. <br />
호출은 `invoke` 메서드를 사용한다. <br />
```kotlin
fun sum(a: Int, b: Int): Int {
    return a + b
}

fun main() {
    val kFunction = ::sum // KFunction2<Int, Int, Int>
    println(kFunction.invoke(1, 2) + kFunction(3, 4))
    // 10
}
```

`call` 메서드는 타입 안정성이 떨어지고, 인자 개수도 맞지 않아도 컴파일에 성공되기 때문에 <br />
가능하면 `invoke`를 사용하는 것이 좋다. <br />

- `KProperty`: 프로퍼티에 대한 정보를 담고 있는 인터페이스로 `KCallable`을 상속받는다.
  - 프로퍼티의 이름, 타입, 게터와 세터 등을 제공한다.
  - `get` 메서드를 사용해 프로퍼티 값을 읽을 수 있다.
  - 객체의 프로퍼티 값을 얻으려면 프로퍼티 참조를 저장한 뒤, `get`에 인스턴스를 넣어주면 값을 뽑아올 수 있다.

```kotlin
var counter = 0

fun main() {
    val kProperty = ::counter
    kProperty.setter.call(21)
    println(kProperty.get()) // 21
}
```
```kotlin
class Person(val name: String, val age: Int)

fun main() {
    val person = Person("John", 30)
    val memberProperty = Person::age
    println(memberProperty.get(person)) // 30
}
```

### 리플렉션을 사용해 객체 직렬화 구현

직렬화 함수는 객체의 모든 프로퍼티를 직렬화한다. <br />
- 원시 타입이나 문자열은 적절히 `JSON 수`, `불리언`, `문자열 값` 등으로 변환된다.
- 컬렉션은 JSON 배열로 직렬화된다.
- 원시 타입이나 문자열, 컬렉션이 아닌 다른 타입인 프로퍼티는 내포된 JSON 객체로 직렬화된다.

```kotlin
private fun StringBuilder.serializeObject(obj: Any) {
    val kClass = obj::class as KClass<Any>
    val properties = kClass.memberProperties

    properties.joinToStringBuilder(
        this, prefix = "{", postfix = "}") { prop ->
        serializeString(prop.name)
        append(": ")
        serializePropertyValue(prop.get(obj))
    }
}
```

### 어노테이션을 활용해 직렬화 제어

`@JsonExclude`, `@JsonName`, `@CustomSerializer` 가 `serializeObject`에서 어떻게 처리되는지 살펴보면,

- `@JsonExclude` 어노테이션이 붙은 프로퍼티는 직렬화에서 제외한다.
- 아래와 같은 방식으로 프로퍼티에서 JsonExclude 어노테이션이 붙지 않은 프로퍼티만 남기면 된다.

```kotlin
val properties = kClass.memberProperties.filter { it.findAnnotation<JsonExclude>() == null }
```

- `@JsonName` 어노테이션이 붙은 프로퍼티는 직렬화 시 이름을 변경한다.
- 아래와 같이 프로퍼티에 `@JsonName` 어노테이션이 붙은 경우, `@JsonName`의 메타데이터에서 이름을 추출해 해당 이름으로 직렬화한다.

```kotlin
// JsonName 선언부
annotation class JsonName(val name: String)

data class Person(
    @JsonName("alias") val firstName: String,
    val age: Int
)

//
val jsonNameAnn = prop.findAnnotation<JsonName>()
val propName = jsonNameAnn?.name ?: prop.name
```

위 두 내용을 모두 포함한 로직을 작성해보면, 아래와 같다.

```kotlin
import kotlin.reflect.KProperty1
import kotlin.reflect.full.findAnnotation

private fun StringBuilder.serializeProperty(
  prop: KProperty1<Any, *>, obj: Any,
) {
    val jsonNameAnn = prop.findAnnotation<JsonName>()
    val propName = jsonNameAnn?.name ?: prop.name
    serializeString(propName)
    append(": ")
    serializePropertyValue(prop.get(obj))
}

private fun StringBuilder.serializeObject(obj: Any) {
    (obj::class as KClass<Any>) 
        .memberProperties
        .filter { it.findAnnotation<JsonExclude>() == null }
        .joinToStringBuilder(this, prefix = "{", postifx = "}") {
            serializeProperty(it, obj)
        }
}
```

- `@CustomSerializer` 어노테이션이 붙은 프로퍼티는 직렬화 시 커스텀 직렬화 로직을 사용한다.
- 아래와 같이 프로퍼티에 `@CustomSerializer` 어노테이션이 붙은 경우, <br />
`@CustomSerializer`의 메타데이터에서 직렬화 함수를 추출해 해당 함수로 직렬화한다.<br />
`birthDate` 프로퍼티를 직렬화하면서 `getSerializer()`를 호출하면 `DateSerializer`가 반환된다. <br />

`getSerializer()` 함수는 `findAnnotation` 함수를 호출해서 `@CustomSerializer` 어노테이션의 인스턴스가 <br />
있는지 찾는다. 그 어노테이션의 `serializerClass`가 직렬화기 인스턴스를 얻기 위해 사용해야할 클래스다.

```kotlin
import java.util.Date

data class Person(
    val name: String,
    @CustomSerializer(DateSerializer::class) val birthDate: Date
)
```
```kotlin
fun KProperty<*>.getSerializer(): ValueSerializer<Any?>? {
    val customSerializerAnn = findAnnotation<CustomSerializer>() ?: return null
    val serializerClass = customSerializerAnn.serializerClass
    val valueSerializer = serializerClass.objectInstance // object 로 Serializer를 구현한 경우, null이 아님
      ?: serializerClass.createInstance()
    @Suppress("UNCHECKED_CAST")
    return valueSerializer as ValueSerializer<Any?>
}
```

```kotlin
private fun StringBuilder.serializeProperty(
    prop: KProperty1<Any, *>, obj: Any,
) {
    val jsonNameAnn = prop.findAnnotation<JsonName>()
  val propName = jsonNameAnn?.name ?: prop.name
    serializeString(propName)
    append(": ")

    // 커스텀 직렬화기 사용
    val value = prop.get(obj)
    val jsonValue = prop.getSerializer()?.toJsonValue(value)
        ?: value
    serializePropertyValue(jsonValue)
}
```

### JSON 파싱과 객체 역직렬화

역직렬화 시 올바른 결과를 만들어내려면 실행 시점에 타입 파라미터에 접근해야 한다. <br />
이는 타입 파라미터에 `reified` 를 붙여야 한다는 뜻이고, 그로 인해 결국 함수를 `inline`으로 선언해야하만 한다.

```kotlin
inline fun <reified T: Any> deserialize(json: String): T
```

```kotlin
data class Author(val name: String)
data class Book(val title: String, val author: Author)

fun main() {
    val json = """{"title": "Effective Kotlin", "author": {"name": "Marcin Moskala"}}"""
    val book = deserialize<Book>(json)
    println(book)
    // Output: Book(title=Effective Kotlin, author=Author(name=Marcin Moskala))
}
```

JSON 역직렬화기는 흔히 쓰는 방법에 따라 3단계로 구현돼 있다.
- 어휘 분석기 (lexical analyzer) -> 렉서 (lexer) : 입력 문자열을 토큰의 리스트로 변환
- 문법 분석기 (syntax analyzer) -> 파서 (parser) : 토큰의 리스트를 구조화된 표현으로 변환
- 객체 생성기 : 파싱한 결과로 객체를 생성하는 컴포넌트

JsonObject 인터페이스는 현재 역직렬화하는 중인 객체나 배열을 추적한다.<br />
파서는 현재 객체의 새로운 프로퍼티를 발견할 때마다 그 프로퍼티에 해당하는 JsonObject의 함수를 호출한다.

```kotlin
interface JsonObject {
    fun setSimpleProperty(propertyName: String, value: Any?)
    fun createObject(propertyName: String): JsonObject
    fun createArray(propertyName: String): JsonObject
}
```

```kotlin
// Json 문자열 변환 과정
{"title": "Catch-22", "author": {"name": "Joseph Heller"}}

// 🔽 렉서: Json을 토큰으로 나눈다.
|{|"title"|:|"Catch-22"|,|"author"|:|{|"name"|:|"Joseph Heller"|}|}|

// 🔽 파서: 여러 다른 의미 단위를 처리한다.
o1.setSimpleProperty("title", "Catch-22")
val o2 = o1.createObject("author")
o2.setSimpleProperty("name", "Joseph Heller")

// 🔽 역직렬화기: 필요한 클래스의 인스턴스를 생성해 반환한다.
Book("Catch-22", Author("Joseph Heller"))
```

### JSON 역직렬화 구현

역직렬화의 경우 해법이 완전히 제네릭해야 한다.
`시드`라는 단어로 `JsonObject`을 구현하는 클래스를 만들고, <br />
JSON에서는 객체, 컬렉션, 맵과 같은 복합 구조를 만들 필요가 있기 때문에 <br />
각각 `ObjectSeed`, `ObjectListSeed`, `ValueListSeed` 로 값을 만들예정.

```kotlin
interface Seed : JsonObject {
    fun spawn(): Any?
    fun createCompositeProperty(
        propertyName: String,
        isList: Boolean
    ): JsonObject
    override fun createObject(propertyName: String): JsonObject {
        return createCompositeProperty(propertyName, isList = false)
    }
    override fun createArray(propertyName: String): JsonObject {
        return createCompositeProperty(propertyName, isList = true)
    }
    // ...
}
```

`spawn()` 메서드는 만들어낸 객체를 생성한다. <br />
이 메서드는 `ObjectSeed`의 경우 생성된 객체를 반환하고, `ObjectListSeed`, `ValueListSeed`의 경우 <br />
생성된 객체의 리스트를 반환한다. <br />

```kotlin
// 최상위 역직렬화 함수
fun <T : Any> deserialize(json: Reader, targetClass: KClass<T>): T {
    val seed = ObjectSeed(targetClass, ClassInfoCache())
    Parser(json, seed).parse()
    return seed.spawn()
}
```

객체 직렬화 파싱 과정:

1. **ObjectSeed 생성** - 직렬화할 객체의 프로퍼티 저장
2. **파서 호출** - json(입력 스트림 리더)과 시드를 인자로 전달
3. **결과 생성** - 입력 데이터 끝에서 spawn() 함수로 최종 객체 생성

ClassInfoCache는 클래스의 프로퍼티 정보를 캐싱하는 역할을 한다. <br />
이 캐시 정보를 사용해서 클래스의 인스턴스를 만든다.

```kotlin
class ObjectSeed<out T: Any>(
        targetClass: KClass<T>,
        override val classInfoCache: ClassInfoCache
) : Seed {
    // targetClass의 인스턴스를 만들 때 필요한 정보를 캐시한다.
    private val classInfo: ClassInfo<T> = classInfoCache[targetClass]

    private val valueArguments = mutableMapOf<KParameter, Any?>()
    private val seedArguments = mutableMapOf<KParameter, Seed>()

    private val arguments: Map<KParameter, Any?> // 생성자 파라미터와 그 값을 연결하는 맵을 만듬
        get() = valueArguments + seedArguments.mapValues { it.value.spawn() }

    override fun setSimpleProperty(propertyName: String, value: Any?) {
        val param = classInfo.getConstructorParameter(propertyName)
        // 생성자 파라미터 값이 간단한 값인 경우 그 값을 기록
        valueArguments[param] = classInfo.deserializeConstructorArgument(param, value)
    }

    override fun createCompositeProperty(propertyName: String, isList: Boolean): Seed {
        val param = classInfo.getConstructorParameter(propertyName)
        // 프로퍼티에 대한 DeserializeInterface 어노테이션이 있다면 그 값을 가져온다.
        val deserializeAs = classInfo.getDeserializeClass(propertyName)
        val seed = createSeedForType(
                deserializeAs ?: param.type.javaType, isList)
        return seed.apply { seedArguments[param] = this }
    }

    override fun spawn(): T = classInfo.createInstance(arguments)
}
```

- `ObjectSeed`는 `Seed` 인터페이스를 구현하며, <br />
  `targetClass`의 생성자 파라미터를 기반으로 프로퍼티를 저장한다.
- `setSimpleProperty` 메서드는 간단한 프로퍼티 값을 저장한다.
- `createCompositeProperty` 메서드는 복합 프로퍼티를 위한 `Seed`를 생성한다. <br />
  이때, `DeserializeInterface` 어노테이션이 있다면 해당 클래스를 사용해 역직렬화한다.
- `spawn` 메서드는 저장된 프로퍼티 값을 사용해 `targetClass`의 인스턴스를 생성한다. <br />
  이때, `classInfo.createInstance(arguments)`를 호출해 생성자 파라미터와 값을 연결한다.
- `classInfo`는 `ClassInfoCache`에서 가져온 클래스 정보를 사용해 <br />
  생성자 파라미터를 찾고, 역직렬화할 때 필요한 정보를 제공한다.
- `createSeedForType` 함수는 주어진 타입에 맞는 `Seed`를 생성한다. <br />
  이 함수는 타입이 리스트인지 아닌지에 따라 적절한 `Seed`를 반환한다.
- `deserializeAs`는 `DeserializeInterface` 어노테이션이 있는 경우, <br />
  해당 클래스를 사용해 역직렬화할 수 있도록 한다. <br />
  만약 어노테이션이 없다면, 기본적으로 `param.type.javaType`을 사용한다.
- `valueArguments`와 `seedArguments`는 각각 간단한 값과 복합 객체를 저장하는 맵이다. <br />
  이 두 맵을 합쳐서 최종적으로 생성자 파라미터와 값을 연결하는 `arguments` 맵을 만든다. <br />
- `arguments` 맵은 `KParameter`와 그에 해당하는 값을 연결한다. <br />
  이 맵을 사용해 `classInfo.createInstance(arguments)`를 호출하여 최종 객체를 생성한다.

### callBy()와 리플렉션을 사용해 객체 만들기

앞서 `KCallable.call()` 메서드를 사용해 함수를 호출하는 방법을 살펴봤다. <br />
`KCallable.call`은 디폴트 파라미터를 지원하지 않는다는 한계가 있다.

제이키드에서 역직렬화 시 생성할 객체에 디폴트 생성자 파라미터 값이 있는데도 JSON에서 관련 프로퍼티를 꼭 지정하게 하고 싶지는 않을 것이다.
따라서 디폴트 파라미터 값을 지원하는 다른 메서드인 `KCallable.callBy()`를 사용한다. <br />

-> JSON에 없는 필드도 객체에 기본값이 설정되어있다면, 해당 기본값으로 설정할 수 있게 해준다.

```kotlin
data class User(
    val name: String,
    val age: Int = 25  // 기본값
)
```

`KParameter.type` 프로퍼티를 활용하면 파라미터의 타입을 알 수 있다. <br />
실행 시점에 타입을 얻기 위해 `typeOf<>()` 함수를 사용해 `KType` 인스턴스를 얻을 수 있다.

```kotlin
fun serializerForType(type: Type): ValueSerializer<Any?>? =
    when (type) {
        typeOf<Byte>() -> ByteSerializer
        typeOf<Int>() -> IntSerializer
        typeOf<Boolean>() -> BooleanSerializer
        // ..
        else -> null
    }
```
