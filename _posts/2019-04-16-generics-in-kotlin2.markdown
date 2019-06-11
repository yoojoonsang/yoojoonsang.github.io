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
    private Object key;		//K -> Object로 교체됨
    private Object value;	//V -> Object로 교체됨
    
    public Entry(Object key, Object value) {
        this.key = key;
        this.value = value;
    }
    
    public Object getKey() { return key; }
    public Object getValue() { return value; }
}
```

<br>

코틀린 또한 JVM을 위한 언어로 설계되었기 때문에 역시 소거의 개념이 존재합니다. 아래의 코드를 살펴보면, 일단 타입 파라미터 T에 대한 리스트를 생성하는 부분에서 에러가 나타납니다. 만약 아래의 코드가 컴파일 된다면 런타임 때 타입 매개변수에 대한 어떠한 정보도 제공 할 수 없기 때문입니다.

```kotlin
fun <T> getElementsOfType(list: List<Any>): List<T> {
    //타입 T에 대한 새 리스트를 생성한다.
    var newList = arrayListOf<T>()	

    // list에서 타입 T에 해당하는 원소를 모두 찾고, newList에 추가한다.
    for (element in list) {
        if (element is T) {		//에러 발생
            newList.add(element)
        }
    }

    return newList
}
```
![스크린샷1](../../images/type-erased.png)

<br>

실제로도 getElementsOfType 함수를 컴파일하고 다시 디컴파일 해보면 어느 곳에서도 타입 매개변수에 대한 단서를 찾아볼 수 없음을 확인할 수 있습니다. 

```java
public List getElementsOfType(List list) {
      ArrayList newList = new ArrayList();
      Iterator var3 = list.iterator();

      while(var3.hasNext()) {
         Object element = var3.next();
         if (element != null ? element instanceof Object : true) {
            newList.add(element);
         }
      }

      return (List)newList;
}
```

<br>

이러한 문제를 해결하기 위해 코틀린에서는 타입 구체화(type reification)라는 기능을 제공하는데, 이 기능은 **<u>런타임에 인라인 함수의 타입 정보 유지</u>**를 가능하게 해줍니다. 여기서 **구체화 가능한 타입**(reifialbe type)과 **비구체화 타입**(non-reifiable type)에 대한 개념을 알 필요가 있는데, 내용은 다음과 같습니다.

|        구분        |                             설명                             |                    예                    |
| :----------------: | :----------------------------------------------------------: | :--------------------------------------: |
| 구체화 가능한 타입 |         런타임에 타입 정보를 점검 받을 수 있는 타입          | 비 제네릭 타입 (String, BigDecimal, ...) |
|   비 구체화 타입   | 소거의 영향에 의해 타입 정보의 일부 또는 전체가 런타임에 손실되는 타입 |               제네릭 타입                |

<br>

결론부터 말하면 우리는 **비구체화된 타입을 구체화된 타입으로 바꾸어** 런타임에도 타입 정보를 유지할 수 있게 해주어야 합니다. 방법은 간단합니다. **함수를 인라인으로 선언**하고 타입 매개변수 앞에 **reified 키워드를 추가**해주기만 하면 이제 비구체화된 타입을 구체화된 타입으로 바꿀 수 있게 됩니다. 실행 또한 잘 됩니다.

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
fun main() {
    val someList = arrayListOf("Hello", 1234, BigDecimal(123456789), "World", BigDecimal(987654321))
    val bigDecimalOnly = getElementsOfType<BigDecimal>(someList)
    
    println(bigDecimalOnly)
}
```

<br>

다음은 위의 코드를 컴파일하고 디컴파일 한 결과입니다. instanceof의 피연산자로 타입 매개변수 T가 아닌, BigDecimal이 사용되는 걸 확인할 수 있습니다.

```java
public static final void main() {
      ArrayList someList = CollectionsKt.arrayListOf(new Object[]{"Hello", 1234, new BigDecimal(123456789), "World", new BigDecimal(987654321)});
      int $i$f$getElementsOfType = false;
      ArrayList newList$iv = new ArrayList();
      Iterator var4 = ((List)someList).iterator();

      while(var4.hasNext()) {
         Object element$iv = var4.next();
         if (element$iv instanceof BigDecimal) {
            newList$iv.add(element$iv);
         }
      }

      List bigDecimalOnly = (List)newList$iv;
      System.out.println(bigDecimalOnly);
}
```

<br>

한편 자바의 경우에는 아래와 같이 리스트의 타입을 확인하는게 불가능합니다. JVM은 stringList가 스트링의 리스트인지 이해할 수 없기 때문입니다. 

```java
boolean b = stringList instanceof List<String>;	//Illegal generic type for instanceof
```

<br>

그나마 stringList가 리스트인지 여부만 확인이 가능합니다.

```java
boolean b = stringList instanceof List;	
```

<br>

반면에 코틀린의 경우에는 언어차원에서 리스트의 구체적인 타입을 확인할 수 있는 기능을 지원 합니다. 

```kotlin
val stringList = arrayListOf("Hello")

if (stringList is List<String>) {
    println("This is a list of string")
}

>> This is a list of string
```

<br>

물론 이 경우에도 앞서와 마찬가지로 런타임에는 타입이 소거된 코드가 동작하지만 JVM은 이미 컴파일러가 타입 체킹을 완료했을 거라는 가정 하에 아래 코드를 실행하게 됩니다.

```java
ArrayList stringList = CollectionsKt.arrayListOf(new String[]{"Hello"});
if (stringList instanceof List) {
	String var1 = "This is a list of string";
	System.out.println(var1);
}
```

