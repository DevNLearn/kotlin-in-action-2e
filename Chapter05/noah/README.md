# 람다를 사용한 프로그래밍
- 람다식과 멤버 참조
- 함수형 인터페이스를 정의하고 자바의 함수형 인터페이스 사용
- 수신 객체 지정 람다

코틀린 표준 라이브러리는 람다를 많이 사용한다.  람다를 사용하면 쉽게 공통 코드 구조를 라이브러리 함수로 뽑아 낼 수 있다.

## 1. 람다식과 멤버 참조

코드에서 일련의 동작을 변수에 저장하거나 다른 함수에 넘겨야 하는 경우가 있다. 자바에서는 `익명 내부 클래스`를 통해 이런 목적을 달성했다. 하지만, `익명 내부 클래스`를 사용하여 함수에 넘기거나 변수에 저장하기 번거롭다.

이러한 문제를 해결하기 위해 함수를 값처럼 다루는 방법이 있으며 이때 `람다식`을 사용하면 코드를 간결하게 사용할 수 있다.

### 익명 내부 클래스 선언 vs 람다식

object 키워드로 `익명 내부 클래스`를 선언을 통해 함수에 인자 넘길 수 있지만, 람다식을 통해 불필요한 코드를 제거하여 간결하게 만들 수 있다.

```kotlin
//object 키워드로 익명 내부 클래스 선언
button.setOnClickListener(object: OnClickListener) { 
    override fun OnClick(v: View) {
        println("I was clicked!")
    }
}

// 람다식
button.setOnClickListener {
    println("I was clicked!")
}
```

## 람다와 컬렉션

코드에서 중복을 제거하는 것은 프로그래밍 스타일을 개선하는 중요한 방법 중 하나이다. 코틀린은 람다를 활용해 컬렉션 처리에 강력하고 편리한 표준 라이브러리를 제공한다.

```kotlin
data class Person(val name: String, val age: Int)
```

코틀린에서 컬렉션에서 최대값을 구하고 싶을 때 `maxByOrNull` 함수를 사용해 검색한다. `maxByOrNull` 함수는 선택자 로직을 인자로 받아 가장 큰 원소를 찾아 주는 함수이다. 아래 코드에서는 선택자 로직을 람다로 전달 한다.

> 람다가 인자를 하나만 받고 그 인자에 구체적인 이름을 붙이고 싶지 않을때 `암시적 이름`인 `it`를 사용한다
>

```kotlin
fun main() {
    val people = listOf(Person("Alice", 29), Person("Bob", 31))
    println(people.maxByOrNull {it.age})
    // Person(name=Bob, age=31}
}
```

람다가 단순히 함수나 프로퍼티에 위임할 경우에는 멤버 참조를 사용할 수 도 있다.

```kotlin
people.maxByOrNull(Person::age)

```

컬렉션에 대해 수행하는 작업은 `람다` 또는 `멤버 참조`를 인자로 사용하는 라이브러리 함수를 통해 간결하게 표현될 수 있다.

## 람다식 문법

코틀린 람다식은 항상 중괄호({})로 둘러 쌓여 있고 화살표가 파라미터와 본문을 구분해준다.

```kotlin
// (파라미터 -> 본문)
(x: Int, y: Int -> x + y}
```

람다식을 변수에 저장할 수 있다.

```kotlin
fun main() {
    val sum = { x: Int, y: Int -> x + y}
    println(sum(1, 2))
}
```

람다식을 직접 호출 할 수도 있다.

```kotlin
fun main() {
    { println(42) }()
}
```

하지만, 람다식을 직접 호출하는 구문은 읽기 어렵고 쓸모 없다. 람다를 만들자마자 바로 호출하느니 본문의 코드를 직접 실행하는 편이 낫다.

코드의 일부분을 블록으로 둘러싸 실행하고 싶다면 `run`을 사용하면 된다. `run`은 인자로 받은 람다를 실행해주는 라이브러리 함수다.

```kotlin
fun main() {
   val myFavoriteNumber = run {
       println("I'm thinking!")
       println("I'm doing some more work...") 
   }
}
```

함수 호출 시 맨 뒤에 있는 인자가 람다식이라면 괄호 밖으로 빼낼 수 있고, 함수의 유일한 인자이고 괄호 뒤에 람다를 썼다면 빈 괄호를 없앨 수 있다.

