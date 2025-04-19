
## 람다

### 람다란?
- 다른 함수에 넘길 수 있는 작은 코드 조각을 의미하는 람다 표현식
- 어떤 동작을 변수에 저장하거나 인자로 전달하고 싶을 때 예전 자바에서는 익명 내부 클래스를 사용했다.
  - 상당히 번거롭다
    ```java
    Runnable runnable = new Runnable() {
        @Override
        public void run() {
            System.out.println("Hello");
        }
    };
    ```
- 함수를 값처럼 다루는 람다식을 사용하면 클래스의 인스턴스를 넘기는 방법 대신 직접 함수를 넘겨줄 수 있다
    ```java
    Runnable runnable = () -> System.out.println("Hello");
    ```

**함수형 프로그래밍 정리**
1. 일급 시민 함수
   - 함수를 값으로 다룰 수 있어 변수에 저장하거나 인자로 전달하거나 반환할 수 있다
2. 불변성
   - 객체 생성 이후 내부 상태가 변하지 않는 불변 객체로 설계할 수 있다.
3. 순수 함수 (사이드 이펙트 X)
   - 같은 입력에 대해 항상 같은 출력을 반환하는 참조 투명성을 가지기에 값으로 다룰 수 있는 것.
   - 순수하다고 표현하며 부수 효과가 없다 (동작이 예측이 가능하다)
   - 외부로부터 영향을 받지 않음과 동시에 내부에서 외부에도 영향을 주지 않는다
    ```kotlin
    fun main() {
        val numbers = listOf(1, 2, 3, 4, 5)
        val factor = 2
    
        // 람다
        fun multiply(list: List<Int>, factor: Int) = list.map { it * factor }
    
        val result = multiply(numbers, factor)
    
        // 원본은 변하지 않음(불변성) + 항상 같은 결과를 반환(순수함수)
        println("Original numbers: $numbers") // [1, 2, 3, 4, 5]
        
        // 새로운 객체를 반환
        println("Multiplied numbers: $result") // [2, 4, 6, 8, 10]
    }

**람다 리팩토링 예제**
```kotlin
// 클릭을 처리하는 OnClickListener 인터페이스 인스턴스를 전달
button.setOnClickListener(object: OnclickListener {
    override fun onClick(v: View) { println("Button clicked!") }
})
```

- 인스턴스를 선언하지 않고 람다를 통해 동작 자체를(함수를) 넘겨 리팩토링
```kotlin
button.setOnClickListener { println("Button clicked!") }
```

### 람다와 컬렉션
- 컬렉션을 다룰 때 대부분 패턴화된 작업을 하게 되는데 코틀린은 람다를 적용해 컬렉션을 더 편리하게 다루는 라이브러리를 제공한다

**가장 나이 많은 사람을 찾는 예제 리팩토링**
```kotlin
data class Person(val name: String, val age: Int)

fun findTheOldest(people: List<Person>) {
    var maxAge = 0
    var oldest: Person? = null
    
    for (person in people) {
        if (person.age > maxAge) {
            maxAge = person.age
            oldest = person
        }
    }
    println(oldest)
}

fun main() {
    val people = listOf(Person("Alice", 30), Person("Bob", 25), Person("Charlie", 35))
    findTheOldest(people) // Person(name=Charlie, age=35)
}
```

- 코틀린의 라이브러리 함수 중 `maxByOrNull`을 사용해 리팩토링
```kotlin
data class Person(val name: String, val age: Int)

