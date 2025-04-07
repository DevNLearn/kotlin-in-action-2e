
# 코틀린 기초

## 함수와 변수

### 함수
**함수 선언**
- `fun` 키워드 사용
- 모든 코틀린 파일의 최상위 수준에 정의할 수 있으므로 클래스에 속하지 않음

**`main` 함수** 
- 최상위에 있는 `main` 함수를 진입점으로 지정
  - `main` 함수의 인자 여부는 관계 없이 아무 값도 반환하지 않는다
- 자바처럼 클래스 안에 `main` 함수를 정의하면 실행 버튼이 생기지 않음
  ```kotlin
  @SpringBootApplication
  class KopringApplication {
      fun main(args: Array<String>) {
        runApplication<KopringApplication>(*args)
      }
  }
  ```
- `main` 함수를 클래스 밖으로 빼 최상위에 둬야 진입점으로 인식됨
  - 이때 `main` 함수는 `static`이 아니므로 `@SpringBootApplication` 어노테이션을 붙일 수 없음
  - 따라서 `@SpringBootApplication` 어노테이션을 붙인 클래스와 `main` 함수를 따로 정의해야 함
  ```kotlin
  @SpringBootApplication
  class KopringApplication
  
  fun main(args: Array<String>) {
      runApplication<KopringApplication>(*args)
  }
  ```

**println 함수**
- 자바 System.out.println()에 대한 래퍼 함수

### 파라미터와 반환값이 있는 함수
- 파라미터 이름이 먼저 오고 그 뒤에 파라미터 타입 지정
- 타입과 이름을 콜론(`:`)으로 구분
- 여러 개의 파라미터는 쉼표(`,`)로 구분
- 반환 타입은 파라미터 목록 닫는 괄호 뒤에 콜론(`:`)으로 구분해 작성

```kotlin
fun max(a: Int, b: Int): Int {
    return if (a > b) a else b
}
```

**expression**
- 코틀린은 모든 것이 표현식(expression)으로 되어있음
  - 표현식(expression)? 
    - 값을 생성하며 다른 식의 하위 요소로 사용될 수 있는 것 (값으로 사용될 수 있는 것)
    - 코틀린은 루프(for, while, do/while)을 제외한 대부분의 제어 구조가 식
      ```kotlin
      // if문은 식이기 때문에 값으로 사용 가능
      val c: Int = if (a > b) a else b
      ```
- vs 문(statement)
  - 문은 값을 생성하지 않으며 다른 식의 하위 요소로 사용될 수 없음
  - 코틀린은 대입 연산이 항상 문으로 취급되므로 아무 값도 반환하지 않는다.
  ```java
  // 자바의 경우 대입 연산이 값을 반환하기 때문에 에러 발생은 안 하지만 실수할 여지가 생긴다
  if ((a=5) > 0) { System.out.println(a); }
  ```
  ```kotlin
  // 코틀린의 경우 대입 연산이 문으로 취급되기 때문에 에러 발생
  if ((a=5) > 0) { ... }
  ```
  - `Assignments are not expressions, and only expressions are allowed in this context` 경고문
  - 컴파일 에러로 조기 대처 가능

### 간결한 식 본문 함수
- 식 본문 함수는 중괄호 없이 식을 바로 반환하는 함수로 return 키워드 없이 `=` 뒤에 식을 작성
- 반환타입을 적어주지 않아도 코틀린 컴파일러가 타입을 추론
  ```kotlin
  fun max(a: Int, b: Int) = if (a > b) a else b
  ```

<br/>

## 변수
- 읽기 전용 변수 `val`
  - 자바의 `final` 키워드와 유사
  - 선언 시점에 딱 한번 초기화해야 하며 이후 변경 불가
- 재대입 가능 변수 `var`
  - 선언 시점에 초기화하지 않아도 되며 초기화되더라도 이후 재대입 가능
  - 값은 변경 가능하지만 타입은 변경 불가
- 기본적으로 `val`을 사용하고, 꼭 수정이 필요한 경우에만 `var` 사용
  - `val`의 불변성과 순수 함수(부수 효과가 없는 함수) 조합으로 함수형 프래그래밍 이점 얻을 수 있음

