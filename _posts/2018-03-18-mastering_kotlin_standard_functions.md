---
layout: post
title: "Mastering Kotlin standard functions: run, with, let, also and apply(번역)"
---

원문 : <https://medium.com/@elye.project/mastering-kotlin-standard-functions-run-with-let-also-and-apply-9cd334b0ef84>



# Mastering Kotlin standard functions: run, with, let, also and apply

![img](https://cdn-images-1.medium.com/max/800/1*9nUzj5iRxj_Hddni6ob28w.png)



Kotlin의 몇몇 [표준함수](https://github.com/JetBrains/kotlin/blob/master/libraries/stdlib/src/kotlin/util/Standard.kt)들은 너무 비슷해서 어떤것을 써야 하는지 잘 모르는 경우가 많습니다. 여기서 저는 그것들의 명확한 구분법과 어느 경우에 어떤 것을 써야 하는지 소개 하겠습니다.



## Scoping functions

중점을 두려는 함수들은 *run, with, T.run, T.let, T.also* and *T.apply* 입니다. 이 함수들의 주요 기능은 호출자 함수의 내부 범위를 제공하는 부분을 보고 저는 이 함수들을 스코핑 함수 라고 부릅니다. 

스코핑을 표현하는 가능 간단한 방법은 *run* 함수 입니다. 



```kotlin
fun test() {
    var mood = "I am sad"
    
    run {
        val mood = "I am happy"
        println(mood) // I am happy
    }
    println(mood) // I am sad
}
```

이 예제를 보면 `test`함수 안에 분리된 스코프를 가질 수 있습니다. 출력문 앞에 `I am happy`라는 값으로 재 정의된 `mood` 변수를 정의하고 `run` 스코프로 완전히 닫혀있습니다.



이 스코핑 함수 자체는 크게 유용해 보이지 않습니다. 하지만 단지 스코핑이외에 다른 좋은 점이 있는데, 뭔가를 리턴하는 것 입니다. 즉, 스코프 안의 마지막 오브젝트를 리턴합니다.



그래서 아래 예제처럼 뷰를 두번 호출하지 않고 깔끔하게 `show()`를 두가지 뷰에 모두 적용할 수 있습니다.

```kotlin
run {
    if (firstTimeView) introView else normalView
}.show()
```



## 3 attributes of scoping functions

스코핑 함수를 흥미롭게 만들려면 3가지 속성을 사용하여 동작들을 분류 하세요. 저는 이 속성들을 서로 서로 구분하기 위해 사용할 것입니다.



### 1. Normal vs. extension function

`with`와 `T.run`을 보면, 두 함수는 실제로 많이 비슷합니다. 아래 예제는 동일한 동작을 합니다.

```kotlin
with (webview.settings) {
    javaScriptEnabled = true
    databaseEnabled = true
}

// similarly

webview.settings.run {
    javaScriptEnabled = true
    databaseEnabled = true
}
```

하지만, 한가지 차이점은 하나는 일반 함수 즉 `with`이고 다른 하나는 확장 함수 즉 `T.run` 이라는 점입니다.



그렇다면 각각의 이점은 무엇 일까요?



만약 `web view.settings`가 널이 될 수 있다고 상상해 보면, 아래 모양처럼 될것입니다.

```kotlin
// Yack!
with (webview.settings) {
    this?.javaScriptEnabled = true
    this?.databaseEnabled = true
}

// Nice.
webview.settings?.run {
    javaScriptEnabled = true
    databaseEnabled = true
}
```

이 경우에는 그것을 사용하기 전에 널 가능성 체크를 적용할 수 있는  `T.run` 확장 함수가  확실히 더 좋습니다.



### 2. This vs. it argument

`T.run`과 `T.let`을 보면 두 함수는 매개변수를 받아들이는 방식 한가지를 빼고 비슷 합니다. 아래 예제는 두 함수를 이용한 동일한 로직을 보여 줍니다.

```kotlin
stringVariable?.run {
    println("The length of this String is $length")
}

// Similarly.

stringVariable?.let {
    println("The length of this String is ${it.length}")
}
```

`T.run`함수 시그니처를 확인해 보면 `T.run`은 단지 `block: T.()`확장 함수로 만들어 진것을 알 수 있습니다. 그래서 모든 스코프 내에서 `T`는 `this`로 참조할 수 있습니다. 프로그래밍에서 거의 모든 경우 `this`는 생략할 수 있습니다. 그래서 위의 예제에서 우리는 `println` 구문 내에서, `${this.length}` 대신  `$length`를 사용할 수 있습니다. 저는 이것을 ***이것을 매개변수로(this as argument)*** 보낸다 고 부릅니다. 

반면에 `T.let` 함수 시그니처를 보면 `T.let`은 자기 자신을 함수로 전송 합니다. 즉 `block: (T)` 로 되어 있습니다. 그래서 람다식에서 매개변수를 보내는 것과 비슷합니다. 그것은 스코프 함수 내에서 `it`로 참조될 수 있습니다. 그래서 저는 이것을 ***그것을 매개변수로(it as argument)*** 보낸다고 부릅니다.

위 내용을 보면 의미 함축을 할 수 있는 `T.run`이 `T.let`보다 우위인것 처럼 보이지만, 아래처럼 `T.let` 함수가 갖는 이점이 있습니다.

- `T.let`은 외부 클래스의 함수/멤버 와 내부 함수/멤버를 명확히 구분하여 사용할 수 있습니다.
- 만약 함수의 파라미터로 그것이 보내지는 것처럼 `this`가 생략될 수 없는 상황에서 `it`은 `this`보다 짧게 사용할 수 있고 더 명확합니다.
- `T.let`은 더 좋은 이름으로 변경을 허용 합니다. 즉, `it`을 다른 이름으로 바꿀수 있습니다.

```kotlin
stringBariable?.let {
    nonNullString ->
    println("The non null string is $nonNullString")
}
```



### 3. Return this vs. other type

이제 `T.let`과 `T.also`를 보면 내부 함수 스코프를 보면 이 둘은 동일 합니다.

```kotlin
stringVariable?.let {
    println("The length of this String is ${it.length}")
}

// Exactly the same as below

stringVariable?.also {
    println("The length of this String is ${it.length}")
}
```

하지만 이 둘의 미묘한 차이는 어떤 것을 리턴하는가 입니다. `T.let`은 다른 타입의 값을 리턴하고, `T.also`는 `T` 자체, 즉`this`를 리턴 합니다. 

위 둘 다 함수 체이닝에 유용합니다. `T.let`은 오퍼레이션을 변화시킬수 있고, `T.also`는 동일 변수 즉 `this`를 수행할 수 있게 합니다. 

아래 간단한 예제가 있습니다.

```kotlin
val original = "abc"

// Evolve the value and send to the next chain
original.let {
    println("The oroginal String is $it") // "abc"
    it.reversed() // evolve it as parameter to send to next let
}.let {
    println("The reverse String is $it") // "cba"
    it.length  // can be evolve to other type
}.let {
    println("The length of the String is $it")  // 3
}

// Wrong
// Same value is sent in the chain (printed answer is wrong)
original.also {
    println("The original String is $it")  // "abc"
    it.reversed() // even if we evolve it, it is useless
}.also {
    println("The reverse String is ${it}")  // "abc"
    it.length. // even if we evolve it, it is useless
}.also {
    println("The length of the String is ${it}")  // "abc"
}

// Corrected for also (i.e. manipulate as original string)
// Same value is sent in the chain
original.also {
    println("The original String is $it")  // "abc"
}.also {
    println("The reverse String is ${it.reversed()}")  // "cba"
}.also {
    println("The length of the String is ${it.length}") // 3
}
```

위에서 `T.also`는 의미 없는 것처럼 보이지만, 우리는 그것들을 함수의 단일 블럭으로 쉽게 합칠 수 있습니다. 잘 생각해 보시면, 좋은 점이 있습니다.



1. 동일 객체에 대해서 매우 분명한 분리를 제공합니다. 즉, 함수영역을 작게 만들어 줍니다.
2. 그것은 체이닝 빌더 연산으로, 사용되기 전에 자기 변경에 매우 강력합니다.



둘을 합쳐서 체인을 하면 즉, 자신을 변화시키거나 자신을 유지하는 기능은 아래처럼 강력합니다.

```kotlin
// Normal approach
fun makeDir(path: String): File {
    val result = File(path)
    result.mkdirs()
    return result
}

// Improved approach
fun makeDir(path: String) = path.let { File(it) }.also { it.mkdirs() }
```



### Looking at all attributes

3가지 속성을 통해 우리는 꽤 많은 함수 동작을 알게 되었습니다. 제가 소개하지 않은 `T.apply`함수를 살펴 봅시다. `T.apply`의 3가지 속성은 ...

1. 확장 함수 입니다.
2. 그것의 매개변수로 `this`를 보냅니다.
3. `this` 즉, 자기 자신을 리턴 합니다.



그러므로, 아래처럼 사용하는 것을 상상할 수 있습니다.

```kotlin
// Normal approach
fun createInstance(args: Bundle): MyFragment {
    val fragment = MyFragment()
    fragment.arguments = args
    return fragment
}

// Improved approach
fun createInstance(args: Bundle) = MyFragment().apply { arguments = args }
```



또는 체이닝 안된 객체를 체이닝 하게 만들 수도 있습니다.

```kotlin
// Normal approach
fun createIntent(intentData: String, intentAction: String): Intent {
    val intent = Intent()
    intent.action = intentAction
    intent.data = Uri.parse(intentData)
    return intent
}

// Improved approach, chaining
fun createIntent(intentData: String, intentAction: String) = 
	Intent().apply { action = intentAction }
			.apply { data = Uri.parse(intentData) }
```



### Function selections

이제 우리는 3가지 속성에 따라 함수들을 명확히 구분할 수 있습니다. 그리고 그 기반으로 우리가 원하는 것에 따라 어느 함수를 사용할 것인지 결정하는 트리를 만들수 있습니다.

![](https://cdn-images-1.medium.com/max/800/1*pLNnrvgvmG6Mdi0Yw3mdPQ.png)



이 결정 트리가 함수들의 명확하게, 여러분의 결정을 쉽게만들어 주고, 여러분이 적절한 함수 사용을 마스터 하는데 도움이 되길 바랍니다. 



이 블로그에 대한 응답으로 여러분이 이 함수들을 사용하는 좋은 실례를 제공해 주십시오. 여러분의 응답을 기다립니다. 이것은 다른 사람들에게 도움이 됩니다.



이 블로그가 여러분에게 가치가 있고 도움이 되길 바랍니다. 다른 사람들과 공유 하세요.

제 다른 재미있는 토픽들은 [여기](https://medium.com/@elye.project/) 에서 보실 수 있습니다.



Follow me on [*medium*](https://medium.com/@elye.project)*,* [*Twitter*](https://twitter.com/elye_project) or [*Facebook*](https://www.facebook.com/elyeproj/) for little tips and learning on Android, Kotlin etc related topics. ~Elye~

















