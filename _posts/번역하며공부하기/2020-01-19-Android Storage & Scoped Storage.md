---
layout: post
title: Android Storage & Scoped Storage(번역)
category: 번역하며 공부하기
tags: [Android Storage]
comments: true
---

원문 - [Android Storage & Scoped Storage](https://kalaiselvan369.medium.com/android-storage-scoped-storage-63949f8637e)

## Overview

Android는 디바이스 저장소의 읽고 쓰는 작업에 대한 많은 개선을 거쳐 오고 있습니다. 그 시작은 rumtime Permissions이었는데요, 디바이스 공유 저장소에 접근하려거든 필히 사용자에게 접근 권한을 얻어야만 하는 정책이죠. 그때부터 시작하여 Android는 앱이 정말로 그 데이터가 필요할 때만 디바이스로 부터 데이터를 읽거나 쓰도록 점점 강하게 제한을 걸어왔습니다.

Android의 저장소는 아래와 같이 분류할 수 있습니다.

1. 앱 특화 저장소
2. 공유 저장소
3. Preference
4. 데이터베이스

Android에 scoped storage가 도입된 이유를 이해하기 위해서는, 앱이 어떻게 내부/외부 저장소안에 있는 미디어 파일과 documents들에 접근하고, 수정하고, 저장하는지에 대한 기본적인 이해가 필요하고, 디바이스의 공유 파일에 접근하기 위해 제공되는 permission들에 대한 이해도 필요합니다.

## Application Storage

앱 저장소는 두 가지로 나눌 수 있어요.

1. 내부 저장소(Internal Storage) - 여기에 저장되는 파일은 외부로 부터 보호되며 오직 여기에 파일을 생성한 자기 자신(앱)만이 접근할 수 있음.
2. 외부 저장소(External Storage) - android 기기 저장소, SD 카드, 등등이 될 수 있음

내부 저장소(Internal Storage)에 저장되는 파일은 다른 앱에서는 접근이 **불가능** 합니다. 오직 그 파일을 생성한 앱 만이 접근할 수 있어요. 이는 앱의 중요하고 민감한 파일들을 외부로 부터 보호하도록 보안되어있습니다.

외부 저장소(External Storage)에 저장된 파일은 적절한 권한을 획득한 다른 앱들이 **접근할 수도 있습니다**. 비록 다른 앱이 이 파일들에 접근하는게 가능하긴 하지만, 이 디렉토리에 저장된 파일들은 오직 당신의 앱에서만 사용되도록 고안되었습니다.

만약 다른 앱에서 당신이 만든 앱이 만든 파일에 접근해야만 해서 외부 저장소에 파일을 저장하고 싶다면, 그 파일은 외부 저장소의 **공유 저장소(shared storage)**에 저장해야합니다. 예를 들자면, 카메라 앱이 있겠네요. 카메라 앱에서 생성/수정된 파일은 Gallery app, File manager app 등등에서 접근이 가능해야만 합니다.

## 내부 저장소의 앱 특화 디렉토리

Android OS는 앱이 디렉토리에 접근할 수 있도록 내부 저장소에 상당한 양의 공간을 제공합니다. 보통 내부 저장소는 context 객체로 부터 호출할 수 있습니다.

1. 앱 영구 디렉토리

```kotlin
val file = File(context.filesDir, filename)
```

2. 앱 캐시 디렉토리

```kotlin
val cacheFile = File(context.cacheDir, filename)
```

## 외부 저장소의 앱 특화 디렉토리

만약 내부 저장소가 앱 특화 파일을 모두 저장하기에 부족하다면, 외부 저장소를 사용하세요. 내부 저장소의 앱 특화 디렉토리와 마찬가지로 context 객체에서 접근이 가능합니다.

1. 앱 영구 디렉토리

```kotlin
val file = File(context.filesDir, filename)
```

2. 앱 캐시 디렉토리

```kotlin
val cacheFile = File(context.cacheDir, filename)
```

## 저장소 위치 선택하기

때때로 디바이스는 내부 메모리의 파티션을 외부 저장소로 할당하기 위해 SD 카드 슬롯을 제공하기도 합니다. 그렇게 하여 디바이스가 여러개의 저장소 볼륨을 사용할 수 있게 합니다. 따라서 여러 다른 저장소 볼륨에 접근하려면 특별한 함수를 호출해야합니다.

```kotlin
val externalStorageVolumes: Array<out File> =
        ContextCompat.getExternalFilesDirs(applicationContext, null)
val primaryExternalStorage = externalStorageVolumes[0]
```

배열의 첫 번째 요소는 디바이스에 의해 외부 저장소 볼륨에 할당된 "primary 외부 저장소"로 간주됩니다.

## 저장소 가용성

대부분의 경우 외부 저장소는 외부에 장착된 디스크(SD card)를 의미하기도 합니다. 따라서 특정 볼륨이 접근 가능한지를 먼저 파악하고나서 외부 저장소로부터 앱 특화 데이터를 읽고/쓰는 것이 중요합니다.

```kotlin
// 외부 저장소를 포함한 볼륨이 읽고 쓸수 있는 상태인지 확인
fun isExternalStorageWritable(): Boolean {
    return Environment.getExternalStorageState() == Environment.MEDIA_MOUNTED
}

// 외부 저장소를 포함한 볼륨이 적어도 읽을 수 있는 상태인지 확인(쓰지는 못하더라도)
fun isExternalStorageReadable(): Boolean {
     return Environment.getExternalStorageState() in
        setOf(Environment.MEDIA_MOUNTED, Environment.MEDIA_MOUNTED_READ_ONLY)
}
```

## 여유 공간 쿼리

디바이스에 사용 가능한 저장 공간이 많지 않은 경우가 흔하기 때문에 앱은 신중하게 공간을 사용해야합니다. 아래 코드를 사용하여 사용 가능한 여유 공간을 확인할 수 있고, 저장소 관련 작업을 수행할 수 있습니다.

```kotlin
// 앱은 내부 저장소에 10 MB를 필요로 합니다.
const val NUM_BYTES_NEEDED_FOR_MY_APP = 1024 * 1024 * 10L;

val storageManager = applicationContext.getSystemService<StorageManager>()!!
val appSpecificInternalDirUuid: UUID = storageManager.getUuidForPath(filesDir)
val availableBytes: Long =
        storageManager.getAllocatableBytes(appSpecificInternalDirUuid)
if (availableBytes >= NUM_BYTES_NEEDED_FOR_MY_APP) {
    storageManager.allocateBytes(
        appSpecificInternalDirUuid, NUM_BYTES_NEEDED_FOR_MY_APP)
} else {
    val storageIntent = Intent().apply {
        action = ACTION_MANAGE_STORAGE
    }
    // 사용자가 파일을 삭제할 수 있도록 prompt를 보여줍니다.
}
```

[Part 2](https://medium.com/swlh/android-file-storage-file-provider-5990c9baf52d) 에서는 앱을 하나 만들어 보면서 우리가 다루었던 주제들을 실험해 보겠습니다. 또한 File Provider API도 알아봅니다.
