---
layout: post
title: Scoped Storage — Android(번역)
category: 번역하며 공부하기
tags: [Android Storage]
comments: true
---

원본 - [Scoped Storage — Android](https://kalaiselvan369.medium.com/scoped-storage-android-ee9a3926c975)

## 도입부

android storage & file storage 시리즈의 마지막으로 scoped storage의 장점에대해 학습해보겠습니다. 안드로이드의 파일 저장소에 대한 기본적인 개념을 익히기 위해서는 [Part 1](https://wooooooak.github.io/%EB%B2%88%EC%97%AD%ED%95%98%EB%A9%B0%20%EA%B3%B5%EB%B6%80%ED%95%98%EA%B8%B0/2021/01/19/Android-Storage-&-Scoped-Storage/) 과 [Part 2](https://wooooooak.github.io/%EB%B2%88%EC%97%AD%ED%95%98%EB%A9%B0%20%EA%B3%B5%EB%B6%80%ED%95%98%EA%B8%B0/2021/01/23/Android-File-Storage-&-File-Provider/) 를 참조하시면 좋습니다. Scoped storage는 android 10에서 도입되었으며 개발자에게는 앱에 이 기능을 적용시키기 전까지 해당 기능을 유보할 수 있는 선택권이 주어집니다. 그러나 android 11 부터는 scoped storage가 의무 적용되어야 하기 때문에 유보할 수 없습니다.

## 핵심 기능

1. 본인의 앱에 한하여 어떠한 저장소 권한 없이도 내부 & 외부 저장소에 무제한 접근할 수 있습니다.
2. 앱의 특정 미디어 파일을 media collection에 쓰려는 경우 어떤 저장소 권한도 필요하지 않습니다. 그러나 해당 앱의 파일 및 다른 앱이 만든 파일들을 모두 조회하시려면 읽기 권한이 필요합니다.
3. 오직 특정 앱에만 해당하는 컬렉션으로 파일을 구성합니다.
4. 다른 앱의 파일 및 앱 관련 데이터들은 scoped storage에 저장되기 때문에 보호됩니다.
5. 사용자가 앱을 삭제하는 경우, 해당 앱에 저장된 파일들은 모두 제거됩니다.

**주의** : 공유 저장소(shared storage)에 파일을 저장하는 앱의 경우 구체적인 특정 파일에 저장한 모든 파일을 앱 디렉토리로 옮겨 추후에도 그 파일들이 유지되도록 해주어야 합니다.

아래는 media store에 파일을 저장하는 코드 스니펫입니다. Android 10부터는 어떤 저장소 권한도 필요하지 않습니다.

```kotlin
fun saveFileToMediaStore() {

    // 경로와 이미지 정보 명시
    val values = ContentValues().apply {
        put(MediaStore.Images.Media.DISPLAY_NAME, "paris")
        put(MediaStore.Images.Media.MIME_TYPE, "image/jpeg")
        put(MediaStore.Images.Media.RELATIVE_PATH, Environment.DIRECTORY_PICTURES)
        put(MediaStore.Images.Media.IS_PENDING, 1)
    }

		// 파일이 생성될 저장소 명시
    val imageUri =
        appContext.contentResolver.insert(MediaStore.Images.Media.EXTERNAL_CONTENT_URI, values)

		// assets으로 부터 파일을 읽고 위에서 생성한 파일 쓰기
    val assetManager = appContext.assets
    val inputStream: InputStream
    var bitmap: Bitmap? = null
    try {
        inputStream = assetManager.open(SAMPLE_FILE_NAME)
        bitmap = BitmapFactory.decodeStream(inputStream)
    } catch (e: Exception) {
        Timber.e(e)
    }

    appContext.contentResolver.openOutputStream(imageUri!!).use { out ->
        bitmap!!.compress(Bitmap.CompressFormat.JPEG, 90, out)
    }

    values.clear()
    values.put(MediaStore.Images.Media.IS_PENDING, 0)
    appContext.contentResolver.update(imageUri, values, null, null)

}
```

**주의** : 읽기 권한은 앱 특화 파일일지라도 필요합니다. 그 이유는 시스템이 그 파일을 이전 버전의 앱에 해당하는 것으로 간주하기 때문입니다.

`IS_PENDING` vlalue는 1 에서 0으로 업데이트 해줍니다. 1인 경우는 특정 파일이 쓰기 혹은 기타 작업을 하고 있다는 것을 나타내는 것이고, 그에 따라 이미지 컬렉션에 노출되지 않습니다. 0으로 할당되면, 그때서야 컬렉션에 노출됩니다.

위 작업을 하고나면 File Manager Application을 사용하여 Picutures 디렉토리에 파일이 저장된 것을 확인할 수 있습니다.
