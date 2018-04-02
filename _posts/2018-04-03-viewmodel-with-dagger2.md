---
layout: post
title: ViewModel with Dagger2 (Android Architecture Components) (번역)
description: ViewModel과 Dagger2
categories: [Architecture]
tags: [ViewModel, Dagger2]
comments: true
---

원문 : <https://proandroiddev.com/viewmodel-with-dagger2-architecture-components-2e06f06c9455>

# ViewModel with Dagger2 (Android Architecture Components)

![](https://cdn-images-1.medium.com/max/1600/1*jm9hqqh7206xwQKDImb7aw.png)

여러분 안녕하세요! 👋  이 글에서 저는 ViewModel(안드로이드 아키텍처 컴포넌트) 와 Dagger2 인젝션을 사용하는 팁을 공유하려고 합니다. 

만약 ViewModel과 LiveData에 친숙하지 않다면, 제 이전 글 : https://proandroiddev.com/the-death-of-presenters-and-the-rise-of-viewmodels-aac-f14d54b419a 을 읽어보실것을 강력히 추천 합니다.

시작하기 전에, 만약 당신이 프리젠터를 ViewModel과 LiveData로 교체하려고 한다면, 제가 교체 작업했던 내용을 확인해 보세요. https://github.com/sanogueralorenzo/Android-Kotlin-Clean-Architecture/pull/9

리팩토링을 할 때는 작업량이 얼마나 큰지, 그걸 통해 얻는 이득이 무엇인지를 항상 평가하는 것이 중요 합니다. 다른 해결책으로 다음과 같은 몇가지 일반적인 규칙을 세우는 것입니다:

> 지금부터 ViewModel과 LiveData를 사용하기 시작한다.  
> 프리젠터에 주요 변경 사항이 일어날 때 ViewModel로 변경 한다.

어쨋든, Dagger2 이슈로 돌아와서 코드로 뛰어 듭시다.

제 샘플 프로젝트를 예제로 사용하겠습니다:  
https://github.com/sanogueralorenzo/Android-Kotlin-Clean-Architecture


## 맵퍼 사용시 @Inject 와 ViewModel 을 사용합시다 
```kotlin
class PostListViewModel @Inject constructor(private val useCase: UsersPostsUseCase,
                                            private val mapper: PostItemMapper) : ViewModel() {

    val posts = MutableLiveData<Data<List<PostItem>>>()
    private val compositeDisposable = CompositeDisposable()

    init {
        get()
    }

    fun get(refresh: Boolean = false) =
            compositeDisposable.add(useCase.get(refresh)
                    .doOnSubscribe { posts.postValue(Data(dataState = DataState.LOADING, data = posts.value?.data, message = null)) }
                    .subscribeOn(Schedulers.io())
                    .observeOn(Schedulers.io())
                    .map { mapper.mapToPresentation(it) }
                    .subscribe({
                        posts.postValue(Data(dataState = DataState.SUCCESS, data = it, message = null))
                    }, { posts.postValue(Data(dataState = DataState.ERROR, data = posts.value?.data, message = it.message)) }))

    override fun onCleared() {
        compositeDisposable.dispose()
        super.onCleared()
    }
}
```

우리 ViewModel에 @Inject 를 추가해도 박스 밖에서 동작하지 않습니다😢. 그럼, 우리의 ViewModel이 의존성을 알게 하려면 어떻게 하면 될까요? 

## 우리만의 ViewModelFactory만들기 😄
```kotlin
@Singleton
class ViewModelFactory @Inject constructor(private val viewModels: MutableMap<Class<out ViewModel>, Provider<ViewModel>>) : ViewModelProvider.Factory {

    override fun <T : ViewModel> create(modelClass: Class<T>): T = viewModels[modelClass]?.get() as T
}

@Target(AnnotationTarget.FUNCTION, AnnotationTarget.PROPERTY_GETTER, AnnotationTarget.PROPERTY_SETTER)
@kotlin.annotation.Retention(AnnotationRetention.RUNTIME)
@MapKey
internal annotation class ViewModelKey(val value: KClass<out ViewModel>)

@Module
abstract class ViewModelModule {

    @Binds
    internal abstract fun bindViewModelFactory(factory: ViewModelFactory): ViewModelProvider.Factory

    @Binds
    @IntoMap
    @ViewModelKey(PostListViewModel::class)
    internal abstract fun postListViewModel(viewModel: PostListViewModel): ViewModel

    //Add more ViewModels here
}
```

위 코드와 이 ViewModelModule을 우리 Injector 클래스에 추가하는 것이 우리가 해야할 전부 입니다! (뭐든 새로운 ViewModel을 ViewModelModule에 추가하는 것을 기억하세요 😅)


## 마지막으로, 우리 액티비티가 ViewModelProvider.Factory를 주입받고 그것이 ViewModelProviders.of() 메서드에 팩토리로 (두번째 파라미터) 전달합니다.
```kotlin
class PostListActivity : AppCompatActivity() {

    @Inject
    lateinit var viewModelFactory: ViewModelProvider.Factory
    @Inject
    lateinit var postListNavigator: PostListNavigator
    @Inject
    lateinit var userDetailsNavigator: UserDetailsNavigator

    private val avatarClick: (String) -> Unit = { userDetailsNavigator.navigate(this, it) }
    private val itemClick: (PostItem) -> Unit = { postListNavigator.navigate(this, it) }
    private val adapter = PostListAdapter(avatarClick, itemClick)

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_post_list)
        getAppInjector().inject(this)

        postsRecyclerView.adapter = adapter
        swipeRefreshLayout.setColorSchemeColors(ContextCompat.getColor(this, R.color.colorAccent))

        val vm = ViewModelProviders.of(this, viewModelFactory)[PostListViewModel::class.java]
        vm.posts.observe(this, Observer(::updatePosts))
        swipeRefreshLayout.setOnRefreshListener { vm.get(refresh = true) }
    }

    private fun updatePosts(data: Data<List<PostItem>>?) {
        data?.let {
            when (it.dataState) {
                DataState.LOADING -> swipeRefreshLayout.startRefreshing()
                DataState.SUCCESS -> swipeRefreshLayout.stopRefreshing()
                DataState.ERROR -> swipeRefreshLayout.stopRefreshing()
            }
            it.data?.let { adapter.addItems(it) }
            it.message?.let { toast(it) }
        }
    }
}
```

끝입니다! @Inject로 만들어진 우리의 ViewModel이 이제 동작 합니다. 💃

추가로, [Antonio Leiva](https://medium.com/@antoniolg) 한테 배운 정말 멋진것을 알려 드리겠습니다.
```kotlin
//Standard way 
val vm = ViewModelProviders.of(this, viewModelFactory)[PostListViewModel::class.java]
vm.posts.observe(this, Observer(::updatePosts))
swipeRefreshLayout.setOnRefreshListener { vm.get(refresh = true) }

//Antonio's Leiva way
withViewModel<PostListViewModel>(viewModelFactory) {
    observe(posts, ::updatePosts)
    swipeRefreshLayout.setOnRefreshListener { get(refresh = true) }
}
```

보시다시피, `withViewModel` 이 더 🆒 한 방법입니다. 당신에게 필요한 단지 3개 메서드는 다음과 같습니다 :
```kotlin
inline fun <reified T : ViewModel> FragmentActivity.getViewModel(viewModelFactory: ViewModelProvider.Factory): T {
    return ViewModelProviders.of(this, viewModelFactory)[T::class.java]
}

inline fun <reified T : ViewModel> FragmentActivity.withViewModel(viewModelFactory: ViewModelProvider.Factory, body: T.() -> Unit): T {
    val vm = getViewModel<T>(viewModelFactory)
    vm.body()
    return vm
}

fun <T : Any, L : LiveData<T>> LifecycleOwner.observe(liveData: L, body: (T?) -> Unit) {
    liveData.observe(this, Observer(body))
}
```

처음 2개는 viewModelFactory 가 추가되지만 이와 관련된 것과 더 많은 (dagger injection을 사용하지 않은 팩토리 같은) 예제들은 여기에 있습니다 : https://antonioleiva.com/architecture-components-kotlin/ 👏

당신이 아직 여기 있다면, 아마도 어떻게 ViewModel에 동적으로 파라미터를 @Inject 하는지 궁금해 하는 거겠죠? (예를 들면 새로운 액티비티에게 intent로 전달한 user id 같은)

당신이 할 수 있더라도, MVVM에서 ViewModel 은 dagger를 통해서라도 view로부터 어떤것도 얻어서는 안됩니다.

대신, 액티비티가 userId를 우리 ViewModel에게 설정할 수 있도록 간단한 해결책을 만들어 봅시다.
```kotlin
class UserDetailsViewModel @Inject constructor(private val useCase: UserUseCase,
                                               private val mapper: UserItemMapper) : ViewModel() {

    val user = MutableLiveData<UserItem>()

    var userId: String? = null
        set(value) {
            if (field == null) {
                field = value
                get()
            }
        }
    
    //method to fetch user
    fun get(refresh: Boolean = false) {}
}
```

여기에서 중요한 것은, userId 값은 한번만 설정되고 그 동작이 일어날 때 user 정보를 가져오는 get()메서드가 실행 된다는 점입니다. 이것은 스크린이 사라지거나 로테이션이 얼마든 되도 데이타를 다시 가져오는 것을 피하게 합니다. 

이 이슈가 왜 ViewModel 이 정보를 뷰로 부터 얻지 않아야 하는지 이해하는 흥미로운 점입니다.  
https://github.com/googlesamples/android-architecture-components/issues/207

## Remember to follow, share & hit the 👏 button if you’ve liked it! (:


Thanks to [Simon Percic](https://medium.com/@simonpercic?source=post_page).


---
