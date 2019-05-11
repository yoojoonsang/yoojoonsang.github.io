---
layout: post
title:  "Generics in Kotlin #2"
categories: jekyll update
description: 이 장에서는 타입 소거에 대해 알아봅니다.
---
자바 설계자는 소거(erasure)라는 기술을 사용해 제네릭을 구현했습니다. 소거는 컴파일 시점에 **<u>타입 매개변수를 삭제</u>**하는 프로세스로 제네릭과 제네릭 이전의 자바를 호환하고자 도입했다고 합니다. 예를 들어, 아래와 같은 제네릭 클래스가 있다고 해봅시다.

```java
public class Entry<K, V> {
    private K key;
    private V value;
    
    public Entry(K key, V value) {
        this.key = key;
        this.value = value;
    }
    
    public K getKey() { return key; }
    public V getValue() { return value; }
}
```

<br>

위의 제네릭 클래스는 소거(컴파일)가 완료되면 다음과 같이 제네릭 타입 **K**와 **V**가 **Object** 타입으로 교체된 클래스로 변환이 됩니다.

```java
public class Entry {
    private Object key;
    private Object value;
    
    public Entry(Object key, Object value) {
        this.key = key;
        this.value = value;
    }
    
    public Object getKey() { return key; }
    public Object getValue() { return value; }
}
```

<br>

코틀린 또한 JVM을 위한 언어로 설계되었기 때문에 역시 소거의 개념이 존재합니다. 게다가 코틀린은 타입에 대해 보다 더 엄격한 언어이기 때문에, 소거에 민감합니다. 간단한 예를 하나 살펴보겠습니다. 아래의 코드는 아무 문제 없이 동작할 것 같지만 컴파일 에러를 발생시킵니다. 런타임에 타입 T에 대한 어떠한 타입 정보도 제공 할 수 없기 때문입니다.

```kotlin
fun <T> getElementsOfType(list: List<Any>): List<T> {
    //타입 T에 대한 새 리스트를 생성한다.
    var newList = arrayListOf<T>()

    // list에서 타입 T에 해당하는 원소를 모두 찾고, newList에 추가한다.
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

이러한 문제를 해결하기 위해 코틀린에서는 타입 구체화(type reification)라는 기능을 제공하는데, 이 기능은 **<u>런타임에 인라인 함수의 타입 정보 유지</u>**를 가능하게 해줍니다. 여기서 **구체화 가능한 타입**(reifialbe type)과 **비구체화 타입**(non-reifiable type)에 대한 개념을 알 필요가 있는데, 내용은 다음과 같습니다.

|        구분        |                             설명                             |                    예                    |
| :----------------: | :----------------------------------------------------------: | :--------------------------------------: |
| 구체화 가능한 타입 |         런타임에 타입 정보를 점검 받을 수 있는 타입          | 비 제네릭 타입 (String, BigDecimal, ...) |
|   비 구체화 타입   | 소거의 영향에 의해 타입 정보의 일부 또는 전체가 런타임에 손실되도록  받은 타입 |               제네릭 타입                |

<br>

따라서 우리는 비구체화된 타입을 구체화된 타입으로 바꾸어 런타임에도 타입 정보를 유지할 수 있게 해주어야 합니다.
방법은 간단합니다. 함수를 인라인으로 선언하고 타입 매개변수 앞에 reified 키워드 하나만 추가해주면 컴파일러가 알아서 타입 정보를 채워주게 됩니다.

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

이게 어떻게 가능한걸까요? 힌트는 함수가 inline으로 정의된다는 사실에 있습니다. 함수가 inline으로 정의됨에 따라 함수 코드는 호출되는 쪽에 복사되는데, 이에 따라 컴파일러는 타입 매개변수 T에 대한 참조를 적절한 타입에 대한 참조로 변경할 수 있게 되고, 아래 코드는 아무런 문제 없이 동작할 수 있게 됩니다.

```kotlin
val someList = arrayListOf("Hello", 1234, BigDecimal(123456789), "World", BigDecimal(987654321))
val bigDecimalOnly = getElementsOfType<BigDecimal>(someList)
```

