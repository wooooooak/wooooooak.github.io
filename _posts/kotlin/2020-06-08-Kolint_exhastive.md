---
layout: post
title: Kotlin exhaustive with sealed class and when
category: Kotlin
tags: [exhaustive, sealed_class, when]
comments: true
---

프로그래밍을 하다보면 열거형(Enum)이 필요할 때가 굉장히 많다. 대표적인 예로 요일(월,화,수...일)이나 방향(동,서,남,북)처럼 특정 카테고리 안에서 열거할 수 있는 값들을 나타낼 때 쓰인다. 더 나아가서 우리가 창조해낸 객체지향 세상에서 얼마든지 다양한 열거형들이 추가로 생겨날 수 있다.

코틀린에서는 언어 자체에서 `Sealed class`라는 개념을 지원하기 때문에 Enum을 사용할 일은 거의 없다. Enum을 사용하면 각각의 타입을 나타내는 값들이 싱글톤으로 인식되는 반면, `Sealed class`를 사용하면 얼마든지 동일한 타입의 객체를 생성해 낼 수 있으며, 싱글톤으로도 사용이 가능하다.

`Sealed class`가 익숙하지 않다면 [공식문서](<[https://kotlinlang.org/docs/reference/sealed-classes.html](https://kotlinlang.org/docs/reference/sealed-classes.html)>)를 참조하자.

## when

우선 `when`의 특징을 간략하게 짚고 넘어가보자. `when`은 `if`와 마찬가지로 식(expression)과 문(statement)로 사용이 가능하다. 식으로 쓰인다는 말은 반환되는 값이 있다는 말로 이해할 수 있다. 아래의 예제를 보자.

![carbon (4)](https://user-images.githubusercontent.com/18481078/84029300-1f7e9280-a9cd-11ea-9ddc-ede225094cf7.png)

위의 코드는 `if`와 `when`이 식(expression)으로 사용 되어 값을 반환한다. 값을 반환하도록 사용되었기 때문에 `when` 에서는 꼭 `else`가 있어야만 한다. 만약 `else`가 없다면 `fruit`가 "apple", "orange", "banana"가 아닐 때 어떤 값을 반환해야 하는지 정할 수 없으므로 컴파일 에러가 난다.

반면 `if`, `when`을 문(statement)로 사용할 때는 값을 반환하지 않는다. 아래 예제는 statement로 사용하는 예제이다.
<img src="https://user-images.githubusercontent.com/18481078/84029264-11c90d00-a9cd-11ea-8f76-a74d5e74ea95.png" width="500" />

앞선 예제와는 다르게 값을 반환하지 않는다. 따라서 `else`문이 없어도 상관없다. 조건에 해당하지 않으면 아무 일도 일어나지 않을 뿐 컴파일에러, 런 타임에러는 일어나지 않는다.

## when + sealed class

`Sealed class`의 효력은 `when`과 함께할 때 강력하게 나타난다. `Sealed class`와 `when` expression을 함께 쓰는 경우를 상상해보자. `Sealed class`를 사용하는 이유는 어떤 값이 특정한 카테고리 안에 속해 있음을 보장하기 위해서이다. UI를 그리는 코드를 작성하다 보면 데이터를 요청한 이후의 상태는 크게 세가지로 나눌 수 있을 것이다. **loading**이거나, 요청에 **실패(Error)** 했거나, **성공(Success)** 하여 값을 받아왔거나. 이를 아래 처럼 나타낼 수 있고, 실제로 사용한다면 expression 보다는 statement로 사용할 일이 많다.

<img src="https://user-images.githubusercontent.com/18481078/84029264-11c90d00-a9cd-11ea-8f76-a74d5e74ea95.png" width="400" />

여기서 조심 해야할 부분은 `when`이 statement로 사용되었기 때문에, `uiState`가 `Loaindg`, `Error`, `Success`에 걸리지 않더라도 컴파일 에러가 나지 않고, 런 타임 때 코드는 아무런 행동도 하지 않는다는 것이다.

**다른 누군가가 필요에 의하여 `UiState`에 `Retry`라는 상태 값을 추가하면 어떻게 될까?** 바로 위의 when문에서 컴파일 에러를 띄워주지 않기 때문에 Retry 상태 체크를 까먹을 확률이 매우 높다. `else`를 사용할 수 있긴 하지만 `Retry`가 아니라 다른 값이 추가된다면 그에 대응하지 못한다.

**먼 훗날 본인 혹은 다른 개발자가 UiState에 다른 상태를 추가하게 될 경우, 컴파일 에러를 발생시켜 이를 파악하고자 한다면 when을 식(expression)으로 사용할 수도 있다.**

<img src="https://user-images.githubusercontent.com/18481078/84029089-cca4db00-a9cc-11ea-84f3-51378d4c1ca6.png" width="500" />

![image](https://user-images.githubusercontent.com/18481078/84029029-b39c2a00-a9cc-11ea-9513-51bf2fa15baa.png)

`when`을 식으로 사용하면 꼭 값을 반환해야 하기에 `Retry`를 체크하라는 에러가 뜬다. 그러나 좋은 방법은 아닌 것 같다. 먼 훗날 hello라는 변수를 보면 "사용하지도 않는 변수가 왜 있지?"라며 **코드의 가독성을 해칠 가능성이 보인다.** 또 다른 방법으로는 아래 이미지와 같이 `let`을 사용하는 방법이 있는데, 이것 역시 먼 훗날 다른 개발자가 보기에는 혼란을 일으킬 수 있는 코드이다.

![image](https://user-images.githubusercontent.com/18481078/84028959-98c9b580-a9cc-11ea-991a-d3f1359c0768.png)

`let`을 사용하게 되면 statement처럼 쓴 `when`을 식(expression)처럼 사용할 수 있다. `let` 함수는 자신을 호출한 객체를 람다 블록의 수신 객체로 사용하기 때문이다. 즉 `when`이 무언가를 반환 해야만 하는 것이다. 따라서 컴파일 에러가 난다. 하지만 이것 역시 먼 훗날 다른 개발자가 코드를 볼 때 의미 없어 보이는 .let 블럭을 지울 확률이 높아 보인다.

## exhaustive

프로젝트 여기저기서 exhaustive라는 코드가 종종 보여서 구현체를 보았더니 아주 심플하다.

```kotlin
val <T> T.exhaustive: T
    get() = this
```

**exhaustive : (하나도 빠뜨리는 것 없이) 철저한**

바로 위에서 `let`을 사용하여 statement를 expression으로 대체한 원리와 같다. exhaustive라는 의미와 부가적인 코드가 없기 때문에 가독성이 좋고 추후 혼란을 일으킬 여지가 없어보인다.

<img src="https://user-images.githubusercontent.com/18481078/84028630-09bc9d80-a9cc-11ea-86e7-ec4e06d696ef.png" width="500" />
