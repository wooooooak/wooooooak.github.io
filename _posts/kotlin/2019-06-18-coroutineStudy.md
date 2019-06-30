---
layout: post
title: 코루틴을 이해하기 위한 발악 1편
category: Kotlin
tags: [coroutine, study]
comments: true
---

코루틴은 우리가 흔히 알고 있는 함수의 상위 개념이라고 볼 수 있다. 일반 함수의 경우 caller가 함수를 호출하면 호출당한 함수는 caller에게 어떤 값을 return하고 끝이난다. 그러나 코루틴은 suspend/resume도 가능하다. 즉, caller가 함수를 call하고, 함수가 caller에게 값을 return하면서 종료하는 것 뿐만 아니라 값을 return하지 않고 잠시 멈추었다가 필요할 때에 다시 이어서 resume(재개)할 수도 있다.

코루틴으로 메인쓰레드를 너무 오래 블락시키는 Long running task문제를 해결 할 수 있다. 안드로이드 플랫폼은 메인쓰레드에서 5초 이상 걸리는 긴 작업을 할 경우 앱을 죽여버린다. 그래서 network나 db접근같이 오래걸리는 작업은 모두 다른 스레드에서 작업하고, 그 결과를 받아 ui를 그려주는 것은 다시 Main 스레드로 돌아와서 작업해야한다. 기존에는 이런 작업을 콜백으로 처리했다.

```kotlin
class MyViewModel: ViewModel() {
    fun fetchDocs() {
        get("dev.android.com") { result ->
            show(result)
        }
    }
}
```

`get` 함수는 비록 메인 스레드에서 호출되었지만, 네트워크를 타고 데이터베이스에 접근하는 기능은 다른 스레드에서 해야만 한다. 그리고 `result`정보가 도착하면 콜백 함수는 메인스레드에서 동작해야 한다. 이런 비동기 작업을 코루틴을 이용해서 더 읽기 쉽고 작성하기 편하게 할 수 있다.

- 참고로 RxJava, RxKotlin 역시 이런 비동기 처리를 손쉽게 처리할 수 있도록 도와준다. 가끔 코루틴이 RxJava를 완전히 대체할 수 있냐고 묻는 사람들이 있는데, 섣불리 대답하기 어려운 질문이다. 비동기 처리 측면에서 본다면 거의 100% 대체할 수 있다고 볼 수 있다고 생각하지만, 만약 프로젝트를 Rx 프로그래밍으로 진행 중이라면 그 패러다임을 다루는 편의성까지 대체할 순 없기 때문이다.

## 공식 문서 보면서 발악하기

```kotlin
import kotlinx.coroutines.*

fun main() {
    GlobalScope.launch { // launch a new coroutine in background and continue
        delay(1000L) // non-blocking delay for 1 second (default time unit is ms)
        println("World!") // print after delay
    }
    println("Hello,") // main thread continues while coroutine is delayed
    Thread.sleep(2000L) // block main thread for 2 seconds to keep JVM alive
}
// Hello,
// World!
```

`delay`는 suspend함수다. suspend란 잠시 중단한다는 의미이고, 잠시 중단한다면 언젠가는 다시 resume된다는 뜻이다. 위 코드에서는 delay라는 suspend가 끝이나면 그때 caller가 resume시켜 아랫줄 코드를 실행시킨다.

delay라는 함수는 현재 실행중인 thread를 block시키진 않지만 코루틴은 일시 중지시킨다. thread입장에서는 non-blocking이다.

문서에서 **blocking**과 **non-blocking**이 자주 나오는데, 이것은 쓰레드 입장에서 봐야한다. 우선은 쓰레드를 멈춘다면 blocking이고, 쓰레드를 멈추지 않는다면 non-blocking이라고 이해하자.

위 코드에서 `GlobalScope`라는 것이 보인다. `launch`라는 코루틴 빌더는 늘 어떤 코루틴 스코프안에서 코루틴을 `launch`한다. 위에서는 새로운 코루틴을 `GlobalScope`에서 `launch`하도록 했다. 이 말은 Global이 의미하는 것 처럼, 새롭게 `launch`된 코루틴은 해당 어플리케이션 전체의 생명주기에 적용된다는 말이다.

### runBlocking

