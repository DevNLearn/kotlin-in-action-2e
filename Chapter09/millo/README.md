
# 연산자 오버로딩과 관례


**다루는 내용**

- 연산자 오버로딩
- 관례: 어떤 언어 기능과 미리 정해진 이름의 함수를 연결해주는 기법
- 위임 프로퍼티

**관례에 의존하는 코틀린**

언어 기능을 타입에 의존하는 자바와 달리 코틀린은 관례를 채택했다

- 이유는 고정되어 있는 기존 자바 클래스를 코틀린에 적용하기 위해서이다.
- 확장 함수를 구현하면서 관례에 따라 이름을 붙이면 기존 자바 코드 수정 없이 새로운 기능을 부여할 수 있다

## 산술 연산자 오버로딩

자바는 오직 기본 타입에 대해서만 산술 연산자를 사용할 수 있다

- String도 BigInteger도 컬렉션도 산술 연산자를 사용할 수 없다
- 위대하신 코틀린은 가능하다

### **이항 산술 연산 오버로딩**

```kotlin
data class Point(val x: Int, val y: Int) {
    operator fun plus(other: Point): Point {
        return Point(x + other.x, y + other.y)
    }
}

fun main() {
    val point1 = Point(1, 2)
    val point2 = Point(3, 4)
    println(point1 + point2) // Output: Point(x=4, y=6)
}
```

- 연산자를 오버로딩할 땐 함수 앞에 `operator` 키워드를 붙여 관례를 따르는 함수임을 명시해야 한다
- `+` 연산자가 `plus` 함수 호출로 변환된다
- 위 예제처럼 멤버 함수로 정의해도 되고, 확장 함수로 정의해도 된다.

**코틀린이 열어둔 연산자**

- 직접 연산자를 만들순 없고 코틀린 언어가 지정해둔 연산자만 오버로딩할 수 있다
- 관계를 따르기 위한 함수 이름 또한 연산자별로 지정되어 있다

| 지정된 연산자(중위 함수) 식 | 지정된 함수 이름 |
| --- | --- |
| a * b | times |
| a / b | div |
| a % b | mod |
| a + b | plus |
| a - b | minus |

**이항 연산자 함수 규칙**

1. 오버로딩을 했더라도 연산자간 우선순위는 표준 연산자 우선순위와 동일하게 동작한다
    - ex) 항상 `+` 보다 `*` 가 먼저 수행
2. 연산자의 두 피연산자가 같은 타입일 필요는 없다

    ```kotlin
    operator fun Point.times(scale: Double): Point {
        return Point((x * scale).toInt(), (y * scale).toInt())
    }
    
    fun main() {
        val p = Point(10, 20)
        println(p * 1.5) // Point(x=15, y=30)
    }
    ```

3. 코틀린 연산자는 교환법칙을 따르지 않는다
    - `a * b`와 `b * a`가 같으려면 양방향의 `operator fun`을 정의해야 한다
4. 연산자 함수의 반환 타입이 꼭 두 피연산자 중 하나와 일치하지 않아도 된다
5. 일반 함수와 마찬가지로 `operator` 함수도 오버로딩할 수 있다
    - 이름은 같지만 파라미터 타입이 서로 다른 연산자 함수를 여러 개 만들 수 있다

**코틀린은 표준 숫자 타입에 대해 직접 비트 연산자를 정의하지 않는다**

- 내부적으로 Java의 비트 연산자를 사용할 수 있도록 오버라이딩된 함수를 제공한다

| 연산 종류 | 자바 | 코틀린(`infix fun`) | 설명 |
| --- | --- | --- | --- |
| AND | `&` | `and(other)` |  |
| OR | `L` | `or(other)` |  |
| XOR | `^` | `xor(other)` |  |
| INV(NOT) | `~` | `inv()` | 반전 |
| SHL | `<<` | `shl(bitCount)` | shl |
| SHR | `>>` | `shr(bitCount)` | 부호 유지하고 shr |
| USHR | `>>>` | `ushr(bitCount)` | 부호 무시하고 shr |

### 복합 대입 연산자 오버로딩

- 이항 연산자를 오버로딩 하면 그와 관련있는 복합 대입 연산자도 자동으로 지원한다
    - ex) `+` 연산자 오버로딩 → `+=` 연산자 지원
    - 컴파일러가 자동으로 `a += b`를 `a = a + b`로 변환해준다
    - 실제 내부 상태를 바꾸는 게 아니라 객체의 참조가 바뀌는 연산이기에(재할당) `val` 객체에 대해서는 `plus()` 오버로딩만으론 `+=`를 사용할 수 없다

**그렇다면 `+=` 연산이 실제 객체의 내부 상태를 변경하게 하고 싶다면?**

- 반환 타입이 `Unit`인 `xxxAssign` 함수를 `operator`로 정의하면 된다
    - 객체 내부의 상태를 직접 바꾸는 함수이므로 아무 것도 반환하지 말라는 뜻으로 반환타입을 `Unit`으로 강제한다
- `var` 뿐만 아니라 `val`에 대해서도 `+=` 연산이 가능하다
    - `val`은 참조 자체는 불변이지만 참조된 객체 내부 상태는 변경 가능하기 때문이다

**복합 대입 연산자 오버로딩 주의**

