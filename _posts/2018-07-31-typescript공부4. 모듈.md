---
layout: post
title: Typescript Quickstart 공부4. 모듈
category: typescript
tags: [typescript, 타입스크립트 퀵스타트]
comments: true
---

## 네임스페이스
네임스페이스는 하나의 독립된 이름 공간을 만들고 여러 파일에 걸쳐 하나의 이름 공간을 공유할 수 있다. 같은 네임스페이스의 이름 공간이라면 파일 B가 파일 A에 선언된 모듈을 참조할 수 있는데, **참조할 때는 별도의 참조문을 선언하지 않아도 된다**. 이 말은 곧, 파일이 다르더라도 같은 네임스페이스 안에 속한다면 변수든, 함수든, 클래스든 이름이 중복되어선 안된다는 말이다. 즉, 네임스페이스(내부모듈)은 여러개의 파일을 마치 하나의 파일인 것 처럼 사용할 수 있게 하는 것이다.

### 선언하는 방식
{% highlight typescript %}
namespace Hello { ... }
{% endhighlight %}

한 파일에 여러 네임스페이스를 선언할 수 있다. 이럴 경우, 같은 파일이지만 export된 모듈들만 불러올 수 있다.
{% highlight typescript %}
namespace MyInfo1 {
  export let name = "happy1";
  export function getName2() {
    return MyInfo2.name2;
  }
}

namespace MyInfo2 {
  export let name2 = "happy2";
  export function getName() {
    return MyInfo1.name;
  }
}

console.log(MyInfo.getName2()); // happy2
console.log(MyInfo2.getName()); // happy1
{% endhighlight %}


### 네임스페이스 모듈
네임스페이스는 export를 이용해 모듈로 선언할 수 있다.
{% highlight typescript %}
export namespace Car { ... }
{% endhighlight %}
그런 다음 import문으로 불러올 수 있다.
{% highlight typescript %}
import * as ns from "./car1";
{% endhighlight %}

## 외부 모듈
원래 알던 내용들이고, 딱히 타입스크립트 만의 어떤 내용이 있지 않은것 같다. 몇가지 내가 몰랐던 점만 적자.
#### 디폴드 모듈 선언 2가지 방법
{% highlight typescript %}
// 기존에 늘 하던 방식
export default {
  title: "hello world!",
  length: 11
};
{% endhighlight %}

{% highlight typescript %}
// 변형된 형태
export a = {
  title: "hello world!",
  length: 11
};
export { a as default };
{% endhighlight %}

당연한 이야기지만 default 키워드는 한 번만 사용해야 하므로 아래와 같은 코드는 불가능하다.
{% highlight typescript %}
export { p as default, h as default };
{% endhighlight %}

### 끝내며
이번 파트도 역시 typescript를 위한 파트라기 보다는 기본적인 모듈을 이해하기 위한 파트였던 것 같다. 딱히  typescript만의 특별한 점은 크게 없는 것 같다. typescript는 es6 모듈 형식이 default라는 것 정도...