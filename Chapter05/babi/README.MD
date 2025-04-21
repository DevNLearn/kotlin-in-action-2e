## 5장

### 람다식


#### 함수형 프로그래밍의 특징
- 일급 시민인 함수
  - 변수에 담을수 있다
  - 함수의 인자로 전달할 수 있다
  - 함수의 반환값으로 전달할 수 있다
- 불변성
  - 객체가 만들어진 이후, 내부 상태가 변하지 않음을 보장
- 부수 효과 없음
  - 함수가 동일한 입력에 대해 같은 출력을 반환한다.
---

#### 람다식으로 간단하게!

```kotlin
interface OnClickListener {
    fun onClick(v: View)
}

// 복잡;;
button.setOnClickListener(object: OnClickListener {
    override fun onClick(v: View){
        println("hello")
    }
})

// 간결!
button.setOnClickListener {
    println("hello")
}
```

---

#### 람다식의 문법
- 람다는 따로 선언해서 변수에 저장할 수 있다. (하지만 보통 변수에 저장하기보단 바로 함수에 인자로 넘긴다)
- 코틀린 람다식은 항상 중괄호(`{ }`)로 둘러싸여 있다.
- 화살표 (`->`) 를 중심으로 좌항이 파라미터(인자 목록), 우항이 본문 (람다 본문)이다.
- 람다식은 변수에 저장할 수 있다
```kotlin
fun main() {
    val sum = { x: Int, y: Int -> x + y }
    println(sum(1, 2))
}
```
- 코드의 일부분을 블록으로 둘러싸 실행해야 한다면 `run` 함수를 사용한다
  - 인자로 받은 람다를 실행하는 라이브러리 함수
```kotlin
fun main() {
    run { println(42) }
}
```

---

#### 람다식. 간단하게 개선작업
- 다음과 같이 `Person` 객체가 존재하고, `name`, `age` 두개의 필드를 가진다고 가정한다. 그리고 `Person` 객체의 리스트를 변수에 저장하자
```kotlin
val people = listOf(
        Person(name = "Alice", age = 29),
        Person(name = "Babi", age = 30)
)
```
- 가장 나이가 많은 `Person` 객체를 가져오려면 다음과 같이 사용할 수 있다
- `Person` 타입의 인자를 받고, 인자의 `age` 값을 반환한다.
```kotlin
people.maxByOrNull({ p: Person -> p.age })
```
- 함수 호출 시, 맨 뒤에 있는 인자가 람다식이라면 람다를 괄호 밖으로 뺄 수 있다.
```kotlin
people.maxByOrNull() { p: Person -> p.age }
```
- 람다가 함수의 유일한 인자며, 괄호 뒤에 람다를 사용했다면 빈 괄호를 삭제할 수 있다.
```kotlin
people.maxByOrNull { p: Person -> p.age }
```
- 여기서 컴파일러가 유추할수 있는 인자타입을 제외하자.
```kotlin
people.maxByOrNull { p -> p.age }
```
- 람다의 파라미터가 하나뿐이고, 타입을 컴파일러가 추론할 수 있다면 디폴트 파라미터 이름인 `it`을 사용할 수 있다
```kotlin
people.maxByOrNull { it.age }
```
- 람다의 멤버 참조를 통해 명시적으로 더 짧게 사용할 수도 있다
```kotlin
people.maxByOrNull(Person::age)
```

---

- 여러줄로 이루어진 람다 식의경우, 명시적으로 `return` 이 필요하지 않다. 맨 마지막에 있는 식이 람다의 결괏값이 된다.
```kotlin
fun main() {
    val sum = { x: Int, y: Int ->
        println("hello x >> [$x], y >> [$y]")
        x + y  // 결과값
    }
    
    // hello x >> [1], y >> [2]
    // 3
    print(sum(1, 2))
}
```

---

#### 변수 캡처
- 람다를 함수 내에서 정의하면, 함수의 파라미터 뿐만 아니라, 람다보다 앞에 선언된 로컬 변수까지 람다에서 사용할 수 있다.
- 자바의 람다와 다르게 `final` 이 아니라도 접근할 수 있으며, 값을 수정할 수도 있다.(자바는 `final`이라 수정할 수 없다)
```kotlin
fun printWithPrefix(messages: Colelction<String> , prefix: String) {
    messages.forEach {
        println("$prefix $it")
    }
}

fun main() {
    val errors = listOf("403 Forbidden", "404 Not Found")
    // Error: 403 Forbidden
    // Error: 404 Not Found
    printWithPrefix(errors, "Error: ")
}
```

```java
void printMessageWithPrefixJava(Collection<String> messages, String prefix){
    messages.forEach(it -> {
        System.out.println(prefix + it);
    });
}

public static void main(String[] args) {
  
    List<String> errors = List.of("403 Forbidden", "404 Not Found");
    // Error: 403 Forbidden
    // Error: 404 Not Found
    printMessageWithPrefixJava(errors, "Error: ");
}
```
- 참고로, 자바에서는 파라미터에 명시적으로 `final`을 써주지 않더라도 수정을 하지 않는다면 에러가 발생하지 않는다.
```java
void printMessageWithPrefixJava(Collection<String> messages, String prefix){
    messages.forEach(it -> {
        // Variable used in lambda expression should be final or effectively final
        System.out.println(prefix + it);
    });
    prefix = "change!";
}
```