- 코드에서 `+=`를 만났을 때 컴파일러는 2가지 방식 중 하나로 처리한다
    1. `plusAssign()`이 정의되어 있는 경우 호출
    2. 없을 경우 `plus()` 호출하고 `a = a + b`로 변환(재할당)
- 그런데 `plus()`, `plusAssign()` 둘 다 정의되어 있을 경우 컴파일러는 어떤 걸 써야할지 결정할 수 없어서 오류를 발생킨다
    - 해결법
        1. 중위 함수가 아닌 일반 함수로 호출하기
        2. `plus`와 `plusAssign` 연산을 동시에 적용하지 않기
            - 불변 객체는 `plus()`만 정의해 `+=`를 아예 사용 못하도록 한다
            - 변경 가능한 객체에는 `plusAssign()`만 정의해 내부 상태를 바꿀 수 있도록 한다
                - 변경 가능한 객체인데 새 객체를 만드는 `plus()`는 굳이 만들 필요가 없으니 정의하지 말자

**컬렌션 복합 대입 연산자 활용**

- 코틀린 표준 컬렉션은 이미 `plus`, `plusAssign`, `minus`, `minusAssign`을 오버로딩해 제공한다

| 컬렌션 타입 | 복합 대입 연산자 동작 |
| --- | --- |
| 불변 컬렉션
(`List`, `Map`) | 새로운 컬렉션 재할당
|
| 변경 가능 컬렉션
(`MutableList`, `MutableMap`) | 컬렉션 내부 직접 변경 |

### 단항 연산자 오버로딩

- 이항 연산자와 마찬가지로 미리 정해진 이름을 operator로 정의하면 된다

| 단항 연산자 식 | 지정된 함수 이름 |
| --- | --- |
| `+a` | unaryPlus |
| `-a` | unaryMinus |
| `!a` | not |
| `++a`, `a++` | inc |
| `--a`, `a--` | dec |

**예시) BigDecimal**

<img src="image.png" alt="image 1" width="50%">

```kotlin
var bd = BigDecimal.ZERO
pritln(bd++) // 0
pritln(bd) // 1
pritln(++bd) // 2
```

## 비교 연산자 오버로딩

코틀린은 산술 연산자와 마찬가지로 기본 타입 값 뿐만 아니라 모든 객체에 대해 비교 연산을 수행할 수 있다

- 자바는 equals나 compareTo를 호출해줘야 했지만 코틀린은 비교 연산자로 간결하게 대체 가능하다
- 조상인 Any(혹은 Any?)에 equals가 있으니 모든 객체에 대해 가능한 것

### **동등성 연산자 equals**

- `==` 또는 `!=` 비교 연산자 호출 시 equals 호출로 컴파일된다
    - data class의 경우 equals까지 자동으로 만들어주니 매우 편리
- 내부에서 인자가 null인지 검사하므로 nullable한 값에 대해서도 사용 가능하다
    - `a == b` 호출 시 `a?.equlas(b) ?: (b == null)`로 변환
    - a와 b가 모두 null일 경우에만 결과가 true

**동등성 연산자 ===**

- 자바 == 연산자와 같은 동작으로 두 피연산자가 서로 같은 객체를 가리키는지(원시 타입인 경우 두 값이 같은지) 비교한다
- equals 직접 구현 시 먼저 ===로 자신과의 비교를 통해 최적화할 수 있다

    ```kotlin
    class Point(val x: Int, val y: Int) {
        override fun equals(obj: Any?): Boolean {
            if (this === obj) return true // 자신과의 비교로 최적화
            if (obj !is Point) return false // 타입 비교
            return x == obj.x && y == obj.y // 프로퍼티 비교
        }
        
        override fun hashCode(): Int { ... }
    }
    ```


**Any의 equals에는 operator가 붙어있다**

- `==` 비교 연산자가 내부적으로 equals()를 호출하기 위해서 연산자 오버로딩을 허용하기 위해 `operator`를 명시한다
- Any의 자식 객체들은 operator를 붙이지 않아도 자동으로 상위 클래스의 operator가 지정되기에 명시하지 않아도 된다
- Any에서 상속받은 equals가 확장 함수보다 우선순위가 높기 때문에 equlas는 확장 함수로 정의할 수 없다

### 순서 연산자 compareTo

- Comparable 인터페이스를 구현한 객체는 compareTo 메서드를 통해 서로 다른 객체의 크기를 비교해 정수로 나타낸다.
- 자바의 경우 이를 위해 `e1.compareTo(e2)` 처럼 명시적으로 사용해야 한다
- 코틀린의 경우 compareTo에 대한 관례를 제공해 비교 연산자(`<`, `>`, `≤`, `≥`)를 통해 호출될 수 있도록 지원한다

**코틀린 표준 라이브러리의 compareValuesBy를 활용해 compareTo 쉽게 정의하기**

1. Comparable 인터페이스 구현 및 compareTo 오버라이드
2. compareValuesBy를 통해 여러 비교 함수들을 넣어 순차적으로 두 객체를 비교

```kotlin
class Person(
    val firstName: String, val lastNme: String
) : Comparable<Person> {
    override fun compareTo(other: Person) = 
        compareValuesBy(this, other, Person::firstName, Person::lastNme)
}
```

**그러나 직접 비교 로직을 구현하는 것이 속도가 더 빠르긴 하다**

