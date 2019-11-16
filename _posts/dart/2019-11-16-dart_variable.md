---
layout: post
title: Dart variable(다트 변수)
category: Dart
tags: [dart, flutter]
comments: true
---

## 변수 선언

변수를 선언할 때 사용하는 키워드는 아래와 같다.

- var : 변수(mutable)
- final : 변하지 않는 값(immutable + 런타임 선언 가능)
- const : 변하지 않는 값(imutable + 컴파일 타임 선언)
- dynamic : 자바로 치면 Object
- 각종 타입들

다트(Dart)는 Type Safe한 언어이지만, 코틀린과 동일하게 타입 추론이 가능한 언어이므로 선언시에 타입을 명시하지 않아도 된다.

```dart
void main() {
    var a = 1;
    final b = 2;
    cosnt c = 3;
    dynamic d = 4;
    d = 'four';
    String e = 'five'; // 타입 명시
}
```

`final`과 `const`의 차이점에 대한 내용은 [Dart: final 과 const](https://medium.com/dartlang-korea/dart-final-%EA%B3%BC-const-bc8c6c024ef4)를 참고하자.

## Type

언어에서 기본적으로 제공하는 타입은 아래와 같다.

- Number
- String
- Booleans
- List
- Set
- Map
- Runes
- Symbols

몇 가지 살펴볼만한 타입들을 알아보자.

### Number

Dart의 Number에는 `int`와 `double`이 있다. 둘 모두 `num`이라는 추상 클래스를 상속 받았다.
한 가지 주의해야할 것은, `double`에 `int`형 값을 넣으려고 할 경우 자동 형변환을 해주지 않는다는 점이다.

```dart
void main(){
    int a = 1;
    double b = a; // 자바에선 가능하지만 다트에선 불가능
}
```

명시적으로 형변환을 해주거나, `daouble` 타입을 num으로 해주어야 한다.

```dart
void main(){
    // 명시적 타입 캐스팅
    int a1 = 1;
    double b1 = a1 as double;

    // num 타입으로 받기
    int a2= 1;
    num b2 = a;
}
```

### String

문자열을 선언 할 때는, single qoute, double qoute를 구분하지 않는다.

```dart
void main() {
    final str1 = 'hello';    // single qoute
    final str2 = "hello2";   // double qoute
}
```

modern programming language답게 문자열 템플릿을 제공한다.

```dart
void main() {
    final myName = 'yongjun';
    final strTmeplate = 'Hello, My name is ${yongjun}';
}
```

### List & Set & Map

```dart
void main() {
    // List
    var myList1 = [1, 2, 3, 4, 5];
    List<String> myList2 = ['one', 'two', 'three', 'four', 'five'];
    var myList3 = <String>['one', 'two', 'three', 'four', 'five'];

    // Set
    var set1 = {1, 2, 3};
    Set<String> set2 = Set<String>();

    // Map
    var map2 = {
        'key1' : 'myValue1',
        'key2' : 'myValue2,
    };
}
```