---

#### 멤버참조
- 람다를 이용해서 인자로 넘길수 있게 되었다.
- 넘기려는 코드가 벌써 함수로 선언되어있다면, 함수를 값으로 바꿔 넘길수 있는 방법이 있다.
- 자바 8에서도 사용되던 방식으로 이중 콜론(`::`)을 사용한다.
- 이와같이 사용하는걸 멤버 참조라고 부른다
```kotlin
val getAge = Person::age
```

---


### 함수형 인터페이스: 단일 추상 메서드
- 인터페이스 내에 추상 메서드가 단 하나뿐이라면, **단일 추상 메서드 인터페이스**라고 불린다.
- 자바에서는 `Runnable`, `Callable` 등의 함수형 인터페이스들이 존재한다.

---

#### 람다를 파라미터로 전달
- 명시적으로 객체를 선언하면, 매번 호출할 때마다 새로운 인스턴스가 생기게 된다.
- 하지만 람다에서는 자신이 정의된 함수의 변수에 접근하지 않으면 함수가 호출될 때마다 람다에 해당하는 익명함수가 사용된다
- 람다가 자신을 둘러싼 환경의 변수를 캡처하면 더이상 함수 호출에 같은 인스턴스를 사용할 수 없다.
- 이럴때, 컴파일러는 호출마다 새로운 인스턴스를 만들고, 그 객체 안에 캡처한 변수를 저장한다.
```kotlin
fun handleComputation(id: String) {
    postponeComputation(1000) { // 호출마다 새로운 Runnable 인스턴스 생성
        println(id)     // id 캡처
    }
}
```
---

#### SAM(Single Abstract Method) 변환
- 컴파일러가 생성한 함수로 람다를 단일 추상 메서드 인터페이스의 인스턴스로 명시적으로 변환해준다.
- 함수형 인터페이스에서 인스턴스를 반환하는 함수가 있다면 람다를 직접 반환할 수 없고, 람다를 SAM 생성자로 감싸야 한다.
```kotlin
// 일반적인 방식
button.setOnClickListener(object : View.OnClickListener {
  override fun onClick(v: View) {
    println("Clicked!")
  }
})

// SAM 변환을 이용한 람다 전달
button.setOnClickListener { v ->
  println("Clicked!")
}

```

- SAM 생성자는 함수형 인터페이스의 이름과 같다. 
- 값을 반환할 때 외에 람다로 생성한 함수형 인터페이스 인스턴스를 변수에 저장할때도 SAM 생성자를 사용할 수 있다.
- JAVA의 함수형 인터페이스의 인스턴스를 반환 하고 싶을 경우 사용한다
- 컴파일러가 자동 생성 해 준다
 
---

### 수신 객체 지정 람다

#### with
- 객체 선언이후, 이름을 반복하지 않고 해당 객체에 연산을 하는 기능이다.
```kotlin
fun alphabet(): String {
    val builder = StringBuilder()
  return with(stringBuilder) {
      for (letter in 'A'..'Z'){
          this.append(letter) // 여기서 this 는 `val builder = StringBuilder()` 가 된다
      }
  }
}
```
- 그리고 `with` 구문 내에서는 `this`를 생략해도 상관없다.
```kotlin
fun alphabet(): String {
    val builder = StringBuilder()
  return with(stringBuilder) {
      for (letter in 'A'..'Z'){
          append(letter) // 동일
      }
  }
}
```
- `with`가 반환하는 값은 람다 코드를 실행한 결과며, 결과는 람다식의 본문의 마지막 식의 값이다.
- 하지만 람다의 결과 대신, **수신 객체**가 필요하다면 `apply` 라이브러리 함수를 사용하면 된다
---

#### apply

- `with` 과 거의 비슷한 기능을 하지만, `apply`는  **수신 객체**를 반환한다.
- 아까 위의 `with` 에서의 `alphabet()`함수를 재정의 해본다면 다음과 같다.
```kotlin
fun alphabet() = StringBuilder().apply {
    for (letter in 'A'..'Z') {
        append(letter)
    }
}
```
- 인스턴스를 만들면서 바로 프로퍼티를 초기화해야하는경우 쓰일수 있다.
---

#### also

- `also` 함수도 수신 객체를 받으며, 동작을 한 이후 **수신 객체**를 돌려준다
- 하지만 람다 안에서는 수신 객체를 인자로 참조하여야한다. (디폴트 이름인 `it`을 부여받음)
- 개인적으로는 로그 남길때 유용하게 사용했다
```kotlin
val file = File()
              .also{
                  logService.addLog(it)
              }
```
