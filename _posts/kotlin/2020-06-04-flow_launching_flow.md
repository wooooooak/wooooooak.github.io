---
layout: post
title: Launching flow
category: Kotlin
tags: [coroutine_flow, launchIn]
comments: true
---

이 포스팅은 [공식문서](https://kotlinlang.org/docs/reference/coroutines/flow.html#launching-flow)를 보며 공부한 내용입니다.

flow를 사용하면 어딘가로부터 비동기적으로 들어오는 이벤트를 표현하기 쉽습니다. 보통 이런 경우, 비동기적으로 흘러 들어오는 이벤트(값)을 처리하기위해 `addEventListener` 메서드에다가 코드 블럭을 작성해 놓으면, 나머지 다른 작업(다른 코드 블럭)들 역시 수행시킬 수 있습니다. flow에서는 `onEach` 연산자가 그 역할을 합니다. 하지만 `onEach` 는 중간 연산자이기 때문에 그 자체만으로는 기능을 할 수 없고, `collect` 같은 terminal 연산자가 필요합니다.

`onEach` 연산자 다음에 `collect` terminal 연산자를 사용하면, flow를 collect 하는 코드 이후에 배치된 코드들은 collection이 완료되고 나서야 실행이 됩니다. 아래 예제를 보면 "Done"이 `events()` flow의 collect가 끝난 이후에 찍히는 것을 확인할 수 있습니다.

```kotlin
// Imitate a flow of events
fun events(): Flow<Int> = (1..3).asFlow().onEach { delay(100) }

fun main() = runBlocking<Unit> {
    events()
        .onEach { event -> println("Event: $event") }
        .collect() // <--- Collecting the flow waits
    println("Done")
}
```

```kotlin
Event: 1
Event: 2
Event: 3
Done
```

그러나 위의 예시와 같이 사용하면 찜찜한 점이 있습니다. 아마도 대부분의 상황에서는 비동기 적으로 발생하는 event를 계속해서 기다리고만 있는 게 아니라, 다른 코드들이 수행되는 도중에 event가 발생하는 그 순간에만 onEach 코드 블럭을 수행하고 싶을겁니다. `launcIn` terminal 연산자를 사용하면 그렇게 동작시킬 수 있습니다. `collect` 대신 `launchIn` 을 사용하면 flow collection은 다른 코루틴에서 동작하기 때문에 다른 코드 블럭들과 함께 동작하게 됩니다.

```kotlin
fun main() = runBlocking<Unit> {
    events()
        .onEach { event -> println("Event: $event") }
        .launchIn(this) // <--- Launching the flow in a separate coroutine
    println("Done")
}
```

```kotlin
Done
Event: 1
Event: 2
Event: 3
```

`launchIn` 를 사용할 땐 파라미터로 Collection을 수행할 코루틴의 Scope를 넣어주어야 합니다. 위의 예시에는 `runBlocking`의 coroutineScope를 넣어 주었기 때문에, `events()` flow가 끝나기 전까지는 `runBlocking`이 끝나지 않게 됩니다.

실제 애플리케이션에서 CoroutineScope는 특정 lifetime을 갖는 entity로 부터 생깁니다. entity가 lifetime을 다하면 해당 CoroutineScope이 cancel되며 scope에 속해있던 flow 역시 cancel됩니다. 이런점에서 보면 `onEach { ... }.launchIn(scope)` 구문은 `addEventListener` 와 비슷해 보입니다. 그러나 CoroutineScope의 종료시 함께 종료되기 때문에 `removeEventListener` 같은 이벤트 제거 함수가 필요없다는 차이점이 있습니다.

`lanchIn`은 `job` 을 반환합니다. 반환되는 job을 통해서 해당 scope 전체를 취소하지 않고도 flow만 cancel 시킬 수 있고, `join` 함수를 사용하여 원하는 시점에 대기할 수 도 있습니다.