fun main() {
    val people = listOf(Person("Alice", 30), Person("Bob", 25), Person("Charlie", 35))
    println(people.maxByOrNull { it.age }) // Person(name=Charlie, age=35)
}
```

`maxByOrNull`을 살펴보자
```kotlin
inline fun <T, R : Comparable<R>> kotlin.collections.Iterable<T>.maxByOrNull(selector: (T) -> R): T? { /* compiled code */ }
```
- 제네릭 `T`로 받아 `R`로 반환하는 데 이 때 `R`은 `Comparable`을 상속받은, 즉 비교할 수 있는 타입이어야 한다
- `Iterable<T>`를 확장한 함수로 `T`를 요소로 가지는 반복 가능한 컬렉션에 대해 동작한다 (List, Set 등)
- 유일한 인자인 `selector`는 `T`를 받아 `R`을 반환하는 람다인데 `T`의 요소를 `R`로 변환하는 함수이다
- 리스트가 비어있을 경우 `null`을 반환하도록 반환타입이 `T?`이므로 호출할 때에도 `?.` 세이프 콜을 사용해야 한다
- 전달하는 인자는 람다의 결과값으로 비교할 수 있는 값이어야 한다 (여기서는 나이 `{ it.age }`)
- 실제 동작하는 코드(최댓값찾기)는 라이브러리로 바이너리로 제공되어 소스 코드에서는 볼 수 없네요

### 람다 문법
- 코틀린 람다식은 항상 중괄호(`{}`)로 감싸져 있으며 `->`로 인자 목록과 본문을 구분한다

**`run`을 사용해 인자로 받은 람다 바로 실행해주기**
```kotlin
fun main() {
    // 1.
    val result = run { 1 + 2 }
    println(result) // 3

    // 2.
    println(run { 1 + 2 }) // 3

    // 3.
    run { println(1+2) }  // 3

    // 람다 출력
    run { println {1 + 2} } // () -> kotlin.Int
}
```

**람다 호출 여러 방법**
```kotlin   
data class Person(val name: String, val age: Int)
fun main() {
    val people = listOf(Person("Alice", 30), Person("Bob", 25), Person("Charlie", 35))

    // 1. 정식 호출
    people.maxByOrNull({ p: Person -> p.age })
    
    // 2. 함수 호출 시 맨 뒤에 있는 인자가 람다식이면 그 람다를 괄호 밖으로 분리 가능
    people.maxByOrNull() { p: Person -> p.age }
    
    // 3. 람다가 유일한 인자이면 호출 시 빈 괄호 생략 가능
    people.maxByOrNull { p: Person -> p.age }
    
    // 4. 컴파일러가 추론할 수 있는 람다의 인자 타입 생략 가능 (확장 함수이기에 항상 컬렉션 원소 타입과 동일하므로 추론 가능)
    people.maxByOrNull { p -> p.age }
    
    // 5. 디폴트 파라미터 it 사용
    people.maxByOrNull { it.age }
    
    // 6. 멤버 참조 사용 가능
    people.maxByOrNull(Person::age)
}
```
- 람다가 유일한 인자일 때에는 괄호 밖으로 빼는 게 좋고, 둘 이상일 때에는 괄호 안에 넣는 게 좋다
- 컴파일러가 불평하기 전까진 타입 생략해주자
- 람다 안 람다가 내포된 경우 `it` 사용하지 말자
  - 어떤 람다에 속했는지 파악하기 어려우며 가독성이 떨어진다
  - `it`

**이전 챕터의 `joinToString()` 리팩토링**
```kotlin
data class Person(val name: String, val age: Int)
fun main() {
    val people = listOf(Person("Alice", 30), Person("Bob", 25))
    val name = people.joinToString(
        separator = ", ",
        transform = { p: Person -> p.name }
    )
    println(name) // Alice, Bob
}
```

- `joinToString()`의 `transform` 인자는 람다식이므로 괄호 밖으로 추출 가능
```kotlin
data class Person(val name: String, val age: Int)
fun main() {
    val people = listOf(Person("Alice", 30), Person("Bob", 25))
    val name = people.joinToString(", ") { p: Person -> p.name }
    println(name) // Alice, Bob
}
```

- 람다 안이 한 줄이 아닌 여러 줄인 경우 맨 마지막 식이 람다의 결괏값이 된다
  - 이 때는 명시적인 `return`이 필요하지 않다
```kotlin
fun main() {
    val sum = { x: Int, y: Int -> 
        println("x: $x, y: $y")
        x + y
    }
    println(sum(1, 2)) // 3
}
```

### 변수 캡처
- 익명 내부 클래스와 동일하게 람다 안에서도 함수의 파라미터와 로컬 변수를 참조할 수 있다
- 람다를 함수 안에서 정의하면 함수의 파라미터뿐 아니라 람다 정의보다 앞에 선언된 로컬 변수까지 참조할 수 있다
  - 람다 안에서 `final`이 아닌 변수를 참조할 수 있고 심지어 변경해도 된다!
    - 자바의 경우 `final`로 선언된 변수만 참조할 수 있고, `final`이니 변경할 수 없었다
  - 이렇게 람다 안에서 바깥 함수의 로컬 변수를 참조하는 것을 **변수 캡처**라고 한다
```kotlin
fun printMessagesWithPrefix(
    messages: Collection<String>,
    prefix: String
) {
    // 람다 안에서 final이 아닌 변수를 참조하는 모습..!
    messages.forEach { println("$prefix $it") }
}