- 이유는 넘겨주는 비교 함수들에서 리플렉션을 사용하게 되고 내부에서 Comparator<T> 객체를 동적으로 생성해 비교하기 때문이다

  <img src="image 1.png" alt="image 1" width="50%">

    - for 문으로 반복해서 비교 함수들을 적용하는 모습
- 아무리 inline으로 최적화 하더라도 JVM 수준에서 람다/함수 객체를 완전히 제거하기 어렵다

## 컬렉션 범위 관례

### `get`: 인덱스로 원소 접근하기

코틀린은 맵의 원소에 접근할 때 마치 자바의 배열처럼 `[]`을 사용해 원소를 get하거나 set한다

- Map 및 MutableMap 인터페이스에서 제공해주는 인덱스 접근 연산자를 사용해 읽을 땐 get() 쓸 땐 set() 연산자 메서드로 변환해주는 원리이다.

    ```kotlin
    public operator fun get(key: K): V?
    public operator fun set(index: Int, element: E): E
    ```


**객체에 get 관례 직접 구현해서 사용할 수도 있다**

- 마찬가지로 operator 붙여서 구현 후 `[]` 사용해서 접근하면 된다
- 반드시 관례의 파라미터로 Int가 아닌 다른 타입을 사용해도 되고, 여러 파라미터를 받는 관례를 정의할 수도 있다
- 불변 객체가 아닐 경우 set 관례도 직접 구현해서 사용할 수 있다

```kotlin
operator fun Point.get(index: Int) =
    when (index) {
        0 -> x
        1 -> y
        else -> throw IndexOutOfBoundsException("Index $index out of bounds for Point")
    }

fun main() {
    val point = Point(10, 20)
    println(point[0]) // Output: 10
    println(point[1]) // Output: 20
}
```

### `in`: 컬렉션에 존재하는지 검사

`in` 연산자로 객체가 컬렉션에 들어있는지 검사하는데 내부적으로 `contains` 함수가 호출된다

- `a in c` 호출 시 컴파일러는 `c.contains(a)`로 변환한다

**in 관례 구현 예시**

```kotlin
data class Rectangle(val upperLeft: Point, val lowerRight: Point) {
    operator fun contains(p: Point): Boolean {
        return p.x in upperLeft.x..<lowerRight.x && 
                p.y in upperLeft.y..<lowerRight.y
    }
}

fun main() {
    val rect = Rectangle(Point(0, 0), Point(10, 10))
    println(Point(5, 5) in rect) // true
    println(Point(15, 5) in rect) // false
}
```

참고) 여기서 범위를 끝값을 포함하지 않는 열린 범위 `..<` 연산자를 사용한 이유는 보통 사각형 표현 시 왼쪽 위 좌표는 포함, 오른쪽 아래 좌표는 미포함하는 경우가 많기 때문이다

### 객체로부터 범위 만들기

범위를 만드려면 `..` 구문을 사용했는데 이는 `rangeTo` 함수 호출로 변환된다

- `start..end` 호출 시 컴파일러는 `start.rangeTo(end)`로 변환한다
- `..<` 구문은 열린 범위를 만드는 `rangeUntil` 함수 호출로 대응된다
- 코틀린 표준 라이브러리에는 모든 `Comparable` 객체에 대해 적용 가능한 `rangeTo` 및 `rangeUntil` 함수가 확장 함수로 정의되어 있다 (`Ranges.kt`에서 확인 가능)

**범위 연산자는 우선순위가 낮다**

- `0..n + 1`은 `0..(n+1)`로 동작한다
- `0..n.forEach {}`는 우선순위가 낮아 컴파일 에러가 나므로 `(0..n).forEach {}`와 같이 범위를 괄호로 둘러싸야 한다

### iterator 관례

코틀린의 for 루프는 범위 검사와 동일하게 `in` 연산자를 사용하는데 이 때는 검사 목적의 in과 의미가 다르다.

- 검사 목적의 `in`이 `contatins`로 변환되었따면 루프의 `in`은 `iterator`로 변환된다
- `iterator()`를 호출해 자바와 마찬가지로 iterator를 얻어 `hasNext`, `next` 호출을 반복하는 식으로 변환된다
- 역시 iterator 또한 관례이기에 확장 함수로 정의할 수 있다
    - 실제로 코틀린 표준 라이브러리는 String의 상위 클래스인 CharSequence에 대한 iterator 확장 함수를 제공한다

        ```kotlin
        for (c in "abc") {
            print("$c ") // a b c
        }
        
        // 컴파일 시 아래처럼 변환
        val iterator = "abc".iterator()
        while (iterator.hasNext()) {
            val c = iterator.next()
            print("$c ")
        }
        ```


**iterator를 구현해 for 루프 안에서 LocalDate의 범위를 in으로 순회할 수 있도록 만들어보자**

```kotlin
operator fun ClosedRange<LocalDate>.iterator(): Iterator<LocalDate> =
    object : Iterator<LocalDate> {  // Iterator 구현
        private var current = start

        override fun hasNext(): Boolean {
            return current <= endInclusive  // 비교 연산자 compareTo 관례 사용
        }

        override fun next(): LocalDate {
            val result = current
            current = current.plusDays(1)
            return result
        }
    }

fun main() {
    val newYear = LocalDate.ofYearDay(2025, 1)
    val daysOff = newYear.minusDays(1)..newYear
    for (dayOff in daysOff) { println(dayOff) } // in 연산자 iterator 관례 사용
    // 2024-12-31
    // 2025-01-01
}
```

