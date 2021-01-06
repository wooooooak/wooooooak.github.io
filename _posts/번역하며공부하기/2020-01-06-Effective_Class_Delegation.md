---
layout: post
title: Effective Class Delegation(번역)
category: 번역하며 공부하기
tags: [delegation, kotlin, OOP]
comments: true
---

원본 - [Effective Class Delegation](https://zsmb.co/effective-class-delegation/)

[Effective Java book](https://www.amazon.com/Effective-Java-Joshua-Bloch-ebook/dp/B078H61SCH) 의 '**Item 18: 상속보다는 합성을 사용하라**'는 가장 인상 깊은 내용 중 하나입니다. 간단히 말하자면 아래와 같습니다.

> 상속은 재사용 하고 싶은 코드를 이미 가지고 있는 클래스를 상속함으로써 코드를 재사용하는 인기있는 방법이다. 그러나 이 방법은 에러를 유발하기 쉽다. 하위 클래스가 상위 클래스의 자세한 구현들에 의존하게 되어 캡슐화를 위반하기 때문이다.

## Problem statement

아래에 책에 나오는 예제를 그대로 가져왔습니다.

```kotlin
// 잘못된 예 - 상속을 잘못 사용했다!
public class InstrumentedHashSet<E> extends HashSet<E> {
    // 추가된 원소의 수
    int addCount = 0;

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }
}
```

이 `HashSet`의 하위 클래스는 더해진 원소의 갯수를 추적하는 클래스입니다만, 제대로 동작하지 않습니다. 그 이유는 상위 클래스의 `addAll` 함수 내부에서 `add` 함수를 사용하기 때문입니다. `addAll`에서 count를 증가시키고, `add`에서 또 다시 count를 증가시키게되어 중복으로 counting하게 됩니다.

```kotlin
val set = InstrumentedHashSet<Int>()
set.addAll(listOf(1, 2, 3, 4, 5))
println(set.addCount) // 10
```

이를 해결하기 위해 단순히 override한 `addAll` 함수를 지우면 되지 않을까라고 생각할 수 있습니다. 그렇게 하면 동작은 하겠지만, 상위 클래스의 `addAll` 함수의 구현이 언제 또 바뀔지 모른다는 점을 고려하면 좋지 않은 방식이겠죠. 또는 count를 했는지를 체크하는 flag를 두는 방식을 고려할 수도 있지만, 생각만 해도 복잡하네요.

## Tha Java solution

그럼 어떻게 해야할까요? 책에서 추천하는 방법은 **상속이 아니라 합성**을 사용하는 것입니다. 클래스를 새로 하나 만들고, `HashSet`을 상속하는게 아니라 `HashSet`객체를 가지고 있는 방법이죠.

```kotlin
public class InstrumentedSet<E> implements Set<E> {
    int addCount = 0;

    private final Set<E> set;

    public InstrumentedSet(Set<E> set) { this.set = set; }

    public boolean add(E e) {
        addCount++;
        return set.add(e);
    }

    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return set.addAll(c);
    }

    // ...
}
```

이렇게 바꾸면 사용하는 측에서 `Set()`을 넘겨주어야 합니다.

```kotlin
val set = InstrumentedSet<Int>(HashSet())
set.addAll(listOf(1, 2, 3, 4, 5))
```

좋습니다. 하지만 이제는 새로운 문제가 있어요. 우리가 `Set` 의 모든 인터페이스들을 다 구현해야한다는 것이죠. `add`와 `addAll`을 구현했지만, `Set`인터페이스는 12개가 더 넘는 메서드들을 요구합니다🤣. 우리는 이 현상을, 껍데기 메서드가 호출되면 가지고 있는 `set` 객체의 메서드를 그대로 다시 호출하는 [biolerplate](https://en.wikipedia.org/wiki/Boilerplate_code)라고 부르죠.

Effective Java에서는 이를 해결하기 위해서 중간에 `ForwardingSet`클래스를 추가하는 방법을 소개합니다. 이 클래스는 `InstrumentedSet`가 상속받을 수 있구요, 우리가 필요한 딱 두 개의 메서드만 override하면 되도록 나머지 함수들을 구현해 놓았습니다.

```kotlin
public class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;
    public ForwardingSet(Set<E> s) { this.s = s; }
    public void clear() { s.clear(); }
    public boolean contains(Object o) { return s.contains(o); }
    public boolean isEmpty() { return s.isEmpty(); }
    /* Lots of more methods... */
}

public class InstrumentedSet<E> extends ForwardingSet<E> {
    int addCount = 0;

    public InstrumentedSet(Set<E> s) { super(s); }

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }
}
```

아마 이 방법이 Java에서는 최선이 아닐까 생각합니다.

## Going Kotlin

그럼 이제는 Kotlin으로 `InstrumentedSet` 를 구현해볼까요? 클래스 위임을 통해 구현해봅시다.
클래스 위임이란 다른 특정 객체에게 interface를 구현하도록 위임하는 것입니다.
네 맞습니다. 바로 위에서 Java코드로 직접 구현한 것과 같은 일입니다.

```kotlin
class InstrumentedSet<E>(
	private val set: MutableSet<E>
) : MutableSet<E> by set
```

(`java.util.Set` 인터페이스와 동일한 코틀린의 `MutableSet`을 사용합니다.)

이렇게 하여 `InstrumentedSet`는 `set` 프로퍼티를 통해 `MutableSet` 인터페이스를 구현했습니다. `InstrumentedSet`의 메서드가 호출될 때면 간단하게도 `set` 객체의 동일한 메서드가 그대로 호출됩니다. 자바로 만든 `FowardingSet`를 단 한 줄로 구현한셈이죠. 이 코드를 자바로 디컴파일해서 살펴볼까요.

```kotlin
public final class InstrumentedSet implements Set {
   private final Set set;

   public InstrumentedSet(@NotNull Set set) {
      this.set = set;
   }

   public int getSize() {
      return this.set.size();
   }

   public boolean add(Object element) {
      return this.set.add(element);
   }

   public void clear() {
      this.set.clear();
   }

   // ...
}
```

이제 우리가 할 일은 `add`와 `addAll`메서드를 추가된 요소의 갯수를 카운팅 하도록 수정하는 일입니다.

```kotlin
class InstrumentedSet<E>(
        private val set: MutableSet<E> = HashSet()
) : MutableSet<E> by set {
    var addCount = 0

    override fun add(element: E): Boolean {
        addCount++
        return set.add(element)
    }

    override fun addAll(elements: Collection<E>): Boolean {
        addCount += elements.size
        return set.addAll(elements)
    }
}
```

`set` 파라미터의 기본값을 `HashSet`으로 만들었습니다. Client는 필요하다면 `MutableSet` 구현체를 넘길 수도 있겠지만 굳이 그럴 필요가 없기 때문에 편의상 default value를 설정했습니다.

이게 끝입니다. 우리가 직접 구현하지 않은 `MutableSet` 메서드들은 모두 `set`객체로 위임되었습니다. Java코드와 마찬가지로 코드는 예상처럼 잘 동작합니다.

```kotlin
val set = InstrumentedSet<Int>()
set.addAll(listOf(1, 2, 3, 4, 5))
println(set.addCount) // 5
```

## 마무리

흥미로운 점이 하나 있는데요, 위임에 의한 구현이라 칭해지는 이 feature가 여러 행사에서 Kotlin lead designer인 Andrey Breslav로 하여금 "worst" feature라고 불렸다고하네요([KotlinConf 2018 closing panel discussion](https://www.youtube.com/watch?t=646&v=heqjfkS4z2I&feature=youtu.be)). 몇몇 상황에서는 이런 위임이 상황을 더 복잡하게 만들기도 한다네요. 하지만 간단한 상황에서는 많은 boilerplate코드를 제거할 수 있습니다.
