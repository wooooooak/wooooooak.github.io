---
layout: post
title: AAC(Android Architecture Component) ViewModel에 대한 짧은 고찰
category: Android
tags: [mvvm, AAC, ViewModel]
comments: true
---

## 오해의 소지가 많은 AAC ViewModel

안드로이드 아키텍쳐 컴포넌트(AAC)는 ViewModel이라는 추상클래스를 제공한다. 안드로이드 개발자나 또는 아키텍쳐에 관심이 많은 개발자라면 ViewModel이란 단어를 들었을 때 아마도 가장 먼저 MVVM패턴을 떠올릴 것이다. 마이크로소프트에서 처음 MVVM을 선보이면서 ViewModel이란 단어를 **뷰와 모델 사이에서 데이터를 관리하고 바인딩 해주는 요소**로 칭했기 때문이다. 실제 MVVM패턴 에서의 viewModel의 역할이 그렇다.

그럼 구글이 안드로이드 개발의 편의성을 위해 제공하는 ViewModel은 어떨까? 앱 개발의 대세가 MVVM이다 보니 많은 개발자가 이름만 보고서는 MVVM의 ViewModel 일거라고 생각한다. 그러나 일반적인 MVVM의 viewModel과 전혀 상관이없다. 1도 상관이 없다. 심지어 [안드로이드 공식 문서](https://developer.android.com/topic/libraries/architecture/viewmodel)에도 viewModel을 설명하면서 mvvm 패턴을 이야기 하지 않는다. AAC의 뷰모델이 MVVM패턴의 뷰모델이라면 당연히 언급이 있어야 하는데, 없다.

## AAC ViewModel이란?

> The ViewModel class is designed to store and manage UI-related data in a lifecycle conscious way. The ViewModel class allows data to survive configuration changes such as screen rotations.

[안드로이드 공식 문서](https://developer.android.com/topic/libraries/architecture/viewmodel)에서 가장 첫 문장에 나오는 부분이다. 화면 회전같은 환경 변화에서 뷰에 사용되는 데이터를 유지시키기 위한, 라이프사이클을 알고있는 클래스라고 한다. 우리가 알고있는 mvvm에서의 viewModel(뷰와 모델 사이에서 데이터를 관리하고 바인딩해주는)역할과는 설명이 다르다. 즉, 이름만 viewModel이지 MVVM의 viewModel과 전혀 관계가 없는 클래스라는 것이다.~~(개발 입문자 입장에서 자바와 자바스크립트 관계보다 더 혼란..)~~

구글이 viewModel을 잘 못 알고 만든 것일까? 절대 그렇지 않다. 한 가지 잘 못한게 있다면 이름을 헷갈리게도 viewModel로 지었다는점...? 사용자가 화면을 회전 시킬 경우를 생각해보자. 기존 view에는 로그인 된 사용자의 정보를 가지고 있다. 사용자가 화면을 회전 시키면 액티비티는 종료되었다가 다시 생성된다. 그렇기 때문에 사용자의 데이터를 또 다시 받아와야 한다. 그저 화면 회전이 되었을 뿐인데 말이다. 그러나 AAC 뷰모델은 화면을 회전 시켜도 로그인 정보를 그대로 보존해놓기 때문에 다시 유저 로그인 정보를 불러오지 않아도 된다. 이건 정말 편한 기능이다(직접 구현 해보면 얼마나 귀찮고 어려운 작업인지 알게 될것이다). AAC의 ViewModel은 딱 이 역할만 하는 클래스일 뿐이다. ViewModel을 생성하고 거기에 유저 로그인 정보를 넣었을 뿐, 그렇다고 이게 MVVM 패턴이 되지는 않는다.

그런데 생각해보니 조금 이상한 점이 있다. 분명 앱 개발의 대세는 mvvm 패턴이 되어가고 있는데 "mvvm 패턴 샘플 예제"를 검색해서 안드로이드 앱 예제를 보면 대부분 AAC ViewModel을 사용하고 있다. AAC ViewModel은 MVVM의 ViewModel과 전혀 상관이 없다고 했는데.... 인터넷에 올라온 예제들이 모두다 잘못된 예제일까? 그렇게 star를 많이 받은 안드로이드 MVVM 예제 샘플 코드들이 모두 AAC ViewModel을 MVVM의 ViewModel로 착각하고 만든 것일까? 그렇지 않다. AAC의 ViewModel을 MVVM의 ViewModel로써 사용할 수 있다. MVVM의 뷰모델의 역할은 위에서 보았듯이 **뷰와 모델 사이에서 데이터를 관리하고 바인딩해주는 것**이다. 뷰모델이 가지고 있는 데이터를 옵저버블하게 해주고, 뷰에서는 데이터 바인딩으로 그것을 구독하고 있으면 되는 것이다. AAC ViewModel이라고 그게 안될리가 없다. 오히려 화면 회전시 데이터를 유지시켜주는 기능까지 있으므로 더 좋다. AAC ViewModel에 LiveData를 사용하여 바인딩 시킬 수 있다.

안드로이드 개발을 하면서 MVVM패턴을 사용하려고 한다면, 꼭 AAC의 ViewModel을 사용하지 않아도 MVVM구현은 가능하다. 선택은 개발자의 몫이다. 그러나 놓치지 말아야 할 것은, AAC의 뷰모델은 MVVM에서 말하는 ViewModel과 다르다는 것을 아는 것이다. 이걸 모르고 AAC ViewModel을 mvvm ViewModel처럼 사용한다면 개발 도중 어려움이 생길 가능성이 크다. 한 가지 예를 들자면, **AAC ViewModel은 ViewModelProviders를 사용해서 ViewModel을 만드는데, 이렇게 만들어진 뷰모델은 그 액티비티에서 딱 하나만 존재하게 된다. 액티비티 한 개 내에서만 유효한 싱글톤인 셈이다.** 이런 특성은 일반적인 MVVM에서는 강제되는 것이 아니기 때문에 혼란이 올 수 있다.

바로 위에서 "뷰모델은 그 액티비티에서 딱 하나만 존재하게 된다"라고 설명했는데 오해의 소지가 있을수 있을 것 같아서 추가적으로 언급하자면, 이 말이 **뷰 한개에 뷰모델 유형이 딱 한개 존재해야 한다는 것은 아니다.** 예를 들어 SignUpActivity 뷰가 있다고 했을 때, 그에 대응하는 SignUpViewModel 딱 하나만이 존재해야 한다는 것은 아니다. MVVM패턴에서는 뷰와 뷰모델은 1:n 관계이기 때문이다. 개발자는 필요에 따라서 얼마든지 UserPersonalDataViewModel, UserAccountViewModel 등등 여러가지 뷰모델로 나눠서 사용이 가능하다. 다만 UserPersonalDataViewModel을 한번 생성하면, 그 액티비티에서 UserPersonalDataViewModel을 여러번 생성해도 그것은 싱글톤이기 때문에 하나의 객체만 계속 사용된다는 것이다.

## 결론

처음 mvvm을 공부해 나갈 때, 내게 더욱 더 혼란을 주는 몇 가지 강연들이 있었다.

> MVVM 패턴 이야기 하면서 구글 이야기를 한다면 자리를 박차고 나가라. - 강사룡님

또는

> MVVM은 AAC의 ViewModel과 연관성이 없다. AAC ViewModel을 전제로 MVVM을 설명하려고 한다면 단언컨데 우리 회사 1차 면접도 통과하지 못할 것이다.- 정승욱님

AAC ViewModel에서 제대로 알아야겠다고 마음먹기 전에 이런 이야기를 들었다. 그래서 그 당시 나는 "구글이 MVVM을 잘 이해하지도 못하면서 전세계 개발자를 대상으로 ViewModel 컴포넌트를 제공하고 있구나~ "라는 오해를 했다. 그러는 한편, 저 사람들은 자기가 얼마나 잘났길래 감히 구글이 틀렸다고 말하지? 라는 생각을 했다. **그러나 구글도, 강사룡님도, 정승욱님도 틀리지 않았다. 틀린건 오직 나 뿐이었다.** 구글이 ViewModel을 내놓았으니 당연히 mvvm의 viewModel이겠거니~ 라며 얕게 생각했던 탓에 구글이 맞는지, 저 강사분들이 맞는지 헷갈렸던 것이다. 정말이지 강사룡님 말씀처럼 MVVM 패턴 이야기 하면서 그것과 전혀 아무런 관계도 없는 구글의 ViewModel를 설명하는 사람이 있다면 그자리는 피하는게 좋을 수도 있다. 또한 당연히 면접자리에서 MVVM을 설명해보라는 질문에 쌩뚱맞게 AAC ViewModel을 말한다면 어느 회사에서든 떨어지지 않을까.

구글은 MVVM과 전혀 상관없는, 그저 이름만 ViewModel인 클래스를 제공했을 뿐 "MVVM의 ViewModel이니까 가져다 쓰세요~"라고 말한 적이 없다. 그냥 나처럼 멍청한 개발자들이 이름만 보고 헷갈려할 뿐....~~(솔직히 구글도 너무하긴 했다 이름이 viewModel이라니)~~

##참고자료

- [안드로이드 아키텍처 컴포넌트, ViewModel 이해하기](https://medium.com/@jungil.han/%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8-viewmodel-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-2e4d136d28d2)
- [구글 공식문서 - viewModel](https://developer.android.com/topic/libraries/architecture/viewmodel)
- [권태환님의 블로그](https://thdev.tech/androiddev/2018/08/05/Android-Architecture-Components-ViewModel-Inject/)
- [MVVM with Grab Architecture - 정승욱님](https://tv.naver.com/v/4637223?query=mvvm&plClips=false:4637223:3547873:3390780:4635548)
