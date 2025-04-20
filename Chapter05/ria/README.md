# 람다
> **람다** : 다른 함수에 넘길 수 있는 작은 코드 조각  
> **함수형 프로그래밍**
>   - 일급 시민인 함수 : 함수를 값으로 다룰 수 있음. 함수를 변수에 저장, 파라미터로 전달, 함수에서 다른 함수 리턴
>   - 불변성 : 객체를 만들 때, 일단 만들어진 다음에는 내부 상태가 변하지 않음을 보장하는 방법으로 설계 가능
>   - 부수 효과 없음 : 함수가 같은 입력에 대해 항상 같은 출력을 내놓음 (파라미터에 의해서만 영향)
## 람다식과 멤버 참조
### 코드 블록을 값으로 다루기
- 예시1: 이벤트 발생 시 이 핸들러 실행
- 예시2: 데이터 구조의 모든 원소에 이 연산 적용

Java에서는 익명 내부 클래스를 통해 해결했지만, 번거로움  
```java
    import java.util.*;

    public class AnonymousInnerClassExample {
        public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
        numbers.forEach(new java.util.function.Consumer<Integer>() {
            @Override
            public void accept(Integer number) {
                System.out.println(number * 2); 
            }
        });
        }
    }
```
함수를 값처럼 다루면 ? 클래스를 선언, 클래스의 인스턴스를 함수에 넘기는 대신 **함수를 직접 다른 함수에 전달**
```kotlin
// 클릭을 처리하는 OnClickListener 인터페이스에 상응하는 인스턴스를 전달받고 싶은 상황
// OnClickListener에는 onClick이라는 하나의 메소드만 있음
button.setOnClickListener(object: OnClickListener {
    override fun onClick(v: View) {
        println("Clicked") 
    }
})
```
이 코드를 람다식으로 변경하면
```kotlin
button.setOnClickListener {
    println("Clicked")
}
```
메소드가 하나뿐인 익명 객체를 대신 사용할 수 있음!

