---
layout: post
title: Flow Exception
category: Kotlin
tags: [coroutine_flow, flow_exception]
comments: true
---

이 포스팅은 [공식문서](https://kotlinlang.org/docs/reference/coroutines/flow.html#flow-exceptions)를 보며 공부한 내용입니다.

# Flow Exception

Flow emitter 또는 flow 내부 연산 코드가 exception을 던질 경우 collection은 그 exception과 함께 종료되어버립니다. 이런 exception을 다루는 몇 가지 방법을 살펴봅시다.

## Collector try and catch

collector 쪽에서 try/catch 블럭을 사용하여 excption을 간단히 처리할 수 있습니다.

```kotlin
fun foo(): Flow<Int> = flow {
    for (i in 1..3) {
        println("Emitting $i")
        emit(i) // emit next value
    }
}

fun main() = runBlocking<Unit> {
    try {
        foo().collect { value ->
            println(value)
            check(value <= 1) { "Collected $value" }
        }
    } catch (e: Throwable) {
        println("Caught $e")
    }
}
```

```
Emitting 1
1
Emitting 2
2
Caught java.lang.IllegalStateException: Collected 2
```

위 코드는 collect 연산자 코드에서 일어나는 exception을 잡아내고, exception이 발생한 이후의 값들은 emit되지 않음을 볼 수 있습니다. 물론 `flow {...}` 블럭에서 에러가 나더라도 collect 측의 `catch`가 잡아냅니다.

## Everything is caught

위에 보았던 예제는 exception이 emitter에서 일어나든, 중간 연산자에서 일어나든, collect와 같은 terminal 연산자에서 일어나든지 상관없이 모든 exception을 잡아냅니다. 예를 들어 아래 예제를 보면 emit되는 값이 `string`으로 mapping되는 과정에서 exception을 일으키고, 이를 성공적으로 잡아내며 collection을 멈춥니다.

```kotlin
fun foo(): Flow<String> =
    flow {
        for (i in 1..3) {
            println("Emitting $i")
            emit(i) // emit next value
        }
    }
    .map { value ->
        check(value <= 1) { "Crashed on $value" }
        "string $value"
    }

fun main() = runBlocking<Unit> {
    try {
        foo().collect { value -> println(value) }
    } catch (e: Throwable) {
        println("Caught $e")
    }
}
```

```
Emitting 1
string 1
Emitting 2
Caught java.lang.IllegalStateException: Crashed on 2
```

## Exception transparency

emit 하는 쪽에서 exception을 캡슐화 하고 싶으면 어떻게 할까요? 즉 collector 측에서는 예외처리를 하지 않고, emitter 측에서 에러를 감지하여 처리를 하려면 어떻게 해야 할까요?

Flows는 `exception`에 투명해야합니다. `flow {...}` 빌더를 `try/catch` 블럭 안에서 사용하여 값을 emit하는 것은 `exception`에 투명하지 못한 행위입니다. `exception`에 투명하다는 말은, emitter에서 발생한 에러는 다른 곳에서 발생한 에러와 섞이지 않으며 emitter가 발생한 `exception`이라는 것이 훤히 보여야 한다는 말로 이해할 수 있습니다. collector측에서 try/catch로 다 잡아낸다고 생각해보세요. emitter가 발생한 exception이든 collector가 발생한 exception이든 상관없이 퉁쳐서 에러를 처리하게 될 가능성이 큽니다.

emitter는 catch 연산자를 통하여 exception transparency를 유지할 수 있고 `exception` 처리를 캡슐화 할 수 있습니다. catch 연산자 안에서 예외를 분석하여 어떤 예외가 포착되었는지에 따라 다른 방식으로 대응할 수 있습니다.

- throw를 사용하여 `throw` 를 다시 던질 수 있습니다.
- `exception` 상황이지만 평소처럼 emit 할 수 있습니다.
- `exception` 을 무시하거나 단순히 로그를 찍거나 하는 등의 코드를 넣을 수 있습니다.

아래는 `exception`을 catch 하는곳에서 text를 emit하는 코드입니다.

```kotlin
foo()
    .catch { e -> emit("Caught $e") } // emit on exception
    .collect { value -> println(value) }
```

위의 코드와 같이 try/catch 없이도 예외처리가 가능합니다.

## transparent catch

`catch` 는 중간 연산자로써 오직 Upstream에서 발생한 exception만 처리할 수 있습니다(catch 아래의 연산자에서 발생한 exception은 처리하지 못합니다). 즉 `collect {...}` 블럭에서 일어난 예외는 catch의 downstream에서 일어난 예외임으로 catch가 처리할 수 없습니다.
`catch` 는 중간 연산자로써 오직 upstream에서 발생한 exception만 처리할 수 있습니다(catch 아래의 연산자에서 발생한 exception은 처리하지 못합니다). 즉 `collect {...}` 블럭에서 일어난 예외는 catch의 downstream에서 일어난 예외임으로 catch는 처리할 수 없습니다.

```kotlin
fun foo(): Flow<Int> = flow {
    for (i in 1..3) {
        println("Emitting $i")
        emit(i)
    }
}

fun main() = runBlocking<Unit> {
    foo()
        .catch { e -> println("Caught $e") } // does not catch downstream exceptions
        .collect { value ->
            check(value <= 1) { "Collected $value" }
            println(value)
        }
}
```

```
Emitting 1
1
Emitting 2
Exception in thread "main" java.lang.IllegalStateException: Collected 2
	at ....
	....
```

## Catching declaratively

catch연산자를 사용하면 선언적으로 exception을 처리할 수 있기 때문에 매력적으로 보입니다. 따라서 모든 에러를 catch를 사용하여 선언적으로 다루고 싶은 욕심이 생기기 마련입니다. 어떻게 가능할까요? collect 연산자에 있는 코드들을 `onEach` 연산자로 옮기고 `catch` 연산자 앞쪽에 배치하면 가능합니다. flow가 발행하는 값의 Collection은 인자 없는 `collect()` 를 호출하여 실행시킵니다.

```kotlin
foo()
    .onEach { value ->
        check(value <= 1) { "Collected $value" }
        println(value)
    }
    .catch { e -> println("Caught $e") }
    .collect()
```

```kotlin
Emitting 1
1
Emitting 2
Caught java.lang.IllegalStateException: Collected 2
```

위의 결과를 보면 try/catch를 사용하지 않았음에도 성공적으로 "Caught..."를 확인할 수 있습니다.

## 끄

읕
