# 1. 가변성을 제한하라.

var 또는 mutable (읽고 쓸 수 있는 프로퍼티)을 사용하면 해당 변수 또는 객체는 상태를 가질 수 있게 되는데

요소가 상태를 갖는 경우, 해당 요소의 동작은 사용 방법뿐 아니라 그 이력(history)에도 의존하게 된다.

```kotlin
class BankAccount {

    /**
     * var로 선언함으로써 balance 변수는 변경 가능한 Long 값을 가지게 됌.
     */
    var balance: Long = 0L

    fun withDraw(amount: Long) {
        if (balance < amount) { // balance 값이 음수라면 출금할 수 없음. balance 의 이력에 의존할 수 밖에 없다.
            throw IllegalStateException()
        }
        balance -= amount
    }
}
```

mutable 프로퍼티인 balance로 간단하게 출금 로직을 구현하였지만 이는 양날의 검이다.

예를 들어, balance 값에 따라 타입이 정해진다고 한다면

```kotlin
class BankAccount {

    enum class Type {
        DEFAULT, MINUS, VIP
    }

    val type: Type
        get() = when {
            balance < 0 -> Type.MINUS
            balance > 100_000_000 -> Type.VIP
            else -> Type.DEFAULT
        }

    var balance: Long = 0L

    fun withDraw(amount: Long) {
        if (balance < amount) { 
            throw IllegalStateException()
        }
        balance -= amount
    }
}
```

이 때, balance 값은 변경 가능하므로 해당 BankAccount의 타입은 런타임에서 디버깅을 해봐야 유추가 가능하다.

만약 balance 값이 val 이라면 코드 레벨에서 해당 계좌의 타입이 무엇인지 알 수 있을 것이다.

```kotlin
class BankAccount {

    enum class Type {
        DEFAULT, MINUS, VIP
    }

    val type: Type = when {
        balance < 0 -> Type.MINUS
        balance > 100_000_000 -> Type.VIP
        else -> Type.DEFAULT
    }

    val balance: Long = 1_000_000L

    fun withDraw(amount: Long) {
        if (balance < amount) { 
            throw IllegalStateException()
        }
        balance -= amount
    }
}
```

이처럼 상태를 관리함으로써 실행을 추론하기 어려워진다.

### 코틀린에서 가변성 제한하기

var은 getter와 setter를 모두 제공하지만, val은 변경이 불가능하므로 getter만 제공한다.

그래서 val은 var로 오버라이드할 수 있다.

```kotlin
/**
 * val -> var override enable
 */
interface Element {
    val active: Boolean
}
class ActualElement: Element {
    override var active = true
}
```

getter로 정의된 변수는 스마트 캐스팅할 수 없다.

getter를 정의하면 사용할 때마다 값을 새로 얻어오는데, 사용하는 시점에 따라 다른 결과가 나오게 되므로

컴파일러는 해당 값을 에측할 수가 없다.

```kotlin
/**
 * getter can't smart casting
 *  - 값을 사용하는 시점의 name 에 따라서 다른 결과가 나올 수 있기 때문
 */
val name: String?
    get() = "http"

fun testCasting() {
    if (name != null) {
        println(name.length) // error
    }
}
```

### 가변 컬렉션과 읽기 전용 컬렉션 구분하기

map과 set을 사용할 때 mutable한 키를 사용해서는 안된다.

```kotlin
class Person(
    var name: String
) {
    override fun equals(other: Any?): Boolean {
        if (this === other) return true
        if (javaClass != other?.javaClass) return false

        other as Person

        if (name != other.name) return false

        return true
    }

    override fun hashCode(): Int {
        return name.hashCode()
    }
}

fun mutableKeyIsUnAppropriate() {
    val map = mutableMapOf<Person, Int>()
    val person = Person("moon")

    map[person] = 10
    println(person in map) // true

    person.name = "shin"
    println(person in map) // false
}
```

equals()가 true이면 hashCode()를 비교하게 되는데 이 hashCode()는 보통의 경우 클래스의 프로퍼티 값에 따라 달라지게 되므로 같은 객체일지라도 Map에서 더 이상 찾을 수 없게 된다.

### 다른 종류의 변경 가능 지점

```kotlin
val list1 = mutableListOf<Int>()
var list2 = listOf<Int>()
```

mutable collection을 사용하면 아이템에 추가, 삭제와 같은 조작은 편리해지나 멀티스레드 처리나 해당 아이템의 변경에 대한 이벤트를 추적하는 것이 어렵다.

mutable property를 사용하면 아이템 추가, 삭제와 같은 조작은 불편해지나 멀티스레드 처리에 대해 더 안정성이 있고 사용자 정의 세터 또는 델리게이트를 활용해서 변경을 추적할 수 있다.

```kotlin
// immutable
var list = listOf<Int>()

fun addItem(element: Int) { 
    list = list.toMutableList().apply {
        add(element)
    }
}

// mutable
val mutableList = mutableListOf<Int>()

fun addItem(element: Int) {
    mutableList.add(element)
}
```

변경에 대한 추적 및 멀티스레드 처리에 대한 니즈를 고려해서 방법을 선택하면 좋을 것 같다.