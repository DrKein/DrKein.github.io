---
layout: post
title: Kotlin Syntax Part I - why am I excluded?(번역)
img: how-to-start.jpg # Add image post (optional)
tags: [Kotlin, syntax]
---



# Kotlin Syntax Part I - why am I excluded?(번역)

원문 : https://proandroiddev.com/kotlin-syntax-part-i-why-am-i-excluded-86772a61fade

![img](https://cdn-images-1.medium.com/max/2000/1*QcHZLfsT95RIkww_2Hoqyg.png)


Hi everyone again, welcome to article number on 2 of the Kotlin Playground Series. There wasn’t that much code in the previous article but if you read it, you probably saw some new syntax that you’re not familiar with if coming from Java.  
여러분 안녕하세요, Kotlin Playground 시리즈 두번째 글에 오신것을 환영 합니다. 지난번 글에 그렇게 많은 코드가 있진 않았지만, 그 글을 읽어 보셨다면, Java에서 보던 것과는 다른 새로운 문법들을 보셨을 겁니다.

In this and following article I’ll cover some of Kotlin base syntax as I believe it’s important before we jump into other examples. But enough talk and let’s look at some Kotlin code.  
다른 예제로 넘어가기 전에 이 글과 다음 글에서 제가 중요하다고 생각하는 몇 가지 Kotlin 기본 문법을 다루겠습니다. 

But enough talk and let’s look at some Kotlin code.  
말은 충분히 했으니, Kotlin 코드를 봅시다.

```kotlin
val immediateAssignment: Int = 1

val inferredTypeInt = 2

val deferredAssignment: Int
deferredAssignment = 3

var mutableInferredTypeString = "some string"
mutableInferredTypeString += "some concat"
```