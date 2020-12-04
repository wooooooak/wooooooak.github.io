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

### AndroidJUnit4::class

[AndroidJUnit4](https://developer.android.com/training/testing/junit-runner#kotlin)는 test runner다. 즉 테스트를 실행하는 주체다. junit에는 이런 runner가 없이는 테스트가 실행되지 않으며, runner를 따로 지정해주지 않으면 기본제공하는 runner로 실행된다. RunWIth를 사용하여 Runner를 교체할 수 있다.

AndroidJUnit4는 AndroidX Test Library가 Local test와 Instrumented Test에서 서로 다르게 동작할 수 있도록 도와준다. Context를 얻을 때, local test에서는 simulated context를 제공하고, instrumeted test에서는 실제 context를 제공할 수 있는 이유가 AndroidJUnit4 Runner 덕분이다. 따라서 AndroidJUnit4 test runner없이 AndroidX Test를 사용하면 제대로 동작하지 않을 가능성이 크다. 따라서 AndroidX Test 라이브러리를 사용할 땐 AndroidJUnit4 라이브러리를 사용하자.

## Rule

JUnit에는 [JUnit Rule](https://junit.org/junit4/javadoc/4.12/org/junit/Rule.html)이라는 것이 있다. 테스트를 작성하다보면 여러 테스트 클래스에서 테스트 사전 작업, 직후 작업이 동일할 때가 있다. 즉 코루틴을 사용하는 대부분의 테스트 클래스에서는 @Before에서 Main Dispatcher를 바꾸고, @After에서 되돌려 놓는다. 이런 작업을 하나의 Rule로 만들어 놓으면 @Before, @After마다 자동으로 수행되어 보일러 플레이트 코드를 줄일 수 있다.
`@get:Rule` annotation을 붙여 사용한다.

```kotlin
@get:Rule
val mainCoroutineRule = MainCoroutineRule()
```

참고로 AndroidX test에는 ActivitySenarioRule, ServiceTestRule등과 같은 유용한 Rule들을 제공한다.

### InstantTaskExecutorRule

[LiveData](https://developer.android.com/topic/libraries/architecture/livedata)를 사용하는 ViewModel 테스트를 진행한다고 해보자.

```kotlin
class MyViewModel: ViewModel() {
    private val _myLiveData = MutableLiveData<String>()
    val myLiveData: LiveData<String> = _myLiveData

    fun method1() {
        myLiveData.postValue("value")
    }
}
```

위와 코드가 있을때 주의할 점은, postValue이 백그라운드 Thread에서 일을 수행한다는 것이다.
MyViewModelTest.kt 에서는 `myViewModel.method1()` 수행 이후 liveData의 value가 제대로 변했는지 확인하고자 할 것이다. 그러나 백그라운드에서 작업으로인한 비동기 처리 때문에 값이 변경되기도 전에 테스트가 끝나버려 실패하게 된다. 이럴때 사용하는 것이 [InstantTaskExecutorRule](https://developer.android.com/reference/androidx/arch/core/executor/testing/InstantTaskExecutorRule)이다.

[InstantTaskExecutorRule](https://developer.android.com/reference/androidx/arch/core/executor/testing/InstantTaskExecutorRule) : 모든 [Architecture Components](https://developer.android.com/topic/libraries/architecture)-related background 작업을 백그라운드에서가 아닌 동일한 Thread에서 돌게하여 동기적인 처리가 가능하도록 해준다.

따라서 LiveData를 테스팅한다면 이 Rule을 적용하자.

#### local test에서 주의할 점

local test에서는 postValue가 아닌 setValue를 사용한다 하더라도 InstantTaskExecutorRule을 적용하지 않는다면 여전히 테스트는 실패한다. 에러는 아래와 같다.

```
java.lang.NullPointerException
	at androidx.arch.core.executor.DefaultTaskExecutor.isMainThread(DefaultTaskExecutor.java:77)
	at androidx.arch.core.executor.ArchTaskExecutor.isMainThread(ArchTaskExecutor.java:116)
	at androidx.lifecycle.LiveData.assertMainThread(LiveData.java:461)
	at androidx.lifecycle.LiveData.setValue(LiveData.java:304)
	at androidx.lifecycle.MutableLiveData.setValue(MutableLiveData.java:50)
    ...
```

`LiveData.setValue()`는 내부적으로 `isMailThread()`함수를 사용하여 현재 쓰레드가 메인 쓰레드인지를 확인한다. local test에서는 당연하게도 real Android환경이 아니기 때문에 MainThread(=UiThread)를 사용하지 못한다(Android MainThread는 android.os의 Looper를 사용). 그래서 에러가 난다.

그렇다면 `InstantTaskExecutorRule`을 사용하더라도 MainThread(UiThread)를 사용하는 건 아니니까 여전히 에러가 나야하지 않을까?라는 의문이 떠오른다. 그래서 Rule의 내부를 타고 들어가보니 `isMainThread()`가 `true`로 하드코딩 되어있는걸 확인할 수 있었다.

```java
/**
 * A JUnit Test Rule that swaps the background executor used by the Architecture Components with a
 * different one which executes each task synchronously.
 * <p>
 * You can use this rule for your host side tests that use Architecture Components.
 */
public class InstantTaskExecutorRule extends TestWatcher {
    @Override
    protected void starting(Description description) {
        super.starting(description);
        ArchTaskExecutor.getInstance().setDelegate(new TaskExecutor() {
            @Override
            public void executeOnDiskIO(Runnable runnable) {
                runnable.run();
            }

            @Override
            public void postToMainThread(Runnable runnable) {
                runnable.run();
            }

            @Override
            public boolean isMainThread() {
                return true;
            }
        });
    }

    @Override
    protected void finished(Description description) {
        super.finished(description);
        ArchTaskExecutor.getInstance().setDelegate(null);
    }
}
```

결론은 InstantTaskExecutorRule을 써야한다는 것이다.

### 코루틴 테스트를 위한 MainCoroutineRule
