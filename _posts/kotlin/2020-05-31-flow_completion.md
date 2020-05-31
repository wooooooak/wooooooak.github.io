---
layout: post
title: Flow Completion
category: Kotlin
tags: [coroutine_flow, flow_completion]
comments: true
---

flow가 방출한 모든 값을 정상적으로 다 수집했기 때문에 flow가 종료되었든, 또는 에러가 발생해서 flow가 종료되었든 종료 이후에 어떤 액션을 취하고 싶을 수가 있습니다. 어떤 이유에서든 flow가 끝났을 때 액션을 취하기 위해 명령형, 또는 선언형으로 코드를 작성할 수 있습니다.

## Imperative finally block

flow collection이 끝났을 때 어떤 행동을 하고자 한다면, try/catch 를 사용하여 명령형으로 코드를 작성 할 수 있습니다.

```kotlin
fun foo(): Flow<Int> = (1..3).asFlow()

fun main() = runBlocking<Unit> {
    try {
        foo().collect { value -> println(value) }
    } finally {
        println("Done")
    }
}
```

```kotlin
// 결과
1
2
3
Done
```

## Declarative handling

선언형으로 코드를 작성하고 싶다면, [onCompletion](<[https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/on-completion.html](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/on-completion.html)>) 중간 연산자를 사용하세요. `onCompletion` 연산자는 colletion이 끝나고 나면 실행됩니다.

```kotlin
foo()
    .onCompletion { println("Done") }
    .collect { value -> println(value) }
```

`onCompletion`의 장점은 선언형이라는 점에서 그치지 않습니다. 가장 큰 장점은 `onCompletion` 이 받는 람다의 nullable한 `Throwable` 파라미터 입니다. 즉 `onCompletion` 이 받는 람다의 파라미터(`Throwable`)가 null이면 collection이 완전하게 수행되었음을 알 수 있고, null이라면 exception으로 인해 flow가 completion 되었음을 알 수 있습니다. 아래에서 `foo()` flow가 처음 1을 방출하고 에러를 발생시키는 코드를 볼 수 있습니다.

```kotlin
fun foo(): Flow<Int> = flow {
    emit(1)
    throw RuntimeException()
}

fun main() = runBlocking<Unit> {
    foo()
        .onCompletion { cause -> if (cause != null) println("Flow completed exceptionally") }
        .catch { cause -> println("Caught exception") }
        .collect { value -> println(value) }
}
```

```kotlin
// 결과
1
Flow completed exceptionally
Caught exception
```

`onCompletion` 은 `catch` 와는 다르게 `exception` 을 처리하지는 않습니다. 바로 위의 예제에서 볼 수 있듯이 `exception`은 `onCompletion` 이후 계속해서 downstream으로 흐릅니다. 그렇기 때문에 에러 처리는 `catch`에서 합니다.

## Successful completion

`onCompletion`이 `catch`와 다른 점은, `onCompletion`은 항상 모든 예외를 받는다는 점입니다. 성공적으로 collection이 완료 되었더라도 `nul`l을 수신하고 코드가 실행되는 반면, catch는 전혀 실행되지 않습니다.
