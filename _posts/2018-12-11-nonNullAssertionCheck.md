---
layout: post
title: postfix ! (난 undefined가 아니에요)
category: typescript
tags: [typescript, grammar]
comments: true
---

## ! postfix
`!` postfix는 해당 프로퍼티가 null 또는 undefined가 아니라는 것을 나타낸다. 

타입스크립트가 타입 체킹을 해줌으로써 runtime시점에 발생할 수 있는 많은 에러들을 컴파일 시점에 알려주지만 어쩔수 없이 runtime에 에러가 나는 경우가 있다. 아래 예제를 보자.
```typescript

interface Person {
    name: string;
    hobby?: string;
}

type eventType = string;

const person: Person = {
    name: 'jun'
};

let eventName = person.hobby; // 'string | undefined' 형식은 string' 형식에 할당할 수 없습니다.
```

위의 코드를 실행해보면 `eventName`에 빨간 줄이 그인다(옵션 설정에서 strictNullChecks: true를 했을 경우). `person`객체에 `name` 속성은 분명하게 있지만, `hobby` 속성은 ?가 붙어있어 optional하기 때문에 hobby가 있을 수도, 없을 수도(undefined) 있기 때문이다. 그리고 실제 `person` 객체에는 hobby가 없기 때문이다.

이런 경우 귀찮지만 아래 코드와 같이 손수 타입 체킹을 해줄 수가 있다.
```typescript
let eventName;
if (person.hobby) {
	eventName = person.hobby;
}

```

그러나 개발자가 `person`객체에 hobby가 있다고 확신한다면? 간단히 ! 접미사를 사용해서 아래와 같이 할 수 있다.

```typescript
const eventName: eventType = person.hobby!;
```
이 것은 아래처럼 말하는 것과 같다.
> hobby는 null이나 undefined가 아니에요!

물론 정말 null이나 undefined가 아니어야만 한다!

## 언제 ! 를 쓰나?
개인적으로는 redux + typescript를 사용하면서 Action의 Payload 타입을 지정할 때 써먹었다. 액션의 interface를 보면,
```typescript
export interface Action<Payload> extends BaseAction {
    payload?: Payload;
    error?: boolean;
}
```
이렇게 payload가 optional이다. 즉, redux-actions 라이브러리는 payload가 기본적으로 없을 수도 있다고 생각하는 것이다. 물론 정말 payload가 없는 액션도 있지만, payload가 있는 액션을 처리할 때 ! 접미사를 사용한다(이 액션에는 payload가 undefined가 아니에요!). 

긴 코드를 제외하고 reducer가 새로운 상태를 반환하는 부분만 보면 아래와 같다.
```typescript
return {
    ...state,
    category: action.payload!
};
```

### TypeScript Deep Dive 예제
[TypeScript Deep Dive (영어)](https://basarat.gitbooks.io/typescript/)에는 !를 사용하는 다른 예시도 있다.
```typescript
let a: number[]; // No assertion
let b!: number[]; // Assert

initialize();

a.push(4); // TS ERROR: 값을 할당하기도 전에 사용할 수 없습니다.
b.push(4); // OKAY: because of the assertion

function initialize() {
  a = [0, 1, 2, 3];
  b = [0, 1, 2, 3];
}
````
> `!` 는 컴파일러에게 당신을 믿으라고 말하는 것이다. 그럼 컴파일러는 코드가 값을 할당하지 않았더라도 불평하지 않을 것이다. 


## 참고자료
* [TypeScript Deep Dive (영어)](https://basarat.gitbooks.io/typescript/docs/options/strictNullChecks.html)

* [How to properly use TypeScript with redux-actions? - Question](https://github.com/redux-utilities/redux-actions/issues/282)