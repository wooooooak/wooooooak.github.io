---
layout: post
title: 언제 takeIf()와 takeUnless()를 쓸까?
category: Kotlin
tags: [kotlin, Kotlin Standard Library]
comments: true
---

`takeIf()`와 `takeUnless()`는 자주 쓰이는 함수는 아니지만 코드를 더 읽기 쉽게 하는 좋은 함수다. 그러나 내게는 종종 이 함수들의 잘 못된 예시 코드들이 보인다. 이 글에서는 이 함수들이 무엇이고, 어떻게 제대로 사용하는지에 대해 살펴볼 것이다.

## `takeIf()`와 `takeUnless()`의 기능

간단히 말하자면, `takeIf()`는 null이 아닌 객체에서 호출될 수 있고, predicate(Boolean을 리턴하는)함수를 인자로 받는다. 만약 predicate가 true를 반환하는 식이라면, `takeIf()`는 null아닌 그 객체를 리턴할 것이고, predicate가 만족되지 않아 false를 반환한다면 null을 리턴할 것이다.

아래의 함수를

```kotlin
return if(x.isValid()) x else null
```

아래처럼 사용할 수 있다.

```kotlin
return x.takeIf { it.isValid() }
```

`takeUnless()`는 반대다. predicate함수가 false를 반환하면 null이 아닌 객체를 리턴하고, true를 반환하면 null을 반환한다.

아래의 함수를

```kotlin
return if(!x.isError()) x else null
```

아래처럼 사용할 수 있다.

```kotlin
return x.takeUnless { it.isError }
```

위의 두 함수는 모두 Kotlin Standard Library 1.1 부터 존재하던 것이다. 코드 원형은 아래에 기재했지만, 실제로는 어노테이션이나 contract specification같은 것들이 더 존재한다. 이 글에서는 함수의 역할과 사용법에 집중하기 위해 간편화 했다.

```kotlin
// In Standard.kt

public inline fun <T> T.takeIf(predicate: (T) -> Boolean): T? {
    return if (predicate(this)) this else null
}

public inline fun <T> T.takeUnless(predicate: (T) -> Boolean): T? {
    return if (!predicate(this)) this else null
}
```

## `takeIf()`와 `takeUnless()`가 독이 되는 경우

겉으로보면 `if(someCondition) x else null` 이라는 코드는 `x.takeIf { someCondition }`으로 대체되고, `if(!someCondition) x else null`는 `x.takeUnless { someCondition }`으로 대체되는 것 같다. 하지만 여기에는 미세하게 차이가 있음을 알아야한다.

#### 차이점 1 : 연산의 순서

아래의 코드를 보자

```kotlin
return if(x.isValid()) doWorkWith(x) else null
```

위의 코드를 아래와 같이 쓰고싶을 것이다.

```kotlin
return doWorkWith(x).takeIf { x.isValid() }
```

그러나 바로위의 코드를 실행 할 경우 에러가 난다. 왜일까? 그 이유는 `doWorkWith(x)` 함수가 predicate 평가 시점 이전에 호출되기 때문이다. x가 유효한지 유효하지 않은지를 판단하기도 전에 `doWorkWith(x)`가 호출되는 셈이다. 만약 `doWorkWith()`가 오직 유효한 입력값에서만 동작하는 함수라면 에러가 난다.

#### 차이점 2 : 초과 연산

`if/else`절에서는 x가 유효하지 않을 경우 따로 연산을 하지 않는다. else절에 가지 않기 때문이다. 반면 `takeIf()`를 사용할 경우 코드는 항상 `doWorkWith(x)`를 호출하게 된다. 이것은 predicate가 false일 때 초자 불필요하게 연산을 한다. predicate가 어떤 에러없이 안전하게 호출되는 것은 좋지만, 굳이 필요하지 않을 때도 연산을 하는 것이다.

#### 차이점 3 : 부수 효과

초과 연산의 연장선으로 볼 수 있다. 원하던 원하지 않던 predicate가 실행되기 때문에 부수효과가 일어날 수 있는 것이다. 나는 함수형 프로그래밍 애찬론자는 아니기에 부수효과에 대해 왈가왈부하진 않겠다.
그러나 predicate가 로그를 찍는다고 생각해보자. predicate 함수로 인해 데이터를 생성하고, 원하지 않는 작업들이 수행될 수도 있다.

## `takeIf()`와 `takeUnless()`가 약이 되는 경우

바로 위에서 `takeIf()`와 `takeUnless()`가 독이 되는 경우를 보았다. 이제는 약이 되는 경우, 즉 사용하기에 적합한 경우를 살펴보자.

#### 예시 1 : 객체가 식이 아닐 때

`takeIf()`를 호출하는 것이 그저 값(객체)일 때 위에서 말한 3가지 문제를 피할 수 있다. 식에 대해서 `takeIf()`를 호출한 다면 에러를 유발하기 쉽다.

#### 예시 2 : predicate가 좀 복잡해서 읽기 힘들 때

간단한 예시를 보자. 아래의 함수를 더 좋게 바꿀 수 있다.

```kotlin
return if (x) y else null
```

이렇게 바꾸자

```kotlin
return y.takeIf { x }
```

이렇게 바꾸는 것이 더 읽기 쉽다는 것은 전적으로 내 주관이다. 나는 개인적으로 predicate가 복잡하면 복잡할수록 `takeIf()`와 `takeUnless()`가 더 빛을 본다고 생각한다.

아래의 코드를

```kotlin
return if (evensOnly && x % 2 == 0) x else null
```

아래처럼 바꿔보자

```kotlin
return x.takeIf { evensOnly && x % 2 == 0}
```

다시 말하지만 개인 취향이고 내 생각이다. 나는 `takeIf()`가 더 읽기 쉽다고 생각한다.

#### 예시 3 : 어떤 함수가 특정 객체의 조건부로 호출될 때

`if`문에서 많은 일을 한다고 가정해보자.

```kotlin
return if (someString.isNotBlank()) {
    someMoreWork(someString)
} else {
    null
}
```

위의 코드는 아래처럼 바뀔 수 있다.

```kotlin
return someString.takeIf { it.isNotBlank() }?.let { someMoreWork(it) }
```

다시 말하지만, `takeIf`가 더 읽기 쉽다는 것은 주관적인 생각이다.
