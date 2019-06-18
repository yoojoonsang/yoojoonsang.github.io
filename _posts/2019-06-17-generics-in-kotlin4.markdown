---
layout: post
title:  "Generics in Kotlin #4"
categories: jekyll update
description: 이 장에서는 스타 프로젝션에 대해 알아봅니다.
---

이번에는 Car 클래스와 이를 상속한 Hyundai 클래스의 예를 통해 스타프로젝션에 대해 알아보겠습니다. 

```kotlin
open class Car
class Hyundai : Car()
```

<br>

다음은 `MutableList<Car>` 타입의 파라미터를 입력받고, 파라미터의 원소들을 하나씩 출력해주는 함수입니다.

```kotlin
fun printCars(cars: MutableList<Car>) {
    for (car in cars) {
       println(car)
    }
}
```

<br>

이제 Car 타입의 객체로 구성된 MutableList를 만들고 printCars 함수를 호출해 보겠습니다. 함수 호출 결과는 주석의 내용과 같습니다.

```kotlin
fun main() {
    var cars1 = mutableListOf(Car(), Car(), Car())

    printCars(cars1)	//함수 호출 결과: Car@a09ee92, Car@30f39991, Car@452b3a41
}
```

<br>

이번에는 Hyundai 타입의 객체로 구성된 MutableList를 통해 printCars 함수를 호출해 보겠습니다.  하지만 이번에는 MutableList의 무공변성(Invariance) 때문에 컴파일이 되지 않습니다. 

```kotlin
fun main() {
	var hyundais1 = mutableListOf(Hyundai(), Hyundai(), Hyundai())

    printCars(hyundais1)	//컴파일 에러
}
```

![스크린샷1](../../images/mutablelist-hyundai.png)

<br>

여기서 우리는 두가지 방법을 선택해 위의 문제를 해결해 볼 수 있습니다. 첫번째 방법은 printCars 함수를 아래와 같이 제네릭 타입의 함수로 만드는 방법입니다. 

```kotlin
fun <T> printCars(cars: MutableList<T>) {
    for (car in cars) {
       println(car)
    }
}
```

<br>

그리고 두번째 방법은 스타 프로젝션(Star-Projection)을 사용하는 방법입니다. 이 방법을 사용하면 위의 제네릭 방식과 마찬가지로 모든 MutableList를 파라미터로 입력받을 수 있게 됩니다. 일반적으로 이 방식은 아래의 함수처럼 입력된 파라미터가 구체적으로 어떤 타입인지 알 필요가 없을 때 많이 사용합니다.

```kotlin
fun printCars(cars: MutableList<*>) {
    for (car in cars) {
       println(car)
    }
}
```

<br>

반대로 구체적인 타입을 알아야 할 경우 (예컨데, 쓰기 작업이 필요한 경우)에는 스타 프로젝션을 사용할 수 없습니다. 스타 프로젝션을 사용하면 정확히 어떠한 타입의 파라미터가 입력되는지 파악할 수 없기 때문입니다. 예를 들어 아래의 코드를 보면 copyCars 함수 내부에서는 타입 파라미터로 임의의 MutableList가 입력된다는 정보만 알지 구체적으로 어떠한 타입의 MutableList가 입력되는지는 알 수 없기 때문에, 아래와 같은 컴파일 에러가 발생하게 됩니다. 

```kotlin
fun <T> copyCars(src: MutableList<*>, dst: MutableList<*>) {
    for (car in src) {
        dst.add(car)	//컴파일 에러
    }
}
```

![스크린샷1](../../images/star-projection-compile-error.png)