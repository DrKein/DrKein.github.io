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

```kotlin
package io.github.jitinsharma

const val API_KEY = "abdfkdfgl453"
class Helper {
    fun getSum(first: Int, second: Int): Int = first + second
    fun sliceFilterAndSort(list: List<String>): List<String> = 
            list.subList(0, 4).filter { it.length > 3 }.sortedBy { it.length }

    companion object {
        val helperId: Int = 0
        fun getHelperType() : String = "Helper234"
    }
}

data class Model(
        var id: Int = 0,
        var type: String = ""
)
```

위 예제는 Kotlin으로 작성된 다수의 클래스와 변수를 가진 단순한 코드 조각 입니다. 주목해야 할 중요한 점은 *kotlin-stdlib* 이외 다른 라이브러리를 임포트 하지 않는 점 입니다.

그리고 아래는 이 코드를 jar로 컴파일 하기 위한 build.gradle 입니다.

```kotlin
group 'io.github.jitinsharma'
version '1.0-SNAPSHOT'

buildscript {
    ext.kotlin_version = '1.2.21'

    repositories {
        mavenCentral()
    }
    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
    }
}

apply plugin: 'java'
apply plugin: 'kotlin'

sourceCompatibility = 1.8

repositories {
    mavenCentral()
}

dependencies {
    compile "org.jetbrains.kotlin:kotlin-stdlib-jdk8:$kotlin_version"
}

compileKotlin {
    kotlinOptions.jvmTarget = "1.8"
}
compileTestKotlin {
    kotlinOptions.jvmTarget = "1.8"
}
```

이제 우리는 *./gradlew assemble*을 이용하여 .jar파일로 컴파일할 수 있고 Android 프로젝트에 임포트 할 수 있습니다.

