---
layout: post
title: Kotlin Syntax Part I - why am I excluded?(번역)
description: >
  talking about Kotlin syntax
image: 
categories: [Kotlin]
tags: [Kotlin, syntax]
comments: true
---

원문 : <https://proandroiddev.com/kotlin-syntax-part-i-why-am-i-excluded-86772a61fade>


# Kotlin Syntax Part I - why am I excluded?(번역)



![img](https://cdn-images-1.medium.com/max/2000/1*QcHZLfsT95RIkww_2Hoqyg.png)


Hi everyone again, welcome to article number on 2 of the Kotlin Playground Series. There wasn’t that much code in the previous article but if you read it, you probably saw some new syntax that you’re not familiar with if coming from Java.  
여러분 안녕하세요, Kotlin Playground 시리즈 두번째 글에 오신것을 환영 합니다. 지난번 글에 그렇게 많은 코드가 있진 않았지만, 그 글을 읽어 보셨다면, Java에서 보던 것과는 다른 새로운 문법들을 보셨을 겁니다.

In this and following article I’ll cover some of Kotlin base syntax as I believe it’s important before we jump into other examples. But enough talk and let’s look at some Kotlin code.  
다른 예제로 넘어가기 전에 이 글과 다음 글에서 제가 중요하다고 생각하는 몇 가지 Kotlin 기본 문법을 다루겠습니다. 

But enough talk and let’s look at some Kotlin code.  
말은 충분히 했으니, Kotlin 코드를 봅시다.


### Classes
There are some differences in declaring classes in comparison with Java. Let’s look at some examples in the code below. An important aspect in Kotlin is that you can declare multiple classes without any relation in the same file. Probably not what you want to do but nice for this example.  
자바와 비교했을 때, 클래스를 선언하는데 차이가 있습니다. 아래 나오는 예제 코드를 살펴 봅시다. Kotlin 에서 중요한 형상은 서로 연관이 없는 여러개의 클래스를 같은 파일안에 선언할 수 있다는 점입니다. 아마 여러분 모두 이렇게 하고 싶지 않을 텐데, 이 예제에서는 좋은 방법입니다.

Kotlin classes are declared using the keyword class and consist of the class name, the header and the body surround by curly braces. But there a lot of optional things when declaring Kotlin classes.  
Kotlin 클래스는 `class` 키워드와 클래스 이름, 헤더 그리고 중괄호로 감싸진 몸통으로 선업 됩니다. 하지만 Kotlin 클래스를 선언할 때 수많은 옵션들이 있습니다.

``` kotlin
class ClassWithEmptyBody

class ClassWithBody(val name: String) {
	val nameLength = name.length
}
```

If your class has a body you need to wrap it in curly braces but if there’s no class body you can actually omit it.  
만약 만들려는 클래스가 바디를 가지고 있으면 바디를 중괄호로 감싸주어야 하지만, 클래스가 바디를 갖지 않으면 중괄호를 생략할 수 있습니다. 

``` kotlin
class ClassWithoutConstructorKeyword (val name: String) {
    val nameLength = name.length
}

class ClassWithConstructorKeyword constructor (val name: String) {
    val nameLength = name.length
}

class ClassWithAccessModifier public constructor (val name: String) {
    val nameLength = name.length
}

class ClassWithInitialization public constructor (val name: String) {
    val nameLength : Int

    init {
        nameLength = name.length
    }
}

class ClassWithMultipleConstructors (val name: String) {
    val nameLength = name.length

    constructor(name: String, age: Int) : this(name) {
        val yearOfBirth = LocalDateTime.now().year - age
    }
}
```


The constructor keyword is also optional if your class header doesn’t need any access modifiers or annotations.  
클래스 헤더가 어노테이션이나 접근 지정자(access modifiers)가 필요하지 않으면  `constructor` 키워드 역시 옵션입니다. 


As in Java, a class can have multiple constructors, but in Kotlin apart from properties declaration, we can’t have any code in the main constructor. All initialisation code needs to be inside an init method.  
Java에서 처럼, 클래스는 중복된 생성자를 가질 수 있습니다. 하지만 Kotlin은 속성 선언을 제외하면 주 생성자는 코드를 가질 수 없습니다. 모든 초기화 코드는 `init` 메서드 안에 있어야 합니다.


``` kotlin
val ClassWithEmptyBody = ClassWithEmptyBody()

val ClassWithBody = ClassWithBody("a name")
```
To instantiate classes we just assign the Class calling it as if it was a normal function as there’s no new keyword in Kotlin.  
클래스를 인스턴스화 하려면 Kotlin에는 `new` 키워드가 없으므로 마치 일반 함수처럼 클래스를 호출 합니다.

``` kotlin
open class ParentClass (someNumber: Int)

class ChildClass (someNumber: Int) : ParentClass(someNumber)


interface InterfaceExample {
    fun test()
}

class InterfaceImplementation : InterfaceExample {
    override fun test() {
        //Some implementation
    }
}
```
And to end the classes section, we can see that in Kotlin there’s no extends or implements keywords. To extend or implement another class we can just use : and that’s it. Easy right?  
클래스 섹션 마지막으로, Kotlin에는 `extends` 또는 `implements` 키워드가 없는걸 볼 수 있습니다. 다른 클래스를 상속하거나 구현하려면 그냥 `:`만 쓰면 됩니다. 쉽죠?



## Functions

Moving on to functions, as with constructors if you want them public, no need to use the access modifier. While in Java the default access modifier is limited, in Kotlin is public so no need to have it everywhere.  
함수로 넘어가 봅시다. 생성자에서 처럼 함수가 public 이길 원한다면, 접근 제어자를 사용할 필요가 없습니다. Java에서 기본(default) 접근 제어자는 제한적 이지만, Kotlin은 기본이 public이므로 모든곳에서 public을 가질 필요가 없습니다.

```kotlin
fun aFunction(x: Int, y: Int): Int {
    return x + y
}

fun aFunctionWithExplicitReturnType(x: Int, y: Int): Unit {
    println("sum of $x and $y is ${x + y}")
}

fun aFunctionWithoutReduntant(x: Int, y: Int) {
    println("sum of $x and $y is ${x + y}")
}

fun aFunctionAsExpression(x: Int, y: Int) = x + y
```

As you can see, the return type of a function in Kotlin is in the end rather than the beginning like in Java. The same is also true for the params, with the return type being preceded by the param name and a colon.  
보시는 것처럼 Kotlin 함수의 리턴 타입은 Java와 달리 마지막에 있습니다. 리턴 타입과 마찬가지로 파라미터도 이름과 콜론이 앞에 옵니다.

Functions that don’t return anything (Void functions in Java) have return type Unit in Kotlin. And because in Kotlin we don’t have to explicitly write obvious things this is also optional and you can omit it if your function doesn’t have a return value.  
Kotlin에서는 아무것도 리턴하지 않는 함수(Java의 Void 함수같은)는 `Unit` 형을 리턴 합니다. 그리고 Kotlin에서는 명백한 것은 명시적으로 적을 필요가 없기 때문에, 함수가 리턴 값을 갖지 않으면 이것도 생략할 수 있습니다.  

Another very cool feature regarding Kotlin functions is that they can be declared as expressions. If your function has a return type you can literally use as in the last example above and drop the function body. While this is super cool, it can be a bit dangerous as people can go wild with this, so please use it with care.  
Kotlin 함수의 또 다른 멋진 기능은 표현식으로 선언할 수 있다는 것입니다. 함수가 리턴 타입이 있는 경우 위 마지막 예제처럼 문자 그대로 수식으로 사용하고 함수 본문을 없앨 수 있습니다. 이것은 굉장히 멋진 기능이지만 꽤 위험하니 조심해서 사용하세요.



## Variables

In Kotlin we can use either the keyword `var` or `val` to define variables. The difference between both is that `var` is used to define mutable variables, while `val` is used to define read-only or immutable variables depending on the train you join, but we’ll get to that after. First, let’s see some examples:  

Kotlin 에서는 변수 정의를 위해 `var` 또는 `val` 키워드를 사용할 수 있습니다. 이 둘의 차이점은 `var`는 가변 변수를 정의하는데 사용되는 반면, `val`은 읽기전용 또는 불변 변수를 정의하는데 사용 됩니다. 우선 예제를 봅시다.

```kotlin
val immediateAssignment: Int = 1

val inferredTypeInt = 2

val deferredAssignment: Int
deferredAssignment = 3

var mutableInferredTypeString = "some string"
mutableInferredTypeString += "some concat"
```

Looking at the second variable above, it’s initialised with a default value, so Kotlin automatically infers its type as `Int`. This is another cool feature from Kotlin called Inferred type. So if the type is inferred automatically that means type declaration it’s not explicitly necessary, which in Kotlin means we can drop it. This works with custom objects as well.

두번째 변수를 보시면, 기본 값으로 초기화 되었고, Kotlin은 자동으로 변수의 타입이 `Int`인것을 추론 합니다. 이것은 추론 타입이라 불리는 Kotlin의 또 다른 멋진 기능 입니다. 그래서 만약 자동으로 추론이 가능한 타입이라면 타입을 선언하는 것이 불필요하다는 것을 의미하며, 그 말은 Kotlin에서는 선언을 하지 않아도 된다는 뜻입니다. 이 기능은 커스텀 오브젝트에서도 동작 합니다.

Going back to `val` variables and the fact that they’re read-only or immutable there is some discussion around this. If we’re talking about regular variables we can definitely say that they’re immutable but it’s a different story when talking about class properties. Let’s see the example below:

`val`변수로 돌아 와서, 그것들이 읽기 전용 또는 불변이라는 사실에 대해서 몇가지 의견이 있습니다. 만약 우리가 보통의 변수들을 말한다면 그것들이 확실히 불변이라고 말할 수 있지만, 클래스 프로퍼티에 대해 말할때는 다릅니다. 아래 예제를 봅시다:

```kotlin
class MutableValExample(val yearOfBirth: Int) {
    val age: Int
        get() = LocalDateTime.now().year - yearOfBirth
}
```

In the context of class properties, `val` only means that a property won’t have a setter method which means that the property would be immutable. But as we can see above, we can have a custom getter method that returns a different value each time someone accesses the property. So, while the property is immutable we’re actually getting a different value every time the date.

클래스 프로퍼티의 상황에서, `val` 은 단지 그 프로퍼티가 setter 메서드를 갖지 않아서 불변이라는 것을 의미 합니다. 하지만 위에서 보는 것처럼, 우리는 커스텀 getter메서드를 만들어서 그 메서드에 접근할 때 마다 다른 값을 리턴하도록 할 수 있습니다.  결국 프로퍼티가 불변일지라도, 날짜에 대해서 다른 값을 얻게 됩니다.

This is why some people call `val` variables read-only rather than immutable. The important is to be aware of this use case, so maybe a good tip is to try not to use `val` properties with custom getters as it can get confusing.

이런 이유로 일부 사람들이 `val` 변수를 불변보다는 읽기 전용 이라고 부릅니다. 이러한 사용 형태에 주의하는 것이 중요하고, 그래서 혼란을 줄 수 있으므로 커스텀 getter를 가지는 프로퍼티에 `val` 을 사용하지 않는 것이 좋은 팁이 될수 있습니다.

And if you’re ever confused about `var` and `val` and which one is which, [Kaushik Gopal](https://medium.com/@kaushikgopal) gave a nice tip in episode [85 of Fragment Podcast](http://fragmentedpodcast.com/episodes/85/) where they have a casual Kotlin discussion with Dan Kim (I actually recommend this episode is quite nice):

그리고 만약 당신이 계속 `var` 와 `val` 이 혼동되면, Dan Kim과 Kotlin이야기를 나누는 [85 of Fragment Podcast](http://fragmentedpodcast.com/episodes/85/) 에서 [Kaushik Gopal](https://medium.com/@kaushikgopal) 이 제시한 좋은 팁이 있습니다. (이 에피소드가 꽤 좋아서 추천합니다)

> Think of `var` as ‘variable’, so by definition something that can change, and `val`as ‘value’, so something that has a value and doesn’t necessarily change.
>
> `var` 는 변경 가능한 어떤 것을 정의하는 변수(variable)라고 생각 하고, `val` 은 변경될 필요가 없는 어떤 값(value) 이라고 생각 하세요.



# Nullability

Probably one of the biggest arguments people use to make the switch from Java to Kotlin is the fact that Kotlin is a null-safe language. But what does this mean? Let’s check those 3 seagulls on the left of the image and find out what new operators are those.

아마도 사람들이  Java에서 Kotlin 으로 변경을 하는데 가장 큰 이유는 이유 중 하나는 Kotlin이 null-safe 언어라는 사실일 것이다. 그런데 그게 뭘 의미 하는 걸까? 여기 이미지 왼쪽에 있는 갈매기 3마리를 확인해 보고 어떤 새로운 오퍼레이터가 무엇인지 알아봅시다. 

Null reference was considered by its own inventor, Tony Hoare, [the billion dollar mistake](https://en.wikipedia.org/wiki/Tony_Hoare) and the truth is, as Java developers we spend way too much time dealing Null Pointer Exceptions (NPE) right?

Null을 창조한 Tony Hoare 조차, Null 참조가 [10억 달러의 실수](https://en.wikipedia.org/wiki/Tony_Hoare) 라고 했으며, Java개발자들이 널 포인터 예외(NPE) 때문에 많은 시간을 소비하는 것은 사실 입니다. 그렇죠?

Well, Kotlin was designed in a way that NPE should not happen unless one of these things happen:

> Some java code causing it
>
> Explicitly calling `throw NullPointerException()`
>
> Using the `!!` operator

Kotlin은 아래 상황중 하나가 아니면, NPE가 일어나지 않도록 설계 되었다:

> Java 코드에서 발생
>
> 명시적으로 `throw NullPointerException()` 호출
>
> `!!` 오퍼레이터 사용



But let’s look at the Kotlin syntax around null-safety and how it works. Before going into the operators in the image, let’s see what the `?` operator on its own is about.

이제 null-safety및 작동 원리에 대한 Kotlin구문을 살펴보겠습니다. 이미지에 있는 오퍼레이터를 알아보기 전에 `?` 오퍼레이터 자체에 대해 알아보겠습니다.

```kotlin
fun NonNullString() {
    var nonNullString: String = "some string"
    nonNullString = null // compilation error
}

fun NullString() {
    var nullString: String? = "some string"
    nullString = null // nothing wrong
}
```

Looking at the example above, first, we have an example of a regular String variable in Kotlin, which by default is not nullable. If we try to assign `null` to it, we get a compilation error.

위 예제를 살펴보면, 우선 기본 값이 null이 아닌 Kotlin의 일반적인 String 변수의 예제가 있습니다. 이 변수에 `null` 을 할당 하려고 하면 컴파일 에러가 발생 합니다.

But if we look at the second example there’s no compilation error, what’s the difference? By appending our variable type with `?` like we see above with `var nullString :String?` we declare our variable nullable and we can assign `null` to it without any compilation errors.

하지만, 두번째 예제를 보면 컴파일 에러가 없습니다. 차이가 무었일까요? 위에 보이는 `var nullString :String?` 처럼 변수 타입뒤에 `?` 를 붙이면 컴파일 에러 없이 변수가 널이 될 수 있고 `null`을 할당할 수 있습니다.

Cool right? This means we have full control of the nullability of our variables and we have to explicitly declare them nullable if we want them to accept `null` values.

멋지죠? 이것은 우리가 변수에 널이 가능한지 여부를 완벽히 컨트롤 할 수 있고, 우리가 원하는 경우 `null` 을 가질수 있도록 선언할 수 있다는 뜻입니다.

Now let’s look at the operators in the feature image, starting with the `?.`operator. This is called the `Safe call operator` and by the name, or if you’re familiar with C#, for example, you can guess what it does. Let’s see an example:

이제, `?.` 오퍼레이터 부터 시작되는 대표 이미지에 있는 오퍼레이터들을 살펴 봅시다. 이것은 `안전 호출 오퍼레이터` 라고 불리고이름을 보거나 또는 예를 들어 만약 당신이 C#과 친숙 하다면, 그게 무엇인지 추측할 수 있습니다. 예제를 봅시다:

```kotlin
var nullString: String? = "some string"
var length :Int
var lengthOrNull :Int?

nullString.length // compilation error
length = nullString?.length //compilation error
lengthOrNull = nullString?.length //nothing wrong
```

Ok, so we have our `nullString` nullable variable from before and some operations on it. A couple with compilation errors and the last one without, let’s check them in more detail.

좋습니다, 이제 우리는 널이 가능한 `nullString` 변수와 그 변수에 몇개의 연산자를 가지고 있습니다. 두개의 컴파일 에러가 있고 마지막 것은 에러가 없습니다. 좀더 자세히 들여다 봅시다.

```
Only safe (?.) or non-null asserted (!!.) calls are allowed on a nullable receiver of type String?
오직 안전한 (?.) 또는 널이 아님을 가정한 (!!.) 호출들만 널이 가능한 String? 타입에 널 가능한 리시버로 사용할 수 있습니다. 
```

The first assignment will give us this compilation error. Why? Well, the reason is obvious, if the variable is null the call would result in an NPE at runtime. But because Kotlin is null safe we get the error at compilation time. Nice, right?

첫번째 변수 할당은 컴파일 에러를 내보냅니다. 왜냐? 이유는 명백합니다. 만약 그 변수가 널이라면 런타임에 NPE를 발생합니다. 하지만 Kotlin은 널에 안전하며 우리는 컴파일 시점에 에러를 받게 됩니다. 참 좋지요?

```
Type mismatch: inferred type is Int? but Int was expected
```

In the second assignment we’re using the `?.` operator to read the length of the string so what’s the problem? The safe call operator will return the length of the string if this is not null or null otherwise. And because `length` is of the type `Int` (not nullable), we cannot assign this expression to it.

두번째 값 할당에서 우리는스트링의 길이를 읽기 위해  `?.` 오퍼레이터를 사용 했습니다. 그런데 뭐가 문제 일까요? 안전 호출 오퍼레이터는 만약 스트링이 널이 아니면 스트링의 길이를 리턴 하고, 널인 경우 널을 리턴 합니다. 그리고 `length` 는(널이 될수 없는) `Int` 형이어서 이 식을 할당할 수 없습니다.

The way to go in this case is to use the last example and assign `nullString?.length` to `lengthOrNull` that is of type `Int?` therefore nullable.

이런 경우 사용해야 되는 방법은 마지막 예제처럼 결과 값이 널일 수 있기 때문에 `Int?` 형인 `lengthOrNull` 변수에 `nullString?.length` 를 할당하는 방법 입니다.

Moving on to `?:` or `Elvis Operator` as it’s called. This is a simple operator that basically returns the expression to its left if its not null, or the one to the right if the left one is null. Let’s see an example.

다음으로`엘비스 오퍼레이터` 라 불리는 `?:` 로 넘어 갑시다. 이것은 기본적으로 왼편의 식이 널이 아니면 왼편의 식을 리턴하고 널인 경우 오른편의 식을 리턴하는 간단한 오퍼레이터 입니다. 예제를 봅시다:

```kotlin
var nullString: String? = "some string"
var length :Int

nullString.length // compilation error
length = nullString?.length ?: -1
```

As with the previous example, with the first assignment, we will get the same compilation error. But in the second assignment, because we’re using the `?:`operator to return length as -1 in the case nullString is effectively `null` we can now assign `length` to an `Int` (non-nullable) variable.

이전 예제에서 처럼 첫번째 접근에서 동일한 컴파일 에러가 발생 합니다. 하지만 두번째 접근에서는 `?:` 오퍼레이터를 사용하여 nullString 이 `널` 인경우 -1을 리턴하도록 했기 때문에 에러가 발생하지 않고 (널이 될수없는) `Int` 변수에 `length`를 할당할 수 있습니다.

One more to go. The `!!` operator, or the **FORBIDDEN ONE**. There’s a reason why is the last one of the 3 seagulls. There’s only a reason to use this, and it’s if you like NPEs, which I hope you don’t. No developer does right? **RIGHT?**

한가지 더. `!!` 오퍼레이터 또는 **금지된 것**이 있습니다. 3마기 갈매기중 마지막인 이유 입니다. 이걸 사용할 한가지의 이유가 있는데, 그러지 않기를 바라지만, 당신이 NPE를 좋아하는 경우 입니다. 어떤 개발자도 안 그렇죠? **그렇죠?**

```
val length = someString!!.length
```



This operator is called `Not-null assertion` and basically tells the compiler that you’re 100% sure that the expression you’re using in it will never be null. Did I mention [the billion dollar mistake](https://en.wikipedia.org/wiki/Tony_Hoare) before? I know I did but it’s very relevant to mention it again here. And please go up again and see the list of the only 3 things that can cause NPE in Kotlin, our friend `!!` is right there.

이 연산자는 `Not-null assertion` 이라고 불립니다. 컴파일러에게 당신이 이 식은 100% 널이 될 수 없다고 말하는 것과 같습니다. 내가 [10억 달러의 실수](https://en.wikipedia.org/wiki/Tony_Hoare) 를 언급 했던가요? 내가 말했던걸 알지만 여기서 다시 언급하는것은 의의가 있습니다. 위로 다시 올라가서 Kotlin 에서 NPE가 발생할 수 있는 3가지를 다시 살펴 보시면, 우리 친구 `!!` 가 바로 거기 있습니다.

Please don’t use this operator unless you really have to, which is never hopefully. If you’re using Android Studio converter to convert Java to Kotlin you’ll see loads of these but don’t assume they’re good. That’s because it’s converting Java code that is not null-safe and I would actually use it as a good excuse to look at your code and take advantage of Kotlin null-safe features to refactor it.

당신이 정말적으로, 정말로 해야 하는 경우가 아니라면, 이 오퍼레이터를 사용하지 마세요. 만약 안드로이드 스튜디오에서 Java를 Kotlin으로 변환하면 이런것을 많이 보게 되겠지만, 그것들이 좋은 거라고 인식하지 마세요. 왜냐하면 널에 안전하지 않은 Java코드를 변환하기 때문이고, 그것은 사실 Kotlin의 널에 안전한 기능을 사용하여 당신의 코드를 개선하는 것의 장점을 잘 보여주는 좋은 사례입니다.



# But wait, where did the semi-colon go?

Ok and now finally to explain why the poor seagull on the right is feeling excluded next to those operators we just saw in the previous section about Nullability.

좋습니다. 이제 우리가 방금 살펴본 널가능성에 관련된 오퍼레이터들 다음에 서있는 오른쪽의 불쌍한 갈매기가 왜 외면당한 느낌인지 설명하겠습니다. 

Well, that’s because we don’t actually need the semi-colon all over the place like we do in Java (you probably noticed it already). Kotlin provides semi-colon inference, so statements, declarations etc, are separated by the pseudo-token [SEMI](https://kotlinlang.org/docs/reference/grammar.html#SEMI) and in most cases, there’s no need for semi-colons in Kotlin code.

자, 왜냐하면 우리는 사실 (당신이 이미 눈치챘겠지만) Java에서 처럼 모든곳이 세미 콜론이 필요하지 않기 때문 입니다. Kotlin은 구문, 변수선언 에서 처럼 세미콜론 추론을 제공하고, 의사토큰  [SEMI](https://kotlinlang.org/docs/reference/grammar.html#SEMI) 로 구분되어 대부분의 경우 Kotlin코드에서 세미 콜론을 사용할 필요가 없습니다.

This is one of the biggest differences between Java and Kotlin syntax, but there are a couple of exceptions:

이것은 Java와 Kotlin 문법의 가장 큰 차이점 중 하나입니다. 하지만 두가지 예외가 있습니다:

```kotlin
enum class SemiColonNotRequired {
    WITH,
    WITHOUT
}

enum class SemiColonRequired {
    WITH,
    WITHOUT;

    fun isRequired(): Boolean = this == WITH
}
```

If you have an enum that also contains a property or a function you explicitly need to use a semi-colon to separate the enum constant list from the property or function definition.

만약 속성이나 함수를 포함하는 열거형이 있다면, 열거형 상수 리스트와 속성 또는 함수 정의와 구분하기 위해 세미 콜론을 사용해야 합니다. 

If we try to remove the semi-colon from the second example we get a compilation error straight away telling us we need to use a semi-colon to close the enum constants entry.

만약 우리가 두번째 예제에서 세미 콜론을 제거하려고 하면 곧바로 열거형 상수 목록을 끝내기 위해 세미 콜론을 사용해야 한다는 컴파일 에러가 발생 합니다.

![pic2|center|0x0](https://cdn-images-1.medium.com/max/1600/1*kA1u0KfgrQNfHhZwZTKRcw.png)



The only other situation where you need to use a semi-colon is if you want to have 2 statements on the same line like in the example below:

세미 콜론을 사용해야 하는 또 다른 상황은 아래 예제 처럼 2개의 구문을 한 줄에 표현하기 원할 때 입니다 :

```kotlin
var list = 1..10
list.forEach { val result = it * 2; println("$it times 2 is $result")}
```

If we try and replace that semi-colon with just a comma, we also get a compilation error telling us we need to use a semi-colon to separate expressions on the same line:

만약 우리가 세미 콜론을 콤마로 바꾸려고 하면 동일한 라인에서 두개의 구문을 사용하려면 세미 콜론을 사용해야 한다는 컴파일 에러가 발생 합니다.

![img](https://cdn-images-1.medium.com/max/1600/1*nc2H7jxuC4c1AzSneOLLRg.png)



And that’s all for today, hopefully, you are now more comfortable with some of the basic Kotlin syntax. If you enjoyed the article don’t forget to give some 👏 and please share your ideas and comments.

오늘 내용은 여기까지 입니다. 이제 몇 가지 Kotlin 기본 문법에 여러분들이 조금 더 편안해 졌기를 바랍니다. 이 글이 마음에 들었다면  👏 주시는것 잊지 마시고, 여러분의 생각과 코멘트를 공유해 주세요.

As usual, you can find the code used for this article in the Series Github project under the `syntax` package. See you in number 3 for more Kotlin syntax examples.

이 글에 사용된 코드들은 언제나 깃헙 프로젝트 `syntax` 패키지에서 찾으실 수 있습니다. 

