---
layout: post
title: Typescript Quickstart 공부6. 타입선언과 변경, 타입 호환
category: typescript
tags: [typescript, 타입스크립트 퀵스타트]
comments: true
---

## 타입 에일리어스
type alias를 사용하여 기존 타입에 새로운 이름을 지을 수 있다. interface와 비슷해 보인다(사실은 대부분 interface와 class 만드는게 주로 하는 일이라고 하는데, 쓰다 보면 가끔 interface를 type alias로 써도 좋을 때가 있다고 한다).

{% highlight typescript %}
type myAlias = string | undefined;
type User = {
  id: number;
  alias?: myAlias;
  city: string;
}
{% endhighlight %}

#### 타입 에일리어스를 이용한 배열 타입 선언
{% highlight typescript %}
type MyArrayType = Array<number|boolean>;
let myArray: MyArrayType = [1, true];
{% endhighlight %}


## 타입 추론
타입스크립트에서는 값을 할당할 때 타입을 명시하지 않으면 타입 추론을 통해 타입이 결정된다.
{% highlight typescript %}
let x = 1; // x의 타입은 number가 된다.
{% endhighlight %}

그러나 javascript와는 달리 let 선언이라 할지라도, 처음 값이 할당 될 때 타입이 정해지기 때문에 다른 타입의 값을 할당 할 수는 없다.
{% highlight typescript %}
// typescript
let first = 1;
let second = "a"; // error
{% endhighlight %}

## 타입 어셜션
타입 어셜션(type assertion)을 이용하면 타입스크립트 컴파일러가 타입 어셜션 정보를 이용해 컴파일을 수행한다. 따라서 타입 어설션은 컴파일 과정까지만 유효하고 컴파일 후에는 사라진다. 

또한 형변환과 다르다. 형변환은 실제 데이터 구조를 바꾸는 것이다. 반면 타입 어설션은 '타입이 이것이다'라고 컴파일러에게 알려주는 것이다. 대부분의 사용사례는, 더 넓은 범위의 타입이 있을 때, 예를 들어 유니온 타입으로 선언된 변수가 있을 떄 이 변수가 이 환경에서는 딱 한가지 타입으로만 될 수 있는 상황이라고 프로그래머가 확신을 하는 경우에 사용된다.

 타입 어설션의 선언 방식은 꺽쇠 괄호 방식과 as 문법을 이용한 방식이 있다.

{% highlight typescript %}
type myNum = any;

let num1: number = <number>myNum; // 선호되지 않음
let num2: number = myNum as number;
{% endhighlight %}
그러나 꺽쇠 기호로 사용하는 타입 어설션은 리액트의 jsx와 혼동되기 때문에 권장되지 않는다. as를 쓰자.

{% highlight typescript %}

{% endhighlight %}