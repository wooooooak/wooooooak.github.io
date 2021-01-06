---
layout: post
title: Data Driven Testing with Kotest(번역)
category: 번역하며 공부하기
tags: [Kotest, Data Driven Testing]
comments: true
---

**원본 - [Data Driven Testing with Kotest](https://proandroiddev.com/data-driven-testing-with-kotlintest-a07ac60e70fc)**

미리 정의된 입력&출력 값 Set을 사용하여 동일한 테스트를 반복해서 빠르게 실행하는 것을 [data driven testing](http://spockframework.org/spock/docs/1.1/data_driven_testing.html), [table driven testing](https://github.com/golang/go/wiki/TableDrivenTests) 또는 [table driven property checks](https://www.scalatest.org/user_guide/table_driven_property_checks)라고 부릅니다. 뭐라고 부르던지 Kotest는 이 테스트 패러다임을 훌륭하게 지원하며 이 글에서 그런 테스트를 어떻게 작성하는지를 다루려고 합니다.

두 integer 값 중 큰 값을 반환하는 max 함수 테스트를 생각해보죠. 일반적인 테스트 방식으로는 함수를 여러번 호출할 것입니다.

```kotlin
"maximum of two numbers" {
  Math.max(2, 3) shouldBe 3
  Math.max(0, 0) shouldBe 0
  Math.max(4, -1) shouldBe 4
  Math.max(-2, -1) shouldBe -1
}
```

잘 동작합니다. 하지만 data inputs 또는 argument가 아주 많을 경우 코드가 매우 길고 장황해질 것입니다. 또한 test logic이 지금 처럼 한 줄이 아니라 더 복잡해진다면 더욱더 보기 싫어지겠죠. Data-driven-testing를 통해 데이터를 테스트 로직에서 꺼낼 수 있습니다. 가독성이 좋아지겠죠.

먼저 꺼낼것은 inputs과 output을 포한한 데이터 set입니다. 이 set은 종종 table이라고 칭하는데요, 이 패러다임이 table driven testing이라고도 불리는 이유입니다. 각각의 data set은 row라고 불립니다.

본격적으로 시작하기에 앞서, `kotest-assertions` 라이브러리를 추가해야 합니다. [maven central](https://search.maven.org/search?q=a:kotlintest-assertions)에서 최신 버전을 확인할 수 있습니다.

Kotest에는 Data-drvien-testing을 구성하기 위해 두 가지 방법이 존재합니다.

## 첫 번째 방법

첫 번째로 `table` 함수를 사용하여 미리 tables을 정의하는 것입니다. `header` 함수를 사용하여 각 parameter에 이름을 지정할 수 있고, 각 `row` 함수의 인자로 row data를 넣으면 됩니다.

예를 들자면, 아래는 위에서 작성한 함수를 아래와 같이 작성할 수 있겠네요.

```kotlin
"maximum of two numbers" {
  table(
      headers("a", "b", "max"),
      row(2, 3, 3),
      row(0, 0, 0),
      row(4, -1, 4),
      row(-2, -1, -1)
  ).forAll { a, b, max ->
    Math.max(a, b) shouldBe max
  }
}
```

우리가 지정한 table은 3가지 arg로 정의되어있음을 확인할 수 있습니다(a, b, max). 그리고 4가지 data row를 정의했죠. 첫 번째, 두 번째 인자는 input(a, b)구요, 마지막 세 번째 인자는 기대하는 결과값(max)입니다.

table을 정의하고 나면 각 row별로 테스트를 진행할 lambda test function을 작성해야합니다. 여기에는 두 가지 옵션이 있는데, `forAll` 을 사용하여 모든 row들의 input이 통과될 것이라고 명시할 수 있고, `forNone` 을 사용하여 모든 row들의 input이 실패한다고 명시할 수도 있습니다. 즉 위의 테스트에 `forAll`를 사용한 경우 shouldBe를 사용하고, `forNone`을 사용한 경우 shouldNotBe를 사용해야 테스트가 통과됩니다.

이게 전부입니다. 이렇게 하여 4개의 data set을 테스트하는 코드가 완성되었습니다. 지금까지 봐서는, 맨 처음 보았던 예제보다 코드가 더 길어졌을 뿐이라고 생각할 수 있을겁니다. 그렇다면 이제 logic이 좀 더 복잡해서 한 줄이 훌 쩍 넘는 코드를 테스트해봅시다.

```kotlin
"pixel extraction example" {
  table(
      headers("path", "x", "y", "r", "g", "b"),
      row("space.jpg", 1, 2, 255, 0, 0),
      row("space.jpg", 0, 0, 255, 255, 0),
      row("worldcup.jpg", 23, 2, 17, 84, 221),
      row("mountain.jpg", 67, 825, 0, 0, 0),
      row("piano.jpg", 845, 53, 255, 0, 46),
      row("sunshine.jpg", 14, 423, 155, 65, 37)
  ).forAll { path, x, y, r, g, b ->

    // load image from resources
    val image = ImageIO.read(javaClass.getResourceAsStream(path))

    val rgb = image.getRGB(x, y)

    // shift 16 to get red
    rgb shr 16 and 0x000000FF shouldBe r

    // shift 8 to get green
    rgb shr 8 and 0x000000FF shouldBe g

    // just mask to get blue
    rgb and 0x000000FF shouldBe b
  }
}
```

image에서 pixel 값을 가져오고, 그 pixel의 RGB 값을이 주어진 input과 같은지를 테스트 합니다. 이런 테스트는 입력 값이 중요하죠.

보시는것 처럼 테스트 로직이 길어졌습니다. 물론 이를 다른 함수로 빼내어 간단히 한 줄로 호출할 수도 있겠죠. 하지만 각 row를 반복하는 loop를 작성해야하고, 함수를 호출해야하고, 에러 reporting 코드를 작성해야만합니다. 그렇게 깔끔하게 테스트 코드를 리팩토링 하다보면 결과적으로 위의 코드처럼 데이터가 이끄는 테스팅 형태가 되어버리겠죠. 깔끔하게 하면 할수록 위의 코드처럼 될 것입니다.

### error handling

당연히 처음 부터 모든 data set 테스트가 통과하진 않겠죠. 때문에 error handling은 매우 중요합니다. 테스트가 실패할 때 우리는 어떤 row가 테스트에 실패했는지, 그 row들의 매개변수들을 알고 싶어합니다. max 예제로 다시 돌아가서 마지막 row가 실패하는 케이스를 추가해봅시다.

```kotlin
"maximum of two numbers" {
  table(
      headers("a", "b", "max"),
      row(2, 3, 3),
      row(0, 0, 0),
      row(4, -1, 4),
      row(-2, -1, -1),
      row(4, 3, 2)
  ).forAll { a, b, max ->
    Math.max(a, b) shouldBe max
  }
}
```

max(4,3)은 명백히 2가 아닙니다. 따라서 결과는 fail이겠죠. 이를 실행하면 Kotest의 ouptut은 아래와 같습니다.

```kotlin
java.lang.AssertionError:
Test failed for (a, 4), (b, 3), (max, 2) with error expected: 2 but was: 4
```

보시다시피 실패한 input들에 대한 정보가 각자의 매칭되는 파라미터 이름과 함께 노출됩니다.

Kotest의 Data-Driven-Testing의 error handling 장점은 [AJ Alt](https://github.com/ajalt)의 노고 덕분에 여기서 멈추지 않습니다. kotest는 하나의 row가 실패했다고해서 테스트를 멈추지 않습니다. 하나의 row가 실패하더라도 나머지 row들을 테스트 하죠. 따라서 모든 row에 대해서 테스트를 돌며 실패한 모든 row의 정보를 알 수 있습니다. 아래의 코드를 봅시다. 실패하는 row 두 개가 더 추가되었습니다.

```kotlin
"maximum of two numbers" {
  table(
      headers("a", "b", "max"),
      row(2, 3, 3),
      row(0, 0, 0),
      row(4, -1, 4),
      row(-2, -1, -1),
      row(4, 3, 2),
      row(0, 0, 1),
      row(1, 2, 3)
  ).forAll { a, b, max ->
    Math.max(a, b) shouldBe max
  }
}
```

```kotlin
The following 3 assertions failed:
1) Test failed for (a, 4), (b, 3), (max, 2) with error expected: 2 but was: 4
 at com.sksamuel.kotest.data.DataDrivenTestingTest$2.invoke(DataDrivenTestingTest.kt:37)
2) Test failed for (a, 0), (b, 0), (max, 1) with error expected: 1 but was: 0
 at com.sksamuel.kotest.data.DataDrivenTestingTest$2.invoke(DataDrivenTestingTest.kt:37)
3) Test failed for (a, 1), (b, 2), (max, 3) with error expected: 3 but was: 2
 at com.sksamuel.kotest.data.DataDrivenTestingTest$2.invoke(DataDrivenTestingTest.kt:37)
```

모든 실패 케이스들이 다 노출되는 것을 볼 수 있습니다.

## 두 번째 방법

Kotest의 또 다른 특징 중 하나는 lambda 정의에 사용되는 변수의 이름을 추론할 수 있다는 점입니다. data-driven-testing을 하기 위한 두 번째 방법이죠.

max 테스트를 다시 봅시다. 아래와 같이 다시 작성할 수 있수 있을 것 같네요.

```kotlin
"maximum of two numbers" {
  forall(
      row(2, 3, 3),
      row(0, 0, 0),
      row(4, -1, 4),
      row(-2, -1, -1)
  ) { a, b, max ->
    Math.max(a, b) shouldBe max
  }
}
```

`table` 함수 대신 `forall` 함수로 시작했으며 인자로는 곧바로 row를 넘겼습니다(header 정의는 없습니다). Kotest는 람다에 부여된 파라미터 이름을 이용하여 각 row의 인자 값들의 이름을 추론할 수 있습니다. 따라서 람다 파라미터에 a, b, max 라는 이름을 부여해줍니다.

두 번째 방식이 첫 번째 방식보다 덜 장황하고 중복이 없기 때문에 보통 더 선호되는 방식입니다. 개인적으로도 더 우아한 방법이라 생각됩니다.

에러를 다루는 방법은 이전과 동일합니다. 극단적인 예시를 보자면 아래와 같습니다.

```kotlin
"contrived example of parameter name inference" {
  forall(
      row(1, 2, 3, 4, 5, 6, 7, 8, 9)
  ) { foo1, foo2, foo3, foo4, foo5, foo6, foo7, foo8, foo9 ->
    foo1 shouldBe 0
  }
}
```

위 코드가 실패하면 에로 로그는 아래와 같이 나타납니다.

```kotlin
java.lang.AssertionError:
Test failed for (foo1, 1), (foo2, 2), (foo3, 3), (foo4, 4), (foo5, 5), (foo6, 6), (foo7, 7), (foo8, 8), (foo9, 9) with error expected: 0 but was: 1
```

### 마무리

개인적으로 data-driven-test를 참 좋아합니다. 한번 test logic을 작성해 놓으면 다른 코드는 필요 없이 아주 쉽고 빠르게 coverage를 추가할 수 있기 때문입니다.

[여기 GitHub](https://github.com/kotest/kotest)에서 Kotest 프레임워크를 사용하기에 충분한 이유가 되는 다른 기능들을 살펴볼 수 있습니다.
