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

## 기본 타입
string, number, boolean, symbol, enum, 문자열 리터럴

symbol은 Symbol() 함수를 이용해 생성한 고유하고 수정 불가능한 데이터 타입으로 객체 속성의 식별자로 사용.
{% highlight javascript %}
let hello = Symbol();
{% endhighlight %}

enum은 number의 확장된 타입. 첫 번째 Enum 요소에는 숫자 0 값이 할당.
{% highlight javascript %}
enum WeekDay {Mon, Tue, Wed, Thu}
let day: WeekDay = WeekDay.Mon
{% endhighlight %}

문자열 리터럴 타입은 string 타입의 확장 타입. 아래는 type 키워드를 이용해 "keyup" 문자열 또는 "mouseover"문자열만 허용하는 문자열 리터럴 타입을 정의한 것.
{% highlight typescript %}
type EventType = "keyup" | "mouseover";
{% endhighlight %}