```kotlin
people.maxByOrNull({p: Person -> p.age})
people.maxByOrNull(){p: Person -> p.age} // 괄호 밖으로 빼낼 수 있음
people.maxByOrNull{p: Person -> p.age} // 유일한 인자라면 빈 괄호를 제거 가능
```

> 마지막 인자만 람다인 경우에는 람다를 밖을 빼내는 스타일을 좋은 스타일로 여긴다. 둘 이상의 람다를 사용하는 받는 경우는 모두 괄호안에 넣는 편이 낫다.
>

람다 파라미터의 타입도 추론이 가능하여 파라미터 타입을 명시 할 필요 없고, 파라미터 이름을 디폴트 이름(`it`)으로 바꿀 수 있다.

```kotlin
people.maxByOrNull{ p -> p.age }
people.maxByOrNull{ it.age } 
```

람다를 변수에 저장할 때는 파라미터의 타입을 추론할 문맥이 존재하지 않아 파라미터 타입을 명시해야 한다.

```kotlin
val getAge = { p: Person -> p.age }
people.maxByOrNull(getAge)
```

### 변수 접근

람다는 함수의 파이널 변수가 아닌 파라미터와 로컬 변수를 참조할 수 있다.

```kotlin
fun printMesssageWithPrefix(message: Collection<String>, prefix: String) {
    message.forEach {
        println("$prefix $it")
    }
}
```

```kotlin
fun printProblemCounts(response: Collection<String>) {
    var clientErrors = 0
    var serverErrors = 0
    response.forEach {
        if (it.startWith("4") {
            clientErros++
        } else if (it.startsWith("5")) {
            serverErrors++
        }
    }
    
    println("$clientErros client errors, $serverErros server errors")
}
```

### 멤버 참조

`멤버 참조`는 메서드를 호출하거나 프로퍼티에 접근하는 함수 값을 만들어주며 `::`을 사용하는 식을 멤버 참조라고 부른다.

멤버 참조는 그 멤버를 호출하는 람다와 같은 타입이다.

```kotlin
people.maxByOrNull(Person::age)
people.maxByOrNull{ person: Person -> person.age }
```

최상위 함수나 프로퍼티를 참조 할 수도 있다.

```kotlin
fun salute() = println("Salute!")
fun main {
    run(::salute)
}
```

다른 함수에게 작업을 위임하는 경우 멤버 참조를 제공하면 파라미터 이름과 함수를 반복하지 않아도 되서 편리하다.

```kotlin
val action = { 
    person: Person, 
    message: String -> sendEmail(person, message) // sendEmail 함수에게 작엄을 위임
}
val nextAction = ::sendEmail
```

생성자 참조를 사용하면 클래스 생성 작업을 연기하거나 저장해 둘 수 있다.

```kotlin
data class Person(val name: String, val age: Int)
fun main() {
    val createPerson = ::Person
    val p = createPerson("Alice", 29)
    println(p)
    // Person(name=Alice, age=29)
}
```

### 값과 엮인 호출 가능 참조

`값과 엮인 호출 가능 참조`를 사용하면 해당 인스턴스의 메서드를 참조할 수 있다.

```kotlin
fun main() {
    val seb = Person("Sebastian", 26)
    val personsAgeFunction = Person::age
    print(personsAgeFunction(seb))
    val sebsAgeFunction = seb::age
    println(sebsAgeFunction())
}
```

## 2. 자바의 함수형 인터페이스 사용: 단일 추상 메서드

코틀린 람다는 자바 API 호환된다. 자바에서 제공하는 함수형 인터페이스에 람다를 사용할 수 있다.

> `인터페이스` 안에 추상 메서드가 하나 인터페이스를 `함수형 인터페이스`나 `단일 추상 메서드(SAM, Single Abstract Method) 인터페이스` 라고 부른다.
>

```kotlin
// 자바
public interface OnClickListener {
    void onClick(View v);
}

public class Button {
    public void setOnClickListener(OnclickListener onclickListener);
}
```

```kotlin
// 자바 8 이전
button.setOnClickListener(new OnclickListener() {
     @Override
     public void onClick(View v) {
         /*...*/
     }
})

// 자바 8 이후
button.setOnclickListener(view -> { /*..*/ })

// 코틀린
button.setOnclickListener { view -> /*..*/ }
```

