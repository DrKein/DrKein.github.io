---
layout: post
title: Using Kotlin takeIf(or takeUnless)(번역)
img: i-rest.jpg # Add image post (optional)
tags: [Kotlin, takeif, takeUnless]
---

원문 : https://medium.com/@elye.project/using-kotlin-takeif-or-takeunless-c9eeb7099c22


# Using Kotlin takeIf(or takeUnless)

![img](https://cdn-images-1.medium.com/max/1600/1*HyevTu9l1QUBcWJ6Vx9ThQ.png)

In Kotlin’s [standard functions](https://github.com/JetBrains/kotlin/blob/master/libraries/stdlib/src/kotlin/util/Standard.kt), there’s two function i.e. `takeIf` and `takeUnless`, that at first glance, what’s so special about it? Is it pretty much just `if`?

코틀린 기본 함수에는 얼핏보면, 특별한게 뭐지? 그냥 `if`같은거 아냐? 라는 생각이 드는 함수 `takeIf`와 `takeUnless`두가지 함수가 있습니다.


Or one could go the extreme, replace every `if` it sees as below (**NOT **recommended).

누군가는 아래 예제처럼 극단적으로 모든 `if`를 교체하기도 합니다. (추천하지 **않습니다**)

```kotlin
// Original Code
if (status) { doThis() }
// Modified Code
takeIf { status }?.apply { doThis() }
```



##Under the hood 

Like any other thing, `takeIf` (or `takeUnless`) do have it’s place of use. I share my view of them in the various scenario. Before we proceed, let’s look at the implementation of it.

다른 것들과 마찬가지로, `takeIf`(또는 `takeUnless`) 그걸 사용할 곳이 있습니다. 저의 관점을 다양한 시나리오로 공유하겠습니다. 그 전에, 이것의 구현체를 살펴 보겠습니다.

### The implementation signature

```kotlin
public inline fun <T> T.takeIf(predicate: (T) -> Boolean): T?
  = if (predicate(this)) this else null
```

From it, we notice that

1. It is called from the T object itself. i.e. `T.takeIf`.
2. The `predicate` function takes T object as parameter.
3. It returns `this` or `null` pending on the `predicate` evaluation.

시그니처를 통해 아래 내용을 알 수 있습니다.

1. T 오브젝트 자신으로 부터 호출됨. 즉 `T.takeIf`.
2. `predicate`함수가 파라미터로 T 오브젝트를 받습니다.
3. `predicate` 판단 결과에 따라, `this`또는 `null`을 리턴 합니다.



## Appropriate use

Based on the characteristics above, I could derive it’s usage as oppose to `if`condition in the below scenarios.

위에 설명한 특징을 바탕으로, 아래 시나리오의 `if` 에 대항하는 사용법을 유추할 수 있습니다.

### 1. It is called from the T object itself. i.e. T.takeIf.

### 1. T오브젝트 자신에서 호출 될때 즉, T.takeIf.

The benefit of handling cases with nullability check. An example as below

아래 예제 처럼 널 가능성을 체크할 때 

```kotlin
// Original code
if (someObject != null && status) {
   doThis()
}
// Improved code
someObject?.takeIf{ status }?.apply{ doThis() }
```



### 2. The predicate function takes T object as parameter

### 2. 서술 함수가 파라미터로 T 오브젝트를 받을 때 

Given this takes T as parameter to the predicate, one could further simply the code with `takeIf` as below

서술자의 파라미터로 T를 사용할 때, 아래처럼 더 단순한 코드를 사용할 수 있습니다.

```kotlin
// Original code
if (someObject != null && someObject.status) {
   doThis()
}
// Better code
if (someObject?.status == true) {
   doThis()
}
// Improved code
someObject?.takeIf{ it.status }?.apply{ doThis() }
```

The `better code` helps, but requires additional explicit eyesore `true`keyword in the evaluation. So it’s not ideal.

`better code`도 도움이 되지만, 수식 평가에 눈에 거슬리는 추가적인 `true` 키워드를 사용해야 합니다. 이상적이지 않죠.



### 3. It returns this or null pending on the predicate evaluation.

### 3. 서술식에 따라 this 또는 null을 리턴 합니다.

Since it is returning `this` if it is true, it could be used for chaining the operation. Hence something as below would be improved.

판단식이 참 일 때 `this` 를 리턴하므로, 오퍼레이션 체이닝에 사용할 수 있습니다. 따라서 아래처럼 개선됩니다.

```kotlin
// Original code
if (someObject != null && someObject.status) {
   someObject.doThis()
}
// Improved code
someObject?.takeIf{ status }?.doThis()
```

Or a better way of getting some data or quit (example taken from [Kotlin Doc](https://kotlinlang.org/docs/reference/whatsnew11.html#also-takeif-and-takeunless))

또는, 어떤 데이타를 얻거나 종료하는 좋은 방법 입니다.([Kotlin Doc](https://kotlinlang.org/docs/reference/whatsnew11.html#also-takeif-and-takeunless) 의 예제)

```kotlin
val index 
   = input.indexOf(keyword).takeIf { it >= 0 } ?: error("Error")
val outFile 
   = File(outputDir.path).takeIf { it.exists() } ?: return false
```



## Be cautious

One word of cautious though. Check out the below code.

조심할 점이 하나 있습니다. 아래 코드를 보세요.

```kotlin
// Syntactically still correct. But logically wrong!!
someObject?.takeIf{ status }.apply{ doThis() }
// The correct one (notice the nullability check ?)
someObject?.takeIf{ status }?.apply{ doThis() }
```

The first line will just `doThis()` regardless of if `status` is true of false. The reason is, even when `takeIf` returns `null`, it is still being called. (This is with assumption `doThis()` is not the function of `someObject`)

첫번째 줄은 `status` 가 참이든 거짓이든 상관 없이 `doThis()` 를 실행합니다. 이유는 `takeIf`가 `null` 을 리턴해도 호출되기 때문 입니다. (이것은 `doThis()`가 `someObject`의 함수가 아닌 경우를 가정합니다.)

The `?` check here is very subtle and most important

여기서 `?`는 아주 미묘하지만 매우 중요합니다.



Hopefully the provides some reference how `takeIf` (or `takeUnless`) could be better used. Feel free to provide some good real example of how you use these functions as response to this blog. I would love to hear from you. This may benefits others.

Also, for other functions in [standard functions](https://github.com/JetBrains/kotlin/blob/master/libraries/stdlib/src/kotlin/util/Standard.kt), you could refers to [my other blog](https://android.jlelse.eu/mastering-kotlin-standard-functions-run-with-let-also-and-apply-9cd334b0ef84).



I hope you appreciate this post and it’s helpful for you. Do share with others.

You could check out my other interesting topics [here](https://medium.com/@elye.project/).

Follow me on [*medium*](https://medium.com/@elye.project)*,* [*Twitter*](https://twitter.com/elye_project) *or* [*Facebook*](https://www.facebook.com/elyeproj/) for little tips and learning on Android, Kotlin etc related topics. ~Elye~

