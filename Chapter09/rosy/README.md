# 9장 연산자 오버로딩과 다른 관례
### 관례란?
코틀린에서는 특정 언어 기능이 미리 정해진 함수 이름과 연결되는 방식을 관례라고 부른다.
예를 들어, a + b와 같은 연산은 코틀린 내부적으로 a.plus(b)라는 함수 호출로 처리된다.
이를 통해 코틀린에서는 연산자를 자유롭게 사용할 수 있으며, 사용자 정의 객체에도 연산자를 적용할 수 있다.  

### 관례를 도입한 이유
이러한 관례를 채택한 이유는 바로 자바 클래스의 확장성에 있다.
기존 자바 클래스의 인터페이스는 이미 고정되어 있기 때문에 코틀린에서 직접 수정하기 어렵다. 
하지만 코틀린의 **확장 함수** 를 사용하면 기존 자바 클래스를 수정하지 않고도 새로운 기능을 추가할 수 았다. 
덕분에 코틀린에서는 연산자 오버로딩이 자유롭고 직관적인 문법으로 표현된다.

---

## 9.1 산술 연산자를 오버로드해서 임의의 클래스에 대한 연산을 더 편리하게 만들기
### 연산자 오버로딩이란?
- 코틀린에서는 특정 연산자(`+`, `-`, `*`, `/`, `%`)를 직접 정의하여 사용할 수 있다.
- 이를 **연산자 오버로딩(Operator Overloading)** 이라 한다.
- 예를 들어, `a + b`는 내부적으로 `a.plus(b)`로 해석된다.
- 코틀린은 **관례(Convention)** 에 의해 특정 함수 이름이 미리 정의되어 있다.
  - `+` → `plus()`
  - `-` → `minus()`
  - `*` → `times()`
  - `/` → `div()`
  - `%` → `mod()`

####  연산자 오버로딩의 목적
- 연산자 오버로딩의 주된 목적은 **사용자 정의 클래스에서도 산술 연산자처럼 직관적인 사용이 가능하도록 하는 것**이다.
- 예를 들어, `BigInteger`나 `LocalDate` 같은 클래스에서 `add()`나 `plusDays()` 같은 함수 호출을 하지 않고도, `+`, `-` 연산자를 사용할 수 있다면 코드 가독성이 훨씬 좋아진다.
- 특히, 복잡한 수식이나 수학적 연산이 많은 코드에서는 메서드 호출보다 직관적인 연산자를 사용하면 코드가 깔끔해진다.


### 이항 산술 연산 오버로딩
- **이항 산술 연산자**란 두 개의 피연산자를 가지는 산술 연산을 말한다.
  - 예: `a + b`, `a - b`, `a * b` 등
- 코틀린에서는 이러한 이항 연산자를 오버로딩하여, 사용자 정의 객체도 산술 연산이 가능하도록 만들 수 있다.
```kotlin
data class Point(val x: Int, val y: Int) {
    operator fun plus(other: Point): Point {
        return Point(x + other.x, y + other.y)
    }
}

fun main() {
    val p1 = Point(10, 20)
    val p2 = Point(30, 40)
    println(p1 + p2)  // Point(x=40, y=60)
}
```
    
- operator 키워드를 사용해 오버로딩을 명시한다.
- `+` 연산자를 오버로딩하면 내부적으로 plus 함수가 호출된다. 
- 두 개의 Point 객체를 더하는 plus 연산이 자연스럽게 동작한다. 
- operator 키워드는 이 함수가 코틀린의 연산자 오버로딩 규칙에 맞춰 사용됨을 의미한다.



### 확장 함수로도 연산자 오버로딩이 가능
- 코틀린에서는 확장 함수(Extension Function) 를 사용하여 클래스 외부에서도 연산자 오버로딩이 가능하다.
- 예를 들어, Point 클래스의 plus 연산자를 확장 함수로 만들면 아래와 같이 구현할 수 있다.

```kotlin
operator fun Point.plus(other: Point): Point {
    return Point(x + other.x, y + other.y)
}
```
- 클래스 내부에 정의하지 않고도 외부에서 오버로딩이 가능하다. 
- 특히, 자바의 기존 클래스도 확장 함수를 통해 연산자 오버로딩이 가능하다.

### 오버로딩이 가능한 이항 산술 연산자 목록
- 코틀린에서는 특정 연산자만 오버로딩할 수 있으며, 그 연산자는 정해져 있다.

| 식             | 함수 이름         |
  |---------------| --- | 
| `a*b`  | `times`       |
| `a/b`  | `div`        |
| `a%b`  | `mod`     |
| `a+b`  | `plus`   |
| `a-b`  | `minus`   |

### 연산자 함수와 자바
- 코틀린에서 정의된 연산자 오버로딩 함수는 자바에서도 호출할 수 있다.
- 모든 오버로딩된 연산자는 함수로 정의되며, 자바에서는 긴 이름(FQM - Fully Qualified Method Name) 을 사용해 일반 함수처럼 호출할 수 있다.
- 반대로, 자바 메서드가 코틀린의 관례에 맞는 이름을 가지고 있다면, 코틀린에서 연산자처럼 사용할 수 있다.
#### 예시
- 자바 메서드가 plus()로 정의되어 있다면, 코틀린에서 + 연산자로 사용할 수 있다.
- 자바 메서드가 div()로 정의되어 있다면, 코틀린에서 / 연산자로 사용할 수 있다.
#### 주의
- 자바에서는 operator 키워드가 없기 때문에, 메서드 이름과 파라미터 개수만 일치하면 된다.
- 만약 이름만 다른 경우, 확장 함수를 사용하여 코틀린 스타일의 연산자 오버로딩을 지원할 수 있다.

