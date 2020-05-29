---
layout: post
title: 잠금 화면 (Screen Lock) 구현 및 이슈 해결
category: Android
tags: [ScreenLock, ProcessLifecycleOwner, Lifecycle, LifecycleObserver]
comments: true
---

# 앱 자체 화면 잠금(Lock Screen 구현)

**앱 내 자체 화면이란, 안드로이드 설정 메뉴에서 잠금 화면을 설정하는게 아닌, 자신의 앱을 켜고 끌 때마다 나타나는 잠금 화면을 말한다.**

<img src="https://user-images.githubusercontent.com/18481078/83267688-8d71cf80-a1ff-11ea-8640-20eaa2e34137.png" width="250" height="500">

최근에 앱 내 화면 잠금 기능을 구현할 일이 생겼다. 평소 '디바이스를 켜고 끌 때마다 잠금 해제를 하는데 앱을 켜고 끌 때도 해야 해?' 라는 생각에 전혀 사용하지 않았던 기능이었다(사실 메이저 앱들이 이런 기능을 가지고 있는지 조차 몰랐다).

구현을 하기 위해 가장 먼저 들었던 원초적인 생각은 이랬다.
"사용자가 어떤 액티비티를 보고 있다가 잠시 화면을 나가고 재진입 할 줄 모르니, `BaseActivity`의 `onStart`에서 `ScreenLockActivity`를 띄우자."
그러나 이 생각대로 구현하게 된다면, A 액티비티가 B 액티비티를 호출할 때도 B 액티비티의 `onStart`에서 `ScreenLockActivity`를 띄울 테니 내가 원하는 형태가 아님을 쉽게 생각할 수 있었다.

떠오르는게 액티비티 또는 프래그먼트의 라이프사이클 뿐이라면 `BaseActivity`의 `onStart`()에서 어떻게든 처리할 수는 있을 것이다. screen on&off에 대한 이벤트를 브로드캐스트를 등록하여 받고, 단순 액티비티간 호출일 때는 예외 처리를 하고, 여러가지 상태 값들을 sharedPreference로 관리하여 구현하면 안될건 없다.

그러나 좀 더 단순하게, 더 멀리서 내다보면 앱 내에서 ScreenLockActivity를 띄워줘야 할 때는 앱이 **Background에서 Foregrounde로 진입할 때 뿐**이라는 사실을 알아차릴 수 있다. 힘들게 브로드 캐스트를 등록하지 않고도, 단순 액티비티 간의 이동일 때를 구분하지 않아도 된다. 액티비티나 프레그먼트의 라이프사이클이 아니라 앱 라이프사이클을 감지하는 `ProcessLifecycleOwner` 를 사용하면 간단하게 처리가 가능하다.

## ProcessLifecycleOwner

