## 2장

### 코틀린 함수와 변수
- `fun` 으로 함수를 선언
- 세미콜론이 필요하지 않음
```kotlin
    fun print(){
        println("Hello, world!")
    }
```

- 파라미터로 넘어오는 변수는 변수명 다음에 데이터타입이 들어온다
- 함수의 반환값은 함수와 콜론으로 구분한다
```kotlin
    fun max(a: Int, b: Int): Int{
    return if (a > b) a else b
}
```
- 조금 더 간결하게 표현할 수도 있는데, `식 본문 함수` 라고 부른다
- 이때는 반환 데이터타입 및 `return`도 생략 가능하다
```kotlin
    fun max(a: Int, b: Int) =
        if (a > b) a else b
```
- 변수 선언시 `val`, `var` 두가지가 존재한다. (java에서 final `l` 로 val 은 불변하다고 외웠다.)
- 자동으로 타입을 추론하기때문에 굳이 안써줘도 괜찮지만 복잡한 로직에서는 써주는편이긴하다.
---

### **'문'과 '식'**
- 처음 코틀린을 사용했을때 가장 편하다고 느꼈던 부분
- `식`은 값을 만들어내고, `문`은 코드블록으로 사용되며 값을 만들어내지 않는다.
- 코틀린은 루프(`for`, `while`, `do/while`)를 제외한 대부분의 제어 구조가 `식`
---
- 자바에서 다음과 같은 코드는
```java
    boolean foo(int num){
        boolean res;
        if(num % 2 == 0){
            res = true;
        }
        else {
            res = false;
        }
        return res;
    }
```
- 코틀린으로는 다음과 같다.
```kotlin
    fun foo(num: Int): Boolean {
        return if(num % 2 == 0){
            true
        } else{
            false
        }
    }
```
---

### 문자열 템플릿
- 코틀린에서는 아주 간단하게 문자열 템플릿 기능을 제공한다.
- 자바에서 다음과 같이 사용해야했다면
```java
    void print(String name){
        System.out.println(name + "님 반갑습니다.");
        System.out.println(String.format("%s님 반갑습니다.", name));
    }
```
- 코틀린에서는 다음과 같이 사용한다. 심지어 자동완성도 된다
```kotlin
    fun print(name: String){
        println("${name}님 반갑습니다.")
    }
```

---

### 클래스와 프로퍼티
- 코틀린에서는 단순히 다음과 같이 선언하더라도 getter, setter를 자동으로 만들어준다
```kotlin
class Person(
        val name: String,
        var isStudent: Boolean
)

fun personMethod(){
    // new 없이 생성
    val person = Person("babi", true)
    // val 은 변경 불가능
    person.name = "babi2"
    // var 는 변경 가능
    person.isStudent = false
    // "babi"
    println(person.name)
    // false
    println(person.isStudent)
}
```

---

### 코틀린의 enum
- 유일하게 세미콜론이 들어가야한다.
- 우리가 쓰는 java 와 비교하면 다음과 같고, 아주 간결하다.
```java
enum Color{
    RED(255, 0, 0),
    GREEN(0, 255, 0),
    BLUE(0, 0, 255);
    
    private final int r;
    private final int g;
    private final int b;
    
    Colors(int r, int g, int b){
        this.r = r;
        this.g = g;
        this.b = b;
    }
}
```

---

```kotlin
enum class Color(
        val r: Int,
        val g: Int,
        val b: Int
) {
    RED(255,0,0),
    GREEN(0,255,0),
    BLUE(0, 0, 255);
}
```

---

### when 과 is
- 코틀린의 `when` 과 가장 유사한걸로 자바의 `switch`가 있겠다.
- 최근 JDK 버전이 올라가면서 꽤나 쓸만해지긴했지만 비교를 위해 구버전 `switch`와 비교를 하겠다.
```java
    void switchTest(int num){
    switch (num){
        case -1 -> System.out.println("음수");
        case 0 -> System.out.println("0");
        case 1 -> System.out.println("양수");
        default -> System.out.println("에러");
    }
}
```

