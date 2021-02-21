---
layout: post
title: How The Android Image Loading Library Glide and Fresco Works?(번역)
category: 번역하며 공부하기
tags: [Bitmap, Glide, Image Loading]
comments: true
---

원본 - [How The Android Image Loading Library Glide and Fresco Works?](https://blog.mindorks.com/how-the-android-image-loading-library-glide-and-fresco-works-962bc9d1cc40)

저는 오늘 어렵게 슥듭한 지식을 공유해보려합니다.

> 지식은 그것을 갈망하는 자에게 온다.

따라서 아주 주의 깊게 글을 읽어주세요. 개발자들은 커피를 좋아하죠. 커피 한 잔들 하시고 시작해봅시다.

## 시작

안드로이드에서 이미지(bitmaps)을 다루는 일은 메모리 부족 현상(OOM)을 일으킬 수 있어 매우 까다로운 작업입니다. OOM은 안드로이드 개발자에게 가장 큰 악몽과도 같죠.

다행히도 우리 모두는 open-source 세상에 살고있습니다. 저도 오픈소스를 굉장히 사랑합니다. 거의 모든 것을 Android 커뮤니티에서 얻고 있는 만큼, 저도 제 지식을 여러분께 공유하는 것을 좋아합니다.

**아무튼, 이제 ImageView에 이미지를 로딩하는 과정에서 마주치는 문제들을 살펴봅시다.**

- Out of memory(OOM) 에러
- 느린 로딩 문제
- UI의 반응이 멈추는 현상. 부드로운 스크롤이 되지 않는 현상

우리는 이런 현상을 심각하게 받아들여야만 합니다. 그리고 이런 문제들은 [Glide](https://github.com/bumptech/glide)나 [Fresco](https://github.com/facebook/fresco)같은 라이브러리를 사용하여 해결할 수 있습니다.

저는 이런 이슈들을 마주칠 때 마다 잠을 설치곤 했습니다. 그러다가 어느날 안드로이드에 library라는 단어를 알게되었는데요, 예전에는 안드로이드에 라이브러리를 쓸 수 있다는 사실조차 알지 못했습니다. 저는 라이브러리가 무엇인지에 대해 읽었고, 이미지 로딩을 위해 Glide라는 라이브러리를 알게 되었죠.

이게 어떻게 문제들을 해결하는지 하나씩 알아봅시다.

## Out of memory error

가장 큰 악몽이죠. 이를 해결하기 위해 Glide는 다운샘플링이란 것을 해줍니다. 다운샘플링은 bitmap(image)을 실제 view가 요구하는 사이즈로 줄여주는 것을 의미합니다. 예를 들어 우리에게 2000*2000 사이즈 이미지가 있다고 해봅시다. view size는 400*400입니다. 이때 Glide는 2000*2000이미지를 로딩하는게 아니라, 다운 샘플링하여 400*400으로 만들고 이를 view에 보여주는 방식입니다.

```kotlin
GlideApp.with(context).load(url).into(imageView);
```

Glide는 imageView를 파라미터로 받기 때문에 imageView의 사이즈를 알 수 있습니다.

Glide는 원본 이미지 전체를 메모리에 로딩하지 않고 다운 샘플링을합니다. 이를 통해 bitmap은 메모리를 적게 차지하고, OOM문제는 해결됩니다. 행복하네요.

## Slow loading

느린 로딩은 bitmap을 view에 로딩할 때 생기는 또 다른 문제입니다. 주된 원인은 view가 window를 벗어났음에도 불구하고 다운로딩이나 bitmap을 디코딩하는 작업들을 취소하지 않기 때문입니다. 따라서 더이상 필요하지 않은 작업들이 계속해서 불필요하게 동작하는 것이죠. Glide는 이것도 해결해줍니다. 적절히 불필요한 작업들을 취소하고 오직 사용자에게 보여지는 image만 로딩합니다. 이게 이미지를 빠르게 로딩해주는 비결입니다.

Glide는 Activity와 fragment의 lifecycle을 알고 있습니다. 이를 통해 어떤 이미지들이 취소되어야 하는지를 알 수 있게 됩니다.

다른 방법으로는 메모리 캐시를 만드는 방법인데요, 이를 통해 매번 bitmap을 디코딩할 필요가 없어집니다. Glide는 설정가능한 사이즈의 캐시를 만들어서 bitmap을 캐싱합니다.

캐싱에는 두 가지 레벨이 있습니다.

1. 메모리 케시
2. 디스크(Disk) 캐시

**Glide에 URL을 넘기면, 아래와 같은 일이 벌어집니다.**

1. memory에 URL key에 해당하는 이미지가 있는지 확인
2. memory에 캐시된게 있다면 그대로 가져와서 사용
3. memory에 없다면, disk에 있는지 확인
4. disk에 있다면 disk로부터 bitmap을 로딩후 memory 캐시로 옮기고, bitmap을 view에 로딩
5. disk에 없다면, 네트워크에서 이미지를 다운로드하고 disk캐시에 옮긴 후 또 한 번 memory 캐시에 옮김. 그리고 bitmap을 view에 로드.

이런 방법으로 이미지 로딩을 항상 빠르게 작업합니다.

## Unresponsive UI

UI가 반응하지 않는 가장 크고 중요한 문제는 main 쓰레드에서 너무 많은 작업이 일어나기 때문입니다. 우리는 UI 랜더링과 관련된 모든 작업은 메인 쓰레드에서 동작해야한다는 사실을 알고 있습니다. 그리고 Android는 UI를 16ms마다 업데이트 하죠. 만약 어떤 작업이 16ms보다 더 걸린다면 android는 그 update를 skip하게 되고 결과적으로 frame이 skip됩니다. frame을 skip하는 것은 초당 frame을 적게 만드는 원인이죠.

대학 시절에는 더 높은 초당 프레임 수(FPS)를 가진 movie clip위해 고군분투 했었습니다. FPS가 높을 수록 더 부드럽게 재생되니까요.

만약 FPS가 낮으면 사용자들은 반응성이 좋지 않은 지연된 UI를 보게 됩니다. bitmap을 로드하는 동안, 심지어는 백그라운드에서 로드하더라도 UI는 지연됩니다. 왜 그럴까요?

그 이유는 bitmap은 사이즈가 너무 크기 때문에 가비지 컬렉터(GC)가 매우 바쁘게 움직이기 때문입니다.

실제로 **가비지 컬렉터가 실행되는 동안, 애플리케이션은 동작하지 않습니다**.

가비지 컬렉터는 동작할 시간이 필요하고, 동작하는동안 시스템이 여러 frame을 skip하게끔 만듭니다. 따라서 가비지 컬렉터가 범인이죠.

Glide는 이를 어떻게 해결할까요?

Bitmap Pool을 사용하여 해결합니다.

Glide는 bitmap pool 컨셉을 사용하여 가능한 가비지 컬렉터 호출을 최소화 합니다.

Bitmap pool을 사용함으로써 애플리케이션에서 메모리의 지속적인 할당 및 할당 해제를 방지하고, GC 오버헤드를 줄이며 애플리케이션이 원활하게 샐행되도록합니다.

어떻게 메모리의 지속적인 할당 및 해제를 피하는 걸까요?

bitmap의 [inBitmap](https://developer.android.com/reference/android/graphics/BitmapFactory.Options.html#inBitmap) 프로퍼티를 사용합니다(이는 bitmap 메모리를 재사용 합니다).

[Re-using Bitmaps (100 Days of Google Dev)](https://www.youtube.com/watch?v=_ioFW3cyRV0)

Android 애플리케이션에서 몇 개의 비트맵을 로드해야한다고 생각해봅시다.

bitmap 두 개(bitmapOne, bitmapTwo)를 하나씩 로드해야 한다고 해보죠. bitmapOne을 로드할 때면 bitmapOne을 위한 memory를 할당하게 됩니다. 이후 더 이상 bitmapOne이 필요 없게 되더라도 bitmap을 재활용하지 마세요(재활용에는 가비지컬렉터 호출이 필요하니까요). 대신 bitmapOne을 bitmapTwo의 [inBitmap](https://developer.android.com/reference/android/graphics/BitmapFactory.Options.html#inBitmap)으로 사용하세요. 이런 식으로 동일한 메모리를 bitmapTwo를 위해 재사용할 수 있습니다.

어떻게 동작하는지 코드를 살펴봅시다.[inBitmap](https://developer.android.com/reference/android/graphics/BitmapFactory.Options.html#inBitmap) 프로퍼티를 주의 깊게 살펴보세요.

```kotlin
Bitmap bitmapOne = BitmapFactory.decodeFile(filePathOne);
imageView.setImageBitmap(bitmapOne);
// lets say , we do not need image bitmapOne now and we have to set // another bitmap in imageView
final BitmapFactory.Options options = new BitmapFactory.Options();
options.inJustDecodeBounds = true;
BitmapFactory.decodeFile(filePathTwo, options);
options.inMutable = true;
options.inBitmap = bitmapOne;
options.inJustDecodeBounds = false;
Bitmap bitmapTwo = BitmapFactory.decodeFile(filePathTwo, options);
imageView.setImageBitmap(bitmapTwo);
```

bitmapTwo를 디코드하는 동안 bitmapOne의 매모리를 재사용하고 있습니다.

이런 방법으로 더이상 bitmapOne를 참조하지 않음에도 가비지 컬렉터가 호출되지 않게 할 수 있습니다. 오히려 bitmapTwo가 bitmapOne이 사용했던 메모리를 그대로 사용할 수 있게 되었죠.

한 가지 중요한 사실은, bitmapOne의 사이즈가 bitmapTwo의 사이즈와 동일하거나 더 커야한다는 것입니다. 그래야 bitmapOne의 메모리가 재사용 될 수 있습니다.

Android 버전에 따라 bitmap 재사용을 할 때 고려해야할 몇 가지 사항들이 있습니다. 이 [프로젝트](https://github.com/amitshekhariitbhu/GlideBitmapPool)를 참조하시면 좋을것 같네요.

아무튼 Glide는 bitmap을 위해 bitmap pool을 사용합니다.

bitmap pool을 더 이상 필요하지 않은 비트맵들이지만 새로운 비트맵을 위해 재사용 될 수 있는 비트맵 리스트라고 봐도 좋습니다.

어떤 비트맵이라도 재활용될 수 있다면, Glide는 그 비트맵을 bitmap pool에 넣습니다.

Glide가 새로운 bitmap을 로드해야 할 때면, 같은 메모리의 bitmap pool으로부터 재사용 가능한 bitmap을 찾아 가져갑니다. GC가 호출되지 않고, recycling도 없게됩니다.

Fresco 역시 Glide와 같은 일을 합니다. 약간 다를수는 있어도 그 컨셉은 거의 동일해요.

시간내어 긴 글 읽어주셔서 감사합니다.