fun main() {
    val errors = listOf("403 Forbidden", "404 Not Found")
    printMessagesWithPrefix(errors, "Error")
    // Error 403 Forbidden
    // Error 404 Not Found
}
```

- 람다 안에서 바깥 함수의 로컬 변수 변경 예제
```kotlin
fun main() {
    var sum = 0
    val numbers = listOf(1, 2, 3)
    
    // 람다 안에서 바깥 함수의 로컬 변수를 변경하는 모습..!
    numbers.forEach { sum += it }
    
    println(sum) // 6
}
```

**로컬 변수의 생명주기**
- 원래 함수 안 로컬 변수는 함수와 동일하지만 이 로컬 변수를 캡처한 람다를 반환하거나 다른 변수에 저장한 경우 생명주기가 바뀔 수 있다.
- 로컬 변수를 캡처한 람다를 그 로컬 변수를 선언한 함수가 끝난 후에 실행해도 람다의 본문 코드는 여전히 로컬 변수에 접근 가능하다.

이것이 가능한 이유가 뭘까?
- 먼저 캡처한 변수가 `final`이라면 그대로 람다 코드를 변수 값과 함께 저장해 사용한다
- `final`이 아닌 경우엔 변수를 특별한 래퍼로 감싸서 나중에 참조할 수 있게끔 한 다음, 그 특별한 래퍼에 대한 참조를 람다 코드와 함께 저장한다
```kotlin
// final 로컬 변수를 캡처한 경우
fun main() {
    val a = 1
    val lambda = { println(a) }
    lambda() // 1
}

// final이 아닌 로컬 변수를 캡처한 경우
fun main() {
    var a = 1
    val lambda = { println(a) }
    a = 2
    lambda() // 2
}
```

자바로 변환해보면 아래와 같다
```java
public final class MainKt {
    public static final void main() {
        // final 로컬 변수를 캡처한 경우
        final int a = 1;
        Function0 lambdaA = (Function0) (new Function0() {
            public Object invoke() {
                this.invoke();
                return Unit.INSTANCE;
            }

            public final void invoke() {
                int var1 = a;
                System.out.println(var1);
            }
        });
        lambdaA.invoke();   // 1
        
        // final이 아닌 로컬 변수를 캡처한 경우
        final Ref.IntRef b = new Ref.IntRef();  // 코틀린의 특별한 래퍼 Ref
        b.element = 1;
        Function0 lambdaB = (Function0) (new Function0() {
            public Object invoke() {
                this.invoke();
                return Unit.INSTANCE;
            }

            public final void invoke() {
                int var1 = b.element;
                System.out.println(var1);
            }
        });
        b.element = 2;  // Ref 래퍼의 내부에 있는 값을 변경
        lambdaB.invoke();
    }
}
```

- 람다를 실행할 때 로컬 변수를 캡처한 시점의 값을 사용하지 않고 호출 시점의 값을 사용한다는 점을 기억하자
  - 특히 람다를 이벤트 핸들러나 다른 비동기 작업에 활용할 경우 주의해야 한다
```kotlin
class Button(
    private var onClickListener: (() -> Unit)? = null,
) {
    fun onclick(listener: () -> Unit) { onClickListener = listener }
    fun click() { onClickListener?.invoke() }   // 람다 호출해주는 invoke
}

fun tryToCountButtonClicks(button: Button): Int {
    var clicks = 0
    button.onclick { clicks++ }
    return clicks
}

fun main() {
    val button = Button()
    val clicks = tryToCountButtonClicks(button) // 호출 시점에 이미 clicks 0으로 캡쳐
    button.click()
    button.click()
    button.click()

    println(clicks)  // 항상 0을 출력
}
```

- 당연한 말이지만 예상대로 동작시키려면 클릭 수를 저장하는 로컬 변수를 함수 밖으로 빼내면 된다.
```kotlin
class Button(
    var clicks: Int = 0,
    private var onClickListener: (() -> Unit)? = null,
) {
    fun onclick(listener: () -> Unit) { onClickListener = listener }
    fun click() { onClickListener?.invoke() }
}

