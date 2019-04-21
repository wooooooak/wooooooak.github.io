---
layout: post
title: Attempt to invoke virtual method 'boolean androidx.recyclerview.widget.RecyclerView$ViewHolder.shouldIgnore()' on a null object reference 에러
category: error
tags: [recycler-view, android]
comments: true
---

## 증상
안드로이드에서 recyclerView를 사용하던 중 아래와 같은 에러가 날 때가 있다.
```
java.lang.NullPointerException: Attempt to invoke virtual method 'boolean androidx.recyclerview.widget.RecyclerView$ViewHolder.shouldIgnore()' on a null object reference
        at androidx.recyclerview.widget.RecyclerView.findMinMaxChildLayoutPositions(RecyclerView.java:4101)
        at androidx.recyclerview.widget.RecyclerView.dispatchLayoutStep1(RecyclerView.java:3835)
        at androidx.recyclerview.widget.RecyclerView.onMeasure(RecyclerView.java:3330)
        at android.view.View.measure(View.java:23169)
        at androidx.constraintlayout.widget.ConstraintLayout.internalMeasureChildren(ConstraintLayout.java:1227)
        at androidx.constraintlayout.widget.ConstraintLayout.onMeasure(ConstraintLayout.java:1572)
        at android.view.View.measure(View.java:23169)
        at android.view.ViewGroup.measureChildWithMargins(ViewGroup.java:6749)
        at android.widget.FrameLayout.onMeasure(FrameLayout.java:185)
        at android.view.View.measure(View.java:23169)
        at android.view.ViewGroup.measureChildWithMargins(ViewGroup.java:6749)
        at android.widget.FrameLayout.onMeasure(FrameLayout.java:185)
        at android.view.View.measure(View.java:23169)
        at android.view.ViewGroup.measureChildWithMargins(ViewGroup.java:6749)
        at android.widget.LinearLayout.measureChildBeforeLayout(LinearLayout.java:1535)
        at android.widget.LinearLayout.measureVertical(LinearLayout.java:825)
        ...
```

## 해결
구글링 해보니 원인이 상당히 다양한것 같다. 불행히도 나에게 맞는 해결법은 없었는데, 나의 경우 recycler뷰에 addView를 해주는 코드 때문에 에러가 났다. 기존에 사용하던 일반 스크롤 뷰를 리사이클러 뷰로 바꾸는 과정에서 지우지 않은 코드가 있었던 것이다.

아래 코드와 같이 recyclerView에 addView를 해주면 
`Attempt to invoke virtual method 'boolean androidx.recyclerview.widget.RecyclerView$ViewHolder.shouldIgnore()' on a null object reference` 에러가난다. recyclerView에는 꼭 어댑터를 연결해서 데이터를 바인딩해주자.
```kotlin
myRecyclerView.addView(userImageView)
```