### 교환 법칙은 자동으로 지원되지 않음
- 코틀린에서는 연산자의 교환 법칙을 자동으로 지원하지 않는다. 
  - 예를 들어, a * 2.5는 가능하지만 2.5 * a는 불가능하다.
```kotlin
operator fun Point. times (scale: Double): Point {
  return Point ((x * scale). toInt(), (y * scale). toInt())
}

fun main() {
  val p = Point (10, 20)
  printIn(p * 1.5)
  // Point (x=15, y=30) 
}
```
- 이를 지원하려면 별도의 오버로딩이 필요하다.
```kotlin
operator fun Double.times(p: Point): Point {
    return Point((p.x * this).toInt(), (p.y * this).toInt())
}

fun main() {
    val p = Point(10, 20)
    println(1.5 * p)  // Point(x=15, y=30)
}
```
  - Double 타입에서 Point를 곱하는 형태로 오버로딩했다. 


### 다른 타입 조합도 가능
- 연산자의 왼쪽과 오른쪽이 서로 다른 타입이라도 오버로딩이 가능하다. 
- 예를 들어, Char와 Int 조합으로 * 연산을 정의할 수 있다.
```kotlin
operator fun Char.times(count: Int): String {
    return toString().repeat(count)
}

fun main() {
    println('a' * 3)  // aaa
}
```

### 비트 연산자
- 코틀린에서는 비트 연산자(&, |, ^, <<, >>)를 직접 오버로딩하지 않는다.
- 대신, 중위 표기법(Infix Notation) 을 통해 비트 연산이 가능하다.

  | 연산자    | 함수 이름         |
  | ------ | ------------- |
  | `shl`  | 왼쪽 시프트        |
  | `shr`  | 오른쪽 시프트       |
  | `ushr` | 부호 없는 오른쪽 시프트 |
  | `and`  | 비트 AND        |
  | `or`   | 비트 OR         |
  | `xor`  | 비트 XOR        |
  | `inv`  | 비트 NOT        |

```kotlin
fun main() {
    println(0x0F and 0xF0) // 0
    println(0x0F or 0xF0)  // 255
    println(0x1 shl 4)     // 16
}
```

### 복합 대입 연산자 오버로딩
- +=, -=, *=, /= 같은 복합 대입 연산자도 오버로딩이 가능하다.
```kotlin
fun main() {
    var point = Point(1, 2)
    point += Point(3, 4)
    println(point)  // Point(x=4, y=6)
}

```
- 코틀린에서는 plusAssign, minusAssign, timesAssign, divAssign 등의 이름으로 오버로딩한다.
```kotlin
data class Point(var x: Int, var y: Int) {
    operator fun plusAssign(other: Point) {
        x += other.x
        y += other.y
    }
}

fun main() {
    var p1 = Point(10, 20)
    val p2 = Point(30, 40)
    p1 += p2
    println(p1) // Point(x=40, y=60)
}
```
- 복합 대입 연산자는 변경 가능한 객체에 유용하다.
- 새로운 객체를 생성하는 것이 아닌, 참조된 객체를 직접 수정한다.

### 복합 대입 연산자의 컴파일 처리
- 복합 대입 연산자(+=, -=)는 코틀린에서 다음 두 가지 방식으로 컴파일될 수 있다.
  - plus 오버로딩을 사용: 
    - 예를 들어, a += b는 a = a.plus(b)로 처리된다. 
  - plusAssign 오버로딩을 사용:
    - 만약 plusAssign이 정의되어 있다면, a.plusAssign(b)로 처리된다.
```kotlin
fun main() {
    val list = mutableListOf(1, 2)
    list += 3
    println(list) // [1, 2, 3]
}
```

### 단항 연산자 오버로딩
- 단항 연산자는 하나의 피연산자로 연산이 이루어진다. 
  - 예를 들어, -a, +a, !a, ++a, a-- 같은 연산자가 이에 해당한다. 
- 코틀린에서는 다음과 같은 함수 이름을 사용한다.

  | 연산자          | 함수 이름        |
  | ------------ | ------------ |
  | `+a`         | `unaryPlus`  |
  | `-a`         | `unaryMinus` |
  | `!a`         | `not`        |
  | `++a`, `a++` | `inc`        |
  | `--a`, `a--` | `dec`        |

```kotlin
operator fun Point.unaryMinus(): Point {
    return Point(-x, -y)
}

fun main() {
    val p = Point(10, 20)
    println(-p)  // Point(x=-10, y=-20)
}
```
### 증가 연산자 오버로딩
- 코틀린에서는 ++, --와 같은 증가/감소 연산자도 오버로딩이 가능하다.
- 특히, BigDecimal 같은 자바 클래스에서도 사용할 수 있다.
```kotlin
import java.math.BigDecimal

operator fun BigDecimal.inc() = this + BigDecimal.ONE

fun main() {
    var bd = BigDecimal.ZERO
    
    println(bd++)  // 0
    println(bd)    // 1
    println(++bd)  // 2
}
```
- operator fun BigDecimal.inc()로 ++ 연산자를 오버로딩했다. 
- 후위 연산(bd++)의 경우, 현재 값을 반환한 후 증가된다. 
- 전위 연산(++bd)의 경우, 먼저 증가한 후 값을 반환한다. 
- 후위와 전위의 동작 차이는 일반 변수의 ++ 연산자와 동일하게 작동한다.