**변수 타입 추론**
- 식 본문 함수와 마찬가지로 타입을 지정하지 않으면 컴파일러가 초기화 식을 분석해 초기화 식의 타입을 변수 타입으로 추론
  - 초기화 식으로 판단하기 때문에 타입 변경 불가 (다른 타입을 변수에 넣고 싶다면 강제 형변환 필요)
- 변수 선언 시 즉시 초기화하지 않고 나중에 값을 대입하고 싶을 때는 타입을 명시적으로 지정해야 함
  - 코틀린 컴파일러가 변수 타입을 추론할 근거가 없기 때문

**`val` 읽기 전용 변수가 가리키는 객체의 내부 값은 변경될 수 있다**
- 즉, `val`로 선언된 읽기 전용 변수는 객체의 주소값을 변경할 수 없지만 객체의 내부 값은 변경할 수 있다
  ```kotlin
  fun main() {
    val languages = mutableListOf("Java")
    languages.remove("Java")
    languages.add("Kotlin")
  }
  ```

### 문자열 템플릿
- `$` 기호를 사용해 문자열 안에 변수 사용 가능 
  - `${}`로 감싸면 표현식(expression)도 사용 가능
  - 컴파일러는 해당 표현식을 정적으로 검사하기 때문에 존재하는 변수만 사용 가능
  ```kotlin
  fun main() {
    val name = readln()
    println("Hello, ${if (name.isBlank()) "someone" else name}!")
    // Blank input: Hello, someone!
    // "Millo" input: Hello, Millo!
  }
  ```

<br/>

## 클래스와 프로퍼티
  ```kotlin
  class Person(
    val name: String,   
    var isStudent: Boolean
  )
  ```
- 코틀린은 자바와 마찬가지로 객체지향 언어이기 때문에 클래스를 사용해 객체 캡슐화
  - 데이터를 캡슐화하고 캡슐화한 데이터를 다루는 코드를 한 주체(클래스)로 묶어 관리
- 기본 가시성으로 `public` 지정되어 있어 지정자 생략 가능
- 프로퍼티를 언저 기본 기능으로 제공
  - 프로퍼티? 
    - 클래스의 속성으로, 자바의 필드와 접근자를 합친 것
    - 프로퍼티는 게터와 세터 메서드를 자동으로 생성
      - 자바의 경우 접근자 메서드(게터, 세터)를 직접 작성해야 함
  - 변수와 마찬가지로 읽기 전용 프로퍼티 `val`과 재대입 가능한 프로퍼티 `var`로 구분
    - 읽기 전용 프로퍼티는 게터 메서드만 생성
    - 재대입 가능한 프로퍼티는 게터와 세터 메서드 모두 생성
    - 코틀린에서는 직접 프로퍼티명으로 게터와 세터 메서드 접근 가능
    - 자바에 노출되는 접근자 메서드 네이밍
      - getter
        - is로 시작하는 필드: 이름 그대로 `${propertyName}`
        - 그 외: `get${propertyName}`
      - setter
        - is로 시작하는 필드: `is${propertyName}`
        - 그 외: `set${propertyName}`

**자바 코드로 변환했을 때 각 필드가 private로 변하는 이유**
위의 Person 클래스를 자바로 변환해보면 아래와 같이 나온다.
  ```java
  public final class Person {
     @NotNull
     private final String name;
     private boolean isStudent;
  
     public Person(@NotNull String name, boolean isStudent) {
        Intrinsics.checkNotNullParameter(name, "name");
        super();
        this.name = name;
        this.isStudent = isStudent;
     }
  
     @NotNull
     public final String getName() {
        return this.name;
     }
  
     public final boolean isStudent() {
        return this.isStudent;
     }
  
     public final void setStudent(boolean var1) {
        this.isStudent = var1;
     }
  }
  ```
- 코틀린의 캡슐화를 자바 변환 후에도 유지하기 위해서이다.
- `private`로 비공개 필드를 선언하고 접근자 메서드를 통해서만 접근 가능하도록 해 데이터 구조를 노출하지 않고 접근하도록 한다.


**커스텀 접근자**
- 프로퍼티가 같은 객체안의 다른 프로퍼티에 의존할 때 필드에 저장하지 않고 커스텀 접근자 사용
- 커스텀 접근자는 `get`과 `set` 키워드를 사용해 정의
- 커스텀 접근자는 해당 프로퍼티에 접근할 때마다 실행
  ```kotlin
  class Rectangle(val height: Int, val width: Int) {
      val isSquare: Boolean
          get() = height == width
        
      val area: Int
          get() = height * width
  }
  ```

