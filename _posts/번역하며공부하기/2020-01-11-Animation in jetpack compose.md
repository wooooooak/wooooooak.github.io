---
layout: post
title: Animations in Jetpack Compose(번역)
category: 번역하며 공부하기
tags: [Compose, Animation]
comments: true
---

원본 - [Animations in Jetpack Compose](https://proandroiddev.com/animations-in-jetpack-compose-bbeaa886210e)

애니메이션은 좋은 사용자 경험을 제공해줄 뿐만 아니라 애플리케이션을 보다 매력적으로 만들어 주는 중요한 역할을 합니다.

Jetpack Compose는 애니메이션을 만들기 위한 좋은 API를 제공합니다. 이를 사용하여 alpha, scale 애니메이션 등 다양한 타입의 애니메이션을 사용할 수 있습니다. 이번 튜토리얼에서는 어떻게 Jetpack Compose에서 애니메이션을 만들고 사용하는지 배워보겠습니다.

뉴스 앱을 만든다고 가정해볼게요. 이 앱은 최신 뉴스들을 리스트로 보여줄겁니다. 우리는 리스트의 모든 아이템이 오른쪽에서 들어오도록 만들어 볼겁니다.

먼저, 애니메이션 로직이 들어있는 `ListItemAnimationDefinition` 클래스를 만들어 봅시다.

```kotlin
class ListItemAnimationDefinition(delay: Int = 0) {
    val slideValue = DpPropKey(label = "offset")
}
```

- `delay` - 애니메이션에 지연시간을 적용합니다.
- `slideValue` - 변할 값을 정의하는 property key입니다. 이미 `ValueAnimation`을 알고 계시다면 이와 비슷한 것이라고 생각해도 좋습니다. 이 값은 UI에 대한 어떤 정보도 가지고 있지 않고, 단지 전달될 값을 업데이트할 뿐입니다.

또한 애니메이션 상태를 정의해보겠습니다. enum class를 사용해서 정의할 텐데, 원하신다면 어떤 형태로든 다르게 구현하실 수도 있습니다.

```kotlin
enum class ListItemAnimationState {
    INITIAL,
    FINAL
}
```

이제 `ListItemAnimationDefinition` 클래스에다가 `transitionDefinition`를 정의합니다. 여기서 INITIAL과 FINAL에 해당하는 애니메이션 로직을 작성합니다.

```kotlin
val definition = transitionDefinition<ListItemAnimationState> {
    state(ListItemAnimationState.INITIAL) {
        this[slideValue] = 90.dp
    }
    state(ListItemAnimationState.FINAL) {
        this[slideValue] = 0.dp
    }
    transition(
        fromState = ListItemAnimationState.INITIAL,
        toState = ListItemAnimationState.FINAL
    ) {
        slideValue using tween(
            delayMillis = delay,
            durationMillis = 300
        )
    }
}
```

위 코드를 한 번 자세히 들여다 보겠습니다.

애니메이션 상태가 INITIAL이라면 `slideValue`는 `90.dp`가 되고, FINAL상태라면 `0.dp`가 됩니다.

그 다음 부분이 굉장히 중요한데요, 애니메이션이 어떻게 동작할지를 결정하게 됩니다. 우리는 `tween`애니메이션을 사용할텐데, 이는 아래와 같은 파라미터를 받습니다.

- `durationMillis` - 애니메이션이 동작하는 시간
- `delayMillis` - 애니메이션이 시작되기 전까지 지연 시간
- `easing` - 시작과 끝 사이에 적용될 움직임의 곡선(번역이 매끄럽지 못해 원본을 표기합니다 - The easing curve that will be used to interpolate between start and end)

Jetpack Compose는 다른 애니메이션 타입들도 제공하기 때문에 관심이 있으시면 아래에서 참고해보시길 추천합니다. [https://developer.android.com/reference/kotlin/androidx/compose/animation/core/TransitionSpec](https://developer.android.com/reference/kotlin/androidx/compose/animation/core/TransitionSpec)

이제 애니메이션을 UI에 적용해봅시다.

```kotlin
@Composable
fun ItemCardView(index : Int) {
    val listItemAnimationDefinition = remember(index) {
            ListItemAnimationDefinition(300)
        }
    val listItemTransition = transition(
        definition = listItemAnimationDefinition.definition,
        initState = ListItemAnimationState.INITIAL,
        toState = ListItemAnimationState.FINAL,
    )
    Card(
        modifier = Modifier
            .height(200.dp)
            .fillMaxWidth()
            .absoluteOffset(x = listItemTransition[listItemAnimationDefinition.slideValue])
    ) {
        Text(
            text = "I'm a card from Jetpack Compose",
            textAlign = TextAlign.Center
        )
    }
}
```

위 코드에는 INITIAL 상태에서 FINAL 상태로 변화할 때 동작하는 애니메이션을 정의했고, 애니메이션이 어떻게 동작할 지 정의한 definition을 인자로 넘겼습니다.

또한 우리는 Jetpack Compose가 여러번 recomposition되는 것을 알고 있기 때문에 `remember`를 사용하여 애니메이션을 저장하고 recomposition이 일어날 때 이를 재사용 합니다.

Card를 animate하기 위해 `absoluteOffset` 함수를 사용하며 x 매개변수에 animated value를 넘겨줍니다.

```kotlin
.absoluteOffset(x =listItemTransition[listItemAnimationDefinition.slideValue])
```

이게 끝입니다. 앱을 실행시켜 결과를 볼까요?

![1_JCANITl25AC8uKXK8v30jw](https://user-images.githubusercontent.com/18481078/104181162-a444f980-5451-11eb-8149-52e03659c5af.gif)

## Additional

FloatPropKey를 사용하여 투명도 애니메이션을 추가할 수도 있습니다.

아래처럼 새로운 변수를 추가합니다.

```kotlin
val alphaValue = FloatPropKey(label = "alpha")
```

INITIAL상태와 FINAL상태에 해당하는 값을 정의해둡니다.

```kotlin
state(ListItemAnimationState.INITIAL) {
    this[slideValue] = 90.dp
    this[alphaValue] = 0.0f
}
state(ListItemAnimationState.FINAL) {
    this[slideValue] = 90.dp
    this[alphaValue] = 0.0f
}
```

이제 `transitionDefinition` 를 아래 코드처럼 넣어줍니다.

```kotlin
alphaValue using tween(
    durationMillis = 400,
    delayMillis = delay
)
```

이 alpha값으로 UI를 바꾸는 것을 잊지 마세요.

```kotlin
.alpha(listItemTransition[listItemAnimationDefinition.slideValue])
```

## Conclusion

이렇게 해서 Jetpack Compose로 애니메이션을 만들고 사용하는 방법을 알아보았습니다. Jetpack Compose에서는 멋진 애니메이션을 아주 쉽게 만들 수 있기 때문에 인상적입니다. 이게 당신에게 도움이 되셨다면 좋겠네요.

언제든 제 [트위터](https://twitter.com/a_rasul98)를 팔로우해주세요. 이 글과 관련된 어떤 질문을 하셔도 좋습니다.

읽어주셔서 감사합니다.
