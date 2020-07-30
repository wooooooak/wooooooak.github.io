---
layout: post
title: 이모티콘 컨테이너 UI 구현 (채팅, like 카카오톡)
category: Android
tags: [UI/UX, Chat app, android, emoticon_container]
comments: true
---

### 카카오톡 이모티콘 컨테이너 UI 모습

<img src="https://user-images.githubusercontent.com/18481078/88927681-bde2f200-d2b2-11ea-8282-304e4d2a3183.gif" width="200" height="440">

# 서론

채팅 앱을 개발하다 보면, 참으로 많은 부분에서 **"카카오톡 처럼 해주세요"** 라는 요구사항이 들어온다. 메세지 공지 UI는 물론이고 검색, 첨부 파일 관리, 설정 화면 등등 모범이 될 만한 UI들이 많기 때문인 것 같다. 실제로 채팅 앱을 개발해보니 카카오톡에서 당연하게 쓰고 있던 사소한 기능과 UI들이 실제로 구현하기에는 상당히 까다롭고 귀찮은 작업들이 많다는 것을 느낄 수 있었다.

그 중에서도 생각 보다 구현이 까다로웠던 부분이 **이모티콘 컨테이너 UI** 였다. 위의 GIF에서 보이듯이 키보드가 올라와있는 상태에서 이모티콘 버튼을 클릭하면 키보드가 내려가고 이모티콘 컨테이너가 나오는데, **마치 키보드 뒤에서 기다리고 있었다는 듯이 이모티콘 컨테이너가 자연스럽게 모습을 드러낸다.**

## 어색한 UI

<img src="https://user-images.githubusercontent.com/18481078/88930201-1b2c7280-d2b6-11ea-97e9-261803eafcb8.gif" width="200" height="440">

카카오톡 이모티콘 컨테이너 UI에 대해서 깊게 살펴보지 않았을 때에는 단순하게 구현 했었다. UI도 UI지만 우선은 기능 개발이 급했기 때문에, 1.키보드 내림 → 2.이모티콘 컨테이너 띄움 순서로 구현해 놓았다.
그러나 이런 UI는  
**1. 채팅 내용을 담은 view가 키보드를 따라 밑으로 쭉 내려갔다가**,
**2. 이모티콘 컨테이너가 올라감에 따라 다시 위로 쭉 올라가는** UI가 된다.
화면이 크게 한번 깜빡이는 것과 비슷하다. 이를 개선하기 위해 카카오톡을 유심히 살펴보았고, 나름대로 구현한 방법을 공유하려 한다.

## 훌륭한 UI

채팅앱의 경우, 대부분 UI가 비슷할 것 같다. 맨 위에 툴바가 있고, 가운데 리사이클러뷰가 있고, 맨 아래 Edit Text가 있는 형태다. Edit Text를 클릭하면 당연히 키보드가 올라온다.

