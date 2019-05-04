---
layout: post
title: map과 flatMap에 대하여
category: Kotlin
tags: [kotlin, FP, basic]
comments: true
---

FlatMap을 이해하기 위해 Map과 FlatMap의 차이점을 알아보자.

## Map

map은 이해하기 쉽다. 반복 가능한 배열을 대상으로, 배열의 각 요소를 하나하나 순회하며 그 요소를 조작하고, 조작한 요소들이 모여있는 **배열을 리턴**해 주는 것이다.

즉 ["A", "B", "C"]라는 배열이 있을 때, 각 요소마다 느낌표(!)를 붙인 배열을 만들고 싶을 경우 아래와 같이 코드를 작성하면 된다.

![map1](https://user-images.githubusercontent.com/18481078/57175175-c700cb80-6e83-11e9-926b-7b18cf1d7808.png)

map이라는 함수이름은 직관적이다. 흔히 1대1 매핑시킨다 라는 말을 할때 적용되는 그 mapping이다.

## FlatMap

이제 FlatMap을 알아보자.

헷갈릴 수 있기 때문에 몇 가지만 미리 인지하고 가자.

- flatMap도 map처럼 결국은 배열(또는 iterable)을 리턴한다.
- 대상이 되는 배열의 요소가 3개라면, flatMap 내부적으로도 3번 호출된다(그러나 결과는 하나의 배열 또는 이터러블 또는 옵저버블이다).
- map은 무조건 1대1 매핑이지만, flatMap은 1대1 뿐만 아니라 1대다 매핑이가능하다.
- flatMap에 넘겨주는 함수는 꼭 **iterable한 값을 리턴해야한다**(iterable을 평평히 펴주는 역할을 할 것임으로).

### flatMap이란?

flatMap은 먼저 매핑 함수를 사용해 각 엘리먼트에대해 map을 수행 후, 결과를 새로운 배열로 평평화한다.

### map vs flatMap

![flatMap1](https://user-images.githubusercontent.com/18481078/57182179-571c3080-6ed7-11e9-97c1-5af52c2d4cc4.png)
flatMap에 넘겨주는 람다는 꼭 iterable(반복가능)한 값이여야 한다. `newList2`를 보면 flatMap의 인자로 넘겨주는 람다의 리턴값에 `toList()`를 사용했다. `toList()`는 `"HI"`를 `[H, I]`와 같이 반복 가능한 List로 만들어 주는 역할을 한다.

flatMap을 사용한 `newList2`를 보자. `testList`의 각 요소(it)들이 `"$it!".toList()`로 인해 각자 배열로 mapping(문자열 A는 배열 [A, !]로 mapping됨)되고, 리턴해준 각각의 배열이 모두 펼쳐져지고 합해져 `[A, !, B, !, C, !]`가 된다.

결과적으로 1대1 매핑이아닌 1대2 매핑이 된것이다.

### flatMap 파헤치기

코틀린은 flatMap을 어떻게 구현했는지 살펴보자.

```kotlin
/**
 * Returns a single list of all elements yielded from results of [transform] function being invoked on each element of original collection.
 */

public inline fun <T, R> Iterable<T>.flatMap(transform: (T) -> Iterable<R>): List<R> {
    // 첫 번째 인자로 빈 배열 하나를 넘겨준다.
    // 이 배열안에 값들을 넣어줄 것이며
    // 이는 곧 flatMap의 결과물이 될 것이다.
    return flatMapTo(ArrayList<R>(), transform)
}

/**
 * Appends all elements yielded from results of [transform] function being invoked on each element of original collection, to the given [destination].
 */
public inline fun <T, R, C : MutableCollection<in R>> Iterable<T>.flatMapTo(destination: C, transform: (T) -> Iterable<R>): C {
    for (element in this) {  // iterable한 객체의 요소들을 순회
        val list = transform(element) // 요소들을 람다에 넣어 호출하고 값을 반환
        destination.addAll(list)
    }
    return destination
}
```

우리가 flatMap 함수를 호출함과 동시에 내부적으로 `destination`이라는 빈 배열이 하나 자동으로 만들어 진다. `flatMapTo`의 `this`는 Iterable<T> 타입이기 때문에 순회가 가능하며, for문으로 순회를 시작한다. 순회를 돌면서 각 요소에다가 우리가 flatMap을 호출하며 함께 넘겨준 람다함수를 적용한다. 람다가 무엇이었는지 상기하기 위해 위에 올려놓은 사진을 다시 보자.

![flatMap1](https://user-images.githubusercontent.com/18481078/57182179-571c3080-6ed7-11e9-97c1-5af52c2d4cc4.png)

여기서 `flatMap`에 넘겨준 람다함수는 9번째 줄의 코드다. 매개변수가 생략되었으나 실은 String 타입의 `it`을 입력 받고 거기에 느낌표(!)를 붙여서 `toList()`를 적용하여 배열을 리턴해 주는 람다이다.

다시 돌아와서, iterable한 객체의 요소들을 순회하며 각각 람다를 적용해 주었기 때문에 `flatMapTo` 함수 정의의 `list`변수는 리스트 타입이다. 그 리스트의 내용물들을 처음에 자동으로 만들어진 `destination` 배열에다가 넣어준다.

여기서 `destination`이 `add`를 호출하는 것이 아니라 `addAll`을 호출하는 것에 주의해야 한다. `addAll`을 사용함으로써 배열 자체를 넣어주는 것이 아니라, 배열의 내용물을 넣어줄 수 있게 된다. 아래는 코틀린 공식 문서의 내용이다.

![image](https://user-images.githubusercontent.com/18481078/57182749-d9a7ee80-6edd-11e9-92c1-27bb76242769.png)