### 람다와 컬렉션
사람의 이름과 나이를 저장하는 데이터 클래스
```kotlin
data class Person(val name: String, val age: Int)
```
나이가 가장 많은 사람을 찾으려면 ?
1. for 루프로 직접 찾기
```kotlin
fun findTheOldest(people: List<Person>) {
    var maxAge = 0;
    var theOldest = Person? = null
    for (person in people) {
        if (person.age > maxAge) {
            maxAge = person.age
            theOldest = person
        }
    }
    println(theOldest)
}

fun main() {
    val people = listOf(Person("Alice", 29), Person("Bob", 31))
    findTheOldest(people) //Person(name=Bob, age=31)
}
```
2. `maxByOrNull` 함수로 컬렉션 검색
```kotlin
fun main() {
    val people = listOf(Person("Alice", 29), Person("Bob", 31))
    println(people.maxByOrNull {it.age}) //Person(name=Bob, age=31)
}
```
3. 람다 쓰기 (함수나 프로퍼티에 위임할 땐 멤버 참조 사용 가능)
```kotlin
people.maxByOrNull(Person::age)
```
### 람다식의 문법
람다는 항상 `{중괄호}`로 싸여 있으며, 파라미터를 지정하고 실제 로직이 담긴 본문 제공
```kotlin
{ x : Int, y : Int -> x + y } // -> 좌항 : 파라미터, -> 우항 : 본문
```
```kotlin
fun main() {
    val sum = {x: Int, y: Int -> x + y}
    println(sum(1, 2)) // 3
}
```
또는
```kotlin
fun main() {
    {println(42)}() // 42
}
```
`run` 을 사용해서 인자로 받은 람다를 실행할 수 있음
```kotlin
fun main() {
    run {println(42)} // 42
}
```
**최상위 수준의 변수**를 정의할 때 몇 가지 설정과 다른 추가 작업이 필요할 때
```kotlin
val myFavoriteNum = run {
    println("mmmm")
    println("hmmmmm")
    42
}
```
리스트에서 제일 나이 많은 사람 찾는 예제에서 원래 **정식 람다**로 쓰면
```kotlin
people.maxByOrNull({p: Person -> p.age})
```
함수 호출 시, 맨 뒤의 인자가 람다식이면 **람다를 괄호 밖으로** 빼낼 수 있음, 그리고 빈 괄호도 없애도 됨
```kotlin
people.maxByOrNull(){p: Person -> p.age} // 람다를 괄호 밖으로
people.maxByOrNull{p: Person -> p.age} // 빈괄호 생략
```
이름을 붙인 인자를 이용해 람다 넘기기
```kotlin
fun main() {
    val people = listOf(Person("Alice", 29), Person("Bob", 31))
    val names = people.joinToString(
        separator = " ",
        transform = {p: Person -> p.name}
    )
    println(names) // Alice Bob
}
```
람다를 괄호 밖으로 빼면
```kotlin
people.joinToString(" ") {p: Person -> p.name}
// people.joinToString(separator) {transform}
```
다시 아까 `maxByOrNull` 예제를 다듬으면  
Person 타입의 객체가 들어있는 컬렉션에 대해서 호출하기 때문에 파라미터도 Person이라는 걸 알 수 있기 때문에 생략 가능
```kotlin
people.maxByOrNull{p -> p.age} // 타입추론
```
디폴트 파라미터 이름 it 사용
```kotlin
people.maxByOrNull{ it.age } // 유일한 파라미터니까 it
```
람다를 변수에 저장할 땐 파라미터 타입을 추론할 수 있는 문맥이 없으니까 타입 추론이 불가능함
```kotlin
val getAge = {p: Person -> p.age} // 타입 명시
people.maxByOrNull(getAge)
```
본문이 여러줄인 람다는 마지막 식이 람다의 결과
```kotlin
fun main() {
    val sum = {x: Int, y: Int -> 
        println("sum of $x and $y")
        x+y
    }
    println(sum(1, 2)) 
    // sum of 1 and 2
    // 3
}
```
### 변수 캡쳐 (현재 영역에 있는 변수 접근)
함수 안에 익명 내부 클래스 선언하면 그 클래스 안에서 함수의 파라미터와 로컬변수에 참조할 수 있는 것처럼  
람다에서도 동일하게 할 수 있음
```kotlin
fun printMessageWithPrefix(messages: Collection<String>, prefix: String) {
    messages.forEach {
        println("$prefix $it")
    }
}

fun main() {
    val errors = listOf("403 Forbidden", "404 Not Found")
    printMessageWithPrefix(errors, "Error: ")
}

// Error: 403 Forbidden
// Error: 404 Not Found
```
> 코틀린 람다 vs 자바 람다 : 코틀린 람다 안에서 final 변수가 아닌 변수에 접근할 수 있음. 람다 안에서 바깥 변수를 변경해도 됨
```kotlin
fun printProblemCounts(responses: Collection<String>) {
    var clientErrors = 0 // 람다가 캡쳐한 변수 
    var serverErrors = 0 // 람다가 캡쳐한 변수
    responses.forEach {
        if (it.startsWith("4")) {
            clientErrors++
        } else if (it.startsWith("5")) {
            serverErrors++
        }
    }
    println("$clientErrors client errors, $serverErrors server errors")
}

fun main() {
    val responses = listOf("200 OK", "418 I'm a teapot", "500 Internal Server Error")
    printProblemCounts(responses)
    // 1 client errors, 1 server errors
}
```
final이 아닌 변수를 캡쳐한 경우, **변수를 특별한 래퍼로 감싸** 나중에 변경하거나 읽을 수 있게한 다음, 래퍼에 대한 참조를 람다 코드와 같이 저장
```kotlin
// 변경 가능한 값을 저장할 원소가 단 하나 뿐인 배열을 선언하거나, 변경 가능한 변수에 대한 참조를 저장하는 클래스 선언
class Ref<T>(var value: T) // 변경 가능한 변수를 캡쳐하는 방법 보여주는 클래스

// 이렇게 동작하는데
fun main() {
    val counter = Ref(0) 
    val inc = { counter.value++ } // 공식적으론 변경 불가능한 변수를 캡쳐, 변수가 가리키는 객체의 필드 값을 바꿀 수 있음
}

// 실제 코드에서는 저런 래퍼를 안만들고, 변수를 직접 바꿈
fun main() {
    var counter = 0
    val inc = { counter++ }
}
```
람다를 **이벤트 핸들러나 다른 비동기적으로 실행되는 코드로 활용**하는 경우, 로컬 변수 변경은 **람다가 실행될 때만** 일어남
```kotlin
fun tryToCountButtonClicks(button: Button): Int {
    var clicks = 0
    button.onClick { click++ } 
    return clicks
}
// 0
```
`onClick` 핸들러는 호출될 때마다 값을 증가시키긴 하지만 값 변경을 관찰할 순 없음  
핸들러는 **clicks가 반환된 이후에 호출**되기 때문  
클릭 횟수를 세는 카운터 변수를 밖으로 옮겨야 함  
```kotlin
fun tryToCountButtonClicks(button: Button) {
    var clicks = 0
    button.onClick {
        clicks++
        println("Clicked $clicks times")
    }
}
```

