# 4. inferred 타입으로 리턴하지 말라

inferred 타입은 정확하게 오른쪽에 있는 피연산자에 맞게 설정된다.

```kotlin
sealed class ToastMessage
class Short(val message: String) : ToastMessage()
class Long(val message: String) : ToastMessage()

interface ToastProvider {
    // 오른쪽 피연산자의 타입인 Short로 추론됌
    fun provide() = Short("Default Message")
}

class MyToastProvider : ToastProvider {
    override fun provide(): Short {  
        ...
    }
}
```

이와 같은 상황은 외부 API, 즉 인터페이스를 만들 때나 발생할 문제로 보여지는데 이때 추론 타입을 사용하지 않아야 할 것 같다.

오른쪽 피연산자로 추론된다는 규칙을 잘 기억하자. 