---
title: "[Flutter] 토이 프로젝트 후기 - (1)"
date: 2019-05-17T10:42:36+09:00
weight: 1
aliases: []
tags: ["flutter"]
author: ["Maru"]
showToc: true
TocOpen: true
draft: false
hidemeta: false
disableShare: false
comments: false
---

[react-native](https://facebook.github.io/react-native/)가 출시 된 지 [4년](https://en.wikipedia.org/wiki/React_Native)이 지났습니다.

모바일 애플리케이션 개발에서 *크로스 플랫폼 동시 개발*을 가능하게 해준 react-native는 굉장히 파워풀한 기술인 거 같습니다.

그렇게 2년이라는 시간이 지나 react-native와 마찬가지로 iOS와 Android를 동시에 개발할 수 있는 프레임워크인 [flutter](https://flutter.dev/)가 처음 등장했습니다.

구글이 만든 프레임워크고 밀어 주려고 하는 기술인 거 같아, 경험해 볼 겸 간단한 앱을 만들기로 했습니다.

이번 글에서는 flutter로 앱을 만들면서 느꼈던 점들을 공유해보겠습니다.

글은 1편과 2편으로 나누어 작성하겠습니다.

<br>

## 📑 간단한 기획

저는 웹에서 토이 프로젝트를 할 때 보통 블로그 형식을 구현했었습니다. 그래서 Flutter로 간단한 메모 앱을 만들어 보기로 했습니다.

하단 내비게이션으로

1. 글 쓰는 화면
2. 글 목록 화면
3. 더 보기 화면

세 화면에 접근할 수 있게 하고 글 데이터는 sqlite를 사용하여 저장하도록 기획했습니다. 디자인은 따로 하지 않았습니다.
<br>

## 👾 Flutter 설치

설치는 Flutter Doc에서 쉽게 할 수 있습니다. ([https://flutter.dev/docs/get-started/install](https://flutter.dev/docs/get-started/install))

Windows, Linux, MacOS는 물론 Chrome OS까지 지원한다고 합니다.

iOS 앱을 만들고 싶다면, MacOS에서 Xcode를 설치해야 하고 따로 설정을 해줘야 합니다. 자세한 내용은 Doc을 [참고](https://flutter.dev/docs/get-started/install/macos)하시면 됩니다.

문서에서 Editor는 Android Studio와 Visual Studio Code를 추천합니다. Android Studio는 Plugin을 Visual Studio Code는 Extension을 설치합니다.

저는 Android Studio를 사용했습니다.

설치가 다 끝나고 flutter 명령어인

```
flutter doctor
```

를 터미널에서 실행하면 flutter 개발에 필요한 사항들이 설치 되었는지 확인 할 수 있습니다.
만약 안드로이드 앱으로만 개발할 예정이면, flutter를 위한 Xcode 설치가 안되어 있어도 flutter는 문제 없이 돌아갑니다.

이제 flutter 개발을 위한 준비가 끝났습니다.

<br>

## 🔥 Hot Reload

첫 프로젝트를 생성하면, 예제 코드가 있길래 바로 빌드를 해봤습니다.

빌드 속도는 일반적으로 AOS, iOS를 빌드 할 때랑 비슷한 속도로 빌드 되었습니다.

이렇게 느린 빌드 속도를 위해 flutter도 react-native처럼 **Hot Reload** 기능(개발 속도를 빠르게 해주는 아주 좋은 기능입니다)을 제공 해줍니다.

앱 개발 경험이 없으신 분들을 위해 살짝 저의 경험을 적어보자면... iOS 빌드 속도가 너무 느리다 보니깐 같이 개발하던 개발자 분과 "코드를 몇 줄을 고치고 나서 빌드를 해야 할까?"라는 주제로 토론을 했던 경험이 있습니다;;

_Hot Reload_ 기능은 _크로스 플랫폼 동시_ 개발 다음으로 flutter와 react-native를 선택해야하는 이유가 될거 같습니다.

<br>

## 🎯 Dart 알아보기

Flutter는 [Dart](https://dart.dev/)라는 언어를 사용합니다._(둘 다 구글에서 만들었다고 합니다...)_

Dart의 첫 인상은 Java 같기도 하고 JS 같기도 한 언어였습니다.

Dart에서 좀 인상 깊었던 표현은 *private*을 표현하는 방법입니다. Java와 비교해도록 하겠습니다.

Java는 private을 다음과 같이 표현합니다.

```Java
// Java
private int num = 0;
```

Dart는 private을 명시해주지 않고, 변수에 `_`를 붙여줍니다.

```Dart
// Dart
int _num = 0;
```

개인적으로는 마음에 들었던 표현이었습니다.

Dart를 간단하게만 훑어보고, 기능 구현을 우선 했기 때문에 Dart에 대한 자세한 얘기는 다음에 기회가 있다면 글로 적어보도록 하겠습니다.

<br>

항상 개발을 위한 환경 세팅은 쉽게 끝나지 않았습니다. 다행히 flutter 환경 세팅은 특별한 문제 없이 진행되었던 거 같습니다.
이번 글은 여기서 마치고, 기능 개발과 UI 구현에 대한 내용은 2편에서 다루도록 하겠습니다. 감사합니다.

<br>

---

**출처**

- [https://flutter.dev/docs](https://flutter.dev/docs)
