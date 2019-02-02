---
layout: post
title: 흔들리는 종 만들기
category: CSS Challenge
tags: [html, css, animation, web_design]
comments: true
---

HTML, CSS만으로 흔들리는 종을 구현해보자! 아래 이미지 말고 생생한 현장을 보고싶다면 [Codepen.io](https://codepen.io/CLINNBT/pen/BMRYyp)로 오면 된다. 직접 수정해 볼 수도 있다. ~~GIF로 만들지 못한점은 정말 죄ㅅ...~~ 

![ring1](/public/img/css_challenge/ring1.PNG) 
![ring2](/public/img/css_challenge/ring2.PNG)

## HTML
먼저 HTML 코드부터 작성하자. HTML은 사실 너무 간단해서 한번에 다 올리겠다.
```html
<head><link rel="stylesheet" href="https://use.fontawesome.com/releases/v5.7.1/css/all.css" integrity="sha384-fnmOCqbTlWIlj8LyTjo7mOUStjsKC4pOpQbqyi7RrhN7udi9RwhKkMHpvLbHG9Sr" crossorigin="anonymous"></head>

<div class="box">
    <div class="bellWrapper">
      <i class="fas fa-bell my-bell"></i>
    </div>
    
    <div class="circle first"></div>
    <div class="circle second"></div>
    <div class="circle third"></div>
</div>
```

Html에서 주목할 점은 font-awesome을 가져오는 링크 뿐이다. cdn으로 font-awesome을 가져와 종모양 아이콘을 사용할 것이다(종소리까지 만들기 너무 힘들 것 같아서가 아니다. 절대).

덧붙이자면 붉은 박스 부분은 `.box`로, 그 아래에 종모양을 감싸는 `.bellWrapper`와 소리가 퍼져나가는 것을 표현할 `circle`을 나란히 작성했다.

![ring3](/public/img/css_challenge/ring3.PNG)
HTML만 작성했을 경우 위의 사진처럼 화면 좌측 상단에 검은색 종모향 하나만 딸랑 존재한다. 이제 CSS를 입혀 본격적으로 작업 해보자.

## CSS
가장 먼저 분홍색 박스를 감싸고 있는 `body`태그를 건드릴 것이다.
```css
.body {
    height: 100vh;
    width: 100vw;
    display: flex;
    align-items: center;
    justify-content: center;
    color: white;
}
```

`height`와 `width`를 `body`에 적용하였는데, 만약 적용하지 않는다면 아래와 같이 된다(사실 아직  박스에 색을 입히지 않아서 모든게 흰색으로 보이기 때문에 임의로 box에 칼라를 입혀서 보여준 것이다. 만약 따라하고 있는 중이라면 아무것도 보이지 않는게 정상이다). 즉 `body`의 크기가 자식 요소의 크기에 맞춰지게 되어 flex를 적용하였음에도 불구하고 화면의 딱 중간에 놓이지 않는다.

![ring4](/public/img/css_challenge/ring4.PNG)
그러니 body의 크기를 스크린 전체 크기로 잡아서 상하좌우 중앙정렬하자. 

중앙정렬은 flex를 사용한 후 아래처럼 아주 간단히 할 수 있다.
```css
    align-items: center;
    justify-content: center;
```
flex를 모른다면 [여기](https://flexboxfroggy.com/#ko)서 배우자. 게임형식으로 쉽게 배울 수 있다.


### box
이제 분홍색 box를 꾸며보자.

```css
.box {
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 10px;
  background: linear-gradient(to bottom,#DD3C54 0%,#ff686b 100%);
  padding: 150px;
}
```
여기까지 하면 바로 위에서 본 사진처럼 나올 것이다. box 내부에서도 상하좌우 중앙 정렬을 위해 flex를 사용했다. 

### 종 꾸미기
```css
.bellWrapper {
  font-size: 50px;
}

.my-bell {
  transform-origin: top;
  animation: bell 2s infinite linear;
}

@keyframes bell{
  0%, 50%{
    transform: rotate(0deg);
	}
  5%, 15%, 25%, 35%, 45% {
    transform: rotate(13deg);
  }
  10%, 20%, 30%, 40% {
    transform: rotate(-13deg);
  }
}
```
먼저 주목할 점은 `.bellWrapper`에서 `font-size:50px`를 해준 것이다. 이는 종 아이콘의 사이즈를 결정하는 부분인데, **어떻게 font-size로 아이콘 크기를 수정할 수 있는 것일까?**

그것은 font-awesome이 텍스트 방식으로 아이콘을 만들 수 있게 해주는 솔루션이기 때문이다. 컴퓨터에서 기본적으로 제공하고 있는 아이콘을 출력하는 방식이 아니라, 특수하게 개발된 Font Awesome 아이콘 전용 폰트 파일을 이용하여 아이콘을 출력한다.

#### .my-bell
`transform-origin: top;` 부분을 보자. `.my-bell`에 `transform`애니메이션(짧은주기의 화전을 좌우로 반복)을 부여할 것인데 이때 기준(origin)이 되는 위치를 지정하는 코드다. `top`으로 지정했으므로 transform의 중심, 기준은 윗부분이 되어 윗부분은 움직이지 않을 것이다.

#### 애니메이션을 50% 까지 잡은 이유
50%로 한 이유는, [Codepen.io](https://codepen.io/CLINNBT/pen/BMRYyp)을 들어가봤다면 알겠지만 뒷부분 50% -> 100%는 종이 흔들리지 않고 쉬게끔 하기 위해서다. infinite로 설정해주었기 때문에 0 ~ 100%를 반복할 것이다. 따라서 0 ~ 50%의 시간은 종이 흔들리고, 50 ~ 100%는 잠쉬 쉬고 다시 0%로 돌아가서 흔들림을 반복한다.

여기까지는 종이 흔들리긴 하지만, 주변에 아무 효과없이 단지 종만 흔들리고 있어서 심심하다. 울림 효과를 적용해보자.
### 울림 표현하기
```css
.circle {
  width: 100px;
  height: 80px;
  position: absolute;
  border: 2px solid white;
  border-radius: 70%;
  border-color: transparent white;
  animation: ring 2s infinite linear both;
}

.second {
  animation-delay: .3s;
}

.third {
  animation-delay: .7s;
}

@keyframes ring{
  0%, 100% {
    opacity: 0;
  }
  
  1% {
    opacity: 1;
  }

  50% {
    width: 250px;
    height: 250px;
    opacity: 0;
  }
}
```

`circle`은 말 그대로 원인데, 결과물을 보면 원은 찾아볼 수 없다. 단지 흰 곡선들이 양옆으로 퍼지는 효과만을 볼 수 있는데, 사실은 퍼지고 있는 곡선은 원의 일부분이다. 즉 원의 위, 아랫부분은 투명색인 것이고 좌우 부분은 흰색인 것이다. 그것은 `border-color: transparent white;` 이 코드로 가능하다. `margin`이나 `padding`을 적용할 때 처럼 short-hand가 동일하게 적용된다. 즉 `transparent`는 상,하에 적용되고 `white`부분은 좌,우에 적용이 된다. 

`position: absolute;`을 해줌으로써 원을 종소리와 동일하게 정 중앙에 위치시킬 수 있었다. 만약 `abolute`적용을 해주지 않았다면 아래 사진처럼 정말 다이나믹한 일이 벌어진다. 흰 원이 종소리를 기준으로 오른쪽에 추가가 되는데, 원 세개가 모두 겹쳐져 있지 않고 자기마음대로 애니메이션을 작동한다.

![ring5](/public/img/css_challenge/ring5.PNG)

아무튼 우리는 `absolute`를 적용해 주었기 때문에 3개의 원이 모두 겹쳐있는 상태이다(애니메이션 효과를 지우고 확인해 보라).

![ring6](/public/img/css_challenge/ring6.PNG)


종이 흔들리지 않을 때는 울림표시가 나타나면 안되므로 애니메이션의 시작부분(0%)에서는 `opacity`를 0으로 해주었다. 또한 겹쳐진 3개의 원이 차례대로 애니메이트되도록 `delay`를 다르게 부여했다.

애니메이션 100%에서 또다시 `opacity : 0`을 해준 이유는, 50%에서 opacity가 0이었다가 100%로 향하면 다시 opacity가 1로 변하기 때문이다. 기존의 opacity가 기본값인 1이기 때문에 돌아가는 것이다. 뿐만아니라 기존의 `width`와 `height`도 원래 값인   `width: 100px;`, `height: 80px;`으로 돌아간다. 그렇게 되면 처음 종이 흔들릴때 울림이 벽에 맞고 다시 종으로 모이는 꼴이 된다.


## 끄
읕!