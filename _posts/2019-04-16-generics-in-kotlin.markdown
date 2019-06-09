---
layout: post
title:  "Generics in Kotlin #1"
categories: jekyll update
---
자바에서는 어떤 타입을 저장해야 할지 알려주지 않고도 List를 선언하는게 가능합니다.
여기에는 문자열을 저장할 수도 있고, 큰 수를 저장할 수도 있습니다. 

```java
List list = new ArrayList<>();
list.add("Hello");
list.add(new BigDecimal(10.5));
```
<br>
하지만 코틀린은 아래와 같은 코드를 허용하지 않습니다. 
일단 타입이 없으면 컴파일부터 되지 않기 때문입니다.  

참고로 자바는 1.5버전부터 제네릭을 도입했기 때문에 이전 버전과 호환성을 유지하기 위해 타입이 명시되지 않은 제네릭 타입을 허용해준다고 합니다. 

```kotlin
val list = arrayListOf<>() //compile error
```
![스크린샷1](../../images/list-without-type.png)

<br>
타입은 꺾쇠(<>) 사이에 명시해주거나, 컴파일러가 타입을 추론할 수 있도록 선언과 동시에 초기화를 해주는 방식으로 전달해 줄 수 있습니다.

```kotlin
val list1 = arrayListOf<String>()
val list2 = arrayListOf("Hello", "World")
val list3 = arrayListOf(BigDecimal(123456789), BigDecimal(987654321))
```

<br>
함수의 파라미터도 타입을 가지므로 타입 정보가 필요합니다.
아래 함수들은 각각 String과 BigDecimal 타입의 원소를 갖는 리스트를 입력받아 리스트의 원소들을 하나씩 출력해주는 함수입니다. 

```kotlin
fun printList(list: List<String>) {
   for (item in list) {
       println(item)
   }
}

fun printList(list: List<BigDecimal>) {
   for (item in list) {
       println(item)
   }
}
```

<br>
String과 BigDecimal 뿐만 아니라 더 많은 타입을 다뤄야 할 때에는 위의 코드를 복사하고 타입만 바꿔 사용할 수도 있습니다.
하지만 특정 타입 외에도 모든 타입을 다루고 싶다면 제네릭 함수를 사용하는 방법 또한 선택해 볼 수 있습니다. 
방법은 간단합니다.
함수 이름 앞에 타입 파라미터를 추가하고 리스트의 원소 타입을 똑같이 바꿔주면 모든 타입의 리스트를 파라미터로 받을 수 있게 됩니다.

```kotlin
fun <T> printList(list: List<T>) {
   for (item in list) {
       println(item)
   }
}
```

<br>
한편, 파라미터로 입력된 리스트가 printList 함수를 직접 호출하도록 바꿔줄 수도 있습니다.
이러한 개념을 extension function이라 부르는데, 아래의 예에서는 printList 함수가 list의 extension function이 됩니다.

```kotlin
fun <T> List<T>.printList() {
    for (item in this) {
        println(item)
    }
}
```
![스크린샷 2](../../images/extension-function.png)

<br>
때로는 타입 파라미터를 제한해야할 필요도 있습니다.
예를 들어, 아래와 같이 리스트의 원소를 Int로 변환하고 그 결과를 출력한다고 해봅시다.
하지만 컴파일러는 이 코드를 허용하지 않습니다.
파라미터로 입력될 리스트가 언제나 숫자 타입의 요소로만 구성될 것임을 보장할 수 없기 때문입니다.

```kotlin
fun <T> convertToInt(list: List<T>) {
    for (num in list) {
        println("${num.toInt()}")   //compile error
    }
}
```
![스크린샷 3](../../images/no-restrict-generic-type.png)

<br>
이 문제는 타입 파라미터를 Number 클래스로 한정하여 해결할 수 있습니다.
방법은 간단합니다.
아래 코드와 같이 타입 파라미터 뒤에 세미 콜론과 Number 클래스를 명시해주기만 하면 됩니다.
Number 클래스는 모든 numeric type의 부모이며 이미 toInt 함수를 선언해두고 있기 때문에 아래 코드는 문제 없이 동작하게 됩니다.
참고로 아무것도 명시하지 않았을 때의 상한(Upper Bound)는 Any?(Nullable Any)입니다. 

```kotlin
fun <T: Number> convertToInt(list: List<T>) {
    for (num in list) {
        println("${num.toInt()}")   
    }
}
```

<br>
상한을 여러 개 설정하고 싶을 때도 있습니다.
예를들어, 파라미터로 CharSequence와 Appendable을 모두 구현한 타입을 입력받고 싶다고 가정해 봅시다.
이 경우 아래와 같이 상한 설정을 타입 매개변수 바깥으로 빼내고 where 절로 분리하는 방법을 선택할 수 있습니다.
이제 where 절에 명시된 모든 조건을 구현한 타입만 파라미터로 사용가능하게 됩니다.

```kotlin
fun <T> append(item1: T, item2: T) 
        where T: CharSequence, T: Appendable {
    println("Result is ${item1.append(item2)}")
}
```


