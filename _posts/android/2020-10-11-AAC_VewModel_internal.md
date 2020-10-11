---
layout: post
title: Jeptpack ViewModel 내부 동작 원리
category: Android
tags: [ViewModel, Jetpack]
comments: true
---

# Jetpack ViewModel 내부 동작 원리

## Jetpack ViewModel을 사용하는 이유

화면 회전과 같은 configuration change가 발생하면 activity instance는 죽고, 다시 새로운 activity instance가 생성된다. instance의 hashcode 값 조차 다르다.

구글이 제공하는 `ViewModelProvider`를 사용하여 `viewModel`을 만들고 액티비티에 두면, 액티비티가 재생성 되어도 `viewModel`은 재생성되지 않고 유지된다.

## 뷰모델 생성 방법

```kotlin
// In Activity

viewModel = ViewModelProvider(this, MyViewModelFactory()).get(CounterViewModel::class.java)
```

보통 `ViewModelProvider`에서 `get`함수를 사용해 viewModel객체를 얻어온다.

## ViewModelProvider 생성자

![ViewModelProvider 생성자](https://user-images.githubusercontent.com/18481078/95671579-c8cfd200-0bd3-11eb-846e-75d2a578efe3.png)

`ViewModelStoreOwner`만 받는 생성자와, `ViewModelStoreOwner`와 `factory` 두 개를 받는 생성자가 존재한다.
뷰모델에 파라미터가 없다면 `factory`를 넘기지 않는데, 이럴땐 `ViewModelProvider`내부적으로 Default Factory를 사용한다.

**ViewModelStoreOwner**는 getViewModelStore()메서드 하나만 가진 인터페이스이다.

![image](https://user-images.githubusercontent.com/18481078/95671588-fc126100-0bd3-11eb-9398-b9a82d27ec6c.png)

우리는 주로 뷰모델 프로바이더 생성자의 첫번째 인자로 activity를 넣어준다. `ViewModelStoreOwner`타입으로 Activity를 넣어준다는 의미인데, 이때는 꼭 `ComponentActivity`의 child 클래스로 구현된 Activity를 넣어주어야 한다. `ComponentActivity`가 `ViewModelStoreOwner`를 구현하고 있기 때문이며 대표적으로 `FragmentActivity`와, 그의 하위 클래스인 `AppCompatActivity`가 있다.

즉, 우리의 Activity에는 이미 `ViewModelStore`를 제공해주는 책임이 구현되어 있었다고 볼 수 있다.

ViewModelProvider 생성자는 파라미터로 받은 값들을 class member로 저장해놓는다.

![image](https://user-images.githubusercontent.com/18481078/95671601-13e9e500-0bd4-11eb-828d-f174e81b3cba.png)

# 뷰모델 객체 얻기

.get(MyClass::class.java)를 호출하면 아래의 함수가 호출된다.

![image](https://user-images.githubusercontent.com/18481078/95671638-514e7280-0bd4-11eb-85c0-2a3215f11308.png)

클래스의 canonical(정식) name을 가져다가, static 하게 선언된 DEFAULT_KEY 뒤에 붙여서 오버로딩된 get을 호출한다. 오버로딩된 get을 보자.

![image](https://user-images.githubusercontent.com/18481078/95671640-557a9000-0bd4-11eb-8d2c-98d359372e49.png)

이전 get함수에서 만들었던 key와 class를 넘겨받는다.
`Activity`가 만들어놓은 `ViewModelStore`를 이용하여 key에 해당하는 `viewModel`이 이미 존재하는지, 혹은 존재하지 않는지를 확인하는 로직이 들어있다. `viewModel`이 존재한다면 그 `viewModel`을 반환하지만, 그렇지 않을 경우 `factory` 객체의 도움을 받아 새로운 `viewModel`을 생성한다. 새로 생성한 `viewModel`은 `viewModelStore`에 저장해놓고 `viewModel`을 반환한다.

사실 별다른 로직은 없어 보인다. **여기까지만 보았을 때는 알 수 있는 점은, 같은 타입의 뷰 모델을 여러번 생성하려 할 경우, 중간 과정에서 만들어지는 tag가 같기 때문에 하나의 뷰 모델 객체만 생성된다는 점 정도겠다.**

아직은 어떻게 activity의 멤버 변수로 가지고 있는 `viewModel`이 Activity 재생성시 유지될 수 있는지를 알 수 없다. `viewModelStore`에 비밀이 숨겨져 있을까? 좀 더 파악해보자.

## ViewModelStore

![image](https://user-images.githubusercontent.com/18481078/95671652-7347f500-0bd4-11eb-9f57-c1d6616ea120.png)

ViewModelStore는 위의 코드가 전부이다. 비밀 로직은 없다. 단지 HashMap을 사용하여 ViewModel을 관리할 뿐이다.

그렇다면 우리가 아직 구현을 파헤치지 못한 부분이 딱 한군데 남아있게 된다. 바로 ViewModelStoreOwner 인터페이스의 `getViewModelStore()` 메서드다. ViewModelProvider의 생성자를 다시 보자.

![image](https://user-images.githubusercontent.com/18481078/95671659-7e9b2080-0bd4-11eb-8610-b508779924b5.png)

우리는 아직 ViewModelStoreOwner로 넘긴 Activity의 getViewModelStore()의 구현을 보지 못했다. 단순하게 ViewModelStore를 가져올 거라고 가볍게 생각했는데 살펴봐야 할 때가 왔다. getViewModelStore를 구현한 ComponentActivity.java 파일을 열어 확인해보자.

### ComponentActivity::getViewModelStore

![image](https://user-images.githubusercontent.com/18481078/95671664-8fe42d00-0bd4-11eb-9bb0-40ccf5827e2c.png)

`NonConfigurationInstances` 타입의 `nc`라는 객체가 보인다. 액티비티가 configuration change로 인해 재생성 되어도, `nc`객체는 소멸된 액티비티의 몇몇 멤버 변수 객체들을 가지고있다. 중간에 보이는 주석 "Restore the ViewModelStore from NonConfigurationInstances"를 보니 상태 변화의 타겟이 아닌 객체를 복구해주는 것 같다. 추가적으로, 액티비티가 재생성될 때 viewModel이 유지되는것으로 보아 액티비티가 맨 처음 생성될 때만 `nc`가 `null`이고, 그 이후부터는 `null`이 아니다.

결국 getViewModelStore도 평범했다. 어디까지 더 깊이 들어가야 할지 모르겠지만 끝이 보이는 느낌이 온다😭.

## NonConfigurationInstances

`NonConfigurationInstances`를 알아보기 전에 먼저 `getLastNonConfigurationInstance()` 함수를 타고 들어가보았더니, 이건 또 `ComponentActivity`의 메서드가 아니라 더 상위 클래스인 `Activity` 클래스의 메서드다.

// Activity.java
![image](https://user-images.githubusercontent.com/18481078/95671679-cae66080-0bd4-11eb-8057-50bc0c4864aa.png)

`mLastNonConfigurationInstances`를 타고 들어가니 `static final class NonConfigurationInstances` 클래스가 있다.

그렇다면 액티비티 객체가 재생성 되고 나서는 `mLastNonConfigurationInstances`이 `null`이 아니라는 말일테고, 그말은 즉 `onDestroy`될 때쯤 `mLastNonConfigurationInstances`를 세팅해주는 곳이 어딘가에는 있어야 한다는 의미이다.

구글링을 해보니 역시나 `onRetainNonConfigurationInstance()` 라는 메서드가 있다. 이 메서드는 ComponentActivity에 구현되어 있으며, configuration change로 인해 Activity가 재생성될 때마다(정확히는 onStop과 onDestroy() 사이에) 시스템에 의해 호출된다고 한다.

// ComponentActivity.java
![image](https://user-images.githubusercontent.com/18481078/95671680-ccb02400-0bd4-11eb-821c-e9bd99e75778.png)

이 함수의 반환값은 `NonConfigurationInstances`타입의 객체다. 공식문서에는 아래와 같이 나와있다.

> The object you return here will always be available from the getLastNonConfigurationInstance() method of the following activity instance as described there.

즉 여기서 리턴하는 nci라는 값은 시스템에 의해 유지된다는 의미이다. `nci`값은 `getLastNonConfigurationInstance()` 함수를 통해 사용할 수 있다.

## 마무리

ViewModel이 유지되는 원리는 알아보았다.
핵심은 activity가 재생성될 때 시스템이 call 해주는 `onRetainNonConfigurationInstance()` 함수라고 볼 수 있겠다. `onRetainNonConfigurationInstance()` 함수의 리턴값이 어떻게 `getLastNonConfigurationInstance()` 메서드에서 사용이 가능한지는 공식 문서에도 나와있지 않을 뿐더러 시스템 내부의 일이라 알아보는 의미가 크지 않을것 같다.

## 참고자료

- [Activity#getLastNonConfigurationInstance()](<https://developer.android.com/reference/android/app/Activity#getLastNonConfigurationInstance()>)
- [Activity#onRetainNonConfigurationInstance()](<https://developer.android.com/reference/android/app/Activity#onRetainNonConfigurationInstance()>)
- [ViewModels: Persistence, onSaveInstanceState(), Restoring UI State and Loaders](https://medium.com/androiddevelopers/viewmodels-persistence-onsaveinstancestate-restoring-ui-state-and-loaders-fc7cc4a6c090)