## 구조 분해 선언 제공: component()

코틀린의 구조 분해 선언은 복합적인 값을 분해해서 별도의 여러 지역 변수를 한꺼번에 초기화할 수 있도록 한 기능인데 이것 또한 관례를 이용한 특성이다

**구조 분해 선언**

```kotlin
data class Person(val name: String, val age: Int)

fun main() {
    val person = Person("Alice", 30)
    val (name, age) = person  // 구조 분해 선언
    println("Name: $name, Age: $age")  // Name: Alice, Age: 30
}
```

구조 분해 선언의 각 변수를 초기화하고자 `componentN`이라는 함수를 호출한다

- data class의 경우 주 생성자에 있는 프로퍼티에 대해 컴파일러가 자동으로 `componentN` 함수를 만들어준다
- 여기서 N은 구조 분해 선언에 있는 변수 위치에 따라 붙는 번호로 위의 예시에서 name이 `component1()` age가 `component2()`가 된다

**실제로 컴파일해보면..**

- Person

    ```kotlin
    public final class Person {
       @NotNull
       private final String name;
       private final int age;
    
       @NotNull
       public final String component1() {
          return this.name;
       }
    
       public final int component2() {
          return this.age;
       }
       
       // getter & 생성자 & copy & equals & hashCode & toString
       // ..
    ```

- main 함수

    ```kotlin
       public static final void main() {
          Person person = new Person("Alice", 30);
          String name = person.component1();
          int age = person.component2();
          String var3 = "Name: " + name + ", Age: " + age;
          System.out.println(var3);
       }
    ```


**구조 분해를 활용해 여러 값 반환하기**

- 반환 타입을 데이터 클래스로 바꾸고 구조 분해 선언 구문을 사용하면 그 클래스를 쉽게 풀어 여러 변수에 넣을 수 있다

```kotlin
data class NameComponents(
    val name: String,
    val extension: String,
)

fun splitFilename(fullName: String): NameComponents {
    val parts = fullName.split(".", limit = 2)
    return NameComponents(parts[0], parts[1])
}

fun main() {
    val (name, ext) = splitFilename("test.txt")
    println("Name: $name Extension: $ext") // Name: test Extension: txt
}
```

**컬렉션에 대해 구조 분해 선언 사용하기**

- 크기가 정해진 컬렉션을 다루는 경우에도 구조 분해 선언으로 원소들에 접근할 수 있다
- 무한히 componentN을 선언할 수는 없으므로 코틀린 표준 라이브러리는 맨 앞 다섯 원소에 대한 componentN을 제공한다

**`Pair` & `Triple` vs 구조 분해 선언**

- `Pair`와 `Triple`은 여러 값을 간편하게 반환할 수 있는 수단이지만, 각 값을 `first`, `second`, `third`처럼 의미 없는 이름으로 접근해야 하기 때문에 코드의 표현력이 떨어진다
    - 물론 그 원소들을 바로 구조 분해 선언으로 받을 수 있다
    - `val (a, b) = pair`
- 반면 구조 분해 가능한 일반 객체(`data class` 등)는 명확한 프로퍼티 이름으로 접근하므로 코드의 표현력이 좋다

### 구조 분해 선언과 루프

변수 선언이 들어갈 수 있는 곳이면 어디든 구조 분해 선언을 사용할 수 있다

- 루프 안에서도 구조 분해 선언을 사용할 수 있는데 맵의 원소에 대해 이터레이션할 때 유용하다

```kotlin
fun printEntries(map: Map<String, String>) {
    for ((key, value) in map) {
        println("$key -> $value")
    }
}

fun main() {
    val map = mapOf("Oracle" to "Java", "Kotlin" to "JetBrains")
    printEntries(map) 
    // Prints: Oracle -> Java, 
    // Kotlin -> JetBrains
}
```

- 이 예제에 쓰인 2가지 코틀린 관례
    1. 루프 안 in 연산자
        - Map 확장 함수로 제공되는 `iterator` 호출
    2. 루프 안 구조 분해 선언
        - Map.Entry 확장 함수로 제공되는 `componentN`
- 컴파일 해보면 iterator() 호출해서 이터레이터 얻은 뒤 .. 엥? componentN 호출이 아닌 바로 getKey(), getValue() 호출하는데요?
    - 알아보니 원칙적으로는 componentN 호출이 맞지만 컴파일러가 성능 최적화를 위해 굳이 확장함수 거치지 않고 바로 getKey() / getValue()를 호출한다고 하네요~

    ```java
    public static final void printEntries(@NotNull Map map) {
        Intrinsics.checkNotNullParameter(map, "map");
        Iterator var2 = map.entrySet().iterator();
    
        while(var2.hasNext()) {
           Map.Entry var1 = (Map.Entry)var2.next();
           String key = (String)var1.getKey();
           String value = (String)var1.getValue();
           String var5 = key + " -> " + value;
           System.out.println(var5);
        }
    }
    ```


**람다가 복합적인 값을 파라미터로 받을 때도 구조 분해 선언을 사용할 수 있다**