-----

## 9.2 비교 연산자를 오버로딩해서 객체들 사이의 관계를 쉽게 검사

### 비교 연산자 오버로딩이란?
- 코틀린에서는 산술 연산자와 마찬가지로 **비교 연산자**도 오버로딩할 수 있다.
- `==`, `!=`, `<`, `<=`, `>`, `>=` 같은 연산자를 오버로딩하면 객체 간의 관계를 더욱 직관적으로 표현할 수 있다.
- 자바에서는 `equals()`나 `compareTo()`를 직접 호출해야 했지만, 코틀린에서는 **연산자 오버로딩**을 통해 간결하게 사용할 수 있다.

### 동등성 연산자: `equals`
- 코틀린에서 `==` 연산자는 **equals** 호출로 컴파일된다.
- `!=` 연산자는 `equals`의 결과를 반대로 뒤집은 값이 반환된다.
- `==`과 `!=`은 **null 안전성**이 자동으로 처리된다.
- `a == b` 는 코틀린 내부적으로 `a?.equals(b) ?: (b == null)` 같은 코드로 처리된다.
- 코틀린에서는 data class를 사용하면 equals()가 자동으로 생성된다.
- 수동으로 구현하고 싶다면 다음과 같이 오버라이딩이 필요하다.
  ```kotlin
  class Point(val x: Int, val y: Int) {
    override fun equals(other: Any?): Boolean {
        if (other === this) return true // 동일한 객체면 바로 true
        if (other !is Point) return false // 타입이 다르면 false
        return other.x == x && other.y == y
    }
  }

  fun main() {
    println(Point(10, 20) == Point(10, 20))  // true
    println(Point(10, 20) != Point(5, 5))    // true
    println(null == Point(1, 2))             // false
  }
   ```

### 순서 연산자: `compareTo`
- 코틀린에서 <, <=, >, >= 연산자는 compareTo 메서드를 호출한다. 
- compareTo는 두 객체를 비교하여 
  - 0보다 작으면: 왼쪽이 작은 경우 
  - 0이면: 두 객체가 같은 경우 
  - 0보다 크면: 왼쪽이 큰 경우
-  Java의 Comparable 
   - 코틀린에서 객체 간의 비교를 위해서는 Comparable 인터페이스를 구현해야 한다. 
   - Java에서 compareTo()를 호출하듯이 코틀린에서도 동일하게 사용된다.
- compareTo의 컴파일 변환
  - `a >= b` 는 다음과 같이 컴파일된다.
    - `a.compareTo(b) >= 0`
  - 이와 같이 모든 순서 비교 연산자는 내부적으로 compareTo() 메서드 호출로 변경된다.
- compareTo 구현하기
  ```kotlin
  class Person(
    val firstName: String, val lastName: String
  ) : Comparable<Person> {
    override fun compareTo(other: Person): Int {
        // 성을 먼저 비교하고, 동일하면 이름을 비교
        return compareValuesBy(this, other, Person::lastName, Person::firstName)
    }
  }
  
  fun main() {
    val p1 = Person("Alice", "Smith")
    val p2 = Person("Bob", "Johnson")
    println(p1 < p2)    // false
  }

  ```
- 코틀린에서는 String 타입도 compareTo를 지원하므로 직접 <, >로 비교할 수 있다.
  ```kotlin
  fun main() {
    println("abc" < "bac") // true
  }
   ```
  - 문자열도 Comparable을 구현하고 있기 때문에 <, >로 비교가 가능하다. 
  - 알파벳 순서(사전 순)로 비교가 진행된다.

-----

## 9.3 컬렉션과 범위에 대해 쓸 수 있는 관례
### 인덱스로 원소 접근: `get`과 `set`
- 코틀린에서는 배열, 리스트, 맵 등에 `[index]` 형태로 접근할 수 있다.
  - 예를 들어, `map[key]` 또는 `list[index]`처럼 사용할 수 있다.
- 내부적으로는 `get`과 `set` 함수가 호출된다.

```kotlin
val map = mutableMapOf("one" to 1, "two" to 2)

// 값을 가져올 때
val value = map["one"]
println(value) // 1

// 값을 변경할 때
map["two"] = 22
println(map) // {one=1, two=22}
```
#### get 연산자 관례
- 직접 클래스에 get 연산자를 오버로딩할 수 있다.
- 예를 들어, Point 클래스의 좌표를 인덱스로 가져오도록 정의할 수 있다.
  ```kotlin
  data class Point(val x: Int, val y: Int)
  
  operator fun Point.get(index: Int): Int {
      return when (index) {
          0 -> x
          1 -> y
          else -> throw IndexOutOfBoundsException("Invalid coordinate $index")
      }
  }
  
  fun main() {
      val p = Point(10, 20)
      println(p[0]) // 10
      println(p[1]) // 20
  }
  ```   
  - operator 키워드를 사용해 get 연산자를 구현하면, p[0]은 p.get(0)으로 컴파일된다. 
  - 인덱스가 유효하지 않으면 예외를 발생시킨다.

