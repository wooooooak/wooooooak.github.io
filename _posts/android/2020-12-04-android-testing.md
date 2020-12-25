---
layout: post
title: Android Testing을 위한 사전 지식
category: Android
tags: [testing, AndroidX test]
comments: true
---

## Local test vs Instrumented test

안드로이드 프로젝트를 만든 후 source sets을 살펴보면 main, test, androidTest 폴더들이 보인다.

`main` - 프로덕트 앱을 구성하는 코드들을 넣는 곳이다.

`test` - [local test](https://developer.android.com/training/testing/unit-testing/local-unit-tests)라고 불리는 test를 담는 곳이다. local test는 오직 개발 장비의 JVM에서 돌아가게 되며 실제 디바이스나 에뮬레이터에서 테스트 하지 않아도 되는 것들을 테스트 한다. 기기 부팅이나 에뮬레이터를 띄우지 않기 때문에 **run 속도가 빠르지만**, android 앱개발을 하는데 android 기기에서 테스트 해보지 않는다는 점에 있어서 정확도가 떨어질 수도 있다.

`android test` - [Instrumented test](https://developer.android.com/training/testing/unit-testing/instrumented-unit-tests)라고 불리는 test를 담는 곳이다. Instrumented test는 실제 기기나 에뮬레이터에서 진행되기 때문에 실제 환경에서 테스트한다는 장점이 있지만 **local test에 비해 많이 느린 편이다.**
참고로 CI서버에서 테스트 자동화가 이뤄진다면, Instrumented test를 위해 에뮬레이터 혹은 실 기기를 서버에 연결해둬야 한다.

잘 설계된 아키텍쳐를 기반으로 앱을 설계하게되면, 굳이 Android framework class에 의존하지 않는 layer들이 있다. view에 표현될 data들을 가지고 로직적인 장난만 치는 viewModel, 앱의 핵심 기능을 담은 domin layer의 클래스 등등 여러 객체들은 local에서 테스트 해도 문제가 없으며 오히려 권장된다.

`Activity`나 `Service`와 같은 Android Framework Component 자체를 테스트 할 경우 때에따라 Instrumented test로 해야만 할 수도 있지만, 단지 그들을 의존하고 있다면 의존 객체를 mocking하여 local test로 돌릴 수 있다.

## [AndroidX Test Library](https://developer.android.com/training/testing/set-up-project)

순수한 ViewModel은 android framework가 제공하는 어떤 것도 의존하지 않는다. 그러나 어떤 불가피한 이유로 viewModel이 Application `context`에 의존한다면 테스트 코드에서는 이를 어떻게 주입해야할까?

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

AndroidX Test는 simulated android environment에서 테스트 전용 클래스와 메서드들을 제공해준다. simulated 환경이기 때문에 실제 안드로이드 기기의 환경이 아니다. simulated android environment를 만들어주는 것이 Robolectric이다. 예전에는 local test에서 `context`를 얻기 위해서 직접 Robolectric을 사용했지만 AndroidX Test 라이브러리 덕분에 직접 사용할 일은 없어졌다.
참고로 Instrumented test에서도 AndroidX Test Library의 같은 메서드를 사용하여 `context`를 얻을 수 있는데, 이때는 simulated environment가 아닌 부팅된 실제 기기의 Application `context`를 반환한다.

### AndroidJUnit4::class

[AndroidJUnit4](https://developer.android.com/training/testing/junit-runner#kotlin)는 test runner다. 즉 테스트를 실행하는 주체다. junit은 이런 runner가 없이는 테스트가 실행되지 않으며, runner를 따로 지정해주지 않으면 기본 제공하는 runner로 실행된다. `@RunWith`를 사용하여 Runner를 교체할 수 있다.

AndroidJUnit4는 AndroidX Test Library가 Local test와 Instrumented Test에서 서로 다르게 동작할 수 있도록 도와준다. `Context`를 얻을 때, local test에서는 `simulated context`를 제공하고, instrumeted test에서는 실제 `context`를 제공할 수 있는 이유가 AndroidJUnit4 Runner 덕분이다. 따라서 AndroidJUnit4 test runner없이 AndroidX Test를 사용하면 제대로 동작하지 않을 가능성이 크다. AndroidX Test 라이브러리를 사용할 땐 AndroidJUnit4 라이브러리를 사용하자.

## Rule

JUnit에는 [JUnit Rule](https://junit.org/junit4/javadoc/4.12/org/junit/Rule.html)이라는 것이 있다. 테스트를 작성하다보면 여러 테스트 클래스에서 테스트 사전 작업, 직후 작업이 동일할 때가 있다. 즉 코루틴을 사용하는 대부분의 테스트 클래스에서는 @Before에서 Main Dispatcher를 바꾸고, @After에서 되돌려 놓는다. 이런 작업을 하나의 Rule로 만들어 놓으면 @Before, @After마다 자동으로 수행되어 보일러 플레이트 코드를 줄일 수 있다.
`@get:Rule` annotation을 붙여 사용한다.

```kotlin
@get:Rule
val mainCoroutineRule = MainCoroutineRule()
```

참고로 AndroidX test에는 [ActivitySenarioRule](https://developer.android.com/reference/androidx/test/ext/junit/rules/ActivityScenarioRule), [ServiceTestRule](https://developer.android.com/reference/androidx/test/rule/ServiceTestRule)등과 같은 유용한 Rule들을 제공한다.

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

위와 코드가 있을때 주의할 점은, [postValue](https://developer.android.com/reference/android/arch/lifecycle/LiveData#postvalue)이 백그라운드 Thread에서 일을 수행한다는 것이다.
`MyViewModelTest.kt` 에서는 `myViewModel.method1()` 수행 이후 liveData의 value가 제대로 변했는지 확인하고자 할 것이다. 그러나 백그라운드에서 작업으로인한 비동기 처리 때문에 값이 변경되기도 전에 테스트가 끝나버려 실패하게 된다. 이럴때 사용하는 것이 [InstantTaskExecutorRule](https://developer.android.com/reference/androidx/arch/core/executor/testing/InstantTaskExecutorRule)이다.

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

이번에는 코루틴 테스트를 해보자

```kotlin
class MyViewModel: ViewModel() {
    var name: String = "hello"
        private set

    fun changeName(newName: String) = viewModelScope.launch {
        name = newName
    }
}
```

위의 코드는 default Dispatcher가 Dispatchers.main으로 설정된 `viewModelScope.launch`로 코루틴을 실행한다. 이 코드를 테스트하기 위해 아래와 같은 테스트 코드가 작성될 것이다.

```kotlin
class MyViewModelTest: ViewModel() {
   @Test
   fun changeNameTest() = runBlocking {
       val myViewModel = MyViewModel()
       val newName = "world"
       myViewModel.changeName(newName)
       Assert.assertEquals(myViewModel.name, newName)
   }
}
```

테스트를 실행하면 에러가 난다.

```
Exception in thread "main @coroutine#1"
java.lang.IllegalStateException: Module with the Main dispatcher had failed to initialize.
For tests Dispatchers.setMain from kotlinx-coroutines-test module can be used
```

`InstantTaskExecutorRule` 설명에서도 나왔듯이 `viewModelScope.launch`가 사용하는 Dispatcher.main은 Android main looper를 사용하기 때문에 local test에서는 사용할 수 없다. 이럴땐 [kotlinx-coroutines-test](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-test/)에서 제공하는 [TestCoroutineDispatcher](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-test/kotlinx.coroutines.test/-test-coroutine-dispatcher/)를 사용하여 해결할 수 있다. `kotlinx-coroutines-test`는 coroutine testing 전용으로 만들어진 라이브러리이다.

```kotlin
@ExperimentalCoroutinesApi
class MyViewModelTest {
    private val testDispatcher = TestCoroutineDispatcher()

    @Before
    fun setup() {
        Dispatchers.setMain(testDispatcher)
    }

    @After
    fun tearDown() {
        //  원래의 상태로 되돌려 놓는다.
        Dispatchers.resetMain()
        // 테스트가 끝났으니 혹시 모를 실행중인 작업을 clean up 시켜준다.
        testDispatcher.cleanupTestCoroutines()
    }

    @Test
    fun testSomething() = runBlockingTest {
        ...
    }
}
```

모든 테스트 클래스마다 작성하기 귀찮으므로 `MainCoroutineRule`을 만들어서 사용하자.

```kotlin
@ExperimentalCoroutinesApi
class MainCoroutineRule(
    val testDispatcher: TestCoroutineDispatcher = TestCoroutineDispatcher()
) : TestWatcher() {

    override fun starting(description: Description?) {
        super.starting(description)
        Dispatchers.setMain(testDispatcher)
    }

    override fun finished(description: Description?) {
        super.finished(description)
        Dispatchers.resetMain()
        testDispatcher.cleanupTestCoroutines()
    }
}

@ExperimentalCoroutinesApi
fun MainCoroutineRule.runBlockingTest(block: suspend () -> Unit) =
    this.testDispatcher.runBlockingTest {
        block()
    }
```

TestCoroutineDispatcher를 사용하려면 kotlinx-coroutines-test 라이브러리를 추가해야한다.

```groovy
testImplementation "org.jetbrains.kotlinx:kotlinx-coroutines-test:$version"
```

## runBlockingTest

바로 위의 코드에서 `runBlockingTest()` 함수를 사용했다. [kotlinx-coroutines-test](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-test/kotlinx.coroutines.test/run-blocking-test.html)는 coroutines test library에서 제공되는 함수로써 특별한 점이 있다.

1. 코드 블럭을 특별한 coroutine context에서 실행하여 동기적으로 즉각 실행한다.
2. delay() 함수가 사용된다면, 실제 delay 시간 만큼 기다리지 않고 테스트를 실행한다. 즉 모든 pending task들을 즉각 실행하며 가상의 clock-time을 딜레이된 시간으로 조정한다.

즉 테스트 코드에서는 대부분 실제로 delay만큼 기다릴 필요가 없기 떄문에 `runBlockingTest()` 함수를 사용하여 테스트 코드 시간을 단축 시킬 수 있다.

## 참고자료

- [Testing Android Applications - Kayvan Kaseb](https://medium.com/kayvan-kaseb/testing-android-applications-b95a4a72f2f9)
- [Kotlin Coroutines in Android — Unit Test](https://medium.com/swlh/kotlin-coroutines-in-android-unit-test-28ff280fc0d5)
- [Codelabs - Advanced Android in Kotlin: Testing Basics](https://developer.android.com/codelabs/advanced-android-kotlin-training-testing-basics#0)
- [kotlinx.coroutines](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-test/)
- [Test apps on Android - google developer](https://developer.android.com/training/testing/fundamentals)
- [architecture-samples - github](https://github.com/android/architecture-samples)
- [Testing Coroutines — Dispatchers - Ralf Stuckert](https://medium.com/@ralf.stuckert/testing-coroutines-dispatchers-956969cfd374)
- [Testing Coroutines — Introduction - Ralf Stuckert](https://medium.com/@ralf.stuckert/testing-coroutines-introduction-4ab6733c7599)

## 끄

읕