코틀린은 함수형 인터페이스를 파라미터로 받는 자바 메서드를 호출할 때 람다를 사용할 수 있게 해준다.

### 람다를 자바 메서드의 파라미터로 전달

함수형 인터페이스를 파라미터로 받는 모든 자바 메서드에 람다를 전달할 수 있다.

```kotlin
// 자바 코드
void postponeComputation(int delpy, Runnable computation);

// 코틀린 람다
postponeComputation(1000) { println(42) }

// 코틀린 객체 식 (명시적 선언) 
postponeComputation(1000, object: Runnable {
    ovveride fun run() {
        println(42)
    }
})
```

명시적으로 객체를 선언하면 매번 호출할 때마다 새로운 익명 클래스 인스턴스가 생성된다. 람다를 사용할 때 함수 본문 안에서 정의된 람다가 캡처하지 않는 경우 람다에 해당하는 익명 객체가 재사용된다.

### SAM 변환: 람다를 함수형 인터페이스로 명시적 변환

SAM 생성자는 컴파일러가 생성한 함수로 람다를 단일 추상 메서드 인터페이스의 인스턴스로 명시적으로 변환해준다.

함수형 인터페이스의 인스턴스를 반환하는 함수가 있따면 람다를 직접 반환할 수 없어 람다를 SAM 생성자로 감싸야한다.

```kotlin
fun createAllDoneRunnable(): Runnable {
    return Runnable { println("All done!")}
}

fun main() {
    createAllDoneRunnable().run()
}
```

SAM 생성자의 이름은 사용하려는 함수형 인터페이스의 이름과 같게 사용한다.

값을 반환할 때 외에 람다로 생성한 함수형 인터페이스 인스턴스를 변수에 저장해야 하는 경우에도 SAM 생성자를 사용할 수 있다.

```kotlin
val listenner = OnClickListener { view ->  // 람다를 사용해 SAM 생성자 호출
    val text = when (view.id) {
        button1.id -> "First button"
        button2.id -> "Second button"
        else -> "Unknown button"
    }
    toast(text)
}
button1.setOnClickListener(listener)
button2.setOnClickListener(listener)
```

> 람다에는 익명 객체와 달리 인스턴스 자신을 가리키는 this가 없다.
>

### 코틀린에서 SAM 인터페이스 정의: fun interface

코틀린에서 함수형 인터페이스를 사용해야 할 때 함수 타입을 사용해 행동을 표현할 수 있다. 명시적으로 함수 타입을 적고 싶을 때 `fun interface` 를 정의한다.

코틀린의 함수형 인터페이스는 하나의 추상 메서드와 다른 비추상 메서드를 여러개 가질 수 있다.

```kotlin
fun interface IntCondition {
    fun check(i: Int): Boolean
    fun checkString(s: String) = chec(s.toInt())
    fun checkChar(c: Char) = check(c.digitToInt())
}

fun main() {
    val isOdd = IntCondition { it % 2 != 0 }
    println(isOdd.check(1))  //true
    println(isOdd.checkString("2")) //false
    println(isOdd.checkChar('3')) //false
}
```

`fun interface`라고 정의된 타입의 파라미터를 받는 함수가 있을 때 람다 구현이나 람다에 대한 참조를 직접 넘길 수 있다.

```kotlin
fun checkCondition(i: Int, condition: IntCondition): Boolean {
    return condition.check(i)
}

fun main() {
    checkCondition(1) { it % 2 != 0 } //람다를 직접 사용
    val isOdd: (Int) -> Boolean = { it % 2 != 0 }
    checkCondition(1, isOdd) //람다에 대한 참조를 사용 
}
```

## 3. 수신 객체 지정 람다: with, apply, also

수신 객체를 명시하지 않고 람다의 본문 안에서 다른 객체의 메서드를 호출하는 것을  `수신 객체 지정 람다`라고 한다.

### with 함수

with는 객체의 이름을 반복하지 않고 대상 객체에 대해 다양한 연산을 수행하는 기능을 제공한다.

