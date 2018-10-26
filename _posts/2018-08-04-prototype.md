---
layout: post
title: javascript 프로토타입(prototype) 이해
category: javacript
tags: [javacript, basic]
comments: true
---

최근들어 거의 대부분을 js로 코딩하고 있음에도 불구하고 제대로 프로토타입을 이해하지 않고 있었다. 사실은 대충 아는 정도였는데, java와 python 코딩을 하면서 class에 너무 익숙해져 있었던 상황에서 얼핏 보았을때 prototype이나 class나 하는 역할은 비슷해 보여서 깊이 공부해보진 않았다. 아무튼 앞으로 js 코딩할 일이 너무 많아질 것 같은 지금 쯔음에 prototype을 공부해보고 정리해 보았다.

## 자바스크립트는 객체지향 언어인가?
우리가 흔히 객체지향을 공부할 때 java나 c++, c#, python을 두고 배운다. 반대로 이 언어들을 배울 때 무조건 꼭 포함된 단원이 객체지향 프로그래밍이며 모두 클래스를 가지고 있다. 그렇다면 자바스크립트는? 클래스가 없다. 그러나 자바스크립트 역시 **객체지향 언어**이다! 우리가 집중해야 할 것은 **객체지향** 프로그래밍이지, 클래스 지향 프로그래밍이 아니다. 객체를 만들 수 있으면 객체지향 언어다. 자바스크립트는 클래스 없이, prototype을 기반으로 객체지향 코딩을 할 수 있게 한다. 그래서 prototype에 대해서 공부해야 한다. java를 class를 배우는 것 처럼 :D

es6에서는 class문법이 추가되어 다른 클래스를 사용하는 언어처럼 코딩할 수 있게 되었지만, 문법이 추가됨으로써 클래스를 흉내낼 뿐, 여전히 내부적으로 prototype을 사용한다. 내 생각이지만 js 개발자들의 많은 수요를 받아들여 클래스 문법이 추가되었다는 것은 prototype보다 class를 활용한 객체지향 코딩이 더 편하고 쉽다는 의미인 것 같기도 하다. 나 역시도 그렇게 느끼고..

## prototype을 언제 왜 쓸까?
모든 Javascript 객체는 다른 객체에 대한 참조인 prototype 프로퍼티를 가지고 있다. 내가 이해한 바로는, 가장 큰 이유로 상속을 위해 prototype을 사용한다(사실은 상속보다는 위임에 가깝다). 
클래스기반 언어를 예로 들었을 때,
{% highlight java %}
  public class Person {
    int head = 1;
    int legs = 2;
  }
{% endhighlight %}
이런 클래스가 있다고 치자. 사람이라면 머리가 하나이고 다리가 두개이니 Person이라는 클래스를 만들어 거기에 넣어두었다. 그리고 Programmer라는 클래스를 만들어 Person을 상속받아 보겠다.
{% highlight java %}
  public class Programmer extend Person {
    boolean hasLaptop;
  }
{% endhighlight %}
이처럼 head나 legs는 사람이라면 모두 해당되는 속성이므로 이를 상속받음으써 코드의 재사용성을 높여주었다. programmer 뿐만 아니라 축구선수, 학생, 요리사 등등의 클래스를 만든다 할지라도 굳이 head나 legs를 선언 해 줄 필요 없이 Person을 상속받기만 하면 된다.

이와 같은 일을 자바스크립트에서는 prototype으로 한다. 내가 공부한 prototype을 조금 자세하게 정리해 보겠다.


## 자바스크립트로 짠 코드부터 보자
{% highlight javascript %}
function Person() {};
Person.prototype.head = 1;
Person.prototype.legs = 2;

const lee = new Person();
console.log(lee.legs)  // 2;

function Programmer() {};
Programmer.prototype = new Person();
Programmer.prototype.hasLaptop = true;

const song = new Programmer();
console.log(song.head, song.hasLaptop); // 1 true

{% endhighlight %}

lee.legs를 출력했더니 마치 Person클래스를 상속받은 것 처럼 2가 출력되었다. prototype이 '원형', '기원' 이라는 뜻임을 보면, 상속이라는 개념과 어울릴 것 같은 느낌이 든다. 위의 코드를 이해하기 위해 조금 더 정리해보겠다.

## 함수 선언시 일어나는 일 부터 보자
{% highlight javascript %}
function Person() {}
{% endhighlight %}
이렇게 구현부에 내용이 없는 함수를 하나 선언하면 js내부적으로 어떤 일이 일어난다. 우선, 함수를 선언 했으니 함수가 생성이 될 것이다. Person이라는 함수를 선언 했으니 Person이라는 함수가 메모리에 생성이 되고, 그와 동시에 Person Prototype Object라는 객체가 메모리에 생성이 된다. Person Prototype Object는 말 그대로 객체인데 이게 핵심! 우리가 위에서 
```
Person.prototype.head = 1;
```
이렇게 head값을 주면 모두 Person Prototype Object의 속성으로 저장되고, Person 함수로부터 만들어진 모든 객체는 이 Person Prototype Object의 속성으로부터 head값을 받아온다. 즉, Person.prototype 으로 Person Prototype Object를 참조할 수 있다는 이야기. 정말 그런지 브라우저에서 실행한 화면을 보자.
![prototype1](/public/img/prototype/one.JPG)

