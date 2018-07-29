---
layout: post
title: Typescript Quickstart 공부1. 변수 선언 및 기본 타입
category: typescript
tags: [typescript, default_type, 타입스크립트 퀵스타트]
comments: true
---

void 설명할 때, 112p 맨 밑에 설명을 참고하자
변수를 선언할 때 값을 할당하지 않았음을 나타내기 위해 선언한 변수에 null을 할당하는 것은 권장하는 방법이 아니다.

## 타입 스트립트는 점진적 타입 검사
javad와 C++은 타입의 생략이 불가능하며 실행 시간 전에 타입 검사를 하는 정적 타입 검사이다. 반면 자바스크립트는 실행 시간에 동적으로 타입 검사를 하는 동적 타입 검사를 가진다.
**타입스크립트**는 점진적 타입 검사를 가진다(파이썬도 그렇단다). 컴파일 시간에 타입 검사를 수행하면서 필요에 따라 타입 선언의 생략을 허용하기도 한다.
## 기본적인 타입
### 기본 타입
string, number, boolean, symbol, enum, 문자열 리터럴

symbol은 Symbol() 함수를 이용해 생성한 고유하고 수정 불가능한 데이터 타입으로 객체 속성의 식별자로 사용.
{% highlight typescript %}
let hello = Symbol();
{% endhighlight %}

enum은 number의 확장된 타입. ES6에 제안. 첫 번째 Enum 요소에는 숫자 0 값이 할당.
{% highlight typescript %}
enum WeekDay {Mon, Tue, Wed, Thu}
let day: WeekDay = WeekDay.Mon
{% endhighlight %}
enum은 number타입의 하위 타입으로 js로 컴파일된 후에는 객체 리터럴이나 배열처럼 객체 타입이 됨. typeof를 통해 확인해보면 object로 표시됨.


문자열 리터럴 타입은 string 타입의 확장 타입. 아래는 type 키워드를 이용해 "keyup" 문자열 또는 "mouseover"문자열만 허용하는 문자열 리터럴 타입을 정의한 것.
{% highlight typescript %}
type EventType = "keyup" | "mouseover";
{% endhighlight %}

### 객체 타입
Arry, Tuple, Function, 생성자, Class, Interface

Array
{% highlight typescript %}
let items: number[] = [1,2,3];
{% endhighlight %}

tuple
{% highlight typescript %}
let x: [string, number];
x = ["tuple", 100];
{% endhighlight %}

생성자 형식
{% highlight typescript %}
new < 타입1, 타입2> (매개변수1, 매개변수2, ...) => 타입
{% endhighlight %}



### 기타 타입

유니언
{% highlight typescript %}
let x: string | number;
{% endhighlight %}

인터섹션 타입: 두 타입을 합쳐 하나로 만들 수 있는 타입 (&)
{% highlight typescript %}
interface Cat { leg: number;}
interface Bird {wing: number;}
let birdCat: Cat & Bird = {leg: 4, wing: 2};
{% endhighlight %}

void, null, undefined

### 변형 타입
non-nullable: null 이나 undefined를 허용하지 않는 타입.
lookup(룩업) type: 인터페이스를 이용해 키값을 설정할 수 있는 타입.

## 타입스크립트의 내장 타입
### any 타입
모든 타입의 최 상위 타입. 따라서 any 타입은 js의 모든 값을 할당받을 수 있다.

타입스크립트 2.1 버전부터는 굳이 any타입은 선언하지 않고 사용해도 되지만(알아서 any를 붙여주기 때문), 명시적으로 any 타입임을 선언하는 것이 더 명확하므로 강제하는게 좋다. 그러기 위해서는 컴파일러 옵션중 noImplicitAny를 true로 설정하면 된다.

### 배열 타입과 제네릭 배열 타입
#### 유티언 타입을 이용해 선언한 배열
{% highlight typescript %}
let myVar: (number | string | boolean)[] = [1, "hi", true];
{% endhighlight %}

#### 제네릭 배열
Array<T> 형태로 선언.
{% highlight typescript %}
let num:Array<number> = [1,2,3];
let num2:Array<number | string> = [1, "Hello", 2];
{% endhighlight %}

타입을 참조할 때는 타입 쿼리를 이용한다. 타입 쿼리는 typeof 연산자를 이용해 참조할 변수의 타입을 얻어와 타입을 지정.
{% highlight typescript %}
let num:Array<number | string> = [1, "Hello", 2];
let num2:typeof num = [1, "Hello", 2];
{% endhighlight %}

#### 튜플 타입
배열 요소에 대응하는 n개에 대한 타입. 튜플에 선언된 타입 수와 배열의 요소 수가 정확히 일치돼야 한다.
{% highlight typescript %}
let x: [string, number] = ["typle", 100];
{% endhighlight %}

