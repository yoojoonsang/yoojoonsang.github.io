---
layout: post
title:  "Generics in Kotlin #2"
categories: jekyll update
description: 이 장에서는 타입 소거에 대해 알아봅니다.
---
자바 설계자는 소거(erasure)라는 기술을 사용해 제네릭을 구현.
제네릭 이전의 자바와 호환 가능하도록 하기 위해 소거를 도입했다고 함.
소거는 컴파일하는하는 동안 타입 매개변수를 삭제하는 프로세스.
이에 따라, 'fun print(list: List<String>)'과 'fun print(list: List<Int>)'는 소거 후 동일한 함수 시그니처를 갖게됨.
런타임에서는 객체를 인스턴스화 했을때, 어떤 타입 매개변수가 사용됐는지 확인 할 수 없게 됨.
타입 T의 인스턴스인지 테스트 할 수 없음


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
타입 구체화는 인라인 함수에 대해 런타임에서 타입 정보를 유지할 수 있게 해준다.
구체화 가능한 타입(reifialbe type)이란 런타임에서 타입 정보를 점검 받을 수 있는 타입에 붙여진 이름
구체화한 것으로 간주하는 타입의 예로는 String이나 BigDecimal 같은 비 제네릭 타입을 들 수 있다.
비구체화 타입(non-reifiable type)은 타입 정보의 일부 또는 전체가 런타임에 손실되도록 소거의 영향을 받은 타입을 말함.

코틀린에서는 타입 구체화(type reification)이라는 기능을 소개하는데, 타입 구체화는 인라인 함수에 대해 런타임에 타입 정보를 유지할 수 있게 해준다.
이러한 기능을 사용하려면, 타입 매개변수 앞에 reified 키워드를 추가해야 한다.

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
이게 어떻게 가능하냐면, 구체화한 함수가 inline으로 정으된다는 사실에 있다.
컴파일러는 호출하는 쪽에서 타입 매개 변수가 사용되는 것을 알기 때문에 T에 대한 참조를 적절한 타입에 대한 참조로 변경하는게 가능.

```kotlin
val someList = arrayListOf("Hello", 1234, BigDecimal(123456789), "World", BigDecimal(987654321))
val bigDecimalOnly = getElementsOfType<BigDecimal>(someList)
```

