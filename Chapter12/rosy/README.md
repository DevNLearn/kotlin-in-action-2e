# 12장 어노테이션과 리플렉션

이전까지 살펴본 Kotlin 기능은 컴파일 타임에 클래스와 함수 이름을 정확히 알고 있어야만 사용할 수 있었다.

하지만 실무에서는 코드에 의미(메타데이터)를 부여하거나, 런타임에 클래스 구조를 분석해야 할 때도 많습니다.

→ 이럴 때 어노테이션과 이플렉션이 필요합니다!

---

## 12.1 어노테이션 선언과 적용

### 어노테이션이란?

- 클래스나 함수에 메타데이터를 부여하는 방법
- 어노테이션이 붙은 선언은 → 도구나 라이브러리가 그 정보를 활용할 수 있음.

### 어노테이션을 적용해 선언에 표지 남기기

- @와 어노테이션 이름을 선언 앞에 넣으면 된다.

    ```kotlin
    @Test
    fun testTrue() {
        assertTrue(1 + 1 == 2)
    }
    ```

  → `@Test`는 JUnit에서 해당 메서드를 테스트로 인식하게 해준다.

- 사용 중단 표시: `@Deprecated`

    ```kotlin
    @Deprecated(
        "Use removeAt(index) instead.",
        ReplaceWith("removeAt(index)")
    )
    fun remove(index: Int) { /* ... */ }
    ```

    - `message`: 사용 중단 이유
    - `ReplaceWith`: 대체 패턴 제시 (IDE 퀵 픽스로 대체 가능)
    - `level`: 경고 수준 설정 (WARNING, ERROR, HIDDEN)

- 어노테이션 인자에 쓸 수 있는 것들

    | 허용 타입 | 예시 |
    | --- | --- |
    | 기본 타입, 문자열 | `"text"`, `123` |
    | 클래스 참조 | `MyClass::class` |
    | 어노테이션 | `ReplaceWith(...)` |
    | 배열 | `path = ["/foo", "/bar"]` |
- 모든 인자는 컴파일 시점에 결정 가능해야 함 → `const val`만 가능!

    ```kotlin
    const val TIMEOUT = 10L
    
    @Test
    @Timeout(TIMEOUT)
    fun testTimeout() { ... }
    ```

    - const 변경자를 빼먹었다면 "Only const val can be used in constant expressions" 컴파일 오류 발생

### 어노테이션 타깃 지정하기

Kotlin에서는 **프로퍼티 하나가 여러 Java 요소로 컴파일**될 수 있어, 어디에 어노테이션을 붙일지 명확히 해야 할 때가 있다.

- 사용 지점 타깃 문법
    - 사용 지점 타깃은 @ 기호와 어노테이션 이름 사이에 붙으며 이름과는 콜론(:)으로 분리된다.

```kotlin
@get:JvmName("obtainCertificate")
@set:JvmName("putCertificate")
var certificate: String = "..."
```

- 사용 가능한 타깃 종류

    | 타깃 | 설명 |
    | --- | --- |
    | `property` | 전체 프로퍼티 (자바 어노테이션에는 사용 불가) |
    | `field` | 필드 |
    | `get` / `set` | 게터 / 세터 |
    | `param` | 생성자 파라미터 |
    | `setparam` | 세터의 파라미터 |
    | `receiver` | 확장 함수의 수신 객체 |
    | `delegate` | 위임 인스턴스 필드 |
    | `file` | 파일에 적용되는 어노테이션 (가장 위에 위치해야 함) |
    
    ```kotlin
    @file:JvmName("StringFunctions") // 파일 전체 이름을 변경
    ```
    
    - package 선언보다 더 앞에 넣어서 사용해줘야한다.

- `@Suppress`
    - 컴파일러 경고 무시하기 위한 어노테이션

    ```kotlin
    @Suppress("UNCHECKED_CAST")
    val strings = list as List<String>
    ```

    - 주로 로컬 변수 선언 등 특정 식에 붙여 경고를 무시
    - IDE에서도 Alt + Enter → Suppress로 쉽게 붙일 수 있음
- 자바와 상호 운용을 위한 어노테이션들