```kotlin
fun main() {
    val map = mapOf("Oracle" to "Java", "Kotlin" to "JetBrains")
    
    map.forEach { (key, value) -> 
        println("$key -> $value")
    }
}
```

컴파일 해보면..

```java
   public static final void main() {
      Map map = MapsKt.mapOf(new Pair[]{TuplesKt.to("Oracle", "Java"), TuplesKt.to("Kotlin", "JetBrains")});
      int $i$f$forEach = false;
      Iterator var3 = map.entrySet().iterator();

      while(var3.hasNext()) {
         Map.Entry element$iv = (Map.Entry)var3.next();
         int var6 = false;
         String key = (String)element$iv.getKey();
         String value = (String)element$iv.getValue();
         String var7 = key + " -> " + value;
         System.out.println(var7);
      }

   }
```

- forEach 사용 시 iterator 호출로 대체되고 구조 분해 선언 또한 getKey() 및 getValue()로 대체

### _ 문자로 구조 분해 값 무시

구조 분해 사용 시 변수 중 일부가 필요없는 경우 `_` 문자로 표현해 깔끔하게 코드르 작성할 수 있다

- 앞쪽의 원소만 필요할 경우 사용하지 않는 뒤쪽 `componentN`은 아예 생략할 수도 있다
- 구조 분해 연산의 결과는 오직 인자의 위치에 따라 결정된다
    - 따라서 제일 뒤쪽이 아닌 이상 앞쪽의 원소는 생략할 수 없고 반드시 `_` 를 사용해 무시해야 한다
    - 타입 정보가 명시적이지 않기에 순서가 바뀌는 것을 주의해야 한다
        - 특히 데이터 클래스의 프로퍼티의 순서가 바뀌면 기존의 구조분해 선언문들이 원치 않은 동작을 하게 된다

## 위임 프로퍼티

위임 프로퍼티를 사용하면 값을 뒷받침하는 백킹 필드에 단순히 저장하는 것보다 더 복잡한 방식으로 작동하는 프로퍼티를 접근자 로직을 매번 재구현할 필요 없이 쉽게 구현할 수 있다

- 더 복잡하다? 백킹 필드가 단순한 저장소라면 프로퍼티는 그 값을 저장하는 것을 넘어서서 제어하거나 조작할 수 있는 인터페이스이기 때문

**코틀린에서 위임(delegate)이란?**

- 객체가 직접 작업을 수행하지 않고 위임 객체에게 그 작업을 맡기는 패턴
- 이 위임 패턴을 프로퍼티에 적용해 접근자 기능을 위임 객체가 수행하도록 하는 것이 위임 프로퍼티
    - 즉, 복잡한 get/set 로직을 재사용 가능하게 하는 추상화 관례

### 위임 프로퍼티 동작

- `by` 키워드 위에 있는 식을 계산해서 위임 객체를 얻는다
- 위임은 위임 객체가 따라야 하는 관례를 따르는 모든 객체를 위임 객체로 사용할 수 있다
    - 위임 관례는 `getValue`와 `setValue` 메서드를 제공해야 한다
        - 변경 가능한 프로퍼티만 `setValue`를 사용한다
    - 선택적으로 `provideDelegate` 함수 구현을 제공해 최초 생성 시 검증 로직을 수행하거나 위임이 인스터늣화되는 방식을 변경할 수 있다

```kotlin
class Delegate {
    override fun getValue(/* ... */) { /* ... */ }
    override fun setValue(/* ... */, value: Type) { /* ... */ }
    override fun providDelegate(/* ... */): Delegate { /* ... */ }
}

class Foo {
    var p: Type by Delegate() // by 키워드로 프로퍼티와 위임 객체 연결   
}

fun main() {
    /*
        위임 프로퍼티가 있는 Foo 객체 생성
        위임 객체가 provideDelegate() 제공하면 그 함수를 호출해서 위임 객체 생성
     */
    val foo = Foo()
    val oldValue = foo.p    // p 프로퍼티에 접근하면 내부에서 delegate.getValue() 호출
    foo.p = newValue        // p 프로퍼티에 값을 대입하면 내부에서 delegate.setValue() 호출
    
}
```

### 프로퍼티 지연 초기화

**지연 초기화(lazy initialization)?**

- 객체의 일부분을 초기화하지 않고 남겨뒀다가 필요할 때 초기화하는 패턴
- 꼭 미리 초기화하지 않아도 되는 객체에 대해 적용 가능
- 예시
    - JPA `FetchType.LAZY` : 연관 엔티티 실제 접근 시점까지 지연
    - 스프링 빈 `@Lazy`: 실제 빈을 사용할 때까지 지연

**프로퍼티 지연 초기화**

백킹 프로퍼티 기법을 적용해 프로퍼티에 지연 초기화 패턴을 적용할 수 있다

- 백킹 프로퍼티에 실제 값을 저장하고 읽기 전용 프로퍼티로 백킹 프로퍼티에 대한 읽기 연산을 외부에 제공하는 방식
- 백킹 프로퍼티는 지연 초기화이므로 nullable하고 일반 프로퍼티는 지연 초기화되므로 non-null해서 두 타입의 프로퍼티를 모두 사용해야 한다
- 같은 개념을 표현하는 프로퍼티에 대한 관례는 백킹 프로퍼티 앞에 밑줄을 붙이는 것

