---
title: "Flat Data 소개"
date: 2021-06-03T21:28:02+09:00
weight: 1
aliases: []
tags: ["GitHub Actions", "GitHub"]
author: ["Maru"]
showToc: true
TocOpen: true
draft: true
hidemeta: false
disableShare: false
comments: true
---

# 소개

[Flat Data](https://octo.github.com/projects/flat-data)는 [GitHub CTO](https://octo.github.com/)라는 팀에서 만든 프로젝트다.

[Simon Willison이라는 개발자가 작년 캘리포니아 산불을 추적하려고 정부 웹사이트의 변경사항을 Git에 수집](https://simonwillison.net/2020/Oct/9/git-scraping/)한 것에 아이디어를 얻어 프로젝트를 만들었다고 한다.

GitHub Actions를 사용해서 데이터를 수집하고 변환해서 보여주는 프로젝트다.

관련 된 프로젝트가 총 3개가 있는데

{{< figure src="/2021/flat-data-introduce/projects.png" caption="[https://octo.github.com/projects/flat-data](https://octo.github.com/projects/flat-data)" align=center >}}

이거 3개만 있으면 데이터를 수집하고 변환하고 보여주는게 가능하다.

### [Flat Action](https://github.com/marketplace/actions/flat-data)

GitHub Action 프로젝트다. 덕분에 특별한 따로 관리 해줘야 하는 infra 없이 데이터를 cron을 통해 주기적으로 가져올 수 있다.

### [Flat Editor](https://marketplace.visualstudio.com/items?itemName=GitHubOCTO.flat)

Visual Studio Code의 Extension이다. 이 프로젝트 보고 flat-data라는 프로젝트가 굉장히 친절하다는 것을 느꼈다. 텍스트 입력 조금과 몇 번의 버튼 클릭이면 데이터 수집을 할 수 있는 Github Action 용 `yaml` 파일이 만들어진다.

<!-- {{< video src="/2021/flat-data-introduce/step1.webm" >}} -->

### [Flat Viewer](https://github.com/githubocto/flat-viewer)

React로 만들어진 웹사이트다. flat-data repository 이름과 그 소유자 정보를 입력하면 flat-data로 수집한 데이터를 굉장히 깔끔한 테이블로 보여준다.

{{< figure src="/2021/flat-data-introduce/flat-view-img.png" caption="[https://github.com/githubocto/flat-viewer](https://github.com/githubocto/flat-viewer)" align=center >}}

# 직접 만들어보자.

홈페이지에는 튜토리얼이 굉장히 잘 되어 있다.

예제가 몇 개 있지만 대표적인 예제인 비트코인 정보를 가져오는 예제를 만들어보자.

---

**Reference**

- [https://octo.github.com/projects/flat-data](https://octo.github.com/projects/flat-data)
- [https://blog.outsider.ne.kr/1551?category=38](https://blog.outsider.ne.kr/1551?category=38)
- [https://simonwillison.net/2020/Oct/9/git-scraping/](https://simonwillison.net/2020/Oct/9/git-scraping/)
