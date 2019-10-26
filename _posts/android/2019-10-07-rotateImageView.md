---
layout: post
title: Android Rotate ImageView 90 degree whenever you touch (이미지 클릭마다 에니메이션과 함께 90도씩 회전시키기)
category: Android
tags: [rotate, imageView]
comments: true
---

### 회전시키고 싶은 이미지 뷰, 또는 특정 버튼 클릭시 90도씩 imageView를 회전시키는 방법.

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    ...
    myImageView.setOnClickListener {
        val currentDegree = it.rotation
        ObjectAnimator.ofFloat(it, View.ROTATION, currentDegree, currentDegree + 90f)
            .setDuration(300)
            .start()
	}
}
```

`OjbectAnimator.ofFloat`메서드의 두 번째 인자로 시작 degree를, 세 번째 인자로 목표 degree를 넣는다. 시작 degree를 항상 imageView의 rotation값을 넣어주고, 목표 degree에 시작 degree + 90f를 해주면 클릭마다 이미지가 연속적으로 회전된다.

물론 위 코드는 `imageView`만 돌아갈 뿐 실제 Bitmap이 변경 되는것은 아니기 때문에 회전된 모습 그대로 서버에 전송하고 싶을 경우에는 실제 이미지 데이터인 Bitmap을 변경시키는 추가적인 코드가 필요하다.

실제 Bitmap자체를 회전할 경우 `Bitmap.createBitmap`이라는 비싼 연산을 수행해야 하는데, 실험 결과 3MB가 넘는 이미지를 회전하는데 드는 비용은 갤럭시s8 기준으로 상당히 느렸다. 따라서 이미지 에디터를 만들거나 뷰어 구현의 경우 매번 실제로 Bitmap을 조작하지 말고 사용자가 보고있는 현재 `imageView`만 회전 시키는게 좋다. 대신, 해당 이미지뷰가 회전됨에 따라서 몇 도 만큼 회전되었는지를 기록하고 있다가, 사용자가 전송 버튼을 눌렀을 때 Bitmap을 기록해둔 각도 만큼 회전시켜 전송하는게 좋다.

다음 포스트에서는 위의 샘플 코드를 보완하여 가로 세로 비율이 맞지 않는 이미지를 자연스럽게 회전시키는 방법을 공유하려한다.
