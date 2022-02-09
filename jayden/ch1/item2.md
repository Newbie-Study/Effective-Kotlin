# 2. 변수의 스코프를 최소화하라.

변수와 프로퍼티의 스코프를 최소화하는 것이 좋다.

- 프로퍼티보다는 지역 변수를 사용하는 것이 좋음
- 최대한 좁은 스코프를 갖게 변수를 사용한다.

스코프를 최소화하는 것은 프로그램을 추적하고 관리하기 쉽게 하고, 읽기 쉽게 만들어주는 이유에서이다.

### 캡처링

```kotlin
suspend fun capturing() {
    var number: Int

    for (i in 0 until 10) {
        number = i

        GlobalScope.launch {
            print("$number ")
        }
    }
    delay(1000L)
}
// 9. 9, 9, 9, 9, 9, 9, 9, 9, 9
```

launch의 실행이 지연되므로 for문을 마친 number의 최종값으로 찍히게 된다.

위와 같은 잠재적인 캡처링 문제를 주의해야 한다

가변성을 피하고 변수의 스코프 범위를 최소화하면 이러한 문제를 자연스럽게 회피할 수 있다.

### Iterable

- 각 단계의 처리는 바로 끝나고 즉각 그 결과를 반환한다
- 전체 collection 에 대해 각 단계의 수행을 완료하고 다음 단계로 넘어간다.

### Sequence

- 전체 단계가 처리된 결과가 요구됬을 때에 실제 연산이 일어나기에 Lazily 이다.
- 각각 하나의 element 에 대해 모든 단계를 수행한다.
- 중간 과정에서 새로운 collection 이 만들어지지 않으며 first collection 에 대한 reference 와 어떠한 연산이 수행되는지를 저장해둔 sequence object 가 반환된다.

### 1. Iterable 과 Sequence 의 동작 비교

1) Iterable

```kotlin
val words = "The quick brown fox jumps over the lazy dog".split(" ")
words.filter { 
	println("filter: $it")
	it.length > 3 
}.map {
	println("length: ${it.length} ($it)")
	it.length
}

// output
filter: The
filter: quick
filter: brown
...
filter: lazy
filter: dog
length: 5 (quick)
length: 5 (brown)
...
length: 4 (lazy)
```

2) Sequence
```kotlin
val words = "The quick brown fox jumps over the lazy dog".split(" ")
val wordsSequence = words.asSequence()
val lengthsSequence = wordsSequence.filter {
    println("filter: $it")
    it.length > 3
}.map {
    println("length: ${it.length} ($it)")
    it.length
}
println(lengthsSequence.toList())

// output
filter: The
filter: quick
length: 5 (quick)
filter: brown
length: 5 (brown)
...
filter: lazy
length: 4 (lazy)
filter: dog
```