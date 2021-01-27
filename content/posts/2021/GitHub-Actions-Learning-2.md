---
title: "[GitHub Action Learning] #2 GitHub Actions 예제"
date: 2021-01-19T23:52:15+09:00
weight: 1
aliases: []
tags: ["CI/CD", "GitHub", "GitHub Actions"]
author: ["Maru"]
showToc: true
TocOpen: true
draft: true
hidemeta: false
disableShare: false
comments: true
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

편의상 GitHub에서 바로 작성 했다.

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

간단하게 GitHub Actions를 만들어 보았다. 이제 YAML 파일에 작성한 코드들을 하나 씩 뜯어보자.

# YAML 파일(workflow 파일) 분석

**`name:`** => workflow의 이름. GitHub Actions 탭의 사이드바에서 확인 할 수 있다.

**`on: `** => workflow 파일은 자동으로 실행시키는 이벤트를 정하는 코드다. push 말고도 [다양한 이벤트](https://docs.github.com/en/actions/reference/events-that-trigger-workflows#webhook-events)를 세팅 할 수 있다. 더 나아가 [branch, tags, path에 대해서 특정 할 수도 있다.](https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions#onpushpull_requestpaths)

**`jobs:`** => 모든 job들을 이 밑에 선언한다.

**`check-bats-version:`** => job의 이름을 정의한다.

**`runs-on: ubuntu-latest`** => job들이 Ubuntu linux runner(서버)에서 실행되게 명시 해줌을 의미한다. GitHub에서는 [또 다른 운영체제](https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions#jobsjob_idruns-on)도 지원해주고 심지어 자기가 만든 runner도 명시 할 수 있다.

**`steps`** => `check-bats-version` job이 실행할 step들을 이 밑에 선언한다.

**`- uses: actions/checkout@v2`** => `uses` 키워드는 `actions/checkout`라는 이름을 가진 액션에서 `v2` 버전을 community action에서 가지고 오라고 job에게 요청하는 역할을 한다. [actions/checkout@v2](https://github.com/marketplace/actions/checkout)은 레포지토리를 checkout하고 runner에 다운로드하여 코드를 실행하게 해주는 action이다. 그렇기 때문에 나의 레포지토리의 코드를 다루는 모든 workflow에서는 이 action을 사용해야 한다.

**`- uses: actions/setup-node@v1`** => node 패키지를 runner에 설치해주는 action이다.

**`- run: npm install -g bats`** => `run` 키워드는 runner에게 작성한 코드를 명령어로 실행하게 해준다.

**`- run: bats -v`** => 명령어를 실행한다.

정리하는 느낌으로 전체 과정을 이미지로 보고 넘어가자

{{< figure src="/2021/GitHub-Actions-Learning-2/workflow-image.png" caption=" " >}}

{{< line_break >}}
{{< line_break >}}

[다음 글]({{< relref "/posts/2021/GitHub-Actions-Learning-3.md" >}} "Next Post")에서는 workflow 파일에서 변수를 사용하는 방법에 대해 알아보겠다.

---

**Reference**

- [Introduction to GitHub Actions](https://docs.github.com/en/free-pro-team@latest/actions/learn-github-actions/introduction-to-github-actions#the-components-of-github-actions)
