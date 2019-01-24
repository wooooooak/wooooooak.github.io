---
layout: post
title: IntelliJ에서 git clone시 프로젝트 인식하기
category: 개발 환경?
tags: [intelliJ, java]
comments: true
---

Java 프로젝트를 git clone하고 인텔리제이에서 작업하려고 하면, 인텔리제이가 해당 코드를 소스코드(프로젝트)로 인식하지 못해서 실행은 물론 build조차 되지 않는 현상이 발생한다.

인텔리제이 오른쪽 윗 부분을 보면 아래 사진처럼 설정을 더하는 칸이 있다.
![컨피그이미지](/public/img/env_img/intellij1.PNG)

이 부분을 클릭해 보았지만 해결책은 아니었다.

## 프로젝트로 인식하기
폴더 디렉토리중에 소스코드가 담긴 루트 디렉토리(src)를 우클릭 하면 아래 이미지와 같은 창이 뜬다.
![컨피그이미지2](/public/img/env_img/intelli2.png)

해당 소스를 이렇게 소스트리로 명시적으로 설정해 주어야 인텔리제이가 소스코드로 인식한다.