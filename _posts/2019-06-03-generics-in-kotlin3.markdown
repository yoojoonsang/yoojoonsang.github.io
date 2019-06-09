---
layout: post
title:  "Generics in Kotlin #3"
categories: jekyll update
description: 이 장에서는 공변성에 대해 알아봅니다.
---

> 무공변성(Invariance)

addInteger라는 이름의 함수를 하나 선언 해보겠습니다. 이 함수는 `MutableList<Any>` 타입의 파라미터를 입력받고, 여기에 숫자 42를 하나 추가하는 일을 하고 있습니다. 얼핏 보았을 때에는 아무런 문제가 없어보입니다.

```kotlin
fun addInteger(list: MutableList<Any>) {
    list.add(42)
}
```

<br>

이번에는 "abc", "def" 두개의 문자열로 구성된 mutableList를 선언하고, addInteger에 전달해보겠습니다. 정상동작 할것 같지만 이 코드는 컴파일이 되지 않습니다. 

```kotlin
fun main() {
    val strings = mutableListOf("abc", "def")
    addInteger(strings)					// 만약 이 줄이 컴파일 된다면
    println(strings.maxBy{ it.length })			// 이 줄을 실행할 때 에러가 발생
}
```

![스크린샷1](../../images/mutablelist-compile-error.png)



이상하죠? `String`은 어찌됐건 `Any`의 하위 타입인데, 어째서 `MutableList<String>` 타입을 `MutableList<Any>` 타입에 넘길수가 없는걸까요? 그 이유는 바로 코틀린에서 타입 매개변수를 기본적으로 **무공변성**(Invariance)으로 만들고 있기 때문입니다. 

다시 말해서, `String`이 `Any`의 하위 타입일지라도, 이와는 별개로 `MutableList<String>`은 `MutableList<Any>`의 하위타입이 아닐 수도 있다는 얘기입니다.  `MutableList<String>`과 `MutableList<Any>` 아무런 관계를 갖지 않게 됩니다.

저처럼 이 개념이 생소한 분들은 왜 이런게 필요할까 의아할수도 있을것 같습니다. 그래서 반대로 가정을 해보겠습니다. 만약 컴파일러가 위의 식을 받아들인다고 하면 어떻게 될까요? 

결과는 정수를 문자열 리스트 뒤에 추가할 수 있게 되고, `MutableList<String>`에 `Integer`가 들어가게 되는 타입 불일치가 발생하게 됩니다. 뿐만 아니라 strings의 원소에 문자열 관련 연산을 하려고 할 때 에러가 발생하게 될 수 있게 됩니다.

<br>



> 공변성(Covariance)

그렇다면 아래의 경우는 어떨까요?

이번에도 역시 `List<String>` 타입의 변수를 선언하여 `List<Any>` 타입의 파라미터를 받는 함수 printContents에 넘기고 있습니다. 단 이번에는 **MutableList**가 아니라 **List**입니다.

```kotlin
fun main() {
    val strings = listOf("abc", "def")printContents(strings)	// abc, def
}

fun printContents(list: List<Any>) {
    println(list.joinToString())
}
```

어라? 이번에는 `List<String>` 타입의 값을 `List<Any>`를 파라미터로 받는 함수에 전달해도 컴파일이 잘됩니다.  실행도 문제없이 잘 됩니다. 이게 가능한 이유는 List의 경우, 타입 파라미터를 **공변성**(Covariance)으로 설정하였기 때문입니다.  이렇게 제네릭 타입 사이에도 하위 타입 관계가 성립하면 그 제네릭 타입을 공변적(Covariant)이라고 합니다.



실제로 List의 구현을 살펴보면 타입 파라미터의 접두사로 out 키워드를 추가해준걸 확인할 수 있는데, out 키워드를 추가함으로써 `List<String>` 이  `List<Any>`의 하위 타입으로 인식되게 됩니다.

![스크린샷1](../../images/list-signature.png)

<br>



반대로 MutableList의 경우에는 타입 파라미터만 명시된걸 확인할 수 있습니다. 이 경우에는 코틀린에서 타입 매개변수를 기본적으로 무공변성으로 만들고 있기 때문에  `List<String>` 가  `List<Any>`의 하위 타입으로 인식되지 않안덨 것입니다.

![스크린샷1](../../images/mutablelist-signature.png)

<br>



> 반공변성(Contravariance)

반공변성(Contravariance)이라는 것도 있습니다. 정확히 공변성의 반대이데, 타입 파라미터를 반공변성으로 설정하면, 타입 자체에서 타입 매개변수 간의 관계가 역전됩니다.

이번에도 예를 통해 살펴보겠습니다. 우선 Food 클래스와 이를 상속하는 FastFood 클래스, 그리고 FastFood 클래스를 상속하는 Burger 클래스를 정의해 보겠습니다.

```kotlin
open class Food
open class FastFood : Food()
class Burger : FastFood()
```

<br>

그리고 Food 클래스와 그 하위 클래스들을 소비할 Consumer 클래스들을 만들어 보겠습니다. EveryBody 클래스는 `Consumer<Food>` 타입으로 Food 클래스를 ModernPeople 클래스는 `Consumer<FastFood>` 타입으로 FastFood 클래스를, 마지막으로  American 클래스는 `Consumer<Burger>` 타입으로 Burger 클래스를 소비할 것입니다.

```kotlin
interface Consumer<T> {
    fun consume(item: T)
}

class EveryBody: Consumer<Food> {
    override fun consume(item: Food) {
        println("Eat Food")
    }
}

class ModernPeople: Consumer<FastFood> {
    override fun consume(item: FastFood) {
        println("Eat Fast Food")
    }
}

class American: Consumer<Burger> {
    override fun consume(item: Burger) {
        println("Eat Burger")
    }
}
```

<br>

이 때 타입 파라미터의 접두어가 없으므로 Consumer 클래스 간에는 무공변성이 유지됩니다.  따라서 서로 상/하위 타입의 개념이 없게되고, `Consumer<Food>` 타입이 필요한 곳에는 EveryBody 클래스, `Consumer<FastFood>`가 필요한 곳에는 ModernPeople 클래스, 마지막으로 `Consumer<Burger>` 타입이 필요한 곳에는 American 클래스가 할당 될 수 있습니다. 

```kotlin
fun main() {
    val foodConsumer: Consumer<Food> = EveryBody()
    val fastFoodConsumer: Consumer<FastFood> = ModernPeople()
    val burgerConsumer: Consumer<Burger> = American()
}
```

<br>

그리고 이제 타입 파라미터의 접두어로 in을 붙여주게 되면, Consumer 클래스에서 타입 파라미터 간의 관계가 역전됩니다. 

```kotlin
interface Consumer<in T> {
    fun consume(item: T)
}
// 아래와 같은 순서로 타입이 역전됨
// Consumer<Burger>  <-  Consumer<FastFood>  <-  Consumer<Food>
```

<br>

이에 따라 `Consumer<Burger>`가 `Consumer<FastFood>`의 슈퍼타입이 되고, `Consumer<FastFood>`가 `Consumer<Food>`의 슈퍼타입이 되게 됩니다. 그리고 그 결과, `Consumer<Burger>`가 필요한 곳에`Consumer<FastFood>`나 `Consumer<FastFood>`를 할당할 수 있게 됩니다.

```kotlin
fun main() {
    val foodConsumer: Consumer<Burger> = EveryBody()
    val fastFoodConsumer: Consumer<Burger> = ModernPeople()
    val burgerConsumer: Consumer<Burger> = American()
}
```