---
layout: post
title: Typescript Quickstart 공부5. 고급 타입
category: typescript
tags: [typescript, 타입스크립트 퀵스타트]
comments: true
---

### 타입 가드
타입을 유니언 타입(|)으로 복수개를 받을 경우, 유니언 타입에 대한 타입 검사를 손수 해서 안전성을 주는 방법을 타입 가드라고 한다. typedof로 확인해서 타입에 알맞는 처리를 함.

### 문자열 리터럴 타입
정의한 문자열만 할당받을 수 있게 하는 타입.
{% highlight typescript %}
let event: "keyup" = "keyup"; // 가능
let event: "keyup" = "keyup2"; // 불가능
{% endhighlight %}

### 룩업 타입(인덱스 접근 타입)
keyof 키워드를 통해 타입 T의 하위 타입을 생성해 낸다.
{% highlight typescript %}
interface Profile {
  name: string;
  gender: string;
  age: number;
}

// 위의 인스턴스를 keyof를 이용해 룩업 타입으로 선언
type Profile1 = keyof Profile;

// 이제 Profile1 타입은 Profile의 속성인 name, gender, age 중 하나를 할당받을 수 있는 하위 타입이 되었다.
let pVlaue1: Profile1 = "name";
{% endhighlight %}

### non-nullable 타입
컴파일러가 null 이나 undefined를 엄격하게 제한한다. 타입스크립트 2.0 전에는 null이나 undefined를 모든 타입의 변수에 할당할 수 있었다. 그러나 이것은 타입에 모호성을 준다. 그래서 이 것을 막기위해 나온 것이다.

이 타입은 :string 이나 :number 처럼 :non-nullable 이렇게 하는게 아니다. 설정을 해줘야 한다. tsconfig.json 파일의 strictNullChecks 옵션을 true로 설정하면 된다. 이렇게 되면 string이든 number든 boolean이든 null이나 undefined를 할당할 수 없다. 

이렇게 설정한 후에도 null이나 undefined를 예외적으로 할당하고 싶을 때가 있을 것이다. 그럴땐 방법이 있다. 유니언 타입으로 만드는 것이다.
{% highlight typescript %}
let title: string | null;
let title2: string | undefined;
{% endhighlight %}

### this 타입
인터페이스와 클래스의 하위 타입이면서 이들을 참조할 수도 있는 타입.
클래스 멤버 변수나 생성자에서 this 타입을 사용하면 가장 가까운 클래스의 인스턴스를 참조한다.

플루언트 패턴을 구현할 때 this 타입 반환을 선언하고 자신을 가리키는 this를 반환한다.
{% highlight typescript %}
Class AddCalc {
  public constructor(public value: number = 0) {}
  public add(operand: number): this {
    this.value += operand;
    return this;
  }
}

class MyCalc extends AddCalc {
  public multiply(operand: number): this {
    this.value *= operand;
    return this;
  }
}

let calc = new MyCalc(3).multiply(5).add(10); // 3*5=100
console.log(calc.value); // 25
{% endhighlight %}

### 정리
위의 타입들은 내가 타입스크립트를 사용하면서 거의 보지 못한 타입들이다. 타입스크립트를 사용하지 얼마 되지 않아서 그럴수도 있고, 내가 몰랐던 거라 보고도 지나친 것일수 도 있지만 두고두고 볼 필요는 있는 것 같다.