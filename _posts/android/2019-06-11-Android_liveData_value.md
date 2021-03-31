---
layout: post
title: AAC LiveData setValue() vs postValue()
category: Android
tags: [AAC, LiveData]
comments: true
---

AAC ViewModel을 사용하면서 구글이 제공하는 LiveData를 많이 사용한다. LiveData는 `setValue()`와 `postValue()`메서드를 제공한다. 두가지 메소드 모두 MutableLiveData의 값을 변경시킬 수 있다. 꼭 이 두가지 메서드로만 데이터를 변경해야 데이터 변화를 감지할 수 있다.

## setValue()

말그대로 값을 set하는 함수다. LiveData를 구독하고 있는 옵저버가 있는 상태에서 setValue()를 통해 값을 변경시킨다면 **메인 쓰레드**에서 그 즉시 값이 반영된다. 중요한 점은 **메인 쓰레드**에서 값을 dispatch시킨다는 점이다.

## postValue()

백그라운드 thread인 상황에서 LiveData 값을 set 하고 싶을 때가 있다. 그럴 때 사용하는 메서드이다. 내부적으로 `new Handler(Looper.mainLooper()).post(() -> setValue())` 이런 코드가 실행된다. 즉 setting하고 싶은 값을 main lopper로 보내끼 때문에 결국 메인 쓰레드에서 값을 변경하게 된다. 공식문서에는 아래와 같이 설명되어있다.

> If you need set a value from a background thread, you can use postValue(Object)
> Posts a task to a main thread to set the given value.
> If you called this method multiple times before a main thread executed a posted task, only the last value would be dispatched.

따라서 postValue()를 한 다음 바로 다음 라인에서 LiveData의 getValue()를 호출한다면, 변경된 값을 받아오지 못할 가능성이 크다. 반면 setValue()로 값을 변경하면 메인쓰레드에서 변경하는 것이기 때문에 바로 다음 라인에서 `getValue()`로 변경된 값을 읽어올 수 있다.

아래의 코드를 보자.

```
 liveData.postValue("a");
 liveData.setValue("b");
```

우선 처음엔 "b"가 설정되고, 이후 메인스레드는 postValue("a")로 부터 값을 "a"로 변경시킨다.

또한 옵저버가 없는 필드에서 `postValue()`를 호출하고 `getValue()`를 호출한다면 `postValue()`로 변경한 값을 얻어올 수 없다.
