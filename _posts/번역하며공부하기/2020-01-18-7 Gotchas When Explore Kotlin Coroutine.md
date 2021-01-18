---
layout: post
title: 7 Gotchas When Explore Kotlin Coroutine(번역)
category: 번역하며 공부하기
tags: [Compose, Animation]
comments: true
---

원본 - [7 Gotchas When Explore Kotlin Coroutine](https://medium.com/mobile-app-development-publication/7-gotchas-when-explore-kotlin-coroutine-64b78f005150)

저는 [Kotlin Coroutine Ecosystem](https://medium.com/mobile-app-development-publication/kotlin-coroutine-scope-context-and-job-made-simple-5adf89fcfe94)을 더 파고들기 위해 연말 휴가를 보냈습니다. 그러면서 굉장히 많은 행동들이 예상 외로 동작한다는 것에 놀랐습니다. 그 중 몇 가지는 문서화 되어 있지만, 일부는 그렇지 않았습니다(적어도 아직 제가 찾지 못했습니다).

올바른 행동이 무엇인지, 왜 그렇게 되는지 알기 위해 StackOverflow에 질문을 해보기도 했고, 머리를 쥐어 뜯기도 했고, 정답을 얻기위한 디버깅을 어떻게 해야하는지 아이디어를 얻기 위해 낮잠을 자보기도 했습니다.

모든 분들에게 도움이 되고자 여기에 공유하려고 합니다.

> Note: 이 글은 Coroutine의 기본적인 지식을 바탕으로 합니다. 그렇지 않다면 아래의 문서들을 참고해보세요.

1. [Differentiating Thread and Coroutine (launch & runBlocking) in Kotlin](https://medium.com/mobile-app-development-publication/differentiating-thread-and-coroutine-launch-runblocking-in-kotlin-28219506c002)
2. [Understanding suspend function of Kotlin Coroutines](https://medium.com/mobile-app-development-publication/understanding-suspend-function-of-coroutines-de26b070c5ed)
3. [Kotlin Coroutine Scope, Context and Job](https://medium.com/mobile-app-development-publication/kotlin-coroutine-scope-context-and-job-made-simple-5adf89fcfe94)

## 1. `runBlocking` 은 App을 중단시킬 수 있습니다.

별다른 기능 없는 앱에서 아래의 코드를 실행시켜보세요.

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)

    runBlocking(Dispatchers.Main) {
        Log.d("Track", "${Thread.currentThread()}")
        Log.d("Track", "$coroutineContext")
    }
}
```

이 코드는 App을 멈추게 할겁니다. `onCreate` 함수가 완료될 수 없어요. 대신, 아래의 코드를 돌린다면 별다른 이상 없이 앱이 동작할 겁니다. (`Dispatchar.Main` 을 지우세요)

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)

    runBlocking {
        Log.d("Track", "${Thread.currentThread()}")
        Log.d("Track", "$coroutineContext")
    }
}
```

이것이 의미하는 바는, main thread 안에서의 `runBlocking` 은 main thread 안에서의 `runBlockin(Dispatchers.Main)`과 같지 않다는 것입니다.

## 2. main thread안의 `runBlocking`은 main thread 안의 `runBlocking(Dispatchers.Main)` 과 다릅니다

처음에는 이 사실에 혼란스러웠습니다.

저는 main thread에서 `runBlocking` 을 사용하면 당연히 `Dispatchers.Main` 을 사용할 거라 생각했었거든요. 그러나 이 생각은 명백히 틀렸더라구요.

이 사실을 증명해보려고 test코드를 작성했습니다.

```kotlin
@Test
fun running() {
    runBlocking {
        println(Thread.currentThread())
        println(coroutineContext)
    }
}
```

결과는 아래와 같습니다.

```kotlin
Thread[main @coroutine#1,5,main]
[CoroutineId(1), "coroutine#1":BlockingCoroutine{Active}@436a4e4b, BlockingEventLoop@f2f2cc1]
```

그런데 아래와 같이 코드를 돌리면,

```kotlin
@Test
fun running() {
    runBlocking(Dispatchers.Main) {
        println(Thread.currentThread())
        println(coroutineContext)
    }
}
```

앱 Crash가 발생합니다.

```kotlin
Java.lang.IllegalStateException: Module with the Main dispatcher had failed to initialize
```

이를 해결하기 위해 `[Dispatchers.Main Delegate](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-test/)` 설정이 필요합니다. 설정을 해준 다음 코드를 다시 돌리면 아래와 같은 결과가 나옵니다.

```kotlin
Thread[Test Main @coroutine#2,5,main]
[CoroutineId(2), "coroutine#2":BlockingCoroutine{Active}@20f637a1, Dispatchers.Main]
```

모두 `runBlocking`을 사용했지만 쓰레드 이름이 다른게 보이시나요? 하나는 `Dispatcher.Main` 이고, 하나는 `BlockingEventLoop@f2f2cc1` 입니다.

더 자세한 정보를 알고 싶으시면 [이 StackOverflow](https://stackoverflow.com/questions/65447319/providing-dispatches-main-for-runblocking-hang-the-android-app-why)를 참고하세요.

## 3. 안드로이드는 Thread.currentThread()로 coroutine id name을 print할 수 없습니다

디버깅을 위해 우리는 `CoroutineName` Context를 사용하여 코루틴에 이름을 할당할 수 있습니다. 그리고 `Thread.currentThread()` 를 로깅함으로써 이름을 볼 수 있습니다.

하지만 안드로이드에서 `Log`를 사용하여 로깅을 하면 이름을 출력하지 않습니다.

아래의 코드를 실행해보세요.

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)
    runBlocking(CoroutineName("My Coroutine")) {
        Log.d("Track", "${Thread.currentThread()}")
    }
}
```

이 코드는 단지 아래와 같은 결과를 나타낼 뿐입니다.

```kotlin
Thread[main,5,main]
```

thread 이름, 우선순위, thread group 정보입니다.

만약 테스트 환경에서 코드를 돌려보면 어떨까요?

```kotlin
@Test
fun running() {
    runBlocking(CoroutineName("My Coroutine")) {
        println("${Thread.currentThread()}")
    }
}
```

결과는 아래와 같습니다.

```kotlin
Thread[main @My Coroutine#1,5,main]
```

`@My Coroutine#1` 는 코루틴 이름과 ID입니다.

### 결론

안드로이드에서 코루틴 이름을 출력하고 싶다면, `coroutineContext` 를 출력하는 게 하나의 방법이 될 수 있습니다.

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)
    runBlocking(CoroutineName("My Coroutine")) {
        Log.d("Track", "${Thread.currentThread()}")
        Log.d("Track", "$coroutineContext")
    }
}
```

위 코드의 결과는 아래와 같습니다.

```kotlin
Thread[main,5,main]
[CoroutineName(**My Coroutine**), BlockingCoroutine{Active}@c3e0260, BlockingEventLoop@1607319]
```

다른 방법은 `kotlinx.coroutines.debug` 를 설정하는 것입니다. Appication에서 설정하시면 됩니다.

```kotlin
System.setProperty("kotlinx.coroutines.debug", "on" )
```

설정하시면 아래와 같이 출력할 수 있습니다.

```kotlin
Thread[main @My Coroutine#1,5,main]
[CoroutineName(My Coroutine), CoroutineId(1), "My Coroutine#1":BlockingCoroutine{Active}@c3e0260, BlockingEventLoop@1607319]
```

## 4. 코루틴 연산이 언제든지 즉각 취소될 수는 없습니다

아래의 코드를 실행해보세요.

```kotlin
runBlocking {
    println("Launching...")
    val job = launch(Dispatchers.IO) {
        repeat(2000) {
            repeat(2000) {
                println("Suspending...")
            }
        }
        println("Done...")
    }
    println("Launched...")
    delay(100)
    println("Canceling...")
    job.cancel()
    println("Canceled...")
}
```

아래와 같은 결과를 볼 수 있으실겁니다.

```kotlin
Track: Launching...
Track: Launched...
Track: Suspending...
Track: Suspending...
Track: Canceling...
Track: Suspending...
Track: Suspending...
Track: Canceled...
Track: Suspending...
Track: Suspending...
Track: Suspending...
Track: Suspending...
:
: (a lot more suspending)
:
Track: Done...
```

살펴보면,

1. blocking도 아니고
2. 다른 쓰레드도 아니고
3. 실행중에 cancel이 실행되었는데

위 코드는 0.1초 뒤에 취소되지 않습니다. 대신, `Track: Done...` 을 만날 때까지 긴 loop를 모두 실행하죠.

### 설명

코루틴에서 suspension과 cancellation는 오직 중단 함수(suspend-function)에서만 발생합니다(e.g. `yield()`, `delay()` ).

[Kotlin 공식 문서](https://kotlinlang.org/docs/reference/coroutines/cancellation-and-timeouts.html#cancellation-is-cooperative)에 나와있듯이 cancellation은 협동적, 협렵적입니다. 따라서 코루틴을 취소하기 위해선 협력이 필요합니다. 우리는 yield()를 사용하거나 `isActive`를 체크해야만 합니다.

이를 더 잘 이해하기 위해서는 아래의 자료를 보고 언제 코루틴이 일시중지되고, 종료되며, 시작되는지를 알아보시는걸 추천드립니다.

[Coroutine suspend function: when does it start, suspend or terminate?](https://medium.com/mobile-app-development-publication/coroutine-suspend-function-when-does-it-start-suspend-or-terminate-2762cabac54e)

코루틴 취소 메서드가 process를 즉각 취소할 수 없다는 점을 고려해볼 때, Newtork Fetching은 어떻게 다룰 수 있을까요? 아래의 자료에서 그 방법을 확인하실 수 있습니다.

[Network Fetch with Kotlin Coroutine](https://medium.com/mobile-app-development-publication/network-fetch-with-kotlin-coroutine-3eee5593da1b)

## 5. delay가 없이는 Android coroutine이 작업을 완료할 수 없는 이유

아래의 코드를 살펴봅시다.

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)
    runBlocking {
        launch {
            repeat(5) {
                Log.d("Track", "First, current thread")
                delay(1)
            }
        }
        launch {
            repeat(5) {
                Log.d("Track", "Second, current thread")
                delay(1)
            }
        }
    }
}
```

결과는 아래와 같은데, 서로 작업을 번갈아가며 5번씩 실행합니다.

```kotlin
Track: First, current thread
Track: Second, current thread
Track: First, current threa
Track: Second, current thread
Track: First, current thread
Track: Second, current thread
Track: First, current thread
Track: Second, current thread
Track: First, current thread
Track: Second, current thread
```

그러나 안드로이드에서 `dealy(1)` 을 제거하고 코드를 돌려보면

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)
    runBlocking {
        launch {
            repeat(5) {
                Log.d("Track", "First, current thread")
                // delay(1)
            }
        }
        launch {
            repeat(5) {
                Log.d("Track", "Second, current thread")
                // delay(1)
            }
        }
    }
}
```

첫 번째, 더이상 서로 번갈아가며 실행하지 않습니다. 이 현상은 함수가 중단되게끔 하는 `yield` 나 `delay` 가 없기 때문이라고 생각할 수 있겠네요.

두 번째, 겨우 두번씩 밖에 실행하지 않고 다른 코루틴으로 실행이 넘어갑니다. 이건 좀 이상하지 않은가요? 왜 이런 일이 일어날까요?

```kotlin
Track: First, current thread
Track: First, current thread
Track: Second, current thread
Track: Second, current thread
```

### 설명

결론 부터 말씀드리면 코루틴 이슈가 아니라 안드로이드 이슈입니다.

안드로이드는 같은 형태의 로그가 두 번 넘게 찍히면, 세번째 로그 부터는 자동으로 제거합니다.

## 6. 취소된 코루틴 Scope를 재사용할 수 있을까요?

아래 코드를 실행해봅시다.

```kotlin
@Test
fun testingLaunch() {
    val scope = MainScope()
    runBlocking {
        scope.cancel()
        scope.launch {
            try {
                println("Start Launch 2")
                delay(200)
                println("End Launch 2")
            } catch (e: CancellationException) {
                println("Cancellation Exception")
            }
        }.join()
    println("Finished")
    }
}
```

`scope.launch` 가 더이상 작동하지 않는 것을 확인할 수 있을 겁니다.

비슷하게 아래 코드를 볼게요.

```kotlin
@Test
fun testingAsync() {
    val scope = MainScope()
    runBlocking {
        scope.cancel()
        val defer = scope.async {
            try {
                println("Start Launch 2")
                delay(200)
                println("End Launch 2")
            } catch (e: CancellationException) {
                println("Cancellation Exception")
            }
        }
        defer.await()
        println("Finished")
    }
}
```

제대로 동작하지 않을 뿐만 아니라, `defer.await()` 에서 crash가 나는 것을 볼 수 있습니다.

```kotlin
kotlinx.coroutines.JobCancellationException: Job was cancelled
; job=SupervisorJobImpl{Cancelled}@39529185
```

`scope.cancel()` 를 제거하면 제대로 동작합니다.

### scope가 취소된 이후에는 scope를 재사용할 수 없습니다.

제 경험상으로, scope가 취소되면 더이상 `launch` 할 수 없습니다. 그것을 재설정 하는 방법도 없습니다.

[StackOverflow](https://stackoverflow.com/questions/65529095/after-a-coroutine-scope-is-cancel-can-it-still-be-used-again)에 이에 대한 내용을 작성했으니, 여러 의견을 댓글에 달아주세요.

이를 해결하기 위한 단 하나의 방법은, scope가 취소되고 나면 그저 새로운 scope를 만드는 것 뿐입니다.

## 7. 기본 코루틴 scope는 예외 처리 중에 다시 launch 할 수 없습니다.

코루틴에서는 `[CoroutineExceptionHandler](https://kotlinlang.org/docs/reference/coroutines/exception-handling.html#coroutineexceptionhandler)` 를 사용하여 간단하게 예외를 캡쳐할 수 있습니다.

```kotlin
private var coroutineScope: CoroutineScope? = null
private val errorHandler = CoroutineExceptionHandler {
    context, error ->
    println("Launch Exception ${Thread.currentThread()}")
    coroutineScope?.launch(Dispatchers.Main) {
        println("Launch Exception Result ${Thread.currentThread()}")
    }
}

@Test
fun testData() {
    runBlocking {
        coroutineScope = CoroutineScope(Dispatcher.IO)
        coroutineScope?.launch(errorHandler) {
           println("Launch Fetch Started ${Thread.currentThread()}")
           throw IllegalStateException("error")
        }?.join()
    }
}
```

위 코드는 `CoroutineExceptionHandler` 에서 예외를 포착할 수 있는지를 확인하기 위해 일부러 예외를 발생시킵니다. 결과는 아래와 같습니다.

```kotlin
Launch Fetch Started Thread[DefaultDispatcher-worker-1 @coroutine#2,5,main]
Launch Exception Thread[DefaultDispatcher-worker-1 @coroutine#2,5,main]
```

예외는 `CoroutineExceptionHandler` 에 의해 포착되었습니다만, 아래 코드는 실행되지 않았습니다.

```kotlin
coroutineScope?.launch(Dispatchers.Main) {
    println("Launch Exception Result ${Thread.currentThread()}")
}
```

이 코드가 실행되지 않은 이유는, 자식 코루틴의 exception이 부모에게 전파되어 부모 scope역시 새로운 launch를 할 수 없는 상황이 되었기 때문입니다. 자식은 자식인데 부모까지 망가뜨리니 뭔가 별로네요.

### 이를 다루려면 SupervisorJob() CoroutineScope가 필요합니다.

명백하게 기본 `CoroutineScope(Dispatcher.IO)` 은 자식 코루틴이 에러를 내뿜으면 제대로 동작하지 않습니다.

자식 코루틴의 예외를 부모와 떼어내기 위해서는 `CoroutineScope` 의 `SupervisorJob()` 이 필요합니다. 이는 [SupervisorJob](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-supervisor-job.html) 문서에 언급되어있습니다.

따라서 아래와 같이 코드를 변경하면 올바르게 작동합니다.

```kotlin
private var coroutineScope: CoroutineScope? = null
private val errorHandler = CoroutineExceptionHandler {
    context, error ->
    println("Launch Exception ${Thread.currentThread()}")
    coroutineScope?.launch(Dispatchers.Main) {
        println("Launch Exception Result ${Thread.currentThread()}")
    }
}

@Test
fun testData() {
    runBlocking {
        coroutineScope = CoroutineScope(
            SupervisorJob() + Dispatcher.IO)
        coroutineScope?.launch(errorHandler) {
           println("Launch Fetch Started ${Thread.currentThread()}")
           throw IllegalStateException("error")
        }?.join()
    }
}
```

결과는 아래와 같습니다.

```kotlin
Launch Fetch Started Thread[DefaultDispatcher-worker-1 @coroutine#2,5,main]
Launch Exception Thread[DefaultDispatcher-worker-1 @coroutine#2,5,main]
Launch Exception Result Thread[Test Main @coroutine#3,5,main]
```

이 글이 코루틴을 학습하는데 많은 도움이 되었으면 좋겠습니다. 감사합니다.
