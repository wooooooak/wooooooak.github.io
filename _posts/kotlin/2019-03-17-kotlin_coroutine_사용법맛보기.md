---
layout: post
title: 코틀린 코루틴 사용법 맛보기
category: Kotlin
tags: [kotlin, coroutine]
comments: true
---

코루틴이 완전히 처음이라면 [코틀린 코루틴 개념익히기](https://wooooooak.github.io/kotlin/2019/08/25/%EC%BD%94%ED%8B%80%EB%A6%B0-%EC%BD%94%EB%A3%A8%ED%8B%B4-%EA%B0%9C%EB%85%90-%EC%9D%B5%ED%9E%88%EA%B8%B0/) 를 읽고 돌아오자!

클라이언트 앱을 만들기 위해서 비동기 처리는 필수적이다. 따라서 클라이언트를 개발할 수 있는 언어, 라이브러리, 프레임워크에는 비동기 처리를 쉽게 도와주는 도구들이 존재한다. 물론 RxKotlin, coroutine 같은 도구들을 사용하지 않고도 처리할 순 있겠지만 상당히 번거로운 작업이 될 것이며, 앞으로 수 많은 비동기 처리를 해야한다면 이런 도구를 적절히 사용할 줄 아는게 유리할 것이다.

## Coroutine

코틀린의 코루틴을 사용하기위해 검색해보면 `runBlocking`, `launch`, `async/awatit`, `job` 등등의 키워들이 있고 예제들이 나온다.

## launch & async

`launch`과 `async`는 코루틴 빌더로써 쓰레드 처럼 코루틴을 시작하는 것이라고 한다. 코루틴 빌더라고 뭔가 있어보이지만 그냥 코루틴을 생성해주는 정도?로 생각하면 될 것 같다. launch와 async는 사실상 같은 일을 한다. 유일한 차이점은 `launch`가 Job을 반환하는 반면 `async`는 Defferd를 반환한다는 점 뿐이다. 따라서 항상 `launch`대신 `async`를 사용해도 문제는 없다고 한다.

launch라는 단어는 보통 미사일을 발사 할 때도 쓴다. 즉, 미사일을 위로 발사하고나면, 돌아오기 전까지 그 사실을 잊어버린다는 것. **즉 비동기가 실행된다는 것이다.** 미사일이 다시 원래의 자리로 돌아오든, 아니면 목표물에 도착했든 완료 신호가 오면 그에 맞는 처리를 할 수도 있다.

`launch`를 예로 들어보자.

```kotlin
fun main() {
    runBlocking { // start main coroutine
        launch { // launch new coroutine in background and continue
            delay(1000L)
            println("world")  // #2
        }
        println("Hello")      // #1
    }
    println("end")            // #3
}
```

위의 코드에서 `runBlocking`은 간한히 { } 블록 안의 비동기 처리들이 끝날 때 까지 코드를 블로킹 시킨다라고만 알고 넘어가자. 아래에서 더 다룬다.

위의 코드를 실행 시키면 출력 순서는 #1, #2, #3이다. `main()` 함수가 실행되면 곧바로 runBlocking을 만나게 되는데, runBlocking을 만났기 때문에 runBlocking 블록 안의 코드들이 모두 끝나야 그 다음 줄인 `println("end")` 코드가 실행된다. 따라서 #3 이 가장 마지막에 출력되는 것이다.

`runBlocking` 블록 안에서 가장 먼저 만나는게 비동기의 시작을 알리는 `launch`함수다. 위에서도 말했듯이 `launch`를 만나면 `launch` 블록의 코드들이 미사일에 담겨 비동기로 수행된다고 생각하자. 간단히 가벼운 백그라운드 쓰레드(코루틴)로 수행된다고 생각해도 좋을 것 같다.

`launch`블록 안에서 가장 먼저 만나는 `delay`라는 함수가 있는데 이것은 코루틴 스코프 안에서만 실행할 수 있는 시간 지연 함수다. 여기서는 `delay`를 사용하여 시간이 걸리는 것(미사일이 제 역할을 하고 다시 돌아오는 시간)을 나타냈지만, 실제로는 외부 api를 요청하거나 데이터베이스 IO작업처럼 시간이 걸리는 작업이라고 생각해도 된다. 아무튼 `delay`바로 밑 줄인 #2는 `delay(1000L)`이 끝나야 실행된다. `launch` 블록에 같이 묶여있기 때문이다.

아까 말햇듯이 `launch`는 코드 블럭을 마치 백그라운드 쓰레드에서 실행시키는 것과 비슷하게 비동기 식으로 동작시킨다. 따라서 `launch`블락 다음 줄 #1 이 바로 호출된다. 미사일을 발사시키는 사람이 미사일을 발사시켜 놓고 나서 미사일이 돌아올 동안 정지하는게 아니라 다른 할일을 하는 것 처럼. 따라서 `luanch`블럭을 먼저 만났음에도 불구하고 1초의 소요시간이 걸리는 비동기 코드(delay) 때문에 #1이 먼저 출력되고, 아까 발사시켰던 미사일이 1초뒤에 돌아오면 아까 멈추었던 `launch` 블럭으로 돌아가 #2가 출력되고, 마지막으로 #3이 출력되는 것이다.

## runBlocking

위의 코드에서 왜 #1 -> #3 -> #2 순서대로 실행되지 않는지 의문이 생길 수도있다. 그 것은 `runBlocking`때문이다. 위에서 언급했듯이 `runBlocking`은 간단히 { } 블록 안의 내부 코루틴(launch든 async든)이 종료되기 전 까지는 작업 중인 경량 쓰레드를 블로킹 시킨다.

쉽게 말해서 `runBlocking`은 { } 블록 안에 비동기 처리 코루틴들이 들어 있음에도 불구하고 마치 일반적인 하나의 함수처럼 작동하는 것이다. 즉, `runBlocking`블록안의 모든 비동기 작업이 끝나고,첫번째 줄에서 마지막 줄까지 모두 끝나야 그 다음 줄 코드가 실행된다. 코루틴이 아닌 일반 서브루틴(우리가 아는 기본적인 함수)처럼 말이다.

## job.join() & defferd.await()

```kotlin
fun main() {
    runBlocking{
        launch {
            delay(1000L)
            println("world")  // #2
        }
        println("Hello")      // #1
    }
    println("end")            // #3
}
```

이 코드는 위의 코드를 그대로 가져온 것인데, 실행 순서를 아래처럼 바꾸고 싶다.

```kotlin
fun main() {
    runBlocking{
        launch {
            delay(1000L)
            println("world")  // #1 이거부터 출력하고 싶다.
        }
        println("hello")      // #2 이건 두 번째
    }
    println("end")            // #3 마지막으로 출력하고 싶다.
}
```

이럴땐 코드에 `job.join()`을 넣어주면 된다.

```kotlin
fun main() {
    runBlocking{
        var job = launch {
            delay(1000L)
            println("world")  // #1
        }
        job.join()
        println("Hello")      // #2
    }
    println("end")            // #3
}
```

위처럼 하면 world -> Hello -> end 순서로 출력이된다. 즉 job의 `join()` 함수를 만나게 되면, 그 순간 `job`을 확인한 후 `job`이 아직 완료상태가 아니라 비동기 처리중인 상태일 경우 `join()`이후의 코드들을 실행시키지 않고 대기시킨다. 그리고 job들이 완료(미사일이 돌아옴)상태가 되면 그때서야 join() 아랫줄 코드들을 실행시킨다.

이제 비동기 요청으로 서버에 데이터를 요청했다고 가정하자. 서버에 데이터를 요청하고 받아오는 시간은 1초이고, 받아온 데이터를 가지고 어떤 연산을 하고싶다.
위의 코드에서 job.join()아랫줄에 연산 코드를 넣으면 되겠지만, 서버로 부터 받아온 데이터가 어디있는가? 그것을 받아오도록 코드를 바꿔보자.

```kotlin
fun main() {
    runBlocking{
        var deffered = async {
            delay(1000L)
            println("world")  // #1
            50 // 서버로 부터 받아온 데이터를 리턴해주는 부분 return은 적지 않는다.
        }
        var dataFromServer = deffered.await()
        println(`Hello $dataFromServer`)      // #2 Hello 50
    }
    println("end")            // #3
}
```

`await()`을 통해 `async`블록에서 넘겨준 값을 가져왔다. `await()`은 `join()`과 같이 `deffered`들이 완료 상태가 되면 가지고 있던 값을 뽑아내준다.

## 마무리

코루틴을 이해한다기 보다는 최소한의 사용법을 익히는 글이었다. 코루틴 스코프까지 이해하고 각각의 Dispatcher들을 모두 이해하면 좋겠지만, 계속 이론만 보면 이해에 발전이 없을 것 같아 이렇게 써보면서 이해 중이다.

`runBlocking`을 이해하고 `launch`, `async`만 사용할 줄 알아도 어느정도 즐거운 코딩은 가능하다. 여러가지 상황에 부딪혀 더 깊게 알아야 될 때도 있겠지만 우선은 이것들 부터 익숙해지자. 더 깊게 이해가 되면 그때 이론적인 부분까지 포스팅해야겠다.
