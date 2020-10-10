---
layout: post
title: Android Serializable vs Parcelable
category: Android
tags: [Serializable, Parcelable]
comments: true
---

안드로이드 개발을 하다보면 액티비티간 데이터를 공유해야 할 경우가 종종 생긴다. 이런 경우를 위해 우리는 intent를 하나 생성하고, 거기에 `putExtra()`로 데이터를 넣은 후 `startActivity(intent)`로 데이터를 공유함과 동시에 새로운 액티비티를 띄운다. String이나 Int등 Primitive Type은 그렇다 치더라도 직접 만든 커스텀 객체의 경우에는 그 객체를 Serializable(직렬화가능)하거나 Parcelable(포장가능?)한 객체로 만들어야만 보낼 수 있다.

### 직렬화란?

직렬화(serialization)란, 객체를 저장 장치에 저장 혹은 네트워크 전송을 위해 텍스트나 이진 형식으로 변환하는 것이다.

```kotlin
Person("JUN", 26) --> 직렬화 --> {"name":"JUN", "age":"26"}
```

이와 반대로 역직렬화(deserialization)란 텍스트나 이진 형식으로 저장된 데이터로부터 원래의 객체를 만들어내는 것이다.

```kotlin
{"name":"JUN", "age":"26"} --> 역직렬화 --> Person("JUN", 26)
```

## 액티비티간 데이터 전달

액티비티간의 데이터 전달이 일반적인 객체와 객체간의 데이터 공유(함수의 파라미터로 넘긴다던가 생성자로 데이터를 넘기는 방식)와 다른 방식으로 동작하는 이유는 무엇일까? 내가 만튼 커스텀 객체의 경우, 객체의 주소값을 넘기지 않고 꼭 값을 직렬화 해서 보내는 이유는 뭘까?

액티비티는 단순히 하나의 앱에서만 데이터를 공유하기도 하지만, 다른 앱, 프로세스들과도 데이터를 공유할 수 있기 때문이다. 개인적으로 만든 앱에서, 카메라 Activity를 시작하는 것이 하나의 예다. 만약 하나의 프로세스에서 다른 프로세스로 객체의 주소값을 넘긴다면 프로세스간에 의존성이 생겨버려 위험하다. 따라서 커스텀 객체를 넘길 때도 꼭 그 값을 넘겨야만 한다.

## Serializable

**Serializable은 표준 JAVA 인터페이스다.**

Serializable를 사용해서 액티비티간 데이터를 공유하는 방법은 쉽다. 클래스가 Serializable를 구현하도록 해놓으면 끝이다. 이렇게 하면 기본적인 방법으로 Serializable를 사용하는 것이다.

```java
package de.vogella.java.serilization;

import java.io.Serializable;

public class MySerializable implements Serializable {
    private int mData;

    public Person(int mData) {
        this.mData = mData;
    }

    public int describeContents() {
         return 0;
     }
}
```

간단하게 구현할 수 있지만, 그말은 시스템이 개발자 모르게 해주는 작업이 많다는 의미다. Serializable는 내부적으로 reflection을 사용하기 때문에, 프로세스 동작 중에 많은 객체들이 추가로 생성, 사용되고, 사용이 끝난 후에는 가비지 컬렉터가 열일하게 되어 앱의 성능이 낮아진다고 한다.

## Parcelable

**Parcelable은 안드로이드 SDK가 포함하고 있는 인터페이스다.** Parcelable은 Serializable이 했던 Reflection을 사용하지 않도록 설계되어 있기 때문에 그만큼 개발자가 수동적으로 해줘야 할 작업들이 있다.

```java
 public class MyParcelable implements Parcelable {
     private int mData;

     public int describeContents() {
         return 0;
     }

     public void writeToParcel(Parcel out, int flags) {
         out.writeInt(mData);
     }

     public static final Parcelable.Creator<MyParcelable> CREATOR
             = new Parcelable.Creator<MyParcelable>() {
         public MyParcelable createFromParcel(Parcel in) {
             return new MyParcelable(in);
         }

         public MyParcelable[] newArray(int size) {
             return new MyParcelable[size];
         }
     };

     private MyParcelable(Parcel in) {
         mData = in.readInt();
     }
 }
```

Serializable과는 달리 꼭 구현해야 하는 필수 메서드들이 존재한다. 이 필수 메서드들을 작성하게 되면 액티비티로 데이터를 넘길 자격이 주어진다.

Parcelable을 구현하는 과정이 귀찮아 보이지만, kotlin을 사용중이라면 귀찮은 작업이 줄어든다. `kotlin-android-extensions`플러그인을 사용하면 `@Parcelize`를 사용하여 아래와 같은 코드로 구현할 수 있다.

```kotlin
@Parcelize
class MyParcelable(
    private val mData: Int
): Parcelable {
     public int describeContents() {
         return 0;
     }
}
```

한 가지 주의해야 할 점은, 위와 같은 코드 작성시 주 생성자에 있는 데이터들만 직렬화가 된다는 점이다. 주생성자 데이터 외의 다른 데이터까지 직렬화 하고싶다면, [Parcelable 적용](https://nobase-dev.tistory.com/238) 블로그 글을 참조하자.

## 결론

Serializable와 Parcelable간에는 성능과 구현의 편리함이라는 trade off가 있다. 일부 글에서는 Serializable에서 직렬화 프로세스를 직접 구현하면 Parcelable의 성능을 뛰어 넘기에 성능과 편리함 모두 승리한다는 글이 있다. 그러나 @Serialize를 직접 구현하더라도 둘 사이의 성능 차이는 유의미하지 않는 것으로 보여지기 때문에, 상황에 따라 주생성자에 대부분의 데이터가 존재하는 객체는 `@Parcelize`를 사용하는 것도 좋을 것 같다. 상황에 따라 개발자 경험을 더 높여 줄 수 있다고 판단되는 것을 잘 선택하자.
