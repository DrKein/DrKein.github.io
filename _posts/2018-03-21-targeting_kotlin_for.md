---
layout: post
title: Targeting Kotlin for both Android and iOS(번역)
tags: [Kotlin, iOS, Android, Multiplatform, Kotlin Native, Konan]
---

원문 : <https://proandroiddev.com/targeting-kotlin-for-both-android-and-ios-dec5b967006a>


# Targeting Kotlin for both Android and iOS


![](https://cdn-images-1.medium.com/max/1600/1*a8vwAg-xkjY6wHhtyyXgUA.png)

Kotlin은 전형적으로 JVM기반 플랫폼을 위한 언어로 사용되어 왔고, 안드로이드 생태계에서 최고 인기를 갖게 되었습니다. 언어가 발전하면서 JVM 플랫폼 밖의 새로운 경계를 노크하고 있고, 그 중 하나가 iOS입니다.

Kotlin 은 Java와 동작할 수 있는 .class 파일로 컴파일되고, Android 를 위해 .dex로 변환 됩니다. 하지만 이제 [Kotlin Native](https://github.com/JetBrains/kotlin-native)를 사용하여 임베디드 시스템, 맥OS, iOS등 VM없이 바이트 코드를 직접 실행할수 있는 플랫폼을 타겟할 수 있습니다. 

예제를 보겠습니다.

{% gist 5c66cf39d10cc08766ce5673bc002914 %}


