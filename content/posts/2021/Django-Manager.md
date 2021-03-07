---
title: "Django Manager"
date: 2021-02-18T00:26:52+09:00
weight: 1
aliases: []
tags: ["django", "python"]
author: ["Maru"]
showToc: true
TocOpen: true
draft: true
hidemeta: false
disableShare: false
comments: true
---

최근에 Spring 공부를 시작했는데, service layer라는 개념이 참 괜찮아 보였다. 그래서 Django에도 service layer를 사용한 예제가 있는지 찾아 본 결과, service layer 보다는 _model method_, _Manager_, *QuerySet*을 이용해서 `models.py`를 확장시키는 방향을 지향한다고 한다.(참고로 실무에서 Django를 사용 중이다)