보는 것 처럼 Person함수를 선언하기만 했을 뿐인데 Person을 출력하면 이렇게 많은 내용이 출력된다. js 내부적으로 뭔가가 일어난 것이다. 내용을 보면 prototype이라는 속성이 있다. 이게 아까 말한 Person Prototype Object이다! 메모리 어딘가에 만들어진 이 Person Prototype Object객체를 참조하고 있는 것이다. 즉 Person Prototype Object 는 constructor와 __proto__라는 속성을 가지고 있는 셈이다. 일반 객체이기 때문에 속성을 마음대로 추가할수 있다. 위에서 본 코드
```
Person.prototype.head = 1;
```
처럼. 이렇게 코드를 작성한 후에 변화된 내용을 보자.
![prototype2](/public/img/prototype/two.JPG)

Person.prototype, 즉 Person Prototype Object의 속성에 head값이 설정된 것이 보인다.

## Person Prototype Object의 __proto__는 뭘까?
__proto__는 객체가 생성될 때 조상이었던 함수의 Prototype Object를 가리킨다. 즉, 어떤 객체이든 함수로 부터 생겨나기 때문에 모든 객체는 조상이 되는 함수가 존재할 수 밖에 없는데, 바로 그 함수의 Prototype Object를 가르키는 것이다.

앞에서 첨부한 Person을 로그찍은 사진을 보시면 __ proto __: Object 라고 되어있는 것을 볼 수 있다. __proto__는 객체가 생성될 때 자신의 조상었던 함수의 Prototype Object를 가리킨다고 했었다. 자바스크립트에서는 함수도 일급 객체다. 그럼 Person함수(사실은 일급 객체)가 생성 될 떄 조상이었던 함수의 Prototype Object가 __proto__라는 것인데, 우리가 Person함수를 만들 때 어떤 함수를 상속받지도, new 연산자를 사용하지도 않았는데 도대체 Person함수를 생성할 때 조상이었던 함수가 뭘까? 

바로 Object라는 함수이다. 이제막 js를 접하신 분들은 이상하게 들릴 수도 있지만, js의 모든 객체(함수 포함)는 함수로부터 만들어 지는데 자바스크립트에서 기본적으로 이 함수를 Object라는 이름의 함수로써 제공한다. Person함수(객체)의 조상도 Object함수인 것. 

이제 또 사진을 보자.

![prototype3](/public/img/prototype/threeJPG.JPG)

아까 분명히 
```
Person.prototype.head = 1;
```
이 코드를 수행했음에도 불구하고 song 객체를 만들어 로그를 찍어보면 __proto__속성 밖에 없다. 그러나 문제 될 것이 없다! song 객체는 Person함수로 부터 만들어 졌으므로 song의 __proto__는 Person Prototype Object를 가르키고, 이 Person Prototype Object 안에 모든게 들어있으니까 
```
console.log(song.head);
```
와 같이 출력해도 js가 알아서 __proto__을 뒤져 head값을 찾고 출력해준다. 또 사진을 보자.

![prototype4](/public/img/prototype/four.JPG)

위 사진처럼 prototype을 건들이지 않고 딱 song객체에만 age를 추가하면 이렇게 결과가 나온다. 이 age는 prototype object에 전혀 영향을 미치지 않는다.

## prototype chain
위의 내용을 이해했다면 프로토타입 체인은 아주 쉽게 이해가 가시리라 생각된다. 말 그대로 프로토타입이 체인처럼 이어져 있는 것인데. Person은 Object 함수로 부터 만들어 졌으므로 Person의 prototype은 Object함수이다. 그렇다면 한 단계 더 깊어져서 Programmer라는 함수가 Person을 조상으로 한다면 어떻게 될까? 우선은, 이게 어떻게 가능한 것일까? 코드는 이미 앞에서 보았다.

{% highlight javascript %}
function Programmer() {};
Programmer.prototype = new Person();

const kim = new Programmer();
console.log(kim.head);
{% endhighlight %}

바로 이 코드. 다시 한번 언급하지만 prototype은 메모리 어딘가에 저장된 Prototype Object을 가르킨다고 했었다. 따라서 간단하게 Programmer의 prototype을 Person으로 가르킨게 끝이다. 그럼 Programmer의 조상은 Person이다.

코드 마지막줄의 kim에서 kim.head를 출력했하려고 하는데, kim자신에게는 head가 없다. 
``` 
kim.head = 1;
```
이렇게 해준 적이 없으니까. 그래서 Programmer의 prototype Object를 뒤져보는데 또 head가 없다, 그럼 또 그 조상인 Person Prototype Object를 뒤져서 head를 찾아낸다. 이게 바로 prototyhpoe chain이다! 



## 정리
개인적으로 prototype과 __proto__가 언제 어디서 쓰이는지가 자꾸 헷갈렸다.
정리하자면 prototype을 사용할 수 있는곳은 함수뿐이다. 함수만이 prototype속성을 사용할 수 있다.

{% highlight javascript %}
Person.prototype; // {head: 1, constructor: 함수, __proto__: 객체}

kim.prototype;  // undefined
{% endhighlight %}

반면 __proto__는 js의 모든 객체에서 사용된다. 모든 객체는 자신을 만든 조상 객체를 참조할 필요가 있으니까. 함수도 객체이니 함수에서도 사용된다. 함수도 무언가로부터 생성된 객체라는 점을 잊지 말자!



### 더 좋은 자료
- [
Insanehong 님의 Javascript 기초 - Object prototype 이해하기](http://insanehong.kr/post/javascript-prototype/)
- [Nextree JavaScript : 프로토타입(prototype) 이해](http://www.nextree.co.kr/p7323/)
- [오승환님의 [Javascript ] 프로토타입 이해하기](https://medium.com/@bluesh55/javascript-prototype-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-f8e67c286b67)