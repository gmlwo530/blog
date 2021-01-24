---
title: "[Flutter] 토이 프로젝트 후기 - (2)"
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
comments: true
---

1편에서 이어서 2편은 프로젝트를 진행 과정을 소개하고 후기를 마무리 하겠습니다.

## 🗂 Flutter

1. **Widget**

Flutter에서 대부분의 UI들은 Widget Class에 속해 있습니다.
예를 들어, [AlertDialog](https://api.flutter.dev/flutter/material/AlertDialog-class.html) 문서를 보면 AlertDialog가 어디에 속해있는지 확인 할 수 있습니다.

2. **StatelessWidget & StatefulWidget**

React를 사용해보신 개발자 분들에게는 친숙한 개념일 수도 있습니다. Flutter의 모든 UI 즉 Widget은 상태(state)를 가집니다.

Widget이 StatelessWidget을 상속 할 때는 상태가 변하지 않아도 되는 UI로 그려집니다.

Widget이 StatefullWidget을 상속 할 때는 상태가 변하는 UI로 그려집니다. Flutter에서 제공하는 `setState` 함수를 통해서 widget의 상태를 변화 시킬 수 있습니다. (최근에 알게 되었는데 setState 말고도 widget의 상태를 관리해주는 방법들이 제공된다고 합니다. 특히 [provider](https://github.com/google/flutter-provide)라는 프레임워크가 2019 Google I/O에서 언급이 되었는데, 굉장히 잘 만들었으니 꼭 사용하라고 언급 되었습니다.)

이전에 react 개발 경험이 있어 state가 낯설지 않았습니다. react 개발을 했을 때 또 느꼈지만, ui의 state를 관리해줄 수 있다는 점은 굉장히 좋은 방법인거 같습니다.

<br>

## 🗄 Save Data

앱을 간단하게 하기 위해서 데이터 저장은 sqlite를 이용해서 디바이스 내부에 저장히기로 결정했습니다. Flutter에서 sqlite를 이용하는 방법을 찾아보니, package를 이용하는게 제일 편리하다고 판단하여, [sqflite](https://github.com/tekartik/sqflite)라는 package를 추가 했습니다.

flutter가 입에 오르기 시작한지 얼마되지 않은 만큼, package들도 생겨나기 시작한지 얼마 안된 거 같습니다. star 갯수나 doc의 완성도가 좀 떨어지는 느낌을 받았지만 package 덕분에 쉽게 sqlite를 사용 할 수 있었습니다.

```dart
  import 'package:path/path.dart';
  import 'package:path_provider/path_provider.dart';
  import 'package:sqflite/sqflite.dart';

  ...

  initDB() async {
    ...

    return await openDatabase(path, version: 1, onOpen: (db) {
    }, onCreate: (Database db, int version) async {
      await db.execute("CREATE TABLE Line ("
          "id INTEGER PRIMARY KEY,"
          "content TEXT,"
          "created_at INTEGER"
          ")");
    });
  }

  getLine(int id) async {
    final db = await database;
    var res = await db.query("Line", where: "id = ?", whereArgs: [id]);
    return res.isNotEmpty ? Line.fromJson(res.first) : null ;
  }
```

- package를 사용해도 sql 문구를 통해 Database를 만들고, data를 불러와야 했습니다. `,` 같은 사소한 오타가 적지 않게 개발 시간을 많이 잡아 먹었습니다.
- data를 가지고 와서 json으로 변경을 해주어야 했습니다. 이 과정에 `dynamic`, `factory`라는 개념을 사용 했는데, 정확히 이해를 하지 못하고 구현했습니다. 나중에 기회가 되면 공부를 해봐야겠습니다.

프로젝트하면서 제일 스트레스 받고, 시간이 오래걸렸던 부분이였습니다. 특히 db를 불러오려면 dart의
[Future](https://dart.dev/tutorials/language/futures) 개념을 이해해야 하는데, 빨리 끝내고 싶어 모르는 상태에서 개발한다고 삽질 좀 했습니다. 역시 시간을 아낄려면, 미리 공부를 해야합니다...

## 🏠 Home

앱을 켰을 때 첫 화면은 글을 바로 쓸 수 있게 만들었습니다.

## ️️️️📄 List

작성 된 글을 볼 수 있는 페이지입니다. 수정은 불가하고, 삭제만 가능합니다.
처음 기획이 한 줄 메모장이여서 각 메모마다 자세히 보는 기능은 만들지 않았습니다.

## ℹ App Icon

App Icon은 App Icon을 설정하기 위해서는 iOS, Android에 직접 접근하여 각 플랫폼에 맞게 설정 해줘야 합니다.

설정하는 것은 어렵지 않지만, 한 번에 관리 하고 싶다면 아래 package로 flutter 프로젝트 단에서 관리 할 수 있습니다.

[flutter_launcher_icons](https://pub.dev/packages/flutter_launcher_icons)

<br>
새로운 언어, 낯선 UI 체계 등 다양한 부분에서 새로운 도전이였습니다. 항상 새로운 것을 하는 것은 재밌지만, 본인의 것으로 만들려면 꾸준히 공부를 해야합니다.
Dart라는 언어가 별로 매력적이지도 않았고, flutter가 아직 많이 성장하지도 않아서 flutter 프로젝트는 앞으로 특별한 경우가 있지 않은 이상 안 하고 싶었습니다.

![](/image/Flutter-토이-프로젝트-후기-2/flutter-image-3.png)

그런데 2019 Google I/O에서 flutter로 iOS, Android, Desktop App, 그리고 Web을 한 번에 개발 할 수 있게 할거라는 발표를 듣고, 생각이 바꼈습니다.

안 그래도 iOS 개발을 해야 하는 프로젝트가 있었는데, flutter로 개발해도 재밌을거 같습니다.

flutter가 꽃길만 걷길 기원하면서 이번 글 마치겠습니다. 감사합니다

---

**참고**

- [https://developers.googleblog.com/2019/05/Flutter-io19.html](https://developers.googleblog.com/2019/05/Flutter-io19.html)
