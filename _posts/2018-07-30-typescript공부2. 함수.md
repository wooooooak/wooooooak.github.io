---
layout: post
title: Typescript Quickstart 공부2. 함수
category: typescript
tags: [typescript, 타입스크립트 퀵스타트]
comments: true
---

## 기본 형태
{% highlight typescript %}
function pow( x: number, y: number = 2 ): number {
  // bla bla
}
{% endhighlight %}

## 나머지 매개변수(rest parameter)에 타입 지정
rest parameter는 개수가 정해지지 않은 인수를 배열로 받을 수 있는 기능이다. 따라서 타입을 배열로 받아야 한다.
{% highlight typescript %}
const concat = (a: string, ...restParameter: string[]) => {
  return a + " " + restParameter.json(" ");
}
{% endhighlight %}

## 객체 리너럴의 선언과 객체 리터럴 타입의 선언
일반적으로 객체 리터럴 선언은 아래와 같이 한다.
{% highlight typescript %}
let person = {
  name: 'elecoder',
  hello: function (name2: string) {
    console.log("hello, " + this.name +" " + name2);
  }
};

console.log("World"); // hello, elecoder World
{% endhighlight %}

객체 리터럴의 hello 속성에 선언된 함수는 내부의 로컬 스코프에서 다른 객체 속성에 접근하려 할 때 코드 어시스트가 동작 하지 않는다(설명에는 나와 있지 않지만 내 생각에는 this가 어디에 바인딩 될지 모르기 때문이지 않을까?). 이런 상황에서 타입스크립트를 쓰는 장점 중에 하나인 코드 어시스트를 받기 위해서는, this에도 타입을 정해주면 된다. **객체 리터럴의 타입은 인터페이스를 이용해 정의한다.**
{% highlight typescript %}
// this는 반드시 첫 번째 매개변수로 선언해야한다.
// 컴파일되면 없어지는 매개변수이다.
interface PersonType {
  name: string;
  hello(this: PersonType, name2: string): string;
}

let person = {
  name: 'elecoder',
  hello: function (name2: string) {
    console.log("hello, " + this.name +" " + name2);
  }
};

console.log("World"); // hello, elecoder World
{% endhighlight %}
이제 코드 어시스트가 발동한다.(생각보다 거의 모든게 타입 지원이 가능하구나...)

## 익명 함수 할당시 팁
익명 함수의 매개변수나 반환값에 타입을 일일히 지정해 줄 수 있지만, 익명 함수가 구현체이므로 타입을 선언하면 형태가 복잡해진다. 따라서 익명 함수에 선언된 타입을 별도로 type 엘리어스로 분리하여 선언하고, 필요할 때 타입을 사용하면 좋다.
{% highlight typescript %}
// reuse 가능
type calcType = (a: number, b: number) => number;

let addCalc: calcType = (a, b) => a + b;
let minusCalc: calcType = (a, b) => a - b;
{% endhighlight %}