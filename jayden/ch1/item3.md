# 3. 최대한 플랫폼 타입을 사용하지 말라

플랫폼 타입(platform type)이란, 다른 프로그래밍 언어에서 전달되어서 nullable인지 아닌지 알 수 없는 타입을 말한다.

그래서 자바를 코틀린과 함께 사용할 때, 자바 코드를 직접 조작할 수 있다면 가능한 @Nullable과 @NonNull 어노테이션을 붙여서 사용해야 한다.

```Java
public class JavaClass {
    public String getValue() { 
        return null;
    }
}
```

```kotlin
fun main() {
    val value = JavaClass().value // 타입은 String!
    println(value.length) // 코드 레벨에서는 NonNull인지 알 수 없음, 런타임 NPE 발생
}
```

따라서 다른 프로그래밍 언어를 같이 사용해야 한다면 어노테이션을 이용하여 이러한 위험을 사전에 제거해야 한다.