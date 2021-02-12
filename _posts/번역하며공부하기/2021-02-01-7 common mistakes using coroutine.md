---
layout: post
title: 7 common mistakes you might be making when using Kotlin Coroutines(번역)
category: 번역하며 공부하기
tags: [Coroutines]
comments: true
---

원본 - [7 common mistakes you might be making when using Kotlin Coroutines](https://www.lukaslechner.com/7-common-mistakes-you-might-be-making-when-using-kotlin-coroutines/)

개인적으로 코틀린 코루틴은 비동기, 동시성 코드 작성을 매우 쉽게 해준다고 생각합니다. 하지만 코루틴을 사용하는 많은 개발자들이 흔히들 하는 실수 몇 가지들이 있더라구요.

## 일반적인 실수 #1: 코루틴을 launch할 때 새로운 job 객체를 생성한다.

때때로 코루틴을 취소하는 등, 코루틴을 다루기 위해 `job`이 필요합니다. `launch`와 `async` 코루틴 빌더 모두 `job`을 파라미터로 받기 때문에, 새로운 `job` 객체를 생성하고 `launch{}`와 같은 코루틴 빌더의 파라미터로 넘기는 것을 생각하기 쉽습니다. 이렇게 할 경우 `job`에 대한 레퍼런스를 가지게 되고, 추후에 `.cancel()`같은 메서드를 호출할 수 있겠죠.

```kotlin
fun main() = runBlocking {

    val coroutineJob = Job()
    launch(coroutineJob) {
        println("performing some work in Coroutine")
        delay(100)
    }.invokeOnCompletion { throwable ->
        if (throwable is CancellationException) {
            println("Coroutine was cancelled")
        }
    }

    // cancel job while Coroutine performs work
    delay(50)
    coroutineJob.cancel()
}
```

코드는 문제없어 보입니다. 코드를 돌려보면 코루틴은 잘 취소되었음을 알수 있습니다.

```kotlin
>_

performing some work in Coroutine
Coroutine was cancelled

Process finished with exit code 0
```

그러나 이번엔, `CoroutineScope`안에서 코루틴을 실행해보고, Coroutine의 `Job`이 아닌 이 `scope`를 취소시켜 볼게요.

```kotlin
fun main() = runBlocking {

    val scopeJob = Job()
    val scope = CoroutineScope(scopeJob)

    val coroutineJob = Job()
    scope.launch(coroutineJob) {
        println("performing some work in Coroutine")
        delay(100)
    }.invokeOnCompletion { throwable ->
        if (throwable is CancellationException) {
            println("Coroutine was cancelled")
        }
    }

    // cancel scope while Coroutine performs work
    delay(50)
    scope.cancel()
}
```

scope가 취소될 경우 scope안의 모든 코루틴은 취소되어야만 합니다. 하지만, 위 코드를 실행시켜보면 그렇게 동작하지 않아요.

```kotlin
>_

performing some work in Coroutine

Process finished with exit code 0
```

“Coroutine was cancelled”는 로그에 찍히지 않습니다.

왜 그럴까요?

코루틴은 비동기, 동시성 코드를 안전하게 제공하기 위해 "구조적 동시성(Structured Concurrency)"라고 불리는 특징을 가집니다. "구조적 동시성" 메커니즘 중 하나는 scope가 취소되면 `CoroutineScope`의 모든 코루틴이 취소된다는 것입니다. 이 메커니즘이 제대로 동작하기 위해서는 Scope의 Job과 Coroutine의 Job 사이의 계층이 잘 형성되어있어야 합니다. 아래 이미지 처럼요.

![image](https://user-images.githubusercontent.com/18481078/106439391-f7dbbd80-64ba-11eb-89a6-4f3b350e3494.png)

그러나 아까 본 예시에는 예상치 못한 일이 일어납니다. 별개의 `Job` 객체를 `launch()`빌더에 넘김으로써, 이를 해당 코루틴과 무관한 `Job`으로 정의하게 된것입니다. 대신, 이는 새로운 코루틴의 parent job이 됩니다. parent job은 `Coroutine Scope`의 `Job`이 아니며 단지 새롭게 객체화된 `Job` 객체일 뿐입니다.

따라서, 이 코루틴의 `job`은 더이상 `CoroutineScope`의 `job`과 연관되지 않습니다.

![image](https://user-images.githubusercontent.com/18481078/106440641-6b31ff00-64bc-11eb-9163-38f514f92c40.png)

우리는 구조적 동시성을 무너뜨렸고 그 결과 scope를 취소하더라도 더 이상 코루틴은 취소되지 않게 되는 것이죠.

이 문제의 해결법은 간단히 `launch{}`가 반환하는 `job`을 사용하여 코루틴을 제어하는 것입니다.

```kotlin
fun main() = runBlocking {
    val scopeJob = Job()
    val scope = CoroutineScope(scopeJob)

    val coroutineJob = scope.launch {
        println("performing some work in Coroutine")
        delay(100)
    }.invokeOnCompletion { throwable ->
        if (throwable is CancellationException) {
            println("Coroutine was cancelled")
        }
    }

    // cancel while coroutine performs work
    delay(50)
    scope.cancel()
}
```

이렇게 하면, Scope를 취소했을 때 Scope내의 모든 코루틴은 잘 취소됩니다.

```kotlin
>_

performing some work in Coroutine
Coroutine was cancelled

Process finished with exit code 0
```

## 일반적인 실수 #2: 잘못된 SuperviserJob 사용

때때로 아래와 같은 이유 때문에 `SupervisorJob`을 사용하고 싶을 겁니다.

1. 예외(exception)가 job 계층 전반에 퍼지는 것을 막기 위해
2. 코루틴중 하나가 실패할 때 형제(sibling) 레벨의 코루틴들의 취소를 막기 위해

`launch{}`와 `async{}` 코루틴 빌더는 Job을 파라미터로 받기 때문에, SupervisorJob을 이 코루틴 빌더의 파라미터에 넘기자고 생각할 수 있습니다.

```kotlin
launch(SupervisorJob()){
    // Coroutine Body
}
```

그러나, 실수#1과 동일하게, 구조적 동시성의 취소(cancellation) 메커즘을 망가뜨리는 행위입니다. 이는 `supervisorScope{}` scoping 함수를 사용하여 해결할 수 있습니다.

```kotlin
supervisorScope {
    launch {
        // Coroutine Body
    }
}
```

## 일반적인 실수 #3: cancellation을 지원하지 않음

suspend 함수 내부에서 factorial number 계산처럼 무거운 작업을 수행하고 싶다고 가정해봅시다.

```kotlin
// factorial of n (n!) = 1 * 2 * 3 * 4 * ... * n
suspend fun calculateFactorialOf(number: Int): BigInteger =
    withContext(Dispatchers.Default) {
        var factorial = BigInteger.ONE
        for (i in 1..number) {
            factorial = factorial.multiply(BigInteger.valueOf(i.toLong()))
        }
        factorial
    }
```

이 suspend 함수는 "협력적 취소(cancellation)"를 지원하지 않는 문제가 있습니다. 이는 실행된 코루틴이 조기에 취소되더라도, 계산이 끝나기 전에는 절대 취소되지 않는다는 것을 의미합니다. 이런 현상을 피하기 위해서 주기적으로 아래 나열된 것들을 잘 사용해야 합니다.

- [ensureActive()](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/ensure-active.html)
- [isActive()](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/is-active.html)
- [yield()](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/yield.html)

아래에는 [ensureActive()](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/ensure-active.html) 를 사용하여 cancellation을 지원하는 방법이 있습니다.

```kotlin
suspend fun calculateFactorialOf(number: Int): BigInteger =
    withContext(Dispatchers.Default) {
        var factorial = BigInteger.ONE
        for (i in 1..number) {
            ensureActive()
            factorial = factorial.multiply(BigInteger.valueOf(i.toLong()))
        }
        factorial
    }
```

코틀린 표준 라이브러리의 suspend 함수들(e.g 'delay()')은 모두 cancellation에 협력적입니다. 다만 직접 만든 suspend 함수라면 절대로 cancellation을 잊어선 안됩니다.

## 일반적인 실수 #4: 네트워크 요청이나 database 쿼리시 dispatcher를 변경한다.

이건 사실 '실수'라고 하긴 그렇지만, 분명히 코드를 좀 더 읽기 어렵게하고, 아마도 효율성을 아주 약간 떨어뜨리는 일입니다. 몇몇 개발자들은 네트워크 요청을 위한 Retrofit, 혹은 database연산을 위한 Room을 사용하는 `suspend` function 호출 전에 백그라운드 dispatcher로 바꿔야 한다고 생각합니다.

사실 이는 불필요한 작업입니다. 모든 suspend 함수는 'main safe' 해야한다는 규칙이 있고, 다행히도 Retrofit과 Room은 이를 잘 지키고 있기 때문입니다. 더 자세한 내용은 [여기](https://www.lukaslechner.com/do-i-need-to-call-suspend-functions-of-retrofit-and-room-on-a-background-thread/)에 있습니다.

## 일반적인 실수 #5: try/catch를 사용한 예외 처리

코루틴 예외를 다루기란 힘든 일입니다. 저 역시 이를 이해하기위해 많은 시간을 할애했으며, 다른 개발자들에게 설명하기위해 [글](https://www.lukaslechner.com/why-exception-handling-with-kotlin-coroutines-is-so-hard-and-how-to-successfully-master-it/) 작성과 [발표](https://www.droidcon.com/media-detail?video=481189746)를 했습니다. 심지어는 이를 요약하여 [Cheat Sheet](https://www.lukaslechner.com/coroutines-exception-handling-cheat-sheet/)을 만들기도 했죠.

코루틴 예외처리에는 정말 직관적이지 않은 것 하나가 있는데, 그것은 예외로 인해 실패한 코루틴을 `try/catch`로 잡아낼 수 없다는 것입니다.

```kotlin
fun main() = runBlocking<Unit> {
    try {
        launch {
            throw Exception()
        }
    } catch (exception: Exception) {
        println("Handled $exception")
    }
}
```

코드를 돌려보시면 예외처리가 되지 않아 충돌이 납니다.

```kotlin
>_

Exception in thread "main" java.lang.Exception

Process finished with exit code 1
```

코틀린 코루틴은 비동기 코드를 기존 코딩 구조처럼 사용할 수 있도록 보장합니다. 하지만 이 경우에는 사실이 아닌데요, 많은 개발자의 예상과는 달리 전통적인 `try-catch` 블럭으로 예외를 다룰 수 없습니다. 예외 처리를 하고 싶다면 코루틴 내부에서 직접 `try-catch`를 사용하던지 혹은 `CoroutineExceptionHandler`을 사용해야 합니다.

더 자세한 이야기는 앞서 언급한 이 [포스팅을](https://www.lukaslechner.com/why-exception-handling-with-kotlin-coroutines-is-so-hard-and-how-to-successfully-master-it/) 참고해주세요.

## 일반적인 실수 #6: 자식 코루틴에 CoroutineExceptionHandler 삽입

짧고 간단하게 말하겠습니다. 자식 코루틴을 통해 `CoroutineExceptionHandler` 를 삽입하는 것은 아무런 효과도 없습니다. 예외 처리는 부모 코루틴에게 전파되기 때문입니다. 따라서 `CoroutineScope` 에 삽입하던지 혹은 root나 부모 코루틴에 삽입해야만 합니다.

## 일반적인 실수 #7: CancellationExceptions 캐치

코루틴이 취소될 때, 현재 실행중인 suspend 함수는 코루틴 내에서 `CancellationException`를 던집니다. 이는 보통 코루틴을 "예외적으로" 완료시키는 행위이며 실행중인 코루틴은 즉각 종료됩니다.

```kotlin
fun main() = runBlocking {

    val job = launch {
        println("Performing network request in Coroutine")
        delay(1000)
        println("Coroutine still running ... ")
    }

    delay(500)
    job.cancel()
}
```

0.5초가 지나고 나면 `delay()` `suspend` 함수는 `CancellationException`를 던지고, 코루틴은 예외적으로 완료되며 실행도 종료됩니다.

```kotlin
>_

Performing network request in Coroutine

Process finished with exit code 0
```

이제는 `delay()`가 네트워크 요청을 나타낸다고 생각하고, 네트워크 처리 예외를 처리하기 위해 `try-catch` 블럭으로 감싸서 모든 예외를 캐치해봅시다.

```kotlin
fun main() = runBlocking {

    val job = launch {
        try {
            println("Performing network request in Coroutine")
            delay(1000)
        } catch (e: Exception) {
            println("Handled exception in Coroutine")
        }

        println("Coroutine still running ... ")
    }

    delay(500)
    job.cancel()
}
```

`catch` 구문이 네트워크 실패 HttpExceptions를 캐치해낼 뿐만 아니라 CancellationExceptions까지도 잡아버립니다! 따라서 코루틴은 "예외적으로 완료"되지 않고 계속해도 동작하게됩니다.

```kotlin
>_

Performing network request in Coroutine
Handled exception in Coroutine
Coroutine still running ...

Process finished with exit code 0
```

이는 디바이스 자원을 낭비하며 때에 따라서는 앱 충돌로 이어질 수 있습니다.

이 문제를 해결하기 위해서는 아래처럼 ``HttpExceptions` 만 캐치하거나,

```kotlin
fun main() = runBlocking {

    val job = launch {
        try {
            println("Performing network request in Coroutine")
            delay(1000)
        } catch (e: HttpException) {
            println("Handled exception in Coroutine")
        }

        println("Coroutine still running ... ")
    }

    delay(500)
    job.cancel()
}
```

`CancellationExceptions`를 다시 던지면 됩니다.

```kotlin
fun main() = runBlocking {

    val job = launch {
        try {
            println("Performing network request in Coroutine")
            delay(1000)
        } catch (e: Exception) {
            if (e is CancellationException) {
                throw e
            }
            println("Handled exception in Coroutine")
        }

        println("Coroutine still running ... ")
    }

    delay(500)
    job.cancel()
}
```

이렇게 해서 코틀린 코루틴을 사용할 때 흔히 하는 일반적인 실수 7개를 살펴보았습니다. 만약 더 많은 실수들을 알고 계시다면 댓글로 알려주세요!

그리고 다른 개발자분들이 같은 실수를 반복하지 않도록 이 포스팅을 많이 공유해주세요. 감사합니다!
