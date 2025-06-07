# 널이 될 수 있는 값
> 널이 될 수 있는 타입  
> 널이 될 가능성이 있는 값을 다루는 구문의 문법  
> 널이 될 수 있는 타입과 널이 될 수 없는 타입의 변환  
> 코틀린의 널 가능성 개념과 자바 코드 사이의 상호운용성

### NullPointerException을 피하고 값이 없는 경우 처리 : 널 가능성
컴파일러가 여러 가지 오류를 컴파일 시 미리 감지해서 실행 시점에 발생할 수 있는 예외의 가능성을 줄임

### 널이 될 수 있는 타입으로 널이 될 수 있는 변수 명시
널이 될 수 있는 타입은 프로그램 안의 프로퍼티나 변수에 null을 허용하게 만드는 방법  
어떤 변수가 널이 될 수 있으면 그 변수에 대해 메소드 호출을 했을 때 npe가 발생할 수 있음  
코틀린은 이런 메소드 호출을 금지함으로써 예외를 방지함

null이 인자로 들어올 수 없다면 : `fun strLen(s: String) = s.length`  
`strLen(null)` 이렇게 하면 에러 발생함  
null을 포함하는 모든 문자열을 인자로 받을 수 있게 하려면 타입 이름 뒤에 물음표를 붙여야 함 `fun strLenSafe(s: Sting?) = ...`

어떤 타입이든 타입 이름 뒤에 물음표를 붙이면 그 타입의 변수나 프로퍼티에 null 참조를 저장할 수 있음  
물음표가 없는 타입은 null 참조를 저장할 수 없다는 의미, 기본적으로 모든 타입은 null이 아닌 타입

널이 될 수 있는 값을 널이 될 수 없는 타입의 변수에 대입할 수 없음

```kotlin
fun main() {
    val x: String? = null
    val y: String = x
    // Error
}
```

널이 될 수 있는 타입의 값으로 대체할 수 있는 것? null과 비교하는 것  
일단 null과 비교하고, null이 아님이 확실한 영역에서는 해당 값을 null이 아닌 타입의 값처럼 사용 가능

### 안전한 호출 연산자로 null 검사와 메소드 호출 합치기: ?.
`str?.uppercase()`   
`if(str != null) str.uppercase()`  
위의 둘은 같음

호출하려는 값이 null이면 이 호출은 무시되고, null이 아니면 일반 메소드 호출처럼 작동  
따라서 저 결과물이 null이 될 수 있음  

```kotlin
class Employee(val name: String, val manager: Employee?)

fun managerName(employee: Employee): String? = employee.manager?.name

fun main() {
    val ceo = Employee("Boss", null)
    val developer = Employee("Bob", ceo)
    println(managerName(developer)) // Boss
    println(managerName(ceo)) // null
}
```

### 엘비스 연산자로 null에 대한 기본값 제공: ?:
null 대신 사용할 기본값 지정할 때 사용하는 연산자  
```kotlin
fun greet(name: String?) {
    val recipient: String = name ?: "unnamed"
    println("Hello, $recipient")
}
```

throw와 엘비스 연산자 함께 사용 가능
```kotlin
val address = person.company?.address
    ?: throw IllegalArgumentException("No address") // 예외 발생시킬 수 있음
```

### 예외를 발생시키지 않고 안전하게 타입을 캐스트하기: as?
as를 사용할 때마다 is로 미리 as 변환 가능한 타입인지 검사해볼 수도 있지만  
```kotlin
override fun equals(o: Any?): Boolean {
    val otherPerson = o as? Person ?: return false
    return otherPerson.firstName == firstName && otherPerson.lastName == lastName
}
```
파라미터로 받은 값이 원하는 타입인지 쉽게 검사하고 캐스트할 수 있고, 맞지 않으면 쉽게 false 반환할 수 있음

### 널이 아님 단언: !!
어떤 값이든 널이 아닌 타입으로 강제로 바꿀 수 있음  
null에 대해 !! 하면 NPE 발생

호출된 함수가 언제나 다른 함수에서 null이 아닌 값을 전달 받는다는 사실이 분명하다면 굳이 null 검사를 안해도 되게 단언문 사용하면 됨

### let 함수
안전한 호출 연산자와 함께 사용하면 원하는 식을 평가해서 결과가 null인지 검사한 다음 그 결과를 변수에 넣는 작업을 간단한 식을 사용해 한꺼번에 처리 가능  
```kotlin
fun sendEmail(email: String) {} //null이 아닌 파라미터 넘겨받음
fun main() {
    val email: String? = "abc@bcd.com"
    if(email != null)
        sendEmail(email)
}

// email이 null이 아닐 때만 호출됨
email?.let {email -> sendEmail(email)} 
email?.let { sendEmail(it)}
```

