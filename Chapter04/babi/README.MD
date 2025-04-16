## 4장

### 코틀린에서의 클래스와 인터페이스 기본적인 내용

- 자바와 다르게 구현이 있는 메서드도 정의할 수 있다.
- 클래스를 상속 또는 인터페이스를 구현하려면 클래스 뒤에 상속/구현 하려는 클래스를 콜론(`:`) 을 붙이면 된다.
- 메서드 오버라이드시, `override` 변경자가 필수다

```java
// java
interface Clickable {
    void click();
}

public class Button implements Clickable {
    @Override
    public void click(){
        // Todo
    }
}

```


```kotlin
interface Clickable {
    fun click()
}

class Button: Clickable {
    override fun click(){
        // Todo
    }
}
```

---

- 두개이상의 인터페이스를 하나의 클래스에서 구현하고, 동일한 이름의 메서드와 파라미터가 존재하더라도, 구현 클래스에서 명확하다면 에러가 나지 않는다.
- 그러니깐 간단한 코드로 본다면
#### A, B 인터페이스의 구현없는 메서드
```kotlin
// A interface
interface KotInterface1 {
    fun test()
}

// B interface
interface KotInterface2 {
    fun test()
}

// A, B 구현체
class TestKot : KotInterface1, KotInterface2{
    // 구현 필수
    override fun test() {
        print("hi")
    }
}
```
---
#### 한쪽만 구현되어있는 메서드
```kotlin
// A interface
interface KotInterface1 {
    fun test() = print("hi I'm 1")
}

// B interface
interface KotInterface2 {
    fun test()
}

// A, B 구현체
class TestKot : KotInterface1, KotInterface2{
    // 실제로 구현하지 않더라도 `KotInterface1` 인터페이스에서 디폴트 구현이 되어있어서 상관 없음
//    override fun test() {
//        // implements
//    }
}
```
---
#### 양쪽 다 구현 메서드를 사용
```kotlin
// A interface
interface KotInterface1 {
    fun test() = print("hi I'm 1")
}

// B interface
interface KotInterface2 {
    fun test() = print("hi I'm 2")
}

// A, B 구현체
class TestKot : KotInterface1, KotInterface2{
    // 구현하지 않으면 어떤 디폴트 메서드를 사용해야할지 모르기때문에 구현 필수
    override fun test() {
        print("hi")
        
        // 특정 메서드를 지정하고 싶다면 super 를 사용해서 지정할 수 있다
        super<KotInterface1>.test2()
    }
}
```

---

### 변경자들

- 코틀린은 기본적으로 모든 클래스와 메서드는 `final` 을 사용하여 하위 클래스가 오버라이드를 할 수 없다.
  - 클래스를 작성한 사람의 의도와 다른 방식으로 메서드를 오버라이드하는 위험이 존재하여 닫았다고 한다.
- 상속을 허용하기 위해서는 `open` 변경자를 붙여주면 된다.
- 또한 코틀린에서는 `abstract` 클래스는 `abstract` 함수를 만들수 있고, 해당 함수는 반드시 상속받는쪽에서 구현을 해야한다.
- 사실 코드가 보기 편하긴해서, 인터페이스 하나, 클래스 두개로 만들수있는 상황을 만들어봤다.

```kotlin
// 최초 인터페이스
interface FirstInterface {
  fun basicMethod()
  fun secondMethod()
  fun defaultMethod() = print("It's B")
}

// 인터페이스를 구현한 클래스
// open 으로 선언되어있어 상속을 또 받을수 있다
open class SecondClass : FirstInterface {
  override fun basicMethod() = print("Its' basicMethod")

  // 해당 메서드는 더이상 오버라이드 할수 없다.
  final override fun secondMethod() = print("cannot override")

  // 해당 메서드는 디폴트메서드로 지정되어있기에 반드시 오버라이드 하지않아도 된다
  // override fun defaultMethod() = print("Hello~")

  // 해당 클래스에서 생성한 신규 메서드. 하지만 final 로 선언되어 더이상 하위에서 오버라이드 할수 없다.
  fun newMethodA() = print("It's new A")

  // 해당 클래스에서 생성한 신규 메서드. open 으로 열려있어 하위에서 오버라이드 할수 있다.
  open fun newMethodB() = print("It's new B")
}

class ThirdClass : SecondClass {
  // SecondClass 에서 구현되어있어서, 굳이 반드시 오버라이드 하지않아도 된다
  // override fun basicMethod() = print("Its' new new A")

  // SecondClass 에서 구현되지 않았지만, FirstInterface 에서 구현이 열려있기때문에 오버라이드가 가능하다 
   override fun basicMethod() = print("Its' new new A")

  // 여기서는 secondMethod 를 오버라이드 할수 없다.

  // 여기서는 newMethodA 를 오버라이드 할수 없다.

  // 구현체에서 선언된 open 메서드는 하위에서 오버라이드 가능하다.
  override fun newMethodB() = print("yea~")
}
```

