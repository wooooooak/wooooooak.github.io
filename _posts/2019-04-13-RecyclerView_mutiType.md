---
layout: post
title: 멀티 뷰 타입 RecyclerView(리사이클러뷰) 만들어보기 (feat. Kotlin)
category: Android
tags: [recycler_view, kotlin]
comments: true
---

[이전 포스트](https://wooooooak.github.io/android/2019/03/28/recycler_view/)에서는 아주 기본적인 리사이클러뷰의 사용법에 대해서 다뤄보았다. 따라서 뷰의 모양이 끝까지 일정한 채팅방 목록 리스트같은 경우 쉽게 구현할 수 있게 되었다.

이제는 여러개의 뷰 타입, 즉 리사이클러뷰 내에서 단 한 개의 뷰 형태만을 쭉쭉 랜더링하는게 아니라 **다수의 뷰 형태를 가지는 객체들을 랜더링하는 방법을 알아보자.** 말이 어렵다면... 아래 이미지와 같은 샘플 앱을 하나의 리사이클러뷰로 만들어볼 것이다.
![결과물미리보기](/public/img/android/recycler2_1.png)

아래로 스크롤을 내리면 카테고리 2번도 있는데...(풀 스크린 찍는 방법을 모르겠다ㅠ)

아무튼 이번 예제에서는 총 3개의 다른 뷰 형태를 가지는 리사이클러 뷰를 다뤄볼 것이다. 세가지 다른 뷰 형태란 아래와 같다.
* 단순 카테고리 명을 표시하는, 텍스트만 있는 뷰
* 글 아래에 이미지가 크게 박힌 뷰
* 사진 오른쪽에 사진 제목과 컨텐츠 내용이 담긴 뷰 

## 사전 준비
recyclerView와 cardView, material디자인을 설치할 것이다. 아래코드를 그냥 복사 붙여넣기해도 되지만, 시간이 지남에따라 버전업 되는 부분은 알아서 할수 있으리라 믿는다!
```gradle
    implementation 'androidx.recyclerview:recyclerview:1.0.0'
    implementation 'com.google.android.material:material:1.0.0'
    implementation 'androidx.cardview:cardview:1.0.0'
```
또한, 코드중에 snow라는 이름의 이미지 파일을 사용하는데, 인터넷에 아무 사진이나 다운받아서 프로젝트에 넣어도 무방하고, [깃헙](https://github.com/wooooooak/recyclerView2-tutorial)의 예제코드를 그대로 다운받아서 실행해봐도 무방하다.

## 프로젝트 구조
![프로젝트구조](/public/img/android/recycler2_5.png)

## layout
우선 가장 먼저 리사이클러뷰를 담을 메인 layout은 아래와 같다.
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
        xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:tools="http://schemas.android.com/tools"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".MainActivity">

    <androidx.appcompat.widget.Toolbar
            android:layout_width="match_parent"
            android:layout_height="50dp"
            app:layout_constraintTop_toTopOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            android:id="@+id/toolbar">
        <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="Toolbar"
                android:gravity="center"
        />
    </androidx.appcompat.widget.Toolbar>

    <androidx.recyclerview.widget.RecyclerView
            android:id="@+id/recycler_view"
            android:layout_width="0dp"
            android:layout_height="0dp"
            app:layout_constraintTop_toBottomOf="@id/toolbar"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
    />
</androidx.constraintlayout.widget.ConstraintLayout>
```

toolbar를 사용하기 때문에 style파일로 들어가서 아래와 같이 `NoActionBar`로 수정해주어야한다.

```xml
<resources>

    <!-- Base application theme. -->
    <style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
    </style>

</resources>
```

위에서 언급한 3가지 뷰 타입에 대해서 각각의 레이아웃을 살펴보자.

#### 카테고리명을 담당할 뷰 레이아웃 (text_type.xml)
```xml
<LinearLayout
        xmlns:card_view="http://schemas.android.com/apk/res-auto"
        xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@+id/card_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="8dp"
        android:layout_marginStart="24dp"
>
    <TextView
            android:id="@+id/title"
            android:layout_width="match_parent"
            android:textAlignment="textStart"
            android:layout_height="wrap_content"
            android:textColor="@color/colorTitle"
            android:textSize="20sp"
    />
</LinearLayout>
```
#### 제목 아래에 사진이 크게 박혀있는 뷰 레이아웃 (image_type)
```xml
<com.google.android.material.card.MaterialCardView
        xmlns:card_view="http://schemas.android.com/apk/res-auto"
        xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@+id/card_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_margin="16dp"
        card_view:cardElevation="10dp">
    <LinearLayout
            android:layout_width="match_parent"
            android:orientation="vertical"
            android:layout_height="wrap_content">
        <TextView
                android:id="@+id/title"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:padding="10dp"
        />
        <ImageView
                android:id="@+id/background"
                android:layout_width="match_parent"
                android:layout_height="150dp"
                android:scaleType="centerCrop"
                android:src="@drawable/snow"
        />
    </LinearLayout>
</com.google.android.material.card.MaterialCardView>
```

#### 진 오른쪽에 사진 제목과 컨텐츠 내용이 담긴 뷰 레이아웃 (image_type2.xml)
```xml
<com.google.android.material.card.MaterialCardView
        xmlns:card_view="http://schemas.android.com/apk/res-auto"
        xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:tools="http://schemas.android.com/tools" android:id="@+id/card_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_margin="16dp"
        card_view:cardElevation="10dp">

    <androidx.constraintlayout.widget.ConstraintLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content">

        <ImageView
                android:layout_width="181dp"
                card_view:srcCompat="@drawable/snow"
                android:id="@+id/imageView2"
                card_view:layout_constraintTop_toTopOf="parent" card_view:layout_constraintStart_toStartOf="parent"
                card_view:layout_constraintBottom_toBottomOf="parent"
                card_view:layout_constraintVertical_bias="0.0" android:layout_height="125dp"/>
        <TextView
                android:text="제목"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content" card_view:layout_constraintTop_toTopOf="parent"
                android:id="@+id/titleView" card_view:layout_constraintStart_toEndOf="@+id/imageView2"
                android:layout_marginStart="8dp" card_view:layout_constraintEnd_toEndOf="parent"
                android:layout_marginEnd="8dp" card_view:layout_constraintHorizontal_bias="0.524"
                android:layout_marginTop="28dp" android:textAppearance="@style/TextAppearance.AppCompat.Subhead"
        />
        <TextView
                android:text="내용"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                card_view:layout_constraintBottom_toBottomOf="parent"
                android:id="@+id/contentView" card_view:layout_constraintStart_toEndOf="@+id/imageView2"
                android:layout_marginStart="8dp" card_view:layout_constraintEnd_toEndOf="parent"
                android:layout_marginEnd="8dp" android:layout_marginTop="8dp"
                card_view:layout_constraintTop_toBottomOf="@+id/titleView"
                card_view:layout_constraintHorizontal_bias="0.512" card_view:layout_constraintVertical_bias="0.102"/>
    </androidx.constraintlayout.widget.ConstraintLayout>
</com.google.android.material.card.MaterialCardView>
```

프로젝트에 필요한 세가지 뷰 레이아웃 작성은 모두 끝났다. 이제 [이전 포스트](https://wooooooak.github.io/android/2019/03/28/recycler_view/)에서도 그랬듯이 데이터를 만들고, 어댑터를 만들어주고, 레이아웃 매니저만 정해주면 끝난다.

## Kotlin Code
### model
`Model.kt` 클래스는 랜더링 하고 싶은 데이터를 가지고 있을 클래스이다.
```kotlin
data class Model(val type: Int, val text: String, val data: Int, val contentString: String?) {
    companion object {
        const val TEXT_TYPE = 0
        const val IMAGE_TYPE = 1
        const val IMAGE_TYPE_2 = 2
    }
}
````
* **첫 번째 인자 값** : 우리가 만든 3가지 형태의 뷰들 중, 어떤 형태의 뷰인지 Int값으로 넘겨줄 것이다. 그 Int은 `ViewTypeEnum` 을 사용한다.
* **두 번째 인자 값** : 텍스트를 입력받을 파라미터이다. 텍스트 하나만 랜더링 하는 뷰에서는 그 텍스트를, 텍스트1개 이미지1개인 뷰에서 텍스트를, 텍스트2 이미지1개인 뷰에서는 제목 부분을 담당할 String이다.
* **세 번째 인자 값** : 이미지가 필요한 뷰라면 이미지를 넣어줄 파라미터.
* **네 번째 인자 값** : 텍스트2 이미지1개인 `image_type1.xml` 뷰에서 제목 아래의 컨텐츠 부분의 값을 담당할 String이다.

### MainActivity
`MainActivity`는 [이전 포스트](https://wooooooak.github.io/android/2019/03/28/recycler_view/)와 동일하게 어댑터와 레이아웃 매니저를 설정해주는 부분이다. 더불어 데이터 리스트에 값을 만들어 어댑터에 넣어줄 것이다.

```kotlin
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        setSupportActionBar(toolbar)

        val list = mutableListOf<Model>().apply {
            add(Model(Model.TEXT_TYPE, "카테고리 1번!", 0, null))
            add(Model(Model.IMAGE_TYPE, "텍스트뷰 아래에 이미지가 있는 뷰타입.", R.drawable.snow, null))
            add(Model(Model.IMAGE_TYPE_2, "안녕, 제목부분이 될거야", R.drawable.snow, "내용부분!"))
            add(Model(Model.IMAGE_TYPE, "다시 한 번 텍스트 옆에 이미지가 있는 뷰타입", R.drawable.snow, null))
            add(Model(Model.IMAGE_TYPE_2, "제목2!!", R.drawable.snow, "사진에 대한 설명?"))

            add(Model(Model.TEXT_TYPE, "카테고리 2번!", 0, null))
            add(Model(Model.IMAGE_TYPE, "새로운 카테고리 시작!.", R.drawable.snow, null))
            add(Model(Model.IMAGE_TYPE, "다음생엔 울창한 숲의 이름모를 나무로 태어나 평화로이 살다가 누군가의 유서가 되고 싶다.", R.drawable.snow, null))
            add(Model(Model.IMAGE_TYPE_2, "제목부분.", R.drawable.snow, "내용부분"))
        }

        val adpater = MultiViewTypeAdapter(list, this)
        recycler_view.layoutManager = LinearLayoutManager(this, RecyclerView.VERTICAL, false)
        recycler_view.adapter = adpater
    }
}
```

### 대망의 MultiViewTypeAdapter
어쩌면 [이전 포스트](https://wooooooak.github.io/android/2019/03/28/recycler_view/)를 보던 중
```kotlin
override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): MyViewHolder {
    val view = LayoutInflater.from(parent.context).inflate(R.layout.item, parent, false)
    Log.d("tag1" , "onCreateViewHolder")
    return MyViewHolder(view)
}
````
이부분의 `viewType`인자가 무엇인지 궁금했을 분이 계셨을 수도 있을것 같다. 정말 섬세하신분...

`viewType`변수명에서 느낄 수 있듯이 viewType이 구분되어 들어오는 값이다. 이 viewType은 어디서 넘어올까?

`onCreateViewHolder`가 호출되기 전, `getItemViewType(position: Int): Int`함수가 먼저 호출되어 리턴 값이 넘겨지는 것이다. 따라서 우리는 이 함수에서 적절히 뷰타입을 구분하여 리턴해주면 된다. 뷰타입을 구분하는 방법이야 여러가지가 있겠지만, 우리는 처음 `Model`객체를 생성할 때 부터 첫 번째 인자로 `type` 값을 구분해서 넣어줬으므로 그대로 넘겨주면 된다.

```kotlin
class MultiViewTypeAdapter(private val list: MutableList<Model>) :
    RecyclerView.Adapter<RecyclerView.ViewHolder>() {

    private var totalTypes = list.size

    // getItemViewType의 리턴값 Int가 viewType으로 넘어온다.
    // viewType으로 넘어오는 값에 따라 viewHolder를 알맞게 처리해주면 된다.
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): RecyclerView.ViewHolder {
        val view: View?
        return when (viewType) {
            Model.TEXT_TYPE -> {
                view = LayoutInflater.from(parent.context).inflate(R.layout.text_type, parent, false)
                TextTypeViewHolder(view)
            }
            Model.IMAGE_TYPE -> {
                view = LayoutInflater.from(parent.context).inflate(R.layout.image_type, parent, false)
                ImageTypeViewHolder(view)
            }
            Model.IMAGE_TYPE_2 -> {
                view = LayoutInflater.from(parent.context).inflate(R.layout.image_type2, parent, false)
                ImageTypeView2Holder(view)
            }
            else -> throw RuntimeException("알 수 없는 뷰 타입 에러")
        }
    }

    override fun getItemCount(): Int {
        return totalTypes
    }

    override fun onBindViewHolder(holder: RecyclerView.ViewHolder, position: Int) {
        Log.d("MultiViewTypeAdapter", "Hi, onBindViewHolder")
        val obj = list[position]
        when (obj.type) {
            Model.TEXT_TYPE -> (holder as TextTypeViewHolder).txtType.text = obj.text
            Model.IMAGE_TYPE -> {
                (holder as ImageTypeViewHolder).title.text = obj.text
                holder.image.setImageResource(obj.data)
            }
            Model.IMAGE_TYPE_2 -> {
                (holder as ImageTypeView2Holder).title.text = obj.text
                holder.content.text = obj.contentString
                holder.image.setImageResource(obj.data)
            }
        }
    }

    // 여기서 받는 position은 데이터의 index다.
    override fun getItemViewType(position: Int): Int {
        Log.d("MultiViewTypeAdapter", "Hi, getItemViewType")
        return list[position].type
    }

    inner class TextTypeViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
        val txtType: TextView = itemView.findViewById(R.id.title)
    }

    inner class ImageTypeViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
        val title: TextView = itemView.findViewById(R.id.title)
        val image: ImageView = itemView.findViewById(R.id.background)
    }

    inner class ImageTypeView2Holder(itemView: View) : RecyclerView.ViewHolder(itemView) {
        val title: TextView = itemView.findViewById(R.id.titleView)
        val content: TextView = itemView.findViewById(R.id.contentView)
        val image: ImageView = itemView.findViewById(R.id.imageView2)
    }
}
```

결국 뷰홀더를 여러개 만들고, `onCreateViewHolder`에서 데이터에 따라 그에 맞는 뷰홀더를 생성해주는게 전부이다. 

사실 뷰 타입을 `Model`의 `compaion object`로, 또 숫자로 관리한다는 게 좋진 않지만 여러개의 뷰 타입을 다루는 리사이클러 뷰를 공부하는데는 지장이 없다. 더 좋은 코드로 리팩토링 하고자 한다면, [github](https://github.com/wooooooak/recyclerView2-tutorial)에서 소스를 다운받은 후, [Android RecyclerView Multiple Layout Sealed Class 포스팅 - 영문](https://www.codexpedia.com/android/android-recyclerview-multiple-layout-sealed-class/)이나, [Kotlin Sealed class를 사용한 UI 상태 관리 포스팅](https://medium.com/@lazysoul/kotlin-sealed-class%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%9C-ui-%EC%83%81%ED%83%9C-%EA%B4%80%EB%A6%AC-1-3-98cf37207c13)을 읽고 스스로 리팩토링 해보는 것도 좋은 방법이 될 것 같다.

## 끄
읕