![Classes Generated](https://cdn-images-1.medium.com/max/1600/1*GGGoF41MuTlrN4zoj_zUaQ.png)

Base.kt로 부터 생성된 세개의 클래스 파일을 볼 수 있습니다.

- BaseKt.class - 상수 API_KEY를 위한 클래스
- 각각 분리된 Helper와 Model 클래스

이제 이 코드를 Activity에서 매우 쉽게 호출할 수 있습니다.
```kotlin
package io.github.jitinsharma.androidapp

import android.os.Bundle
import android.support.v7.app.AppCompatActivity
import io.github.jitinsharma.API_KEY
import io.github.jitinsharma.Helper
import io.github.jitinsharma.Model
import kotlinx.android.synthetic.main.activity_main.*

class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        val helper = Helper()
        val sum = helper.getSum(2,3)
        sumView.text = "Sum: " + sum

        val modifiedList = helper.sliceFilterAndSort(
                listOf("Adam","Aakash","John","Enrique","Abhishek"))
        println(modifiedList)

        val helperId = Helper.helperId
        val helperType = Helper.getHelperType()
        println("$helperId $helperType")

        val model = Model()
        println(model)
        val model2 = model.copy(id = 3)
        println(model2)

        val key = API_KEY
        println(key)
    }
}
```


## iOS로 이동

Kotlin Native 플러그인을 이용하여 iOS에서도 비슷하게 사용할 수 있는지 알아봅시다.

우리의 Base 프로젝트 폴더에 아래처럼 build.gradle 을 추가 합니다.
```gradle
buildscript {
    ext.kotlin_native_version = '0.6'
    repositories {
        mavenCentral()
        maven {
            url "https://dl.bintray.com/jetbrains/kotlin-native-dependencies"
        }
    }
    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-native-gradle-plugin:$kotlin_native_version"
    }
}
group 'io.github.jitinsharma'
apply plugin: "konan"
konan.targets = ["iphone", "iphone_sim"]
konanArtifacts {
    framework('Base') {
        srcDir 'base/src/main/kotlin'
    }
}
```

몇 가지 주목할 점이 있습니다.

- **konan**은 Kotlin코드를 멀티 플랫폼으로 사용할 수 있도록 해주는 Kotlin Native 플러그인 입니다.
- konan.targets 를 이용하여 어느 플랫폼을 대상으로 바이트 코드를 생성할지 지정합니다. *raspberry pi*같은 다른 다른 플랫폼을 추가할 수도 있습니다.
- konanArtifacts 를 이용하여 결과물을 필요에 따라 그 이름과 src 디렉토리를 생성하도록 합니다.

*./gradlew build*를 실행하면 아래처럼 결과물이 생성됩니다.

![Objective C framework created after build](https://cdn-images-1.medium.com/max/1600/1*nn6AHDhQU7Zvz9FBI4k4sg.png)

이제 **iOS framework**인 **Base.framework**라는 이름의 파일이 만들어 졌고, Xcode에서 곧바로 임포트할 수 있습니다. 같이 해보시죠!

![Link framework file to a project](https://cdn-images-1.medium.com/max/1600/1*yqpst-IZIxBYesSVAm57IA.png)

우리는 **Base.kt**로 부터 변환된 코드가 담긴 **Base.h**파일을 받았습니다.

헤더 파일은 읽기에 조금 복잡하지만 Xcode가 이런 파일들을 좀더 이해하기 쉬운 Swift로 변환하는 기능을 제공합니다. 아래 내용은 헤더 파일의 내용입니다. 

```swift
import Foundation

open class KotlinBase : NSObject {
    open class func initialize()
}

extension KotlinBase : NSCopying {
}

open class BaseHelper : KotlinBase {
    public init()
    open func getSum(first: Int32, second: Int32) -> Int32
    open func sliceFilterAndSort(list: [String]) -> [String]
}

open class BaseHelperCompanion : KotlinBase {
    public convenience init()
    open func getHelperType() -> String
    open var helperId: Int32 { get }
}

open class BaseModel : KotlinBase {
    public init(id: Int32, type: String)
    open func component1() -> Int32
    open func component2() -> String
    open func doCopy(id: Int32, type: String) -> BaseModel
    open var id: Int32
    open var type: String
}

open class Base : KotlinBase {
    open class func API_KEY() -> String
}
```

- 기본적으로 NSObject를 상속한 클래스 이름 **KotlinBase** 가 생성 되었고 NSCopying과 이 클래스를 상속한 기타 다른 클래스들이 구현 되었습니다.
- 우리가 iOS 프레임워크 생성시 build.gradle에 기입한 이름 "Base"를 모든 클래스들이 접두어로 사용합니다.
- Kotlin의 **Int** 는 Swift의 Int가 아닌 **Int32**로 변환 되었습니다.
- Kotlin의 List<T> 는 Swift의 Array로 변환 되었습니다.
- Companion object는 init()메서드를 가진 별도 클래스로 변환 되었습니다.
- 모듈 레벨의 API_KEY상수는 클래스 내부 함수로 변환 되었습니다.
- 데이타 클래스로 부터 변환된 **BaseModel**클래스는 데이타 클래스의 copy()와 유사한 *doCopy* 함수를 갖습니다. 하지만 모든 인수들이 초기화중 또는 복사중 특정되어야 하므로, 기본 초기화 또는 복사는 불가능 합니다. 

이제 Swift파일에서 이 함수들을 호출해 봅시다.

```swift
import UIKit
import Base

class ViewController: UIViewController {
    @IBOutlet weak var sumView: UITextField!
    override func viewDidLoad() {
        super.viewDidLoad()
        let base = BaseHelper()
        
        sumView.text = "Sum: \(base.getSum(first: 2, second: 3))"
        let value = base.sliceFilterAndSort(list: ["Adam","Aakash","John","Enrique","Abhishek"])
        value.forEach { (value) in
            NSLog(value)
        }
        
        let model = BaseModel(id: 2, type: "model1")
        let model2 = model.doCopy(id: 3, type: model.type)
        NSLog("\(model2)")
        
        let key = Base.API_KEY()
        let helperId = BaseHelperCompanion.init().helperId
        NSLog(key)
        NSLog("\(helperId)")
    }
}
```

![](https://cdn-images-1.medium.com/max/1600/1*MdYOXbj0Mpias_biYP2xDw.gif)


우리는 기본 Kotlin코드를 이용하여 플랫폼쪽 코드 없이 다중 플랫폼에서 실행시켰습니다. 비록 위 코드가 실제 제품수준의 어플리케이션에 유용하지 않더라도, 앞으로 Kotlin Native와 Kotlin Multiplatform이 더 성숙해지면 일반 프로젝트에 더 많은 로직을 적용할 수 있게 될 것입니다.

[Kotlin Serialization](https://github.com/Kotlin/kotlinx.serialization)은 이 컨셉으로 JVM과 JS를 위한 라이브러리 이고 곧 네이티브 지원이 가능 합니다. 이것을 통해 하나의 라이브러리로 모든 플랫폼에서 데이타 클래스를 JSON으로 serialize및 deserialize할 수 있습니다.

전체 코드는 여기에 있습니다.

https://github.com/jitinsharma/kotlin_multi_target

읽어주셔서 감사합니다.

