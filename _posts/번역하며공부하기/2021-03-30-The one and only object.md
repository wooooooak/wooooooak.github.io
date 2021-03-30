---
layout: post
title: The one and only object(번역)
category: 번역하며 공부하기
tags: [Kotlin, Object]
comments: true
---

원본 - [The one and only object](https://medium.com/androiddevelopers/the-one-and-only-object-5dfd2cf7ab9b)

자바에서 `static` 키워드는 인스턴스화된 객체가 아닌, object와 관련된 메서드와 프로퍼티를 나타낼 때 쓰입니다. 그리고 굉장히 널리 쓰이는 디자인 패턴인 Singleton을 생성할 때도 쓰입니다. 싱글톤은 다른 여러 객체들에서 접근할 수 있는 오직 하나의 객체를 생성하는 것입니다.

코틀린은 싱글톤 패턴을 구현하기 위해 더 우아한 방법을 제공합니다. 바로 object 키워드입니다. Java와 Kotlin 이 Singleton을 구현하는 방법에 어떤 차이가 있는지, Kotlin에서 static 키워드를 사용하지 않고 어떻게 Singleton을 만드는지(먼저 말씀드리면 object 키워드를 사용합니다), 또 내부적으로 어떻게 동작하는지 궁금하시다면 끝까지 읽어주세요.

우선, 잠시 뒤로 물러나서 오직 단 하나의 객체인 Singleton이 왜 필요한지를 알아봅시다.

## Singleton이란?

Singleton은 클래스가 단일 객체를 가지고 있고, 그 객체에 대해 global 접근이 가능한 패턴을 말합니다. 그래서 앱의 여러 부분에서 공유해서 사용할 객체이거나 객체를 생성할 때 많은 resource가 드는 경우에 유용하게 사용됩니다.

## Singleton In Java

클래스가 단일 객체를 가지고 있음을 보장하려면 object를 생성에 직접 관여를 해야합니다. 단일 객체를 가진 클래스를 만드려면, 생성자를 `private`으로 하고 그 object 접근에 대해 `static`하게 접근 할 수 있도록 만들어야 합니다. 한편 보통 싱글톤은 객체 생성 비용이 많이 들기 때문에 앱이 시작할 때 Singleton을 만들고 싶지는 않을 겁니다. 이런 상황을 위해 이미 생성된 객체가 있는지를 확인하는 static 메서드를 제공해줍니다. 이 메서드는 이미 생성된 instance가 있으면 그것을 반환하고, 없다면 새로 생성하여 반환합니다.

```kotlin
<!-- Copyright 2019 Google LLC.
SPDX-License-Identifier: Apache-2.0 -->

public class Singleton{
    private static Singleton INSTANCE;
    private Singleton(){}
    public static Singleton getInstance(){
        if (INSTANCE == null){
            INSTANCE = new Singleton();
        }
        return INSTANCE;
    }
    private int count = 0;
    public int count(){ return count++; }
}
```

위 코드는 겉보기엔 괜찮아 보일지 몰라도 thread-safe하지 못합니다. 하나의 쓰레드가 if문을 통과하였지만, 또다른 쓰레드가 싱글톤을 생성하고 있었다면 작업을 잠시 멈추게 될 수 있습니다. 그러다가 다시 if문 안에서 resume되면, 그때는 또 다른 instance를 만들게 되겠죠.

```kotlin
<!-- Copyright 2019 Google LLC.
SPDX-License-Identifier: Apache-2.0 -->

public class Singleton{
    private static Singleton INSTANCE;
    private Singleton(){}
    public static Singleton getInstance(){
        if (INSTANCE == null) {                // Single Checked
            synchronized (Singleton.class) {
                if (INSTANCE == null) {        // Double checked
                    INSTANCE = new Singleton();
                }
            }
        }
        return INSTANCE;
    }
    private int count = 0;
    public int count(){ return count++; }
}
```

이문제를 해결하기 위해 double checked locking을 사용할 수 있습니다. 이를 사용하면 instance가 `null`일 때, `synchronized` 키워드가 lock을 걸고, 두 번쨰 if문에서 instance가 정말로 null인지를 다시 체크합니다. 만약 instance가 null이라면 Singleton을 만듭니다. 그러나 이것 만으로도 충분하진 않습니다. instance를 `volatile`로 선언해줘야합니다. [Volitile](https://en.wikipedia.org/wiki/Double-checked_locking#Usage_in_Java) 키워드는 변수가 동시에 동작하는 쓰레드들로 인해 비동기적으로 수정될 수 있음을 컴파일러에게 알려줍니다.

이 모든것은 여러분이 싱글톤이 필요할 때마다 작성해야 하는 보일러 플레이트 코드입니다. 사실 간단한 작업임에도 불구하고 코드는 꽤나 복잡하네요. 그래서 자바에서는 대부분 enum을 사용하여 싱글톤을 만들기도 합니다.

## Singleton in Kotlin

코틀린은 어떨까요? 코틀린은 static method와 static fields가 없는데, 그렇다면 어떻게 싱글톤을 만들까요?

안드로이드 스튜디오나 IntelliJ를 사용하면 이해하기가 좀 더 수월해집니다. Java로 만들어진 Singleton을 Kotlin으로 변환해보면, 모든 static property들과 메서드들은 companion object로 옮겨집니다.

```kotlin
<!-- Copyright 2019 Google LLC.
SPDX-License-Identifier: Apache-2.0 -->

class Singleton private constructor() {
    private var count = 0
    fun count(): Int {
        return count++
    }

    companion object {
        private var INSTANCE: Singleton? = null// Double checked

        // Single Checked
        val instance: Singleton?
            get() {
                if (INSTANCE == null) { // Single Checked
                    synchronized(Singleton::class.java) {
                        if (INSTANCE == null) { // Double checked
                            INSTANCE =
                                Singleton()
                        }
                    }
                }
                return INSTANCE
            }
    }
}
```

우리의 예상대로 코드가 변환되었지만 이걸 좀 더 간단하게 변경할 수 있습니다. 간단하게 하기 위해서 생성자와 companion 키워드를 지우고 object 키워드를 클래스명 앞에 붙여주세요. object와 `companion objects`의 차이는 글 아래에서 다루도록 할게요.

```kotlin
<!-- Copyright 2019 Google LLC.
SPDX-License-Identifier: Apache-2.0 -->

object Singleton {
    private var count: Int = 0

    fun count() {
        count++
}

```

`count()` 메서드를 사용하고 싶다면, Singleton object를 통해서 접근할 수 있습니다. 코틀린에서 `object`는 단일 instance를 가지는 특별한 클래스입니다. class 대신 `object`키워드를 사용하여 class를 만들면, 코틀린 컴파일러는 생성자를 private하게 만들어주고, object에 대한 static reference를 만들어주고, static block 안에서 reference를 초기화 해줍니다.

Static block은 static field에 처음 접근할 때 딱 한번만 호출됩니다. JVM은 static block을 처리할 때, 비록 `synchronized`키워드가 없다고 하더라도 synchronized block과 매우 비슷하게 다룹니다. Singleton 클래스가 초기화 될 때, JVM synchronized block위에서 lock을 획득하여 다른 쓰레드가 동시에 접근할 수 없도록 해줍니다. lock이 풀릴때면 Singleton 객체는 이미 생성이 되었고 static block은 두번 다시는 수행되지 않습니다. 이렇게 Singleton 객체가 단 하나만 있음을 보장하게 됩니다. 게다가 이 object는 thread-safe함과 동시에 object에 처음 접근하는 순간 생성됩니다(lazliy-created).

그럼 디컴파일된 Kotlin byte code를 살펴보고 실제로 어떤 일이 일어났는지 확인해봅시다.

> Kotlin class의 바이트 코드를 확인하려면 Tools > Kotlin > Show Kotlin Bytecode 를 선택하세요. Kotlin byte code가 나오면, Decompile을 눌러 decompile된 java code를 확인할 수 있습니다.

```kotlin
<!-- Copyright 2019 Google LLC.
SPDX-License-Identifier: Apache-2.0 -->

public final class Singleton {
   private static int count;
   public static final Singleton INSTANCE;
   public final int getCount() {return count;}
   public final void setCount(int var1) {count = var1;}
   public final int count() {
      int var1 = count++;
      return var1;
   }
   private Singleton() {}
   static {
      Singleton var0 = new Singleton();
      INSTANCE = var0;
   }
}
```

그러나 object도 한계가 있습니다. object 선언은 생성자를 가질 수 없기에 파라미터를 받을 수가 없습니다. 만약 파라미터를 받을수 있다고 하더라도, 생성자에 전달된 non-static 파라미터는 static block에서 접근할 수 없기 때문에 의미가 없을 겁니다.

> static 초기화 블록은 다른 static method처럼 오직 클래스의 static property에만 접근할 수 있습니다. Static block은 객체 생성 전에 호출되기 때문에 객체의 property나 생성자로 받아온 파라미터를 사용할 수 없습니다.

## Companion object

companion object는 object랑 비슷합니다. companion object는 언제나 클래스 안에 선언되며 클래스의 프로퍼티에 접근할 수 있습니다. 또한 companion object는 굳이 이름이 필요없습니다. 만약 companion object에 이름을 부여하면, 호출하는 쪽에서 companion object의 이름을 사용하여 member에 접근할 수 있습니다.

```kotlin
<!-- Copyright 2019 Google LLC.
SPDX-License-Identifier: Apache-2.0 -->

class SomeClass {
    //…
    companion object {
        private var count: Int = 0
        fun count() {
            count++
        }
    }
}
class AnotherClass {
    //…
    companion object Counter {
        private var count: Int = 0
        fun count() {
            count++
        }
    }
}
// usage without name
SomeClass.count()
// usage with name
AnotherClass.Counter.count()
```

예를 들면 위에 코드에는 이름이 있는 companion object와 이름이 없는 companion object가 있습니다. `count()`를 호출하는 쪽에서는 마치 SomeClass의 static 멤버인 것 처럼 `count()`를 사용할 수 있습니다. 또한 AnotherClass의 경우 static 멤버를 사용하듯이 Counter를 사용하여 `count()` 메서드에 접근할 수 있습니다.

companion object는 private 생성자가 포함된 inner class로 디컴파일 됩니다. host 클래스는 오직 자신만 접근할 수 있는 synthetic 생성자를 통해 inner class를 초기화 합니다. 그리고 다른 클래스에서 companion object에 접근할 수 있도록 public refernece를 유지합니다.

```kotlin
<!-- Copyright 2019 Google LLC.
SPDX-License-Identifier: Apache-2.0 -->

public final class AnotherClass {
    private static int count;
    public static final AnotherClass.Counter Counter = new AnotherClass.Counter((DefaultConstructorMarker)null);

    public static final class Counter {
        public final void count() {
            AnotherClass.count = AnotherClass.count + 1;
        }
        private Counter() { }
        // $FF: synthetic method
        public Counter(DefaultConstructorMarker $constructor_marker) {
            this();
        }
    }

    public static final class Companion {
        public final void count() {
            AnotherClass.count = AnotherClass.count + 1;
        }
        private Companion() {}
    }
}
```

## Object 표현식

지금까지 우리는 object keyword를 object 선언식으로만 살펴보았습니다. object keyword는 표현식으로도 사용될 수 있습니다. 표현식으로 사용하면, object keyword는 익명 객체와 익명 inner class를 생성할 수 있습니다.

잠시동안 어떤 값을 홀딩하고 있을 객체가 필요하다고 해봅시다. 객체를 선언하고, 원하는 값들과 함께 초기화 하여 추후에 접근할 수 있습니다.

```kotlin
<!-- Copyright 2019 Google LLC.
SPDX-License-Identifier: Apache-2.0 -->

val tempValues = object : {
    var value = 2
    var anotherValue = 3
    var someOtherValue = 4
}

tempValues.value += tempValues.anotherValue
```

generated code를 보면, 위의 코드는 <undefinedtype>으로 표기된 익명 Java class로 변환되어 getter와 setter와 함께 익명 객체가 저장됩니다.

```kotlin
<!-- Copyright 2019 Google LLC.
SPDX-License-Identifier: Apache-2.0 -->

<undefinedtype> tempValues = new Object() {
    private int value = 2;
    private int anotherValue = 3;
    private int someOtherValue = 4;

    // getters and setters for x, y, z
    //...
};
```

또한 object keyword는 boilerplate code 없이도 익명 클래스를 생성할 수 있도록 도와줍니다.

```kotlin
<!-- Copyright 2019 Google LLC.
SPDX-License-Identifier: Apache-2.0 -->

val t1 = Thread(object : Runnable {
    override fun run() {
         //do something
    }
})
t1.start()

//Decompiled Java
Thread t1 = new Thread((Runnable)(new Runnable() {
     public void run() {

     }
}));
t1.start();
```

---

object keyword는 thread-safe한 singleton을 생성하도록 도와줄 뿐만 아니라, 익명 객체와 익명 클래스를 별도의 추가 코드 없이 생성하도록 도와줍니다. object와 companion object를 사용하면 코틀린은 static keyword를 사용한 것과 동일한 기능을 하도록 코드를 생성해줍니다. 게다가 object를 표현식으로 사용하면 boilerpalte code 없이도 익명 객체와 익명 클래스를 만들 수 있습니다.