```kotlin
class Person(val nme: String) {
    private var _emails: List<Email>? = null // 백킹 프로퍼티
    
    val emails: List<Email>
        get() {
            if (_emails == null) _emails = loadEmails(this) // lazy 초기화
            return _emails!!
        }
}
```

- 외부에서는 emails 프로퍼티만 접근 가능
- 실제 초기화 시점은 emails 프로퍼티에 접근해 처음 get()이 호출될 때

**백킹 프로퍼티 기반 지연 초기화의 단점**

1. 지연 초기화되는 프로퍼티가 많아지면 프로퍼티마다 _xxx, xxx 2개의 프로퍼티를 관리해야 해서 복잡해지고 가독성이 떨어진다
2. 구현 자체가 스레드 안전하지 않기 때문에 멀티 스레드가 모두 백킹 프로퍼티인 emails에 접근하면 무거운 연산인 loadEmails 초기화 로직이 여러번 호출될 수 있다

### **by lazy() 기반 지연 초기화**

**lazy 함수?**

```kotlin
public actual fun <T> lazy(initializer: () -> T): Lazy<T> = SynchronizedLazyImpl(initializer)
public actual fun <T> lazy(lock: Any?, initializer: () -> T): Lazy<T> = SynchronizedLazyImpl(initializer, lock)
```

- 초기화 로직을 담은 람다를 받아서 `Lazy<T>` 를 구현한 `SynchronizedLazyImpl`를 반환한다

SynchronizedLazyImpl의 내부 구현을 보면 사실 이전의 백킹 프로퍼티로 지연 초기화한 코드와 유사하다

```kotlin
private class SynchronizedLazyImpl<out T>(initializer: () -> T, lock: Any? = null) : Lazy<T>, Serializable {
    private var initializer: (() -> T)? = initializer
    @Volatile private var _value: Any? = UNINITIALIZED_VALUE
    // final field is required to enable safe publication of constructed instance
    private val lock = lock ?: this

    override val value: T
        get() {
            val _v1 = _value
            if (_v1 !== UNINITIALIZED_VALUE) {
                @Suppress("UNCHECKED_CAST")
                return _v1 as T
            }

            return synchronized(lock) {
                val _v2 = _value
                if (_v2 !== UNINITIALIZED_VALUE) {
                    @Suppress("UNCHECKED_CAST") (_v2 as T)
                } else {
                    val typedValue = initializer!!()
                    _value = typedValue
                    initializer = null
                    typedValue
                }
            }
        }

    override fun isInitialized(): Boolean = _value !== UNINITIALIZED_VALUE

    override fun toString(): String = if (isInitialized()) value.toString() else "Lazy value not initialized yet."

    private fun writeReplace(): Any = InitializedLazyImpl(value)
}
```

- 백킹 프로퍼티 _value가 초기화 전이라는 뜻의 `UNINITIALIZED_VALUE` 마커로 초기화해뒀다가 첫 접근 시 전달받은 초기화 람다를 실행해 _value에 지연 초기화한다
- 이후 접근 시엔 백킹 프로퍼티로부터 저장된 값을 얻어서 Lazy<T>로 변환해 반환한다
- synchronized로 초기화 람다 수행 로직을 감싸 동기화했고, 초기화가 완료되면 초기화 람다를 null로 만들어 메모리에서 해제하고 GC 대상이 되도록 한다

**by 키워드 + lazy 함수**

by 키워드는 프로퍼티의 위임 객체를 지정하는 키워드였는데 lazy 함수의 반환 타입인 Lazy<T>는 이 위임 객체의 관례인 getValue를 제공하기에 가능한 것이다

- 이전의 백킹 프로퍼티 기반 지연 초기화를 by lazy로 리팩토링하면 아래와 같이 간결해진다

    ```kotlin
    class Person(val name: String) {
        val emails: List<Email> by lazy { loadEmails(this) }
    }
    ```

- 초기화가 늦게 되기는 하지만 한번 되고 나선 불변이기에 반드시 `val` 에서만 사용할 수 있다
    - 애초에 `Lazy<T>`가 `getValue`만 제공하기 때문에 `var` 에서 `by lazy` 쓰면 컴파일러가 `setValue`를 찾을 수 없어서 에러가 발생한다

### 옵저버 패턴: 위임 프로퍼티 활용

**옵저버 패턴**

- 객체의 상태가 바뀔 때 그 변화를 자동으로 다른 객체(옵저버 or 리스너)에게 알려주는 패턴
    - ex) UI의 객체가 변화면 UI도 자동으로 갱신

**기본 구조**

- **Subject - 주체, 관찰 대상 객체**
    - 옵저버를 등록/해제하고 상태 변화 시 알림을 전달하는 역할
    - 상태 변화를 감지하고 모든 등록된 옵저버에게 알림을 전파
- **Observer - 관찰자**
    - Subject의 상태를 관찰하고 변경이 발생했을 때 업데이트
    - 알림을 받으면 해당 정보를 바탕으로 동작을 수행

**Subject와 Observer의 느슨한 결합을 통해 DIP 원칙을 따른다.**

- 직접 참조하지 않고 Subject는 Observer의 인터페이스에만 의존한다
    - 어떤 구현체인지 알 수도 알 필요도 없음
    - `Observer` 인터페이스를 구현한 어떤 객체든 쉽게 교체하거나 확장 가능

