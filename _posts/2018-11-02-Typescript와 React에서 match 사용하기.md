---
layout: post
title: Typescipt, React 에서 match사용하기
category: Frontend
tags: [React]
comments: true
---

## 매치(match) 가 필요한 경우
react를 사용하여 라우팅을 할 때, 주소 값뒤에 :id, :name과 같이 추가적인 정보가 필요한 경우가 있다. 예를 App.js파일에서 라우팅 설정을 한다면 아래와 같을 수 있다.

#### App.tsx
```javascript
import { BrowserRouter, Route } from "react-router-dom";
...

class App extends React.Component<Props, IState> {

  ...

  render () {
    return (
      <BrowserRouter>
        <React.Fragment>
          <Route exact={true} path="/" component={IntroPage} />
          <Route exact={true} path="/signIn" component={SignInPage} />
          <Route exact={true} path="/signUp" component={SignUpPage} />
          ...
          <Route exact={true} path="/post/:postId" component={PostPage} />
        </React.Fragment>
      </BrowserRouter>
    );
  }
}

```

 만약 유저가 /post/1 이라는 주소로 들어오게 되고 postId로 무언가 하고싶다면(여기서는 단순 log를 찍어보는 것으로 하자), 타입스크립트를 적용하지 않은 React인 경우 PostPage컴포넌트에서 this.props.match.postId를 해주면 그만임을 알고 있다. (PostPage는 Functional Component로 하겠다.)

#### PostPage.js (pure React.js)
```javascript
const PostPage = ({match}) => {
  console.dir(match); // params, isExact, path, url 속성들...
  console.log(match.params.postId); // 1
  return <div>blablabla</div>
}
```
그러나 타입스크립트를 사용하게 되면, 늘 해당 컴포넌트가 받을 Props를 정의해 주었듯이, match를 받을 것이므로 match에 대한 Props 타입이 필요하다. 그렇다면 match의 속성인 params, isExact, paht, url 등의 타입을 일일히 만들어야 할까? 그렇지 않다. 만약 그렇다면 match 뿐만 아니라 location, history등의 객체도 있는기 때문에 상당히 복잡해질 것이다.

## react-router를 사용하면 된다.
흔히 stateless component에서 Props를 설정할 때, 아래와 같이 interface를 작성하는 경우가 많다.

#### stateless component에서 Props를 설정하는 간단한 예시
```typescript
interface Props{
  name: string;
  age: number;
}

const PersonPage:React.SFC<Props> = ({name, age}) => (
  <p>{name}, {age}</p>
)
```

match를 받을 때도 다를 것이 없다. react-router가 제공하는 RouteComponentProps 인터페이스를 사용하면 된다. 동작하는 코드를 보기전에 RouteComponentProps가 어떻게 되어있는지 알면 이해하기 쉬울 것 같다.

#### RouteComponentProps
```typescript
export interface match<P> {
  params: P;
  isExact: boolean;
  path: string;
  url: string;
}

...

export interface RouteComponentProps<P, C extends StaticContext = StaticContext, S = H.LocationState> {
  history: H.History;
  location: H.Location<S>;
  match: match<P>;
  staticContext?: C;
}

```

위의 코드를 보면 RouteComponentProps가 interface임을 알수 있고, match 뿐만 아니라 history, locaion 타입이 정의되어 있음을 알 수 있다. 그런데 **제네릭타입 P를 보면 디폴트 값이 없기 때문에 우리가 직접 입력해 줘야 한다.** 즉, P값으로 우리가 어떤 interface 타입을 부여하면, match객체의 params는 P라는 인터페이스 타입을 가지는 것이다. 따라서 아래와 같이 인터페이스를 정의하고 P값으로 부여하면, match.params 객체에서 postId를 받아와 출력할 수 있다.

```typescript
import * as React from "react";
import { withRouter, RouteComponentProps } from "react-router";

interface MatchParams {
  postId: string;
}

const PostPage: React.SFC<RouteComponentProps<MatchParams>> = ({ match }) => {
  console.log(match.params.postId);
  return (
    <p>{match.params.postId}</p>
  )
};

export default withRouter(UserPage);
```

## 참고자료
* [What TypeScript type should I use to reference the match object in my props?
](https://stackoverflow.com/questions/48138111/what-typescript-type-should-i-use-to-reference-the-match-object-in-my-props)