**커스텀 접근자 vs 멤버 함수**
- 구현이나 성능 차이는 없으나 의미를 어디에 두느냐 차이
- 커스텀 접근자는 클래스의 특성인 프로퍼티에 가까울 때 사용
- 멤버 함수는 클래스의 동작에 가까울 때 사용

<br/>

## Enum
- 열거형 클래스는 `enum class` 키워드를 사용해 정의
  - 코틀린에서 `enum`은 소프트 키워드로 `class` 키워드와 함께 사용해야 함
  - 반면 `class` 키워드는 하드 키워드로 식별자로 사용할 수 없어서 clazz나 aClass처럼 써야함
- 일반적인 클래스와 마찬가지로 생성자와 프로퍼티 사용 가능
```kotlin
enum class Color(
    val r: Int,
    val g: Int,
    val b: Int,
) {
    RED(255, 0, 0),
    ORANGE(255, 165, 0),
    YELLOW(255, 255, 0),
    GREEN(0, 255, 0),
    BLUE(0, 0, 255),
    INDIGO(75, 0, 130),
    VIOLET(238, 130, 238),
    ; // 열거형 상수 뒤에 반드시 세미콜론(;)을 붙여야 함
  
  fun rgb() = (r * 256 + g) * 256 + b
  fun printColor() = println("$this is ${rgb()}")
}
```

<br/>

## When
- `when`은 자바의 `switch`문과 유사하지만 더 강력하고 유연함
- `if`문과 마찬가지로 식이기 때문에 값을 반환할 수 있음
- 자바와 달리 각 분기 끝에 `break`를 붙이지 않아도 됨
- 한 분기 안에서 값 사이를 콤마(`,`)로 구분해 여러 값을 지정할 수 있음
- `when`식을 사용할 때마다 컴파일러는 각 분기를 검사해 모든 가능한 경로를 다루는지 검사
  - 모든 분기를 다루지 않을 경우엔 else 키워드로 디폴트 케이스 제공해야 컴파일 에러 막을 수 있음
```kotlin
fun getWarmthFromColor(): String {
    val color = Color.ORANGE
    return when (color) {
        Color.RED, Color.ORANGE, Color.YELLOW -> "warm (red = ${color.r}"
        Color.GREEN -> "neutral (green = ${color.g})"
        Color.BLUE, Color.INDIGO, Color.VIOLET -> "blue (blue = ${color.b})"
    }
}

fun main() {
    println(getWarmthFromColor())
}
```

**when 식의 대상을 변수에 캡처**
- `when` 식의 대상을 변수에 캡처해 `when` 식의 영역으로 제한되어 각 분기 안에서만 사용 가능
```kotlin
fun getWarmthFromColor(): String {
  return when (val color = Color.ORANGE) {
    Color.RED, Color.ORANGE, Color.YELLOW -> "warm (red = ${color.r}"
    Color.GREEN -> "neutral (green = ${color.g})"
    Color.BLUE, Color.INDIGO, Color.VIOLET -> "blue (blue = ${color.b})"
  }
}
```

**when의 분기 조건에 다른 여러 객체 사용**
- 인지로 아무 객체나 사용할 수 있고, 그 인자를 각 분기 조건의 객체와 같은지 비교
```kotlin
fun mix(c1: Color, c2: Color) = 
    when (setOf(c1, c2)) {
        setOf(Color.RED, Color.YELLOW) -> Color.ORANGE
        setOf(Color.YELLOW, Color.BLUE) -> Color.GREEN
        setOf(Color.BLUE, Color.RED) -> Color.INDIGO
        else -> throw Exception("Dirty color")  // 디폴트 케이스로 철저함 보장
    }
```
- 이 때 각 분기에서 인자 값과 일치하는 조건을 찾을 때에는 동등성 (`==`) 비교를 사용한다

**인자없는 when식**
- 위의 예제처럼 하면 매번 객체가 생성되기 때문에 가비지 컬렉터가 수거할 객체가 늘어나게 된다
- 인자 없이 `when`을 사용해 불필요한 객체 생성 없이 작성할 수 있다.
  - 아무 인자도 없으려면 각 분기의 조건이 Boolaen 결과를 계산해야 한다
