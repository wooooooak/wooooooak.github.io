---
layout: post
title: TestCoroutineDispatcher 제어하기
category: Android
tags: [testing, coroutine-testing, kotlin]
comments: true
---

[Android Testing을 위한 사전 지식](https://wooooooak.github.io/android/2020/12/04/android-testing/)에서 어떻게 android code를 테스트하는지 살펴보았다. 주로 안드로이드 OS 특성과 코루틴의 특징들로 인해 주의해야할 점들을 알아보았다.

이번 포스트에서는 코루틴을 테스트 할 때 필요한 몇가지 기능들을 더 알아보자.

### Progress bar 상태 변경 테스트

보통 ViewModel에서 Repository 혹은 UseCase로 데이터를 요청한다. 네트워크나 DB 조회로 인한 시간이 걸리기 때문에 그동안 Progress bar를 띄워주는 건 보편적이다.

```kotlin
class MyViewModel(): ViewModel() {

    private val _isLoading = MutableLiveData<Boolean>()
    val isLoading: LiveData<Boolean> = _isLoading

    fun loadData() = {
        _isLoading.value = true // TRUE 인지 테스트
        viewModelScope.launch {
            repository.fetchData() // 호출 되었는지 테스트
            _isLoading.value = false // FALSE 인지 테스트
        }
    }
}
```

`loadData()`라는 간단한 함수를 테스트 하기 위해서는 주석에 적혀 있듯 3가지를 테스트 한다.

1. isLoading이 true인지 확인
2. loadData가 호출되었는지 확인
3. isLoading이 false인지 확인

테스트 코드는 대략 아래와 같이 시작될 것이다.

```kotlin
class MyViewModelTest() {
    private val testDispatcher = TestCoroutineDispatcher()
    ...
    fun test_loadData() = runBlockingTest {

        // when
        myViewModel.loadData()

        // then
        // 1. isLoading == true인지 확인
        // 2. loadData가 호출되었는지 확인
        // 3. isLoading == false인지 확인
    }
}
```

then에서 각자 선호하는 assertion 라이브러리를 사용해 원하는 값이 매칭되는지를 확인한다.

하지만 then 주석을 풀고 코드를 작성한 후 실행하면 첫 `isLoading`이 `false`이기 때문에 테스트는 실패한다.

**`TestCoroutineDispatcher`는 코루틴 작업들을 즉각적으로 실행하고 완료하게 설계되어있다. 즉 assertion 코드를 만나기 전에 loadData의 실행이 모두 끝난다는 것이다.** 대부분의 경우에는 이같은 특성이 테스트를 빠르게, 쉽게, 가독성 좋게 해주지만 위 코드 처럼 **코루틴 중간에 변경되는 값을 테스트할 때는 난감하다.**

이럴 땐 `TestCoroutineDispatcher`의 [pauseDispatcher](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-test/kotlinx.coroutines.test/-delay-controller/pause-dispatcher.html)와 [resumeDispatcher](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-test/kotlinx.coroutines.test/-delay-controller/resume-dispatcher.html)를 사용하여 Dispatcher 타이밍을 제어할 수 있다.

`pauseDispatcher`는 `TestCoroutineDispatcher`를 멈춰놓음으로써 새로운 코루틴이 즉각 실행되지 않도록 막는다(단지 queue에 들어가서 대기한다). 따라서 `viewModel.loadData()`처럼 새로운 코루틴을 실행하는 함수를 테스트할 때 유용하게 쓰인다. 새로운 코루틴이 실행되기 전에 assertion을 할 수도 있고 필요한 setup을 해놓을 수도 있다.

`resumeDispatcher`는 paused Dispatcher를 resume시킨다.

따라서 아래와 같이 테스트 코드를 작성하면 isLoading 상태를 확인할 수 있다.

```kotlin
class MyViewModelTest() {
    private val testDispatcher = TestCoroutineDispatcher()
    ...
    fun test_loadData() = runBlockingTest {
        pauseDispatcher() // 새로운 코루틴은 실행되지 않고 준비만 한다.

        // when
        myViewModel.loadData()

        // then
        check(viewModel.isLoading.getOrAwaitValue() == true)

        // 아까 대기시켰던 코루틴을 resume시킨다.
        // repository.loadData() 와 _isLoading.value = false가 실행
        resumedDispatcher()

        isCalled(repository.fetchData()) // repository의 fecthData가 호출되었는지 확인
        check(viewModel.isLoading.getOrAwaitValue() == false)
    }
}
```
