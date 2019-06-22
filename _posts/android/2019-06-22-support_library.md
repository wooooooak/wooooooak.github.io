---
layout: post
title: Android support library(서포트 라이브러리 이해하기)
category: Android
tags: [libaray, support-library]
comments: true
---

support 라이브러리는 표준 프레임워크 API에서 사용할 수 없었던 손쉬운 개발 및 여러 기기에 걸친 지원을 위한 추가 편의 클래스 및 기능을 제공한다.

서포트 라이브러리 버전에 따라 다르긴 하지만 손 쉽게 Material 디자인을 사용할 수도, RecyclerView를 사용할 수도, 표준에서 제공되는 클래스의 업데이트된 버전을 사용할 수도 있게 해준다.

## 지원 라이브러리의 용도

**가장 중요한 것은, 이전 버전에 대한 하위 호환성문제 해결이다.** 예를 들어 안드로이드 버전이 올라가면서 새로운 API가 나왔다면, 그 이전 버전에서도 새로운 API를 사용 할 수 있도록 지원해주는 것이다. 공식 문서에서는 Fragment를 예로 설명한다. Fragment는 API Level 11(Android 3.0)에서 표준 기능으로 새롭게 추가된 뷰인데, API Level 10 이하에서도 Fragment를 사용할 수 있게 도와준다. 서포트 라이브러리가 없었다면, 코드에서 API레벨을 확인하고, API Level 11 이상인 경우와 그렇지 않은 경우를 나누어 줘야 했을 것이다.

또한 표준은 아니지만 RecyclerView같은 편의 클래스들도 지원해준다.

자세한 내용은 [공식 문서](https://developer.android.com/topic/libraries/support-library/features?hl=ko)를 참고하자.

## minSdk는 14로!

원래는 support-v4와 support-v7과 같이 _v# 표기법_ 으로 지원하는 최소 API 레벨을 나타냈다고 한다. v4는 API Level 4 이상 기기부터 지원하고, v7는 API Level 7 이상 기기부터 서포트 패키지를 지원했다고 한다. 그러나 support libarary 26.0.0(2017년 7월 버전)부터 지원되는 모든 API 레벨이 최소 Android 4.0(API레벨 14)로 변경되었다. 따라서 현재는 v#에 적혀있는 버전이 큰 의미가 없으며 v4 와 v7라이브러리 패키지는 동일한 최소 API 수준(API Level 14)을 지원한다.

## 출시 버전

`implement 'com.[android.support](http://android.support):appcompat-v7:26.1.0'` 와 같이 v# 뒤에 출시 버전을 적어준다. `implement 'com.android.support:appcompat-v7:26.1.0'` 의 경우 이 서포트 라이브러리가 빌드된 API 버전을 나타낸다. 즉 당신이 사용하려고 하는 서포트라이브러리 버전이 26.1.0이라면, 이 라이브러리는 API Level 26에서 빌드해보며 만든 라이브러리이니 참고하라는 것이다. 이 말은 곧 당신이 API Level 26을 타겟으로 앱을 만들고 있다면, 문제없이 사용 가능하다는 것이다. 물론 API Level 26 이하, 14 이상의 기기도 하위 호환성이 어느정도 보장된다는 뜻이다. **다만 27 이상을 타겟으로 할 경우는 새롭게 나온 API 버전에 출시된 모든 기능과 호환될 것이라고 가정하면 안된다.** 만약 당신이 가장 최근 API Level인 Pie(API Level 28)을 타겟으로 앱을 만들다면, 서포트 라이브러리 버전을 28.0.0 으로 바꾸는 것을 고려해보아야 할것이다.

참고로 [android.support](http://android.support) 패키지 형태로 배포하는 버전은 28.0.0이 마지막이다. 28 이상의 API Level부터는 모두 AndroidX 형태로 제공할 것이라고 한다. 새로운 프로젝트라면 androidx를 사용할 것을 권장하고 있고, 심지어 기존에 android.support로 사용했던 패키지도 모두 androidx로 migration할 것을 권장하고있다. 아마 androidx가 Jetpack의 구성요소를 포함하고 있기 때문에 사용을 장려하는 것으로 보인다.
