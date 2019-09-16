---
layout: post
title: Dialog Fragment 모서리 둥글게 하기 (round corner)
category: Android
tags: [dialog, radius]
comments: true
---

`DialogFragment`를 상속받아 커스텀 다이얼 로그를 띄울 때 아래 그림처럼 모서리를 둥글게 하기 위해서는 커스텀 다이얼로그 클래스에 몇가지 코드가 필요하다.
![image](https://user-images.githubusercontent.com/18481078/64946525-0ddce400-d8ae-11e9-87b4-048246c9eafb.png)

### layout 파일

```xml
<androidx.constraintlayout.widget.ConstraintLayout
        xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="@drawable/alert_white_border"
        >
        ...
        <!-- 추가로 넣고 싶은 요소들 -->
        ...
</androidx.constraintlayout.widget.ConstraintLayout>
```

### @drawable/alert_white_border

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle">
    <solid android:color="@android:color/white" />
    <corners android:radius="20dp" />
</shape>
```

이제 `CustomDialog`클래스에 layout을 inflate 시키면 끝날것 같지만 그냥 inflate시키면 모서리가 여전히 직각이다. 따라서 아래의 두 코드를 삽입해주자.

```kotlin
dialog?.window?.setBackgroundDrawable(ColorDrawable(Color.TRANSPARENT))
dialog?.window?.requestFeature(Window.FEATURE_NO_TITLE)
```

### CustomDialog

```kotlin
class CustomDialog : DialogFragment() {

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        val view = inflater.inflate(R.layout.dialog_detail_info, container,false)
        dialog?.window?.setBackgroundDrawable(ColorDrawable(Color.TRANSPARENT))
        dialog?.window?.requestFeature(Window.FEATURE_NO_TITLE)

        ...

        return view
    }

}
```

`dialog?.window?.requestFeature(Window.FEATURE_NO_TITLE)` 이코드를 삽입 하지 않아도 모서리는 둥글게 나오지만, android version 4.4 이하에서는 blue line이 다이얼로그 상단에 나타난다고 하니 꼭 넣어주자.