#### set 연산자 관례
- 값의 변경이 가능한 경우 set 연산자도 오버로딩할 수 있다.
  ```kotlin
  data class MutablePoint(var x: Int, var y: Int)

  operator fun MutablePoint.set(index: Int, value: Int) {
    when (index) {
        0 -> x = value
        1 -> y = value
        else -> throw IndexOutOfBoundsException("Invalid coordinate $index")
    }
  }
  
  fun main() {
    val p = MutablePoint(10, 20)
    p[1] = 42
    println(p) // MutablePoint(x=10, y=42)
  }
  ```
  - `operator` 키워드를 사용해 set 연산자를 구현하면, p[1] = 42는 p.set(1, 42)로 컴파일된다. 
  - 인덱스에 맞게 값을 수정하고, 유효하지 않으면 예외를 발생시킨다.

### in 키워드를 사용한 컬렉션 포함 여부 검사: `contains`
- 코틀린에서는 in 연산자를 통해 객체가 컬렉션에 포함되어 있는지 검사할 수 있다.
- 내부적으로 contains 메서드를 호출한다.
#### in 연산자 관례
- 사용자 정의 클래스에서도 in 연산자를 오버로딩할 수 있다. 
- 예를 들어, Rectangle 클래스에 특정 좌표가 포함되는지 검사할 수 있다.
  ```kotlin
  data class Rectangle(val upperLeft: Point, val lowerRight: Point)
  
  operator fun Rectangle.contains(p: Point): Boolean {
    return p.x in upperLeft.x..<lowerRight.x &&
        p.y in upperLeft.y..<lowerRight.y
  }
  
  fun main() {
    val rect = Rectangle(Point(10, 20), Point(50, 50))
    println(Point(20, 30) in rect) // true
    println(Point(5, 5) in rect)   // false
    println(Point(50, 50) in rect) // false
  }
    ```
  - operator 키워드를 사용해 contains를 오버로딩했다. 
  - in 연산자는 내부적으로 rect.contains(Point(20, 30))로 컴파일된다. 
  - `..<` 키워드를 사용해 열린 범위(Open Range) 를 만든다. 
    - 10..<50은 10부터 49까지만 포함하고, 50은 포함되지 않는다. 
    - 그래서 Point(50, 50)은 범위에 속하지 않아 false가 반환된다.

####  열린 범위 (Open Range)와 닫힌 범위 (Closed Range)
| 표현식         | 설명    | 포함 여부                |
|-------------| ----- | -------------------- |
| `10..50`    | 닫힌 범위 | 10부터 50까지 포함         |
| `10..<50`, `10 until 50` | 열린 범위 | 10부터 49까지 포함 (50 제외) |


### 범위 생성: `rangeTo`와 `rangeUntil`
- 코틀린에서는 .. 연산자를 통해 범위(Range) 를 생성할 수 있다.
- 내부적으로 `rangeTo` 함수가 호출된다.

#### 날짜의 범위 생성
- 자바 8의 LocalDate와 함께 범위를 생성할 수 있다.
  ```kotlin
  import java.time.LocalDate

  fun main() {
    val start = LocalDate.now()
    val end = start.plusDays(10)
    val range = start..end
    println(start.plusWeeks(1) in range) // true
  }
  ```
  - rangeTo가 호출되어 범위가 생성된다. 
  - 날짜도 ..를 통해 쉽게 범위로 만들 수 있다. 
  - 특정 날짜가 범위 안에 포함되는지 검사할 수 있다.

### for 루프를 위한 이터레이터: `iterator`
- 코틀린의 for 문은 in 키워드를 사용하며, 내부적으로 iterator()가 호출된다. 
- iterator()가 호출되면 hasNext()와 next()를 순회한다.
#### 날짜의 이터레이터
- LocalDate의 범위를 for 루프에서 사용할 수 있도록 확장한다.
  ```kotlin
  import java.time.LocalDate

  operator fun ClosedRange<LocalDate>.iterator(): Iterator<LocalDate> = 
    object : Iterator<LocalDate> {
    var current = start
    override fun hasNext() = 
         current <= endInclusive
  
    override fun next(): LocalDate {
        val thisDate = current
        current = current.plusDays(1)
        return thisDate
      }
  }
  
  fun main() {
    val newYear = LocalDate.ofYearDay(2042, 1)
    val daysOff = newYear.minusDays(1)..newYear
    for (dayOff in daysOff) {
        println(dayOff)
    }
    // 2041-12-31
    // 2042-01-01
  }

   ```

------

## 9.4 component 함수를 사용해 구조 분해 선언 제공  
### 구조 분해 선언 (Destructuring Declaration)
- 복합적인 데이터를 한 번에 여러 변수로 분해하여 할당하는 기법이다. 
- 데이터 클래스나 컬렉션, 특정 객체의 여러 속성 값을 한꺼번에 추출할 때 사용된다. 
- 구조 분해를 통해 여러 개의 변수를 동시에 초기화할 수 있어 코드가 간결해진다


#### 구조 분해 선언 기본 사용법
```kotlin
data class Point(val x: Int, val y: Int)

fun main() {
    val p = Point(10, 20)
    val (x, y) = p
    println(x) // 10
    println(y) // 20
}
```
- val (x, y) = p 구문을 통해 p 객체의 두 속성 값을 각 변수에 할당한다. 
- 이는 내부적으로 아래와 같이 componentN() 함수 호출로 변환된다
  ```kotlin
  val x = p.component1()
  val y = p.component2()
   ```
