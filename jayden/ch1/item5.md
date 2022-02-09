# 5. 예외를 활용해 코드에 제한을 걸어라

코틀린에서 코드의 동작에 제한을 걸 때 아래와 같은 방법을 사용할 수 있다.

- require 블록 : 아규먼트를 제한할 수 있다.
- check 블록 : 상태와 관련된 동작을 제한할 수 있다.
- assert 블록 : 어떤 것이 true인지 확인할 수 있다. (테스트에서만 작동)
- return 또는 throw와 함께 사용하는 Elvis 연산자

### 아규먼트

```kotlin
fun parseProfileData(data: Intent) {
    val profileId = data.getLongExtra("KEY_PROFILE_ID", 0L)
    require(profileId > 0L) { "profileId must be > 0L" }
    ...
}
```

이와 같은 형태의 입력 유효성 검사 코드는 함수의 가장 앞부분에 배치하여 코드를 읽을 때 쉽게 확인할 수 있다.

require 함수는 조건을 만족하지 못할 때 무조건적으로 IllegalArgumentException을 발생시키므로 제한을 무시할 수 없다.

그러나, 안드로이드 어플리케이션에서 저렇게 제한을 두게 된다면 크래시 에러를 피할 수 없지 않을까?

제한을 두어서 에러 상황을 미리 파악하거나 오사용을 방지하는 것은 좋지만 변수가 많은 안드로이드에서 사용하기가 현실적으로 어려운 부분이 많아보인다.

```kotlin
fun parseProfileData(data: Intent) {
    val profileId = data.getLongExtra("KEY_PROFILE_ID", 0L)
    if (profileId <= 0L) {
        Log.e("...", "profileId must be > 0L")
        return
    }
    ...
}
```

현실적으로는 위와 같은 예외 처리를 사용하게 되지 않을까 싶다.

### 상태

```kotlin
fun speak(text: String) {
    check(isInitialized)
    ...
}
```

어떤 구체적인 조건을 만족할 때만 함수를 사용할 수 있게 해야 할 때가 있다.

check 함수는 require와 비슷하지만, 지정된 예측을 만족하지 못할 때 IllegalStateException을 throw한다.