이때 중요한 점은 [windowSoftInputMode](<[https://developer.android.com/guide/topics/manifest/activity-element.html#wsoft](https://developer.android.com/guide/topics/manifest/activity-element.html#wsoft)>) 이다. 안드로이드 개발자라면 기본적으로 알아야 할 지식일 뿐만 아니라, 카카오톡 같은 UI를 구현하기 위해서 필자가 사용한 **핵심 내용**이기 때문에 꼭 숙지하자("[android adjustResize](https://www.google.com/search?q=android+adjustResize&oq=android+adjustResize&aqs=chrome..69i57j0l7.447j0j1&sourceid=chrome&ie=UTF-8)" 라고만 구글링 해도 좋은 자료가 많이 나온다).

키보드가 올라오면 Edit Text를 가리지 않도록 하기 위해, 그리고 toolbar가 화면 위로 Over되지 않도록 하기 위해 adjustResize 속성을 넣어 주게 된다.

```kotlin
<activity android:windowSoftInputMode="adjustResize" ... >
```

그런데 `adjustResize`속성을 사용하면 키보드가 올라올 때 키보드의 높이에 따라 Activity의 사이즈가 재 조정 된다. **Activity가 키보드 위에서 부터 시작하도록 사이즈가 줄어들었기 때문에 실제로 키보드 뒤에 이모티콘 컨테이너를 넣는건 불가능해 보인다.** 여러분이 이모티콘 컨테이너를 어떤 뷰로 만들었더라도 이미 키보드와 키보드 뒤의 영역은 view를 넣을 수 있는 Activity가 아니기 때문이다. 키보드가 올라와있는 상태에서 이모티콘 버튼을 누르면, 마치 키보드 뒷 공간에 이미 뷰가 존재했던것 처럼 느껴졌었지만, 실제로 adjustResize모드에서는 그게 불가능하다는 결론이 나왔다.

그럼 도대체 어떻게 저런 UI가 나올수 있을까를 한참 고민하던 중에, 필요할 때만 android:windowSoftInputMode를 동적으로 바꿔주면 가능하지 않을까 라는 생각이 들었다.

### 아이디어 : windowSoftInputMode 속성 이용하기

`windowSoftInputMode`속성에는 `adjustNothing`이라는 속성이 있다. 이 속성을 적용하면 키보드가 올라오고 내려갈 때 Activity와 view에 아무런 영향도 없다. 그저 키보드가 올라오면 **키보드 영역만큼 화면이 가려지는 것**이고, 내려가면 단지 내려가는 것이다. 즉, `adjustNothing` 일때는 `adjustResize`와는 다르게 activity 크기를 조절하지 않는다는 말이고, 키보드가 화면을 덮어버리는 방식이다.

**화면을 덮어버린다.** 즉 우리가 이모티콘 컨테이너 뷰를 만들어 놓으면, 그 뷰를 덮어버릴 수 있다는 이야기이다. 평소에는 `adjustResize`를 유지하다가 이모티콘 컨테이너를 보여줄때와 숨길때만 잠시 `adjustNothing`로 바꾸자.

### xml 구성

view 구성은 아래와 같다.

```kotlin
<ConstraintLayout ...>
    <ToolBar .../>
    <RecyclerView .../> // 메세지들이 노출되는 공간
    <EditText .../>     // 화면 아랫쪽 텍스트 입력 필드
    <EmoticonContainer
        android:id="@+id/emoticon_container"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:visibility="gone"
        app:layout_constraintTop_toBottomOf="EditText"
    .../>
</ConstraintLayout>
```

`EmoticonContainer`는 `EditText` 아래쪽에 배치하고 우선 `visibility`를 `gone`으로 해놓는다. 채팅방에 들어가자마자 이모티콘 컨테이너를 띄울 필요는 없으니까.

동시에 `height`는 `wrap_content`로 해놓자. **좋은 UI를 위해서, 추후에 키보드 높이로 다시 세팅해줄 것이다.**

### 이모티콘 버튼 클릭시 호출되는 코드 구성

```kotlin
// 이모티콘 버튼 클릭시 매번 호출되는 코드
lifecycleScope.launch {
      if (showContainer) { // 상황 1
          window.setSoftInputMode(WindowManager.LayoutParams.SOFT_INPUT_ADJUST_NOTHING)
          emoticon_container.visibility = View.VISIBLE
          hideKeyboard()
          delay(100)
          window.setSoftInputMode(WindowManager.LayoutParams.SOFT_INPUT_ADJUST_RESIZE)
      } else { // 상황 2
          window.setSoftInputMode(WindowManager.LayoutParams.SOFT_INPUT_ADJUST_NOTHING)
          showKeyboard() // 키보드 올리는 코드
          delay(100)
          emoticon_container.visibility = View.GONE
          window.setSoftInputMode(WindowManager.LayoutParams.SOFT_INPUT_ADJUST_RESIZE)
      }
  }
```

상황을 가정해보자.

---

**상황 1.** 키보드가 올라와 있는 상태에서 사용자가 이모티콘 버튼 클릭(키보드는 내려가고, 마치 이모티콘 컨테이너가 뒤에 있었다는 듯 나타난다, 이때 리사이클러 뷰를 포함한 다른 뷰 들은 전혀 미동이 없어야 한다)

코드에서는 if문에 걸리는 상황이다. 이모티콘 컨테이너를 띄워야 하더라도 당장 키보드를 내려선 안된다.

1. `adjustNothing`로 바꿔 준다. 이제 키보드 뒤에 이모티콘 뷰 를 넣을 수 있는 조건이 되었다(`adjustResize` 모드 였다면 키보드 위쪽에 이모티콘 컨테이너, 그 위에 EditText가 배치되어 버림).
2. `gone`이었던 컨테이너를 `visible`로 바꿔 주자(여기서 중요한 점은 컨테이너의 높이가 키보드의 높이와 동일해야 한다는 점인데, 이 방법은 뒤에서 다룬다). **키보드 뒤에 컨테이너가 숨어져있는 상태가 되었다.**
3. 컨테이너를 가리고 있는 키보드를 `hide()`시켜주자. hide 시키면 뒤에 숨어있던 컨테이너가 자연스럽게 보이며, 키보드가 내려 갔어도 컨테이너 높이 때문에 editText를 비롯한 다른 뷰 들은 여전히 미동 없이 그 자리에 있다.
4. 키보드를 내리자마자 delay를 0.1초 정도 주었다. `hideKeyboard()`를 호출 했다고 해서 키보드가 바로 사라지는게 아니라 끝까지 내려가는데 시간이 걸리기 때문이다. 만약 delay를 주지 않고 곧바로 `adjustResize`로 설정 해버린다면 키보드가 내려가는 도중에 적용이 되어 버려서 이모티콘 컨테이너가 내려가던 키보드 위에 달라붙게 된다.
5. 딜레이 이후 다시 `adjustResize`로 바꿔주자.

---

**상황 2.** 이모티콘 컨테이너가 띄워져 있는 상태에서 사용자가 다시 이모티콘 아이콘을 클릭하여 키보드가 올라오는 상황(키보드가 자연스럽게 이모티콘 컨테이너를 덮는 UI가 되어야 한다)

코드에서는 else문에 걸리는 상황이다. 이때도 성급히 키보드를 올려버리면 안된다.

1. `adjustNothing`로 바꿔 준다. 이제 키보드가 이모티콘 컨테이너를 덮을 수 있는 조건이 되었다.
2. 키보드를 올려 키보드가 이모티콘 컨테이너를 덮어버리게 하자.
3. 다 덮을 때 까지 delay를 0.1s 주자.
4. 이모티콘 컨테이너를 `gone`으로 바꾸자.
5. 아름다운 UI가 동작했으니, 아무일 없었다는듯 `adjustResize`를 적용시키자.

---

이렇게 하면 이모티콘 버튼을 계속 누르더라도 카카오톡과 동일한 아주 자연스러운 UI로 동작하게 된다. 컨테이너가 올라와있는 상태에서 Edit Text를 클릭하는 상황은 위 코드에 없지만, else문과 동일한 로직으로 사용 가능하다.

### 이모티콘 컨테이너 높이를 키보드 높이로 조정하기

키보드가 이모티콘 컨테이너를 덮는다 하더라도, 컨테이너의 높이가 키보드의 높이와 다르면 아무런 소용이 없다. 키보드 높이와 컨테이너의 높이가 완벽히 동일해야 한다.

```kotlin
// onCreate()
var rootHeight = -1
root_view.viewTreeObserver.addOnGlobalLayoutListener {
    if (rootHeight == -1) rootHeight = root_view.height // 매번 호출되기 때문에, 처음 한 번만 값을 할당해준다.
    val visibleFrameSize = Rect()
    root_view.getWindowVisibleDisplayFrame(visibleFrameSize)
        val heightExceptKeyboard = visibleFrameSize.bottom - visibleFrameSize.top
		// 키보드를 제외한 높이가 디바이스 root_view보다 높거나 같다면, 키보드가 올라왔을 때가 아니므로 거른다.
        if (heightExceptKeyboard < rootHeight) {
                // 키보드 높이
                val keyboardHeight = rootHeight - heightExceptKeyboard
        }
}
```

`addOnGlobalLayoutListener` 메서드를 사용하여 `r`oot_view`의 높이를 알 수 있다. 리스너로 등록한 코드 블럭은 레이아웃의 크기나 위치가 바뀔 때 마다 호출이 되는데, 키보드가 올라오고 내려갈 때 역시 매번 호출된다.

키보드가 올라왔을 때를 기회 삼아서 키보드 높이를 구할 수 있는데,
**키보드 높이 = 전체 높이 - 키보드를 제외한 높이**이다.

키보드를 제외한 높이는 `getWindowVisibleDisplayFrame` 를 사용하여 구할 수 있다.

```kotlin
val visibleFrameSize = Rect()
root_view.getWindowVisibleDisplayFrame(visibleFrameSize)
val heightExceptKeyboard = visibleFrameSize.bottom - visibleFrameSize.top
```

최종적으로 이렇게 구한 `keyboardHeight` 를 이모티콘 컨테이너의 `height`로 할당해주면 된다.

<img src="https://user-images.githubusercontent.com/18481078/88928722-1bc40980-d2b4-11ea-9d06-52c74a3ee147.gif" width="200" height="440">

## 끄

읕