- 구조 분해 선언은 데이터 클래스에서 기본으로 제공된다. 
- 데이터 클래스의 주 생성자에 포함된 프로퍼티를 자동으로 componentN() 함수로 변환한다.

#### componentN 함수 구현
- 데이터 클래스가 아니더라도, componentN() 함수를 수동으로 구현할 수 있다. 
- 각 속성에 대해 component1(), component2() 같은 함수를 만들면 구조 분해가 가능하다.
```kotlin
class Point(val x: Int, val y: Int) {
    operator fun component1() = x
    operator fun component2() = y
}

fun main() {
    val p = Point(10, 20)
    val (x, y) = p
    println(x)  // 10
    println(y)  // 20
}
```
- operator 키워드를 사용해 componentN 함수를 구현한다. 
- N의 번호는 구조 분해 선언의 변수 순서를 따른다. 
- p.component1()과 p.component2()를 통해 구조 분해가 이루어진다.

#### 함수에서 여러 값을 반환할 때 활용
- 함수에서 여러 값을 반환해야 할 때 구조 분해를 사용하면 편리하다.
  ```kotlin
  data class NameComponents(val name: String, val extension: String)
  
  fun splitFilename(fullName: String): NameComponents {
    val result = fullName.split('.', limit = 2)
    return NameComponents(result[0], result[1])
  }
  
  fun main() {
    val (name, ext) = splitFilename("example.kt")
    println(name)  // example
    println(ext)   // kt
  }
  ```
  - `split()` 함수를 사용하여 파일 이름과 확장자를 분리한다.

#### 구조 분해와 컬렉션
- 컬렉션에서도 구조 분해를 사용할 수 있다.
  ```kotlin
  data class NameComponents(
    val name: String, 
    val extension: String)
  
  fun splitFilename(fullName: String): NameComponents {
    val (name, extension) = fullName.split('.', limit = 2)
    return NameComponents(name, extension)
  }
  ```
  
### 구조 분해 선언과 루프
- 맵(Map) 의 원소를 순회할 때 구조 분해를 사용하면 가독성이 좋아진다.
  ```kotlin
  fun printEntries(map: Map<String, String>) {
    for ((key, value) in map) {
        println("$key -> $value")
    }
  }
  
  fun main() {
    val map = mapOf("Oracle" to "Java", "JetBrains" to "Kotlin")
    printEntries(map)
    // Oracle -> Java
    // JetBrains -> Kotlin
  }
  ```
  - Map.Entry에 대해 구조 분해를 사용하여 키와 값을 한꺼번에 할당한다. 
  - 내부적으로는 다음과 같이 변환된다.
    ```kotlin
    for (entry in map.entries) {
        val key = entry.component1()
        val value = entry.component2()
    }
    ```
  - 키와 값을 직접 분해하여 코드를 간결하게 작성할 수 있다.

### 구조 분해와 무시 변수: `_`
- 구조 분해 중 일부 변수가 필요하지 않다면 **_**를 사용하여 무시할 수 있다. 
- 사용하지 않는 구조 분해 값 무시하기
  ```kotlin
  data class Person(
    val firstName: String, 
    val lastName: String,   
    val age: Int, 
    val city: String
  )

  fun introducePerson(p: Person) {
    val (firstName, _, age) = p
    println("This is $firstName, aged $age.")
  }
  
  fun main() {
    val person = Person("John", "Doe", 30)
    introducePerson(person)  // This is John, aged 30.
  }
  ```
  - 구조 분해 중 필요 없는 값은 _로 대체하여 할당을 무시할 수 있다. 
  - 이를 통해 불필요한 변수 생성을 방지하고 코드의 가독성을 높인다.

> **코틀린 구조 분해의 한계와 단점**   
> 코틀린의 구조 분해 선언은 위치에 의존적으로 동작한다.     
> 즉, 구조 분해 시 값의 매핑은 변수의 이름이 아닌 생성자의 정의 순서에 따라 결정된다.    
> 이 때문에 생성자 순서가 바뀌거나 필드가 추가되면 잘못된 값이 변수에 할당될 수 있다.    
> 이러한 특성 때문에 구조 분해는 작고 변경 가능성이 적은 데이터 클래스에만 적합하며, 복잡한 엔티티에서는 리팩토링 시 큰 문제를 일으킬 수 있다.    
> 이를 개선하기 위해 이름 기반 구조 분해가 논의 중이며, 향후 값 클래스(Value Class)가 도입되면 해결될 것으로 기대된다.    

----

## 9.5 프로퍼티 접근자 로직 재활용: 위임 프로퍼티
### 위임 프로퍼티(Delegated Property)란?
- 위임 프로퍼티는 객체가 직접 값을 가지지 않고, 위임 객체가 값을 처리하도록 맡기는 디자인 패턴이다. 
- 코틀린에서는 by 키워드를 통해 프로퍼티 접근 로직을 다른 객체에 위임할 수 있다. 
- 위임된 객체가 getValue와 setValue 메서드를 오버라이딩하여 값을 읽고, 쓸 수 있다.
```kotlin
class Foo {
    var p: String by Delegate()
}
```
- 위의 예시에서 p 프로퍼티는 직접 값을 가지지 않고, Delegate 객체에 의해 값을 저장하고 읽는다. 
- Delegate 클래스가 getValue, setValue 메서드를 제공하면 p의 getter와 setter가 자동으로 연결된다.

