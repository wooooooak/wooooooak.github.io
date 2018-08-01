---
layout: post
title: Typescript Quickstart 공부7. 제네릭
category: typescript
tags: [typescript, 타입스크립트 퀵스타트]
comments: true
---

제네릭 역시 typescript만의 특성이아니라 정적타입을 지원하는 언어, 대표적으로 자바와 동일한 개념이기 때문에 크게 기록할만 한 부분은 없었다. 아주 조금, typescript의 제네릭을 사용할 때 알아두면 좋을 것들, 주의해야 할 사항들만 기록!

### 제네릭 함수, 클래스 선언 방법
함수나 클래스의 이름 뒤에 선언한다.
정의한 문자열만 할당받을 수 있게 하는 타입.
{% highlight typescript %}
function arrayConcat<T>(array1: T[], array2: T[]): T[] {
  return array1.concat(array2);
}
let array1: number[] = [1,2,3];
let array2: number[] = [4,5,6];

// resultConcat은 타입 선언이 불필요함.
let resultConcat = arrayConcat<number>(array1,array2);
{% endhighlight %}

### 타입 매개변수 간 연산시 주의 사항
우선 코드를 보자.
{% highlight typescript %}
function concat<T>(strs: T, strs2: T) {
  // return strs + strs2 에러
  return String(strs) + String(strs2);
}
cancat("abc","123"); // 타입 인수를 생략했기 때문에 타입을 추론해야 함
concat<string>("abc","123");
{% endhighlight %}
코드를 보면 strs + strs2는 불가능 하다. 프로그래머는 변수 명도 strs와 같이 문자열임을 알고 있긴 하지만 사실 T로 어떤게 들어올 지는 모르는 일이다. + 연산이 불가능한 객체가 T로 들어올 수도 있는 거니까. 그래서 T T 간 연산이 불가능하다. 그럼 T T 연산을 가능하게 하는 방법을 살펴보자.

#### 오버로드 함수를 이용한 타입 매개변수 간의 연산
오버로드 함수를 이용하면 T+T 와 같은 타입 매개변수 간에 연산이 가능하다.
* 오버로드 함수: 이름만 같고 매개변수의 타입이나 개수가 다르게 선언된 함수
{% highlight typescript %}

function concat<T>(strs: T, strs2: T): T;
function concat(strs: any, strs2: any) {
  return strs + strs2;
}
console.log(concat<string>("abc","123)); // abc123

{% endhighlight %}

본 함수를 건들지 않고 위에 오버로드 함수를 추가해줌으로써 본 함수가 제네릭 함수가 되도록 했다. 이렇게 하면 strs + strs2 연산도 정상! 추가적으로 다음과 같은 코드는 컴파일 에러가 난다.
{% highlight typescript %}
console.log(concat("abc", 1111));
{% endhighlight %}
함수의 매개변수 타입에는 일치하지만 오버로드 함수에 선언된 타입 매개변수의 형식과 일치하지 않기 때문.

### 타입 매개변수의 확장 extends
타입 구조 정의 파일을 보면 제네릭의 많은 부분이 
{% highlight typescript %}
<T extends string>
{% endhighlight %}
과 같이 extends가 붙는걸 많이 본적이 있다. 이게 뭔가했었는데 설명이 잘 나와있다. 아주 간단한데, T에도 타입을 제약한 것이다. 즉, 
{% highlight typescript %}
<T extends string | number>
{% endhighlight %}
이렇게 되어있다면 T는 string 이나 number 밖에 오지 못한다. 그러나 이렇게 한다고 해서 앞선 예제의 strs + strs2가 가능해 지는 것은 아니다.


### 타입 매개변수에 인터페이스를 상속하기
제네릭 클래스에 전달된 매개변수가 클래스이고 타입 매개변수일 때 코드 어시스트를 받지 못할 때가 있다. 이유는 타입 매개변수 <T>에 타입이 없기 때문이다. 코드 어이스트 지원을 받고 더욱 명시적으로 타입을 선언하려면 <T extends 인터페이스>처럼 클래스에 대한 인터페이스를 상속해주면 된다.