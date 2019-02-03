---
layout: post
title: Search Box와 검색어 추천 만들기
category: CSS Challenge
tags: [html, css, javascript, web_design]
comments: true
---

HTML, CSS 그리고 JAVASCRIPT로 Search Box 디자인을 해보자. 우리의 목표는 [여기 링크](https://codepen.io/CLINNBT/pen/qgjEWp)에서 볼 수 있고 아래 이미지로도 볼 수 있다.

![searchBox1](/public/img/css_challenge/SearchBox1.PNG) 
![searchBox3](/public/img/css_challenge/SearchBox3.PNG)

해보자!

## HTML
```html
<div class="frame">
  <div class="center">
    <input type="text" placeholder="Start typing ..."/>
    <div id="recommend" class="invisible">
        <div class="item"><span class="text"></span></div>
        <div class="item">Veritatis et <span class="text"></span></div>
        <div class="item">ut aliquid ex <span class="text"></span> vero eos</div>
    </div>
  </div>
</div>
```
실제로 검색어를 추천하는게 아니라, 검색어를 추천할 때 생길 디자인을 하는 것이 목적이므로 검색어들을 하드코딩했다.

`.frame`은 박스 파란 박스부분이고, `.center`는 우리가 앞으로 넣을 것들을 `.frame`의 중앙에 놓기위한 박스라고 생각하면 된다. `input`은 검색어를 입력하는 곳이고 `#recommend`는 추천 검색어들이 뜨는 공간이다. `div`로 했기 때문에 요소들이 아래쪽으로 쭉쭉 나열될 것이다.


## CSS
먼저 `.frame`을 구현하자.
```css
.frame {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 400px;
  height: 400px;
  margin-top: -200px;
  margin-left: -200px;
  border-radius: 2px;
  box-shadow: 4px 8px 16px 0 rgba(0,0,0,0.1);
  background: #5CA4EA;
}
```
단순히 파란색 박스를 body의 정 중앙에 놓고 그림자나 배경색을 더한 코드일 뿐이다. 

`.center`를 구현하자.
```css
.center {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%,-50%);
}
```
`.center`도 역시 중앙에 위치해야 함으로 `position: absolute`로 지정해 주었다. 

`input`을 구현하자.
```css
input {
  height: 1.8em;
  width: 220px;
  padding: 0 10px;
  box-shadow: 3px 3px 10px #566270;
  outline: none;
  border: none;
  color: #4F86C6;
}

::placeholder {
	color: #84B1ED;
}
```

높이와 넓이를 적당하게 지정해주고, 텍스트가 너무 인풋박스에 밀착되어지지 않도록 `padding`을 넣어주었다. 

일반적으로 input 박스에 마우스를 클릭해놓으면 테두리가 파란색으로 변하게 되는데 `outline: none`을 하면 파란색 테두리가 사라진다.

input의 `color`를 바꾸어줬는데, 여기서 color란 사용자가 입력한 텍스트만 적용이 된다. 즉 placeholder에는 적용되지 않기 때문에 따로 `::placeholder`지정자로 설정해 줘야 한다. color 뿐만 아니라 font-size같은 일반 디자인도 가능하다.


이제 `#recommend`를 구현하자. 사용자가 텍스트를 입력하면 `input`박스 밑에 나타날 영역이다.
![searchBox5](/public/img/css_challenge/SearchBox5.PNG) 
```css
#recommend {
	margin-top: 1px;
	position: absolute;
	background: white;
	padding: 0 10px;
}

.item {
	height: 1.8em;
	width: 220px;
	outline: none;
}

.item:hover {
	color: #9baec8;
}
.text {
	font-weight: bold;
}
```

주목할 점은 `#recommend`의 `position: absolute`이다.  `abolute`는 뷰포트에 상대적으로 위치가 지정되는 것이 아니라 가장 가까운 곳에 위치한 조상 엘리먼트에 상대적으로 위치가 지정된다.

HTML 코드 일부를 다시 보자.
```html
<div class="center">
    <input type="text" placeholder="Start typing ..."/>
    <div id="recommend" class="invisible">
        <div class="item"><span class="text"></span></div>
        <div class="item">Veritatis et <span class="text"></span></div>
        <div class="item">ut aliquid ex <span class="text"></span> vero eos</div>
    </div>
  </div>
```
즉 `#recommend`의 가장 가까운 곳에 위치한 조상 엘리먼트는 `.center`이고, 바로 윗 이웃 요소로 `input`이 있기 때문에 그 바로 아래 영역에 절대적 위치로 잡히게 된다. 만약 `#recommend`가 `absolute`가 아니라 `relative` 또는 `static`이었다면 절대적인 자리가 잡히지 않기 때문에 바로 윗 이웃 요소인 `input`과 함께 가운데 자리를 차지하려고 침범할 것이다. 우리는 `absolute`를 사용하여 이웃 요소의 자리를 침범하지 않고 해결할 수 있다.

거의 다 왔다. 마지막으로 `#recommed`영역은 최초에 보이지 않아야 하는 영역이므로 이를 처리해주자.
```css
.invisible {
	display: none;
}
```
여기 까지 했다면, 아래와 같은 이미지가 된다.
![searchBox4](/public/img/css_challenge/SearchBox4.PNG)

검색 박스에 텍스트를 넣어봐도 아무 변화가 없다. 이제 간단한 javascript 코딩으로 텍스트가 입력되면 추천 검색어 영역이 표시되고, 동시에 입력한 텍스트가 추천 검색어에도 뜨게 하자.

## JAVASCRIPT
```javascript
const inputBox = document.querySelector("input");
const recommendBox = document.querySelector("#recommend");
const texts = document.querySelectorAll(".text");

inputBox.addEventListener("keyup", (e) => {
	if (e.target.value.length > 0) {
		recommendBox.classList.remove('invisible');
		texts.forEach((textEl) => {
			textEl.textContent = e.target.value;
		})
	} else {
		recommendBox.classList.add('invisible');
	}
})
```

순수 자바스크립트 코드다. 이미 자바스크립트를 조금 다룰줄 안다면 너무 쉬운 난이도라서 딱히 설명할 부분도 없다.

여기까지 했다면 [이 링크](https://codepen.io/CLINNBT/pen/qgjEWp)와 동일한 결과물을 얻을 수 있다. 전체 코드도 [이 링크](https://codepen.io/CLINNBT/pen/qgjEWp)에 있다.
## 끄
읕!