### 위임 프로퍼티의 기본 문법과 내부 동작
- 위임 프로퍼티의 기본 문법은 다음과 같다
  ```kotlin
  var p: Type by Delegate()
  ```
  - p 프로퍼티는 Delegate() 객체에 의해 값이 관리된다. 
  - by 뒤의 객체가 위임 객체가 된다.
    
- 위 코드는 다음과 같이 내부적으로 변환된다
  ```kotlin
  class Foo {
    private val delegate = Delegate()
    var p: Type
        set(value: Type) = delegate.setValue(/*....*/, value)
        get() = delegate.getValue(/*....*/)
  }
   ```
  - by 키워드를 사용하면 getValue, setValue 메서드가 위임 객체에서 호출된다. 
  - 프로퍼티의 접근 로직을 직접 작성할 필요 없이, 재사용 가능하도록 캡슐화할 수 있다.

### 위임 프로퍼티 사용: by lazy()를 사용한 지연 초기화
#### 지연 초기화(Lazy Initialization)란?
- 지연 초기화는 객체의 일부를 필요할 때까지 초기화하지 않고 미뤄두는 디자인 패턴이다. 
- 메모리 사용 최적화와 성능 향상을 목적으로 사용된다. 
- 보통 초기화 비용이 큰 객체나, 반드시 초기화되지 않아도 되는 값들을 나중에 사용할 때 로딩한다.
- 데이터베이스, 네트워크 요청, 큰 컬렉션 로딩 등 비용이 큰 작업에 유용하다.
- 코틀린에서는 이를 간단하게 처리하기 위해 by lazy 키워드를 제공한다. 

#### 지연 초기화를 수동으로 구현
```kotlin
class Email { /*...*/ }

fun loadEmails(person: Person): List<Email> {
    println("${person.name}의 이메일을 가져옵니다.")
    return listOf(/*...*/)
}

class Person(val name: String) {
    private var _emails: List<Email>? = null  

    val emails: List<Email>
        get() {
            if (_emails == null) {
                _emails = loadEmails(this) // 최초 접근 시에만 초기화
            }
            return _emails!!
        }
}

fun main() {
    val p = Person("Alice")
    p.emails  // Load emails for Alice
    p.emails  // 두 번째 호출부터는 캐싱된 값을 반환
}

```
- emails 프로퍼티는 초기에는 null로 존재한다. 
- 최초로 접근했을 때만 loadEmails가 실행되어 값이 로딩된다. 
- 두 번째 호출부터는 캐싱된 _emails 값을 그대로 반환한다.
- `_emails`: 뒷받침하는 프로퍼티(Backing Property)
  - 실제 값을 저장하는 역할을 한다. 
  - 외부에서 접근할 수 없도록 private으로 선언된다. 
- `emails`: 공개 프로퍼티(Public Property)
  - getter를 통해 _emails의 값을 반환한다. 
  - 최초 접근 시 초기화 로직이 실행되며, 두 번째부터는 캐싱된 값을 반환한다.

#### `by lazy`를 사용한 간소화
- 위의 수동 구현을 by lazy로 간단하게 처리할 수 있다.
  ```kotlin
  class Person(val name: String) {
    val emails by lazy { loadEmials(this) }
  }
  ```
  - by lazy를 사용하면, 직접 초기화 체크를 할 필요가 없다. 
  - 최초 접근 시에만 초기화가 진행되고, 이후에는 캐싱된 값이 반환된다. 
  - 내부적으로 스레드 안전(Thread-safe) 하게 동작하며, 동기화 처리도 알아서 해준다.

### 위임 프로퍼티 구현
- 위임 프로퍼티를 구현하는 방법을 통해 속성 값이 변경될 때 자동으로 변경 통지를 받을 수 있다. 
- 이를 **옵저버블 프로퍼티(Observable Property)** 라고 부르며, UI나 데이터 동기화에 유용하다.

#### 옵저버(Observer)와 옵저버블(Observable)
- Observer: 값이 변경되면 알림을 받는 객체 
- Observable: 값이 변경되면 등록된 Observer들에게 알림을 전달하는 객체
```kotlin
// Observer 인터페이스 정의
fun interface Observer {
    fun onChange(propertyName: String, oldValue: Any?, newValue: Any?)
}

// Observable 클래스 정의
open class Observable {
    private val observers = mutableListOf<Observer>()

    // 옵저버 등록
    fun addObserver(observer: Observer) {
        observers += observer
    }

    // 변경 사항을 모든 옵저버에게 통지
    fun notifyObservers(propertyName: String, oldValue: Any?, newValue: Any?) {
        observers.forEach { it.onChange(propertyName, oldValue, newValue) }
    }
}
```

#### Person 클래스에 옵저버 기능 추가
- 옵저버 기능을 추가한 Person 클래스
  ```kotlin
  class Person(val name: String, age: Int, salary: Int) : Observable() {
  
      var age: Int = age
          set(newValue) {
              val oldValue = field
              field = newValue
              notifyObservers("age", oldValue, newValue)
          }
  
      var salary: Int = salary
          set(newValue) {
              val oldValue = field
              field = newValue
              notifyObservers("salary", oldValue, newValue)
          }
  }
  ```
  - age와 salary 값이 변경될 때마다 notifyObservers가 호출된다. 
  - 옵저버들에게 변경 사항이 전달된다.
