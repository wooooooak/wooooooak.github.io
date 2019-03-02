---
layout: post
title: 람다식에서 return문 사용하기 (정확하게 알고 쓰자)
category: Kotlin
tags: [lambda, kotlin, scope]
comments: true
---

코틀린은 함수형 프로그래밍을 지원하기 때문에 익명함수, 람다식을 사용할 경우가 흔하다. map, filter, reduce, forEach등 함수를 인자로 받는 함수들(고차함수)의 매개변수로 우리만의 로직이 담긴 함수를 넘길때 람다를 자주 사용한다. 자주 사용하는 만큼 꼭 집고 넘어가야할 사항이 있다.

## 일반 함수를 넘겼을 때 (return에 주목하자)
일반적으로 함수 내부의 return은 그 함수만 종료시킨다. 물론 코틀린의 label을 사용해서 종료시킬 함수를 지정할 수 있지만 그렇지 않은 경우 return문을 포함하는 가장 가까운 함수를 종료시키게 된다. 우선 람다식이 아니라 일반적인 익명 함수를 넣은 예시를 보자.
```kotlin
fun exampleFunc() {
    var ints = listOf(0,1,2,3)  
    ints.forEach(
        fun(value: Int) {
            if (value == 0) return
            print("value ")
        }
    }
}

// 1 2 3
```
0 1 2 3 이 담긴 배열에서 forEach를 돌며 0이 아닌 요소만 출력하는 함수다. 우선은 `forEach`의 인자로 이름 없는 함수를 넘겨주었다. 이 경우는 if문에서 `value == 0`이 걸리게 되면 우리가 넘긴 이름 없는 함수만 종료된다. **이것은 맨 처음 말했듯이 일반적인 함수로써 함수 내부의 return은 그 함수만 종료시킨다는 원칙에 맞는 현상이다.** 따라서 결과값은 1 2 3 이 출력된다.

## 람다식을 넘겼을 때
```kotlin
fun exampleFunc2() {
    var ints = listOf(0,1,2,3)
    ints.forEach {
        if (it == 0) return
        print(it)
    }  
}
// 아무것도 출력되지 않음
```
`forEach`의 인자로 람다식을 넘기니 결과값이 달라진다. 왜 그럴까? **람다식은 자기 자신의 block 범위를 가지지 않기 때문이다. 즉 람다식 내부의 context는 자신을 감싸고 있는 외부 block인 것이다.** javascript의 화살표함수와 비슷한 성향을 가지고 있다(화살표 함수 내부의 this는 자신을 감싸는 context를 가리킴). 따라서 위의 코드의 경우, 람다식 내부의 return은 자기 자신을 종료시키는 것이 아니라 exampleFunc2를 종료시킨다.

## 나는 람다식에 대해서만 return 하고 싶은데?
람다식 자체만 return으로 끝내고 싶다면(아마 거의 대부분이 그런 상황일 것이다) 방법이 있다. kotlin의 label문법을 사용하면 된다. 만약 label이 없었다면 코틀린에서 람다의 사용성은 상당히 줄어들었을 것이라 생각한다. 코드를 보자.

```kotlin
fun exampleFunc3() {
    var ints = listOf(0,1,2,3)
    ints.forEach label@ {
        if (it == 0) return@label
        print(it)
    }
}
// 1 2 3
```
람다식에서 return문을 만나게 되면, 원래는 자신의 context는 `exampleFunc3`이기 때문에 `exampleFunc3`이 종료되야 한다. 하지만 label을 return하도록 코딩했으므로 해당 라벨이 가리키는 자기 자신(람다)만 종료된다.

## 매번 label을 사용해야 해?
이런식으로 람다를 사용할 일이 굉장히 많은데 그때마다 label을 사용해야 할까? 꼭 그렇지는 않다. 물론 완전히 안쓸수는 없지만 약간의 편법(?)이 존재한다. 

암시적 label이란 것을 사용하면 label을 줄일 수 있다. 람다식을 사용할 때 **암시적 label**은 자동으로 람다가 사용된 함수의 이름이 된다. 예를 들어 `forEach()`가 람다를 사용했다면, 그 람다의 암시적 label 이름은 "forEach"인 것이다. 코드를 보자.
```kotlin
fun exampleFunc4() {
    var ints = listOf(0,1,2,3)
    ints.forEach {
        if (it == 0) return@forEach
        print(it)
    }
}
// 1 2 3
```
위 코드처럼 람다식 첫 부분에 label을 따로 명시하지 않아도 된다. 암시적 label이 forEach를 가르키기 때문이다. 물론 `return`문에 아직도 @라벨이 나타나긴 하지만 하나라도 더 줄인게 어디야 ... 

만약 ints.forEach가 아니라 ints.map을 사용할 때면 람다식 내부의 return문에 return@map 과 같이 선언해주면 된다.

## 예제는 예제일 뿐
코드를 보면서 답답해 하셨던 분들이 계실거라 생각한다. 왜냐하면 위의 코드는 굳이 라벨링 필요 없이 아래와 같이 짜도 되기 때문이다.
```kotlin
fun exampleFunc5() {
    println(listOf(1,2,3,4,5).filter { it > 5 })
}
```
예제는 예제일 뿐 방법은 다양하다.