[ProcessLifecycleOwner](<[https://developer.android.com/reference/android/arch/lifecycle/ProcessLifecycleOwner](https://developer.android.com/reference/android/arch/lifecycle/ProcessLifecycleOwner)>) 는 애플리케이션 프로세스의 라이프사이클을 감지할 수 있게 도와준다. 이를 통해 앱이 포그라운드로 진입했는지, 백그라운드로 진입했는지 알 수 있다.

### 사용법

애플리케이션 라이프사이클을 감지할 LifeCycleObserver 클래스를 만든다.

```kotlin
class AppLifecycleObserver(
    private val context: Context
) : LifecycleObserver {

    @OnLifecycleEvent(Lifecycle.Event.ON_START)
    fun onForeground() {
        if (isUserUseScreenLock) ScreenLockActivity.start(context)
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_STOP)
    fun onBackground() {
        //background
    }
}

class ScreenLockActivity: Activity() {

	...

  companion object {
    fun start(context: Context) {
        val intent = Intent(context, ScreenLockActivity::class.java).apply {
            addFlags(Intent.FLAG_ACTIVITY_NEW_TASK)
            addFlags(Intent.FLAG_ACTIVITY_SINGLE_TOP)
        }
        context.startActivity(intent)
    }
  }
}
```

함수 이름은 의미가 없다. `@OnLifecycleEvent` 어노테이션이 중요하다. `ON_START`는 앱이 포그라운드에 진입할 때 호출되며, `ON_STO`P은 백그라운드로 진입하면 호출된다. 앱이 포그라운드에 진입할 때마다 `ScreenLockActivity`를 start 시켜주면 된다.

이제 `ProcessLifecycleOwner` 를 이용하여 `AppLifecycleObserver`를 등록하자.

```kotlin
class AppApplication : Application() {

    override fun onCreate() {
        super.onCreate()
        ProcessLifecycleOwner.get().lifecycle
            .addObserver(AppLifecycleObserver(applicationContext))
				...
    }
}
```

Activity나 Fragment에서 등록한게 아니라, **Application클래스에서 등록**해주는 점을 유의하자.
이렇게 함으로써 어떤 브로드 캐스트도, 다른 어떤 복잡한 로직도 없이 대부분의 메이저 앱들과 **거의 흡사하게** 동작하는 ScreenLockActivity을 만들 수 있다.

## 보완점

바로 위에서 **"거의 흡사하게" 라는 표현을 썻다.**
높지 않은 확률이지만 위의 코드에는 ScreenLockActivity를 무시한 채 앱에 진입할 수 있는 심각한 버그가 있다. 사실은 그 **버그와 그것을 해결하는 방법**이 이 포스팅의 핵심이다.

App Process LifeCycle에 대해 잘 모른다면, 아마도 사용자가 비밀번호를 입력하지 않고 뒤로가기를 눌렀을 때 앱을 종료시키기 위해 ScreenLockActivity의 onBackKeyPressed()에다가 finishAffinity()를 작성할 것이다.

```kotlin
// ScreenLockActivity.kt
override fun onBackPressed() {
    finishAffinity()
}
```

문제가 없어보이지만 앱의 라이프사이클과 엮여 문제를 일으키는 부분이 있다.

`finishAffinity()`가 호출되면 앱은 종료되며 백그라운드로 진입(홈 화면 보임)하게 되는데, **"백그라운드 진입 → 포그라운드 진입"을 아주 빠르게 실행했을 경우 `OnStart` 라이프사이클을 타지 않아 `ScreenLockActivity`가 무시된채 앱이 실행되어 버린다.** back키를 누르는 순간 `finishAffinity()`가 호출되어 `ScreenLockActivity`는 사라졌지만, 포그라운드로 돌아왔을 때 다시 `ScreenLockActivity`이 불러와져야 하는데 그렇지 못하는 것이다.

이 부분 때문에 많은 고민을 하다가 떠오른 아이디어 하나. 잠금화면에 진입했을 때 뒤로가기를 누르면 어차피 앱을 사용하지 못하게 할테니, **ScreenLockActivity** **액티비티를** `finishAffinity()`**하지 말고 그저 home 화면으로 이동만 시키면 어떨까?** 이렇게 한다면 잠금 화면이 떠있는 상태에서 Back Key를 눌렀을 때 단순히 홈키로 이동하는 것이기 때문에, 사용자가 아주 빠르게 우리의 앱을 다시 클릭한다 하더라도(즉 OnStart()가 호출되지 않더라도) 다시 잠금 화면이 뜨게 된다. 심지어 아주 빠르게.

### 그래도 완벽하진 않을텐데?

완벽하지 않다고 느낄 부분이 있을 수 있다. 예를 들어 사용자가 앱을 잘 사용하던 중에 홈 키를 눌러 홈 화면으로 이동했고, 아주 빠르게 다시 앱 런쳐를 눌러 앱을 키면 `OnStart()` 가 호출되지 않기 때문에 `ScreenLockActivity`가 뜨지 않는다. ScreenLockActivity가 떠있는 상태에서 뒤로가기를 했을 때 생기는 치명적인 버그는 해결되었지만, 여전히 조금의 아쉬운 점이라고 볼 수 있겠다(**그러나 이건 어쩔 수 없는 부분인건지, 아니면 이게 맞는 동작인건지 카카오톡 역시 동일하게 동작한다**).

그래서 여기저기 문서를 찾아 보았다. 명쾌한 답을 얻지는 못했지만 나름 추측해 볼만한 문서가 있었다. [App startup time - 공식문서](https://developer.android.com/topic/performance/vitals/launch-time#in-this-document)를 보면 앱을 켤 때 항상 똑같은 속도로 앱이 켜지는게 아니라, Hot start 상태, Warm start 상태, Hot start 상태에 따라 속도가 다르다. **즉, 안드로이드에서는 앱을 다시 켜는 속도에 관한 정책이 있다는 것인데, 그렇다면 아마도 앱을 종료했다가 아주 빠르게 다시 앱을 키게 될 경우에 한해서는 실제로 메모리에서 앱을 종료하고 실행시키는 것이 비효율 적이라 판단하여 OnStart와 OnStop 라이프사이클을 실행시키지 않는게 아닐까**라는 생각을 해볼 수 있었다. 지금 생각해보면 이게 사용성 측면에서 올바른 동작인 것 같기도 하다.

## 결론

`ProcessLifecycleOwner`를 사용하여 앱이 포그라운드에 진입할 때 마다 `ScreenLockActivity`를 실행시키되, `ScreenLockActivity`의 `onBackPressed()`에서 `finishAffinity()` 를 사용하지 말고, 단순히 홈 화면으로 이동시키는 코드를 넣자.

```kotlin
override fun onBackPressed() {
    moveTaskToBack(true)
}
```