| 어노테이션 | 설명 |
| --- | --- |
| `@JvmName` | Java에서 호출 시 이름 변경 |
| `@JvmStatic` | 객체 선언이나 companion object의 메서드를 Java의 static 메서드로 노출 |
| `@JvmOverloads` | 디폴트 파라미터가 있는 함수에 대해 오버로드된 버전 생성 |
| `@JvmField` | 프로퍼티를 getter 없이 Java 필드로 노출 |
| `@JvmRecord` | `data class`를 Java의 `record` 클래스로 컴파일 (Java 16+) |

### 어노테이션으로 JSON 직렬화 방식 제어하기

- 직렬화/역직렬화란?
    - **직렬화**: 객체 → 문자열 (저장/전송용)
    - **역직렬화**: 문자열 → 객체
- Kotlin에서도 객체를 JSON으로 바꾸기 위해 여러 라이브러리가 있다.
    - kotlinx.serialization (Kotlin 전용)
    - Jackson, Gson (Java 기반이지만 Kotlin과 호환)
- 제이키드(JKid) 예제 (https://github.com/Kotlin/kotlin-in-action-2e-jkid)
    - 학습 목적의 간단한 순수 Kotlin JSON 직렬화 라이브러리
    - 이번 장에서는 JKid를 직접 구현하며 어노테이션과 리플렉션을 학습
- Person 클래스의 인스턴스를 직렬화, 역직렬화.

    ```kotlin
    data class Person(val name: String, val age: Int)
    
    fun main() {
        val person = Person("Alice", 29)
        println(serialize(person)) // {"age": 29, "name": "Alice"}
    }
    ```

    - 역직렬화는 타입을 직접 명시해야 한다.

    ```kotlin
    val json = """{"name": "Alice", "age": 29}"""
    println(deserialize<Person>(json))
    // Person(name=Alice, age=29)
    ```

- 어노테이션으로 직렬화 방식을 바꾸기

    | 어노테이션 | 설명 |
    | --- | --- |
    | `@JsonExclude` | 이 프로퍼티는 직렬화/역직렬화 대상에서 제외 |
    | `@JsonName("...")` | 프로퍼티의 JSON 키 이름을 변경 |
    
    ```kotlin
    data class Person(
        @JsonName("alias") val firstName: String,
        @JsonExclude val age: Int? = null
    )
    ```
    
    - `firstName` → `"alias"`라는 이름으로 직렬화
    - `age`는 제외됨 (기본값 반드시 지정!)

### 어노테이션 선언

- 어노테이션 클래스 선언 방법

    ```kotlin
    annotation class JsonExclude
    ```

    - `annotation` 키워드를 붙여서 선언
    - 내부 코드 없음 (본문 정의 불가)
- 파라미터가 있는 어노테이션

    ```kotlin
    annotation class JsonName(val name: String)
    ```

    - 모든 파라미터는 `val`로 선언해야 함
    - 적용 시 일반 생성자 호출처럼 사용
- 자바 비교

    ```kotlin
    public @interface JsonName {
        String value();
    }
    ```

    - 코틀린에서는 `name = "xxx"`처럼 **이름 지정 가능**, 또는 순서만으로 생략도 가능.

### 메타어노테이션: 어노테이션 사용 제어

- 메타어노테이션이란?
    - **어노테이션을 정의하는 어노테이션**
    - 대표 메타어노테이션: `@Target`, `@Retention`

```kotlin
@Target(AnnotationTarget.PROPERTY)
annotation class JsonExclude
```

- 이 어노테이션은 프로퍼티에만 사용 가능함
- 여러 타깃 동시 지정 가능

    ```kotlin
    @Target(AnnotationTarget.PROPERTY, AnnotationTarget.FIELD)
    ```

    - 자바에서 사용하려면 `FIELD`도 추가해야 함
- `@Retention`
    - 어노테이션 정보 유지 시점 지정

    ```kotlin
    @Retention(AnnotationRetention.RUNTIME)
    ```

    - 코틀린은 기본이 RUNTIME → 리플렉션 가능

### 클래스 참조를 어노테이션 인자로 사용하기

- `@DeserializeInterface` 인터페이스 구현체 지정

    ```kotlin
    interface Company { val name: String }
    
    data class CompanyImpl(override val name: String): Company
    
    data class Person(
        val name: String,
        @DeserializeInterface(CompanyImpl::class) val company: Company
    )
    ```

    - 인터페이스는 인스턴스를 만들 수 없으므로, 어떤 클래스를 사용해야 하는지 지정해줘야 함.
- 클래스 참조를 인자로 받는 어노테이션

    ```kotlin
    annotation class DeserializeInterface(val targetClass: KClass<out Any>)
    ```

    - `KClass<out Any>` → 클래스 참조를 받아 저장할 수 있음
    - `out`은 공변성 처리 (모든 하위 타입 허용)

### 제네릭 클래스 참조를 어노테이션 인자로 사용하기

- 제이키드는 기본 타입이 아닌 프로퍼티를 내포된 객체로 직렬화한다.
- ValueSerializer 인터페이스

    ```kotlin
    interface ValueSerializer<T> {
        fun toJsonValue(value: T): Any?
        fun fromJsonValue(jsonValue: Any?): T
    }
    ```

    - ValueSerializer<Date>를 구현하는 DateSerializer를 만들었다고 가정함.
- 커스텀 직렬화 예제

    ```kotlin
    data class Person(
        val name: String,
        @CustomSerializer(DateSerializer::class) val birthDate: Date
    )
    ```

- `@CustomSerializer` 어노테이션 선언

    ```kotlin
    annotation class CustomSerializer(
        val serializerClass: KClass<out ValueSerializer<*>>
    )
    ```

    - `KClass<out ValueSerializer<*>>`: 모든 타입의 ValueSerializer를 허용
    - * 는 **스타 프로젝션** (제네릭 타입 알 수 없을 때 사용) ← 11장에서 나옴!
- 어노테이션에 클래스를 인자로 넘길 때 보통 `MyClass::class`처럼 클래스 참조를 사용한다.
    - 어노테이션에서는 `KClass<out MyClass>` 같은 타입을 사용
    - 만약 클래스가 제네릭이라면 타입 인자를 정확히 할 수 없기 때문에 `KClass<out MyClass<*>>`처럼 작성
      → 컴파일러가 타입을 굳이 몰라도 어노테이션 인자로 안전하게 사용 가능하다.

---

## 12.2 리플렉션: 실행 시점에 코틀린 객체 내부 관찰

- 리플렉션이란?
    - 리플렉션은 **실행 시점에 객체의 클래스 정보, 프로퍼티, 함수에 접근할 수 있는 기능**이다.
    - 보통은 `person.name`처럼 정해진 프로퍼티나 메서드에 접근하지만, 리플렉션을 사용하면 **어떤 클래스인지 몰라도 공통 방식으로 다룰 수 있다.**

      > 예: JSON 직렬화 라이브러리는 다양한 객체를 처리해야 하므로 리플렉션을 필수로 사용함.

- Kotlin 리플렉션 vs Java 리플렉션
    - Kotlin에서는 `kotlin.reflect` 패키지의 API (`KClass`, `KProperty`, `KFunction`)를 주로 사용
    - Java의 `java.lang.reflect`도 사용할 수 있고, **Kotlin 컴파일 결과는 Java 바이트코드이므로 호환됨**
    - Android처럼 용량이 중요한 플랫폼에서는 별도의 `kotlin-reflect` 의존성을 직접 추가해야 함

        ```kotlin
        dependencies {
            implementation("org.jetbrains.kotlin:kotlin-reflect")
        }
        ```


### 리플렉션 API 기초

- 주요 인터페이스

    | 타입 | 설명 |
    | --- | --- |
    | `KClass<T>` | 클래스 정보를 표현 |
    | `KCallable<R>` | 프로퍼티/함수 공통 상위 |
    | `KFunction<R>` | 함수 |
    | `KProperty<T>` | 프로퍼티 |
- Kotlin에서 클래스 자체를 표현할 땐 `KClass`를 사용한다.

    ```kotlin
    val kClass = person::class
    println(kClass.simpleName)         // Person
    kClass.memberProperties.forEach { println(it.name) } // name, age
    ```

- `KCallable`은  함수와 프로퍼티를 아우르는 공통 상위 인터페이스다.

    ```kotlin
    interface KCallable<out R> {
        fun call(vararg args: Any?): R
    }
    ```

    - `call()`을 쓰면 함수도 실행, 프로퍼티도 읽기 가능!

        ```kotlin
        fun foo(x: Int) = println(x)
        
        val f = ::foo
        f.call(42) // 42
        ```

        - 또는 `KFunction2<Int, Int, Int>` 처럼 함수 타입이 명확할 경우 `invoke()`도 가능

        ```kotlin
        val f: KFunction2<Int, Int, Int> = ::sum
        println(f.invoke(1, 2))     // 3
        println(f(3, 4))            // 7
        ```

- 프로퍼티 읽고 쓰기

    ```kotlin
    var counter = 0
    val prop = ::counter
    prop.setter.call(21)
    println(prop.get()) // 21
    ```

    - 멤버 프로퍼티도 `get(instance)`로 접근

    ```kotlin
    val person = Person("Alice", 29)
    val ageProp = Person::age
    println(ageProp.get(person)) // 29
    ```


### 리플렉션으로 객체 직렬화하기

- 전체 흐름

    ```kotlin
    fun serialize(obj: Any): String = buildString {
        serializeObject(obj)
    }
    ```

    - `StringBuilder`를 확장 함수로 사용해 `serializeObject()`에서 값들을 누적함

    ```kotlin
    private fun StringBuilder.serializeObject(obj: Any) {
        val kClass = obj::class as KClass<Any>
        val properties = kClass.memberProperties
        properties.joinTo(this, prefix = "{", postfix = "}") { prop ->
            serializeString(prop.name)
            append(": ")
            serializePropertyValue(prop.get(obj))
        }
    }
    ```

    - 클래스의 모든 프로퍼티를 가져와서 이름과 값을 JSON 형식으로 직렬화
    - prop.get(obj) → 리플렉션을 통해 실제 값을 꺼냄

### 어노테이션으로 직렬화 제어하기

- 이제 리플렉션으로 얻은 프로퍼티에 **@JsonExclude, @JsonName, @CustomSerializer** 등의 어노테이션을 적용해 제어할 수 있다.
- `@JsonExclude`으로 직렬화에서 제외하고 싶을 때 적용

    ```kotlin
    val properties = kClass.memberProperties
        .filter { it.findAnnotation<JsonExclude>() == null }
    ```

    - 해당 어노테이션이 붙은 프로퍼티는 JSON에서 제외
- `@JsonName` 으로 인자 프로퍼티 직렬화하여 JSON에 넣을 때 사용할 이름 처리

    ```kotlin
    val jsonNameAnn = prop.findAnnotation<JsonName>()
    val propName = jsonNameAnn?.name ?: prop.name
    ```

    - 어노테이션이 있다면 지정한 이름(`alias` 등), 없으면 원래 프로퍼티 이름 사용
- `@CustomSerializer` 처리
    - 직렬화할 때 사용자 지정 serializer 사용

    ```kotlin
    annotation class CustomSerializer(
        val serializerClass: KClass<out ValueSerializer<*>>
    )
    
    val value = prop.get(obj)
    val jsonValue = prop.getSerializer()?.toJsonValue(value) ?: value
    ```

    - `getSerializer()`는 objectInstance나 createInstance()로 인스턴스를 가져옴
- 전체 직렬화 흐름

    ```kotlin
    private fun StringBuilder.serializeProperty(prop: KProperty1<Any, *>, obj: Any) {
        val jsonNameAnn = prop.findAnnotation<JsonName>()
        val propName = jsonNameAnn?.name ?: prop.name
    
        serializeString(propName)
        append(": ")
    
        val value = prop.get(obj)
        val jsonValue = prop.getSerializer()?.toJsonValue(value) ?: value
        serializePropertyValue(jsonValue)
    }
    ```

    - 특정 객체의 프로퍼티 하나를 JSON 키-값 형태로 문자열로 직렬화
    - 어노테이션(`@JsonName`, `@CustomSerializer`)을 고려해 직렬화 방식을 유연하게 적용
    - 최종적으로 `StringBuilder`에 `"key": value` 형태의 문자열을 만들어 추가함
    - `findAnnotation<JsonName>()`을 통해 어노테이션이 있다면 해당 이름으로 키를 설정하고, 없다면 기본 프로퍼티 이름 사용
    - `getSerializer()`를 통해 커스텀 직렬화기가 지정되어 있다면 해당 직렬화 방식으로 값 변환, 없다면 일반 값 그대로 사용
    - `serializePropertyValue()`는 값의 타입에 따라 적절한 JSON 표현으로 직렬화함 (`String`, `Number`, `Boolean`, `List`, 중첩 객체 등)

### JSON 파싱과 객체 역직렬화

- 역직렬화란?
    - **역직렬화는 JSON 문자열을 객체로 다시 되돌리는 과정**
    - 직렬화보다 복잡한 이유는 클래스와 프로퍼티 정보를 리플렉션으로 분석해서 새로운 객체를 만들어야 하기 때문이다.
- 역직렬화 함수의 형태

    ```kotlin
    inline fun <reified T: Any> deserialize(json: String): T
    ```

    - `inline` + `reified`는 실행 시점에도 타입 정보(T)를 알 수 있게 해준다.
    - 이렇게 해야 JSON → 객체 변환이 가능함
- 처리 단계: 3단계 구조
    1. **Lexer(렉서)**: JSON 문자열을 토큰으로 나눔 (예: `{`, `"title"`, `42` 등)
    2. **Parser(파서)**: 토큰을 구조화된 키-값으로 분석
    3. **역직렬화기**: 파싱된 값으로 실제 객체를 생성
- JsonObject 인터페이스

    ```kotlin
    interface JsonObject {
        fun setSimpleProperty(name: String, value: Any?)
        fun createObject(name: String): JsonObject
        fun createArray(name: String): JsonObject
    }
    ```

    - 파서는 이 인터페이스를 호출하면서 **객체 구조를 구성한다.**
    - `setSimpleProperty`: 간단한 값 (문자열, 숫자 등)
    - `createObject` / `createArray`: 중첩 객체나 리스트
- Seed: 생성될 객체 정보를 저장하는 구조
    - **Seed = 객체가 생성되기 전 상태(씨앗)**
    - `ObjectSeed`, `ValueListSeed`, `ObjectListSeed` 등 다양한 형태 존재
    - `spawn()` 메서드를 호출하면 실제 객체가 생성!
- 역직렬화의 흐름

    ```kotlin
    fun <T: Any> deserialize(json: Reader, targetClass: KClass<T>): T {
        val seed = ObjectSeed(targetClass, ClassInfoCache())
        Parser(json, seed).parse()
        return seed.spawn()
    }
    ```

    - JSON → 파싱 → Seed에 값 누적 → spawn() 호출 → 객체 생성
- ObjectSeed 내부 구조

    ```kotlin
    class ObjectSeed<T: Any>(...) : Seed {
        val valueArguments = mutableMapOf<KParameter, Any?>()
        val seedArguments = mutableMapOf<KParameter, Seed>()
        override fun spawn(): T = classInfo.createInstance(arguments)
    }
    ```

    - `valueArguments`: 단순 값들 저장
    - `seedArguments`: 중첩 객체용 시드 저장
    - spawn 시점에 재귀적으로 값을 생성해 최종 객체를 만들어냄

### callBy()와 리플렉션을 사용해 객체 만들기

- `KCallable.callBy`를 활용한 객체 생성
    - `call()`은 기본값을 무시하지만
    - `callBy()`는 **기본값과 널 허용 여부까지 자동 처리**해준다.

        ```kotlin
        constructor.callBy(argumentsMap)
        ```

    - `argumentsMap`은 `KParameter`를 키로 하고, 각 파라미터 값이 들어가야 한다.
- 타입 일치와 변환
    - JSON의 값이 객체의 파라미터 타입과 일치하지 않으면 예외가 발생한다.
    - 숫자 → Int / Long / Double, Boolean 등 구분 필요하다.
    - 이걸 도와주는 게 `ValueSerializer`

    ```kotlin
    fun serializerForType(type: KType): ValueSerializer<out Any?>? = ...
    ```

    - `@CustomSerializer`가 있으면 그걸 사용하고, 없으면 타입에 따라 기본 serializer 사용
- `BooleanSerializer` : Booelan 값을 위한 직렬화기

    ```kotlin
    object BooleanSerializer : ValueSerializer<Boolean> {
        override fun fromJsonValue(jsonValue: Any?) = 
            if (jsonValue is Boolean) jsonValue
            else throw Exception("Boolean expected")
    }
    ```

    - 타입 검증 후 적절히 변환해줌
- 캐시 클래스: `ClassInfoCache`
    - 리플렉션은 비용이 크므로 결과를 **ClassInfo**에 미리 저장해 재사용
    - 파라미터 이름 ↔ KParameter ↔ 어노테이션 정보 맵핑

    ```kotlin
    class ClassInfo<T> {
        val jsonNameToParam: Map<String, KParameter>
        val paramToSerializer: Map<KParameter, ValueSerializer<*>>
        ...
    }
    ```

    - 생성 시 필요한 정보는 여기서 다 꺼냄
- 필수 파라미터 확인

    ```kotlin
    private fun ensureAllParametersPresent(arguments: Map<KParameter, Any?>) {
        for (param in constructor.parameters) {
            if (param !in arguments && !param.isOptional && !param.type.isMarkedNullable) {
                throw Exception("Missing value for ${param.name}")
            }
        }
    }
    ```

    - **기본값이 없고 null도 안 되는 파라미터**는 반드시 JSON에 포함되어야 함

---

## 요약

- 코틀린에서 어노테이션을 적용할 때는 aryAnotation(parans) 구문을 사용한다.
- 코틀린에서는 파일과 식 등 넓은 범위의 타깃에 대해 어노테이션을 붙일 수 있다.
- 어노테이션 인자로 기본 타입 값, 문자열, 이념, 클래스 참조, 다른 어노테이 션 클래스의 인스턴스, 배열을 사용할 수 있다.
- @get : JvmName에서처럼 어노테이션의 사용 지점 타깃을 명시하면 하나의 코 틀린 선언이 여러 가지 바이트코드 요소를 만들어내는 경우 정확히 어떤 부분에 어노테이션을 적용할지 지정할 수 있다.
- 어노테이션 클래스를 정의할 때는 annotation class로 시작한다. 이 클래스 는 모든 파라미터를 val 프로퍼티로 표시한 주 생성자가 있어야 하고, 분문은 없어야 한다.
- 메타어노테이션을 사용해 타지, 어노테이션 유지 모드 등 여러 어노테이션 특성을 지정할 수 있다.
- 리플렉션 AP를 통해 실행 시점에 객체의 메서드와 프로퍼티를 동적으로 열거하고 접근할 수 있다. 리플렉션 AP에는 클래스, 함수 등 여러 종류의 선언을 표현하는 인터페이스가 들어있다.
- 클래스의 경우 KLass 인스턴스를 얻기 위해 ClassName::clas를 사용한 다. 객체로부터 KClass 인스턴스를 얻으려면 objName::class 를 사용한다.
- KFunction과 KProperty 인터페이스는 모두 KCallable을 확장한다. KCallable 은 제네릭 call 메서드를 제공한다.
- KCallable.callBy 메서드를 사용하면 메서드를 호출하면서 디폴트 파라미 터 값을 사용할 수 있다.
- KFunction0, KFunction1 등의 인터페이스는 모두 파라미터 개수가 다른 함 수를 표현하며 invoke 메서드를 사용해 함수를 호출할 수 있다.
- KProperty0, KProperty1은 수신 객체의 개수가 다른 프로퍼티들을 표현하며 값 을 얻기 위한 get 메서드를 지원한다. KMutablePropersy0과 KMutableProperty1 은 각각 KProperty0과 KProperty1을 확장하며 set 메서드를 통해 프로퍼티 값을 변경할 수 있다.
- KType의 실행 시점 표현을 얻기 위해 typeOf<T>() 함수를 사용한다.