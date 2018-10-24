---
layout: post
title: 이벤트 위임(event delegation) feat.bubbling and capture
category: javacript
tags: [basic, event]
comments: true 
--- 

## 이벤트 버블링(Event Bubbling)
브라우저는 특정 element(div 등)에서 이벤트(click 등)가 발생했을 때, 그에 해당하는 이벤트를 자신의 부모 요소를 포함해 최상위 element인 document 요소까지 이벤트를 전파시킨다.

간단한 코드를 보자면,

```css
// css
#one{
  height: 300px;
  width: 300px;
  background-color: green;
}
#two{
  height: 200px;
  width: 200px;
  background-color: yellow;
}
#three{
  height: 100px;
  width: 100px;
  background-color: blue;
}
```

```html
<body>
  <div id="one">
  one
    <div id="two">
     two
     <div id="three">
        three
      </div>
    </div>
  </div>
</body>
```

```javascript
var divThree = document.getElementById("three");
var divTwo = document.getElementById("two");
var divOne = document.getElementById("one");

divThree.addEventListener('click', printLog, {capture: true});
divTwo.addEventListener('click', printLog);
divOne.addEventListener('click', printLog, {capture: true});

function printLog(event) {
	console.log(event.currentTarget.id);
}
```

이렇게 하면 결과물은 아래 이미지와 같다.
![event1](/public/img/event/event1.JPG)

여기서 가장 아래 요소인 파란색 three 영역을 클릭하면 콘솔에는 아래와 같이 출력된다.
```
three
two
one
```

three만 클릭했음에도 불구하고 two와 one이 출력되었다. 이유는 이벤트 버블링 때문인데 three가 클릭될 때 부모요소부터 body까지 click이벤트를 전파시키는 것이다. 여기서 two를 클릭한다면 결과는
```
two 
one
```
일 것이고, 만약 two 요소의 이벤트가 click이 아니라 다른 이벤트가 등록이 되어있다면 그 것은 실행되지 않는다. 이런 원리로 인해 부모 요소에서 자식 요소의 이벤트를 감시할 수 있다. 그 것을 이용하는 것이 이벤트 위임(delegation)이다.



## 이벤트 위임(delegation)
버블링의 원리를 이용해서, 자식 요소가 많을 경우 자식 요소 하나하나에 이벤트를 붙이지 않고, **부모 요소에서 자식 요소의 이벤트들을 제어하는 방식**이다.

```html
<body>
  <ul id="list">
   <li id="one">
     one
   </li>
   <li id="two">
     two
   </li>
   <li id="three">
     three
   </li>
  </ul>
</body>
```
```javascript
var list = document.getElementById("list");


list.addEventListener('click', printLog);


function printLog(event) {
 console.log(event.target.id);
}
```

![event2](/public/img/event/event2.JPG)

위의 경우 li 요소에 이벤트를 하나하나 등록하지 않고도 원하는 이벤트 처리를 할 수 있다. one을 클릭하면 one이 출력되고, two를 클릭하면 two가 출력된다.
li 요소를 클릭했을 때, bubbling으로 인해 부모의 click이벤트가 실행되기 때문이다. 

이렇게 이벤트 위임을 하면 장점이 있다.
* 동일한 여러개의 이벤트를 한 곳에서 관리하기 때문에 관리가 수월해진다.
* 동적으로 이벤트를 매번 추가하지 않기 때문에 메모리 사용이 효율적이다.

## 이벤트 캡쳐(capture)
이벤트 캡쳐는 이벤트 버블과 반대 방향으로 진행되며, 최상위 요소로부터 이벤트가 발생한 요소를 찾아가는 방식이다. 물론 찾아가는 과정에서 동일한 이벤트를 실행한다.

사용하는 방법은 {capture: true} 옵션을 주는 것이다.
```javascript
var divThree = document.getElementById("three");
var divTwo = document.getElementById("two");
var divOne = document.getElementById("one");

divThree.addEventListener('click', printLog, {capture: true});
divTwo.addEventListener('click', printLog, {capture: true});
divOne.addEventListener('click', printLog, {capture: true});

function printLog(event) {
	console.log(event.currentTarget.id);
}
```

동일하게 three를 클릭하면 bubbling때와 달리
```
one
two
three
```
가 출력된다. 만약 코드가 아래와 같다면?
```javascript
var divThree = document.getElementById("three");
var divTwo = document.getElementById("two");
var divOne = document.getElementById("one");

divThree.addEventListener('click', printLog, {capture: true});
divTwo.addEventListener('click', printLog); // 여긴 capture하지 않음
divOne.addEventListener('click', printLog, {capture: true});

function printLog(event) {
	console.log(event.currentTarget.id);
}
```

three를 클릭했을 때 결과는 
```
one
three
two
```
이다. 즉, capture를 할 요소들만 캡쳐가 되고, 그렇지 않은 two같은 경우는 평소와 같이 버블링 되어서 three가 실행된 이후에 실행된다.