### 멤버 참조
메소드를 호출하거나 한 프로퍼티에 접근하는 함수 값을 만들어줌
클래스 이름과 참조하려는 멤버의 사이에 `::` 위치
```kotlin
val getAge = Person::age
people.maxByOrNull(Person::age) // 멤버참조
```
최상위에 선언된 함수나 프로퍼티 참조도 가능  
클래스 이름 생략하고 ::으로 바로 참조 시작  
run에 넘기고, run이 람다를 호출
```kotlin
fun salute() = println("salute")
fun main {
    run(::salute)
}
```
람다가 인자가 여럿인 다른 함수에 작업 위임 시, 멤버 참조 제공하면 편리
```kotlin
val action = {person: Person, message: String ->
    sendEmail(person, message)
}
val nextAction = ::sendEmail //람다 대신 멤버 참조 쓸 수 있음
```
**생성자 참조**를 사용하면 클래스 생성 작업을 연기하거나 저장해둘 수 있음
```kotlin
data class Person(val name: String, val age: Int)
fun main() {
    val createPerson = ::Person
    val p = createPerson("Alice", 29)
    println(p)
    // Person(name=Alice, age=29)
}
```
**확장 함수**도 같은 방식으로 참조 가능
```kotlin
fun Person.isAdult() = age >= 21
val predicate = Person::isAdult
```
### 값과 엮인 호출 가능 참조
같은 멤버 참조 구문을 사용해 특정 객체 인스턴스에 대한 메소드 호출에 대한 참조를 만들 수 있음
```kotlin
fun main() {
    val seb = Person("Sebastian", 25)
    val personsAgeFunc = Person::age // 사람이 주어지면 나이 반환
    println(personsAgeFunc(seb))
    // 25
    val sebsAgeFunc = seb::age // 값과 엮인 호출 가능 참조
    println(sebsAgeFunc()) // 아무 파라미터를 지정하지 않아도 됨
    // 25
}
```
## 자바의 함수형 인터페이스 사용 : 단일 추상 메소드
```kotlin
button.setOnClickListener {
    println("Clicked")
}
```
```java
public class Button {
    public void setOnClickListener(OnClickListener l) {}
}
// OnClickListener 인터페이스에는 메소드가 하나만 있음
public interface OnClickListener {
    void onClick(View v);
}
//8 이전
button.setOnClickListener(new OnClickListener(){
    @Override
    public void onClick(View v) {
        
    }
});
// 8 부터
button.setOnClickListener(view -> {});
```
```kotlin
button.setOnClickListener{view -> println()}
```
OnClickListener가 단일 추상 메소드 인터페이스여서 이런 코드가 가능함  
Runnable을 인자로 받을 때
```java
// java 코드
void postponeComputation(int delay, Runnable computation);
```
코틀린에서는 람다를 Runnable 인스턴스로 변환해줌 (Runnable을 구현하는 익명 클래스의 인스턴스)
```kotlin
// kotlin 코드
postponeComputation(1000){println(42)}
```
Runnable을 명시적으로 구현하는 익명 객체로도 같은 효과
```kotlin
postponeComputation(1000, object : Runnable {
    override fun run() {
        println(42)
    }
})
```
근데 명시적으로 객체 선언하면 매번 호출할 때마다 새 인스턴스가 생김  
근데 람다는 자신이 정의한 함수의 변수에 접근하지 않으면 함수가 호출될 때마다 람다에 해당하는 익명 객체가 재사용됨
```kotlin
postponeComputation(1000){println(42)} // 전체 프로그램에 Runnable 인스턴스 하나만 생성
```
람다가 자신을 둘러싼 환경의 변수를 캡처하면 더 이상 각 함수 호출에 같은 인스턴스 재사용 불가능
```kotlin
fun handleComputation(id: String) { // 이 함수 호출될 때마다 Runnable 인스턴스 생성
    postponeComputation(1000) {
        println(id) // id 캡쳐
    }
}
```
### SAM 변환 (함수형 인터페이스로 명시적 변환)
SAM 생성자의 이름은 사용하려는 함수형 인터페이스의 이름과 같음  
하나의 인자만 받아서 함수형 인터페이스를 구현하는 클래스의 인스턴스를 반환함
```kotlin
fun createAllDoneRunnable(): Runnable {
    return Runnable {println("All done")}
}
fun main() {
    createAllDoneRunnable().run()
    // All done
}
```
람다로 생성한 함수형 인터페이스 인스턴스를 변수에 저장해야 하는 경우에도 SAM 생성자를 사용할 수 있음  
```kotlin
val listener = OnClickListener {view ->
    val text = when (view.id) {
        button1.id -> "First"
        button2.id -> "Second"
        else -> "Unknown"
    }
    toast(text)
}
button1.setOnClickListener(listener)
button2.setOnClickListener(listener)
```
## 코틀린에서 SAM 인터페이스 정의 : fun interface
정확히 하나의 추상 메소드만 포함하지만 다른 비추상 메소드를 여러개 가질 수 있음  
함수 타입의 시그니처에 맞지 않는 여러 복잡한 구조를 표현할 수 있음
```kotlin
fun interface IntCondition {
    fun check(i: Int): Boolean // 추상 메소드는 하나
    fun checkString(s: String) = check(s.toInt())
    fun checkChar(c: Char) = check(c.digitToInt())
}

fun main() {
    val isOdd = IntCondition { it % 2 != 0 }
    println(isOdd.check(1))
    // true
    println(isOdd.checkString("2"))
    // false
    println(isOdd.checkChar('3'))
    // true
}
```
함수형 인터페이스는 동적으로 인스턴스화 됨
```kotlin
fun checkCondition(i: Int, condition: IntCondition): Boolean {
    return condition.check(i)
}
fun main() {
    checkCondition(1) {it % 2 != 0}
    // true
    val isOdd: (Int) -> Boolean = { it % 2 != 0}
    checkCondition(1, isOdd) // 시그니처에 일치하는 람다에 대한 참조 이용
}
```
## 수신 객체 지정 람다 : with, apply, also
### with
```kotlin
fun alphabet(): String {
    val result = StringBuilder()
    for (letter in 'A' ..'Z') {
        result.append(letter)
    }
    result.append("\nNow I know the alphabet")
    return result.toString()
}
fun main() {
    println(alphabet())
    //ABCDEFGHIJKLMNOPQRSTUVWXYZ
    //Now I know the alphabet
}
```
with를 쓰면  
첫번째 인자로 받은 객체를 두번째 인자로 받은 람다의 수신 객체로 만듦  
명시적인 this 참조를 사용해 그 수신 객체에 접근할 수 있음  
this.를 없애고 메소드나 프로퍼티 이름만 사용해 접근도 가능
```kotlin
fun alphabet(): String {
    val stringBuilder = StringBuilder()
    return with(stringBuilder) {
        for (letter in 'A'..'Z') {
            this.append(letter)
        }
        this.append("\nNow I know the alphabet")
        this.toString()
    }
}

// this. 제거
fun alphabet(): String { 
    val stringBuilder = StringBuilder()
    run with(stringBuilder) {
        for (letter in 'A'..'Z') {
        append(letter)
    }
        append("\nNow I know the alphabet")
        toString()
    }
}

// with와 식을 본문으로 하는 함수
fun alphabet(): with(StringBuilder()) {
    for (letter in 'A'..'Z') {
        append(letter)
    }
    append("\nNow I know the alphabet")
    toString()
}
```
with에 인자로 넘긴 객체의 클래스와 with를 사용하는 코드가 들어있는 클래스 안에 이름이 같은 메소드가 있으면?  
this 참조 앞에 레이블을 붙이면 호출하고 싶은 메소드를 명확하게 정할 수 있음  
```kotlin
this@OuterClass.toString()
```

