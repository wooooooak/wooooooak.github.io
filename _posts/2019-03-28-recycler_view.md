---
layout: post
title: RecyclerView의 원리와 사용법(feat. Kotlin)
category: Android
tags: [recycler_view, kotlin]
comments: true
---
카카오톡의 채팅 대화방, pinterest 앱의 수 많은 리스트 데이터들을 효율적으로 렌더링 하기 위해서 안드로이드에서는 어떤 방법을 사용할까? 기본적으로 제공되는 ListView가 있지만, 최근들어 구글에서 공식적으로 제공하고 있는 Recycler View를 사용해 이들을 구현한다. 

Recycler View는 ListView가 할 수 있는 모든 일을 할 뿐만 아니라 커스터마이징도 쉬우며, 효율성도 더 좋다. 그럼 Recycler view가 왜 더 좋은지 알아보고, 이를 활용해 간단히 리스트 데이터들을 렌더링 하는 방법을 알아보자.

## Recycler view의 재활용성
![recycler_view1](/public/img/android/recycler1.png)
개인적으로 위의 그림이 RecyclerView를 이해하는 데 도움이 많이 되었다.

그림을 이해하자면, ListView와는 다르게 RecyclerView는 이름에서 알 수 있듯이 재활용이 가능한 뷰이다. 무엇을 재활용 할까? 오른 쪽 그림을 보자. 파란색 라인 한 개가 채팅방 리스트 한 개라고 가정하자. 전체 채팅방 리스트는 100개가 훌쩍 넘을 수가 있다. 그러나 정작 화면에 보여지는 채팅방 목록은 한 번에 10개 조차 되지 않는다. 

매번 사용자가 아래로 스크롤 할 때 마다 맨 위에 위치한 뷰 객체가 새로 삭제되고, 아랫 부분에서 새로 나타날 채팅방 뷰 객체를 새로 생성하면 결국 100개의 뷰 객체가 삭제되고 생성되는 것일 뿐만 아니라, 스크롤을 위아래로 왔다 갔다 하면 수 백개의 뷰 객체가 새로 생성되고 삭제됨을 반복한다.

리사이클러 뷰는 사용자가 아래로 스크롤 한다고 가정했을 때, 맨 위에 존재해서 이제 곧 사라질 뷰 객체를 삭제 하지않고 아랫쪽에서 새로 나타나날 파란색 뷰 위치로 객체를 이동시킨다. 즉 뷰 객체 자체를 재사용 하는 것인데, 즁요한 점은 뷰 객체를 재사용할 뿐이지 뷰객체가 담고 있는 데이터(채팅방 이름)는 새로 갱신된다는 것이다. 어쨋거나 뷰 객체를 새로 생성하지는 않으므로 효율적인 것이다. 

결과적으로 보자면, 맨 처음 화면에 보여질 10개 정도의 뷰 객체만을 만들고, 실제 데이터가 100개든 1000개든 원래 만들어 놓은 10개의 객체만 계속 해서 재사용 하는 것이다. 말만 들어도 효율적으로 보이지 않는가?

### ViewHolder
위에서 설명 햇듯이, 스크롤을 밑으로 내릴 때, 맨 위에 존재해서 이제 곧 사라질 파란색 뷰 객체는 맨 아래로 이동하여 재활용된다. 즉, 10개의 뷰 객체만 계속해서 위에서 아래로 이동하면서 재사용 되는 것이며, 우리는 딱 10개 정도의 뷰 객체만을 만들어서 가지고 있으면 된다. 10개의 뷰 객체들은 언제든 text라던지 이미지가 바뀔 수 있다(뷰 객체를 재사용 할 뿐이지 재사용 될 때의 데이터는 계속 바껴야 하기 때문이다). 

따라서 맨 처음 10개의 뷰객체를 기억하고 있을(홀딩) 객체가 필요한데 이 것이 ViewHolder이다. 나중에 전체 코드를 보겠지만, ViewHolder 코드 부분만 보자면 아래와 같다.

![recycler_view1](/public/img/android/re2.png)
```kotlin
class MyViewHolder(view: View): RecyclerView.ViewHolder(view) {
    var textField = view.chat_title
}
```
위 사진의 textView라고 적힌 TextView id 값은 `chat_title`이다.

우리는 저기 보이는 한 줄의 채팅방 목록의 TextView에 각자 다른 채팅방 이름을 부여할 것이며, 100개의 채팅방 목록을 나열해 볼 것이다. 그러나 100개의 뷰 객체를 하나 하나 모두 만들어 주진 않을 것이며 딱 10개 내외의 MyViewHolder 객체를 만들어서 계속 재사용 해 줄 것이다. **MyViewHolder가 `textField`를 가지고 있음을 기억하자. 스크롤을 내릴 때 마다 이 부분에 데이터만 바꿔주면 뷰객체는 그대로 이면서 데이터만 바뀌게 되는 것이다.**

## Adapter와 LayoutManager
리사이클러 뷰를 사용하려면 두 가지가 필수적으로 필요하다. `Adpater`와 `LayoutManager`이다. 

`Adapter`는 100개의 채팅방 제목 이름이 담긴 리스트를 리사이클러 뷰에 바인딩 시켜주기 위한 사전 작업이 이루어 지는 객체이다. 직접 작성해서 리사이클러뷰에 적용시키면 된다.

`LayoutManager`는 많은 역할을 하지만 간단하게 스크롤을 위아래로 할지, 좌우로 할지를 결정하는 것이라고 생각하자. 실제로는 더 많은 역할을 하지만 자세한건 구글링...ㅎ