```kotlin
fun mix(c1: Color, c2: Color) = 
    when {
        (c1 == Color.RED && c2 == Color.YELLOW) || (c1 == Color.YELLOW && c2 == Color.RED) -> Color.ORANGE
        (c1 == Color.YELLOW && c2 == Color.BLUE) || (c1 == Color.BLUE && c2 == Color.YELLOW) -> Color.GREEN
        (c1 == Color.BLUE && c2 == Color.RED) || (c1 == Color.RED && c2 == Color.BLUE) -> Color.INDIGO
        else -> throw Exception("Dirty color") 
    }
```

<br/>

## 스마트 캐스트
- 표현식(experssion)을 인코딩할 때 이진 트리와 같은 구조로 구성한다
- 각 노드는 표현식의 결과를 나타내며, 각 노드의 자식 노드는 그 노드의 하위 표현식을 나타낸다
- Num은 항상 Leaf 노드, Sum은 항상 자식이 둘 있는 중간 노드로 구성된다

**Expr 마커 인터페이스 2가지 구현**
1. `Num` 이라면 그 값 반환
2. `Sum`이라면 왼쪽과 오른쪽 자식 노드에 대해 재귀 호출

```kotlin
interface Expr  // 마커 인터페이스
class Num(val value: Int) : Expr
class Sum(val left: Expr, val right: Expr) : Expr

fun eval(e: Expr) = 
    when (e) {
        is Num -> e.value  // 스마트 캐스트
        is Sum -> eval(e.left) + eval(e.right)  // 재귀 호출
        else -> throw IllegalArgumentException("Unknown expression")
    }

fun main() {
    println(eval(Sum(Sum(Num(1), Num(2)), Num(4))))  // 7
}
```
- `is` 연산자로 타입을 검사하면 `e`의 타입이 `Num`으로 스마트 캐스트됨
- 스마트 캐스트란?
  - 어떤 변수의 타입을 확인한 다음에 그 타입에 속한 멤버에 접근하기 위해 명시적으로 변수 타입을 반환하지 않아도 컴파일러가 타입 추론
    - 즉, `as`로 명시적으로 캐스팅하지 않아도 됨
  - 자바에서는 `instanceof` 연산자로 타입을 검사한 후에 명시적으로 캐스팅해야 함
- 다만, 스마트 캐스트는 `is`로 변수에 든 값의 타입을 검사한 후 그 값이 바뀔 수 없는 경우에만 작동한다
  - 스마트 캐스트 조건
    - `val`로 선언되었으면서 커스텀 접근자를 사용하지 않은 프로퍼티
    - 해당 조건이 아니라면 해당 프로퍼티에 대한 접근이 항상 같은 값을 가리키는지 보장할 수 없기 때문

<br/>

## 이터레이션
**while, do-while**
- 자바와 마찬가지로 while과 do-while 사용 가능
- 내포된 루프의 경우 레이블 지정 가능
  - 레이블은 `@` 기호로 시작하는 식별자
  - 레벨을 지정한 루프는 `break`와 `continue` 키워드 사용 시 레이블 참조 가능

**범위 이터레이션**
- `..` 연산자를 사용해 범위를 지정 후 `for`문을 사용해 범위 이터레이션 가능
  - 범위는 폐구간으로 양 끝을 포함하는 구간
- `downTo` 연산자를 사용해 역순으로 범위 이터레이션 가능
- `step` 연산자를 사용해 범위 이터레이션 시 증가폭 지정 가능
- `in` 연산자를 사용해 범위에 포함되는지 확인 가능

```kotlin
fun main() {
    for (i in 1..100) { println(i) }                // 1부터 100까지
    for (i in 100 downTo 1 step 2) { println(i) }   // 100부터 1까지 2씩 감소
}
```

