---
layout: post
title: 코루틴 취소(Coroutine Cancellation)
category: Kotlin
tags: [coroutine_cancellation, coroutine, CancellationException]
comments: true
---

실행중인 코루틴을 취소하는 방법은 [공식문서](https://kotlinlang.org/docs/reference/coroutines/cancellation-and-timeouts.html#cancelling-coroutine-execution)에도 나와있듯 간단하다. job 객체의 `cancel()` 을 호출하면 취소된다고 한다.

**The launch function returns a Job that can be used to cancel the running coroutine:**

```kotlin
val job = launch {
    repeat(1000) { i ->
        println("job: I'm sleeping $i ...")
        delay(500L)
    }
}
delay(1300L) // delay a bit
println("main: I'm tired of waiting!")
job.cancel() // cancels the job
job.join() // waits for job's completion
println("main: Now I can quit.")

// reulst
job: I'm sleeping 0 ...
job: I'm sleeping 1 ...
job: I'm sleeping 2 ...
main: I'm tired of waiting!
main: Now I can quit.
```

코루틴 취소가 이렇게 간단해 보이지만 실제 프로젝트에서 사용하다보면 뜻대로 행동하지 않는다. 분명히 `cancel()`을 시켜주었지만 동작이 멈추지 않고 계속해서 실행되는 경우가 있다.

```kotlin
fun main() = runBlocking {
    val job = launch {
        while(true) {
            println("while")
        }
    }
    delay(1000L) // delay a bit
    println("main: I'm tired of waiting!")
    job.cancel() // cancels the job
    job.join() // waits for job's completion
    println("main: Now I can quit.")
}

// result
while
while
while
...
...
...
```

위 코드를 보면 분명 `cancel()` 해주는 코드가 있지만, `"while"`만 계속 찍히며 종료되지 않는다. 사실 위 코드가 왜 취소되지 않고 영원히 동작하는가를 파악하지 못했다는 건, 코루틴 `Cancellation`에 대한 이해 부족도 있지만 코드 플로우를 잘 따라가지 못한 것도 한 몫 한다.

`launch`로 실행한 코루틴은 메인 쓰레드에서 돌아간다. 코드가 실행되면 `launch`로 코루틴을 실행 시키고 곧바로 `delay`를 만나는데, `delay`는 `suspend`함수이기 때문에 다른 코루틴으로 제어권을 넘긴다. 이때 제어권을 넘겨받은 또다른 코루틴, 즉, 아까 메인 쓰레드에서 `launch`했던 `while`문 코드가 실행이 되는데, 이 `while`문은 조건이 항상 `true`라서 사실상 메인 쓰레드를 계속 해서 잡고 있는 것이다. `delay`에서 걸어준 1초가 끝이 났지만, `while`이 thread를 계속해서 사용 중임으로 `resume`되지 못한다. 따라서 앱이 종료되지 않음은 물론이고 `"main: I'm tired of waiting!"` 문장도 찍히지 않는 것이다.

위의 코드에서 `luanch`의 context를 `Dispatcher.Deafult` 바꿔주면 다르게 동작한다.

```kotlin
fun main() = runBlocking {
    val job = launch(Dispatchers.Default) {
        while(true) {}
    }
    delay(1000L) // delay a bit
    println("main: I'm tired of waiting!")
    job.cancel() // cancels the job
    job.join() // waits for job's completion
    println("main: Now I can quit.")
}

//result
main: I'm tired of waiting!
// 영원히 끝나지 않음.
...
```

`runBlocking` 내의 코드는 메인 쓰레드에서, `launch` 코루틴은 다른 쓰레드에서 돌아가기 때문에 백그라운드의 `while`문이 메인 쓰레드의 흐름을 방해하지 못한다. 따라서 `"main: I'm tired of waiting!"` 문장이 찍힌다. 곧바로 백그라운드 `job`을 `cancel`시키고, `job`이 종료될 때 까지 `join` 한다.

**일반적으로는 `job.cancel()` 을 호출 했으니 취소되어 곧바로 `job.join()`으로 넘어갈 것이라고 생각할 수 있지만 실제로 코드는 그렇게 동작하지 않는다.** 위 코드 역시 영원히 끝나지 않는다. 왜 일까?
[공식문서](<[https://kotlinlang.org/docs/reference/coroutines/cancellation-and-timeouts.html#cancellation-is-cooperative](https://kotlinlang.org/docs/reference/coroutines/cancellation-and-timeouts.html#cancellation-is-cooperative)>)에는 이렇게 나와있다.

**However, if a coroutine is working in a computation and does not check for cancellation, then it cannot be cancelled**

즉, long running loop 같은 작업을 실행 중인 코루틴이 있을 경우, 작업 중에 cancellaion을 체크하여 취소해주지 않는다면 취소되지 않는 다는 말이다. `cancel()`을 호출해도, 곧바로 코루틴이 완전히 종료되는게 아니라, loop내부에서 `isActive`가 `false`인지를 체크해서 명시적으로 종료 해야 한다는 것이다.

그렇다면 가장 처음에 보았던 공식 문서 예제는 어떻게 취소가 되었을까?

## suspend 함수가 Exception을 던진다.

```kotlin
val job = launch {
    repeat(1000) { i ->
        println("job: I'm sleeping $i ...")
        delay(500L)
    }
}
delay(1300L) // delay a bit
println("main: I'm tired of waiting!")
job.cancel() // cancels the job
job.join() // waits for job's completion
println("main: Now I can quit.")

// reulst
job: I'm sleeping 0 ...
job: I'm sleeping 1 ...
job: I'm sleeping 2 ...
main: I'm tired of waiting!
main: Now I can quit.
```

비밀은 `delay()` 에 있다. 정확하게 말하자면 `suspend` 함수가 내부적으로 `isActive`를 체크하여 `isActive`가 `false`일 때 `Exception`을 던지는 것이다.

[공식문서](<[https://kotlinlang.org/docs/reference/coroutines/cancellation-and-timeouts.html#cancellation-is-cooperative](https://kotlinlang.org/docs/reference/coroutines/cancellation-and-timeouts.html#cancellation-is-cooperative)>)에는 아래의 문장으로 설명되어 있다.

**All the suspending functions in kotlinx.coroutines are cancellable. They check for cancellation of coroutine and throw CancellationException when cancelled.**

실제로 delay()함수 내부를 보면 `suspendCancellableCoroutine` 로 구현되어 있음을 알 수 있다.

```kotlin
/**
 * If the [Job] of the current coroutine is cancelled or completed while this suspending function is waiting, this function
 * immediately resumes with [CancellationException].
 */
public suspend fun delay(timeMillis: Long) {
    if (timeMillis <= 0) return // don't delay
    return suspendCancellableCoroutine sc@ { cont: CancellableContinuation<Unit> ->
        cont.context.delay.scheduleResumeAfterDelay(timeMillis, cont)
    }
}
```

`suspendCancellableCoroutine` 의 내부 구현을 따라가다 보면 isActive를 체크하는 코드가 보인다.

```kotlin
...
if (resumeMode == MODE_CANCELLABLE) {
            val job = context[Job]
            if (job != null && !job.isActive) {
                val cause = job.getCancellationException()
                cancelResult(state, cause)
                throw recoverStackTrace(cause, this)
            }
        }
...
```

## CancellationException

cancel()의 타겟인 코루틴 내부에 suspend 함수가 있다면, 그 suspend 함수가 `CancellationException`을 throw한다. 그러나 `suspend`가 없다면, `CancellationException` 는 throw되지 않는다. 이런 경우에는 `isActive`를 체크하여 명시적으로 `CancellationException` 를 던질 수 있다.

## 참고자료

- [공식문서 - Cancellation and Timeouts](https://kotlinlang.org/docs/reference/coroutines/cancellation-and-timeouts.html#cancelling-coroutine-execution)
- [Cancelling Kotlin Coroutines](https://proandroiddev.com/cancelling-kotlin-coroutines-1030f03bf168)