fun tryToCountButtonClicks(button: Button) {
    button.onclick { button.clicks++ }
}

fun main() {
    val button = Button()
    tryToCountButtonClicks(button)
    button.click()
    button.click()
    button.click()

    println(button.clicks) // 3
}
```

### 멤버 참조
- 람다식으로 동작을 넘기고 싶은데 이미 그 동작이 함수로 선언된 경우엔 그 함수를 값으로 바꿔 넘기면 된다
- 클래스 이름과 참조하려는 멤버(프로퍼티나 메서드) 이름 사이에 `::`를 붙여주면 된다.
  - 자바와 동일하고, 멤버 참조라 불린다
  - 최상위 함수를 참조할 때에는 클래스 이름 없이 `::` 뒤에 멤버만 적어주면 된다
- 멤버 참조는 말 그대로 호출이 아닌 참조이기 때문에 뒤에 괄호(`()`)를 붙이지 않는다
```kotlin
data class Person(val name: String, val age: Int)

fun main() {
    val people = listOf(Person("Alice", 30), Person("Bob", 25))

    // 람다 사용
    val maxAge1 = people.maxByOrNull { it.age }
    println(maxAge1?.age) // 30

    // 멤버 참조 사용
    val maxAge2 = people.maxByOrNull(Person::age)
    println(maxAge2?.age) // 30
}
```

**멤버 참조 활용처**
- 람다가 인자가 여럿인 다른 함수에게 작업을 위임하는 경우 멤버 참조를 제공하면 파라미터 이름과 함수를 반복하지 않아도 되는 장점이 있다
```kotlin
data class Person(val name: String, val email: String)
fun sendEmail(person: Person, message: String) {
    println("Sending email to ${person.email} with message: $message")
}

// 람다 사용 시
val actionWithLambda = { person: Person, message: String -> 
    sendEmail(person, message)
}

// 멤버 참조 사용 시
val actionWithReference = ::sendEmail

fun main() {
    val person = Person("Alice", "alice@test.com")
    val message = "Hello, Alice!"

    actionWithLambda(person, message)
    actionWithReference(person, message)
}
```

**생성자 참조**
- 클래스 생성 작업을 연기하거나 저장해두고 싶을 때 생성자 참조를 활용하면 된다
- 생성자 참조는 클래스 이름과 생성자 이름 사이에 `::`를 붙여주면 된다
```kotlin
data class Person(val name: String, val age: Int)
fun Person.isAdult() = age >= 21

fun main() {
    val person = Person("Alice", 30)
    
    // 일반적인 호출
    println(person.isAdult()) // true
    
    // 멤버 참조
    val isAdultFunction = Person::isAdult   // isAdult() 메서드 참조해 저장해두고
    println(isAdultFunction(person))  // 필요한 시점에 호출해서 사용 
}
```

### 값과 엮인 호출 가능 참조
- 특정 객체 인스턴스에 대한 메서드 호출에 대한 참조도 만들 수 있다
  - 이전 멤버 참조는 클래스의 멤버에 대한 참조라서 인스턴스를 인자로 전달해 호출하는 느낌
  - 이번엔 특정 인스턴스를 이미 지정해두고 그에 대한 참조를 저장해서 인자 전달 필요없이 호출하면 엮여있는 특정 인스턴스에 대해 호출하는 느낌
```kotlin
data class Person(val name: String, val age: Int)

fun main() {
    val millo = Person("Millo", 27)
    
    val getAgeFunction = Person::age // 멤버 참조
    println(getAgeFunction(millo)) // 27
    
    val getMilloAgeFunction = millo::age // 값과 엮인 호출 가능 참조
    println(getMilloAgeFunction()) // 27
}
```

## 함수형 인터페이스 사용: 단일 추상 메서드
- 추상 메서드가 단 하나뿐인 인터페이스를 **함수형 인터페이스**라고 한다
  - 자바의 Runnable, Callable, Comparator ...
- 코틀린은 이 함수형 인터페이스를 파라미터로 받는 경우 호출 시 람다를 인자로 전달하면 컴파일러가 자동으로 람다를 함수형 인터페이스 인스턴스로 변환해준다.
  - 여기서 함수형 인터페이스 인스턴스는 함수형 인터페이스를 구현하는 익명 클래스의 인스턴스를 의미하며 컴파일러가 대신 이 인스턴스를 만들고 람다를 그 인스턴스의 유일한 추상 메서드 본문으로 만들어준다
```kotlin
data class Person(val name: String, val age: Int)

