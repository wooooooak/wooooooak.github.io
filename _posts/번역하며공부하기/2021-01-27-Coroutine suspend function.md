---
layout: post
title: Coroutine suspend function - when does it start, suspend or terminate?(번역)
category: 번역하며 공부하기
tags: [Coroutine]
comments: true
---

원본 - [Coroutine suspend function: when does it start, suspend or terminate?](https://medium.com/mobile-app-development-publication/coroutine-suspend-function-when-does-it-start-suspend-or-terminate-2762cabac54e#id_token=eyJhbGciOiJSUzI1NiIsImtpZCI6ImVlYTFiMWY0MjgwN2E4Y2MxMzZhMDNhM2MxNmQyOWRiODI5NmRhZjAiLCJ0eXAiOiJKV1QifQ.eyJpc3MiOiJodHRwczovL2FjY291bnRzLmdvb2dsZS5jb20iLCJuYmYiOjE2MTE2NjY4NzksImF1ZCI6IjIxNjI5NjAzNTgzNC1rMWs2cWUwNjBzMnRwMmEyamFtNGxqZGNtczAwc3R0Zy5hcHBzLmdvb2dsZXVzZXJjb250ZW50LmNvbSIsInN1YiI6IjExNzc4Njk0MDMzNzg1MzQ2MDYxMSIsImVtYWlsIjoid29vb29vb2FrQGdtYWlsLmNvbSIsImVtYWlsX3ZlcmlmaWVkIjp0cnVlLCJhenAiOiIyMTYyOTYwMzU4MzQtazFrNnFlMDYwczJ0cDJhMmphbTRsamRjbXMwMHN0dGcuYXBwcy5nb29nbGV1c2VyY29udGVudC5jb20iLCJuYW1lIjoiWU9OR0pVTiIsInBpY3R1cmUiOiJodHRwczovL2xoMy5nb29nbGV1c2VyY29udGVudC5jb20vYS0vQU9oMTRHamZBTi14Vy1mMllxaW9BLWNZb2M5dnVNZWR3U1EwS2dfTllmTDRMdz1zOTYtYyIsImdpdmVuX25hbWUiOiJZT05HSlVOIiwiaWF0IjoxNjExNjY3MTc5LCJleHAiOjE2MTE2NzA3NzksImp0aSI6IjkxMWU4MDdjM2NlNzJiMGJhZWE3MThhYmVlZWFkNjhkOGRlOWNjNTQifQ.xK9vlQUyZlbdPolRghlItCP380wB5ExhDE_EiS34sULtr0Suyq2ZUJwwmUrgkhz3ihjnxqm2gqQMc-JAsgcy-kBijn_xBRL0EgxE0IDGn11RJYGdsRu3MK-i4U4_DZo5q6BZw73uGfsloLOnJck2Dcn2f6aE_H26rGWFmep6WrTmFtKjCJriYuwBvcKD79s2dP0OZOS6F_ozhBJX6Vh7eGaOKSk6BhfAGiyundHirpsGZJo329HncVK_p-P2y6whSW2YDRkWfQ5xddzqfExmPzPx-No_LP5O5IjoKj2whXJKw6PtCGOIt1vLz2qf5wv3fEXhW0tnCdabqFY0U4C_WQ)

_코루틴 라이프사이클을 잘 컨트롤하는 방법 알기_

거의 3년이란 시간동안 코루틴을 잘 이해하기위해 suspend function이 정확히 무엇인지를 탐구해왔습니다.

[Understanding suspend function of Kotlin Coroutines](https://medium.com/mobile-app-development-publication/understanding-suspend-function-of-coroutines-de26b070c5ed)

그러나 아직 답을 내지 못한 질문이 있었는데, 언제 그게 시작되고, 멈추고(suspend), 종료되는가?라는 질문입니다. 이것을 알고 코루틴을 사용하면 보다 더 정확하게 사용할 수 있습니다.

## 언제 suspend되는가?

suspend 함수를 다루는 글이니까 suspension 지점부터 살펴볼게요.

### suspend 재진입

아래와 같이 `delay`와 함께 `launch`를 사용하는 코드가 있습니다.

```kotlin
fun launchWithDelay() {
    runBlocking {
        launch {
            println("First start")
            delay(200)
            println("First ended")
        }

        launch {
            println("Second start")
            delay(300)
            println("Second ended")
        }
    }
}
```

첫 번째 코루틴이 `delay`를 만나게 되면 이 함수는 일시중지(suspend)되고, 두 번째 코루틴이 진행됩니다.

따라서 결과는 아래와 같습니다.

```kotlin
1st start
2nd start
1st end
2nd end
```

### delay없이 suspend

만약 `delay`를 없앤다면 어떨까요?

```kotlin
fun launchWithoutDelay() {
    runBlocking {
        launch {
            println("1st start")
            println("1st end")
        }

        launch {
            println("2nd start")
            println("2nd end")
        }
    }
}
```

결과는 아래와 같습니다.

```kotlin
1st start
1st end
2nd start
2nd end
```

보시다시피 더이상 두 함수를 교차 진행하지 않습니다. 코루틴 블럭에 `delay`가 없다면 일시중지하지 않는것 처럼 보입니다.

### yield와 함께쓰는 suspend

만약 일시중지를 하고는 싶은데, `delay`를 하고싶지는 않다면 어떻게 할까요? `yield`를 사용하시면 됩니다.

```kotlin
fun launchWithYield() {
    runBlocking {
        launch {
            println("1st start")
            yield()
            println("1st end")
        }

        launch {
            println("2nd start")
            yield()
            println("2nd end")
        }
    }
}
```

이제 `delay`를 사용했을 때와 같은 결과를 얻을 수 있습니다.

```kotlin
1st start
2nd start
1st end
2nd end
```

이쯤해서, [yield](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/yield.html)란 무엇일까요?

_다른 코루틴이 실행 가능한 상태라면, 현재 코루틴 디스패쳐의 thread(혹은 thread pool)를 다른 코루틴 에게 양보합니다_

_이 suspending 함수는 취소 가능합니다. 만약 yield 함수가 호출되었을 때 혹은 이 함수가 dispatch를 기다리고 있는 상황에서 현재 코루틴의 Job이 이미 취소 되었거나 완료되었다면, 이 함수는 [CancellationException](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-cancellation-exception/index.html)을 내뿜으며 resume됩니다. 이는 즉각 취소됨을 의미합니다. 만약 이 함수가 일시중지 된 상태에서 job이 취소된다면(yield함수로 인해 일시중단 되어 코루틴을 빠져나온 상태), 성공적으로 resume되지 못합니다._

간단히 말하자면 yield는 다른 suspension 함수로 제어를 넘길 수 있는지 확인하는 check point이며, 코루틴이 취소 되었는지를 확인하는 check point이기도 합니다.

## 언제 시작는가?

start되는 시점을 살펴봅시다.

- `async-await`의 경우, 코루틴은 오직 start 함수가 호출될 때 시작됩니다.
- `runBlocking`의 경우, blocking되기 때문에 호출되자 마자 즉시 시작됩니다.
- `launch`의 경우는 언제 시작될까요?

### Join없이 Start

간단한 예제를 볼게요.

```kotlin
fun startWithoutJoin() {
    runBlocking {
        println("runBlocking start")
        launch {
            println("Launch start")
        }
        println("runBlocking end")
    }
}
```

결과는 아래와 같습니다.

```kotlin
runBlocking start
runBlocking end
Launch start
```

`luanch`는 오직 suspend 함수(여기서는 runBlocking 함수)가 끝이 날때 시작되는 것을 볼수 있습니다.

### Join과 함께 Start

조금 더 빨리, runBlocking이 종료되기 전에 시작시키고 싶다면, `job`의 `join`을 호출하면 됩니다.

```kotlin
fun startWithJoin() {
    runBlocking {
        println("runBlocking start")
        val job = launch {
            println("Launch start")
        }
        job.join()
        println("runBlocking end")
    }
}
```

결과는 아래와 같습니다.

```kotlin
runBlocking start
Launch start
runBlocking end
```

`join`에는 한가지 제약이 있습니다. 이 함수는 `laucnh`가 끝날 때까지 runBlocking 블럭을 block시킵니다. 이는 `launch`를 다른 쓰레드로 실행했을 때도 동일합니다.

```kotlin
fun startWithJoin() {
    runBlocking {
        println("runBlocking start")
        val job = launch(Dispatchers.IO) {
            println("Launch start")
        }
        job.join()
        println("runBlocking end")
    }
}
```

### Yield와 함께 Start

`launch`가 시작되기도 하면서 기존 suspend 함수를 block하지도 않으려면, `yield`를 사용하면 됩니다. 위에서 설명드린것 처럼 `yield`는 다른 코루틴이 시작되도록 하니까요.

동일한 쓰레드에서 실행할 경우, 2개의 `yield`가 필요합니다.

```kotlin
fun startWithYield() {
    runBlocking {
        println("runBlocking start")
        launch {
            repeat(3) {
                println("Launch start")
                yield()
            }
        }
        yield()
        println("runBlocking end")
    }
}
```

runBlocking의 `yield`는 `launch`가 시작되게 합니다. `launch`안의 `yield`는 runBlocking이 함수를 종료할 수 있도록 다시 제어권을 넘깁니다.

결과는 아래와 같습니다.

```kotlin
runBlocking start
Launch start
runBlocking end
Launch start
Launch start
```

그러나 다른 쓰레드에서 `launch`한다면, runBlocking에 단 하나의 `yield`만 있으면 됩니다.

```kotlin
fun startWithYield() {
    runBlocking {
        println("runBlocking start")
        launch(Dispatchers.IO) {
            repeat(3) {
                println("Launch start")
                // yield()   // This is not needed anymore
            }
        }
        yield()
        println("runBlocking end")
    }
}
```

## 언제 종료되는가?

`yield`와 같은 함수를 만났을 때만 일시중지(suspend)된다는 점을 고려해봤을 때, 종료(취소)는 어떨까요? 종료는 일지중지와 다르게 즉각 반응할까요? 아니면 `yield`와 같은 함수가 필요할까요?

### [CancellationException](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-cancellation-exception/index.html) 알기

취소에 대해서 실습을 해보기 전에, 코루틴이 취소되었다는 것을 알아차리는 방법부터 알아봅시다.

코루틴이 취소되었을 때, [CancellationException](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-cancellation-exception/index.html) 이 트리거됩니다.

`try-catch-finally` 를 사용하여 이를 잡아낼 수 있습니다.

```kotlin
val job = launch {
    try {
        // Coroutine task
    } catch (e: CancellationException) {
        // Cancel is triggered
    } finally {
        // Opportunity to clear up
    }
}
```

### Yield 없이 종료

오래 걸리는 loop작업을 취소시켜 봅시다.

```kotlin
@Test
fun terminateWithoutYielding() {
    runBlocking {
        println("Launching...")
        val job = launch {
            try {
                println("Start looping")
                repeat(2000) {
                    repeat(2000) {
                        repeat(2000) {}
                    }
                }
                println("Done looping")
            } catch (e: CancellationException) {
                println("It is cancelled")
            } finally {
                println("Finally all finish")
            }
        }

        println("Launched...")
        println("Cancelling...")
        job.cancel()
        println("Cancelled...")
    }
}
```

결과는 아래와 같습니다.

```kotlin
Launching...
Launched...
Cancelling...
Cancelled...
```

`launch`가 트리거되기도 전에 취소되었으니 당연한 결과입니다.

하지만 다른 쓰레드에서 `launch`하게 된다면 상황이 다릅니다.

```kotlin
@Test
fun terminateWithoutYielding() {
    runBlocking {
        println("Launching...")
        val job = launch(Dispatchers.IO) {
            try {
                println("Start looping")
                repeat(2000) {
                    repeat(2000) {
                        repeat(2000) {}
                    }
                }
                println("Done looping")
            } catch (e: CancellationException) {
                println("It is cancelled")
            } finally {
                println("Finally all finish")
            }
        }

        println("Launched...")
        println("Cancelling...")
        job.cancel()
        println("Cancelled...")
    }
}
```

`cancel`이 호출될 때, 다른 쓰레드에서 `laucnh`가 trigger된 상태입니다.

```kotlin
Launching...
Launched...
Cancelling...
Start looping
Cancelled...
Done looping
Finally all finish
```

그래서 `job`은 해당 코루틴을 완료하기 전까지 취소되지 않습니다.
그 이유는, [공식 문서](https://kotlinlang.org/docs/reference/coroutines/cancellation-and-timeouts.html#cancellation-is-cooperative)에 나와있듯 코루틴 취소에는 협력(cooperation)이 필요하기 때문입니다.

### Yield와 함께 종료하기

동일한 쓰레드에서 `yield`를 사용하여 `launch`가 실행되게 하고 싶다면 아래와 같이 할 수 있습니다.

```kotlin
fun terminateWithYielding() {
    runBlocking {
        println("Launching...")
        val job = launch {
            try {
                println("Start looping")
                repeat(2000) {
                    repeat(2000) {
                        repeat(2000) {}
                    }
                }
                println("Done looping")
            } catch (e: CancellationException) {
                println("It is cancelled")
            } finally {
                println("Finally all finish")
            }
        }

        println("Launched...")
        yield()
        println("Cancelling...")
        job.cancel()
        println("Cancelled...")
    }
}
```

여기서는 `launch` 블락이 runBlocking의 `cancel()` 호출 전에 시작됨에따라 `cancel()` 호출의 효력이 없습니다.

```kotlin
Launching...
Launched...
Start looping
Done looping
Finally all finish
Cancelling...
Cancelled...
```

따라서 `launch`의 작업이 실행되고 있는 동안 `cancel`이 호출되지 않았기 때문에 전혀 취소 작업이 일어나지 않습니다.

`cancel`을 하기 위해서는 `launch`내부에 `yield`를 넣으면 됩니다.

```kotlin
@Test
fun terminateWithYielding() {
    runBlocking {
        println("Launching...")
        val job = launch {
            try {
                println("Start looping")
                repeat(2000) {
                    repeat(2000) {
                        repeat(2000) {
                            yield()
                        }
                    }
                }
                println("Done looping")
            } catch (e: CancellationException) {
                println("It is cancelled")
            } finally {
                println("Finally all finish")
            }
        }

        println("Launched...")
        yield()
        println("Cancelling...")
        job.cancel()
        println("Cancelled...")
    }
}
```

결과는 아래와 같습니다.

```kotlin
Launching...
Launched...
Start looping
Cancelling...
Cancelled...
It is cancelled
Finally all finish
```

`launch`가 트리거 되었음을 확인할 수 있습니다. 그러나 그 안에서 `yield`가 `cancel`함수가 호출될 수 있도록 한 것입니다. `cancel()`된 이후 `launch`내부의 `yield`에서 취소었는지를 확인하고, 취소되었다면 코루틴을 종료합니다.

다른 쓰레드에서 `launch`하더라도 비슷합니다.

```kotlin
@Test
fun terminateWithYielding() {
    runBlocking {
        println("Launching...")
        val job = launch(Dispatchers.IO) {
            try {
                println("Start looping")
                repeat(2000) {
                    repeat(2000) {
                        repeat(2000) {
                            yield()
                        }
                    }
                }
                println("Done looping")
            } catch (e: CancellationException) {
                println("It is cancelled")
            } finally {
                println("Finally all finish")
            }
        }

        println("Launched...")
        yield()
        println("Cancelling...")
        job.cancel()
        println("Cancelled...")
    }
}
```

`yield`함수가 코루틴이 취소될 수 있게 합니다.

```kotlin
Launching...
Launched...
Start looping
Cancelling...
Cancelled...
It is cancelled
Finally all finish
```

---

코루틴을 시작, 일시 중지, 종료 하기 위해서는 join, delay, yield같은 suspend 함수를 사용해야 한다는 것을 배웠습니다. 그저 쉽게 되는 일이 아니었습니다. 이제 이 사실을 알았으니 더 많은 것을 컨트롤 할 수 있게 되었습니다.
