---
layout: post
title: Android Testing을 위한 사전 지식
category: Android
tags: [testing, AndroidX test]
comments: true
---

## local test vs instrumented test

안드로이드 프로젝트를 만든 후 source sets을 살펴보면 main, test, androidTest 폴더들이 보인다.

`main` - 흔히 우리가 프로덕트 앱을 구성하는 코드들을 넣는 곳이다.

`test` - [local test](https://developer.android.com/training/testing/unit-testing/local-unit-tests)라고 불리는 test를 담는 곳이다. local test는 오직 개발 장비의 JVM에서 돌아가게 되며 실제 디바이스나 에뮬레이터에서 테스트 하지 않아도 되는 것들을 테스트 한다. 기기 부팅이나 에뮬레이터를 띄우지 않기 때문에 run 속도가 빠르지만, android 앱개발을 하는데 android 기기에서 테스트 해보지 않는다는 점에 있어서 정확도가 떨어질 수도 있다.

`android test` - [Instrumented test](https://developer.android.com/training/testing/unit-testing/instrumented-unit-tests)라고 불리는 test를 담는 곳이다. Instrumented test는 실제 기기나 에뮬레이터에서 진행되기 때문에 실제 환경에서 테스트한다는 장점이 있지만 local test에 비해 많이 느린 편이다.
참고로 CI서버에서 테스트 자동화가 이뤄진다면, Instrumented test를 위해 에뮬레이터 혹은 실 기기를 서버에 연결해둬야 한다.

잘 설계된 아키텍쳐를 기반으로 앱을 설계하게되면, 굳이 Android framework class에 의존하지 않는 layer들이 있다. view에 표현될 model들을 가지고 로직적인 장난만 치는 viewModel, 앱의 핵심 기능을 담은 domin layer의 클래스 등등 여러 객체들은 local에서 테스트 해도 문제가 없으며 오히려 권장된다.

## [AndroidX Test Library](https://developer.android.com/training/testing/set-up-project)

순수한 ViewModel은 android framework가 제공하는 어떤 것도 의존하지 않는다. 그러나 어떤 불가피한 이유로 viewModel에 android의 context가 필요하다면 어떡해야할까?

[AndroidX Test Library](https://developer.android.com/training/testing/set-up-project)가 이를 해결해준다. AndrodX Test Library는 테스트 전용 Component(Application, Activity 등)들과 메서드들을 제공해준다. local test중에 테스트용 Android framework class들이 필요하다면 이를 사용하자.

AndroidX Test Library를 사용하기 위해서는 아래와 같은 절차가 필요하다.

1. AndroidX Test core 와 ext 라이브러리 추가
2. [Robolectric](http://robolectric.org/) Testing Library 추가
3. 사용하는 테스트 클래스에 **@RunWith(AndroidJUnit4::class)** 추가

```gradle
    // AndroidX Test - JVM testing
testImplementation "androidx.test.ext:junit-ktx:$androidXTestExtKotlinRunnerVersion"
testImplementation "androidx.test:core-ktx:$androidXTestCoreVersion"
testImplementation "org.robolectric:robolectric:$robolectricVersion"
```

잠깐, Robolectric과 @RunWith(AndroidJUnit4::class)는 왜 추가할까?

### [Robolectric](http://robolectric.org/)이란?

AndroidX Test는 simulated android environment에서 테스트 전용 클래스와 메서드들을 제공해준다. simulated 환경이기 때문에 실제 안드로이드 기기의 환경이 아니다. simulated android environment를 만들어주는 것이 Robolectric이다. 예전에는 local test에서 context를 얻기 위해서 Robolectric을 직접 사용했지만 AndroidX Test 라이브러리 덕분에 직접 사용할 일은 없어졌다.
참고로 Instrumented test에서도 AndroidX Test Library의 같은 메서드를 사용하여 context를 얻을 수 있는데, 이때는 simulated environment가 아닌 부팅된 실제 기기의 Application context를 반환한다.

### @RunWith(AndroidJUnit4::class)

[AndroidJUnit4](https://developer.android.com/training/testing/junit-runner#kotlin)는 test runner다. 즉 테스트를 실행하는 주체다. junit에는 이런 runner가 없이는 테스트가 실행되지 않으며, runner를 따로 지정해주지 않으면 기본제공하는 runner로 실행된다. RunWIth를 사용하여 Runner를 교체할 수 있다.

AndroidJUnit4는 AndroidX Test Library가 Local test와 Instrumented Test에서 서로 다르게 동작할 수 있도록 도와준다. Context를 얻을 때, local test에서는 simulated context를 제공하고, instrumeted test에서는 실제 context를 제공할 수 있는 이유가 AndroidJUnit4 Runner 덕분이다. 따라서 AndroidJUnit4 test runner없이 AndroidX Test를 사용하면 제대로 동작하지 않을 가능성이 크다. 따라서 AndroidX Test 라이브러리를 사용할 땐 AndroidJUnit4 라이브러리를 사용하자.
