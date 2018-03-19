---
layout: post
title: Kotlin Design Patterns(번역)
img: design_patterns.jpg
tags: [Kotlin, Design, patterns]
---


원문 : <https://proandroiddev.com/kotlin-design-patterns-8e152540ee2c>



# Kotlin Design Patterns 

![](https://cdn-images-1.medium.com/max/1600/1*qdDNIUIP6dlK6oQ6WviZ_w.jpeg)

*This article was originally posted on* [*June 2017 on LinkedIn*](https://www.linkedin.com/pulse/kotlin-design-patterns-alexey-soshin)



이런 말이 있습니다. "디자인 패턴은 특정 언어의 단점을 해결하는 방법이다." [1996년도 쯔음 Lisp나 Scheme 옹호자들](http://norvig.com/design-patterns/)이 한 말이 아니라면, 매우 흥미로울 것입니다.



하지만, Kotlin 언어 디자이너들은 이 말에 진심으로 감명을 받은거 같습니다.



## Singletone

물론 마음속에 떠오르는 첫 번째 디자인 패턴은 싱글톤 입니다. 그리고 이것은 바로 언어 자체에 내장되어 있습니다. ***object*** 를 보세요:
```kotlin
object JustSingleton {
    val value: String = "Just a value"
}
```

이제 하나의  `JustSingleton`이 생겼습니다. `value`는 패키지 어디서든 사용 가능합니다.

그리고, 생각할지도 모르겠지만, 정적 초기화가 아닙니다. 안에서 뭔가 시간이 걸리는 일을  해봅시다:

```kotlin
object SlowSingleton {
    val value: String
    init {
        var uuid = ""
        val total = measureTimeMillis {
            println("Computing")
            for(i in 1..10_000_000) {
                uuid = UUID.randomUUID().toString()
            }
        }
        value = uuid
        println("Done computing in ${total}ms")
    }
}
```



최초 호출될 때 느리게 초기화 됩니다:

```kotlin
@org.testng.annotations.Test
fun testSingleton() {
    println("Test started")
    for (i in 1..3) {
        val total = measureTimeMillis {
			println(SlowSingleton.value)
        }
        println("Took $total ms")
    }
}
```



출력:

```
Test started
Computing
Done computing in 5376ms
"45f7d567-9b3e-4099-98e6-569ebc26ecdf"
Took 5377 ms
"45f7d567-9b3e-4099-98e6-569ebc26ecdf"
Took 0 ms
"45f7d567-9b3e-4099-98e6-569ebc26ecdf"
Took 0ms
```



object가 코드상에 정의 되어 있더라도, 당신이 그것을 사용하지 않으면 이 작업은 0ms에 완료 됩니다.

```kotlin
val total = measureTimeMillis {
    //println(SlowSingleton.value)
}
```



출력:

```
Test started
Took 0 ms
Took 0 ms
Took 0 ms
```



## Decorator

다음은 [데코레이터](https://en.wikipedia.org/wiki/Decorator_pattern) 입니다. 이미 알고 계신것처럼, 어떤 다른 클래스에 약간의 기능을 추가할 수 있게 해줍니다. IntelliJ가 당신을 위해 그것을 생성해 줄 수 있습니다. 예~. 이제 한걸음 더 나아갑니다.

새로운 키가 추가될 때 마다 환호하는 HashMap을 만들어 볼까요?

지원 인스턴스를 컨스트럭터에 정의하고, ***by*** 키워드로 모든 메서드를 위임할 수 있습니다.

```kotlin
/**
 * using <code>by</code> keyword you can delegate all but overridden methods
 */
class HappyMap<K, V>(val map: MutableMap<K, V> = mutableMapOf()): MutableMap<K, V> by map {
    override fun put(key: K, value: V): V? {
        return map.put(key, value).apply {
            if (this == null) {
                println("Yay! $key")
            }
        }
    }
}
```

우리는 우리의 맵에 대괄호를 이용하여 값에 접근할 수 있고, 다른 모든 메서드는 기존과 동일하게 동작한다는 것을 명심하세요:

```kotlin
@org.testng.annotations.Test
fun testDecorator() {
    val map = HappyMap<String, String>()
    val result = captureOutput {
        map["A"] = "B"
        map["B"] = "C"
        map["A"] = "C"
        map.remove("A")
        map["A"] = "C"
    }
    assertEquals(mapOf("A" to "C", "B" to "C"), map.map)
    assertEquals(listOf("Yay! A", "Yay! B", "Yay! A"), (result))
}
```





## FactoryMethod

***Companion object*** 를 사용하여 손쉽게 [팩터리 메서드](https://en.wikipedia.org/wiki/Factory_method_pattern) 를 구현할 수 있습니다. 객체가 초기화 되는것을 컨트롤 하려는 유일한 객체 입니다. 아마도 그 안에서 비밀스러운 어떤 일을 하기 때문일 겁니다.

```kotlin
class SecretiveGirl private constructor(val age: Int,
                                        val name: String = "A girl has no name",
                                        val desires: String = "A girl has no desires") {
    companion object {
        fun newGirl(vararg desires: String): SecretiveGirl {
            return SecretiveGirl(17, desires = desires.joinToString(", "))
        }
        fun newGirl(name: String): SecretiveGirl {
            return SecretiveGirl(17, name = name)
        }
    }
}
```



이제 아무도 SecretiveGirl의 나이를 변경할 수 없습니다 :

```kotlin
@org.testng.annotations.Test
fun FactoryMehtodTest() {
    // Cannot do this, constructor is private
    // val arya = SecretiveGirl();
    val arya1 = SecretiveGirl.newGirl("Arry")
    assertEquals(17, arya1.age)
    assertEquals("Arry", arya1.name)
    assertEquals("A girl has no desires", arya1.desires)
    val arya2 = SecretiveGirl.newGirl("Cersei Lannister", "Joffrey", "Ilyn Payne")
    assertEquals(17, arya2.age)
    assertEquals("A girl has no name", arya2.name)
    assertEquals("Cersei Lannister, Joffrey, Ilyn Payne", arya2.desires)
}
```



## Strategy

오늘 마지막은 [전략](https://en.wikipedia.org/wiki/Strategy_pattern) 입니다. Kotlin은 [고차함수](https://kotlinlang.org/docs/reference/lambdas.html) 를 가지고 있어서 이 역시 매우 쉽습니다:

```kotlin
class UncertainAnimal {
    var makeSound = fun() {
        println("Meow!")
    }
}
```

그리고 런타임에 동작을 바꾸려면:

```kotlin
@org.testng.annotations.Test
fun testStrategy() {
    val someAnimal = UncertainAnimal()
    val output = captureOutput {
        someAnimal.makeSound()
        someAnimal.makeSound = fun () {
            println("Woof!")
        }
        someAnimal.makeSound()
    }
    assertEquals(listOf("Meow!", "Woof!"), output)
}
```

이것은 전략 패턴이고, 시그니처를 바꾸는 것은 동작하지 않습니다.

```kotlin
// Won't compile!
someAnimal.makeSound = fun (message: String) {
    println("$message")
}
```



평소대로, 모든 코드는 제 GitHub 페이지에 있습니다:

[github.com/alexeysoshin/KotlinDesignPatterns](https://github.com/alexeysoshin/KotlinDesignPatterns)



그리고, Kotlin과 디자인 패턴을 배우고 싶다면 아주 훌륭한 ["Kotlin in Action" 책](https://www.manning.com/books/kotlin-in-action) 이 있습니다. 비록 곧 이 언어를 사용할 계획이 없더라도, 재미있는 읽을거리 입니다. 





























