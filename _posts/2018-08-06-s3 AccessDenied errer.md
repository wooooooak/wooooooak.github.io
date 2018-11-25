---
layout: post
title: AWS s3 AccessDenied error
category: error
tags: [AWS, s3]
comments: true
---

##  증상
react로 된 정적 파일을 s3로 업로드 하는데 자꾸 AccessDenied error가 떠서 시간을 많이 잡아먹었다.

![s3 error](/public/img/s3error/s3error.JPG)

분명히 시키는 대로 Bucket Permissions Policy도 아래와 같이 제대로 설정 했는데도 불구하고 에러가 뜬것이다.
```
{
    "Version": "2012-10-17",
    "Id": "Policy1533495783554",
    "Statement": [
        {
            "Sid": "Stmt1533495777402",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::elebooks-frontend/*"
        }
    ]
}
```
## 원인
내가 멍청한게 원인이라고 할 수 밖에 없다.... 접속 url을 잘 못 알고있던 것이다. [aws 공식 문서](https://docs.aws.amazon.com/ko_kr/AmazonS3/latest/dev/UsingBucket.html)의 글을 대충 읽고난 후, 내 버킷의 react 앱에 접속 하려면 **http://s3-aws-region.amazonaws.com/bucket** 혹은 bucketname.s3.ap-northeast-2.amazonaws.com 이런 식의 url로 접속 해야 하는 줄 알고 있었던 것이다.

## 해결
s3에 올린 정적 웹사이트 접속의 올바른 url은 아래와 같은 곳에 나와있었다.
![solution](/public/img/s3error/sol.JPG)

## 끝
저번에도 같은 실수 때문에 시간을 날렸었기 때문에 한번 적어두었다.