- 실행 예시
  ```kotlin
    fun main() {
       val person = Person("Seb", 28, 1000)
  
      person.observer += Observer { propName, oldValue, newValue ->
      println(
       """ 
       Property $propName changed from $oldValue to $newValue!
       """.trimIndent()
      )
    }
  
    person.age = 29
    // Property age changed from 28 to 29
    person.salary = 1500
    // Property salary changed from 1000 to 1500
  }
   ```
  - 옵저버가 등록된 후, `age` 와 `salary` 값이 변경되면 자동으로 변경 사항을 출력한다.

#### 리팩토링: 중복된 로직 추출하기
- `age` 와 `salary` 의 중복된 로직을 ObservableProperty로 추출한다
   ```kotlin
    class ObservableProperty(
          val propName: String,
          var propValue: Int,
          val observable: Observable
  ) {
    fun getValue(): Int = propValue
  
    fun setValue(newValue: Int) {
      val oldValue = propValue
      propValue = newValue
      observable.notifyObservers(propName, oldValue, newValue)
    }
  }
    ```
  - `ObservableProperty` 는 값을 저장하고, 변경 시 옵저버에게 통지하는 역할을 한다.
```kotlin
class Person(val name: String, age: Int, salary: Int) : Observable() {
    val _age = ObservableProperty("age", age, this)
    var age: Int
        get() = _age.getValue()
        set(newValue) = _age.setValue(newValue)

    val _salary = ObservableProperty("salary", salary, this)
    var salary: Int
        get() = _salary.getValue()
        set(newValue) = _salary.setValue(newValue)
}
```
- 중복된 로직을 ObservableProperty로 옮겨서 깔끔하게 관리한다. 
- 값 변경 시 자동으로 옵저버에게 전달된다.

#### 위임 프로퍼티로 리팩토링
- 코틀린의 위임 프로퍼티를 사용하면 더 간단하게 표현할 수 있다.
  ```kotlin
  import kotlin.reflect.KProperty

  class ObservableProperty(
  var propValue: Int,
  val observable: Observable
  ) {
    operator fun getValue(thisRef: Any?, prop: KProperty<*>): Int = propValue
    operator fun setValue(thisRef: Any?, prop: KProperty<*>, newValue: Int) {
        val oldValue = propValue
        propValue = newValue
        observable.notifyObservers(prop.name, oldValue, newValue)
    }
  }
  ```
  - `getValue`와 `setValue`를 오버라이딩하여 위임을 처리한다. 
  - KProperty를 통해 속성의 이름을 알 수 있다.

#### 위임 프로퍼티를 통한 Person 리팩토링
```kotlin
class Person(val name: String, age: Int, salary: Int) : Observable() {
    var age by ObservableProperty(age, this)
    var salary by ObservableProperty(salary, this)
}
```
- `by` 키워드를 통해 프로퍼티를 간단하게 위임할 수 있다. 
- 값이 변경될 때마다 자동으로 `ObservableProperty`가 처리한다.

#### Delegates.observable을 활용한 리팩토링
- 코틀린 표준 라이브러리에서 제공하는 Delegates.observable을 사용할 수 있다.
  ```kotlin
  import kotlin.properties.Delegates

  class Person(val name: String, age: Int, salary: Int) : Observable() {
  
      private val onChange = { prop: KProperty<*>, oldValue: Int, 
      newValue: Int ->
          notifyObservers(prop.name, oldValue, newValue)
      }
  
      var age by Delegates.observable(age, onChange)
      var salary by Delegates.observable(salary, onChange)
  }
  ```
  - Delegates.observable을 통해 값 변경 시 자동으로 옵저버에게 전달된다. 
  - 직접 ObservableProperty를 만들 필요 없이 간단하게 처리된다.

### 위임 프로퍼티는 커스텀 접근자가 있는 감춰진 프로퍼티로 변환된다
- 위임 프로퍼티는 내부적으로 커스텀 접근자가 있는 감춰진 프로퍼티로 변환된다.
  ```kotlin
  class C {
    var prop: Type by MyDelegate()
  }
  ```
  - 내부적으로 컴파일러가 다음과 같이 변환한다.
    ```kotlin
    class C {
    private val <delegate> = MyDelegate()
    var prop: Type
        get() = <delegate>.getValue(this, <property>)
        set(value: Type) = <delegate>.setValue(this, <property>, value)
    }
    ```
    - getValue와 setValue가 감춰진 프로퍼티에서 동작한다. 
    - KProperty를 통해 프로퍼티 이름과 정보를 전달받아 사용한다.

### 맵에 위임해서 동적으로 애트리뷰트 접근
- 자신의 프로퍼티를 동적으로 정의할 수 있는 객체를 만들 때, 위임 프로퍼티를 활용할 수 있다. 
- C#에서는 이를 ExpandoObject라고 부르며, 코틀린에서는 MutableMap을 사용하여 구현할 수 있다. 
- 예를 들어, 연락처 관리 시스템에서 추가적인 정보를 동적으로 저장할 수 있다.

