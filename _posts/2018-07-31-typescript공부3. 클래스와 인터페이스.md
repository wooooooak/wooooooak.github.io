---
layout: post
title: Typescript Quickstart 공부2. 클래스와 인터페이스
category: typescript
tags: [typescript, 타입스크립트 퀵스타트]
comments: true
---
## 클래스
### 클래스에서의 접근제한자
es6에서는 private, public, protected와 같은 접근 제한자를 제공하지 않는다. 반면 타입스크립트는 지원하기 때문에 더 완벽한 객체지향 코딩이 가능하다. 생성자 매개변수에 접근 제한자를 추가하면 코드가 깔끔해진다고 한다.
{% highlight typescript %}
class Cube {
  public width: number;
  private length: number;
  protected heigth: number;
  constructor(pWidth: number, pLength: number, pHeight: number) {
    this.width = pWidth;
    this.length = pLength;
    this.heigh = pHeight;
  }
}
{% endhighlight %}

위 코드는 복잡하다고한다(나는 자바 프로그래밍을 해본 경험이 있어서 그런지 개인적으로 이게 가독성이 더 좋다고 생각한다). 아래와 같은 방법으로 더 짧은 코드작성이 가능하다.

{% highlight typescript %}
class Cube {
  constructor(public width: number, private length: number, protected height: number) {}
}
{% endhighlight %}
생성자 매개변수에 접근 제한자를 추가한 것만으로도 생성자 매개변수가 클래스의 멤버 변수가 되는 효과가 있다.

### 기본 접근 제한자
private, public, protected와 함께 default 접근 제한자가 존재한다. 이 것은 접근 제한자 선언을 생략할 때 자동으로 적용된다. 기본 접근 제한자로 사용하면 접근성은 어느정도가 될까? 기본 접근 제한자는 대체로 public이다. 그러나 예외적인 부분이 있는데, 생성자 매개변수에 접근제한자가 생략되면 생성자 내부에서만 접근할 수 있게 된다. 아주 당연한 이야기이다. 생성자 파라미터 부분에 접근제한자가 없다면, 그 것은 그냥 일반적인 함수의 변수 스코프와 다를게 없기 때문이다. 그러나 접근 제한자나 readonly가 붙으면 클래스의 매개변수 속성이 돼 멤버 변수가 된다.

### 추상 클래스
추상 클래스(abstract class) 역시 지원된다.
{% highlight typescript %}
abstract class 추상클래스 {
  abstract 추상메서드();
  abstract 추상멤버변수: string;
  public 구현메서드(): void {
    // 공통적으로 사용할 로직을 추가함
    // 로직에서 필요 시 추상 메서드를 호출해 구현 클래스의 메서드가 호출되게 함
    this.추상메서드();
  }
}
{% endhighlight %}

abstract 키워드는 static이나 private와 함께 선언할 수 없다. public이나 protected는 가능한데, 자식 클래스에서 추상 메서드를 오버라이딩 해야된다는 것을 생각하면 당연한 이야기다.

## 인터페이스
인터페이스는 타입이며 컴파일 후에는 사라진다(es6에는 인터페이스가 없다). 따라서 typeof을 이용해서 인터페이스의 타입을 조사할 수 없다.
{% highlight typescript %}
interface Car {
  // 접근 제한자는 설정할 수 없다.
  speed: number;
}
{% endhighlight %}
자식 인터페이스는 extends 키워드를 이용해 부모 인터페이스를 확장할 수 있다.
{% highlight typescript %}
interface 자식인터페이스이름 extends Car {}
{% endhighlight %}

### 인터페이스의 역할
js객체는 구조를 고정할 수 없고 쉽게 변화하는 특성이 있다. 객체는 유지보수와 확장, 안전성을 고려해 선언과 동시에 고정할 필요가 있다. 인터페이스를 이용해 객체의 구조를 고정할 수 있다.

### 객체 리터럴을 타입으로 사용하기
{% highlight typescript %}
// type 정의
type objectLiteralType =  { name: string, city: string};
// 또는 아래도 가능
// interface Person {
//   name: string;
//   city: string; 
// }

let person: objectLiteralType[];

person = [
  { name: "a", city: "seoul"},
  { name: "b", city: "seoul"}
]
{% endhighlight %}


## readonly 제한자
readonly는 타입스크립트 2.0 부터 지원된다. readonly가 선언된 변수는 초기화 되면 절대 재할당이 불가능하다.
{% highlight typescript %}
interface ICount {
  readonly count: number;
}

class TestReadonly implements ICount {
  readonly count: number;
  getCount() {
    // 아래는 에러가 난다. readonly는 메서드나 함수안에서 선언할 수 없다
    // readonly count2:number = 0;
    // 반면 const는 가능하다.
    const count2:number = 0;
  }
}
{% endhighlight %}

const도 재할당이 불가능한건 마찬가지인데 그렇다면 이 둘은 무슨 차이일까? [스택오버플로우](https://stackoverflow.com/questions/46561155/difference-between-const-and-readonly-in-typescript)에 따르면 const 변수에 재할당이 되는지 안되는지, 그래서 에러를 뿜어 낼지 말지는 오로지 runtime때 알 수 있다. 하지만 readonly를 사용한다면? 컴파일 시간에 알 수 있다. 실수로 값을 재 할당 하려고 하는 순간 빨간줄이 그인다!

## 나머지
클래스와 인터페이스 파트는 전체적으로 분량이 상당히 많지만, 타입스크립트와 관련된 내용이라기 보다는 기본적인 OOP 프로그래밍에 대한 설명, ES6에 대한 설명이 대부분이라서 딱히 적을 부분이 없다.