// 함수형 인터페이스
fun interface PersonComparator : Comparator<Person> {
    override fun compare(o1: Person, o2: Person): Int
}

fun main() {
    val people = listOf(
        Person("Alice", 30),
        Person("Bob", 25),
        Person("Charlie", 35)
    )

    // 람다를 사용하여 함수형 인터페이스 구현
    val ageComparatorWithLambda = PersonComparator { p1, p2 -> p1.age - p2.age } // 나이 오름차순
    
    // 익명 클래스를 사용해 함수형 인터페이스 구현
    val ageComparatorWithAnonymous = object : PersonComparator {
        override fun compare(o1: Person, o2: Person): Int {
            return o1.age - o2.age
        }
    }
    
    people.sortedWith(ageComparatorWithLambda)
    people.sortedWith(ageComparatorWithAnonymous)
    // Person(name=Bob, age=25), Person(name=Alice, age=30), Person(name=Charlie, age=35)
}
```
- 람다를 활용하는 것과 익명 객체로 넘겨주는 것 둘 다 동일한 동작이지만 익명 객체의 경우 매번 호출할 때마다 새 인스턴스가 생긴다 
- 람다의 경우 자신이 정의된 함수의 변수에 접근하지 않으면 함수가 호출될 때마다 람다에 해당하는 익명 객체(컴파일러가 자동으로 변환해준 것)가 재사용된다
  - 여기서 람다 사용 시에 만약 바깥 함수의 변수를 캡쳐했다면 그 때는 더 이상 같은 인스턴스를 재사용하지 않는다
    ```kotlin
    fun main() {
        var count = 0
        val incrementer = { count++ } // 바깥 함수의 변수를 캡쳐한 람다
        incrementer() // 1
        incrementer() // 2 -> 새롭게 바깥 함수의 변수를 캡처해야 하기에 새로운 인스턴스로
    }
    ```

하지만 `inline` 표시가 되어 있는 코틀린 확장 함수를 사용하는 컬렉션에 대해서는 람다를 전달해도 익명 클래스가 생성되지 않는다. 이건 나중에 알아보자.

### SAM 변환: 람다를 함수형 인터페이스로 명시적으로 변환하기
- SAM(Single Abstract Method) 변환은 람다를 함수형 인터페이스의 인스턴스로 명시적으로 변환해준다

**SAM 생성자 직접 사용 예시 1**
```kotlin
fun main() {
    // 익명 클래스 사용
    val thread = Thread(object : Runnable {
        override fun run() {
            println("Thread is running")
        }
    })

    // 일반적인 SAM 변환
    val samThread1 = Thread { println("Thread is running") }

    // SAM 생성자를 직접 사용 -> 변수에 저장하면 재사용 가능해짐
    val runnable = Runnable { println("Thread is running") }
    val samThread2 = Thread(runnable)
    val samThread3 = Thread(runnable)

    thread.start()
    samThread1.start()
    samThread2.start()
    samThread3.start()
}
```

**예시 2**
```kotlin
fun main() {
    val listener = OnClickListener { view ->
        val text = when (view.id) {
            button1.id -> "First button clicked"
            button2.id -> "Second button clicked"
            else -> "Unknown button clicked"
        }
        toast(text)
    }
    
    button1.setOnClickListener(listener)
    button2.setOnClickListener(listener)
}
```
- OnClickListener을 구현하는 객체 선언으로 리스너를 만들수도 있지만 SAM 생성자를 쓰는 게 더 간결하다고 하네요

**람다와 리스너 등록/해제**
- 람다에는 익명 객체와 달리 인스턴스 자신을 가리키는 `this`가 없어서 람다를 변환한 익명 클래스의 인스턴스를 참조할 방법이 없다
  - 람다 안에서의 this는 그 람다를 둘러싼 클래스의 인스턴스를 가리킴
- 만약 이벤트 리스너가 이벤트를 처리하다가 자신의 리스너 등록을 해제해야 할 경우 직접 참조해서 해제를 못 해주기 때문에 람다가 아닌 익명 객체를 사용해야 한다
  - 익명 객체 안에서는 `this`가 그 익명 객체 인스턴스 자신을 가리쳐 참조가 가능하기 때문

