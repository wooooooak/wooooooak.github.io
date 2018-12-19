---
layout: post
title: express와 Typescript 사용 중 req에 임의의 값 넣기
category: Node.js
tags: [typescript, node.js, express]
comments: true
---
## req에 임의의 값이라니?
Typescript와 express를 사용하여 웹 서버를 구축하다보면, req 객체에 임의의 값을 넣어서 사용해야 할 경우가 종종 있다.

예를 들어 사용자 인증에 jwt를 사용하고, 인증이 필요한 addPost라는 라우터가 있다고 가정하면, auth 미들웨어가 사용자 인증을 처리하기 위해 addPost보다 먼저 실행될 것이다.

**사용자 요청 -> auth미들웨어 -> appPost 작동**

addPost는 auth 미들웨어가 무사히 통과되어야만 도달할 수 있다. auth 미들웨어는 사용자로 부터 받은 jwt토큰을 분석한 사용자 데이터를 req에 넣어 addPost라우터로 넘겨준다. 그럼 addPost는 req로 부터 사용자 데이터를 가져다 쓸 수 있다. 

auth 미들웨어의 코드 일부를 보자면 아래와 같다.
```typescript
import * as express from 'express';
...

const authMidlleWare = 
        async (req: express.Request, res: express.Response, next: express.NextFunction) {
                                    
                ...

const decodedUser: User = await decodeToken(token);
req.decodedUser = decodedUser;
next();

                ...
                                    }
```

## 기본설정은 에러가 난다
이 auth 미들웨어에서 에러가 난다. req객체 타입은 `express.Request`인데 이 타입에는 decodedUser라는 속성이 없기 때문에 타입스크립트는 에러를 낸다.

때문에 우리는 express가 제공하는 타입에 약간 정보를 더해야 한다(customType이라고도 한다). 간단히 decodedUser라는 속성을 넣어주기만 하면 된다.

그러기 위해서 우선 루트에 있는 tsconfig.json을 수정해주어야 한다. 우리가 수정할 부분은 "typeRoots"라는 속성이다.
```js
{
   "compilerOptions": {
    //    "typeRoots" : []
   }
}
``` 
기본 설정은 이렇게 주석처리가 되어있고, 기본 값으로 `"./node_modules/@types"` 디렉터리에서 타입을 읽어온다. 우리가 `npm install @types/express`로 설치하면 express의 타입은 `./node_modules/@types`디렉터리에 존재하게 된다.

이제 우리가 할 일은, express타입을 커스터마이징 할 파일을 한개 만들고, 그 파일을 포함하는 디렉터리 위치를 `typeRoots`에 입력하면 된다. 예를 들어 `./src/customType`폴더에서 custom type 파일을 작성할 계획이라면 tsconfig.json파일은 아래와 같다.
```js
{
   "compilerOptions": {
      "typeRoots": ["./node_modules/@types","./src/customType"], 
   }
}
``` 
(typeRoots의 주석이 풀리면 기본값인 ./node_modules/@types도 없어지므로 수동으로 추가해준다.)

**참고: [타입스크립트 공식문서(?)](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html)를 보면 기본 타입 루트 디렉터리가 `./typings`로 되어있는데 이는 옛날에 존재했던 디렉토리라고....지금은 ./node_modules/@types로 바뀐것이라고 하니 알고있자.**

## 커스텀 파일 만들기
이제 `./src/customType` 폴더에서 `express.d.ts`라는 파일을 만들어 `req`타입에 `decodedUser`속성을 추가해보자!

```typescript
// ./src/customType/express.d.ts
import { User } from '../models/User';

declare global {
	namespace Express {
		interface Request {
			decodedUser?: User;
		}
	}
}
```
이 코드는 [Extend Express Request object using Typescript (stackoverflow)](https://stackoverflow.com/questions/37377731/extend-express-request-object-using-typescript) 에서 참고 했다. 

이로써 `express.Request` 타입에는 `decodedUser`가 추가되었다. 이제 addPost에서 `req.decodedUser`를 호출 할 수 있다.

## 참고자료

* [Typescript 공식문서](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html)