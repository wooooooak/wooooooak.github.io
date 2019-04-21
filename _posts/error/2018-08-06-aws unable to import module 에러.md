---
layout: post
title: Cloud Watch에서 unable to import module Error
category: error
tags: [AWS, serverless]
comments: true
---

## 증상
클라우드 워치에서
```
Unable to import module 'dist/app': Error
at Function.Module._resolveFilename (module.js:547:15)
at Function.Module._load (module.js:474:25)
at Module.require (module.js:596:17)
at require (internal/module.js:11:18)
at Object.<anonymous> (/var/task/dist/router/index.js:11:13)
at Module._compile (module.js:652:30)
at Object.Module._extensions..js (module.js:663:10)
at Module.load (module.js:565:32)
at tryModuleLoad (module.js:505:12)
at Function.Module._load (module.js:497:3)
```
에러가 난다.

## 원인
클라우드워치에서 unable to import module "dist/app" 과 같이 에러가 뜨는 것은,
dist/app을 임포트하지 못하는게 아니라 dist/app에서 실행중인 어떤 것이 import가 안된다는 것. 나는 말그대로 dist/app을 import하지 못하는게 문제라고 생각해서 시간을 많이 뺏겼었는데 결과는 dist/app에서 의존하기 위해 import하는 어떤 것의 경로가 잘못 된 것이었다.
꼭 dist/app에서 바로 import 하는게 아니어도 그렇다. 나의 경우에는, 프로젝트 폴더 구조는 아래 이미지와 같았다.

![error1](/public/img/unable/unable.JPG)

핵심 역할을 하는 app.js에서의 import 구문은 틀린게 없었는데, router/index.js에서 
```
import user from './user';
import auth from './auth';
```
이렇게 import를 하는데 폴더 이름이 User, Auth 처럼 되있어서 import 하지 못한 것이 클라우드 워치에서 에러를 unable to import module "dist/app" 이렇게 뿜어준 것이다.(너무 불친절한 에러 표시다ㅠㅠ)

## 해결
간단히 build/app의 import가 정확이 이뤄질 수 있도록 폴더 이름을 user와 auth로 바꿔서 정상 작동하게 되었다. 

나는 src/ 디렉토리를 build 해서 dist파일을 만드는데, babel의 build는 src/ 에서 기존 폴더의 폴더명이 바뀔 경우, dist/ 에서 기존 폴더명을 지우고 새로운 폴더를 만드는게 아니라 기존에 이름이 바뀌기전 폴더를 그대로 놔둔 채로 새로운 폴더를 추가할 뿐이다. **혹시 모르니 자꾸 에러가 난다면 deploy하기 전에 build일을 직접 눈으로 보자!**