```kotlin
// with문을 사용하지 않는 경우
fun alphabet(): String {
    val result = StringBuilder()
    for (letter in `A`..`Z`) {
        result.aappen(letter)
    }
    result.appen("\nNow I know the alphabet!")
    return result.toString()
}

// with문을 사용하는 경우
fun alphabet(): String {
    val stringBuilder = StringBuilder()
    return with(stringBuilder) {
        for (letter in 'A'..'Z') {
            this.append(letter)
        }
        this.append("\nNow I know the alphabet!")
        this.toString()
    }
}

fun main() {
    println(alphabet())
}
```

`this`는 생략 가능하고 with와 식을 본문으로 하는 함수를 활용할 수도 있다.

```kotlin
fun alphabet() = with(StringBuilder()) {
    for (letter in 'A'..'Z') {
        append(letter)
    }
    append("\nNow I know the alphabet!")
    toString()
}
```

with에 인자로 넘긴 객체의 클래스와 with를 사용하는 코드가 들어있는 클래스 안에 메서드 이름이 충돌나면 with 내부의 메서드가 사용된다. 밖에 클래스의 메서드를 사용하고 싶으면 레이블을 지정하여 명확하게 사용할 수 있다.

```kotlin
class Outer {
    override fun toString(): String = "Outer Class"

    fun buildAlphabet(): String {
        return with(StringBuilder()) {
            append("Alphabet: ")
            for (c in 'A'..'C') {
                append(c)
            }
            append(" / outer toString: ")
            this@Outer.toString() // Outer 클래스의 toString 명시
        }
    }
}
```

### apply 함수

`apply`함수는 `with`함수와 비슷하지만 수신 객체를 반환한다는 점에서 다르다.

```kotlin
fun alphabet() = StringBuilder().apply {
    for (letter in 'A'..'Z') {
        append(letter)
    }
    append("\nNow I know the alphabet!")
}.toString()
```

`apply` 함수는 인스턴스를 만들어주면서 즉시 프로퍼티를 초기화해야 하는 경우 유용하게 사용된다.

```kotlin
fun createViewWithCustomAttributes(context: Context) = 
    TextView(context).apply {
        text = "Sample Text"
        textSize = 20.0
        setPadding(10, 0, 0, 0)
    }
```

> 코트린은 buildString, buildList, buildSet, buildMap 같은 유용한 빌더 함수를 제공한다.
>

### 객체에 추가 작업 수행: also

`also` 함수도 `apply` 와 마찬가지로 수신 객체를 받으며, 수신 객체에 대해 어떤 동작을 수행한 후 수신 객체를 돌려준다. 주된 차이는 also의 람다 안에서는 수신 객체를 인자로 참조하여 파라미터 이름을 부여하거나 디폴트 이름인 `it`을 사용해야 한다.

```kotlin
fun main() {
    val fruits = listOf("Apple", "Banana", "Cherry")
    val uppercaseFruites = mutableListOf<String>()
    val reversedLongFruits = fruits
        .map { it.uppercase() }
        .also { uppercaseFruits.addAll(it) }
        .filter { it.length > 5 }
        .also { println(it) }
        .reversed()
}
```

## 4. 요약

- 람다를 사용하면 코드 조각을 다른 함수에게 인자로 넘길 수 있다.
- 람다를 사용하면 공통 코드 구조를 쉽게 추출할 수 있다.
- 코틀린에서 람다가 함수 인자인 경우 괄호 밖으로 빼서 더 깔끔하고 간결하게 만들 수 있다.
- 람다의 인자가 하나뿐인 경우 디폴트 이름(`it`)을 사용할 수 있어 짧고 간단한 하게 사용할 수 있다.
- 람다는 외부 변수를 캡처할 수 있다.
- 람다 안에 있는 코드가 바깥 함수의 변수를 읽어가 쓸 수 있다.
- filter, map, all, any 등의 표준 라이브러리로 이터레이션하지 않고 컬렉션에 대한 연산을 수행 할 수 있다.
- 추상 메서드가 하나뿐인 인터페이스(SAM)를 구현하기 위해 람다를 직접 사용할 수 있다.
- 수신 객체 지정 람다를 사용하면 람다 안에서 정해둔 수신 객체의 메서드를 직접 호출할 수 있다.
- 표준 라이브러리 with, apply, also 등 유용한 함수를 제공한다.