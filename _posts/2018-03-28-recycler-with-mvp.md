---
layout: post
title: RecyclerView with MVP (번역)
description: RecyclerView 에 MVP 적용하기 
categories: [MVP]
tags: [Kotlin, Android, MVP]
comments: true
---

원문 : <http://bajicdusko.com/2017/recycler-view-in-MVP>

# RecyclerView with MVP

MVC/MVP/MVVM, 우리 모두 이것들에 대해 이야기 합니다. 최신에, 멋진데다 유용하기까지 하죠. We’ll see if the MVP is dead, now that [Google announced its own archtecture components](https://developer.android.com/topic/libraries/architecture/index.html), however, that’s something to deal with later.
현재 클린 아키텍쳐 기반 MVP는 우리를 위해 훌륭한 일을 하고 있습니다. 그리고 무엇보다 구현하고 이해하기 쉽습니다. 가끔은 구현이 혼란스러운 경우가 있는데 RecyclerView가 그렇습니다.

보통 `RecyclerView`를 구현할 때, 뷰홀더를 생성하고 거기에 데이타를 바인드 하기 위해 어떤 객체들의 리스트를 어댑터에 전달 했습니다. 
우리가 훌륭한 시민이라면 적어도 `ViewHolder` 로직을 `onBindViewHolder` 메서드 내에서 참조하는 대신 자체 클래스에 구현 해야 합니다. 

```kotlin
class TestAdapter(val context: Context) : RecyclerView.Adapter<TestViewHolder>(){

    private var testItemsList = mutableListOf<TestItem>()

    fun onDataChange(testItems: List<TestItem>){
      testItems.forEach {
        if(!testItemsList.contains(it)){
          testItemsList.add(it)
        }
      }

      notifyDataSetChanged()
    }

    override fun onCreateViewHolder(parent: ViewGroup?, viewType: Int): TestViewHolder =
      TestViewHolder(LayoutInflater.fromContext(context).inflate(R.layout.test_list_item, parent, false))

    override fun onBindViewHolder(holder: TestViewHolder?, position: Int) {
      var testItem = testItemsList[position]
      holder?.title.text = testItem.title
      holder?.description.text = testItem.description
    }

    override fun getItemCount = testItemsList.size()
}
```

그리고 이 어댑터를 사용하는 액티비티 또는 프래그먼트는 아래와 같은 모양입니다.

```kotlin
override fun onDataLoaded(testItems: List<TestItem>){
  var testAdapter = TestAdapter(context)
  recyclerView.adapter = testAdapter
  testAdapter.onDataChange(testItems)
}
```

매우 단순합니다. 제가 `TestViewHolder`의 구현을 보여주지 않았지만, 그것은 단순히 TextView들 초기화 밖에 하지 않습니다. 하지만, 우리가 위 어댑터에 대해서 유닛 테스트를 작성 하고자 한다면 곤경에 처할 것입니다.  
안드로이드 `Context`가 컨스트럭터에 있고 뷰 홀더를 생성할 때 `LayoutInflater`가 있어서, 리스트가 로딩되는 것을 체크하기 위해 모든것을 목킹해야 합니다.  

우리는 그렇게 하고 싶지 않기 때문에 `presenter`를 작성 합니다. 어댑터 프리젠터는 아래 기능을 제공 합니다 :

- 아이템 리스트 유지
- index 또는 position에 해당하는 아이템 제공
- 아이템 갯수 제공
- 변경 발생시 어댑터에게 알리기

```kotlin
class TestAdapterPresenter {

    var view: View? = null
    private var testItems = mutableListOf<TestItem>()

    fun onDataChange(testItems: List<TestItem>) {
        testItems.forEach {
          if(!testItemsList.contains(it)){
            testItemsList.add(it)
          }
        }
        view?.notifyAdapter()
    }

    fun getCount(): Int = testItems.size

    fun getItemAt(position: Int) = testItems[position]

    infix fun itemAtPosition(position: Int): TestItem = testItems[position]

    interface View {
        fun notifyAdapter()
    }
}
```

그리고 변경에 맞춰 어댑터를 변경 합니다.
``` kotlin
class TestAdapter(val context: Context) : RecyclerView.Adapter<TestViewHolder>(), TestAdapterPresenter.View{

    var testAdapterPresenter = TestAdapterPresenter()

    init {
        testAdapterPresenter.view = this
    }

    override fun notifyAdapter() {
        notifyDataSetChanged()
    }

    override fun onCreateViewHolder(parent: ViewGroup?, viewType: Int): TestViewHolder =
      TestViewHolder(LayoutInflater.fromContext(context).inflate(R.layout.test_list_item, parent, false))

    override fun onBindViewHolder(holder: TestViewHolder?, position: Int) {
      var testItem = testItemsList[position]
      holder?.title.text = testItem.title
      holder?.description.text = testItem.description
    }

    override fun getItemsCount() = testAdapterPresenter.getCount()
}
```

이 경우, 어댑터 초기화 부분은 거의 바뀌지 않았습니다. 대신 아이템 리스트를 어댑터에 전달하는 것을 프리젠터에 전달 합니다. 
``` kotlin
    override fun onDataLoaded(testItems: List<TestItem>){
      var testAdapter = TestAdapter(context)
      recyclerView.adapter = testAdapter
      testAdapter.testAdapterPresenter.onDataChange(testItems)
    }
```

우리는 `TestAdapter`에 구현된(implemented) `View` 인터페이스를 생성 했습니다. `TestAdapterPresenter`는 `notifyAdapter()`를 호출할 때 그 대상이 누구인지 알지 못합니다. 이것은 목킹을 쉽게 하고 데이타가 전달되었을 때 그 동작을 테스트하는데 매우 중요합니다.

이제 우리는 앱의 테스트 가능성을 증대시켰습니다. 하지만 더 좋아질 수 있습니다. 맞나요? `TestViewHolder`의 동작을 테스트 하려고 한다면, 우리의 손이 잘 안 움직이고 우리 자신을 돕기 위해 몇 가지 변화를 해야 합니다. MVP 패턴을 따라 `TestViewHolder`를 위한 프리젠터를 생성 합니다. 또한, 뷰 홀더에 연관된 `title`과 `description` 관련 호출들도 이동합니다.

```kotlin
class TestViewHolderPresenter {

  var testAdapterPresenter: TestAdapterPresenter? = null
  var position: Int? = null
  var view: View? = null

  fun bind(){
    if(position != null && testAdapterPresenter != null){
      var testItem = testAdapterPresenter.getItemAt(position)
      view?.setTitle(testItem.title)
      view?.setDescription(testItem.description)
    }
  }

  interface View {
    fun setTitle(title: String)
    fun setDescription(description: String)
  }
}
```
각 각의 뷰 홀더는 리스트의 어떤 위치와 연계되기 때문에, 뷰 홀더 프리젠터에 `position`을 설정하는 것은 매우 자연 스럽습니다. 또한 어댑터 프리젠터와 뷰 홀더 프리젠터 사이에 통신이 가능 하도록 하기 위해, 각각 뷰 홀더 프리젠터는 `testAdapterPresenter` 변수 처럼 어댑터 프리젠터의 참조를 가져야 합니다. 

아래에 `TestViewHolderPresenter.View`를 구현한 `TestViewHolder`와 `title`과 `description` 값을 설정하는 함수가 있습니다.

```kotlin
class TestViewHolder(val view: View) : RecyclerView.ViewHolder(view), TestViewHolderPresenter.View {
  var testViewHolderPresenter = TextViewHolderPresenter()

  @BindView(R.id.test_list_item_tv_title) lateinit var tvTitle: TextView
  @BindView(R.id.test_list_item_tv_description) lateinit var tvDescription: TextView

  init {
    ButterKnife.bind(this, view)
    testViewHolderPresenter.view = this
  }

  override fun setTitle(title: String) {
    tvTitle.text = title
  }

  override fun setDescription(description: String) {
    tvDescription.text = description
  }
}
```

이제 `TestViewHolder`는 확실히 데이타와 어댑터의 위치에 대해 전혀 알 수 없습니다. 뷰 홀더가 고민하는 유일한 일은 전달 받은 값들(`title`, `description`)을 컴포넌트에 설정하는 것 뿐입니다. 전체 로직은 프리젠터로 이동 했고 뷰 홀더에서는 그 로직을 제거했기 때문에 어댑터의 구현이 또 바뀌게 됩니다.

```kotlin
class TestAdapter(val context: Context) : RecyclerView.Adapter<TestViewHolder>(), TestAdapterPresenter.View {

  var testAdapterPresenter = TestAdapterPresenter()

  init {
    testAdapterPresenter.view = this
  }

  override fun notifyAdapter() {
    notifyDataSetChanged()
  }

  override fun onCreateViewHolder(parent: ViewGroup?, viewType: Int): TestViewHolder = 
    TestViewHolder(LayoutInflater.fromContext(context).inflate(R.layout.test_list_item, parent, false))

  override fun onBindViewHolder(holder: TextViewHolder?, position: Int) {
    holder.testViewHolderPresenter.position = position
    holder.testViewHolderPresenter.testAdapterPresenter = testAdapterPresenter
    holder.testViewHolderPresenter.bind()
  }  

  override fun getItemsCount() = testAdapterPresenter.getCount()
}
```

이렇게 변경되면, `TestAdapter` 역시 프리젠터와 뷰 홀더의 초기화만 고민하면 되고 프리젠터에 `position`을 전달하는 것만 신경쓰면 됩니다.

이해를 돕기 위해, 앱에 적용된 프리젠터들을 표현한 간단한 시퀀스 다이어그램이 있습니다.
![](/assets/img/recyclerview-with-mvp-seq-diagram.png)

위에 설명하거나 구현하지 않은 문제는 뷰 홀더에서 프리젠터를 통해 액티비티나 프래그먼트까지 연결되는 콜백 구현 입니다. Kaushik Gopal은 요즘 EventBus의 상태 [Episode 61 of Fragmented Podcast](http://fragmentedpodcast.com/episodes/061/)에서 이 문제를 언급 했습니다. 그는 EventBus가 이곳에서 그 사용성을 찾을수 있다고 결론지었습니다. 하지만 RecyclerView with MVP 파트2에서 EventBus없이 이것을 구현할 것입니다.