**코틀린에서 옵저버 패턴 구현하기**

```kotlin
// 옵저버 함수형 인터페이스
fun interface Observer {
		// 상태 변화 알림 받았을 때 처리 로직
    fun onChange(name: String, oldValue: Any?, newValue: Any?)
}

// 옵저버 리스트 관리
open class Observable {
    val observers = mutableListOf<Observer>()
    
    // 상태 변화시 알림
    fun notifyObservers(propName: String, oldValue: Any?, newValue: Any?) {
        for (obs in observers) {
            obs.onChange(propName, oldValue, newValue)
        }
    }
}
```

- **v1)  일반적인 구현**

    ```kotlin
    class Person(val name: String, age: Int, salary: Int): Observable() {
        var age: Int = age
            set(newValue) {
                val oldValue = field
                field = newValue    // field 키워드 사용해 백킹 필드 접근
                notifyObservers("age", oldValue, newValue)
            }
    
        var salary: Int = salary
            set(newValue) {
                val oldValue = field
                field = newValue    // field 키워드 사용해 백킹 필드 접근
                notifyObservers("salary", oldValue, newValue)
            }
    }
    
    fun main() {
        val p = Person("John", 30, 1000)
        p.observers += Observer { propName, oldValue, newValue ->
            println("Property $propName changed from $oldValue to $newValue")
        }
        
        p.age = 31      // Property age changed from 30 to 31
        p.salary = 2000 // Property salary changed from 1000 to 2000
    }
    ```

    - 프로퍼티가 늘어날수록 중복 코드가 많이 생길 것

- **v2) 도우미 클래스로 프로퍼티 변경 통지 추출**

    ```kotlin
    // 프로퍼티 변경 통지 도우미 클래스
    class ObservableProperty(
        val propName: String,
        var propValue: Int,
        val observable: Observable
    ) {
        fun getValue() = propValue
        fun setValue(newValue: Int) {
            val oldValue = propValue
            propValue = newValue
            observable.notifyObservers(propName, oldValue, newValue)
        }
    }
    
    class Person(val name: String, age: Int, salary: Int): Observable() {
        private val _age = ObservableProperty("age", age, this)
        var age: Int
            get() = _age.getValue()
            set(newValue) { _age.setValue(newValue) }
    
        private val _salary = ObservableProperty("salary", salary, this)
        var salary: Int
            get() = _salary.getValue()
            set(newValue) { _salary.setValue(newValue) }
    }
    ```

    - 여전히 프로퍼티마다 ObservableProperty를 만들고 게터와 세터에서 위임하는 준비 코드가 많이 필요한 상황

- **v3) 위임 프로퍼티 사용: `by lazy`**
    - 위임 객체 관례에 맞게 ObservableProperty 수정 필요

    ```kotlin
    // 프로퍼티 위임 객체 ObservableProperty
    class ObservableProperty(
        var propValue: Int,
        val observable: Observable,
    ) {
        /**
         * 위임 객체 관례에 맞는 operator fun getValue/setValue
         * KProperty<*>는 프로퍼티에 대한 메타정보를 제공(이름, 타입 등)
         */
        operator fun getValue(thisRef: Any?, property: KProperty<*>) = propValue
        operator fun setValue(thisRef: Any?, property: KProperty<*>, newValue: Int) {
            val oldValue = propValue
            propValue = newValue
            observable.notifyObservers(property.name, oldValue, newValue)
        }
    }
    
    class Person(val name: String, age: Int, salary: Int) : Observable() {
        var age by ObservableProperty(age, this)
        var salary by ObservableProperty(salary, this)
    }
    ```

    - 코틀린 컴파일러가 상당 부분 직접 만들어준다.
    - 위임 객체를 감춰진 프로퍼티에 저장하고 주 객체의 프로퍼티를 읽거나 쓸 때마다 위임 객체의 getValue와 setValue를 호출한다

- **v4) 코틀린에서 제공하는 위임 클래스 사용: `Delegates.observable`**
    - 코틀린에서 제공하는 `Delegates.observable`는 프로퍼티의 값이 바뀔때마다 전/후 값을 인자로 받는 콜백을 실행할 수 있는 위임 객체를 생성해준다

    ```kotlin
    class Person(val name: String, age: Int, salary: Int) : Observable() {
        private val onChange = { property: KProperty<*>, oldValue: Any?, newValue: Any? ->
            notifyObservers(property.name, oldValue, newValue)
        }
        
        var age by Delegates.observable(age, onChange)
        var salary by Delegates.observable(salary, onChange)
    }
    ```

    - by 키워드 뒤에는 위임 객체 생성 뿐만 아니라 위임 객체 관례만 지킨다면(getValue, setValue 제공) 어느 식이든 올 수 있다


**Delegates.observable 내부 구현을 보면..**

```kotlin
public inline fun <T> observable(
    initialValue: T, 
    crossinline onChange: (property: KProperty<*>, oldValue: T, newValue: T) -> Unit
): ReadWriteProperty<Any?, T> =
        object : ObservableProperty<T>(initialValue) {
            override fun afterChange(property: KProperty<*>, oldValue: T, newValue: T) = onChange(property, oldValue, newValue)
        }
```