---
- case 가 사라졌다
- default 가 else 로 바뀌었다.
```kotlin
    fun switchTest(num: Int){
    when (num){
        -1 -> println("음수")
        0 -> println("0")
        1 -> println("양수")
        else -> print("에러")
    }
}
```

- 사실 이걸로는 크게 바뀐걸 모르지만 코틀린은 `식` 이며, `is` 와 사용할때 꽤나 유용하게 사용했다.
- 진짜 기가막힌다
```kotlin
    // 메서드 리턴
    fun switchTest(num: Int): String{
        return when (num){
            -1 -> "음수"
            0 -> "0"
            1 -> "양수"
            else -> "에러"
        }
    }

    // 변수에 저장
    fun switchTest(num: Int){
        val str = when (num){
            -1 -> "음수"
            0 -> "0"
            1 -> "양수"
            else -> "에러"
        }
        println(str)
    }

```

---

- 코틀린의 `is` 와 유사하다고 생각하는건 자바의 `instanceof`가 있겠다.
- 그런데 스마트 캐스트가 된다는게 자바와 큰 차이점인것 같다.
- 자바같은경우, 데이터타입을 맞춰주고도 다시한번 더 형변환이 필요하다
```java
    void dataTypeTest(Node node){
        if(node instanceof File){
            // 똑같은 형변환
            File file = (File) node;
            System.out.println(file.getFileName());
        }
    }
```
- 하지만 코틀린은에서는 한번 데이터타입 검사를 했다면 형변환이 필요없다.
```kotlin
    fun dataTypeTest(node: Node){
    when (node){
        is File -> println(node.fileName)
        is Directory -> println(node.path)
    }
}
```

---

### 반복문
- for 과 while 에서 약간씩 다른것들이 존재한다.
- 먼저 while 에서는 break 를 지정해줄수 있다.- 특정 조건이 될때, `outer@` 와 같이 `레이블`을 지정하여 레이블을 참조할 수 있다.

```kotlin
outer@ while (outerCondition){
    while (innerCondition){
        if(shouldExitInner) break
        if(shouldExit) break@outer
    }
}
```
- 코틀린에서는 변수를 초기화하고, 실행할 때마다 변수를 갱신하고, 어떤 값에 도달하면 중단하는 루프가 없다 (`for(int i = 0; i < 5; i++)`)
- 대신 코틀린은 `..` 연산자를 쓰는 `범위` 를 사용한다.
- 자바에서 사용하는 이터레이션은 그대로 사용된다
```kotlin
    fun forMethod(){
        val collection = listOf("red", "green", "blue")
        for(color in collection){
            println("${color}")
        }
    }
```

---

### in
- 값이 범위에 속하는지 확인할 수 있다.
- 신기한건 다음과 같이 `Char` 형 말고, 비교가 가능한 클래스는 모두 가능하다. (예를들면 String 형 끼리도 비교 가능)
```kotlin
fun recognize(c: Char): String {
    return when (c){
        in '0' .. '9' -> "Digit"
        in 'a' .. 'z', in 'A' .. 'Z' -> "Letter"
        else -> "Error"
    }
}

// "Java" <= "Kotlin" && "Kotlin" <= "Scala" 와 동일한 의미
println("Kotlin" in "Java" .. "Scala")
```
- 컬렉션에서도 존재하는지 여부로 사용할 수 있다. 자바의 `contains()` 메서드처럼
```kotlin
// 값으로 존재하지 않음. false
println("Kotlin" in setOf("Java", "Scala"))
```

---

### try
- try 도 `식`이라서 결과값을 그대로 받아올 수 있다
```kotlin
fun tryMethod(node: Node){
    val res = try {
        node.filePath
    } catch (e: IOException){
        null
    }
} 
```