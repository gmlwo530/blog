---
title: "[GitHub Action Learning] #1 GitHub Actions 소개"
date: 2021-01-10T01:58:03+09:00
weight: 1
aliases: []
tags: ["CI/CD"]
author: ["Maru"]
showToc: true
TocOpen: true
draft: true
hidemeta: false
disableShare: false
comments: false
---

최근에 개인 프로젝트를 여러 개 시작 했다. 배포에 대한 걱정을 좀 줄여보고자 처음으로 CI/CD를 공부해보자 마음 먹었다.

CI/CD에 대해서 아는게 하나도 없어서, 서칭을 하다가 그저 탭으로만 마주한 **GitHub Actions**이 CI/CD 툴인걸 알게 되었다. 굉장히 많은 툴들이 있었는데, GitHub Actions은 GitHub에서 CI/CD에 대한 고민도 바로 해결 해주는 느낌이였다.

이렇게 툴은 결정했고 [GihHub Actions Learning 문서](https://docs.github.com/en/free-pro-team@latest/actions/learn-github-actions)를 보면서 공부한 내용을 블로그에 정리했다.

문서의 내용이 많으니 블로그 글은 나눠서 게시하겠다.

# CI/CD 란?

CI/CD에 대한 개념은 굉장히 많은 자료가 있다. 그렇기 때문에 간단하게만 적고 넘어가겠다.

- CI(Continuous Integration) : Build, Test를 실시하는 프로세스를 상시로 실시 해줌으로써 품질을 유지하는게 목표다.

- CD(Continuous Delivery or Continuous Deploy) : 소프트웨어를 더 빠르게, 더 주기적으로 빌드하고 테스트하고 출시하는 것을 목표로 한다.

# GitHub Actions 란?

GitHub Actions은 CI/CD를 포함하여 다양한 소프트웨어 개발 과정을 자동화 해준다. 아마 CI/CD 용도로 제일 많이 사용하겠지만, [awesome-actions](https://github.com/sdras/awesome-actions)(괜찮은 GitHub Actions들을 리스트업 해놓은 레포지토리다.)를 보면 GitHub Actions으로 테스트나 배포 자동화 말고도 _주기적으로 SMS 보내기_, _모니터링_ 등 다양한 것들을 만들 수 있다.

GitHub Actions은 특정 이벤트가 발생하면 실행된다. 예를 들어, 개발자가 push(이벤트)를 하면 정의 해놓은 GitHub Actions이 작동하게 된다.

그림으로 한 번 더 살펴보면,

{{< figure src="/2021/GitHub-Actions-Learning-1/overview-actions-simple.png" caption=" " >}}

이벤트가 발생하면 Job을 포함하고 있는 **workflow**가 자동적으로 실행된다. 그러면 workflow 안의 Job은 action을 순서대로 실행시키기 위해 step을 사용한다.

이런 방식으로 테스트 또는 배포 등 GitHub Actions가 작동한다.

# GitHub Actions의 컴포넌트들

Job을 실행하기 위해 함께 작동하는 GitHub Actions의 컴포넌트들을 알아보자. 나무를 보기 전에 그림으로 숲을 한 번 보자.

{{< figure src="/2021/GitHub-Actions-Learning-1/overview-actions-design.png" caption=" " >}}

### Workflows

workflow는 레포지토리에 추가되는 자동화된 절차다.
Workflow는 하나 이상의 job으로 구성되고 이벤트가 발생 될 때 실행된다. 또는 [cron](https://en.wikipedia.org/wiki/Cron)처럼 주기적으로 workflow를 실행 시킬 수 있다.

### Events

workflow를 실행시키는 특정한 행동이다.
예를 들어, 우리에게 친숙한 GitHub에 커밋을 push하기, issue 또는 PR 생성하기 등이 있다. 이것 말고 다양한 이벤트들을 [공식 문서에 정리 된 리스트](https://docs.github.com/en/free-pro-team@latest/actions/reference/events-that-trigger-workflows)를 통해 알 수 있다.

### Jobs

Job은 step들의 집합이다.
Job은 같은 runner(서버. 뒤에서 자세히 설명한다)에서 실행 되는 step(테스크. 뒤에서 자세히 설명한다)들의 집합이다.
기본적으로 workflow는 여러 개의 job들을 병렬로 실행한다. 물론 순차적으로 실행 할 수 있다. A job의 결과에 따라 B job을 실행 시키고 싶을 수도 있으니깐 말이다.

### Steps

Step은 job 안에서 명령어를 실행시키는 테스크다.
step은 action(명령어. 뒤에서 자세히 설명한다) 또는 쉘 명령어로 이루어진다. step들은 같은 runner에서 실행되고, actions들끼리 데이터를 공유 할 수 있게 해준다.

### Actions

Action은 step에서 job을 만드는 독립적인 명령어다.
자신이 직접 만들거나 [GitHub Marketplace](https://github.com/marketplace?type=actions)에서 만들어진 action을 사용 할 수 있다. Action을 사용하기 위해서, 무조건 step에 action을 넣어야 한다.
필자는 Action의 개념이 정확히 이해되지 않았다. 추가 설명보다는 예제를 만들면서 더 이해 해보도록 하겠다.

### Runners

Runner는 [GitHub Actions runner application](https://github.com/actions/runner)가 설치 된 서버다.
GitHub Actions runner application는 GitHub Actions workflow으로부터 job을 실행시켜주는 어플리케이션이다.
Runner는 한 번에 하나의 job을 실행시키며 과정, 로그, 결과를 기록해서 GitHub에게 돌려준다. 이후 예제에서 GitHub Actions를 GitHub Actions에 실행시키면 로그들이 보일 것이다.

이렇게 나무들을 알아보았다. 마지막으로 숲을 보며 스스로 정리해보자.

{{< figure src="/2021/GitHub-Actions-Learning-1/overview-actions-design.png" caption=" " >}}

다음 글에서는 예제를 만들어보겠다.

---

**Reference**

- [Introduction to GitHub Actions](https://docs.github.com/en/free-pro-team@latest/actions/learn-github-actions/introduction-to-github-actions#the-components-of-github-actions)
- [awesome-actions](https://github.com/sdras/awesome-actions)
