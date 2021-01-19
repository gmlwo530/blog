---
title: "[GitHub Action Learning] #2 GitHub Actions 예제"
date: 2021-01-19T23:52:15+09:00
weight: 1
aliases: []
tags: ["CI/CD", "GitHub"]
author: ["Maru"]
showToc: true
TocOpen: true
draft: true
hidemeta: false
disableShare: false
comments: false
---

GitHub Actions는 YAML로 events, jobs, 그리고 steps를 정의한다. 그리고 YAML 파일은 레포지토리의 `.github/workflows`에 저장된다. 이 YAML을 직접 작성하고 Actions를 실행 시켜보자.

만들 예제는 npm으로 [bats](https://www.npmjs.com/package/bats)를 설치하고 `bats -v`를 실행하는 간단한 액션이다.

{{< line_break >}}

# 예제 만들기

### 1. 레포지토리 생성 및 YAML 파일 작성

GitHub에서 레포지토리를 새로 만들고 `.github/workflows`에 YAML 파일을 작성한다.

{{< figure src="/2021/GitHub-Actions-Learning-2/example_pic_1.png" caption=" " >}}

```yaml
# https://docs.github.com/en/actions/learn-github-actions/introduction-to-github-actions#create-an-example-workflow
name: learn-github-actions
on: [push]
jobs:
  check-bats-version:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v1
      - run: npm install -g bats
      - run: bats -v
```

편의상 GitHubs에서 바로 작성 했다.

{{< figure src="/2021/GitHub-Actions-Learning-2/example_pic_2.png" caption=" " >}}

{{< line_break >}}

### 2. commit 후 push

작성한 YAML 파일을 push하자.

{{< figure src="/2021/GitHub-Actions-Learning-2/example_pic_3.png" caption=" " >}}

`on: [push]` 구문(아래에서 자세히 알아보겠다)에 의해서 push를 하게 되면 actions이 실행 된다.

{{< line_break >}}

### 3. Actions 확인

GitHub에서 actions이 실행 되는 과정과 steps 별로 활동을 깔끔한 ui로 볼 수 있다.

{{< figure src="/2021/GitHub-Actions-Learning-2/example_pic_4.png" caption=" " >}}

{{< figure src="/2021/GitHub-Actions-Learning-2/example_pic_5.png" caption=" " >}}

사이드 바에서 jobs 별로 확인 할 수 있다.

{{< figure src="/2021/GitHub-Actions-Learning-2/example_pic_6.png" caption=" " >}}

{{< line_break >}}

---

**Reference**

- [Introduction to GitHub Actions](https://docs.github.com/en/free-pro-team@latest/actions/learn-github-actions/introduction-to-github-actions#the-components-of-github-actions)
