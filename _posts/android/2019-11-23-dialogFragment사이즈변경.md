---
layout: post
title: DialogFragment의 size(width, height)조절
category: Android
tags: [DialogFragment]
comments: true
---

## 마음처럼 되지 않는 DialogFragment

액티비티나 프레그먼트 위에서 DialogFragment를 띄우다 보면 DialgFragment의 전체 크기 조절이 마음처럼 되지 않는다.

어느 화면에서나 왼쪽 이미지정도의 사이드 여백을 갖는 Dialog를 만들고 싶지만, Dialog안의 내용물에 따라서 자동으로 오른쪽 이미지처럼 랜더링된다.

![여백캡처](https://user-images.githubusercontent.com/18481078/69471711-c14de180-0de5-11ea-86c6-d75b2a3140ad.png)

해당 DialogFragment의 xml 최상단 뷰를 보면 분명히 `android:layout_width="match_parent"`를 선언했기에 알아서 이쁘게 이상적인 여백을 가지고 랜더링 될 것 같지만 마음처럼 되지 않는다.

```xml
<androidx.constraintlayout.widget.ConstraintLayout
        xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="@drawable/white_border">

        ...
</androidx.constraintlayout.widget.ConstraintLayout>
```

## 컨텐츠와 무관하게 이상적인 여백을 가지는 방법

### 1. 지정해둔 dp 크기로 setLayout 하는 방법

첫 번째 방법은 미리 지정해둔 dp 크기를 불러와 코드상에서 dialog의 `width`, `heigth`를 지정해주는 것이다. 구글링을 해보면 stackOverflow에 가장 먼저 나오는 방법.
코드는 아래와 같다.

```java
// .java
// onResume()
int width = getResources().getDimensionPixelSize(R.dimen.popup_width);
int height = getResources().getDimensionPixelSize(R.dimen.popup_height);
getDialog().getWindow().setLayout(width, height);

// .xml
android:layout_width="match_parent"
android:layout_height="match_parent"
```

이 방법은 Dialog의 너비와 높이를 미리 지정해둔 Dp크기로 설정할 수 있지만, 어느 디바이스에서든 동일한 비율의 여백을 보장할 수는 없다. 또한 앱의 여러곳에서 다른 정보를 가지는 DialogFragment를 사용할 경우, 높이는 내용물에 무관하게 `wrap_content`하도록 구현하고 싶은 경우가 있지만 위 방법은 꼭 heigth를 지정해줘야 한다.

필자에겐 맞지 않는 방법이였다.

### 2. 디바이스 크기를 구해서 setAtributes 하는 방법

flutter에는 아래 코드로 쉽게 디바이스의 넓이를 구해 알맞은 비율로 레이아웃을 구성하는 경우 많다.

```dart
final deviceWidth = MediaQuery.of(context).size.width;
```

플루터처럼 여백 비율을 정해서 dialogFragment에 반영해보자.

#### device크기 구하기

android에서는 아래와 같이 디바이스의 크기를 구할 수 있다.

```kotlin
// 꼭 DialogFragment 클래스에서 선언하지 않아도 된다.
val windowManager = this.getSystemService(Context.WINDOW_SERVICE) as WindowManager
val display = windowManager.defaultDisplay
val size = Point()
display.getSize(size)

size.x // 디바이스 가로 길이
size.y // 디바이스 세로 길이
```

#### 여백 비율로 DialogFragment 크기 조절

DialogFragment의 `onResume()`안에서 실행해야 한다.

```kotlin
// .kt
override fun onResume() {
    super.onResume()
    val params: ViewGroup.LayoutParams? = dialog?.window?.attributes
    val deviceWidth = size.x
    params?.width = (deviceWidth * 0.9).toInt()
    dialog?.window?.attributes = params as WindowManager.LayoutParams
}

// .xml
android:layout_width="match_parent"
android:layout_height="wrap_content"
```

위와 같이 선언하면 dialogFragment의 너비는 항상 전체 디바이스 너비의 90%가 되며, 높이는 컨텐츠에 따라 자동 조절된다.

## 끄

읕