### apply 함수
항상 자신에 전달된 수신 객체를 반환함
```kotlin
fun alphabet(): StringBuilder().apply {
    for (letter in 'A'..'Z') {
        append(letter)
    }
    append("\nNow I know the alphabet")
}.toString()
```
buildString
```kotlin
fun alphabet() = buildString {
    for (letter in 'A'..'Z') {
        append(letter)
    }
    append("\nNow I know the alphabet")
}
```
컬렉션 빌더 함수 : buildList, buildSet, buildMap
```kotlin
val fibonacci = buildList {
    addAll(listOf(1, 1, 2))
    add(3)
    add(index = 0, element = 3)
}
val shouldAdd = true
val fruits = buildSet {
    add("Apple")
    if (shouldAdd) {
        addAll(listOf("Apple", "Banana", "Cherry"))
    }
}
val medals = buildMap<String, Int> {
    put("Gold", 1)
    putAll(listOf("Silver" to 2, "Bronze" to 3))
}
```
### also
수신 객체에 대해 어떤 동작을 수행한 후 수신 객체를 돌려줌  
also의 람다 내에서는 수신 객체를 인자로 참조해야 한다는 점이 다름  
파라미터 이름을 부여하거나 디폴트 이름인 it을 사용해야 함  
원래 수신 객체를 인자로 받는 동작을 할 때 유용
```kotlin
fun main() {
    val fruits = listOf("Apple", "Banana", "Cherry")
    val uppercaseFruits = mutableListOf<String>()
    val reversedLongFruits = fruits
        .map { it.uppercase() }
        .also { uppercaseFruits.addAll(it) }
        .filter { it.length > 5 }
        .also { println(it) }
        .reversed()
    // [BANANA, CHERRY]
    println(uppercaseFruits)
    // [APPLE, BANANA, CHERRY]
    println(reversedLongFruits)
    // [CHERRY, BANANA]
}
```
