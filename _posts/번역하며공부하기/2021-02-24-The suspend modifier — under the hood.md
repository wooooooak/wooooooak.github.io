---
layout: post
title: The suspend modifier — under the hood(번역)
category: 번역하며 공부하기
tags: [Coroutine]
comments: true
---

원본 - [The suspend modifier — under the hood](https://manuelvivo.dev/suspend-modifier)

> 컴파일러는 어떻게 코루틴 실행을 일시중지하고 재개하도록 코드를 변경할까요?

Kotline coroutine의 suspend modifier는 안드로이드 개발자에게 자연스럽게 스며들었습니다. 그 내부가 어떻게 동작하는지 궁금하지 않으신가요? 컴파일러는 도대체 어떻게 코루틴을 일시 중지하고 다시 재개하도록 코드를 변경해주는걸까요?

이 원리를 알면 suspend 함수가 왜 자신의 일을 끝마치기 전까지는 return하지 않는지, 그리고 어떻게 쓰레드를 block하지 않고 일시 중지할 수 있는지를 이해하는데 도움이 됩니다.

> **요약;** 코틀린 컴파일러는 모든 suspend 함수에 대해서 코루틴 실행 관리를 위한 state machine을 생성해 줍니다.

Android에서 코루틴을 사용하는게 익숙치 않으신가요? 아래의 coroutine 코드랩을 살펴보세요.

- [Using coroutines in your Android app](https://codelabs.developers.google.com/codelabs/kotlin-coroutines/#0)
- [Advanced Coroutines with Kotlin Flow and Live Data](https://codelabs.developers.google.com/codelabs/advanced-kotlin-coroutines/#0)

비디오로 시청하는 것을 더 좋아하시면 아래 영상을 참고하세요.

[https://www.youtube.com/watch?time_continue=472&v=IQf-vtIC-Uc&feature=emb_title](https://www.youtube.com/watch?time_continue=472&v=IQf-vtIC-Uc&feature=emb_title)

## Coroutines 101

코루틴은 비동기 연산을 간단하게 해줍니다. [문서](https://developer.android.com/kotlin/coroutines)에 설명된것 처럼, 비동기 처리가 아니었다면 메인 쓰레드를 block하고 결국 앱을 멈춰버리는 작업을 위해 코루틴을 사용합니다.

코루틴은 가독성이 좋지 않은 콜백 기반 API들을 직관적으로 보이게끔 해줍니다. 예를들어, 아래의 콜백 기반 코드를 살펴봅시다.

```kotlin
// Simplified code that only considers the happy path
fun loginUser(userId: String, password: String, userResult: Callback<User>) {
  // Async callbacks
  userRemoteDataSource.logUserIn { user ->
    // Successful network request
    userLocalDataSource.logUserIn(user) { userDb ->
      // Result saved in DB
      userResult.success(userDb)
    }
  }
}
```

이 콜백들은 코루틴을 사용하여 순차적인 코드로 변경될 수 있습니다.

```kotlin
suspend fun loginUser(userId: String, password: String): User {
  val user = userRemoteDataSource.logUserIn(userId, password)
  val userDb = userLocalDataSource.logUserIn(user)
  return userDb
}
```

함수 앞에 suspend modifer를 추가하였습니다. 이는 해당 함수가 코루틴 내부에서 실행되야 함을 컴파일러에게 알려줍니다. 일반적으로 suspend 함수를 특정 지점에서 멈출 수 있고, 다시 재개할 수 있는 보통의 함수라고 생각해도 좋습니다.

콜백과는 다르게 코루틴은 쓰레드 변경을 쉽게 할 수 있으며 예외 처리도 쉽습니다.

그런데 함수를 suspend로 사용하면 컴파일러는 정확히 어떤 일을 해주는걸까요?

## Suspend under the hood

다시 `loginUser` susupend 함수로 돌아가서, suspend 함수를 호출하는 함수 역시 suspend 함수여야 합니다.

```kotlin
suspend fun loginUser(userId: String, password: String): User {
  val user = userRemoteDataSource.logUserIn(userId, password)
  val userDb = userLocalDataSource.logUserIn(user)
  return userDb
}

// UserRemoteDataSource.kt
suspend fun logUserIn(userId: String, password: String): User

// UserLocalDataSource.kt
suspend fun logUserIn(userId: String): UserDb
```

코틀린 컴파일러는 내부적으로 [유한 상태 머신(finite state machine)](https://en.wikipedia.org/wiki/Finite-state_machine)을 사용하여 suspend 함수를 최적화된 콜백 형태로 바꿉니다(밑에서 자세히 다룹니다).

결론적으로, **컴파일러는 여러분 대신하여 최적화된 콜백들을 작성해줍니다!**

## Continuation interface

suspend 함수들은 `Continuation`객체를 통해 다른 suspend 함수들과 소통합니다. [Continuation](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.coroutines/-continuation/) 는 몇가지 추가 정보를 가진 제네릭 callback interface일 뿐입니다. 나중에 보시겠지만, 이는 suspend 함수의 generated state machine을 나타냅니다.

Continuation의 정의는 아래와 같습니다.

```kotlin
interface Continuation<in T> {
  public val context: CoroutineContext
  public fun resumeWith(value: Result<T>)
}
```

- `context` 는 continuation 내부에서 사용될 `CoroutineContext`입니다.
- `resumeWith`는 `Result`와 함께 코루틴을 재개합니다. 이는 일시 중지를 유발하는 연산의 결과를 포함할 수도 있고 예외를 포함할 수 도 있습니다.

주의 : Kotlin 1.3부터는 [resume(value: T)](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.coroutines/resume.html)나 [resumeWithException(exception: Throwable)](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.coroutines/resume-with-exception.html) 확장 함수를 사용할 수 있습니다. 이들은 `resumeWith` 호출의 특별판이라고 보시면 됩니다.

컴파일러는 suspend modifier를 함수의 추가 파라미터인 `completeion`(`Continuation` 타입)으로 바꿉니다. 이 completion은 suspend 함수의 결과를 가지고 자신을 호출한 코루틴과 소통할 때 사용됩니다.

```kotlin
fun loginUser(userId: String, password: String, completion: Continuation<Any?>) {
  val user = userRemoteDataSource.logUserIn(userId, password)
  val userDb = userLocalDataSource.logUserIn(user)
  completion.resume(userDb)
}
```

간단하게 하기 위해서, 우리 예제는 `User`대신 `Unit`을 리턴하고 있습니다. `User` 객체는 추가된 `Continuation` 파라미터 안으로 return 됩니다.

실제로 suspend 함수들의 바이트코드는 Any?를 리턴하는데요, 이는 Any?가 `T | COROUTINE_SUSPENDED`의 union type이기 때문입니다. 따라서 함수가 리턴이 가능할 때가 되면 동기적으로 리턴하게끔 됩니다.

> 주의 : 만약 내부적으로 어떤 suspend 함수도 호출하지 않는 함수를 suspend 함수로 지정하면, 컴파일러는 여전히 Continuation 파라미터를 추가하긴 하겠지만 그걸로 어떤 일도 하지 않습니다. 그 함수의 바이트 코드는 일반적인 다른 함수와 다를게 없습니다.

다른 곳에서도 `Continuation` 인터페이스를 볼 수 있습니다.

- [suspendCoroutine](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.coroutines/suspend-coroutine.html) 혹은 [suspendCancellableCoroutine](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/suspend-cancellable-coroutine.html)(대부분의 경우 이게 더 권장됩니다)를 사용하여 callback기반 API를 코루틴으로 바꿀 때, 파라미터로 전달된 코드 블럭을 실행 한 후 중단된 코루틴을 재개하기 위해 `Continuation` 객체와 직접 상호작용합니다.
- suspend 함수에서 `startCoroutine` 확장 함수를 사용하여 coroutine을 시작할 수 있습니다. 이 함수는 Continuation객체를 파라미터로 받는데요, 이는 새 코루틴이 결과를 반환하든, 예외륵 반환하든 완료될 때 호출됩니다.

## Using different Dispatchers

복잡한 계산을 서로 다른 쓰레드에서 실행하도록 서로 다른 Dispatcher를 바꿔가며 작업할 수 있습니다. Kotlin은 어떻게 어디서 일시 중지된 작업을 재개 할 지 알 수 있을까요?

`Continuation`의 하위 타입인 `DispatchedContinuation`라는게 있습니다. 이 클래스의 resume 함수는 `Dispatcher`의 dispatch 호출을 `CoroutineContext` 내에서 사용가능하도록 합니다. 모든 `Dispatcher`들은 `isDispatchNeeded` 함수(`dispatch`전에 호출됨)가 항상 `false`를 반환하는 `Dispatchers.Unconfined`를 제외하고 전부 `dispatch`를 호출합니다.

## The generated State machine

> 아래에서 보시는 코드는 컴파일러에 의해 생성되는 바이트코드와 완전히 일치하지는 않습니다. 단지 내부적으로 어떻게 동작하는지를 설명하기 위한 코틀린 코드입니다. Coroutine version 1.3.3 기준으로 작성되었고, version이 바뀜에 따라 차이가 생길 수 있습니다.

Kotlin 컴파일러는 함수가 일시중지될 수 있는 시기를 내부적으로 식별합니다. 모든 일시 중지(suspension) 지점은 유한 상태 기계로 표현됩니다. 이 상태들은 컴파일러에 의해 label들로 표현됩니다.

```kotlin
fun loginUser(userId: String, password: String, completion: Continuation<Any?>) {

  // Label 0 -> first execution
  val user = userRemoteDataSource.logUserIn(userId, password)

  // Label 1 -> resumes from userRemoteDataSource
  val userDb = userLocalDataSource.logUserIn(user)

  // Label 2 -> resumes from userLocalDataSource
  completion.resume(userDb)
}
```

상태 기계를 더 잘 표현하기 위해서, 컴파일러는 `when`문을 사용합니다.

```kotlin
fun loginUser(userId: String, password: String, completion: Continuation<Any?>) {
  when(label) {
    0 -> { // Label 0 -> first execution
        userRemoteDataSource.logUserIn(userId, password)
    }
    1 -> { // Label 1 -> resumes from userRemoteDataSource
        userLocalDataSource.logUserIn(user)
    }
    2 -> { // Label 2 -> resumes from userLocalDataSource
        completion.resume(userDb)
    }
    else -> throw IllegalStateException(/* ... */)
  }
}
```

이 코드는 서로 다른 state들이 서로 정보를 공유할 수 있는 방법이 없기 때문에 완전하지 못합니다. 컴파일러는 이를 해결하기 위해 동일한 `Continuation`을 사용합니다. `Continuation`의 제네릭 타입이 원래 함수의 리턴 타입(`User`)이 아닌 `Any?`인 이유가 바로 이것이죠.

게다가 컴파일러는 필요한 data를 hold하면서도 `loginUser` 함수를 재귀적으로 호출하여 실행을 재개할 수 있도록 private class를 생성합니다. 아래에서 generated class와 거의 흡사한 클래스를 확인하실 수 있습니다.

> 주석은 컴파일러에 의해 생성된게 아닙니다. 코드를 이해하기 쉽도록 제가 임의로 추가했습니다.

```kotlin
fun loginUser(userId: String?, password: String?, completion: Continuation<Any?>) {

  class LoginUserStateMachine(
    // completion parameter is the callback to the function
    // that called loginUser
    completion: Continuation<Any?>
  ): CoroutineImpl(completion) {

    // Local variables of the suspend function
    var user: User? = null
    var userDb: UserDb? = null

    // Common objects for all CoroutineImpls
    var result: Any? = null
    var label: Int = 0

    // this function calls the loginUser again to trigger the
    // state machine (label will be already in the next state) and
    // result will be the result of the previous state's computation
    override fun invokeSuspend(result: Any?) {
      this.result = result
      loginUser(null, null, this)
    }
  }
  /* ... */
}
```

`invokeSuspend`에서 `loginUser`에 `Continuation`객체를 넘기며 계속 다시 호출하기 때문에, `loginUser` 함수의 나머지 파라미터들은 nullable이 됩니다. 이 지점에서, 컴파일러는 어떻게 state를 이동할 것인지에 대한 정보만 추가하면됩니다.

가장 먼저 해야할 일은 1)지금이 함수가 처음 호출된 상태인지를 알아야 하고, 2)함수가 이전 state에서 재개(resume)된 상태인지를 알아야 합니다. 이는 전달된 continuation이 `LoginUserStateMachine` 타입인지 아닌지를 통해 알 수 있습니다.

만약 처음 호출된 상황이라면, `LoginUserStateMachine` 객체를 생성하고 파라미터로 받은 `completion` 객체를 저장하여 이 인스턴스를 호출한 함수를 어떻게 다시 시작하는지 그 방법을 기억합니다. 만약 처음 호출된게 아니라면, 상태 기계(suspend function)를 계속 실행합니다.

이제 state 이동 및 state간 정보 공유를 위해 컴파일러가 생성한 코드를 살펴봅시다.

```kotlin
fun loginUser(userId: String?, password: String?, completion: Continuation<Any?>) {
    /* ... */

    val continuation = completion as? LoginUserStateMachine ?: LoginUserStateMachine(completion)

    when(continuation.label) {
        0 -> {
            // Checks for failures
            throwOnFailure(continuation.result)
            // Next time this continuation is called, it should go to state 1
            continuation.label = 1
            // The continuation object is passed to logUserIn to resume
            // this state machine's execution when it finishes
            userRemoteDataSource.logUserIn(userId!!, password!!, continuation)
        }
        1 -> {
            // Checks for failures
            throwOnFailure(continuation.result)
            // Gets the result of the previous state
            continuation.user = continuation.result as User
            // Next time this continuation is called, it should go to state 2
            continuation.label = 2
            // The continuation object is passed to logUserIn to resume
            // this state machine's execution when it finishes
            userLocalDataSource.logUserIn(continuation.user, continuation)
        }
          /* ... leaving out the last state on purpose */
    }
}
```

이전 코드 스니펫과 다른점들이 보이시나요? 컴파일러가 생성한 것들을 살펴봅시다.

- `when` 문의 argument가 `LoginUserStateMachine`객체의 `label`이 되었습니다.
- 새로운 state가 진행될 때마다, 함수가 일시 중지되었을 때 오류가 발생했는지 확인합니다.
- 다음 suspend 함수(예-`logUserIn`)를 호출하기 전에, `LoginUserStateMachine`의`label`이 다음 state를 위해 업데이트됩니다.
- state machine 내부에서 다른 suspend function을 호출할 때, `contination` 객체(`LoginUserStateMachine` 타입)를 파라미터로 넘깁니다. 호출될 suspend function 역시 컴파일러에 의해 변경되어 있으며 continuation 객체를 파라미터로 받는 또다른 상태 머신입니다! 그 suspend function의 상태 머신이 종료되면, 이 상태 머신을 다시 실행합니다.

마지막 state는 본인을 호출한 함수의 실행을 재개해야하기 때문에 상황이 조금 다릅니다. 아래 코드와 같이 `LoginUserStateMachine`에 저장된 cont 변수의 resume을 호출해야 합니다.

```kotlin
fun loginUser(userId: String?, password: String?, completion: Continuation<Any?>) {
    /* ... */

    val continuation = completion as? LoginUserStateMachine ?: LoginUserStateMachine(completion)

    when(continuation.label) {
        /* ... */
        2 -> {
            // Checks for failures
            throwOnFailure(continuation.result)
            // Gets the result of the previous state
            continuation.userDb = continuation.result as UserDb
            // Resumes the execution of the function that called this one
            continuation.cont.resume(continuation.userDb)
        }
        else -> throw IllegalStateException(/* ... */)
    }
}
```

보셨다시피 컴파일러는 상당히 많은 일을 대신 해줍니다! 아래와 같은 suspend function을

```kotlin
suspend fun loginUser(userId: String, password: String): User {
  val user = userRemoteDataSource.logUserIn(userId, password)
  val userDb = userLocalDataSource.logUserIn(user)
  return userDb
}
```

이렇게 만들어주죠.

```kotlin
fun loginUser(userId: String?, password: String?, completion: Continuation<Any?>) {

    class LoginUserStateMachine(
        // completion parameter is the callback to the function that called loginUser
        completion: Continuation<Any?>
    ): CoroutineImpl(completion) {
        // objects to store across the suspend function
        var user: User? = null
        var userDb: UserDb? = null

        // Common objects for all CoroutineImpl
        var result: Any? = null
        var label: Int = 0

        // this function calls the loginUser again to trigger the
        // state machine (label will be already in the next state) and
        // result will be the result of the previous state's computation
        override fun invokeSuspend(result: Any?) {
            this.result = result
            loginUser(null, null, this)
        }
    }

    val continuation = completion as? LoginUserStateMachine ?: LoginUserStateMachine(completion)

    when(continuation.label) {
        0 -> {
            // Checks for failures
            throwOnFailure(continuation.result)
            // Next time this continuation is called, it should go to state 1
            continuation.label = 1
            // The continuation object is passed to logUserIn to resume
            // this state machine's execution when it finishes
            userRemoteDataSource.logUserIn(userId!!, password!!, continuation)
        }
        1 -> {
            // Checks for failures
            throwOnFailure(continuation.result)
            // Gets the result of the previous state
            continuation.user = continuation.result as User
            // Next time this continuation is called, it should go to state 2
            continuation.label = 2
            // The continuation object is passed to logUserIn to resume
            // this state machine's execution when it finishes
            userLocalDataSource.logUserIn(continuation.user, continuation)
        }
        2 -> {
            // Checks for failures
            throwOnFailure(continuation.result)
            // Gets the result of the previous state
            continuation.userDb = continuation.result as UserDb
            // Resumes the execution of the function that called this one
            continuation.cont.resume(continuation.userDb)
        }
        else -> throw IllegalStateException(/* ... */)
    }
}
```

---

코틀린 컴파일러는 모든 suspend 함수를 상태 머신으로 변경하여 함수를 일시 중지해야 할 때마다 콜백을 사용해 최적화합니다.

컴파일러가 내부적으로 어떻게 동작하는지 알게 되었으니, suspend 함수가 왜 일을 모두 끝마치기 전까지 return하지 않는지 이해하셨으리라 생각합니다. 또한 어떻게 thread를 block하지 않고 일시중지 하는지도요(함수가 다시 시작될 때 필요한 정보들이 `Continuation` 객체에 저장됩니다!)
