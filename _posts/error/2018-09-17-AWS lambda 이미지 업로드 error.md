---
layout: post
title: AWS lambda - Image upload content-type support error
category: error
tags: [lambda]
comments: true
---

## 증상
lambda로 서버를 운영하는 도중 multer-s3를 사용해 이미지를 S3로 업로드를 할 때, 이미지는 정상적으로 올라가지만 올라간 파일을 열어보면 "이 형식은 지원되지 않는 형식인 것 같습니다" 라는 에러가 떴었다. local 환경의 서버를 통한 이미지는 잘 열렸는데 람다를 통해서 업로드 된 파일만 읽히지 않는 것이었다.

## 원인
lambda는 기본적으로 binary type support가 되지 않는다고 한다. 따라서 api gateway에 Binary type support를 설정해 줘야 했다.

## 해결
수동으로 Binary support를 추가하는 방법도 있고 플러그인을 설치해 serverless-framwork에 설정하는 방법이 있었다. 나는 플로그인으로 설정했다. 

플러그인은 serverless-apigw-binary 이고 npm이나 yarn으로 설치할 수 있다. 이후 serverless.yml에 추가 설정을 아래와 같이 했다
```
custom:
  apigwBinary:
    types:           #list of mime-types 
      - '*/*'
```

그리고 aws-serverless-express에서 설정해 줘야 할 것들이 있다. 이 라이브러리의 깃헙 이슈에서 얻은 정보로 아래와 같이 설정해 줘야 한다.
```javascript
// lambda.js
'use strict'
const awsServerlessExpress = require('aws-serverless-express')
const app = require('./app')

// NOTE: If you get ERR_CONTENT_DECODING_FAILED in your browser, this is likely
// due to a compressed response (e.g. gzip) which has not been handled correctly
// by aws-serverless-express and/or API Gateway. Add the necessary MIME types to
// binaryMimeTypes below, then redeploy (`npm run package-deploy`)
const binaryMimeTypes = [
  'application/javascript',
  'application/json',
  'application/octet-stream',
  'application/xml',
  'font/eot',
  'font/opentype',
  'font/otf',
  'image/jpeg',
  'image/png',
  'image/svg+xml',
  'text/comma-separated-values',
  'text/css',
  'text/html',
  'text/javascript',
  'text/plain',
  'text/text',
  'text/xml'
]
const server = awsServerlessExpress.createServer(app, null, binaryMimeTypes)

exports.handler = (event, context) => awsServerlessExpress.proxy(server, event, context)
```

## 참고 자료
* [aws-serverless-express 이슈](https://github.com/awslabs/aws-serverless-express/issues/99)
* [Does not play nice with multer](https://github.com/dougmoscrop/serverless-http/issues/34)