---
layout: post
title: 수신 객체 지정 람다 - with와 apply
category: Kotlin
tags: [kotlin, with, apply]
comments: true
---

코틀린은 자바에는 없는 **수신 객체 지정 람다**라는 것을 가지고 있다. 처음 이 용어를 접하면 단번에 무슨 뜻인지 알아차리기 쉽지 않다. with와 apply라는 라이브러리 함수가 수신 객체 지정 람다인데, 말 그대로 **람다 함수를 쓸 때 내가 자주 쓰고싶은 객체를 미리 지정해서 사용하는 람다**라고 생각 하면 될 것 같다.

예제코드는 대부분 kolin in action에서 제공하는 코드이다.

## with

어떤 객체의 이름을 반복하지 않고도 그 객체에 대해 다양한 연산을 수행할 수 있다면 좋을 것이다. 먼저 `with`를 사용하지 않는 일반적인 코드를 보자.
![without_with](/public/img/kotlin/with1.png)
코드에서 `result`변수가 꽤 많이 나온다. 이 정도 반복은 봐줄만 하지만, 이코드가 훨씬 더 길거나 result를 더 자주 반복해야 했다면 상당히 귀찮은 작업의 반복이 될 수 있다.

이제 `with`를 사용한 코드를 보자.
![with1](/public/img/kotlin/with3.png)

`with`는 `when`이나 `if`문 처럼 언어가 제공하는 특별한 구문처럼 보이지만, 사실은 파리미터가 2개 있는 함수다. 첫 번째 파라미터는 `stringbuilder`이고, 두 번째 파라미터는 람다이다.

`with`함수가 어떻게 구현 되어있는지 살펴보면 이해가 더 쉽다.
![with2](/public/img/kotlin/origin_with.png)

첫 번째 인자로 받은 객체를 두 번째 인자로 받은 람다의 수신 객체로 만든다. 즉, 두 번째 인자로 받은 람다함수의 this가 곧 첫 번째 인자로 받은 객체가 된다고 생각하면 된다.

앞의 alphabet 함수를 더 리팩토링해서 불필요한 stringbuilder 변수를 없앨 수 도 있다.
![with3](/public/img/kotlin/with4.png)

with가 반환하는 값은 람다 코드를 실행한 결과며, 그 결과는 람다 식의 본문에 있는 마지막 식의 값이다. 그러나 때로는 람다의 결과 대신, 람다 함수 안에서 이리 저리 조작을 가한 수신 객체를 리턴 하고 싶은 경우가 있다. 그럴 때는 apply 함수를 사용할 수 있다.

## aplly

apply 함수는 with와 거의 같다. 단 한 가지 차이는, 자신에게 전달된 객체(수신 객체)를 반환한다는 점 뿐이다. apply가 어떻게 구현되어있는지 보자.
![with3](/public/img/kotlin/apply1.png)

apply가 확장 함수로 정의되어 있는 것을 알 수 있다. apply의 수신 객체가 전달받은 람다의 수신 객체가 되는 것이다.
![with3](/public/img/kotlin/apply2.png)
이 함수에서 apply를 실행한 결과는 StringBuilder 객체다. 따라서 그 객체의 toString을 호출해서 String 객체를 얻을 수 있다.

이런 apply 함수는 객체의 인스턴스를 만들면서 즉시 프로퍼티 중 일부를 초기화해야 하는 경우 유용하다. 자바에서 별도의 Builder 객체가 하는 역할을 담당하는 것이다.