**맵 이터레이션**
- 자주 사용하게 될 `for (x in y)` 루프는 입력 컬렉션의 각 요소를 순회한다.
- 루프는 `y`가 `Iterable` 인터페이스를 구현한 컬렉션일 때만 사용 가능
```kotlin
fun main() {
    val binaryReps = mutableMapOf<Char, String>()   // 가변맵은 원소 이터레이션 순서 보존
    for (char in 'A'..'F') {
        binaryReps[char] = char.code.toString(radix = 2)
    }
  
    for ((letter, binary) in binaryReps) {  // 구조 분해
        println("$letter = $binary")
    }
}
```
- 구조 분해 구문을 맵이 아닌 컬렉션의 현재 인덱스를 유지하면서 컬렉션 이터레이션도 가능
  - 인덱스를 저장하기 위한 변수를 별도로 선언하고 루프에서 매번 그 변수를 증가시킬 필요가 없어진다
  ```kotlin
  fun main() {
      val list = listOf("A", "B", "C")
      for ((index, element) in list.withIndex()) {  // 인덱스와 함께 컬렉션 이터레이션 
          println("$index: $element")
      }
  }
  ```

**in 연산자 활용**
- 어떤 값이 변위에 속하는지(`in`) 혹은 속하지 않는지(`!in`) 검사 가능
- 비교가 가능한 클래스라면(`java.lang.Comparable` 인터페이스 구현) 그 클래스의 인스턴스 객체를 사용해 범위를 만들어 비교할 수 있다.
  - Int, String, Char 등은 Comparable 인터페이스를 구현하고 있어서 범위로 사용 가능
- `in`은 `contains`로 `!in`은 `!contains`로 비교하는 것과 같음
```kotlin
fun isLetter(c: Char) = c in 'a'..'z' || c in 'A'..'Z'
fun isNotDigit(c: Char) = c !in '0'..'9'

// 문자열의 경우 알파벳 순서로 비교
fun main() {
    println(isLetter('q'))
    println(isNotDigit('x'))
}
```

<br/>

## 코틀린 예외 핸들링
- 자바와 마찬가지로 `try-catch` 문을 사용해 예외 처리 가능
  - `throw` 키워드를 사용해 예외를 던질 수 있음
    - 자바와 마찬가지로 호출하는 쪽에서 해당 예외를 잡아 처리
    - 처리하지 않으면 함수 호출 스택을 따라 올라가면서 예외 전파
  - `throw`는 식이므로 다른 식에 포함될 수 있음
- 코틀린은 `throws` 키워드가 없기 때문에 예외를 던질 수 있는 메서드에 대해 예외를 명시할 필요 없음
- 자바처럼 `finally` 블록으로 정상 종료와 예외 발생 여부에 관계 없이 항상 실행되는 블록을 정의할 수 있음
```kotlin
import java.io.BufferedReader
import java.io.StringReader

fun readNumber(reader: BufferedReader): Int? {
    try {
        val line = reader.readLine()
        return Integer.parseInt(line)
    } catch (e: NumberFormatException) {
        return null
    } finally {
        reader.close()
    }
}

fun main() {
    val reader = BufferedReader(StringReader("239"))
    println(readNumber(reader)) // 239
}
```

**코틀린은 checked exception과 unchecked exception을 구분하지 않는다**
- checked exception은 반드시 처리해야 하는 예외로, 자바에서는 checked exception을 처리하지 않으면 컴파일 에러가 발생한다
- 따라서 자바는 함수가 던질 수 있는 예외를 모두 선언해야 하고, 그 함수를 사용하는 쪽에서 그 예외를 전부 캐치해 처리하거나 다시 그 예외를 던질 수 있다고 명시적으로 선언해줘야 한다
- 반면 코틀린은 Checked Exception을 구분하지 않기 때문에 예외를 처리하지 않아도 컴파일 에러가 발생하지 않는다
  - 컴파일러가 예외 처리를 강제하지 않기 때문에 예외를 캐치해서 처리해도 되고 안 해도 된다
  ```kotlin
  fun readNumber(reader: BufferedReader): Int {
      val line = reader.readLine()
      reader.close()
      return Integer.parseInt(line)  // NumberFormatException 발생 가능
  }
  ```

<br/>

## try expression
- `try`는 문으로도 사용할 수 있지만 식으로도 사용 가능
- 정상 동작하면 try 블록의 결과를 반환하고, 예외가 발생하면 catch 블록의 결과를 반환
```kotlin
fun readNumber(reader: BufferedReader) {
    val number = try {
        Integer.parseInt(reader.readLine())
    } catch (e: NumberFormatException) {
        null
    }
    println(number)
}

fun main() {
    val reader = BufferedReader(StringReader("not a number"))
    readNumber(reader)  // 아무것도 출력 안됨
}
```

<br/>