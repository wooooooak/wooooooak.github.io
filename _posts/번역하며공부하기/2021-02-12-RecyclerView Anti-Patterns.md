---
layout: post
title: RecyclerView Anti-Patterns(번역)
category: 번역하며 공부하기
tags: [RecyclerView]
comments: true
---

원문 - [RecyclerView Anti-Patterns](https://proandroiddev.com/recyclerview-antipatterns-8af3feeeccc7)

안드로이드의 `RecyclerView`는 기존 `ListView`를 대체하면서도 굉장히 유용한 first-party 라이브러리입니다. 저는 지금까지 RecycerView의 안티 패턴들을 종종 보았고, adapter 컨셉을 잘 못 이해하고 짠 코드들을 봐왔습니다. 이와 관련된 코드들을 리뷰해본 경험, 그리고 제 후배에게 해준 상세한 설명을 바탕으로, 이와 관련된 내용을 여러분께 공유하고자 합니다. 이는 안드로이드 개발자라면 꼭 알아야만 하는 컨셉입니다.

## 석기 시대

`RecyclerView` 내부에서 일어나는 일들을 파악하기 위해서는, `RecyclerView` 없이 그렇게 동작하도록 구현해보면 됩니다. 아마도 `BaseAdapter`를 상속한 어떤 클래스를 구현해 보았다면 아래와 같은 코드를 보신적 있을거에요. 예를 들어, 커스텀 Spinner Adapter같은거 말이죠. 오직 `TextView` 하나만 보여주는 Adapter 구현을 살펴볼까요?

```kotlin
class ListViewAdapter(context: Context) : ArrayAdapter<Data>(context, R.layout.item) {

    //..Other overrides

    override fun getView(position: Int, convertView: View?, parent: ViewGroup): View {
        val itemAtPosition = getItem(position)!!
        //Inflate
        var itemView = LayoutInflater.from(parent.context).inflate(R.layout.item, parent, false)

        //Bind
        val tvText = itemView!!.findViewById<TextView>(R.id.textView)
        tvText.text = itemAtPosition.text

        return itemView
    }
}
```

잘 동작하는 코드이긴 한데요, 혹시 잘못된 점이 보이시나요? 잘 보시면 매번 view를 inflating하고 있어요. 이는 `ListView`를 스크롤 할 때 마다 성능에 큰 영향을 미칩니다. 이를 최적화 하기 위해서 `Adapter` 인터페이스, 특히 `getView` 메서드를 살펴볼 필요가 있습니다. `convertView` 파라미터는 nullable이고, 주석은 아래와 같아요.

```kotlin
재사용 될 수도 있는 old view.
주의: 이것을 사용할 때는 non-null인지 확인해야 하고, 올바른 타입을 확인해야합니다.
만약 이 view가 올바른 데이터를 뿌리도록 변환할 수 없다면, 이 메서드는 새로운 view를 생성할 수 있습니다.
```

"재사용 될 old view"라고 언급되어있습니다. adapter는 사용자 화면 밖으로 나간 view를 재생성하는 게 아니라 **재활용(재사용)** 합니다. 이는 사용자가 더 부드럽게 스크롤 할 수 있게끔 해줍니다.

![Checkout excalidraw!](https://user-images.githubusercontent.com/18481078/107643403-6f7acb00-6cb9-11eb-93a9-763d6ed96f4a.png)

그러므로 `Adpater`의 재활용 기능을 사용하도록 코드를 최적화해봅시다. 먼저 `convertView`가 null인지 아닌지를 확인하고, 오직 null일 때만 view를 inflate합니다. 만약 **null이 아니라면, 우리는 재활용된 view를 얻었다는 것이고 inflate해줄 필요가 없어집니다.**

```kotlin
val itemView : View
if (convertView == null) {
    itemView = LayoutInflater.from(parent.context).inflate(R.layout.item, parent, false)
} else {
    itemView = convertView
}
```

이제 우리는 오직 뷰가 재활용 되지 않았을 때에만 infalte합니다. 즉, 처음에만 뷰를 생성해주는 거죠. 전체 코드는 아래와 같습니다.

```kotlin
override fun getView(position: Int, convertView: View?, parent: ViewGroup): View {
    val itemAtPosition = getItem(position)!!
    //Inflate
    val itemView : View

    if (convertView == null) {
        itemView = LayoutInflater.from(parent.context).inflate(R.layout.item, parent, false)
    } else {
        itemView
    }

    //Bind
    val tvText = itemView.findViewById<TextView>(R.id.textView)
    tvText.text = itemAtPosition.text

    return itemView
}
```

여기서 조금 더 최적화할 수 있습니다. 현재는 모든 아이템마다 `findViewById`를 사용하고 있습니다. 이것 보다 더 좋은 방법이 있는데요, 이 코드 부분을 view를 inflation 하고 난 직후에만 수행하도록 해봅시다. 이를 위해 나오는 패턴이 바로 `ViewHolder` 패턴입니다. view의 레퍼런스를 저장할 클래스 하나를 만들어 볼게요.

```kotlin
inner class ViewHolder {
  lateinit var tvText : TextView
}
```

그리고 View의 `setTag` 함수를 사용할겁니다. 뷰가 처음 생성될 때, 새로운 `ViewHolder` 객체를 만들고, item vie의 tag에 `ViewHolder`를 할당합니다. 다음번에 view가 재사용 될때는, 단지 tag를 가져와서 `ViewHolder` 타입으로 형변환을 해주면됩니다. 자, 이제 우리는 오직 inflation이 처음 일어날 때만 `findViewById`를 수행하게 되었습니다.

```kotlin
override fun getView(position: Int, convertView: View?, parent: ViewGroup): View {
    val itemAtPosition = getItem(position)!!

    //Inflate
    val viewHolder : ViewHolder
    val itemView : View

    if (convertView == null) {
        itemView = LayoutInflater.from(parent.context).inflate(R.layout.item, parent, false)

        viewHolder = ViewHolder()
        viewHolder.tvText = itemView.findViewById(R.id.textView)

        itemView.tag  = viewHolder
    } else {
        itemView = convertView
        viewHolder = itemView.tag as ListViewAdapter.ViewHolder
    }

    viewHolder.tvText.text = itemAtPosition.text

    return itemView
}
```

여기서 우리는, 조금더 실수를 방지하도록 코드를 수정할 수 있는데요, `findViewById` 로직을 `ViewHolder` 내부로 옮기고, `tvTex`를 불변값으로 선언합니다.

```kotlin
inner class ViewHolder(val itemView: View) {
    val tvText : TextView = itemView.findViewById(R.id.textView)
}
//In getView, use as follows:
viewHolder = ViewHolder(itemView)
```

이렇게 하면, 만약 `tvText` 할당을 까먹었더라도 에러를 내뿜지 않겠죠. 이렇게 해서 Adatper가 뷰를 재사용하고, inflation을 방지하며, `findViewById`를 매번 호출하지 않는 코드를 만들어보았습니다. 그러나 최적화 이점을 얻기 위한 이 모든 과정이 쉽지만은 않습니다. ListView를 만들 때 마다 이와같은 로직을 다시 작성해야할텐데, 이는 명백히 boilerplate입니다. 따라서 이를 쉽게 작성하도록 추상 클래스를 작성할 수도 있습니다.

```kotlin
abstract class AbstractListViewAdapter<T: Any,VH : AbstractListViewAdapter.ViewHolder>(
    context: Context,
    resId: Int
) : ArrayAdapter<T>(context, resId) {

    abstract class ViewHolder(val itemView: View)

    override fun getView(position: Int, convertView: View?, parent: ViewGroup): View {
        val itemAtPosition = getItem(position)!!

        //Inflate
        val viewHolder: VH
        val itemView: View

        if (convertView == null) {
            viewHolder = onCreateViewHolder(parent, getItemViewType(position))
            itemView = viewHolder.itemView
            itemView.tag = viewHolder
        } else {
            itemView = convertView
            viewHolder = itemView.tag as VH
        }

        onBindViewHolder(viewHolder, position)

        return itemView
    }

    abstract fun onBindViewHolder(viewHolder: VH, position: Int)

    abstract fun onCreateViewHolder(parent: ViewGroup, viewType: Int): VH

}
```

이 추상 클래스는 실제로 `RecycerView.Adapter`가 하는 일의 일부분입니다. `RecyclerView`는 우리가 `ListView` adatper를 위해 작성했던 boilerplate 코드들을 모두 다룹니다. 물론 `RecyclerView`는 그것보다 [더 많은 일](https://android.jlelse.eu/anatomy-of-recyclerview-part-1-a-search-for-a-viewholder-404ba3453714)을 하지만, 오늘은 더 많은 것을 다루진 않을 예정입니다.

## 청동기 시대

컨셉은 알았으니, 이제 `RecyclerView`의 anti pattern을 설명해드릴게요. 첫 번째는 view를 완전히 재사용하지 않는 다는 것입니다. 예를들어 아래의 코드를 살펴볼게요.

```kotlin
class RecyclerViewAdapter(
    private val onItemClick : (Data) -> Unit
) : RecyclerView.Adapter<RecyclerViewAdapter.MyViewHolder>() {

    //..Other overrides
    private val itemList: List<Data> = //...DO STUFFS

    inner class MyViewHolder(val itemView: View) : RecyclerView.ViewHolder(itemView) {
        val tvText : TextView = itemView.findViewById(R.id.textView)
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): MyViewHolder {
        val itemView = LayoutInflater.from(parent.context).inflate(R.layout.item, parent, false)
        return MyViewHolder(itemView)
    }

    override fun onBindViewHolder(holder: MyViewHolder, position: Int) {
        val itemAtPosition = itemList[position]
        holder.tvText.text = itemAtPosition.text
        holder.tvText.setOnClickListener {
            onItemClick(itemAtPosition)
        }
    }

}
```

여기서 재활용 할 수 있는 부분이 어디일까요? 바로 `OnClickListener`입니다. 현재는 매번 새로운 listener를 설정해주고 있는데요, 만약 `ViewHolder` 내부 초기화 시기에 이를 설정하거나, `onCreateView`에서 하면 어떨까요. 그럼 오직 한 번만 실행될 것이고 나중에 재활용할 수 있을 것입니다. 그리고 데이터 클래스 전체를 콜백으로 내보내는 대신, 오직 ViewHolder의 position만 리턴할 수 있습니다. 이렇게 하여 `ViewHolder` 밖의 로직이 position으로부터 데이터를 가져올 수 있게 합니다.

```kotlin
inner class MyViewHolder(
    itemView: View,
    private val onTextViewTextClicked: (position: Int) -> Unit
) : RecyclerView.ViewHolder(itemView) {
    val tvText: TextView = itemView.findViewById(R.id.textView)
    init {
        tvText.setOnClickListener {
            onTextViewTextClicked(adapterPosition)
        }
    }
}
```

호출자에게 `itemList`를 노출하여 `itemList[index]`와 같이 사용하도록 하는 대신에, adapter 내부의 로직을 캡슐화 하겠습니다. adapter는 `adapterPosition`을 가지고 그 위치에 맞는 데이터로 변경할 수 있는 item list를 이미 알고 있습니다. 이를 통해 다른 콜백 함수를 노출시키고 호출자로 데이터를 리턴합니다.

```kotlin
//onItemClick is a parameter in Adapter constructor
private val onTextViewTextClicked = { position: Int ->
    onItemClick.invoke(itemList[position])
}

override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): MyViewHolder {
    val itemView = LayoutInflater.from(parent.context).inflate(R.layout.item, parent, false)
    return MyViewHolder(itemView, onTextViewTextClicked)
}
```

이렇게 하여 `OnItemClickListener`도 재사용되고 로직은 여전히 adapter 내부에 캡슐화됩니다.

두 번째 anti-pattern은 adapter 내부에 로직을 가지고 있는 것입니다. adapter와 ViewHolder는 오직 ViewHolder를 사용자에게 보여주는 작업만 할 뿐 그 외에 어떤일도 해서는 안됩니다. 로직은 호출자로 떠넘겨야 합니다. 아래는 adapter 내부에 로직이 있는 코드 샘플입니다.

```kotlin
override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): MyViewHolder {
    val itemView = LayoutInflater.from(parent.context).inflate(R.layout.item, parent, false)
    return MyViewHolder(
        itemView = itemView,
        onTextViewTextClicked = { position: Int ->
            val itemAtIndex = itemList[position]
            val intent = getDetailActivityIntent(itemAtIndex)
            parent.context.startActivity(intent)
        })
}
```

나중에 우리는 같은 UI지만 item을 클릭했을 때 수행되는 액션이 다를 경우가 있을겁니다. 그러나 이런 상태라면 adapter 내부에 로직이 들어가있으므로 같은 UI를 대상으로 adapter를 재사용할 수가 없겠죠. 따라서 우리는 callback/interface로 로직을 노출시켜야 합니다. 만약 여러개의 view를 위해 interface/callback이 여러개 필요하다면, 코드를 더욱 더 서술적으로 표현하여 이를 유지보수하는 분들이 감사하도록 해주세요.

```kotlin
//Which is more descriptive
//Which one shows you all the possible interactions at a first glance?

class RecyclerViewAdapter(
    private val onAddClick: (itemAtIndex: Data) -> Unit,
    private val onRemoveClick: (itemAtIndex: Data) -> Unit,
    private val onItemClick: (itemAtIndex: Data) -> Unit
)

class RecyclerViewAdapter(
    private val onItemViewClick: (clickedViewId: Int, itemAtIndex: Data) -> Unit
)
```

## 황금기

세 번째 anti-parttern은 `ViewHolder` 내부의 view 상태를 직접 변경하는 것입니다. 예를 들어, `CheckBox`의 상태를 변경하는 `ViewHolder`를 살펴봅시다.

```kotlin
override fun onBindViewHolder(holder: MyViewHolder, position: Int) {
    //Note: checkbox clickable is set to false to control the logic ourselves
    holder.itemView.setOnClickListener {
        //Toggle
        holder.checkBox.isChecked = holder.checkBox.isChecked.not()
    }
}
```

이렇게 작성하고 100개의 아이템을 밀어 넣은 후, 처음 두 세개의 아이템을 check한 뒤 아래로 스크롤 하면, 다른 포지션에 있는 아이템들은 check하지 않았음에도 check되어있는 것을 확인할 수 있습니다. 이는 다시한 번 말하지만 view가 재사용되기 때문에 일어나는 현상입니다. check된 view가 재사용될 때, check된 채로 나타납니다. 따라서 우리는 항상 `onBindViewHolder`에서 다시 binding을 해줘야만해요. 그리고 우리는 data class의 `isChecked` 값의 기본값을 `false`로 설정함으로써 문제를 해결할 수 있습니다.

```kotlin
data class Data(
    val text: String,
    val isChecked: Boolean = false
)
```

`onBind` 메서드에서 값을 다시 바인딩 해줍니다.이

```kotlin
override fun onBindViewHolder(holder: MyViewHolder, position: Int) {
    holder.checkBox.isChecked = itemList[position].isChecked
    holder.itemView.setOnClickListener {
        holder.checkBox.isChecked = holder.checkBox.isChecked.not()
    }
}
```

이제 다시 테스트해봅시다. 데이터를 밀어넣고, 몇 개를 check하고, 스크롤을 내리면 제대로 동작 합니다. 그러나 스크롤을 다시 올리면, check된 모든 아이템들의 상태가 다 날라가버립니다! 이는 data class 내부의 `isChecked` 상태가 변경된게 아니기 때문입니다. 여전히 기본값인 `false`인채로 남아있어요. `RecyclerView`를 스크롤 할 때, 데이터를 다시 재활용된 view에 바인딩 하기위하여 `onBind` 메서드가 실행됩니다. 위 코드에서는, check된 상태가 전부 `false`로 대체되네요. 이를 해결하기 위해 adapter 코드를 아래와 같이 수정해봅시다.

```kotlin
override fun onBindViewHolder(holder: MyViewHolder, position: Int) {
    val itemAtPosition = itemList[position]
    holder.checkBox.isChecked = itemAtPosition.isChecked

    holder.itemView.setOnClickListener {
        itemList[position] = itemAtPosition.copy(
            isChecked = itemAtPosition.isChecked.not()
        )
        notifyItemChanged(position)
    }

}
```

이렇게 하면 재활용된 상태 문제를 해결할 수 있습니다. 이제 우리는 앱에 database를 추가하고 사용자가 "저장"버튼을 클릭할 때, database에 어떤 아이템이 check되었는지를 영구 저장할 수 있습니다. 따라서 우리는 `itemList`를 public 변수로 만들어 다른 클래스가 접근할 수 있게 해줄 수 있습니다. 예를 들어 "저장" 버튼이 눌리면 fragment가 `saveToDb(adapter.itemList)`를 호출하는 식으로요. 그리고 사용자는 한 번에 모두 선택하기와 모두 해제하기 같은 기능을 원할 수 도 있습니다. 따라서 함수 두 개를 추가해볼게요: adapter 내부의 `unSelectAll`과 selectAll

```kotlin
fun unselectAll() {
    itemList.map {  data->
        data.copy(isChecked = false)
    }
    notifyDataSetChanged()
}
fun selectAll() {
    itemList.map { data ->
        data.copy(isChecked = true)
    }
    notifyDataSetChanged()
}
```

오직 상태가 변경된 경우에만 notify함으로써 로직을 개선할 수 있습니다.

```kotlin
fun unselectAll() {
    itemList.mapIndexed { position, data ->
        if (data.isChecked) {
            notifyItemChanged(position)
            data.copy(isChecked = false)
        }
    }
}
fun selectAll() {
    itemList.mapIndexed { position, data ->
        if (!data.isChecked) {
            notifyItemChanged(position)
            data.copy(isChecked = true)
        }
    }
}
```

만약 지금 상태에서 처음 adapter코드를 본다면, 아마도 너무 많은 것들이 포함되어있다고 생각하실 겁니다. 만약 양쪽으로 가는 pagination 기능이 추가되거나, 나중에 아이템 삭제/숨김 기능 혹은 그 이상 다른 기능들이 추가되면 어떨까요? adapter가 하는 일은 단지 `ViewHolder`를 바인딩 할 뿐 데이터의 상태를 제어해선 안됩니다. 우리의 `bindView`는 현재 로직을 가지고 있어요! 우리는 adapter를 가능한 추상적으로 만들어야 함을 잊어선 안됩니다. 추상화시키는 한 가지 방법은 선택, 선택 해제, 추가, 삭제 및 기타 여러 로직들을 presenter나 ViewModel로 떠넘기는 것입니다. 그렇게 해서 adapter는 오직 사용자가 아이템을 업데이트하고 싶을 때마다 넘겨주는 item list를 받기만 하면 됩니다. 리팩토링을 한 이후 우리의 adapter는 아래와 같습니다. 재사용이 가능해졌고, 어떤 로직도 가지고 있지 않으며, itemList는 불변 List로 설정되어 내부에서 변경이 불가능하게 만들어졌습니다. 이제 상태를 변경하는 방법은 오직 새로운 list(상태가 변경된)를 받는 것 뿐입니다.

```kotlin
class RecyclerViewAdapter(
    val onCheckToggled: (position: Int, itemAtPosition: Data) -> Unit
) : RecyclerView.Adapter<RecyclerViewAdapter.MyViewHolder>() {

    //..Other overrides
    private var itemList: List<Data> = listOf<Data>()

    fun submitList(itemList: List<Data>) {
        this.itemList = itemList
        notifyDataSetChanged()
    }

    inner class MyViewHolder(
        itemView: View,
        onItemClick: (position: Int) -> Unit
    ) : RecyclerView.ViewHolder(itemView) {

        val checkBox: CheckBox = itemView.findViewById(R.id.checkBox)

        init {
            checkBox.setOnClickListener {
                onItemClick(adapterPosition)
            }
        }
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): MyViewHolder {
        val itemView = LayoutInflater.from(parent.context).inflate(R.layout.item, parent, false)
        return MyViewHolder(
            itemView = itemView,
            onItemClick = { position ->
                val itemAtPosition = itemList[position]
                this.onCheckToggled(position, itemAtPosition)
            }
        )
    }

    override fun onBindViewHolder(holder: MyViewHolder, position: Int) {
        val itemAtPosition = itemList[position]
        holder.checkBox.isChecked = itemAtPosition.isChecked
    }

}
```

하지만 현재 adapter는 item 변경에 대해 충분히 notify하고 있지 않습니다. 위치가 변경되었을 때 마다 매번 전체 list를 다시 바인딩하고 싶지 않아요. 만약 우리가 추가, 삭제, 변경을 모두 구분할 수 있으면 괜찮지 않을까요?

```kotlin
fun submitList(newList: List<Data>) {
    val oldList = this.itemList

    val maxSize = Math.max(newList.size, oldList.size)
    for (index in 0..maxSize) {
        val newData = newList.getOrNull(index)
        val oldData = oldList.getOrNull(index)

        if (newData == null) {
            notifyItemRemoved(index)
            return
        }

        if (oldData == null) {
            notifyItemInserted(index)
            return
        }

        if (newData != oldData) {
            notifyItemChanged(index)
            return
        }
    }
}
```

위 코드는 아주 간단한 diffing(구별을 위한) 메서드 입니다. 실제로는 새로운 아이템이 기존 리스트 사이에 들어와서, 기존의 아이템이 단지 위치만 바뀐 경우도 고려해야합니다. 그리고 이 코드는 main thread에서 diffing(구별)하고 있기 때문에 비효율적입니다. 100개의 아이템이 있다면, main thread에서 100번의 반복문을 수행한다는 의미니까요.

이런 boilerplate와 diffing을 간단히 하기 위해서 `ListAdatper`가 등장했습니다. `ListAdapter`는 ListView를 위한 adapter는 아니구요, 오히려 `RecyclerView.Adapter`를 확장한 adatper입니다. `ListAdatper` 생성자는 `DiffUtil.ItemCallBack` 또는 `AsyncDifferConfig`를 받습니다. 대부분은 DiffUtil.ItemCallBack만으로 충분합니다.

```kotlin
class RecyclerViewAdapter(
    val onCheckToggled: (position: Int, itemAtPosition: Data) -> Unit
) : ListAdapter<Data, RecyclerViewAdapter.MyViewHolder>(
     object: DiffUtil.ItemCallback<Data>() {

        override fun areItemsTheSame(oldItem: Data, newItem: Data): Boolean {
            return oldItem.id == newItem.id
        }

        override fun areContentsTheSame(oldItem: Data, newItem: Data): Boolean {
            return oldItem == newItem
        }

    }
)
```

`areItemsTheSame` 이 true를 반환하면 areContentsTheSame을 수행합니다. 만약 item이 서로 같고, 내용물(content)도 같다면, 이는 변화가 없다는 것을 의미하죠. 한편 item이 서로 같지만, 내용물이 다르다면, 이는 변경으로 간주합니다. 그러나 item이 서로 다를 경우, 이는 아이템이 추가되었거나, 삭제되었거나, 위치가 변경되었을 경우입니다. 이게 모든 diffing 메서드를 수면 아래로 숨겨주는 것이죠. 추가적으로, 이 모든 작업이 백그라운드 thread에서 수행됩니다. 그저 새로운 list를 `submitList` 함수로 넘겨주면 boilerplate 로직을 다 수행해줍니다. 혹시 관심있으시다면 [AsyncListDiffer](https://cs.android.com/androidx/platform/frameworks/support/+/androidx-main:recyclerview/recyclerview/src/main/java/androidx/recyclerview/widget/AsyncListDiffer.java;l=115?q=AsyncListDiffer) 내부 로직을 확인하실 수 있어요. 대부분의 경우, state를 ViewModel이나 Presenter에 유지하는게 좋기 때문에, 이를 강제하는 ListAdapter를 사용하는게 좋습니다. 게다가 라이브러리가 당신을 위해 대부분의 일을 해주니까 굉장히 편합니다.

---

어떻게 Adapter의 API가 `ArrayAdatper`로 시작하여 `ListAdapter`로 진화되어왔는지 이해하셨기를 바랍니다. 아직 [ListAdapter](https://developer.android.com/reference/androidx/recyclerview/widget/ListAdapter)를 사용해 보시지 않으셨다면 꼭 써보시길 추천드리구요, 이때까지 얼마나 많은 작업들을 손수 해왔는지, 이를 사용함으로써 코드 퀄리티가 얼마나 향상되는지를 느껴보시기 바랍니다.
