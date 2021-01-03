---
layout: post
title: Kotlin Inline class
category: Kotlin
tags: [inline_class]
comments: true
---

비즈니스 로직을 작성하다 보면 어떤 타입의 Wrapper를 작성할 때가 있다. 예를 들어 userName을 표현할 때 단순히 String으로 나타낼 수 있겠지만, 좀 더 도메인적인 의미를 나타내 주고 싶을때는 UserName 타입으로 표현할 수 있다.

```kotlin
// primitive type만 사용할 경우
data class person(
	val userName: String,
	val password: String,
)

// 도메인에 특화된 타입을 만들어서 사용하는 경우
data class person(
	val userName: UserName,
	val password: Password,
)
data class UserName(val value: String)
data class Password(val value: String)
```

그러나 이 Wrapper는 추가적인 Heap영역에 할당되므로 런타임 오버헤드가 발생한다. 특히 Wrapping의 대상이 되는 타입이 primitive라면 런타임 성능에 더 악영향을 미치는데, primitive 타입은 보통 런타임에 굉장이 최적화되어 있는 반면 primitive를 wrapping하는 순간 더이상 그 최적화가 의미 없어지기 때문이다.

이 문제를 해결하기 위해 코틀린에서는 `inline class` 를 제공한다. inline class는 생성자로 단 하나의 값만 받을 수 있다. 물론 클래스 내에 프로퍼티와 함수를 정의할 수도 있다.

```kotlin
inline class UserName(val value: String)
inline class Password(val value: String) {
	fun isValid() = value.isNotEmpty()
}
```

inline class 객체는 런타임에 wrapper로써 표현될 수도 있고 wrapping되기 전의 타입으로 표현될 수도 있다. 마치 `Int`클래스가 primitive type인 `int` 로 표현되기도 하고 Wrapper인 `Integer` 클래스로 표현되기도 하는 것과 같다.

코틀린 컴파일러는 성능상의 이유로 wrapper보단 wrapping되기 전의 타입을 더 선호한다. 그러나 때로는 wrapper를 유지해야 할 때가 있기 마련이다. 보통 인라인 클래스를 선언해놓되 다른 타입으로 사용될 때 Wrapper로 유지되곤 한다. 공식문서의 코드를 보면 이해가 수월하다.

```kotlin
interface I

inline class Foo(val i: Int) : I

fun asInline(f: Foo) {}
fun <T> asGeneric(x: T) {}
fun asInterface(i: I) {}
fun asNullable(i: Foo?) {}

fun <T> id(x: T): T = x

fun main() {
    val f = Foo(42)

    asInline(f)    // unboxed: used as Foo itself
    asGeneric(f)   // boxed: used as generic type T
    asInterface(f) // boxed: used as type I
    asNullable(f)  // boxed: used as Foo?, which is different from Foo

    // below, 'f' first is boxed (while being passed to 'id') and then unboxed (when returned from 'id')
    // In the end, 'c' contains unboxed representation (just '42'), as 'f'
    val c = id(f)
}
```

## Inline classes vs type aliases

Inline class와 Type aliase는 타입에 대해 새로운 이름을 부여하고, 런타임에 원래 타입으로 사용된다는 공통점이 있다.

그러나 차이는 존재한다. type aliase는 기존 타입과 Aliase 타입의 호환성이 보장되는 반면 Inline class는 기존 타입과 Inline class를 명확히 구분짓기 때문에 호환되지 않는다. 즉 inline class는 완전히 새로운 type을 만드는 거라고 볼 수 있고, type aliase는 그저 원래 있던 타입에 별칭을 붙여주는 것 뿐이다.

공식문서에 나와있는 예제를 보면 이해하기 수월하다.

```kotlin
typealias NameTypeAlias = String
inline class NameInlineClass(val s: String)

fun acceptString(s: String) {}
fun acceptNameTypeAlias(n: NameTypeAlias) {}
fun acceptNameInlineClass(p: NameInlineClass) {}

fun main() {
    val nameAlias: NameTypeAlias = ""
    val nameInlineClass: NameInlineClass = NameInlineClass("")
    val string: String = ""

    acceptString(nameAlias) // OK: alias 타입을 기존 타입에 넘겨주어도 된다.
    acceptString(nameInlineClass) // Not OK: Inline class를 String타입으로 넣을 수 없다. NameInlineClass 타입은 String타입이 아니기 때문.

    // And vice versa:
    acceptNameTypeAlias(string) // OK: String타입을 Alias타입으로 넣어도 된다. NameTypeAlias는 단지 String의 다른 이름일 뿐이기 때문이다.
    acceptNameInlineClass(string) // Not OK: String과 NameInlineClass는 다른 타입이기 때문에 불가능하다.
}
```

## 참고자료

- [Kotlin Inline class 공식문서](https://kotlinlang.org/docs/reference/inline-classes.html)
