---
layout: post
title: Using CoroutineContext to repeat failed HTTP request(번역)
category: 번역하며 공부하기
tags: [Coroutine, CoroutineContext]
comments: true
---

원본 - [Using CoroutineContext to repeat failed HTTP request](https://proandroiddev.com/using-coroutinecontext-to-repeat-failed-http-request-9d7092d8cec1)

안드로이드 앱에서는 인터넷 연결에 문제가 있을 때처럼 무언가 문제가 있을 때면 요청을 다시 보내야 하는 경우가 흔합니다.

여러개의 HTTP 요청을 보내고나서 그 중 실패한 요청들에 한하여 "재요청" 버튼을 누를 수 있도록 하는 화면이 있다고 가정해봅시다(동시에 여러개의 요청을 보내는 상황으로만 생각하실 필요는 없습니다).

만약 화면에 요청이 단 하나만 있다면 매우 간단한 일이겠죠. 단지 그 요청을 다시 요청하면 되니까요. 그러나 한 화면에서 제각각 다른 시각에 보내진 여러 다른 요청들이 있다면, 우리는 그 요청들을 구분해낼 수 있는 방법이 필요합니다. 마지막에 호출된 함수 자체를 계속 저장해놓는 방법이 있긴 하지만 이건 유연하지 못할 겁니다. 만약 몇몇 요청은 "다시 시도하기" 다이얼로그를 띄워주지 않아도 될 경우 그 변수들을 다 지워줘야 하고, 만약 지워주는 걸 까먹는다면 "다시 시도하기" 버튼을 눌렀을 경우 성공한 함수가 재요청되는 버그가 되겠죠.

간단한 해결법은 CoroutineContext를 사용하여 나중에 다시 시도해야하는 함수를 저장하는 것입니다.

기본적으로 CoroutineContext는 `Job`, `CoroutineDispatcher`, `CoroutineName`, `CoroutineExceptionHandler` 들로 구성됩니다(자세한 내용는 [article about Coroutines](https://medium.com/swlh/kotlin-coroutines-in-android-summary-1ed3048f11c3)에서 확인할 수 있습니다). 그러나 공식 문서의 정의에 따르면 CoroutineContext는 그저 `Element` 객체들의 indexed set입니다. 이말은 즉 `AbstractCoroutineContextElement` 클래스를 확장하여 자신만의 `Element` interface를 구현할 수도 있다는 이야기입니다.

```kotlin
class RetryCallback(val callback: () -> Unit) :
   AbstractCoroutineContextElement(RetryCallback) {
   companion object Key : CoroutineContext.Key<RetryCallback>
}
```

위와 같이 코드를 작성하고나면 우리가 시작한 코루틴의 CoroutineContext에 RetryCallback를 추가할 수 있습니다(launch의 첫 번째 인자로 CoroutineContext를 넘깁니다).

```kotlin
fun someFunction(someParam: List<String>) {
   viewModelScope.launch(exceptionHandler
      + RetryCallback { someFunction(someParam) }) {
      // perform the request, update the view with the result
   }
}
```

그리고 만약 request가 실패한다면 CoroutineExceptionHandler에서 callback을 다룰수 있습니다.

```kotlin
private val exceptionHandler = CoroutineExceptionHandler {
   coroutineContext, throwable ->
   val callback: (() -> Unit)? = coroutineContext[RetryCallback.Key]
                                    ?.callback
   // ...
}
```

CoroutineExceptionHandler에서 단순히 함수를 Activity나 Fragment로 넘기고, 사용자가 "다시 시도하기"버튼을 클릭할 때 그 함수를 실행히시키만 하면 됩니다.
