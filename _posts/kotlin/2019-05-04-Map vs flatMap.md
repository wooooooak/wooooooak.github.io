---
layout: post
title: Map과 FlatMap에 대하여
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

- flatMap도 Map처럼 결국은 배열(또는 iterable)을 리턴한다.
- 대상이 되는 배열의 요소가 3개라면, flatMap 내부적으로도 3번 호출된다(그러나 결과는 하나의 배열이다).
- map은 무조건 1대1 매핑이지만, flatMap은 1대1 뿐만 아니라 1대다 매핑이가능하다.
- flatMap에 넘겨주는 함수는 꼭 iterable한 값을 리턴해야한다(iterable한것을 평평히 펴주는 역할을 할 것임으로).
