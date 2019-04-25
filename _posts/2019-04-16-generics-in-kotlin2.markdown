---
layout: post
title:  "Generics in Kotlin #2"
categories: jekyll update
description: 이 장에서는 타입 소거에 대해 알아봅니다.
---
자바 설계자는 소거(erasure)라는 기술을 사용해 제네릭을 구현했습니다.
이때, 소거는 컴파일 시점에 타입 매개변수를 삭제하는 프로세스로, 제네릭 이전의 자바와 호환 가능하도록 하기 위해 도입했다고 합니다.
예컨데, 'fun print(list: List<String>)'과 'fun print(list: List<Int>)' 함수가 있다면, 소거 후 두 함수는 동일한 함수 시그니처('fun print(list: List)')를 갖게됩니다.
따라서 런타임에 어떤 타입 매개변수가 사용됐는지 확인 할 수 없게 됩니다.

```kotlin
fun <T> getElementsOfType(list: List<Any>): List<T> {
    var newList = arrayListOf<T>()

    for (element in list) {
        if (element is T) {
            newList.add(element)
        }
    }

    return newList
}
```
![스크린샷1](../../images/type-erased.png)

<br>
코틀린에서는 타입 구체화(type reification)이라는 기능을 소개하는데, 타입 구체화는 인라인 함수에 대해 런타임에 타입 정보를 유지할 수 있게 해줍니다.
먼저 구체화 가능한 타입(reifialbe type)이란 런타임에 타입 정보를 점검 받을 수 있는 타입에 붙여진 이름입니다.
예를들어 구체화한 것으로 간주하는 타입으로는 String이나 BigDecimal 같은 비 제네릭 타입을 들 수 있습니다.
반대로 비구체화 타입(non-reifiable type)은 타입 정보의 일부 또는 전체가 런타임에 손실되도록 소거의 영향을 받은 타입을 말합니다.

따라서 우리는 비구체화된 타입을 구체화된 타입으로 바꾸어 런타임에도 타입 정보를 유지할 수 있게 해주어야 합니다.
방법은 아주 간단합니다. 함수를 인라인으로 선언하고 타입 매개변수 앞에 reified 키워드 하나만 추가해주면 끝입니다.

```kotlin
inline fun <reified T> getElementsOfType(list: List<Any>): List<T> {
    var newList = arrayListOf<T>()

    for (element in list) {
        if (element is T) {
            newList.add(element)
        }
    }

    return newList
}
```

<br>
이게 어떻게 가능한걸까요?
힌트는 함수가 inline으로 정의된다는 사실에 있습니다.
함수가 inline으로 정의됨에 따라 함수 코드는 호출되는 쪽에 복사되고, 컴파일러는 호출하는 쪽에서 타입 매개 변수가 사용되는 것을 알게되는데, 
이 때문에 컴파일러는 타입 매개변수 T에 대한 참조를 적절한 타입에 대한 참조로 변경하는게 가능해집니다.

```kotlin
val someList = arrayListOf("Hello", 1234, BigDecimal(123456789), "World", BigDecimal(987654321))
val bigDecimalOnly = getElementsOfType<BigDecimal>(someList)
```

