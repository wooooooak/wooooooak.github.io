---
layout: post
title: Explore Kotlin Annotations(번역)
category: 번역하며 공부하기
tags: [Kotlin, annotation]
comments: true
---

원본 - [Explore Kotlin Annotations](https://proandroiddev.com/explore-kotlin-annotations-d52eddd7e7b6)

(자바와 함께 사용할 때 도움이 되는 Kotlin annotation들)

이번 포스팅에는 Android app을 개발할 때 쓰이는 `Kotlin` annotation들을 살펴보겠습니다. 그 중에서도 **@JvmStatic**, **@JvmOverloads**, **@JvmField**에 대해 알아볼게요.

구글이 Android 개발 언어를 Kotlin으로 공식 지정한 이후로, 많은 개발자 분들이 Java를 Kotlin으로 변경하고 있습니다. 그렇지만 한순간에 모든 Java 코드를 Kotlin으로 전환할 수는 없죠.

그래서 코드의 일부(Class, enum 등등)들은 Kotlin으로 전환하되, 일부는 java와 혼합해서 사용될 수 있습니다. 만약 여러분이 이런 상황에 놓여있다면, 아래에서 다룰 annotation들을 알아두셔야 해요.

## @JvmStatic

코틀린에서는 pakage-level function이 static method로 표현됩니다. 또한 **@JvmStatic annotation**을 사용하여 일부 companion object나 이름있는 object에 정의된 함수에 대한 정적 메서드를 만들 수도 있습니다.

> @Target([AnnotationTarget.FUNCTION, AnnotationTarget.PROPERTY, AnnotationTarget.PROPERTY_GETTER, AnnotationTarget.PROPERTY_SETTER])
> **annotation class** JvmStatic

이 어노테이션이 함수에 적용된다면, 함수에 대해 추가적인 static method가 생성됩니다. 만약 프로퍼티에 적용된다면, 추가적으로 static getter/setter 메서드가 생성됩니다.

아래의 코드를 살펴볼게요.

```kotlin
class ClassName {
    companion object {
        @JvmStatic
        fun iAmStatic() {
            //body of Static function
        }
        fun iAmNonStatic() {
            //body of Non static function
        }
    }
}
```

`iAmStatic()`은 static function이고, `iAmNonStatic()`은 non-static function입니다. Java class에서 위의 메서드를 호출한다고 생각해보세요. 그 결과는 아래와 같습니다.

```kotlin
ClassName.iAmStatic(); // works fine
ClassName.iAmNonStatic(); // compile error
ClassName.Companion.iAmStaticMethod(); // works fine
ClassName.Companion.iAmNonStaticMethod(); // other way to work
```

이름있는 object의 경우에도 똑같이 적용됩니다.

```kotlin
object ObjectName {
    @JvmStatic fun iAmStatic() {
        //body of Static function
    }
    fun iAmNonStatic() {
        //body of Non static function
    }
}
```

Java에서 호출하면 결과는 아래와 같습니다.

```kotlin
ObjectName.iAmStatic(); // works fine
ObjectName.iAmNonStatic(); // compile error
ObjectName.INSTANCE.iAmStatic(); // works
ObjectName.INSTANCE.iAmNonStatic(); // works
```

## @JvmOverloads

> @Target([AnnotationTarget.FUNCTION, AnnotationTarget.CONSTRUCTOR]) **annotation class** JvmOverloads

이 어노테이션이 적용되면, 해당 함수의 default parameter 값을 대체하도록 컴파일러가 overloads를 생섭합니다. 만약 메서드가 N개의 파라미터를 가지고 있고, 그 중 M개가 default value를 가지고 있다면, M개의 overloads가 생성되는 것이죠. N-1개의 파라미터를 가진 것(default value 파라미터중 마지막 하나를 제외함) 한 개, N-2개의 파라미터를 가진 것 한 개, ... 이렇게 결국 모든 overloads가 생성됩니다.

여기 두 개의 필드를 가진 클래스가 있습니다. 필드 하나는 초기화 되었고, 다른 하나는 그렇지 않습니다.

```kotlin
data class Customer( val name: String, val dob: Date = Date())
```

Customer 객체를 생성할 때, 두 번째 argument로 어떠한 값도 넘기지 않으면 현재 시간이 적용될 것입니다. 따라서 이를 코틀린에서 호출할 때는 별다른 컴파일 에러 없이 아래 코드로 수행할 수 있습니다.

```kotlin
val custOne = Customer("Don")
val custTwo = Customer("Don", Date())
```

Java에서 동일하게 호출하면 어떻게 될까요? Java에서는 모든 파라미터를 꼭 입력해주어야 합니다. 그렇지 않으면 컴파일 에러가 발생해요.

```kotlin
Customer custOne = new Customer("Don"); //Here we are passing only one argument, so we will get compile error
Customer custTwo = new Customer("Don", new Date());//No error
```

Java에서 호출할 때 역시 default value를 사용하려면, kotlin 코드에 **@JvmOverloads**를 적용시켜야 합니다. 어노테이션을 적용한 kotlin 코드는 아래와 같습니다.

```kotlin
Customer @JvmOverloads constructor( val name: String, val dob: Date = Date())
```

이제 Java에서 호출할 때도 모든 파라미터를 넘길 필요가 없어집니다.

```kotlin
Customer custOne = new Customer("Don"); //No Error
Customer custTwo = new Customer("Don", new Date());//No error
```

## @JvmField

> @Target([AnnotationTarget.FIELD]) **annotation class** JvmField

이 어노테이션이 적용되면, 컴파일러는 해당 property에 대한 getters/setters를 생성하지 않고 field자체로 노출시킵니다.

아래에는 일반적인 Java코드가 있습니다.

```kotlin
public class Customer {
    public String name;
    public Date dob;

    public getName() {
        return this.name;
    }
    public getDob() {
        return this.dob;
    }
}
```

이에 대응되는 Kotlin 코드는 아래와 같습니다.

```kotlin
data class Customer( val name: String, val dob: Date = Date())
```

만약 위 클래스의 프로퍼티에 접근하려면, 코틀린에서는 아래와 같이 접근하죠.

```kotlin
val customer = Customer("Don", Date())
val name = customer.name
val dob = customer.dob
```

그러나 자바에서는 아래처럼 getter 메서드를 사용해야 합니다.

```kotlin
Customer customer = new Customer("Don", new Date());
String name = customer.getName();
Date dob = customer.getDob();
```

만약 여러분이 특정 field에 대한 getter, setter를 원하지 않고 그저 일반적인 field로 사용하고 싶다면 **@JvmField**를 사용하여 컴파일러가 getter, setter를 생성하지 않도록 할 수 있습니다. **@JvmField** 어노테이션을 적용한 kotlin 코드는 아래와 같습니다.

```kotlin
data class Customer(@JvmField val name: String, val dob: Date = Date())
```

이제 Java에서도 아래처럼 field에 접근할 수 있습니다.

```kotlin
Customer customer = new Customer("Don", new Date());
String name = customer.name;
```

감사합니다.
