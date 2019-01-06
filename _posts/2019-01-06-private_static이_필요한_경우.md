---
layout: post
title: private static이 필요한 경우
category: Java
tags: [basic, java, scope]
comments: true
---

자바를 공부하다 보면 가끔 private static을 볼 수 있다. 공부 초창기를 떠올려보면 접근 제한자인 public, protected, private들도 헷갈렸는데 static까지 붙어버리니 이게 도대체 뭔가싶은 생각도 들었었다. 도대체 private static은 언제 쓰이는 것이고 왜 쓰이는 걸까? 

## private method가 필요한 이유와 동일하다.
결과부터 말하자면 private method가 필요한 이유와 동일하다. 초보자 입장에서 private static 이란게 생소해서 그럴 뿐(~~뭔가 어려워 보이고 대단한 역할을 할 것 처럼 보인다~~), 아주 간단하고 쉬운 내용이다.

그럼 private method가 필요한 코드를 보자.
![profile](/public/img/java/private_static2.png)

`doStudy()`는 public 접근 제한자를 가지게 함으로써 다른 클래스파일에서도 사용이 가능하지만 `doStudy()`구현부의 영어 공부와 java공부는 이 Person클래스 내에서만 사용하면 된다. 그래서 private method면 된다.

`private static`도 동일하다. `doStudy()`가 `static` 함수라면, `studyEnglish()`와 `studyJava()`도 static이어야 하므로 `private static`을 사용하는 것 뿐이다.

![profile](/public/img/java/private_static3.png)

## 결론
정말 이게 끝이다. 처음에는 이게 도대체 뭘까 엄청 대단한 역할을 할 것 같고, 내가 쓸일이나 있을까? 하며 겁을 내곤 하지만, 알고보면 이렇게 간단한 것이다.