위 코드는 쓰레드를 중단시키지 않는 non-blocking `delay(...)` 함수와 쓰레드를 잠시 멈추는 blocking 함수 `Thread.sleep(...)` 를 같이 섞어 쓰고있다. 이렇게 섞어 쓰게되면 무엇이 블락킹 함수이고 무엇이 넌블러킹 함수인지 헷갈릴수 있다. runBlocking 코루틴 빌더를 사용해서 blocking을 조금 더 명확하게 명시해보자.

```kotlin
import kotlinx.coroutines.*

fun main() {
    GlobalScope.launch { // launch a new coroutine in background and continue
        delay(1000L)
        println("World!")
    }
    println("Hello,") // main thread continues here immediately
    runBlocking {     // but this expression blocks the main thread
        delay(2000L)  // ... while we delay for 2 seconds to keep JVM alive
    }
}
```

`Thread.sleep(2000L)` 이 부분이 `runBlocking{ ... }` 이렇게 바뀌었다. Blocking을 run(실행, 시작)한다는 뜻의 runBlocking. 이름만 보아도 꽤 명시적이다. **runBlocking은** 이름이 내포하듯이 **현재 쓰레드(여기선 main 쓰레드)를 블록킹 시키고 새로운 코루틴을 실행시킨다.**

[](https://www.notion.so/c119dc2524f140dda4e1ac10180bece6#af930233b4a94a388201c296d05f863e)

언제까지 블로킹 시킬까? runBlocking 블록 안에 있는 코드가 모두 실행을 끝마칠 때 까지 블록된다. `runBlocking { ... }` 안에 2초의 delay를 주었으므로 2초동안 메인쓰레드가 블록된다. 2초의 딜레이가 끝나면 main()함수는 종료된다. 메인쓰레드가 블록되어있는 2초 동안에, 이전에 launch했던 코루틴은 계속해서 동작하고있다.

한편 delay는 suspend 함수이기 때문에 코루틴이 아닌 일반 쓰레드에서는 사용이 불가능한데, `runBlocking {...}` 블락 안에 `delay()`가 사용가능한 것으로 보아 `runBlocking` 역시 새로운 코루틴을 생성하는 것으로 보인다. 동시에 자신이 속한 쓰레드를 블로킹 시키기도 한다.

위 코드를 한 번 더 진화시켜보자.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> { // start main coroutine
    GlobalScope.launch { // launch a new coroutine in background and continue
        delay(1000L)
        println("World!")
    }
    println("Hello,") // main coroutine continues here immediately
    delay(2000L)      // delaying for 2 seconds to keep JVM alive
}
```

runBlocking을 메인스레드 전체에 걸어줌으로써 시작부터 메인 쓰레드를 블락시키고 top-level 코루틴을 시작한다. 위에서 설명했듯이 runBlocking은 `runBlocking {...}` 블록 안에있는 모든 코루틴들이 완료될때 까지 자신이 속한 스레드를 종료시키지 않고 블락시킨다. 따라서 runBlocking에서 가장 오래 걸리는 작업인 delay(2초)가 끝날 때 까지 메인쓰레드는 죽지 않고 살아있다.

그런데 1초의 시간뒤에 "World!"라는 단어를 찍기위하여 2초를 기다리는 일은 별로 좋아보이지 않는다. 예를들어 1초의 시간이 어떠한 디비를 접속해서 데이터를 가져오는 비동기 처리 작업이라고 한다면, 그때 걸리는 시간이 무조건 1초가 걸린다고 가정할 수는 없으므로 2초라는 구체적인 시간동안 스레드를 죽이지 않는 건 좋지 못하다. 디비를 갔다가 오는 시간이 3초가 걸릴수도 있지 않은가. 따라서 우리는 데이터베이스를 갔다가 어떤 응답을 가져오면, 그 즉시 어떤 일을 처리하고 프로그램을 종료시킬 방법이 필요하다. Job을 통해 그런 일이 가능하다.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
//sampleStart
    val job = GlobalScope.launch { // launch a new coroutine and keep a reference to its Job
        delay(1000L)
        println("World!")
    }
    println("Hello,")
    job.join() // wait until child coroutine completes
//sampleEnd
}
```

위의 코드에는 1초의 딜레이 이후 "World!"가 찍히는 것을 보기위해 2초동안 프로그램을 종료시키지 않는 `delay(2000L)`라는 코드가 없다. 위 코드는 `GlobalScope.launch`로 생성한 코루틴이 제 기능을 다 완수하는 즉시 프로그램을 종료시킨다. job이라는 변수가 특정 코루틴의 레퍼런스를 가지고 있고, job.join()이 job이 끝나기를 계속 기다리기 때문이다. job이 끝나지 않으면 `runBlocking()`으로 생성한 코루틴은 끝나지 않는다.

모든 코루틴 빌더(`runBlocking {}`, `launch {}` 등등)는 빌더로 인해 생성되는 코드 블록 안에다가 CoroutineScope 객체를 추가한다. 위 코드에서는 runBlocking의 블록 안에서 GlobalScope로 코루틴을 만들어 launch했지만, GlobalScope를 사용하지 않고 runBlocking 이 만든 코루틴 스코프와 같은 스코프로 코루틴을 만들 수 있다(그냥 `launch { }`를 호출하면 바로 바깥의 스코프와 동일한 스코프에 생긴다). 게다가 바깥에 있는 코루틴은 안쪽에 있는 코루틴이 끝날때 까지 끝나지 않는다는 성질을 이용해서 코드를 더 깔끔하게 만들 수 있다.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking { // this: CoroutineScope
    launch { // launch a new coroutine in the scope of runBlocking
        delay(1000L)
        println("World!")
    }
    println("Hello,")
}
```

## suspend & resume

이쯤 되서 다시 suspend와 resume을 정리해보자

- suspend : 현재의 코루틴을 멈춘다(현재의 코루틴을!).
- resume : 멈춰있던 코루틴 부분을 다시 시작한다.

suspend와 resume은 콜백을 대체하기 위해 같이 쓰인다.

```kotlin
class MyViewModel: ViewModel() {
    fun fetchDocs() {
        get("dev.android.com") { result ->
            show(result)
        }
    }
}
```

이 함수에서 콜백을 제거하기 위해 코루틴을 사용해보자.

```kotlin
// Dispatchers.Main
suspend fun fetchDocs() {
    // Dispatchers.IO
    val result = get("developer.android.com")
    // Dispatchers.Main
    show(result)
}
// look at this in the next section
suspend fun get(url: String) = withContext(Dispatchers.IO){/*...*/}

```

suspend 함수(`get`)가 제 역할(network요청 처리)을 끝내면 메인쓰레드에 콜백으로 알려주는 것이 아니라, 그저 멈춰있던 suspended coroutine 부분을 시작하는 것이다.

코루틴은 스스로 `suspend(중단)` 할 수 있으며 `dispatcher`는 코루틴을 `resume`하는 방법을 알고 있다. 약간 어려운 말이긴 한데, [Coroutines on Android (part I): Getting the background](https://medium.com/androiddevelopers/coroutines-on-android-part-i-getting-the-background-3e0e54d20bb)라는 글에 애니메이션으로 잘 설명이 되어있으니 참조하자.

## 자주하는 오해

함수 앞에 `suspend`를 적어준다고 해서 그것이 함수를 백그라운드 스레드에서 실행시킨다는 뜻은 아니다. 코루틴은 메인 쓰레드 위에서 돈다. 메인스레드에서 하기에는 너무 오래 걸리는 작업을 하기 위해서는 코루틴을 `Default`나 `IO` dispatcher에서 관리되도록 해야한다. 코루틴이 메인 스레드 위에서 실행되더라도 꼭 dispatcher에 의해서 동작해야만 한다.

#### dispatcher

coroutine context는 어떤 쓰레드에서 해당 coroutine을 실행할지에 대한 dispatcher 정보를 담고 있다.

코루틴들이 어디서 실행되는지를 명시하기 위해 코틀린은 3가지 유형의 Dispatchers를 제공한다. 자세한 내용은 [CoroutineDispatcher](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-dispatcher/)을 클릭해서 확인하자.

결국 개발자가 선택한 Dispatcher에 따라서 실행되는 쓰레드가 달라진다.

## 마무리

코루틴을 제대로 공부하기로 마음먹었지만 쉽지만은 않은 것 같다. 조금씩 감은 잡히지만 긴가민가 한 내용들이 많다. 아직 제대로 알고 쓰는 글이 아니라 어수선한 글이 되었지만, 2편 3편을 쓰다보면 개념도, 글도 깔끔해지지 않을까 기대해본다 ㅠ.ㅠ

아직까지 코루틴을 공부하기엔 한글 자료가 별로없다. 대부분 영어 자료를 읽고 공부하는 중이라 시간도 꽤 걸리는 편이지만 확실히 영어로 눈을 돌리니 좋은 자료가 많다. 코루틴을 익숙해 질 때까지 공부하면서 이참에 영어에 대한 거부감도 더 줄여나가야겠다.
