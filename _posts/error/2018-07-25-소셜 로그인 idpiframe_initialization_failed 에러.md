---
layout: post
title: idpiframe_initialization_failed 에러
category: error
tags: [react-google-login, social-login, chrome]
comments: true
---

## 증상
구글 개발자 콘솔에서 앱 등록을 마친 후 react-google-login로 구글 로그인 연동을 시도하던 중, 로그인 버튼을 클릭하면 idpiframe_initialization_failed 에러가 떳다.

## 원인
 원인을 찾아 구글링을 해보니 공식적으로 에러 원인이 아래와 같이 적혀있었다.
![reason](/public/img/0725/chromeerror.JPG)

따라서 [이 자료](https://support.cloudhq.net/how-to-enable-3rd-party-cookies-in-google-chrome-browser/)를 따라 third party cookies enabled를 설정하러 갔는데 이미 설정이 되어 있는 상황이었다.... 당황했다. 혹시나 해서 파이어폭스에서 실행해보니 정상 작동했다. **아무튼 구글 콘솔 설정이 잘못된건 아니란 뜻.**

그래서 더 열심히 열심히 구글링했다.

## 해결방법
구글 크롬 설정 -> 고급 -> 개인정보 및 보안 -> 인터넷 사용 기록 삭제 -> 모든 쿠키 와 캐시된 이미지 또는 파일을 삭제!

이 생각은 [스택 오버플로우 Google API: Not a valid origin for the client: url has not been whitelisted for client ID “ID”
](https://stackoverflow.com/questions/43964539/google-api-not-a-valid-origin-for-the-client-url-has-not-been-whitelisted-for)에서 얻었다.