- 변경자들을 표로 정리해본다

| 변경자      | 오버라이드 가능?           | 설명                                       |
|----------|---------------------|------------------------------------------|
| final    | 오버라이드할 수 없음(코틀린 기본값) | 클래스 멤버의 기본 변경자                           |
| open     | 오버라이드 가능            | 오버라이드를 가능하게 만들어주는 변경자                    |
| abstract | 반드시 오버라이드 해야함       | 추상 클래스의 멤버에만 붙일수 있고, 추상 멤버에는 구현이 있으면 안된다. |
| override | 오버라이드 가능 여부와는 관계 없음 | 상위 클래스나 인스턴스의 멤버를 오버라이드 하는중.             |


---

- 가시성 변경자는 외부 접근의 제어 역할을 한다. 자바에서도 `public`, `protected`, `private`, `default` 등이 존재한다.
- 기본적으로 코틀린은 메서드에서 아무것도 기재하지않는다면 `public` 이 기본으로 사용된다.
- 하지만 명시적으로 적는걸 강제하고 싶다면 특정 옵션도 제공한다.
  - `Xexplicit- api={strict|warning}` 컴파일러 옵션을 지정하면 명시적 API 모드를 활성화 할 수 있다.
- 코틀린의 가시성 변경자들은 표로 보면 다음과 같다.

| 변경자        | 클래스 멤버             | 최상위 선언           |
|------------|--------------------|------------------|
| public(기본) | 모든 곳에서 접근 가능       | 모든 곳에서 접근 가능     |
| internal   | 같은 모듈 내에서만 접근 가능   | 같은 모듈 내에서만 접근 가능 |
| protected  | 하위 클래스에서 접근 가능(상속) | 최상위 선언할 수 없음     |
| private    | 같은 클래스 내에서만 접근 가능  | 같은 클래스 내에서만 접근 가능 |
- 코틀린에서 internal 변경자로 선언되어있더라도, 자바에서는 internal의 개념이 존재하지 않기 때문에 접근 가능하다.

---

- 봉인된 클래스라는것도 존재하는데, 코틀린 컴파일러에서는 `when`으로 타입을 검사할때 디폴트 분기인 `else`를 붙이도록 강제한다.
- 하지만 무조건 `else`는 없는게 확실한경우라면?
- 예전에 구글 드라이브와 같은 프로젝트를 할때 잘 사용했었던 클래스다. 객체의 종류가 파일과 폴더 두가지밖에 존재하지 않기 때문에 사용하기 적합했다.
- 그리고 해당 클래스를 사용하기위해 몇가지 조건이 있다
  - 하위 클래스들은 봉인된 클래스가 정의된 패키지와 같은 패키지에 속해야한다.
  - 하위클래스가 모두 같은 모듈 안에 위치해야한다.
  - 컴파일 시점에 알수있어야 한다.

```kotlin

sealed class Node(val name: String)

class File(val name: String): Node(name)

class Directory(val name: String): Node(name)

fun isFileNode(node: Node){
    return when (node){
        is File -> true
        is Directory -> false
        // else 가 필요하지 않다!!
        // else -> throw RuntimeException()
    }
}

```

---

### 클래스 생성
- 코틀린에서의 클래스는 주 생성자와 부 생성자가 나뉜다.
- 주 생성자는 클래스 선언과 동시에 생성자를 만드는 형식이다

```kotlin
class User(val name: String)
```