- 다음은 수동으로 Map에 값을 저장하고 가져오는 예시이다.
  ```kotlin
  class Person {
    // 속성을 담을 MutableMap 정의
    private val _attributes = mutableMapOf<String, String>()

    // 속성 추가 메서드
    fun setAttribute(attrName: String, value: String) {
        _attributes[attrName] = value
    }

    // 프로퍼티 정의
    var name: String
        get() = _attributes["name"]!!
        set(value) {
            _attributes["name"] = value
        }
  }
  
  fun main() {
  val person = Person()
  val data = mapOf("name" to "Seb", "company" to "JetBrains")
  
      for ((attrName, value) in data) {
          person.setAttribute(attrName, value)
      }
  
      println(person.name)  // Seb
      person.name = "Sebastian"
      println(person.name)  // Sebastian
  }
   ```
  - `_attributes`라는 MutableMap에 모든 속성을 저장한다. 
  - 프로퍼티 접근 시 Map에 저장된 값을 반환하고, 변경 시 Map을 업데이트한다. 
  - setAttribute를 통해 속성을 추가할 수 있다.
#### 위임 프로퍼티로 개선하기
- 코틀린에서는 by 키워드를 사용하여 **맵(Map)** 을 위임 프로퍼티로 설정할 수 있다. 
- 이를 통해 수동으로 get과 set을 정의하지 않아도 된다.
  ```kotlin
  class Person {
      private val _attributes = mutableMapOf<String, String>()
  
      // 속성 추가 메서드
      fun setAttribute(attrName: String, value: String) {
          _attributes[attrName] = value
      }
  
      // Map을 위임 프로퍼티로 사용
      var name: String by _attributes
  }
  ```
  - `by _attributes`로 설정하면, 프로퍼티의 `get`과 `set`이 Map의 값을 직접 읽고 쓴다. 
  - 기존에 Map에서 직접 꺼내거나 넣을 필요 없이, 프로퍼티 접근만으로 값을 조작할 수 있다.
- `by _attributes`가 동작하는 이유는 코틀린 표준 라이브러리에서 `MutableMap`에 대해 `getValue`와 `setValue`를 확장 함수로 제공하기 때문이다.

### 실전 프레임워크가 위임 프로퍼티를 활용하는 방법
- 위임 프로퍼티는 프레임워크에서도 유용하게 사용된다. 
- 예를 들어, 데이터베이스 테이블에 저장된 값을 프로퍼티처럼 다루기 위해 사용된다.

- 데이터베이스에 Users라는 테이블이 있고, name, age 컬럼이 있다고 가정한다.
  ```kotlin
    object Users : IdTable() {
        val name = varchar("name", length = 50).index()
        val age = integer("age")
    }
    ```
  - Users는 데이터베이스 테이블을 표현하며, 그 안의 name, age는 컬럼을 나타낸다.
- User 클래스를 정의하고, 데이터베이스 컬럼을 위임 프로퍼티로 사용한다.
  ```kotlin
  class User(id: EntityID) : Entity(id) {
     var name: String by Users.name
     var age: Int by Users.age
  }
    ```
  - by 키워드를 사용하여 Users.name과 Users.age를 프로퍼티처럼 사용한다. 
  - 데이터베이스에서 값을 가져오거나 변경하면, 자동으로 동기화된다.
- 코틀린의 getValue와 setValue 메서드를 사용하여 DB 접근이 이루어진다.
  ```kotlin
  operator fun <T> Column<T>.getValue(o: Entity, desc: KProperty<*>): T {
    // 데이터베이스에서 값을 읽어옴
  }
  
  operator fun <T> Column<T>.setValue(o: Entity, desc: KProperty<*>, value: T) {
  // 데이터베이스에 값을 기록함
  }
  ```
  - getValue는 데이터베이스에서 값을 읽어오고, setValue는 데이터베이스에 값을 업데이트한다.
------

## 요약
- 코틀린은 정해진 이름의 함수를 정의함으로써 표준적인 수학 연산을 오버로드할 수 있게 해준다. 자신만의 연산자를 정의할 수는 없지만 중위 함수를 더 표현력이 좋은 대안으로 사용할 수 있다.
- 비교 연산자(==, !=, >, < 등)를 모든 객체에 사용할 수 있다. 비교 연산자는 equals와 compareTo 메서드 호출로 변환된다.
get, set, contains라는 함수를 정의하면 코틀린 컬렉션과 비슷하게 클래스의 인스턴스에 대해 []와 in 연산을 사용할 수 있다.
- 미리 정해진 관례를 따라 범위를 만들거나 컬렉션과 배열의 원소를 이터레이션할 수 있다.
- 구조 분해 선언을 통해 한 객체의 상태를 분해해서 여러 변수에 대입할 수 있다. 함수가 여러 값을 한꺼번에 반환해야 하는 경우 구조 분해가 유용하다. 데이터 클래스에 대해 구조 분해를 거저 사용할 수 있지만, 자신의 클래스에 componentN 함수를 정의하면 구조 분해를 지원할 수 있다.
- 위임 프로퍼티를 통해 프로퍼티 값을 저장하거나 초기화하거나 읽거나 변경할 때 사용하는 로직을 재활용할 수 있다. 위임 프로퍼티는 프레임워크를 만들 때 아주 강력한 도구로 쓰인다.
- 표준 라이브러리 함수인 lazy를 통해 지연 초기화 프로퍼티를 쉽게 구현할 수 있다.
- Delegates.observable 함수를 사용하면 프로퍼티 변경을 관찰할 수 있는 옵저버를 쉽게 추가할 수 있다.
- 맵을 위임 객체로 사용하는 위임 프로퍼티를 통해 다양한 속성을 제공하는 객체를 유연하게 다룰 수 있다.
















  












