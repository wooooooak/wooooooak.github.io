---
layout: post
title: fetch사용시 유의 사항 (json() 함수 사용하기)
category: javacript
tags: [javacript, network, basic]
comments: true
---

js 개발자들은 network request 요청이 필요할 경우 대부분 axios.js나 기타 다른 라이브러리를 쓰는 것 같다. js에서 기본적으로 제공하는 fetch라는 함수가 있지만, fetch를 사용할 경우 응답받은 body데이터의 form을 직접 변환해야 하는데, 이 단계를 자동으로 해주기 때문이고, 에러 핸들링도 편하기 때문이다.

나 역시도 axios나 기타 HTTP requests libarary를 다루는 방법은 잘 알고있다. 그러나 vanilla JS로 사이드 프로젝트를 하면서 fetch를 사용하게 되었는데, axios가 개발자 몰래 자동으로 해준 한 단계가 무엇인지 몰라서 막힌 부분이 있었다. 

## fetch로는 데이터를 바로 사용할 수 없다.
fetch를 사용할 땐 두 단계를 거쳐야 한다. 먼저 올바른 url로 요청을 보내야 하고, 바로 뒤에오는 응답에 대해 json()을 해줘야 하는 것이다(axios는 json()과정을 자동으로 해주는 셈이다).

```javascript
fetch(`https://api.openweathermap.org/data/2.5/weather?lat=${lat}
&lon=${lon}&appid=${API_KEY}`)
    .then(res => console.log(res));

```
위의 코드는 [openweathermap](https://openweathermap.org)라는 사이트에서 현재 위치의 날씨를 가져오는 코드이다. 실행하면 아래와 같은 결과가 나온다.

![fetch](/public/img/jsImage/fetch1.png)

그러나 이것은 내가 원했던 정보가 아니다. [openweathermap](https://openweathermap.org)에 의하면 내가 반환 받아야 할 데이터는 아래와 같다.
```javascript
{"coord":{"lon":139.01,"lat":35.02},"weather":[{"id":800,"main":"Clear","description":"clear sky","icon":"01n"}],"base":"stations","main":{"temp":285.514,"pressure":1013.75,"humidity":100,"temp_min":285.514,"temp_max":285.514,"sea_level":1023.22,"grnd_level":1013.75},"wind":{"speed":5.52,"deg":311},"clouds":{"all":0},"dt":1485792967,"sys":{"message":0.0025,"country":"JP","sunrise":1485726240,"sunset":1485763863},"id":1907296,"name":"Tawarano","cod":200}
```
res의 body에 담겨있을 것 같지만 그렇지 않았다. 이 body는 스트림이고, 데이터가 완전히 다 받아진 상태가 아닌것이다. 이 걸로는 날씨 데이터를 받아와 데이터를 뿌려줄 수가 없다. 그래서 **res 객체의 json()이라는 메서드를 사용한다.** json()은  Response 스트림을 가져와 스트림이 완료될때까지 읽는다. 그리고 다 읽은 body의 텍스트를 Promise형태로 반환한다. 

따라서 서버가 주는 json데이터를 사용하고 싶다면 아래와 같은 코드를 작성해야 한다.
```javascript
fetch(`https://api.openweathermap.org/data/2.5/weather?
lat=${lat}&lon=${lon}&appid=${API_KEY}&units=metric`)
    .then((res) => {
        return res.json(); //Promise 반환
    })
    .then((json) => {
        console.log(json); // 서버에서 주는 json데이터가 출력 됨
    });
```

반면 axios를 사용할 경우 res.json()단계를 넘어가도 좋다. 그러나 axios로 받아오는 데이터는 서버에서 넘겨주는 body데이터 외에 부가적인 정보도 포함되어 있기 때문에 원하는 data만 골라서 사용해야 한다. 이부분은 [axios 공식문서](https://github.com/axios/axios)를 보는편이 낫겠다.

## 참고자료
* [Fetch vs. Axios.js for making http requests (영문)](https://medium.com/@thejasonfile/fetch-vs-axios-js-for-making-http-requests-2b261cdd3af5)
* [Body.json() _ MDN](https://developer.mozilla.org/ko/docs/Web/API/Body/json)