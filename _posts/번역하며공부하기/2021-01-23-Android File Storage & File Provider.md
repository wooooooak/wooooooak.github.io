---
layout: post
title: Android File Storage & File Provider
category: 번역하며 공부하기
tags: [Android Storage]
comments: true
---

원본 - [Android File Storage & File Provider](https://medium.com/swlh/android-file-storage-file-provider-5990c9baf52d)

## 도입부

이전 글에서 안드로이드 파일 저장소에대한 기본적인 사항들과 앱이 각 디렉토리에 파일을 저장할 때 쓰는 몇 가지 API들을 살펴보았습니다. 아직 읽지 않으셨다면 [Part 1](https://wooooooak.github.io/%EB%B2%88%EC%97%AD%ED%95%98%EB%A9%B0%20%EA%B3%B5%EB%B6%80%ED%95%98%EA%B8%B0/2020/01/19/Android-Storage-&-Scoped-Storage/)을 봐주세요. 오늘은 이전 글에서 배운 내용들을 실습해보려 합니다.

## 실습 목록

1. 파일 디렉토리에 파일 저장하기
2. 외부 저장소(external files directroy)에 파일 저장하기
3. File Provider API - content URI를 사용하여 당신의 앱 파일을 외부 시스템에 안전하게 공유하기

## 실습 1

첫 실습과 두 번째 실습에서는 오직 해당 앱의 저장소에만 접근하기 때문에 어떠한 런타임 permission을 선언하진 않습니다. 아래와 같이 디렉토리에 파일을 저장하는 함수를 만들어 봅시다.

```kotlin
fun saveFileInAppDirectory() {
    val file = FileUtils.createFileInStorage(appContext,
        "test.jpeg"
    )
    Timber.d("file path %s", file!!.absolutePath)
    if (!file.exists()) {
        file.createNewFile()
    }
    val assetManager = appContext.assets
    val inputStream: InputStream
    val bitmap: Bitmap
    try {
        inputStream = assetManager.open(SAMPLE_FILE_NAME)
        bitmap = BitmapFactory.decodeStream(inputStream)
        saveBitmap(bitmap, file)
    } catch (e: Exception) {
        Timber.e(e)
    }
}
```

asset 폴더에서 파일을 읽어온 후 디렉토리에 그 파일을 저장합니다. 파일과 관련된 유틸성 함수들을 모아둔 FileUtils 클래스도 있습니다. FileUtils.kt를 만들어 아래 함수를 복사해두세요.

```kotlin
fun createFileInStorage(context: Context, fileName: String): File? {
    val timeStamp: String = System.currentTimeMillis().toString() + Constants.JPEG
    val name = if (fileName.isBlank()) timeStamp else fileName
    return File(getAppFilesDir(context), name)
}
fun createFileInExternalStorage(context: Context, fileName: String): File? {
    val timeStamp: String = System.currentTimeMillis().toString() + Constants.JPEG
    val name = if (fileName.isBlank()) timeStamp else fileName
    return File(getAppExternalFilesDir(context), name)
}
private fun getAppFilesDir(context: Context): File? {
    val file = context.filesDir
    if (file != null && !file.exists()) {
        file.mkdirs()
    }
    return file
}
private fun getAppExternalFilesDir(context: Context): File? {
    val file = context.getExternalFilesDir(Environment.DIRECTORY_PICTURES)
    if (file != null && !file.exists()) {
        file.mkdirs()
    }
    return file
}
```

`saveFileInAppDirectory()` 함수를 호출하면 앱의 특정 디렉토리에 파일이 저장되며 이 디렉토리는 우리 앱의 내부 저장소이기 때문에 다른 앱이나 파일 매니저가 접근할 수 없습니다. 필자의 경우 Pixel 3 API 28 에뮬레이터에서 이 함수를 실행했을 때, 파일이 저장된 경로는 아래와 같았습니다.

```kotlin
/data/user/0/com.example.android.boilerplate/files/test.jpeg
```

## 실습 2

위와 마찬가지로 앱의 외부 저장소에 파일을 저장할 수도 있습니다. 아래와 같이 함수를 호출해봅시다.

```kotlin
fun saveFileInAppExternalDirectory() {
    val file = FileUtils.createFileInExternalStorage(appContext,
        "test.jpeg"
    )
    Timber.d("file path %s", file!!.absolutePath)
    if (!file.exists()) {
        file.createNewFile()
    }
    val assetManager = appContext.assets
    val inputStream: InputStream
    val bitmap: Bitmap
    try {
        inputStream = assetManager.open(SAMPLE_FILE_NAME)
        bitmap = BitmapFactory.decodeStream(inputStream)
        saveBitmap(bitmap, file)
    } catch (e: Exception) {
        Timber.e(e)
    }
}
```

주의 : **SAMPLE_FILE_NAME은 assets folder 이름입니다.**

마찬가지로 같은 환경에서 저장된 파일의 위치를 찍어 보았을 때 결과는 아래와 같았습니다.

```kotlin
/storage/emulated/0/Android/data/com.example.android.boilerplate/files/Pictures/test.jpeg
```

이 경우에는 파일 매니저를 사용하여 에뮬레이터에서 저장된 파일을 확인할 수 있습니다. 파일 매니저 앱을 열고, 오른쪽 상단 모서리에 위치한 setting 버튼을 클릭하여 "내부 저장소 보기" 버튼을 클릭해보세요. 그리고 나서 내부 저장소 폴더에 들어간 뒤 Android/data/<your_app_package_name>/files 에 들어가시면 됩니다.

앱 특정 캐시 디렉토리에 파일을 저장하는 경우에도 위와 동일한 접근 방식을 따르면 됩니다.

## File Provider

가끔은 당신의 앱 특정 디렉토리에 저장된 파일을 다른 앱과 공유하고 싶을 경우가 있을겁니다. 그렇다면 content URI를 사용하여 공유할 수 있습니다. 안드로이드 File Provider는 이를 위한 간단한 API들을 제공합니다. **context URI**를 공유함으로써 다른 앱들이 특정 파일을 쓰고 읽을 수 있도록 임시 권한을 부여합니다.

파일의 content URI을 만들기 위해서 아래와 같은 절차를 따르세요.

1. Android Manifest에 File Proficer를 선언합니다.
2. res 디렉토리의 xml 폴더 경로를 선언합니다.
3. fire provier API를 사용하여 context URI를 생성합니다.

## Manifest에 Fire Provider 선언

아래 코드의 `mydomain` 은 여러분의 도메인을 넣으세요.

```kotlin
<manifest>
    ...
    <application>
        ...
        <provider
            android:name="androidx.core.content.FileProvider"
            android:authorities="com.mydomain.fileprovider"
            android:exported="false"
            android:grantUriPermissions="true">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/file_paths" />
        </provider>
        ...
    </application>
</manifest>
```

## resource 디렉토리의 xml 폴더 경로 선언

File Provider는 당신이 미리 지정해둔 디렉토리안의 파일에 대해서만 content URI를 생성할 수 있습니다. 따라서 res 디렉토리안에 xml폴더를 만들고, `file_paths.xml` 파일을 만들어야 합니다. 그 파일은 아래와 같이 만듭니다.

```kotlin
<paths xmlns:android="http://schemas.android.com/apk/res/android">
   <external-files-path name="pictures" path="Pictures/" />
    ...
</paths>
```

여기서는 우리 앱의 외부 저장소의 파일에 접근할 것이고, 외부 저장소 안의 특정 파일에 대해서만 content URI를 생성할 것이기 때문에 `external-files-path`으로 경로를 선언해주었습니다. 이것은 앱의 외부 저장소에 파일을 저장하는 **실습 2**에 해당하는 내용이고 `path` 는 특정 파일이 위치한 폴더를 나타냅니다. **실습 2**의 경우 path는 앱의 외부 디렉토리 아래의 Pictures 폴더가 되죠.

## 선택사항

만약 **실습 1** 에 해당하는 File Provider를 선언하려 한다면, path 선언은 아래와 같습니다.

```kotlin
<files-path path="/" name="pictures" />
```

주의 : **실습 1** 에서는 **실습 2** 와는 달리 파일 디렉토리 안에 어떤 폴더도 만들지 않았습니다.

## file의 content URI 생성

파일의 content URI를 생성하기 위해서 **실습 2** 의 상황을 예로 들겠습니다. 우리는 assets 에서 파일을 읽어온 후 앱의 외부 디렉토리에 파일을 저장하고, 저장된 파일의 content URI를 생성할 것 입니다.

```kotlin
fun getContentUri(): String? {
    val file = FileUtils.createFileInExternalStorage(appContext,
        Constants.FILE_NAME
    )
    Timber.d("file path %s", file!!.absolutePath)
    if (!file.exists()) {
        file.createNewFile()
    }
    val assetManager = appContext.assets
    val inputStream: InputStream
    val bitmap: Bitmap
    try {
        inputStream = assetManager.open(SAMPLE_FILE_NAME)
        bitmap = BitmapFactory.decodeStream(inputStream)
        saveBitmap(bitmap, file)
        return provideContentUri(file)
    } catch (e: Exception) {
        Timber.e(e)
    }
    return null
}
private fun provideContentUri(file: File): String {
    val contentUri: Uri = FileProvider.getUriForFile(appContext,
        Constants.FILE_PROVIDER_AUTHORITY, file)
    return contentUri.toString()
}
```

위 함수는 아래와 같은 content URI를 리턴합니다.

```kotlin
content://com.mydomain.fileprovider/pictures/test.jpeg
```

끝났습니다!

이 시리즈의 마지막 포스팅에서는 몇가지 실습을 통하여 Scoped Storage를 배워보도록 하겠습니다.
