---
layout: post
title: Android Rotate ImageView 90 degree whenever you touch 2 (이미지 클릭마다 에니메이션과 함께 90도씩 회전시키기)
category: Kotlin
tags: [rotate, imageView]
comments: true
---

이전 포스트에서는 사용자가 버튼을 클릭할 때 마다 이미지 뷰를 애니메이션과 함께 회전시키는 방법을 설명했다. 이미지 용량이 크더라도 버벅임 없이 자연스럾게 회전할 수 있는 방법이었다.

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    ...
    myImageView.setOnClickListener {
        val currentDegree = it.rotation
        ObjectAnimator.ofFloat(it, View.ROTATION, currentDegree, currentDegree + 90f)
            .setDuration(300)
            .start()
	}
}
```

그러나 위의 코드는 한가지 약점이 있다. 가로 세로 길이가 같을 경우 전혀 이상할 것이 없지만 둘 중 하나의 비율이 더 길 경우, 예를 들어 세로 길이가 더 길 경우 90도 회전을 하면 이미지가 일부분 짤리게 되는 현상이 발생한다.

이번 포스팅에서는 카카오톡 사진 전송시 이미지 편집 화면처럼 화면 회전시 자연스럽게 비율 조정까지 하는 방법을 알아보자.

![이미지회전GIF](https://user-images.githubusercontent.com/18481078/67613301-d172a100-f7e6-11e9-8b8d-330abd941ad6.gif)

## Coil

본격적인 코드를 보기에 앞서 한 가지 알아두어야 할 것이 있는데, 바로 [Coil](https://github.com/coil-kt/coil)이다. Coil은 코틀린 코루틴으로 작동하는 가벼운 image loading 라이브러리이며 기타 설정 dsl을 지원하고있다. 코루틴을 사용하지만 라이브러리 자체에 코루틴이 내장되어있으므로 따로 코루틴을 설치할 필요는 없다.

```kotlin
// 예시
image.load(
    "https://www.91-img.com/pictures/133188-v4-oppo-f11-mobile-phone-large-1.jpg"
) {
    crossfade(true)
    placeholder(R.drawable.image)
}
```

## 1. 이미지뷰와 회전 버튼

가장 먼저 이미지를 띄울 `ImageView`와 화면 회전을 위한 `Button`이 필요하다. 이 과정은 다들 아실거라 생각하고 생략한다.

## 2. 디바이스 전체 크기 구하기

화면을 얼마든지 회전시키더라도 이미지의 가로 혹은 세로 길이 중에서 더 긴 부분이 무조건 화면안에 꽉 차야한다. 짤리는 부분이 없어야하기 때문에 미리 디바이스의 가로, 세로 길이를 구해놓고 이미지 회전 마다 이를 이용해야 한다. 예를 들어 세로가 더 긴 이미지에서 오른쪽으로 90도 회전 시킬 경우 이미지뷰의 세로 높이를 디바이스 가로 길이로 서서히 맞추면서 동시에 회전시킬 것이다.

디바이스의 가로 세로 pixel을 구하는 코드는 아래와 같다.

```kotlin
...
val displayMetrics = DisplayMetrics()
windowManager.defaultDisplay.getMetrics(displayMetrics)

val deviceWidth = displayMetrics.widthPixels
val deviceHeight = displayMetrics.heightPixels
...
```

## 3. 버튼 클릭시 회전시키기

해당 부분은 코드부터 보자.

```kotlin
button.setOnClickListener {
    val currentRotation = imageView.rotation // 이미지뷰가 회전되어있는 각도가 몇 인가
    val currentImageViewHeight = imageView.height

    //  gap = 목표 길이 - 현재 길이
    val heightGap = if (currentImageViewHeight > deviceWidth) {
        deviceWidth - currentImageViewHeight
    } else {
        deviceHeight - currentImageViewHeight
    }

    if (currentRotation % 90 == 0.toFloat()) { // 애니메이션 도중 중복 클릭 방지
        // 현재길이를 시작으로 gap만큼 더해준다.
        ValueAnimator.ofFloat(0f, 1f).apply {
            duration = 500
            addUpdateListener {
                val animatedValue = it.animatedValue as Float
                imageView.run {
                    layoutParams.height =
                        currentImageViewHeight + (heightGap * animatedValue)
                            .toInt()
                    rotation = currentRotation + 90 * animatedValue
                    requestLayout()
                }
            }
        }.start()
    }
}
```

핵심은 `heightGap`이다. 우리는 이미지뷰의 height를 기준으로 기능을 동작시킬 것이다. 세로가 더 긴 이미지에서 90도 회전 시킬 경우, 세로 길이를 디바이스의 가로 길이만큼으로 줄여야 하는데, 이미지의 세로(높이)가 100이고 디바이스 가로(너비) 길이가 70이라고 한다면 이미지의 높이를 30만큼 줄여나가야 한다. 목표하는 길이 까지의 gap은 -30인 것이다.

이와 반대로 옆으로 길게 누워있는 이미지라면, 90도 회전 시 imageView의 길이가 디바이스의 높이와 같게 늘어나야 한다. 이 상황은 옆으로 누워있는 이미지 뷰의 높이(70)가 디바이스의 가로 길이(70)와 동일할 것이므로, 회전과 함께 이미지뷰의 길이(70)를 디바이스 세로 길이(100)로 만들어 주어야 한다. 즉, 누워있는 이미지뷰의 높이(70)와 회전 이후 목표 길이(100)인 deviceHeight의 차이가 gap이 된다. 목표하는 길이 까지의 gap은 +30이다.

따라서 최종 식은 아래와 같아야 한다.

```
imageView.height = imageView.height + gap
```

gap을 0부터 서서히 30까지 만들면 자연스러운 애니메이션이 될 것이다.

회전 애니메이션과 함께 현재 이미지의 높이를 시작으로, gap만큼 크기를 변경시킬 것이다. gap은 위에서 말했듯이 상황에 따라 +일수도 -일수도 있다.

#### ValueAnimator

ValueAnimator를 사용해서 애니메이션을 적용했다. `ValueAnimator.ofFloat(0f, 1f)`를 사용하면 원하는 시간(duration)동안 0f 부터 1f까지 서서히 변하는 값을 얻을 수가 있다. 0f부터 시작해서 최종적으로 1f가 될 때 이미지는 90도 회전이 되어있을 것이고, 이미지뷰의 높이는 원하는 길이만큼 조정되어있을 것이다.

전체 코드는 [git Gist](https://gist.github.com/wooooooak/f958b6c50e667fd54b985e2b0b9138e0)에서 확인 할 수 있다.

### 마무리

위 코드는 실제 bitmap이 변경되지 않는다. bitmap연산의 경우 많은 비용이 들기 때문에 사용자의 편의성을 위해 imageView만 회전시킨 것이다. 비트맵 자체를 회전시키는 코드는 간단하지만 자연스러운 애니메이션을 원한다면 결국 imageView에 애니메이션을 걸어주는 코드도 반드시 필요하다.

앞선 포스트에서도 설명했지만, 비트맵 연산이 필요한 경우라면 사용자가 저장 버튼을 누르거나, 화면을 나갈 때 등등 필요한 경우에만 해야한다고 생각한다. 필자의 경우에는 사용자가 회전 버튼을 누를때 마다, 각 이미지가 몇 번 회전 되었는지만 저장해놓고, 필요한 순간에만 (사용자가 버튼을 누른 횟수 \* 90도)를 계산하여 비트맵을 조작하여 성능과 사용자 경험에 큰 이득을 얻었다.
