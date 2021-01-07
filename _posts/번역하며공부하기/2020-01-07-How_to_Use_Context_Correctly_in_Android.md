---
layout: post
title: How to Use Context Correctly in Android(번역)
category: 번역하며 공부하기
tags: [Android, Context]
comments: true
---

원본 - [How to Use Context Correctly in Android](https://maryam-dashti.medium.com/how-to-use-context-correctly-in-android-93e775cf27c5)

Android Context는 Android에서 가장 중요한 객체 중 하나입니다. Context객체는 application의 현재 상태에 대한 맥락이자 아래와 같은 중요한 책임을 가지고 있습니다.

- Activity와 Application에 대한 정보 제공
- database, application 관련 리소스(string, drawable, ...), 클래스들, 파일 시스템, shared preference 등등에 대한 접근 권한
- activity 런칭, 브로드캐스팅, intent 수신 등과 같은 application 레벨 기능 호출

보시다시피 안드로이드의 많은 부분이 Context를 필요로 합니다. 따라서 Context를 잘 못 사용한다면, 메모리 릭으로 이어질 수 도 있습니다. Context에는 크게 두 가지 타입으로 볼 수 있습니다.

### Application Context

이 Context는 application의 생명주기에 종속되어있습니다. 즉 application이 죽지 않고 살아있는 한, 사용 가능하다는 뜻이죠. 이 컨텍스트는 application 클래스의 `getApplicationContext()`함수를 통해 접근할 수 있는 싱글톤 객체입니다. 중요한 점은 UI와 관련된 컨텍스트가 아니라는 점입니다. 따라서 intent를 사용하여 activity를 시작하거나, toast를 보여준다거나, 기타 UI와 관련된 작업을 하신다면 application Context를 사용하시면 안됩니다. 한편 수명이 긴 객체나 쓰레드에서 activity 참조를 가지고 있으면 메모리 누수가 발생할 수 있으니 조심하세요. 이럴 때야 말로 application Context를 사용할 때입니다. 아래는 application context의 기능들입니다.

- resource value들을 Load
- service 시작
- service 바인딩
- braodcast 전송
- broadcastReceiver 등록

### Activity Context

Activity Context는 명확히 Activity의 생명 주기에 바인딩 되어 있기 때문에 액티비티가 살아있는 한 `getContext()` 메서드를 통해 접근할 수 있습니다. Activity Context는 오직 UI와 관련된 작업을 할 때 현재 살아있는 Activity에서 접근해야만 합니다. 그 예는 아래와 같습니다.

- resource value들을 Load
- 레이아웃 인플레이션(Layout Inflation)
- 액티비티 시작
- dialog 또는 toast 보여주기
- service 시작
- service 바인딩
- broadcast 전송
- braodcastReceiver 등록

---

위에서 언급한 두 가지 컨텍스트 외에서 `getBaseContext()` 와 `this` 를 사용하여 Context에 접근하는 것을 보셨을겁니다. `getBaseContext()` 는 ContextWrapper의 메서드인데요, ContextWrapper는 단순히 모든 context 호출을 다른 context로 위임하는 프록시 구현체입니다. original Context를 변경하지 않고 동작을 수정하기 위해서 subclassed 될 수 있습니다.

`getBaseContext()` 를 사용하면 ContextWrapper 클래스에 존재하는 Context를 가져올 수 있습니다.

또한 `this`는 아시다시피 객체를 참조하는 것인데요, Activity 내의 context가 필요할 때면 언제든지 사용할 수 있습니다. 아래는 Context를 요청하는 자바와 코틀린 코드입니다.

```kotlin
// 액티비티에서 다른 액티비티를 시작하려 한다면, `this`를 넘기세요.
val intent = Intent(this, <YourClassname>::class.java)
startActivity(intent)

//view.getContext() 는 현재 activity view를 참조합니다.
listView.setOnItemClickListener { parent, view, position, id ->
    val myElement = adapter.getItemAtPosition(position)
    val intent = Intent(view.getContext(), <YourClassname>::class.java)
    view.getContext().startActivity(intent)
}
```

```kotlin
public class ApplicationListAdapter extends RecyclerView.Adapter<
    ApplicationListAdapter.ApplicationListViewHolder> {
    ...

    @NonNull
    @Override
    public ApplicationListAdapter.ApplicationListViewHolder onCreateViewHolder(
        @NonNull ViewGroup parent, int viewType) {
        //the correct context can be inferred from the parent view as 'parent.getContext()' for inflation.
        View view =
            LayoutInflater.from(parent.getContext())
            .inflate(R.layout.list_row_application, parent, false);
        return new ApplicationListViewHolder(view);
    }
    ...

    public class ApplicationListViewHolder extends RecyclerView.ViewHolder {
        private final ImageView mApplication_icon;

        public ApplicationListViewHolder(@NonNull View itemView) {
            super(itemView);

            mApplication_icon = itemView.findViewById(R.id.application_image);
        }

        private void bind(Application application) {
            //Context is needed outside of the onCreateViewHolder() method which is retrieved via 'itemView.getContext()'.
            Glide.with(itemView.getContext())
                    .load(application.getIcon())
                    .placeholder(R.mipmap.ic_launcher)
                    .into(mApplication_icon);
        }
    }
    ...
}
```