- 초기값과 콜백 람다(onChange)를 받아서 `ObservableProperty` 추상 클래스의 익명 서브클래스를 만들어 반환한다
    - 익명 서브 클래스 만들 때 afterChange()를 오버라이드해서 받은 onChange 콜백 함수를 실행하도록 지정한다

```kotlin
public abstract class ObservableProperty<V>(initialValue: V) : ReadWriteProperty<Any?, V> {
    private var value = initialValue

    protected open fun beforeChange(property: KProperty<*>, oldValue: V, newValue: V): Boolean = true
    protected open fun afterChange(property: KProperty<*>, oldValue: V, newValue: V): Unit {}

    public override fun getValue(thisRef: Any?, property: KProperty<*>): V {
        return value
    }

    public override fun setValue(thisRef: Any?, property: KProperty<*>, value: V) {
        val oldValue = this.value
        if (!beforeChange(property, oldValue, value)) {
            return
        }
        this.value = value
        afterChange(property, oldValue, value)
    }
}
```

- 여기서 핵심은 `ObservableProperty`가 코틀린 위임 기반 인터페이스인 `ReadWriteProperty<Any?, V>`를 구현하고 있다는 점이다
    - 즉, `getValue`, `setValue`를 제공해 위임 관례를 지켜 `by` 키워드로 위임할 수 있는 것
    - `setValue`에서 변경 전 값을 기억하고 변경 여부를 `beforeChange()`로 판단한 뒤 변경이 감지되면 내부 값 value를 갱신한 뒤 `afterChange()`를 호출한다
        - 이 때 우리가 넘긴 콜백 함수 `onChange()`가 호출되는 것이다

실제로 `ReadWriteProperty` 인터페이스를 보면 `getValue`, `setValue`만 `operator fun`으로 가지고 있다

```kotlin
public interface ReadWriteProperty<in T, V> : ReadOnlyProperty<T, V> {
    public override operator fun getValue(thisRef: T, property: KProperty<*>): V
		public operator fun setValue(thisRef: T, property: KProperty<*>, value: V)
}
```

### 위임 프로퍼티는 커스텀 접근자가 있는 감춰진 프로퍼티로 컴파일된다

```kotlin
class C {
    var prop: Type by MyDelegate()
}
```

위임프로퍼티 사용 코드를 컴파일해보면

```kotlin
class C {
    private val <delegate>  = MyDelegate()
    
    var prop: Type
        get() = <delegate>.getValue(this, <property>)
        set(value: Type) = <delegate>.setValue(this, <property>, value)
}
```

- 위임 객체인 `MyDelegate()`는 `<delegate>`라는 이름으로 감춰진 프로퍼티에 저장된다
- 컴파일러는 모든 프로퍼티 접근자 안에 `getValue`와 `setValue` 호출 코드를 생성해준다

### 위임 프로퍼티 활용) 맵에 위임해서 동적으로 키/값 접근

**값을 맵에 저장하는 프로퍼티**

- 속성을 맵에 저장해

```kotlin
class Person {
    private val _attributes = mutableMapOf<String, String>()

    fun setAttribute(key: String, value: String) {
        _attributes[key] = value
    }

    var name: String
        get() = _attributes["name"]!!
        set(value) {
            _attributes["name"] = value
        }
}
```

**위임 프로퍼티로 리팩토링**

```kotlin
class Person {
    private val _attributes = mutableMapOf<String, String>()

    fun setAttribute(key: String, value: String) {
        _attributes[key] = value
    }

    var name: String by _attributes
} 
```

- 가능한 이유는 코틀린 표준 라이브러리가 Map(또는 MutableMap) 인터페이스에 대해 getValue와 setValue 확장 함수를 제공하기 때문
    - getValue에서 맵에 프로퍼티 값을 저장할 때에는 자동으로 프로퍼티 이름을 키로 활용
    - `person.name` 시 `_attributes.getValue(person, prop)`로 대신 호출하고 이건 다시 `_attributes[prop.name]`을 통해 구현된다

### 위임 프로퍼티 실전 활용

**위임 프로퍼티로 DB 칼럼 접근하기**

```kotlin
// 데이터베이스 테이블 (싱글톤)
object Users : IdTable() {
		// name, age 칼럼
    val name: Column<String> = varchar("name", length = 50).index()
    val age: Column<Int> = integer("age")
}

// User 인스턴스는 Users 테이블에 들어있는 구체적인 엔티티
class User(id: EntityID): Entity(id) {
    var name: String by Users.name
    var age: Int by Users.age
}
```

- Entity 클래스는 DB 칼럼을 엔티티의 속성값으로 연결해주는 매핑이 있다
    - 즉 DB에서 가져온 값들을 User 프로퍼티에 매핑
- User 프로퍼티 접근 시 자동으로 Entity 클래스에 정의된 데이터베이스 매핑으로 필요한 값을 가져오고, 수정 시에는 그 객체가 변경된 상태(dirty)로 변하고 프레임 워크가 나중에 적절히 DB에 반영해줘서 편하다
    - ex) `user.age += 1` 을 코틀린에서 사용하면 user에 해당하는 DB 엔티티가 자동 갱신
- Exposed 프레임워크는 Column 클래스 안에 getValue와 setValue 메서드를 정의해 위임 관례를 지킨다
    - Exposed는 젯브레인에서 개발한 SQL 제어용 ORM이라고 하는데 많이 쓰이지는 않는 것 같다