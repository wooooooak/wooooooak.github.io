---
layout: post
title: 사이드바 외부 클릭시 사이드바 닫히게 하는 방법
category: Frontend
tags: [React]
comments: true
---

나는 개인적으로 개발할 때 마다 모든 라이프사이클 api를 사용하진 않는다. 아마 나의 무지함 때문에 필요할때도 올바를 라이프 사이클을 선택하지 못할 때도 있었을 것이고, 정말로 내 프로젝트에서는 필요 없는 api들이 존재 하기 떄문일 수도 있다. 아무튼 lifecycle api를 한번 정리해둠으로써 프로젝트 진행시 틈틈히 참조해야겠다. 

## 컴포넌트가 처음 실행되는 Mount관련 api


**constructor(props)**

컴포넌트가 새로 만들어 질 때마다 호출된다. 프로젝트 구동시 여러 이벤트로 인해 한 컴포넌트가 여러번 생성되고 삭제됨을 반복하기도 하는데 새로 생성될 때 마다 constructor가 실행된다. 
```
return {showInputBox ? (
          <SearchInput
            onChange={this.onChangeInputValue}
            onKeyPress={this.onClickEnterInSearch}
          />
        ) : null}
```
이렇게 부모의 props에 따라서 showInputBox가 false일 경우 react dom에서는 SearchInput가 사라지기 때문에 componentWillUnmount() 함수가 실행되고, 다시 나타나게 되면 constructor가 실행됨


**componentWillMount**

신경쓰지 말자. deprecated되었다. UNSAFE_componentWillReceiveProps로 여전히 사용할 수 있긴 하지만 권장되진 않는다고 한다. 이 api에서 사용하던 것들은 constructor와 componentDidMount에서 사용하면 된다고 한다.

**render()**

컴포넌트를 DOM에 부착하는 함수.

**componentDidMount**

가상 DOM에서 실제 DOM으로 반영된 이후에 실행된다. 주로 실제 dom을 사용해야 하는 외부 라이브러리를 사용할 때나 네트워크 자원을 요청하는 로직을 수행한다.

## Update 관련 api
컴포넌트는 부모로부터 Props가 변경되거나, 자신의 State가 변경될 때 업데이트 된다.

**componentWillReceiveProps(nextProps)**

*deprecate* 된다. UNSAFE_componentWillReceiveProps()로 사용할 수는 있지만, 새로운 api인 static getDerivedStateFromProps()를 사용하자.

**static getDerivedStateFromProps(nextProps, prevState)**

이 메서드는 [velopert](https://velopert.com/3631)님의 자료에 아주 잘 설명 되어있어서 그 부분을 발췌하겠다.
```
static getDerivedStateFromProps(nextProps, prevState) {
  // 여기서는 setState 를 하는 것이 아니라
  // 특정 props 가 바뀔 때 설정하고 설정하고 싶은 state 값을 리턴하는 형태로
  // 사용됩니다.
  /*
  if (nextProps.value !== prevState.value) {
    return { value: nextProps.value };
  }
  return null; // null 을 리턴하면 따로 업데이트 할 것은 없다라는 의미
  */
}
```
state 가 props 에 따라 변해야 하는 로직을 작성하는 것. 추가적으로 static 메서드이기 때문에 this를 사용할 수 없다. 따라서 this.setState()가 아닌 object형태로 return 값을 넘겨야 한다. 이 것은 setState와 정확히 같은 방식으로 동작한다.

**shouldComponentUpdate(nextProps, nextState): boolean**

return으로 true 또는 false를 해줘야 한다. false를 리턴하게 되면, rendering하지 않는다. 랜더링 하지 않는 다는 것은, render()함수가 호출되지 않는 다는 것이고, 그 것은 가상 DOM에 어떤 작업도 하지 않는 다는 것이므로 성능 최적화를 할 수 있다. Props나 State가 변경되어서 컴포넌트가 update될 때, 무조건적인 rendering을 하지 말고 필요없는 redering은 하지 말자.

**componentWillUpdate** 

  
*getSnapshotBeforeUpdate(prevProps, prevState)* 로 대체 된다. 컴포넌트가 props가 변해서든 State가 변해서든 update되기 직전에, 원래의 dom상태를 가져온다. 이 것 역시 [velopert](https://velopert.com/3631)님의 너무 좋은 자료가 있어 발췌한다.
```
 getSnapshotBeforeUpdate(prevProps, prevState) {
    // DOM 업데이트가 일어나기 직전의 시점입니다.
    // 새 데이터가 상단에 추가되어도 스크롤바를 유지해보겠습니다.
    // scrollHeight 는 전 후를 비교해서 스크롤 위치를 설정하기 위함이고,
    // scrollTop 은, 이 기능이 크롬에 이미 구현이 되어있는데, 
    // 이미 구현이 되어있다면 처리하지 않도록 하기 위함입니다.
    if (prevState.array !== this.state.array) {
      const {
        scrollTop, scrollHeight
      } = this.list;

      // 여기서 반환 하는 값은 componentDidUpdate 에서 snapshot 값으로 받아올 수 있습니다.
      return {
        scrollTop, scrollHeight
      };
    }
  }

  componentDidUpdate(prevProps, prevState, snapshot) {
    if (snapshot) {
      const { scrollTop } = this.list;
      if (scrollTop !== snapshot.scrollTop) return; // 기능이 이미 구현되어있다면 처리하지 않습니다.
      const diff = this.list.scrollHeight - snapshot.scrollHeight;
      this.list.scrollTop += diff;
    }
  }
  ```

**componentDidUpdate(prevProps, prevState, snapshot)**

컴포넌트가 업데이트로 인해 render()메서드가 실행 된 이후에 실행되는 함수. 파라미터로 이전 props와 state, snapshot을 받아온다. 여기서 this.props를 하거나 this.state를 사용하여 현재의 상태를 참조할 수 있기 때문에 파라미터로 받는 것이 prev인것은 당연한 일이다.

**componentWillUnmount()**

이벤트, setTimeout, 외부 라이브러리 인스턴스들을 제거해주자


## 더 훌륭한 자료

* [Understanding React — React 16.3 + Component life-cycle](https://medium.com/@baphemot/understanding-react-react-16-3-component-life-cycle-23129bc7a705)
* [velopert님의 블로그](https://velopert.com/3631)
* [zerocho님의 블로그](https://www.zerocho.com/category/React/post/579b5ec26958781500ed9955)