이 두가지를 적용하기 전에 100개의 채팅방 제목을 가진 리스트를 생성하고, 리사이클러 뷰에 적용해보자.
```kotlin
// MainActivity
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_recycler_view)

    val datas = Array(100) {
        "chat $it"
    }

    recycler_view.adapter = MyApdater(datas)
    recycler_view.layoutManager = LinearLayoutManager(this)
}
```

이제 핵심이 되는 `MyAdapter`를 구현해보자.

### MyAdapter
이 부분은 우선 코드부터 보자.
```kotlin
class MyApdater(private var datas: Array<String>) : RecyclerView.Adapter<MyViewHolder>() {

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): MyViewHolder {
        val view = LayoutInflater.from(parent.context).inflate(R.layout.item, parent, false)
        Log.d("tag1" , "onCreateViewHolder")
        return MyViewHolder(view)
    }

    override fun getItemCount(): Int {
        return datas.size
    }

    override fun onBindViewHolder(holder: MyViewHolder, position: Int) {
        Log.d("tag1" , "onBind")
        holder.textField.text = datas[position]
    }
}

class MyViewHolder(view: View): RecyclerView.ViewHolder(view) {
    var textField = view.chat_title
}
```

`MyAdapter`의 생성자로 리스트로 뿌려줄 데이터 배열을 받자. 그리고 `MyAdapter`는 `RecyclerView.Adapter`를 상속 받는데, 제네릭 타입으로 `ViewHolder`를 넣어주어야 한다. 코드에서 넘겨준 `MyViewHolder`는 맨 아랫줄에 작성되어 있다.

#### getItemCount
가장 먼저 실행되는 함수는 `getItemCount`이다. 여기서는 우리가 뿌려줄 데이터의 전체 길이를 리턴하면 된다. 더 깊게 알아볼 필요는 없는 것 같다.
```kotlin
override fun getItemCount(): Int {
    return datas.size
}
```

#### onCreateViewHolder
`getItemCount`다음으로 호출되는 함수는 `onCreateViewHolder`함수이다. 이름에서 알 수 있듯이 `ViewHolder`가 생성되는 함수다. 여기서 `ViewHolder`객체를 만들어 주면 된다. 

```kotlin
override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): MyViewHolder {
    val view = LayoutInflater.from(parent.context).inflate(R.layout.item, parent, false)
    Log.d("tag1" , "onCreateViewHolder")
    return MyViewHolder(view)
}
```

위에서 언급했듯이 맨 처음 화면에 보이는 전체 리스트 목록이 딱 10개라면, 위아래 버퍼를 생각해서 13~15개 정도의 뷰 객체가 생성된다. 정확하게 말하자면 뷰 객체를 담고 있는 `ViewHolder`가 생성되는 것이다. 그래서 `onCreateViewHolder`함수는 딱 13~15번 정도만 호출되고 더 이상 호출되지 않는다.

`return`되는 곳에서 MyViewHolder의 생성자에 view 객체를 넘겨주는데, 이 view객체는 아까 사진에서 본 한개의 채팅방 목록이 디자인 되어있는 레이아웃이다. 즉 `viewHolder`는 그 레이아웃을 인자로 받아서 기억하고 있는 것이다. 이제는 계속해서 재사용되는 뷰 홀더(레이아웃)들에 데이터를 바인딩 해주는 작업만 남았다.

### onBindViewHolder
`onBindViewHolder`함수는 생성된 뷰홀더에 데이터를 바인딩 해주는 함수이다. 이름이 참 직관적이여서 좋다.

예를 들어 데이터가 스크롤 되어서 맨 위에있던 뷰 홀더(레이아웃) 객체가 맨 아래로 이동한다면, 그 레이아웃은 재사용 하되 데이터는 새롭게 바뀔 것이다. 고맙게도 아래에서 새롭게 보여질 데이터의 인덱스 값이 `position`이라는 이름으로 사용가능하다.
``` kotlin
override fun onBindViewHolder(holder: MyViewHolder, position: Int) {
    Log.d("tag1" , "onBind")
    holder.textField.text = datas[position]
}
```
즉 아래에서 새롭게 올라오는 데이터가 리스트의 20번째 데이터라면 position으로 20이 들어오는 것이다.

`onCreateViewHolder`는 `ViewHolder`를 만들기 위해 13~15번 정도밖에 호출되지 않지만, `onBindViewHolder`는 스크롤을 해서 데이터 바인딩이 새롭게 필요할 때 마다 호출된다. 스크롤을 무한정 돌린다면, `onBindViewHolder`도 무한정 호출된다. 무한정 호출된다 하더라도 우리는 딱 13~15개의 뷰 객체만 사용하는 꼴이다.

### 결과와 로그
결과물은 아래와 같다.
![recycler_view1](/public/img/android/re3.png)
사진 아래로 89개의 데이터가 더 있다. 그러나 코드에서 로그를 찍어 놓아기 때문에 우리는 뷰가 몇 개만 호출되었는지 볼 수 있다.
![recycler_view1](/public/img/android/re4.png)
위에서 예시로 든 숫자들과는 조금 다르지만 모바일 화면과 한 개의 데이터 높이에 따라서 숫자는 얼마든지 달라질 수 있다. 이 앱에서 리스트가 총 12개가 표현이 되므로, 위아래 버퍼를 생각해서 `onCreateViewHolder`는 딱 17번만 호출된다. 그리고 생성됨과 동시에 데이터 바인딩을 해줘야 하므로 `onBind`로그가 찍히는 것을 볼 수있다.

첫 화면 이후 스크롤을 아래로 이동시키면 더 이상 `onCreateViewHolder`는 호출되지 않고 `onBind`로그만 찍히는 것을 알 수 있다. ViewHolder를 계속 만들지 않고 재사용하기 때문에 데이터만 새롭게 바인딩 해주는 것이다.

## 끄
읕!