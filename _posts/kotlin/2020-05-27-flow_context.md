---
layout: post
title: Flow Context
category: Kotlin
tags: [coroutine_flow, flow_context]
comments: true
---

이 포스팅은 [공식문서](https://kotlinlang.org/docs/reference/coroutines/flow.html#flow-context)를 보며 공부한 내용입니다.

# Flow context

flow는 자신을 호출한 코루틴의 context에서 값을 발행하게 되어있습니다. 예를 들어, 아래의 `foo` 함수는 `foo` 의 구현과는 무관하게 `foo`를 호출하는 곳의 context에 따라 실행됩니다.

```kotlin
withContext(context) {
    foo.collect { value ->
        println(value) // run in the specified context
    }
}
```

이러한 특성을 context preservation, 즉 컨텍스트 보존이라고 부릅니다.

기본적으로 `flow { ... }` 빌더 안의 코드는 collet를 호출하는 측에서 제공하는 context로 실행됩니다. 아래에는 emit되는 숫자와 함께 쓰레드 정보를 찍는 예제가 있습니다.

```kotlin
fun foo(): Flow<Int> = flow {
    log("Started foo flow")
    for (i in 1..3) {
        emit(i)
    }
}

fun main() = runBlocking<Unit> {
    foo().collect { value -> log("Collected $value") }
}
```

결과는 아래와 같습니다.

```kotlin
[main @coroutine#1] Started foo flow
[main @coroutine#1] Collected 1
[main @coroutine#1] Collected 2
[main @coroutine#1] Collected 3
```

`foo().collect` 가 메인 쓰레드에서 호출되었기 때문에, `foo` flow의 코드블럭 역시 메인쓰레드로 호출됩니다.

## Wrong emission withContext

그러나 시간이 오래걸리는 CPU작업은 `Dispatcher.Default` 컨텍스트에서 실행시키고, UI-update는 `Dispatchers.Main` 컨텍스트에서 실행시키는 경우가 많을거라 생각합니다. 보통같았으면 `withContext`를 사용하여 컨텍스트를 바꾸었을 겁니다. 그러나 `flow { ... }` 빌더는 컨텍스트 보존이라는 특성이 있기 때문에 다른 컨텍스트에서 값을 emit하는 것은 허용되어 있지 않습니다.

아래는 에러를 발생하는 코드입니다.

```kotlin
fun foo(): Flow<Int> = flow {
    // The WRONG way to change context for CPU-consuming code in flow builder
    kotlinx.coroutines.withContext(Dispatchers.Default) {
        for (i in 1..3) {
            Thread.sleep(100) // pretend we are computing it in CPU-consuming way
            emit(i) // emit next value
        }
    }
}

fun main() = runBlocking<Unit> {
    foo().collect { value -> println(value) }
}
```

```kotlin
Exception in thread "main" java.lang.IllegalStateException: Flow invariant is violated:
        Flow was collected in [CoroutineId(1), "coroutine#1":BlockingCoroutine{Active}@5511c7f8, BlockingEventLoop@2eac3323],
        but emission happened in [CoroutineId(1), "coroutine#1":DispatchedCoroutine{Active}@2dae0000, DefaultDispatcher].
        Please refer to 'flow' documentation or use 'flowOn' instead
    at ...
```

## flowOn operator

위의 에러로그를 보면 `flowOn` 함수가 언급되고 있습니다. `flowOn` 함수는 flow emission의 context를 바꾸는데 사용됩니다. 아래는 `flowOn`을 사용하여 context를 바꾸는 예시입니다.

```kotlin
fun foo(): Flow<Int> = flow {
    for (i in 1..3) {
        Thread.sleep(100) // pretend we are computing it in CPU-consuming way
        log("Emitting $i")
        emit(i) // emit next value
    }
}.flowOn(Dispatchers.Default) // RIGHT way to change context for CPU-consuming code in flow builder

fun main() = runBlocking<Unit> {
    foo().collect { value ->
        log("Collected $value")
    }
}
```

메인쓰레드에서 collection을 하는 동안, 백그라운드 쓰레드에서는 `flow {...}` 빌더 안의 코드들이 수행되게 됩니다.

주목할 점은, `flowOn`을 사용함으로써 기본적으로 하나의 코루틴이 emit과 collect를 순차적으로 하던 것을 각각의 코루틴이 동시에 수행하도록 바뀌었다는 점입니다. 이제 collection은 하나의 코루틴("coroutine#1')에서 일어나고, emission은 다른 스레드 위에서 돌아가는 또 다른 코루틴("coroutine#2')에 의해 동작합니다. 이 두가지 일은 동시에 일어나게 됩니다. 즉 flowOn 연산자는 context를 바꾸어야 할 때 새로운 코루틴을 생성하게 되는 것입니다.
