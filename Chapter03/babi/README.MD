## 3장

### 코틀린에서 컬렉션 만들기
- 코틀린은 표준 자바 컬렉션 클래스를 사용한다
- 간단하게 컬렉션들을 생성하는 예제로 다음이 있다 
```kotlin
// set 생성
val set = setOf(1, 2)
// list 생성
val list = listOf(1, 2)
// map 생성
val map = mapOf(1 to "one", 2 to "two")
```

---

- 참고로 `Kotlin` 에서는 `List` 와 `MutableList` 를 구분해서 사용하게 되는데 해당 클래스를 자바클래스로 찍어보면 다음과 같이 나온다
```kotlin
val list = listOf()
println(list.javaClass)
// class java.util.Arrays$ArrayList

val mutableList = mutableListOf()
println(mutableList.javaClass)
// class java.util.ArrayList

```

---

### 함수를 호출하기 쉽게 만들기

#### 파라미터 전달시 이름 붙이기
- 파라미터에 이름을 붙일수 있다.
- 책 예시로 나온것처럼, 자바에서는 전달해야하는 파라미터가 많아질수록 어떤 데이터인지 구분하기 힘들뿐더러 순서를 잘못 적는 경우도 생길 수 있다.
```java
private String addMember(String firstName, String lastName, String address, String phone, String note){
    // 사용자 추가 로직
}

// 호출할때는 어떤 값인지 확인하기 힘들다
addMember("bab", "e", "zip", "010", "climbing");
```
- 하지만 코틀린에서는 파라미터의 이름을 직접 기재할 수 있다. (안해도 된다)
- 파라미터 이름으로 맵핑이 되는 방식이니 만큼, 이름을 기재한다면 순서와는 상관이 없다
```kotlin
private fun addMember(firstName: String, lastName: String, address: String, phone: String, note: String){
    // 사용자 추가 로직
}

// 호출할때도 헷갈리지 않는다!
addMember(
        firstName = "bab",
        lastName = "e",
        address = "zip",
        phone = "010",
        note = "climbing"
)
```

---

#### 디폴트 파라미터
- 개인적으로 테스트코드 작성시 가장 편리하게 사용하고있는 기능중 하나다.
- 실무에서는 하나의 엔티티가 2-30가지의 필드를 갖게되는 경우도 많기때문에 테스트를 위해 모든 필드들을 지정해주는것이 아닌, 기본값을 지정하여 변경된 값만 파라미터에 이름을 붙여 넘겨줄수도 있다.

```kotlin
fun addMember(
        firstName = "bab",
        lastName = "e",
        address = "zip",
        phone = "010",
        note = "climbing"
) {
    // 사용자 추가 로직
}

// address 를 제외하고 모두 기본값으로 들어간다
addMember(address = "house")
```

---

- 코틀린을 사용하여 모듈을 만들때, 디폴트 파라미터 기능은 자바에서 지원하지 않기 때문에 기껏 만들어놓아도 코틀린의 장점을 십분 발휘하지 못한다.
- 그때 사용할수 있는 어노테이션이 `@JvmOverloads` 이다.
- 해당 어노테이션을 추가하면 자바에서는 모든 파라미터를 오버로드하여 생성한다.

---

#### 불필요한 클래스의 삭제

- 자바파일의 기본은 클래스다. 메서드는 기본이 되는 클래스 하위에 존재한다.
- 하지만 코틀린에서는 최상위에 클래스로 선언되어야 할 필요가 없다.
- 최상위 함수로 선언하면, 자바에서 사용하는 정적 메서드로 변환된다.
- 최상위 프로퍼티로 선언하면, 정적 필드로 사용되어 상수처럼도 사용할 수 있다.
- 자바에서의 `public static final` 필드로 노출하고 싶다면 `const` 변경자를 추가한다.

---

#### 확장함수
- 어떤 클래스의 멤버 메서드인 것처럼 호출할 수 있지만 그 클래스의 밖에 선언된 함수이다
  - 표준 자바 컬렉션 클래스에 우리가 원하는 확장함수들을 넣는것도 이와같은 방식이다. 
- 용어적으로는 생소하지만, 다음 예제엥서 `확장할 클래스`는 `수신 객체 타입`, 호출하는 대상 (객체) 는 `수신 객체` 라고 부른다.
```kotlin
// 다음에서는 String 이 수신객체 타입, this 가 수신 객체
fun String.lastChar(): Char = this.get(this.length -1)
```

- 확장함수를 다른 클래스에 존재한다면 당연히 import 를 해줘야 한다.
- 자바에서는 제공하지 않았었던 `as` 키워드로 import 한 클래스나 함수를 다른 이름으로 호출할 수 있다.
- 자바에서 코틀린의 확장함수를 호출할때는, 최상위 함수로 선언되었기에, 정적 메서드로 변환하여 사용하게 된다.
- 확장함수는 자바에서 정적 메서드처럼 사용하기 때문에 당연히 오버라이드 할 수 없다.
- **확장함수와 클래스의 멤버함수의 이름과 시그니처가 같다면 멤버 함수가 호출된다.** 확장함수를 추가해서 생성할때도, 모듈을 생성/수정 하며 멤버함수를 추가할때도 유의하자
```kotlin
   @Test
    fun foo(){
        val mutableList = mutableListOf(1, 2)
        mutableList.get(1)
    }
    // Extension is shadowed by a member: public abstract fun get(index: Int): T
    fun <T> MutableList<T>.get(index: Int){
        
    }
```

---

### 컬렉션 처리

#### 확장된 API들
- 일단 컬렉션 기준으로, 코틀린은 자바와 동일한 클래스를 사용하지만 더 확장된 API 를 제공한다.
- 사실 아주 많으며, 공부하기보다는 IDE에서 코드완성이 될때 생각나면 한번씩 찾아보는게 좋을것 같다
  - https://kotlinlang.org/api/latest/jvm/stdlib/

---

#### 가변 인자
- 가변인자가 있는 메서드를 정의할때 자바에서는 데이터타입 뒤에 `...` 을 붙임으로 가변인자임을 나타낸다. 코틀린에서는 파라미터 앞에 `vararg` 변경자를 붙인다.
- 자바에서는 배열을 직접 넘기면 되었다면, 코틀린에서는 배열을 명시적으로 풀어줘야한다. 배열을 원소단위로 넘어가도록 해주는 스프레드 연산자 (`*`) 를 쓰면 된다.

```kotlin
fun main(args: Array<String>) {
    val list = listOf("args: ", *args)
  print(list)
}
```

--- 

#### 3중 따옴표 문자열
- 3중 따옴표 문자열은 문자열 이스케이프를 피할 목적뿐만 아니라, 줄 바꿈이 있는 텍스트를 쉽게 포함시킬 수 있다.
- 개인적으로 테스트코드에서 `json` 형태의 `request`를 생성할때 요긴하게 사용한다.
- 3중 따옴표 문자열에서는 문자열 템플릿을 사용할 수도 있다.
```kotlin
// 3중 문자열을 통한 json 문자열 생성
val request = """
{
    "name": "babi",
    "age": 30
}
"""

// 문자열 템플릿도 사용 가능
val escape = """${person.name}"""
```

---

